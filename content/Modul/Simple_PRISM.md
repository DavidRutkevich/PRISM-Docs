---

title: "PRISM – Ein orchestrales Modul für unvollständige multimodale Daten"  
description: "Erklärung des PRISM-Moduls für multimodale Segmentierung in der medizinischen Bildverarbeitung"  
date: 2025-01-26  
math: true  

---

Stellen Sie sich ein großes Orchester vor, in dem verschiedene **Instrumente** zusammenwirken, um ein musikalisches Meisterwerk zu erzeugen. Manchmal fehlt ein Instrument – beispielsweise aufgrund von Krankheit – oder es spielt verstimmt. Dennoch soll das gesamte Orchester **harmonisch** klingen. Ähnlich verhält es sich in der medizinischen Bildverarbeitung, wenn einzelne MRT-Sequenzen (**Modalitäten**) fehlen oder nur in reduzierter Qualität vorliegen. Das Ziel von **PRISM** (Präferenzbasierte Regulierung und Selbstdistillation) besteht darin, trotz unvollständiger „Instrumente“ einen **harmonischen Gesamtklang** zu erzielen – sprich eine robuste Segmentierung.

Der vorliegende Artikel erläutert detailliert, wie das PRISM-Modul aufgebaut ist, wie es in bestehende Segmentierungsarchitekturen integriert werden kann und weshalb es in Szenarien mit fehlenden Modalitäten zu besonders stabilen Ergebnissen führt.

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

In einem Orchester hat jedes Instrument seine eigene **Klangfarbe** und deckt unterschiedliche **Frequenzbereiche** ab. Übertragen auf die medizinische Bildverarbeitung entspricht dies:

- **Instrument** = **Modalität** (z. B. T1, T2, FLAIR, T1ce)  
- **Gesamtklang** = **Fusionsmodell**, das alle verfügbaren Modalitäten vereint

Häufig fehlen jedoch einzelne MRT-Sequenzen, weil sie nicht immer aufgenommen werden können. Dies führt zu einem „unvollständigen Orchester“ und beeinträchtigt in klassischen Modellen, die alle Modalitäten gleichberechtigt voraussetzen, die Gesamtperformance.

---

## 2. Die Figur: Klangwellen und Instrumentenbeiträge <a name="klangwellen-und-instrumentenbeitraege"></a>

![Klangwellen und Instrumentenbeiträge (Differenzvergleich à la KL-Divergenz in PRISM)](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/poster_plot.png)

In der Abbildung sind mehrere Wellenkurven dargestellt:

- Die **weiße Kurve** symbolisiert den **Gesamtklang**, analog zum Fusionspfad, der alle verfügbaren Modalitäten kombiniert.  
- Die **bunten Kurven** (z. B. gelb, orange, lila) repräsentieren jeweils eine einzelne **Modalität**. Je nach Qualität und Vorhandensein klingt ein Instrument lauter (höhere Amplitude) oder leiser.

Die Qualität und Präsenz einzelner Modalitäten beeinflusst den Gesamtklang – Modalitäten, die verstärkt vertreten sind, können den Gesamteindruck dominieren, während schwächere Modalitäten zu wenig beitragen. PRISM zielt darauf ab, durch gezielte Regularisierung und Distillation, selbst bei fehlenden oder verrauschten Modalitäten, einen harmonischen Gesamtklang (weiße Linie) zu erzielen. Hierbei wird unter anderem ein KL-Divergenz-basierter Abgleich eingesetzt, der an das Abstimmen von Instrumenten erinnert.

---

## 3. Partitur und Dirigent: Überblick über das PRISM-Modul <a name="partitur-und-dirigent"></a>

PRISM fungiert als eine Art **Dirigent** und **Partitur** in einem:
- Der **Dirigent** (Fusionspfad) übernimmt die Gesamtkoordination, indem er alle Modalitäten zusammenführt.
- Die **Partitur** (Loss-Funktionen und Regularisierungstermen) regelt, wie laut oder leise jedes Instrument (Modalität) spielt und wie dessen Beitrag an den Gesamtklang angepasst wird.

Konkret wird PRISM in zwei zentrale Bausteine unterteilt:

1. **Selbstdistillation:**  
   – Jede Modalität lernt nicht nur aus den eigenen Daten, sondern orientiert sich zusätzlich an der Ausgabe des Fusionspfads.
2. **Präferenzbasierte Regulierung:**  
   – Ein Mechanismus passt das Lerngewicht einzelner Modalitäten dynamisch an, um zu verhindern, dass dominante Modalitäten das Training überwältigen.

Zur Umsetzung wird eine **Indikationsmatrix** \(C\) eingeführt, welche das Vorhandensein (1) oder Fehlen (0) einer Modalität codiert:

\[
C_{nm} =
\begin{cases}
1, & \text{wenn Modalität \(m\) für Beispiel \(n\) vorliegt},\\[1mm]
0, & \text{wenn Modalität \(m\) fehlt}.
\end{cases}
\]

---

## 4. Baustein 1: Selbstdistillation <a name="baustein-1-selbstdistillation"></a>

