---
title: "PRISM - Evaluierung"
description: "Eine Analyse der Segmentierungsergebnisse bei unvollständigen multimodalen MRT-Daten unter Verwendung der PRISM-Methodik."
date: 2025-01-26
type: post
draft: false
translationKey: prism_eval
coffee: 1
tags: ['PRISM', 'Evaluierung']
categories: ['Modul']
---

# PRISM – Ein orchestrales Modul für unvollständige multimodale Daten

Stell Dir ein großes Orchester vor, in dem verschiedene **Instrumente** zusammenwirken, um ein musikalisches Meisterwerk zu schaffen. Manchmal fehlt jedoch ein Instrument (z. B. wegen Krankheit) oder es klingt verstimmt. Dennoch soll das gesamte Orchester **harmonisch** klingen. Ähnlich verhält es sich in der medizinischen Bildverarbeitung, wenn einzelne MRT-Sequenzen (**Modalitäten**) fehlen oder nur in schlechter Qualität vorliegen. Das Ziel von **PRISM** (Präferenzbasierte Regulierung und Selbstdistillation) ist, trotz unvollständiger „Instrumente“ einen **harmonischen Gesamtklang** zu erzielen – sprich eine **robuste Segmentierung**.

In diesem Artikel erfährst Du **im Detail**, wie das PRISM-Modul aufgebaut ist, wie es sich in bestehende Segmentierungsarchitekturen integriert und warum es in Szenarien mit fehlenden Modalitäten zu besonders stabilen Ergebnissen führt.

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

In einem Orchester hat jedes Instrument seine eigene **Klangfarbe** und deckt verschiedene **Frequenzbereiche** ab. Übertragen auf die medizinische Bildverarbeitung bedeutet das:

- **Instrument** = **Modalität** (z. B. T1, T2, FLAIR, T1ce)  
- **Gesamtklang** = **Fusionsmodell**, das alle verfügbaren Modalitäten vereint

Manchmal fehlen jedoch einzelne MRT-Sequenzen, zum Beispiel weil sie nicht aufgenommen werden konnten. Damit wird das Orchester „unvollständig“. In der Praxis führt dies oft zu **Performance-Einbrüchen** bei klassischen Modellen, die alle Modalitäten gleichberechtigt erwarten.

---

## 2. Die Figur: Klangwellen und Instrumentenbeiträge <a name="klangwellen-und-instrumentenbeitraege"></a>

![Klangwellen und Instrumentenbeiträge (Differenzvergleich à la KL-Divergenz in PRISM)](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/poster_plot.png)

In der obigen Abbildung siehst Du verschiedene Wellenkurven:

- Die **weiße Kurve** stellt den **Gesamtklang** dar – analog zum Fusionspfad, der alle verfügbaren Modalitäten zusammenführt.  
- Die **bunten Kurven** (gelb, orange, lila) repräsentieren jeweils eine einzelne **Modalität** (Instrument). Manchmal ist eine Modalität lauter (höhere Amplitude), manchmal leiser.  

Je nach Qualität und Verfügbarkeit kann eine Modalität die Gesamtwelle stärker oder schwächer beeinflussen. Ziel von **PRISM** ist es, trotz dieser Schwankungen und Ausfälle (z. B. wenn eine Modalität fehlt) am Ende eine möglichst **harmonische Gesamtkurve** (weiße Linie) zu erzeugen. In PRISM wird hierfür unter anderem ein KL-Divergenz-basierter Abgleich durchgeführt, der wie ein „Abstimmen“ der Instrumente wirkt.

---

## 3. Partitur und Dirigent: Überblick über das PRISM-Modul <a name="partitur-und-dirigent"></a>

PRISM funktioniert wie ein **Dirigent** und eine **Partitur** in einem:  
- Der **Dirigent** (unser **Fusionspfad**) hat den Überblick über alle verfügbaren Instrumente (Modalitäten) und erzeugt den „Gesamtklang“.  
- Die **Partitur** (unsere **Loss-Funktionen** und **Regularisierungstermen**) regelt, wie laut oder leise jedes Instrument spielt und wie es sich an den Gesamtklang anpasst.

Konkret ist PRISM in zwei zentrale Bausteine unterteilt:

1. **Selbstdistillation**  
   – Jede Modalität lernt nicht nur aus den eigenen Daten, sondern auch aus dem Fusionsmodell.  
2. **Präferenzbasierte Regulierung**  
   – Ein Mechanismus, der das Lerngewicht einzelner Modalitäten dynamisch anpasst, damit keine Modalität „untergeht“ oder zu dominant wird.

Dazu nutzen wir eine **Indikationsmatrix** für das Vorhandensein (1) oder Fehlen (0) einer Modalität:

