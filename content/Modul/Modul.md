
---

title: "PRISM – Relative Präferenz"  
description: "Übersichtliche Erläuterung zur Problemdefinition, Multi-Uni-Selbstdistillation und präferenzbasierten Regulierung in PRISM."  
date: 2025-02-05  
math: true  

---

![Relative Präferenz (Figur 1)](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/8d1a65675a64e6fc6c43bad46dfbbf8bb38a8e75/Relative%20Pref(1).svg)

Die Abbildung zeigt den Mechanismus der präferenzbasierten Regularisierung im Netzwerk, der zur dynamischen Anpassung der Lernraten einzelner Modalitäten dient. Im Folgenden wird der Aufbau der Abbildung sowie deren Funktionalität anhand der zugrunde liegenden mathematischen Zusammenhänge erläutert.

---

### Dynamische Anpassung des modalspezifischen Gewichtungsfaktors

Für jede Modalität \(m\) wird ein modalspezifischer Gewichtungsfaktor \(E^m\) definiert. Dieser Faktor wird über die Trainingsepochen hinweg angepasst, um den Beitrag der jeweiligen Modalität zum Gesamttraining dynamisch zu regulieren. Die Aktualisierungsregel lautet:

\[
E^m_{r+1} = E^m_r - \lambda \, RP_m,
\]

wobei  
- \(\lambda\) ein kleiner Lernratenparameter ist, der die Schrittweite der Anpassung bestimmt,  
- \(RP_m\) die relative Präferenz der Modalität \(m\) repräsentiert.

**Interpretation:**  
- **Bei \(RP_m < 0\):**  
  Wird signalisiert, dass die Modalität im Vergleich zu den anderen schwächer repräsentiert ist. Da \(-\lambda \, RP_m\) in diesem Fall positiv wird, steigt \(E^m\). Dies führt dazu, dass der pixelbasierte Distillationsverlust \(L_{\text{pixel}}\) für diese Modalität stärker gewichtet wird, wodurch das Lernen dieser Modalität beschleunigt wird.  
- **Bei \(RP_m > 0\):**  
  Zeigt sich, dass die Modalität bereits gut repräsentiert ist. Hier erfolgt eine Reduzierung von \(E^m\), um zu verhindern, dass diese Modalität den Trainingsprozess dominiert.

---

### Verstärkung des prototypischen Verlusts

Parallel zur Anpassung des Faktors \(E^m\) wird der globale, prototypbasierte Verlust (auch Push L2-Proto Loss genannt) genutzt, um die Differenz zwischen der unimodalen (Schüler-) und der multimodalen (Lehrer-) Prototypenrepräsentation zu minimieren. Dieser Loss wird folgendermaßen definiert:

\[
L_{m}^{\text{proto}} = \left\| S_{m,s}^{n} - S_{t}^{n} \right\|_2^2,
\]

wobei  
- \(S_{m,s}^{n}\) die durch den unimodalen Zweig erzeugte Kosinus-Ähnlichkeit der Prototypen für Modalität \(m\) in Probe \(n\) ist,  
- \(S_{t}^{n}\) die entsprechende Ähnlichkeit des multimodalen Lehrerzweigs darstellt.

Bei einem negativen \(RP_m\) wird dieser Verlustanteil über einen Maskierungsfaktor zusätzlich gewichtet, sodass der Verlustanteil für schwächer repräsentierte Modalitäten verstärkt wird. Dadurch wird sichergestellt, dass die globalen, semantischen Repräsentationen der unimodalen Pfade enger an die multimodale Referenz herangeführt werden.

---

### Übergang zu Gewichtungen: Wahrscheinlichkeitsinterpretation

Der in der Abbildung dargestellte untere Pfeil symbolisiert die Transformation der berechneten relativen Präferenzwerte \(RP_m\) in Gewichtungen, die als Wahrscheinlichkeiten interpretiert werden können. Diese Gewichtungen modulieren den Einfluss der jeweiligen Modalität auf beide Verlustkomponenten – den pixelbasierten \(L_{\text{pixel}}\) und den prototypbasierten \(L_{\text{proto}}\). Durch diese Umwandlung wird sichergestellt, dass Modalitäten, die hinterherhinken (d. h. mit negativen \(RP_m\)-Werten), einen höheren Anteil am Trainingssignal erhalten.

---

### Funktionalität im Netzwerk

Die präferenzbasierte Regularisierung wird im Netzwerk wie folgt implementiert:

- **Dynamische Lernratenanpassung:**  
  Der Faktor \(E^m\) wird für jede Modalität individuell aktualisiert. Wird \(RP_m < 0\) festgestellt, wird \(E^m\) erhöht, wodurch der pixelbasierte Distillationsverlust \(L_{\text{pixel}}\) dieser Modalität stärker gewichtet wird. Dies führt zu einer schnelleren Anpassung und Verbesserung der schwächeren Modalitäten.

- **Integrierte Verlustfunktionen:**  
  Neben dem pixelbasierten Verlust wird der prototypbasierte Verlust \(L_{\text{proto}}\) zur Minimierung globaler, semantischer Differenzen eingesetzt. Beide Verlustkomponenten werden durch die dynamischen Gewichtungen skaliert, die aus den relativen Präferenzwerten abgeleitet werden.

- **Gewichtete Fusion:**  
  Die Umwandlung der relativen Präferenz \(RP_m\) in Wahrscheinlichkeitsgewichtungen sorgt dafür, dass der Beitrag jeder Modalität zum Gesamtverlust entsprechend ihrer aktuellen Repräsentation angepasst wird. Dadurch wird eine ausgewogene Integration der Modalitäten erreicht, auch wenn einige Modalitäten unvollständig oder schwächer vertreten sind.

---

### Zusammenfassung

Die präferenzbasierte Regularisierung in PRISM basiert auf zwei Hauptmechanismen:  
1. **Dynamische Anpassung des modalspezifischen Gewichtungsfaktors \(E^m\):**  
   Mittels der Update-Regel  
   \[
   E^m_{r+1} = E^m_r - \lambda \, RP_m,
   \]
   wird sichergestellt, dass Modalitäten, die im Vergleich zu anderen vernachlässigt werden (d. h. mit \(RP_m < 0\)), verstärkt berücksichtigt werden.  
2. **Verstärkung des prototypischen Verlusts:**  
   Der Push L2-Proto Loss  
   \[
   L_{m}^{\text{proto}} = \left\| S_{m,s}^{n} - S_{t}^{n} \right\|_2^2
   \]
   wird so gewichtet, dass die globale semantische Repräsentation der schwächeren Modalitäten näher an die multimodale Referenz herangeführt wird.

Die Überführung der relativen Präferenz in Gewichtungen, die als Wahrscheinlichkeiten interpretiert werden können, stellt sicher, dass der Einfluss jeder Modalität auf den Gesamtverlust dynamisch angepasst wird. Dieser Mechanismus ermöglicht eine ausgewogene und robuste Integration der Modalitäten, selbst wenn einzelne Datentypen unvollständig oder schwach vertreten sind.

Kurz zusammengefasst:  
**PRISM** integriert eine dynamische, präferenzbasierte Regularisierung, die durch die Anpassung des modalspezifischen Gewichtungsfaktors \(E^m\) und die Verstärkung des prototypischen Verlusts sicherstellt, dass alle Modalitäten – selbst bei unvollständigen Datensätzen – optimal in das Training einfließen. Dies führt zu einer stabileren und präziseren Segmentierung, ohne dass separate, vollmodale Datensätze benötigt werden.