### Konzept: Internes Teacher–Student-Verfahren

Anstatt ein separates Lehrer- und ein Schüler-Modell zu trainieren, übernimmt das Netzwerk selbst beide Rollen:
- **Fusionspfad (Teacher):**  
  Der Fusionspfad kombiniert alle verfügbaren Modalitäten zu einem Gesamtklang und liefert so eine Referenzausgabe.
- **Modalitäts-Encoder (Student):**  
  Jeder Modalitäts-Encoder lernt zusätzlich aus der Ausgabe des Fusionspfads, um seine eigenen Vorhersagen zu verbessern.

#### Selbstdistillation auf Pixel-Ebene

Auf der Pixel- bzw. Voxel-Ebene wird die **Kullback-Leibler-Divergenz** zwischen den Logits des Fusionspfads und denen der einzelnen Modalitäten minimiert. Mithilfe eines Temperaturparameters \(\mu\) wird die Ausgabe geglättet:

\[
L_{\text{pixel}}^m = 
\sum_{l=0}^{L}
  KL\Bigl[
    \sigma\Bigl(\frac{z_n^{m,l}}{\mu}\Bigr)
    \,\Big\|\,
    \sigma\Bigl(\frac{z_n^l}{\mu}\Bigr)
  \Bigr].
\]

Diese Methode sorgt dafür, dass die unimodalen Pfade ähnliche lokale Vorhersagen wie der Fusionspfad erzielen.

#### Selbstdistillation auf Semantik-Ebene (Prototypen)

Auf der globalen Ebene werden für jede Klasse \(k\) Prototypen gebildet. Für jedes Beispiel \(n\) wird der Lehrer-Prototyp \(c^t_{n,k}\) als Durchschnitt der Pixel-Features berechnet, während der Schüler-Prototyp \(c^{m,s}_{n,k}\) analog für Modalität \(m\) bestimmt wird. Die Differenz zwischen den Kosinus-Ähnlichkeiten (zwischen Pixel-Features und Prototypen) des Lehrers und des Schülers wird minimiert:

\[
L_{\text{proto}}^m =
\sum_i \sum_{k=1}^K \Bigl\|
  S_{n,k}^{m,s}(i) - S_{n,k}^t(i)
\Bigr\|_2^2.
\]

Durch diesen Abgleich werden globale semantische Informationen an den unimodalen Pfaden verankert.

---

## 5. Baustein 2: Präferenzbasierte Regulierung <a name="baustein-2-präferenzbasierte-regulierung"></a>

### Motivation und Konzept

Wie in einem Orchester können dominante Instrumente den Klang übermäßig beeinflussen, während leise Instrumente in den Hintergrund treten. Übertragen auf die medizinische Bildverarbeitung können Modalitäten, die häufig oder in hoher Qualität vorliegen, das Training dominieren, während weniger häufige oder verrauschte Modalitäten untergehen. Um einen ausgewogenen Gesamtklang zu gewährleisten, wird eine **relative Präferenz** definiert.

#### Relative Präferenz (RP)

Für jede Modalität \(m\) wird eine semantische Distanz \(D_n^m\) zwischen dem Schüler- und dem Lehrer-Pfad für ein Beispiel \(n\) berechnet. Anschließend wird der Durchschnittswert \(\bar{D}_n\) aller verfügbaren Modalitäten ermittelt. Die relative Präferenz wird dann wie folgt definiert:

\[
RP_n^m = 1 - \frac{D_n^m}{\bar{D}_n}.
\]

- **Wenn \(RP_n^m < 0\):**  
  Wird signalisiert, dass die Modalität im Vergleich zum Durchschnitt schwach repräsentiert ist.  
- **Wenn \(RP_n^m > 0\):**  
  Ist die Modalität gut repräsentiert.

#### Dynamische Anpassung der Lernraten

Die relative Präferenz wird genutzt, um den modalspezifischen Gewichtungsfaktor \(E^m\) anzupassen:

\[
E^m_{r+1} = E^m_r - \lambda \, RP_m,
\]

wobei \(\lambda\) ein kleiner Lernratenparameter ist.  
- **Bei negativen \(RP_m\)** wird \(E^m\) erhöht, sodass der Verlust \(L_{\text{pixel}}\) dieser Modalität stärker gewichtet und das Lernen beschleunigt wird.  
- **Bei positiven \(RP_m\)** wird \(E^m\) verringert, um eine Überbetonung bereits gut repräsentierter Modalitäten zu vermeiden.

Gleichzeitig wird der prototypische Verlust \(L_{\text{proto}}^m\) verstärkt, wenn \(RP_m < 0\), um die globale Repräsentation der schwächeren Modalitäten zu verbessern.

Die berechneten relativen Präferenzwerte werden zudem in Gewichtungen umgewandelt, die als Wahrscheinlichkeiten interpretiert werden können, wodurch der Einfluss jeder Modalität auf den Gesamtverlust dynamisch skaliert wird.

---

## 6. Gesamtverlustfunktion: Wie alles zusammenklingt <a name="gesamtverlustfunktion"></a>

