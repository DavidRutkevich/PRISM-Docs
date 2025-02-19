---
title: "PRISMS: Detaillierte Erklärung von PML, Cross-Modal Correspondence Distillation und Modality-Agnostic Distillation"
description: "In diesem Artikel stellen wir drei zentrale Bausteine unseres PRISMS-Frameworks vor: das Parallel Multimodal Learning (PML), die Cross-Modal Correspondence Distillation und die Modality-Agnostic Distillation. Diese Komponenten tragen entscheidend dazu bei, dass unser System robuste Segmentierungsergebnisse liefert – selbst wenn einzelne Modalitäten fehlen oder variieren."
math: true
---
In diesem Artikel stellen wir drei zentrale Bausteine unseres PRISMS-Frameworks vor: das **Parallel Multimodal Learning (PML)**, die **Cross-Modal Correspondence Distillation** und die **Modality-Agnostic Distillation**. Diese Komponenten tragen entscheidend dazu bei, dass unser System robuste Segmentierungsergebnisse liefert – selbst wenn einzelne Modalitäten fehlen oder variieren.

---

## 1. Parallel Multimodal Learning (PML)

Beim PML-Ansatz werden alle verfügbaren Modalitäten gleichzeitig in einem einzigen Mini-Batch verarbeitet. Dadurch entsteht ein starker "Lehrer", der das gesamte Wissen aus den verschiedenen Sensoren aggregiert und als Referenz für die weiteren Distillationsprozesse dient.

### Wichtige Komponente: Supervised Loss

Während des Trainings wird für den Lehrer-Branch die typische Kreuzentropie-Verlustfunktion verwendet. Diese lautet:

\[
\mathcal{L}_{pre} = -\sum_{i=1}^{N} \sum_{k=1}^{K} y_{i,k} \log(p_{i,k}),
\]

wobei:
- \( N = h \times w \) die Gesamtzahl der Pixel (oder Voxelknoten) darstellt,
- \( y_{i,k} \) den Ground-Truth-Wert für Klasse \( k \) am Pixel \( i \) bezeichnet,
- \( p_{i,k} \) die vom Modell vorhergesagte Wahrscheinlichkeit für Klasse \( k \) am Pixel \( i \) ist.

Diese Loss-Funktion sorgt dafür, dass das Modell lernt, präzise Vorhersagen zu treffen, indem alle Modalitäten gleichermaßen in den Lernprozess einbezogen werden. Dadurch wird ein solides Fundament geschaffen, auf dem die weiteren Distillationsmechanismen aufbauen können.

---

## 2. Cross-Modal Correspondence Distillation

Ein häufiges Problem in multimodalen Systemen ist der sogenannte **unimodale Bias**: Das Modell tendiert dazu, Modalitäten zu bevorzugen, die leichter zu verarbeiten sind (z. B. RGB-Bilder), während andere Modalitäten vernachlässigt werden. Um diesem Problem entgegenzuwirken, wird im PRISMS-Framework Wissen zwischen verschiedenen Modalitäten ausgetauscht.

### Die Idee dahinter

Durch die **Cross-Modal Correspondence Distillation** sollen die Repräsentationen (Feature Maps) verschiedener Modalitäten – etwa zwischen RGB und Tiefenbildern – aufeinander abgestimmt werden. Dies erfolgt über den Vergleich der Kosinusähnlichkeit zwischen den Feature-Vektoren des Lehrer- und des Schülermodells.

### Die zugrunde liegende Gleichung

Die Distillationsloss-Funktion wird wie folgt definiert:

\[
\mathcal{L}_{cmd} = \sum_{i=1}^4 \sum_{j=1}^{C_i} \mathcal{S}\left(g_d^{i,j}, g_r^{i,j}\right) \log\left(\frac{\mathcal{S}\left(g_d^{i,j}, g_r^{i,j}\right)}{\mathcal{S}\left(f_d^{i,j}, f_r^{i,j}\right)}\right),
\]

wobei:
- \( i \) die verschiedenen Skalenniveaus im Netzwerk darstellt (z. B. von 1 bis 4),
- \( j \) den Index über die Kanäle im jeweiligen Feature-Map-Level bezeichnet (mit \( C_i \) Kanälen),
- \( g_r^{i,j} \) und \( g_d^{i,j} \) die Feature-Repräsentationen aus dem Schülermodell für die RGB- bzw. Tiefenmodalität sind,
- \( f_r^{i,j} \) und \( f_d^{i,j} \) die entsprechenden Feature-Repräsentationen aus dem Lehrer-Modell sind,
- \(\mathcal{S}(x,y)=\frac{x \cdot y}{\lVert x \rVert \lVert y \rVert}\) die Kosinusähnlichkeit zwischen den Vektoren \( x \) und \( y \) darstellt.

### Detaillierte Erklärung