{{< math >}}
$$
C_{nm} =
\begin{cases}
1, & \text{wenn Modalität $m$ für Beispiel $n$ vorliegt},\\
0, & \text{wenn Modalität $m$ fehlt}.
\end{cases}
$$
{{< /math >}}

---

## 4. Baustein 1: Selbstdistillation <a name="baustein-1-selbstdistillation"></a>

### Idee: „Teacher–Student“ innerhalb eines einzigen Netzwerks

Statt ein großes „Lehrer“-Modell und ein kleineres „Schüler“-Modell separat zu trainieren, macht PRISM beides **in einem** Netzwerk. Dabei übernimmt der **Fusionspfad** die Lehrerrolle und jede **einzelne Modalität** die Schülerrolle.

1. **Fusionspfad (Teacher)**  
   – Stellt den Gesamtklang dar, indem er alle verfügbaren Modalitäten vereint.  
2. **Modalitäts-Encoder (Students)**  
   – Jede Modalität wird separat enkodiert und versucht, sich an den Fusionsklang anzupassen.

### Selbstdistillation auf Pixel-Ebene

Auf **Pixel-Ebene** (bzw. Voxel-Ebene) wird die **Kullback-Leibler-Divergenz** zwischen dem Fusionsausgang (Lehrer-Logits) und den Logits jeder einzelnen Modalität (Schüler) minimiert. Das sorgt dafür, dass jede Modalität ähnliche Klassifikationsausgaben lernt wie der Fusionspfad.

### Selbstdistillation auf Semantik-Ebene

Ergänzend dazu vergleichen wir **globale Klassenrepräsentationen** (sogenannte **Prototypen**). Für jede Klasse (z. B. Tumorgewebe, gesundes Gewebe usw.) wird ein Prototyp aus den Pixel-Merkmalen gebildet. Die Differenz zwischen dem Fusionsprototyp (Lehrer) und dem Modalitätsprototyp (Schüler) wird minimiert. Dadurch erhält jede Modalität eine Art **Leitmelodie**, an der sie sich orientieren kann.

---

## 5. Baustein 2: Präferenzbasierte Regulierung <a name="baustein-2-präferenzbasierte-regulierung"></a>

### Warum brauchen wir das?

In einem echten Orchester könnte es passieren, dass ein sehr lautes Instrument (z. B. Trompete) ständig dominiert und leise Instrumente (z. B. Oboe) kaum zu hören sind. Übertragen auf unser MRT-Setting bedeutet das:  
- **Starke Modalität** = hat viele Daten, leichte Merkmalsmuster  
- **Schwache Modalität** = seltene Daten, verrauschte oder unvollständige Sequenzen

Ohne Regulierung würden die „lauten“ Modalitäten das Training dominieren. **PRISM** sorgt daher für ein **dynamisches Ausbalancieren**.

### Relative Präferenz (RP)

Wir definieren eine Metrik, die misst, ob eine Modalität tendenziell **über-** oder **unterrepräsentiert** ist. Die Idee dahinter:

1. Berechne eine **Distanz** (z. B. in Form einer KL-Divergenz oder eines Prototypen-Abstands) zwischen dem **Fusionspfad** und der Modalität \(m\) für ein bestimmtes Trainingsbeispiel \(n\).  
2. Vergleiche diese Distanz mit dem Durchschnitt aller verfügbaren Modalitäten, um zu bestimmen, ob Modalität \(m\) **vernachlässigt** oder **bevorzugt** wird.  