Die finale Verlustfunktion von PRISM integriert alle Komponenten – den Segmentierungsloss, die selbstdistillierenden Verluste und die präferenzbasierte Regularisierung – zu einem harmonischen Gesamt-Loss:

\[
L = L_{\text{seg}} +
\sum_{\substack{m=1,\dots,M \\ C_{nm}=1}}
\Bigl(
  \gamma_1 \,E^m\, L_{\text{pixel}}^m +
  \gamma_2 \,\omega_n^m\, L_{\text{proto}}^m
\Bigr),
\]

wobei  
- \(L_{\text{seg}}\) den Basis-Segmentierungsloss (z. B. Dice + Cross-Entropy) darstellt,  
- \(L_{\text{pixel}}^m\) und \(L_{\text{proto}}^m\) die lokalen und globalen Distillationsverluste bezeichnen,  
- \(E^m\) und \(\omega_n^m\) die dynamisch angepassten Gewichtungsfaktoren sind,  
- \(\gamma_1\) und \(\gamma_2\) als Hyperparameter die jeweiligen Verlustkomponenten gewichten.

Diese Kombination sorgt dafür, dass alle Modalitäten, selbst wenn sie unvollständig oder schwach vertreten sind, gleichwertig in den Trainingsprozess einfließen und ein stabiler Gesamtklang erzielt wird.

---

## 7. Integration in bestehende Architekturen <a name="integration-in-bestehende-architekturen"></a>

PRISM wurde so konzipiert, dass es als **Plug-and-Play-Modul** in verschiedene bestehende Segmentierungsarchitekturen integriert werden kann – sei es in klassischen U-Net-Strukturen oder in modernen transformerbasierten Modellen wie mmFormer oder RFNet.

### 7.1 Orchestrierung

- **Encoderausgänge pro Modalität:**  
  Jede Modalität wird separat enkodiert, sodass die individuellen „Instrumentengruppen“ erhalten bleiben.  
- **Fusionspfad als Dirigent:**  
  Der Fusionspfad vereint alle Encoderausgänge und bildet so den „Gesamtklang“.  
- **Integration der PRISM-Module:**  
  An den Schnittstellen zwischen den Encodern und dem Fusionspfad werden die selbstdistillierenden und regularisierenden Komponenten angewandt. Dies erfordert nur geringfügige Anpassungen, etwa das Bereitstellen der Indikationsmatrix \(C\) und das Einfügen der zusätzlichen Loss-Terme.

### 7.2 Praktische Details

- **Datenmaskierung:**  
  Fehlende Modalitäten werden durch die Indikationsmatrix \(C_{nm}\) maskiert, sodass sie während des Trainings nicht berücksichtigt werden.  
- **Hyperparameter:**  
  Neben den Standardparametern werden die Temperatur \(\mu\), die Gewichtungen \(\gamma_1\) und \(\gamma_2\) sowie der Lernratenparameter \(\lambda\) festgelegt, um den Einfluss der einzelnen Komponenten optimal zu steuern.

---

## 8. Fazit und Ausblick <a name="fazit-und-ausblick"></a>

### Zusammenfassung

Das PRISM-Modul bietet einen innovativen Ansatz zur Integration unvollständiger multimodaler Daten in der medizinischen Bildverarbeitung. Durch die Kombination aus **selbstdistillierender Lernstrategie** und **präferenzbasierter Regularisierung** wird sichergestellt, dass auch Modalitäten, die seltener vorhanden oder verrauscht sind, optimal in den Gesamttrainingsprozess einfließen. Dies führt zu einer stabilen und präzisen Segmentierung, vergleichbar mit einem harmonisch abgestimmten Orchester – selbst wenn einzelne Instrumente fehlen oder verstimmt sind.

### Ausblick

Mögliche Erweiterungen umfassen:
- **Erweiterte Fusionstechniken:**  
  Die Integration von komplexeren Mechanismen wie Cross-Modal Attention könnte die Verbindung zwischen den Modalitäten weiter verbessern.
- **Selbstüberwachte Lernverfahren:**  
  Die Kombination von PRISM mit selbstüberwachten oder halbüberwachten Ansätzen könnte den Bedarf an umfangreichen, vollannotierten Datensätzen weiter reduzieren.
- **Anwendung in weiteren Domänen:**  
  Über die medizinische Bildverarbeitung hinaus kann PRISM in anderen Bereichen, etwa im IoT oder in der Sensorfusion, eingesetzt werden, um die Integration heterogener Datenquellen zu optimieren.

---

Abschließend wird festgehalten:  
**PRISM** orchestriert die verschiedenen Modalitäten in einem einheitlichen System, sodass trotz unvollständiger Daten ein harmonischer Gesamtklang erzielt wird. Die Kombination aus selbstdistillierender Lernstrategie und präferenzbasierter Regularisierung stellt sicher, dass alle Modalitäten – gleich ob dominant oder schwach – ihren Beitrag zur Segmentierung leisten. Dies ebnet den Weg für robuste und zuverlässige Segmentierungsergebnisse in der medizinischen Bildverarbeitung und darüber hinaus.