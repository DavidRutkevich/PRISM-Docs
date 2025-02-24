Nachfolgend wird der Artikel stilistisch überarbeitet, wobei der inhaltliche Kern beibehalten wird:

---

title: "Architektur und Aufbau des PRISMS-Frameworks"  
description: "Detaillierte Erläuterung der Gesamtstruktur (PML, Unimodal- & Cross-Modal Distillation) anhand zentraler Abbildungen"  
math: true  

---

## 1. Gesamtstruktur: Lehrer–Schüler-Aufbau und Distillation

![PRISMS Framework](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/Overall_light.png "Abbildung 1 zeigt das (a) Overall Framework in einer zweistufigen Struktur, die aus einem multimodalen Segmentor (oben) und einem anymodalen Segmentor (unten) besteht.")

### 1.1 Multimodaler Lehrer

- **Multimodale Eingabe:**  
  Im oberen Teil werden sämtliche verfügbare Modalitäten – beispielsweise RGB, Tiefe, Event und LiDAR – in einem einzigen Batch verarbeitet.  
- **Mehrskalige multimodale Eigenschaften:**  
  Der Encoder (z. B. ein Transformer-Backbone) extrahiert mehrskalige Repräsentationen aus jeder Modalität. Anschließend werden diese Repräsentationen, abhängig vom gewählten Fusionsprinzip, zu einem "starken" Feature-Vektor zusammengeführt.  
- **Segmentation Head:**  
  Auf Basis der fusionierten Repräsentationen wird eine Segmentierungskarte erzeugt, die als **Lehrer-Vorhersage** dient. Parallel dazu wird ein klassischer Cross-Entropy-Loss (\(\mathcal{L}_{pre}\)) zur Supervision eingesetzt.

Der multimodale Lehrer dient als Referenzmodell, das sämtliche Modalitäten berücksichtigt und somit eine bestmögliche Vorhersagequalität liefert. Aus diesem Modell werden Feature- und Vorhersageinformationen extrahiert, um sie an das Schüler-Modell weiterzugeben.

### 1.2 Anymodaler Schüler

- **Teilweise oder unvollständige Modalitäten:**  
  Im unteren Teil wird dargestellt, dass die Eingabe **randomisiert** unvollständig sein kann – mal liegen nur einzelne Modalitäten vor (z. B. nur RGB), mal zwei (RGB + Tiefe) oder alle vier.  
- **Mehrskalige anymodale Eigenschaften:**  
  Ein ähnlicher Encoder-Block extrahiert die Features ausschließlich aus den tatsächlich vorhandenen Modalitäten. Ziel ist es, trotz reduzierter Eingabequalität robuste Segmentierungsergebnisse zu erzielen.  
- **Segmentation Head:**  
  Auf Basis der gewonnenen Teil-Features werden die endgültigen Vorhersagen berechnet. Unterschiedliche Distillationsmechanismen stellen sicher, dass das Schüler-Modell die "Denkmuster" des Lehrers übernimmt – auch wenn weniger oder andere Daten zur Verfügung stehen.

### 1.3 Unimodale und Cross-Modal Distillation

Rechts in Abbildung 1 werden zwei wichtige Prozesse auf der Feature-Ebene dargestellt:

1. **Unimodale Distillation:**  
   Für jede Modalität wird der Schüler-Zweig (z. B. \(g_r^{i}\) für RGB) direkt mit den entsprechenden Lehrer-Features (\(f_r^{i}\)) verglichen. Ziel ist es, dass der Schüler in derselben Modalität Repräsentationen erlernt, die denen des Lehrers ähneln. Zur Realisierung werden Loss-Funktionen wie die Kullback-Leibler-Divergenz (\(\mathcal{L}_{umd}\)) eingesetzt.
2. **Cross-Modal Correspondence Distillation:**  
   Zusätzlich werden die Beziehungen zwischen zwei Modalitäten im Schüler mit denen des Lehrers verglichen. Beispielsweise wird die Kosinusähnlichkeit zwischen den RGB- und Tiefen-Features im Schüler (\(\mathcal{S}(g_r^i, g_d^i)\)) mit der des Lehrers (\(\mathcal{S}(f_r^i, f_d^i)\)) kontrastiert. Dies verhindert, dass der Schüler zu stark auf eine einzelne Modalität fokussiert und fördert das Erlernen der komplementären Beziehungen zwischen Modalitäten. Mathematisch wird dieser Prozess über \(\mathcal{L}_{cmd}\) realisiert.

