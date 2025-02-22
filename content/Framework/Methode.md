---
title: "PRISMS: Detaillierte Erklärung von PML, Cross-Modal Correspondence Distillation und Modality-Agnostic Distillation"
description: "In diesem Artikel stellen wir drei zentrale Bausteine unseres PRISMS-Frameworks vor: das Parallel Multimodal Learning (PML), die Cross-Modal Correspondence Distillation und die Modality-Agnostic Distillation. Diese Komponenten tragen entscheidend dazu bei, dass unser System robuste Segmentierungsergebnisse liefert – selbst wenn einzelne Modalitäten fehlen oder variieren."
math: true
---

## 1. Parallel Multimodal Learning (PML)

Beim PML-Ansatz werden alle verfügbaren Modalitäten gleichzeitig in einem einzigen Mini-Batch verarbeitet. Dadurch wird ein starker „Lehrer“ aufgebaut, der das gesamte Wissen aus den unterschiedlichen Sensoren aggregiert und als Referenz für die weiteren Distillationsprozesse dient.

### Wichtige Komponente: Supervised Loss

Für den Lehrer-Branch wird während des Trainings die übliche Kreuzentropie-Verlustfunktion eingesetzt:

\[
\mathcal{L}_{pre} = -\sum_{i=1}^{N} \sum_{k=1}^{K} y_{i,k} \log(p_{i,k}),
\]

wobei  
- \(N = h \times w\) die Gesamtzahl der Pixel (oder Voxelknoten) darstellt,  
- \(y_{i,k}\) den Ground-Truth-Wert für Klasse \(k\) am Pixel \(i\) bezeichnet,  
- \(p_{i,k}\) die vom Modell vorhergesagte Wahrscheinlichkeit für Klasse \(k\) am Pixel \(i\) ist.

Diese Loss-Funktion gewährleistet, dass präzise Vorhersagen gelernt werden, indem alle Modalitäten gleichermaßen in den Lernprozess einbezogen werden. Dadurch wird ein solides Fundament geschaffen, auf dem die weiteren Distillationsmechanismen aufbauen können.

---

## 2. Cross-Modal Correspondence Distillation

Ein häufig auftretendes Problem in multimodalen Systemen ist der **unimodale Bias**: Es wird tendenziell eine Modalität bevorzugt, die leichter zu verarbeiten ist (z. B. RGB-Bilder), während andere Modalitäten vernachlässigt werden. Um diesem Effekt entgegenzuwirken, wird im PRISMS-Framework ein Austausch von Wissen zwischen den verschiedenen Modalitäten vorgenommen.

### Die Grundidee

Die **Cross-Modal Correspondence Distillation** dient dazu, die Feature-Repräsentationen verschiedener Modalitäten – beispielsweise zwischen RGB und Tiefenbildern – aufeinander abzustimmen. Dies erfolgt durch den Vergleich der Kosinusähnlichkeit zwischen den Feature-Vektoren des Lehrer- und des Schülermodells.

### Verlustfunktion

Die Distillationsloss-Funktion wird wie folgt definiert:

\[
\mathcal{L}_{cmd} = \sum_{i=1}^4 \sum_{j=1}^{C_i} \mathcal{S}\left(g_d^{i,j}, g_r^{i,j}\right) \log\left(\frac{\mathcal{S}\left(g_d^{i,j}, g_r^{i,j}\right)}{\mathcal{S}\left(f_d^{i,j}, f_r^{i,j}\right)}\right),
\]

wobei  
- \(i\) die verschiedenen Skalenniveaus im Netzwerk bezeichnet (z. B. 1 bis 4),  
- \(j\) den Kanalindex des jeweiligen Feature-Map-Levels darstellt (mit \(C_i\) Kanälen),  
- \(g_r^{i,j}\) und \(g_d^{i,j}\) die Feature-Repräsentationen aus dem Schülermodell für die RGB- bzw. Tiefenmodalität sind,  
- \(f_r^{i,j}\) und \(f_d^{i,j}\) die entsprechenden Feature-Repräsentationen aus dem Lehrer-Modell darstellen,  
- \(\mathcal{S}(x,y)=\frac{x \cdot y}{\lVert x \rVert \lVert y \rVert}\) die Kosinusähnlichkeit zwischen den Vektoren \(x\) und \(y\) ist.

#### Detaillierte Erklärung

1. **Kosinusähnlichkeit als Übereinstimmungsmaß**:  
   Der Winkel zwischen zwei Feature-Vektoren wird verglichen. Werte nahe 1 deuten auf eine hohe Übereinstimmung hin, während Werte nahe 0 oder negativ auf geringe oder keine Übereinstimmung schließen lassen.