1. **Kosinusähnlichkeit als Maß für Übereinstimmung**:  
   Die Kosinusähnlichkeit vergleicht den Winkel zwischen zwei Feature-Vektoren. Werte nahe 1 deuten auf eine hohe Übereinstimmung hin, während Werte nahe 0 oder negativ auf geringe oder keine Übereinstimmung schließen lassen.

2. **Vergleich zwischen Lehrer und Schüler**:  
   Für jedes Feature-Level und jeden Kanal wird die Kosinusähnlichkeit der Schülermodalitäten (\( g_d^{i,j}, g_r^{i,j} \)) mit der des Lehrers (\( f_d^{i,j}, f_r^{i,j} \)) verglichen. Das logarithmische Verhältnis dieser Ähnlichkeiten wird als Verlust berechnet, um sicherzustellen, dass der Schüler lernt, die gleichen intermodalen Beziehungen herzustellen wie der Lehrer.

3. **Ziel**:  
   Dadurch wird sichergestellt, dass das Modell nicht nur auf einzelne Modalitäten fokussiert, sondern auch die Beziehungen und Abhängigkeiten zwischen den verschiedenen Modalitäten erlernt – was die Robustheit gegenüber dem Ausfall einzelner Modalitäten erhöht.

---

## 3. Modality-Agnostic Distillation

Während die Cross-Modal Distillation dafür sorgt, dass die Feature-Repräsentationen zwischen den Modalitäten harmonisieren, geht die **Modality-Agnostic Distillation** einen Schritt weiter: Sie fokussiert sich auf die Übertragung von semantischem Wissen auf der Vorhersageebene.

### Die Grundidee

Die Modality-Agnostic Distillation stellt sicher, dass die semantischen Vorhersagen des Schülermodells (\( P_{am} \)) möglichst nahe an den Vorhersagen des multimodalen Lehrermodells (\( P_{mm} \)) liegen – unabhängig davon, welche Modalitäten gerade verfügbar sind.

### Die entsprechende Gleichung

Der zugehörige Verlust wird definiert als:

\[
\mathcal{L}_{mad} = \frac{1}{N} \sum_{i=1}^{N} \sum_{k=1}^{K} P_{am}^{i,k} \log\left(\frac{P_{am}^{i,k}}{P_{mm}^{i,k}}\right),
\]

wobei:
- \( N \) die Gesamtzahl der Pixel (oder Voxelknoten) ist,
- \( K \) die Anzahl der Klassen repräsentiert,
- \( P_{am}^{i,k} \) die vom Schülermodell vorhergesagte Wahrscheinlichkeit für Klasse \( k \) am Pixel \( i \) darstellt,
- \( P_{mm}^{i,k} \) die entsprechende Vorhersage des Lehrermodells ist.

### Detaillierte Erklärung

1. **Vergleich auf der Vorhersageebene**:  
   Während die vorherigen Distillationsansätze auf den Feature-Ebenen ansetzen, fokussiert sich \(\mathcal{L}_{mad}\) auf die Endergebnisse – also die Segmentierungskarten. Dies gewährleistet, dass das Schülermodell nicht nur ähnliche interne Repräsentationen erlernt, sondern auch konsistente und semantisch sinnvolle Vorhersagen trifft.

2. **Verwendung der Kullback-Leibler-Divergenz**:  
   Der Verlust misst die Divergenz zwischen den Verteilungen \( P_{am} \) und \( P_{mm} \) über alle Pixel und Klassen. Ein kleiner Wert bedeutet, dass sich die Wahrscheinlichkeitsverteilungen des Schülers eng an die des Lehrers annähern.

3. **Robustheit gegenüber fehlenden Modalitäten**:  
   Da das Lehrermodell auf vollständigen multimodalen Daten trainiert wurde, kann das Schülermodell durch diese Distillation auch dann lernen, konsistente Vorhersagen zu treffen, wenn nur eine Teilmenge der Modalitäten vorliegt.

---

## Zusammenfassung

Im PRISMS-Framework sorgen **PML**, **Cross-Modal Correspondence Distillation** und **Modality-Agnostic Distillation** gemeinsam dafür, dass das Segmentierungsmodell robust und flexibel auf unvollständige und variierende Datenmodalitäten reagiert.  
- **PML** legt den Grundstein, indem es alle Modalitäten gleichzeitig verarbeitet und so einen starken Lehrer aufbaut.  
- **Cross-Modal Distillation** gleicht intermodale Repräsentationen an und verhindert den Dominanzeffekt einzelner, leichter zu lernender Modalitäten.  
- **Modality-Agnostic Distillation** sichert auf der Vorhersageebene ab, dass semantische Informationen konsistent übertragen werden – auch wenn nur ein Teil der Modalitäten verfügbar ist.

Diese dreifache Strategie ermöglicht es PRISMS, in realen Anwendungsszenarien – sei es in der medizinischen Bildgebung, im autonomen Fahren oder in industriellen Anwendungen – zuverlässige Segmentierungsergebnisse zu erzielen, selbst wenn die Dateneingaben unvollständig oder heterogen sind.