### 1.4 Modality-Agnostic Distillation

Ergänzend wird eine **Vorhersage-Distillation** durchgeführt, bei der die finale Segmentierungskarte des Schülers (\(P_{am}\)) mit der des Lehrers (\(P_{mm}\)) verglichen wird. Der entsprechende Verlust \(\mathcal{L}_{mad}\) misst die Diskrepanz zwischen beiden Wahrscheinlichkeitsverteilungen:

\[
\mathcal{L}_{mad} = \frac{1}{N} \sum_{i=1}^{N} \sum_{k=1}^{K} P_{am}^{i,k} \log\left(\frac{P_{am}^{i,k}}{P_{mm}^{i,k}}\right).
\]

Durch diesen Vergleich passt der Schüler seine Vorhersagen an, sodass er auch in komplexen, multimodalen Situationen dem Lehrer ähnelt – selbst wenn nicht alle Modalitäten vorliegen.

---

## 2. PML (Parallel Multimodal Learning) – Ein starker Lehrer

![PML](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/PML%2Bfeature%20distil.png)

Die zweite Abbildung veranschaulicht den **PML-Prozess**, der den zentralen Baustein für den Aufbau des Lehrer-Netzwerks bildet. Alle Modalitäten werden parallel in einem einzigen Mini-Batch verarbeitet.

### 2.1 Gemeinsame Verarbeitung pro Transformer-Block

- **Eingabe:**  
  Pro Batch werden alle Modalitäten einer Szene als Eingabe verwendet.  
- **Parallele Feature-Extraktion:**  
  In jedem Transformer-Block werden zunächst die Feature-Maps aller Modalitäten unabhängig voneinander berechnet. Anschließend erfolgt eine Fusion (z. B. über Mittelung), um einen gemeinsamen multimodalen Feature-Vektor zu erzeugen.  
- **Skalenniveaus:**  
  Typischerweise wird der Prozess in mehreren Stufen (Level 1 bis 4) wiederholt, wobei die Modalitäten schrittweise zusammengeführt werden.

### 2.2 Ziel: Maximaler Informationsgehalt

Das PML führt zu einem robusten **multimodalen Encoder**, der sämtliche Signalquellen vereint. Das finale Feature wird an den Decoder (Segmentation Head) weitergegeben, wodurch ein umfassendes Wissen aus allen Modalitäten (RGB, Tiefe, Event, LiDAR usw.) integriert wird. Das Segmentierungsergebnis des Lehrers wird mithilfe eines klassischen Kreuzentropie-Verlusts (\(\mathcal{L}_{pre}\)) optimiert:

\[
\mathcal{L}_{pre} = -\sum_{i=1}^{N} \sum_{k=1}^{K} y_{i,k} \log(p_{i,k}).
\]

Dieser Ansatz stellt sicher, dass der Lehrer ein möglichst vollständiges Verständnis der Szene erlangt und als effektive Distillationsquelle für den anymodalen Schüler dient.

---

## 3. Zusammenspiel der Komponenten

1. **Lehrertraining mit PML:**  
   Zunächst wird der **Lehrer** mithilfe aller Modalitäten im Parallel-Modus trainiert, sodass ein robustes und umfassendes Modell entsteht.
2. **Anymodales Schülertraining:**  
   Anschließend wird der **Schüler** (anymodaler Segmentor) unter Zuhilfenahme folgender Mechanismen trainiert:  
   - **Unimodale Distillation** (\(\mathcal{L}_{umd}\)) für jede Modalität,  
   - **Cross-Modal Distillation** (\(\mathcal{L}_{cmd}\)) zur Angleichung der Beziehungen zwischen den Modalitäten,  
   - **Modality-Agnostic Distillation** (\(\mathcal{L}_{mad}\)) auf der Vorhersageebene.
