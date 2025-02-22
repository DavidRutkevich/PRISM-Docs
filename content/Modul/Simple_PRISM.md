---

title: "PRISM – Ein orchestrales Modul für unvollständige multimodale Daten"  
description: "Erklärung des PRISM-Moduls für multimodale Segmentierung in der medizinischen Bildverarbeitung"  
date: 2025-01-26  
math: true  

---

Stell Dir vor, ich leite ein großes Orchester, in dem verschiedene **Instrumente** zusammenwirken, um ein musikalisches Meisterwerk zu schaffen. Manchmal fehlt jedoch ein Instrument (z. B. wegen Krankheit) oder es klingt verstimmt. Dennoch soll das gesamte Orchester **harmonisch** klingen. Ähnlich verhält es sich in der medizinischen Bildverarbeitung, wenn einzelne MRT-Sequenzen (**Modalitäten**) fehlen oder nur in schlechter Qualität vorliegen. Das Ziel von **PRISM** (Präferenzbasierte Regulierung und Selbstdistillation) ist, trotz unvollständiger „Instrumente“ einen **harmonischen Gesamtklang** zu erzielen – sprich eine **robuste Segmentierung**.

In diesem Artikel erkläre ich **im Detail**, wie das PRISM-Modul aufgebaut ist, wie es sich in bestehende Segmentierungsarchitekturen integriert und warum es in Szenarien mit fehlenden Modalitäten zu besonders stabilen Ergebnissen führt.

---

## Inhaltsverzeichnis