2. **Vergleich zwischen Lehrer und Schüler**:  
   Für jedes Feature-Level und jeden Kanal wird die Kosinusähnlichkeit der Schülermodalitäten mit der des Lehrers verglichen. Das logarithmische Verhältnis dieser Ähnlichkeiten wird als Verlust berechnet, sodass das Schülermodell lernt, dieselben intermodalen Beziehungen herzustellen wie das Lehrermodell.

3. **Zielsetzung**:  
   Damit wird sichergestellt, dass nicht nur auf einzelne Modalitäten fokussiert wird, sondern auch die Beziehungen und Abhängigkeiten zwischen den verschiedenen Modalitäten erlernt werden – was die Robustheit gegenüber dem Ausfall einzelner Modalitäten erhöht.

---

## 3. Modality-Agnostic Distillation

Neben der Abstimmung der Feature-Repräsentationen zwischen Modalitäten zielt die **Modality-Agnostic Distillation** darauf ab, semantisches Wissen auf der Vorhersageebene zu übertragen.

### Grundidee

Es soll gewährleistet werden, dass die semantischen Vorhersagen des Schülermodells (\(P_{am}\)) möglichst nahe an den Vorhersagen des multimodalen Lehrermodells (\(P_{mm}\)) liegen – unabhängig davon, welche Modalitäten verfügbar sind.

### Verlustfunktion

Der entsprechende Verlust wird definiert als:

\[
\mathcal{L}_{mad} = \frac{1}{N} \sum_{i=1}^{N} \sum_{k=1}^{K} P_{am}^{i,k} \log\left(\frac{P_{am}^{i,k}}{P_{mm}^{i,k}}\right),
\]

wobei  
- \(N\) die Gesamtzahl der Pixel (oder Voxelknoten) ist,  
- \(K\) die Anzahl der Klassen darstellt,  
- \(P_{am}^{i,k}\) die vom Schülermodell vorhergesagte Wahrscheinlichkeit für Klasse \(k\) am Pixel \(i\) angibt,  
- \(P_{mm}^{i,k}\) die entsprechende Vorhersage des Lehrermodells darstellt.

#### Detaillierte Erklärung

1. **Vergleich auf der Vorhersageebene**:  
   Während frühere Distillationsansätze auf den Feature-Ebenen ansetzen, wird hier der Fokus auf die Endergebnisse gelegt – die Segmentierungskarten. Dies gewährleistet, dass das Schülermodell nicht nur ähnliche interne Repräsentationen erlernt, sondern auch konsistente und semantisch sinnvolle Vorhersagen trifft.

2. **Kullback-Leibler-Divergenz**:  
   Die Divergenz zwischen den Wahrscheinlichkeitsverteilungen \(P_{am}\) und \(P_{mm}\) wird über alle Pixel und Klassen gemessen. Ein niedriger Wert zeigt an, dass sich die Verteilungen des Schülermodells eng an die des Lehrermodells annähern.

3. **Robustheit bei unvollständigen Modalitäten**:  
   Da das Lehrermodell auf vollständigen, multimodalen Daten trainiert wurde, kann das Schülermodell durch diese Distillation konsistente Vorhersagen liefern, auch wenn nur ein Teil der Modalitäten vorhanden ist.

---

## Zusammenfassung

Im PRISMS-Framework wird durch den Einsatz von **PML**, **Cross-Modal Correspondence Distillation** und **Modality-Agnostic Distillation** sichergestellt, dass robuste Segmentierungsergebnisse erzielt werden – auch bei unvollständigen oder variierenden Modalitäten.  
- **PML** stellt die Basis dar, indem alle verfügbaren Modalitäten parallel verarbeitet werden und so ein starker Lehrer aufgebaut wird.  
- **Cross-Modal Distillation** sorgt dafür, dass intermodale Beziehungen erlernt und ein unimodaler Bias vermieden wird.  
- **Modality-Agnostic Distillation** überträgt semantisches Wissen auf der Vorhersageebene und gewährleistet damit Konsistenz, selbst wenn nicht alle Modalitäten vorliegen.

Diese dreifache Strategie ermöglicht es, in realen Anwendungsszenarien – sei es in der medizinischen Bildgebung, im autonomen Fahren oder in industriellen Anwendungen – zuverlässige Segmentierungsergebnisse zu erzielen, auch wenn die Datenquellen unvollständig oder heterogen sind.