3. **Ziel: Robustheit bei fehlenden Modalitäten:**  
   Während des Schülertrainings werden die Modalitäten zufällig maskiert oder weggelassen, sodass der Schüler lernt, auch mit unvollständigen Eingaben eine möglichst vollständige Vorhersage zu generieren. Dank des Lehrers kann der Schüler dem "vollen" Kontext näherkommen.

---

## 4. Zusammenspiel der Unimodalen und Cross-Modalen Distillationsverfahren

1. **Unimodale Distillation:**  
   Durch den Vergleich der unimodalen Features (\(g_r^{i}\) etc.) mit den entsprechenden Lehrer-Features (\(f_r^{i}\)) wird sichergestellt, dass der Schüler in jeder Modalität ähnliche Repräsentationen wie der Lehrer erlernt. Dies wird beispielsweise über die Kullback-Leibler-Divergenz realisiert.
2. **Cross-Modal Distillation:**  
   Zusätzlich wird die Beziehung zwischen verschiedenen Modalitäten im Schüler überprüft, indem etwa die Kosinusähnlichkeit zwischen den RGB- und Tiefen-Features des Schülers mit der des Lehrers verglichen wird. Dies verhindert einen starken unimodalen Bias und fördert die Integration komplementärer Informationen.
3. **Modality-Agnostic Distillation:**  
   Schließlich wird die finale Segmentierung des Schülers mit der des Lehrers verglichen, um sicherzustellen, dass auch in der Vorhersageebene ein kohärenter Gesamtklang erzielt wird.

---

## 5. Integration in bestehende Architekturen

PRISM ist als **Plug-and-Play-Modul** konzipiert und kann problemlos in verschiedene Segmentierungsarchitekturen integriert werden, beispielsweise in U-Net, mmFormer oder RFNet.

### 5.1 Orchestrierung

- **Encoderausgänge pro Modalität:**  
  Jede Modalität wird separat enkodiert, sodass die individuellen Informationen erhalten bleiben.  
- **Fusionspfad als Dirigent:**  
  Der Fusionspfad führt die separaten Modalitäten zusammen und bildet den Gesamtklang.  
- **PRISM-Module:**  
  Die selbstdistillierenden Mechanismen (sowohl unimodal als auch cross-modal) sowie die präferenzbasierte Regulierung werden an den Schnittstellen zwischen den Encodern und dem Fusionspfad implementiert.

### 5.2 Praktische Details

- **Datenmaskierung:**  
  Fehlende Modalitäten werden mithilfe der Indikationsmatrix \(C_{nm}\) maskiert, sodass sie während des Trainings nicht berücksichtigt werden.  
- **Hyperparameter:**  
  Neben Standardparametern werden die Temperatur \(\mu\), die Gewichtungen \(\gamma_1\) und \(\gamma_2\) sowie der Lernratenparameter \(\lambda\) festgelegt, um den Einfluss der einzelnen Komponenten optimal zu steuern.

---

## 6. Fazit und Ausblick

### Zusammenfassung

Das PRISMS-Framework bietet einen innovativen Ansatz zur Integration unvollständiger multimodaler Daten in der medizinischen Bildverarbeitung. Durch die Kombination einer internen Teacher–Student-Selbstdistillation mit einer präferenzbasierten Regularisierung wird sichergestellt, dass auch Modalitäten, die seltener vorhanden oder von geringerer Qualität sind, adäquat berücksichtigt werden. Dies führt zu einer stabilen und präzisen Segmentierung, vergleichbar mit einem harmonisch abgestimmten Orchester – selbst wenn einzelne Instrumente fehlen oder verstimmt sind.

### Ausblick

Mögliche Erweiterungen umfassen:
- **Erweiterte Fusionstechniken:**  
  Der Einsatz komplexerer Mechanismen wie Cross-Modal Attention könnte die Integration der Modalitäten weiter verbessern.
- **Selbstüberwachte Lernverfahren:**  
  Die Kombination von PRISM mit selbstüberwachten oder halbüberwachten Ansätzen könnte den Bedarf an umfangreichen, vollständig annotierten Datensätzen weiter reduzieren.
- **Anwendung in weiteren Domänen:**  
  Über die medizinische Bildverarbeitung hinaus kann PRISMS in anderen Bereichen (z. B. Sensorfusion im IoT, Videoanalyse) eingesetzt werden, um die Integration heterogener Datenquellen zu optimieren.