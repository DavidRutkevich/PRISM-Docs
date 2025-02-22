---

title: "PRISM – Relative Präferenz"  
description: "Wie das mehrstufige Modell aufgebaut ist und wie die einzelnen Teile ineinandergreifen."  
date: 2025-02-05  
math: true  

---

![Relative Präferenz (Figur 1)](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/8d1a65675a64e6fc6c43bad46dfbbf8bb38a8e75/Relative%20Pref(1).svg)

Die Abbildung zeigt den Mechanismus der präferenzbasierten Regularisierung in meinem Netzwerk, der zur dynamischen Anpassung der Lernraten einzelner Modalitäten dient. Im Folgenden erläutere ich den Aufbau der Figur und deren Funktionalität im Netzwerk anhand der mathematischen Beziehungen.

---

### Dynamische Anpassung des modalspezifischen Gewichtungsfaktors

Für jede Modalität \(m\) führe ich einen modalspezifischen Gewichtungsfaktor \(E^m\). Dieser Faktor wird über die Trainingsepochen aktualisiert, um den Beitrag der jeweiligen Modalität anzupassen. Die Update-Regel lautet:

\[
E^m_{r+1} = E^m_r - \lambda \, RP_m,
\]

wobei  
- \(\lambda\) ein kleiner Lernratenparameter ist,  
- \(RP_m\) die relative Präferenz der Modalität \(m\) darstellt.

**Interpretation:**  
- **Wenn \(RP_m < 0\):**  
  Dies signalisiere, dass die Modalität schwach repräsentiert ist. Da \(-\lambda \, RP_m\) dann positiv wird, steigt \(E^m\), wodurch der Einfluss des pixelbasierten Distillationsverlusts \(L_{\text{pixel}}\) verstärkt wird. Kurz: Das Lernen dieser Modalität wird beschleunigt.
- **Wenn \(RP_m > 0\):**  
  Die Modalität ist bereits gut repräsentiert, und \(E^m\) wird reduziert, um zu verhindern, dass diese Modalität den Gesamttrainingsprozess dominiert.

---

### Verstärkung des prototypischen Verlusts

Parallel zur Anpassung des Faktors \(E^m\) verstärke ich den globalen, prototypischen Verlust. Dieser Verlust, auch als Push L2-Proto Loss bezeichnet, wird verwendet, um die Differenz zwischen der unimodalen (Schüler-) und der multimodalen (Lehrer-) Prototypenrepräsentation zu minimieren:

\[
L_{m}^{\text{proto}} = \left\| S_{m,s}^{n} - S_{t}^{n} \right\|_2^2,
\]

wobei  
- \(S_{m,s}^{n}\) die Kosinus-Ähnlichkeit der Prototypen des unimodalen Modells für Modalität \(m\) in Probe \(n\) ist,  
- \(S_{t}^{n}\) die entsprechende Ähnlichkeit des multimodalen Modells darstellt.

Wird \(RP_m < 0\) festgestellt, wird dieser Verlustanteil stärker gewichtet, sodass ich die schwache Modalität gezielt „pushen“ kann, um ihre globale Repräsentation an die multimodale Referenz anzugleichen.

---

### Übergang zu Gewichtungen: Wahrscheinlichkeitsinterpretation

Der untere Pfeil in der Abbildung symbolisiert, dass ich die berechneten relativen Präferenzwerte \(RP_m\) in Gewichtungen überführe, die als Wahrscheinlichkeiten interpretiert werden können. Diese Gewichtungen modulieren den Einfluss der jeweiligen Modalität auf den Gesamtverlust – sei es beim pixelbasierten \(L_{\text{pixel}}\) oder beim prototypischen \(L_{\text{proto}}\):

\[
\text{Gewichtung} \sim f(RP_m).
\]

Dies stellt sicher, dass Modalitäten, die hinterherhinken (also negative \(RP_m\) aufweisen), einen größeren Anteil am Trainingssignal erhalten.

---

### Funktionalität im Netzwerk

- **Dynamische Lernratenanpassung:**  
  Ich passe den Faktor \(E^m\) für jede Modalität individuell an. Bei \(RP_m < 0\) erhöhe ich \(E^m\), wodurch der pixelbasierte Verlust \(L_{\text{pixel}}\) für diese Modalität stärker gewichtet wird. Dies ermöglicht ein schnelleres Anlernen schwacher Modalitäten.

- **Integrierte Verlustfunktionen:**  
  Neben \(L_{\text{pixel}}\) nutze ich den prototypischen Verlust \(L_{\text{proto}}\), um globale, semantische Unterschiede zwischen unimodalen und multimodalen Repräsentationen zu minimieren. Beide Verlustkomponenten werden über die Gewichtungen aus den relativen Präferenzen dynamisch skaliert.

- **Gewichtete Fusion:**  
  Die Umwandlung von \(RP_m\) in Gewichtungen sorgt dafür, dass der Beitrag jeder Modalität zum Gesamtverlust abhängig von ihrer Repräsentation dynamisch angepasst wird. Dies führt zu einer ausgewogenen Integration der Modalitäten, selbst wenn einige unvollständig oder schwächer vertreten sind.

---

Diese Mechanismen arbeiten zusammen, um sicherzustellen, dass mein Netzwerk auch bei unvollständigen und unbalancierten Datensätzen stabile und robuste Segmentierungsergebnisse erzielt. Der dynamische Anpassungsprozess über \(E^m\), gekoppelt mit der verstärkten Gewichtung des prototypischen Verlusts und der Überführung in Wahrscheinlichkeitsgewichtungen, bildet das Herzstück der präferenzbasierten Regularisierung in PRISM.