So entsteht die **Relative Präferenz** (RP), die größer 0 sein kann (Modalität wird bevorzugt) oder kleiner 0 (Modalität wird benachteiligt.

### Lernratenanpassung

Mit dieser **RP**-Kennzahl justieren wir die Lernraten bzw. die **Loss-Gewichte** einzelner Modalitäten:  
- Wird eine Modalität **unterdrückt**, bekommt sie ein **höheres Gewicht**, um aufzuholen.  
- Ist eine Modalität **dominant**, wird ihr Lerngewicht etwas reduziert.

Dadurch ergibt sich ein **dynamischer Ausgleich** – keine Modalität soll „zuschreien“, aber auch keine „verstummen“.

---

## 6. Gesamtverlustfunktion: Wie alles zusammenklingt <a name="gesamtverlustfunktion"></a>

Am Ende führen wir alle Bausteine (Segmentierungs-Loss, Selbstdistillation, präferenzbasierte Regulierung) in einem **Gesamt-Loss** zusammen. Schematisch sieht das so aus:

1. **Segmentierungsloss**  
   – z. B. Kombination aus Dice und Cross-Entropy (Lernziel: korrekte Segmentierung)  
2. **Selbstdistillation**  
   - **Pixel-Ebene**: KL-Divergenz zwischen Fusion-Logits und Einzelmodalitäts-Logits  
   - **Semantik-Ebene**: Prototypen-Abgleich (Lehrer–Schüler)  
3. **Präferenzbasierte Regulierung**  
   - Anpassung der Loss-Gewichte anhand der relativen Präferenz

So entsteht eine **harmonische Gesamtpartitur**, in der jeder Instrumentenbeitrag sinnvoll gewichtet ist.

---

## 7. Integration in bestehende Architekturen <a name="integration-in-bestehende-architekturen"></a>

### 7.1 Orchestrierung mit U-Net, RFNet & mmFormer

Egal, ob Du ein **klassisches U-Net** oder ein **transformer-basiertes** Modell (z. B. **mmFormer**, **RFNet**) nutzt – PRISM lässt sich als **Plug-and-Play-Modul** integrieren:

1. **Encoder-Teil pro Modalität**  
   – Wie separate Instrumentengruppen, die jeweils ihren Klang beisteuern.  
2. **Fusionspfad**  
   – Der Dirigent, der alle Modalitäten zusammenführt (z. B. über Concatenation, Attention oder Additionen).  
3. **PRISM-Module**  
   - **Selbstdistillation**: Vergleicht die Ausgaben der Modalitäts-Encoder mit der Fusion.  
   - **Präferenzbasierte Regulierung**: Steuert die Lernrate bzw. die Loss-Gewichte.

### 7.2 Implementierungsdetails

- **Datenmaskierung**:  
  Während des Trainings werden Modalitäten anhand der Indikationsmatrix {{< math >}}$C_{nm}${{< /math >}} maskiert. Fehlende Modalitäten sind komplett „stumm“ (werden nicht enkodiert).  
- **Leichtgewichtige Anpassungen**:  
  Du musst lediglich an den Punkten „Modalitäts-Encoder-Ausgang“ und „Fusion“ einhaken, um die Distillations- und Regulierungstermine zu berechnen.  
- **Hyperparameter**:  
  - Temperaturfaktor für KL-Divergenz  
  - Gewichtungen der einzelnen Verluste (z. B. \(\gamma_1\) für Pixel-Distillation, \(\gamma_2\) für Prototyp-Distillation)  
  - Lernrate \(\lambda\) für die Aktualisierung der **relativen Präferenz**  

---

## 8. Fazit und Ausblick <a name="fazit-und-ausblick"></a>

### Das Ergebnis: Ein harmonisches Orchester trotz fehlender Instrumente

- **Robustheit**: Selbst wenn eine oder mehrere MRT-Sequenzen (Modalitäten) fehlen, erzeugt PRISM ein stabiles Segmentierungsergebnis.  
- **Ausgleich**: Lautere Modalitäten werden automatisch heruntergeregelt, leise bekommen mehr „Verstärkung“.  
- **Einfache Einbindung**: Das Modul kann in fast jede existierende Architektur integriert werden.  

### Mögliche Erweiterungen

- **Erweiterte Fusion**: Einsatz noch komplexerer Mechanismen (z. B. Cross-Modal Attention) für eine noch tiefere Kombination der Modalitäten.  
- **Selbstüberwachte Lernverfahren**: Kombination von PRISM mit selbstüberwachten Ansätzen, um die Menge an benötigten Labels zu reduzieren.  
- **Multi-Domänen-Anwendung**: Über die medizinische Bildverarbeitung hinaus könnte PRISM in anderen Bereichen (z. B. Sensor-Fusion in IoT, Audio-Video-Daten) eingesetzt werden.

---

## Abschließende Worte

Mit **PRISM** sorgen wir dafür, dass das Orchester der Modalitäten trotz fehlender oder „verstimmter“ Instrumente einen **harmonischen Gesamtklang** erzeugt. Die **Multi-Unit-Selbstdistillation** und die **präferenzbasierte Regulierung** sind wie zwei dirigierende Hände, die den einzelnen Instrumenten (Modalitäten) den Einsatz geben und sie dynamisch ausbalancieren.

Falls Du noch Fragen hast, tiefer in den Code eintauchen möchtest oder Ideen für neue Einsatzzwecke hast, freuen wir uns, von Dir zu hören. Gemeinsam können wir das Orchester weiter verfeinern und die Segmentierungsergebnisse in der medizinischen Bildverarbeitung (und darüber hinaus) auf ein neues Level heben.

**Viel Erfolg bei Deinen Experimenten mit PRISM – und denk daran: Selbst mit fehlenden Instrumenten kann ein Orchester großartig klingen, wenn der Dirigent die richtigen Einsätze gibt!**
