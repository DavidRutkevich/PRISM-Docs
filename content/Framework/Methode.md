---

title: "PRISMS: Detaillierte Erklärung von PML, Cross-Modal Correspondence Distillation und Modality-Agnostic Distillation"  
description: "In diesem Artikel werden drei zentrale Bausteine des PRISMS-Frameworks vorgestellt: das Parallel Multimodal Learning (PML), die Cross-Modal Correspondence Distillation und die Modality-Agnostic Distillation. Diese Komponenten tragen entscheidend dazu bei, dass das System robuste Segmentierungsergebnisse liefert – selbst wenn einzelne Modalitäten fehlen oder variieren."  
math: true  

---

## 1. Parallel Multimodal Learning (PML)

Beim PML-Ansatz werden alle verfügbaren Modalitäten gleichzeitig in einem einzigen Mini-Batch verarbeitet. Auf diese Weise wird ein starker „Lehrer“ aufgebaut, der das gesamte Wissen aus den unterschiedlichen Sensoren aggregiert und als Referenz für die folgenden Distillationsprozesse dient.

### Wichtige Komponente: Supervised Loss

Für den Lehrer-Zweig wird während des Trainings die klassische Kreuzentropie-Verlustfunktion verwendet:

\[
\mathcal{L}_{pre} = -\sum_{i=1}^{N} \sum_{k=1}^{K} y_{i,k} \log(p_{i,k}),
\]

wobei  
- \(N = h \times w\) die Gesamtzahl der Pixel (oder Voxelknoten) repräsentiert,  
- \(y_{i,k}\) den Ground-Truth-Wert für Klasse \(k\) am Pixel \(i\) angibt,  
- \(p_{i,k}\) die vom Modell vorhergesagte Wahrscheinlichkeit für Klasse \(k\) am Pixel \(i\) darstellt.

Diese Verlustfunktion sorgt dafür, dass präzise Vorhersagen erlernt werden, indem alle Modalitäten gleichwertig in den Trainingsprozess einbezogen werden. Dadurch wird ein solides Fundament geschaffen, auf dem die weiteren Distillationsmechanismen aufbauen können.

---

## 2. Cross-Modal Correspondence Distillation

Ein häufig auftretendes Problem in multimodalen Systemen ist der **unimodale Bias**: Bestimmte Modalitäten, die leichter zu verarbeiten sind (z. B. RGB-Bilder), werden bevorzugt, während andere vernachlässigt werden. Zur Reduzierung dieses Effekts wird im PRISMS-Framework ein Austausch von Wissen zwischen den verschiedenen Modalitäten vorgenommen.

### Die Grundidee

Die **Cross-Modal Correspondence Distillation** hat zum Ziel, die Feature-Repräsentationen verschiedener Modalitäten – beispielsweise zwischen RGB und Tiefenbildern – aufeinander abzustimmen. Dies geschieht durch den Vergleich der Kosinusähnlichkeiten zwischen den Feature-Vektoren des Lehrer- und des Schülermodells.

### Verlustfunktion

Die Distillationsverlustfunktion wird folgendermaßen formuliert:

\[
\mathcal{L}_{cmd} = \sum_{i=1}^4 \sum_{j=1}^{C_i} \mathcal{S}\left(g_d^{i,j}, g_r^{i,j}\right) \log\left(\frac{\mathcal{S}\left(g_d^{i,j}, g_r^{i,j}\right)}{\mathcal{S}\left(f_d^{i,j}, f_r^{i,j}\right)}\right),
\]

wobei  
- \(i\) die verschiedenen Skalenniveaus im Netzwerk bezeichnet (z. B. 1 bis 4),  
- \(j\) den Kanalindex des jeweiligen Feature-Map-Levels (mit \(C_i\) Kanälen) darstellt,  
- \(g_r^{i,j}\) und \(g_d^{i,j}\) die Feature-Repräsentationen aus dem Schülermodell für die RGB- bzw. Tiefenmodalität repräsentieren,  
- \(f_r^{i,j}\) und \(f_d^{i,j}\) die entsprechenden Feature-Repräsentationen aus dem Lehrer-Modell darstellen,  
- \(\mathcal{S}(x,y)=\frac{x \cdot y}{\lVert x \rVert \lVert y \rVert}\) die Kosinusähnlichkeit zwischen den Vektoren \(x\) und \(y\) berechnet.

#### Detaillierte Erläuterung