1. [Die Rollen im Orchester: Einzelmodalitäten und Fusionsmodell](#die-rollen-im-orchester)
2. [Die Figur: Klangwellen und Instrumentenbeiträge](#klangwellen-und-instrumentenbeitraege)
3. [Partitur und Dirigent: Überblick über das PRISM-Modul](#partitur-und-dirigent)
4. [Baustein 1: Selbstdistillation (Teacher–Student-Ansatz im Modell)](#baustein-1-selbstdistillation)
5. [Baustein 2: Präferenzbasierte Regulierung (Re-Balancing)](#baustein-2-präferenzbasierte-regulierung)
6. [Gesamtverlustfunktion: Wie alles zusammenklingt](#gesamtverlustfunktion)
7. [Integration in bestehende Architekturen](#integration-in-bestehende-architekturen)
8. [Fazit und Ausblick](#fazit-und-ausblick)

---

## 1. Die Rollen im Orchester <a name="die-rollen-im-orchester"></a>

In meinem Orchester hat jedes Instrument seine eigene **Klangfarbe** und deckt verschiedene **Frequenzbereiche** ab. Übertragen auf die medizinische Bildverarbeitung bedeutet das:

- **Instrument** = **Modalität** (z. B. T1, T2, FLAIR, T1ce)  
- **Gesamtklang** = **Fusionsmodell**, das alle verfügbaren Modalitäten vereint

Oftmals fehlen jedoch einzelne MRT-Sequenzen, weil sie beispielsweise nicht aufgenommen werden konnten. Dadurch wird das Orchester „unvollständig“. In der Praxis führt dies häufig zu **Performance-Einbrüchen** bei klassischen Modellen, die alle Modalitäten gleichberechtigt erwarten.

---

## 2. Die Figur: Klangwellen und Instrumentenbeiträge <a name="klangwellen-und-instrumentenbeitraege"></a>

![Klangwellen und Instrumentenbeiträge (Differenzvergleich à la KL-Divergenz in PRISM)](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/poster_plot.png)

In der obigen Abbildung zeige ich verschiedene Wellenkurven:

- Die **weiße Kurve** stellt den **Gesamtklang** dar – analog zum Fusionspfad, der alle verfügbaren Modalitäten zusammenführt.  
- Die **bunten Kurven** (gelb, orange, lila) repräsentieren jeweils eine einzelne **Modalität** (Instrument). Je nach Qualität und Verfügbarkeit kann eine Modalität lauter (höhere Amplitude) oder leiser klingen.

Je nachdem, wie gut oder schlecht eine Modalität vorliegt, beeinflusst sie den Gesamtklang stärker oder schwächer. Mein Ziel mit **PRISM** ist es, trotz dieser Schwankungen und Ausfälle (z. B. wenn eine Modalität fehlt) am Ende eine möglichst **harmonische Gesamtkurve** (weiße Linie) zu erzeugen. Dazu setze ich unter anderem einen KL-Divergenz-basierten Abgleich ein, der wie ein „Abstimmen“ der Instrumente wirkt.

---

## 3. Partitur und Dirigent: Überblick über das PRISM-Modul <a name="partitur-und-dirigent"></a>

Für mich funktioniert PRISM wie ein **Dirigent** und eine **Partitur** in einem:  
- Der **Dirigent** (mein **Fusionspfad**) hat den Überblick über alle verfügbaren Instrumente (Modalitäten) und erzeugt den „Gesamtklang“.  
- Die **Partitur** (meine **Loss-Funktionen** und **Regularisierungstermen**) regelt, wie laut oder leise jedes Instrument spielt und wie es sich an den Gesamtklang anpasst.

Konkret unterteile ich PRISM in zwei zentrale Bausteine:

1. **Selbstdistillation**  
   – Jede Modalität lernt nicht nur aus den eigenen Daten, sondern auch aus dem Fusionsmodell.  
2. **Präferenzbasierte Regulierung**  
   – Ein Mechanismus, der das Lerngewicht einzelner Modalitäten dynamisch anpasst, damit keine Modalität „untergeht“ oder zu dominant wird.

Zur Realisierung verwende ich eine **Indikationsmatrix** für das Vorhandensein (1) oder Fehlen (0) einer Modalität:

\[
C_{nm} =
\begin{cases}
1, & \text{wenn Modalität \(m\) für Beispiel \(n\) vorliegt},\\[1mm]
0, & \text{wenn Modalität \(m\) fehlt}.
\end{cases}
\]

---

## 4. Baustein 1: Selbstdistillation <a name="baustein-1-selbstdistillation"></a>

### Idee: „Teacher–Student“ innerhalb eines einzelnen Netzwerks

Statt ein großes „Lehrer“-Modell und ein kleineres „Schüler“-Modell separat zu trainieren, realisiere ich in PRISM beides **in einem** Netzwerk. Dabei übernimmt der **Fusionspfad** die Lehrerrolle und jede **einzelne Modalität** die Schülerrolle.

1. **Fusionspfad (Teacher)**  
   – Dieser stellt den Gesamtklang dar, indem er alle verfügbaren Modalitäten vereint.  
2. **Modalitäts-Encoder (Students)**  
   – Jede Modalität wird separat enkodiert und versucht, sich am Fusionsklang zu orientieren.

### Selbstdistillation auf Pixel-Ebene

Auf **Pixel-Ebene** (bzw. Voxel-Ebene) minimiere ich die **Kullback-Leibler-Divergenz** zwischen dem Fusionsausgang (Lehrer-Logits) und den Logits jeder einzelnen Modalität (Schüler). Dadurch lernt jede Modalität, ähnliche Klassifikationsausgaben wie der Fusionspfad zu erzeugen.

### Selbstdistillation auf Semantik-Ebene

Zusätzlich vergleiche ich **globale Klassenrepräsentationen** (sogenannte **Prototypen**). Für jede Klasse (z. B. Tumorgewebe, gesundes Gewebe usw.) bilde ich einen Prototyp aus den Pixel-Merkmalen. Der Unterschied zwischen dem Fusionsprototyp (Lehrer) und dem Modalitätsprototyp (Schüler) wird minimiert, sodass jede Modalität eine Art **Leitmelodie** erhält, an der sie sich orientieren kann.

---

## 5. Baustein 2: Präferenzbasierte Regulierung <a name="baustein-2-präferenzbasierte-regulierung"></a>

### Warum brauche ich das?

In einem echten Orchester könnte es passieren, dass ein sehr lautes Instrument (z. B. Trompete) ständig dominiert, während leise Instrumente (z. B. Oboe) kaum zu hören sind. Übertragen auf mein MRT-Setting heißt das:  
- **Starke Modalität**: hat viele Daten und klare Merkmalsmuster.  
- **Schwache Modalität**: ist selten vorhanden oder verrauscht.

Ohne Regulierung würden die „lauten“ Modalitäten das Training dominieren. **PRISM** sorgt daher für einen **dynamischen Ausgleich**.

### Relative Präferenz (RP)

Ich definiere eine Metrik, die misst, ob eine Modalität tendenziell **über-** oder **unterrepräsentiert** ist. Dazu:

1. Berechne ich eine **Distanz** (etwa mittels KL-Divergenz oder Prototypen-Abstand) zwischen dem **Fusionspfad** und der Modalität \(m\) für ein bestimmtes Trainingsbeispiel \(n\).  
2. Vergleiche diese Distanz mit dem Durchschnitt aller verfügbaren Modalitäten, um festzustellen, ob Modalität \(m\) **vernachlässigt** oder **bevorzugt** wird.  

So erhalte ich die **Relative Präferenz** (RP), die größer als 0 sein kann (Modalität dominiert) oder kleiner als 0 (Modalität hinkt hinterher).

### Lernratenanpassung

Mit dem errechneten RP passe ich die Lernraten bzw. die **Loss-Gewichte** einzelner Modalitäten an:  
- Ist eine Modalität **unterrepräsentiert** (RP < 0), gebe ich ihr ein **höheres Gewicht**, um das Lernen zu beschleunigen.  
- Ist eine Modalität **dominant** (RP > 0), reduziere ich ihr Gewicht, um ein Überwiegen zu verhindern.

So stelle ich einen dynamischen Ausgleich her – ich sorge dafür, dass keine Modalität „zuschreit“, aber auch keine „verstummt“.

---

## 6. Gesamtverlustfunktion: Wie alles zusammenklingt <a name="gesamtverlustfunktion"></a>

Am Ende kombiniere ich alle Bausteine – den Segmentierungsloss, die Selbstdistillation und die präferenzbasierte Regulierung – zu einem **Gesamt-Loss**. Das sieht schematisch so aus:

1. **Segmentierungsloss**  
   – Zum Beispiel die Kombination aus Dice und Cross-Entropy (Ziel: korrekte Segmentierung).  
2. **Selbstdistillation**  
   - **Pixel-Ebene**: Minimierung der KL-Divergenz zwischen Fusion-Logits und Einzelmodalitäts-Logits.  
   - **Semantik-Ebene**: Abgleich der Prototypen (Teacher–Student).  
3. **Präferenzbasierte Regulierung**  
   - Dynamische Anpassung der Loss-Gewichte anhand der relativen Präferenz.

So entsteht für mich eine **harmonische Gesamtpartitur**, in der jeder Instrumentenbeitrag sinnvoll gewichtet ist.

---

## 7. Integration in bestehende Architekturen <a name="integration-in-bestehende-architekturen"></a>

### 7.1 Orchestrierung mit U-Net, RFNet & mmFormer

Egal, ob ich ein **klassisches U-Net** oder ein **transformer-basiertes** Modell (z. B. **mmFormer**, **RFNet**) einsetze – PRISM lasse sich als **Plug-and-Play-Modul** integrieren:

1. **Encoder-Teil pro Modalität**  
   – Diese agieren wie separate Instrumentengruppen, die jeweils ihren Klang beisteuern.  
2. **Fusionspfad**  
   – Der Dirigent, der alle Modalitäten zusammenführt (z. B. über Concatenation, Attention oder Additionen).  
3. **PRISM-Module**  
   - **Selbstdistillation**: Vergleicht die Ausgaben der Modalitäts-Encoder mit der Fusion.  
   - **Präferenzbasierte Regulierung**: Steuert die Lernraten bzw. die Loss-Gewichte.

### 7.2 Implementierungsdetails

- **Datenmaskierung**:  
  Während des Trainings maskiere ich Modalitäten anhand der Indikationsmatrix \(C_{nm}\). Fehlende Modalitäten bleiben „stumm“ (werden nicht enkodiert).  
- **Leichtgewichtige Anpassungen**:  
  Ich muss lediglich an den Punkten „Modalitäts-Encoder-Ausgang“ und „Fusion“ ansetzen, um die Distillations- und Regularisierungstermine zu berechnen.  
- **Hyperparameter**:  
  - Temperaturfaktor für die KL-Divergenz  
  - Gewichtungen der einzelnen Verluste (z. B. \(\gamma_1\) für Pixel-Distillation, \(\gamma_2\) für Prototyp-Distillation)  
  - Lernrate \(\lambda\) für die Aktualisierung der **relativen Präferenz**

---

## 8. Fazit und Ausblick <a name="fazit-und-ausblick"></a>

### Das Ergebnis: Ein harmonisches Orchester trotz fehlender Instrumente

- **Robustheit**: Selbst wenn eine oder mehrere MRT-Sequenzen (Modalitäten) fehlen, erziele ich mit PRISM stabile Segmentierungsergebnisse.  
- **Ausgleich**: Über die dynamische Anpassung werden dominante Modalitäten heruntergeregelt und schwache verstärkt.  
- **Einfache Einbindung**: Das Modul lässt sich problemlos in nahezu jede bestehende Architektur integrieren.

### Mögliche Erweiterungen

- **Erweiterte Fusion**: Einsatz noch komplexerer Mechanismen (z. B. Cross-Modal Attention) für eine noch tiefere Kombination der Modalitäten.  
- **Selbstüberwachte Lernverfahren**: Kombination von PRISM mit selbstüberwachten Ansätzen, um die Menge der benötigten Labels zu reduzieren.  
- **Multi-Domänen-Anwendung**: Über die medizinische Bildverarbeitung hinaus könnte ich PRISM in anderen Bereichen (z. B. Sensorfusion im IoT, Audio-Video-Daten) einsetzen.

---

## Abschließende Worte

Mit **PRISM** sorge ich dafür, dass das Orchester der Modalitäten – selbst wenn einzelne Instrumente fehlen oder verstimmt sind – einen **harmonischen Gesamtklang** erzeugt. Die **Multi-Unit-Selbstdistillation** und die **präferenzbasierte Regulierung** wirken wie zwei dirigierende Hände, die den einzelnen Instrumenten (Modalitäten) den richtigen Einsatz geben und sie dynamisch ausbalancieren.

Falls Du Fragen hast, tiefer in den Code einsteigen möchtest oder Ideen für neue Einsatzzwecke hast, freue ich mich, von Dir zu hören. Gemeinsam können wir das Orchester weiter verfeinern und die Segmentierungsergebnisse in der medizinischen Bildverarbeitung (und darüber hinaus) auf ein neues Level heben.

**Viel Erfolg bei Deinen Experimenten mit PRISM – und denke daran: Selbst wenn ein Instrument fehlt, kann das Orchester großartig klingen, wenn der Dirigent die richtigen Einsätze gibt!**