1. **Kosinusähnlichkeit als Maß der Übereinstimmung:**  
   Der Winkel zwischen zwei Feature-Vektoren wird gemessen. Werte nahe 1 deuten auf eine hohe Übereinstimmung hin, während niedrigere Werte auf eine geringere Übereinstimmung schließen lassen.
2. **Vergleich zwischen Lehrer und Schüler:**  
   Für jedes Feature-Level und jeden Kanal wird die Kosinusähnlichkeit der Schülermodalitäten mit derjenigen des Lehrers verglichen. Das logarithmische Verhältnis dieser Ähnlichkeiten wird als Verlust genutzt, sodass das Schülermodell lernt, dieselben intermodalen Beziehungen zu reproduzieren.
3. **Zielsetzung:**  
   Durch diesen Vergleich wird erreicht, dass nicht nur die einzelnen Modalitäten, sondern auch die Beziehungen und Abhängigkeiten zwischen den verschiedenen Modalitäten erlernt werden – was die Robustheit gegenüber dem Ausfall einzelner Modalitäten verbessert.

---

## 3. Modality-Agnostic Distillation

Die **Modality-Agnostic Distillation** zielt darauf ab, semantisches Wissen auf der Vorhersageebene zu übertragen, sodass die Vorhersagen des Schülermodells (\(P_{am}\)) möglichst nahe an den Vorhersagen des multimodalen Lehrermodells (\(P_{mm}\)) liegen – unabhängig von der Modalitätsverfügbarkeit.

### Grundidee

Es wird sichergestellt, dass das Schülermodell semantisch konsistente Vorhersagen liefert, selbst wenn nur ein Teil der Modalitäten zur Verfügung steht. Dies erfolgt durch den Vergleich der Endvorhersagen beider Modelle.

### Verlustfunktion

Der entsprechende Verlust wird definiert als:

\[
\mathcal{L}_{mad} = \frac{1}{N} \sum_{i=1}^{N} \sum_{k=1}^{K} P_{am}^{i,k} \log\left(\frac{P_{am}^{i,k}}{P_{mm}^{i,k}}\right),
\]

wobei  
- \(N\) die Gesamtzahl der Pixel (oder Voxelknoten) repräsentiert,  
- \(K\) die Anzahl der Klassen bezeichnet,  
- \(P_{am}^{i,k}\) die vom Schülermodell vorhergesagte Wahrscheinlichkeit für Klasse \(k\) am Pixel \(i\) angibt,  
- \(P_{mm}^{i,k}\) die entsprechende Vorhersage des Lehrermodells darstellt.

#### Detaillierte Erläuterung

1. **Vergleich auf der Vorhersageebene:**  
   Hier wird nicht nur auf der Ebene der internen Features, sondern auf der Ebene der finalen Segmentierungskarten verglichen.  
2. **Kullback-Leibler-Divergenz:**  
   Die Divergenz zwischen den Wahrscheinlichkeitsverteilungen \(P_{am}\) und \(P_{mm}\) wird über alle Pixel und Klassen berechnet. Ein niedriger Wert signalisiert eine enge Annäherung der Schülervorhersagen an die Lehrervorhersagen.  
3. **Robustheit bei unvollständigen Modalitäten:**  
   Da das Lehrermodell auf vollständigen multimodalen Daten trainiert wurde, wird das Schülermodell durch diesen Prozess in die Lage versetzt, auch bei eingeschränkter Modalitätsverfügbarkeit konsistente Vorhersagen zu treffen.

---

## Zusammenfassung

Im PRISMS-Framework wird durch den Einsatz von **Parallel Multimodal Learning (PML)**, **Cross-Modal Correspondence Distillation** und **Modality-Agnostic Distillation** sichergestellt, dass robuste Segmentierungsergebnisse erzielt werden – selbst wenn einzelne Modalitäten fehlen oder in ihrer Qualität variieren.  
- **PML** bildet die Basis, indem alle verfügbaren Modalitäten parallel verarbeitet werden, um einen starken Lehrer aufzubauen.  
- **Cross-Modal Distillation** gleicht die intermodalen Beziehungen an und reduziert den unimodalen Bias.  
- **Modality-Agnostic Distillation** überträgt semantisches Wissen auf der Vorhersageebene, sodass das Schülermodell konsistente Segmentierungskarten erzeugt.

Diese dreifache Strategie ermöglicht es, in realen Anwendungsszenarien – sei es in der medizinischen Bildgebung, im autonomen Fahren oder in industriellen Anwendungen – zuverlässige Segmentierungsergebnisse zu erzielen, selbst wenn die Datenquellen unvollständig oder heterogen sind.