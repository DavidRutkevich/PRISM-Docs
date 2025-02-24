---

title: "Unvollständige Trainingsdaten (UTD) in der Multimodalen Segmentierung"  
description: "Ein Überblick darüber, was UTD ist, warum es wichtig ist, wie damit umgegangen wird und welche Herausforderungen zu beachten sind."  
date: 2025-01-26  
math: true  

---

<span class="letterine"><i>U</i>nvollständige Trainingsdaten</span> (UTD) stellen in vielen realen Anwendungsszenarien die Regel und nicht die Ausnahme dar. UTD bezeichnet Datensätze, bei denen nicht alle ursprünglich vorgesehenen Modalitäten (oder Merkmale) für jedes Beispiel verfügbar sind. Im medizinischen Kontext kann dies beispielsweise bedeuten, dass für einzelne Patienten bestimmte MRT-Sequenzen nicht erfasst wurden.

![UTDvsVTD](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/VTD_UTD_Comp.png)

## Was ist UTD?

Es wird zwischen **vollständigen Trainingsdaten (VTD)** und **unvollständigen Trainingsdaten (UTD)** unterschieden:  
- **VTD:** Alle Modalitäten sind für jeden Datensatzpunkt vorhanden.  
- **UTD:** Eine oder mehrere Modalitäten fehlen entweder teilweise oder vollständig.

In UTD-Szenarien kann es vorkommen, dass bestimmte Modalitäten konstant fehlen (z. B. kostenintensive oder zeitaufwändige MRT-Sequenzen) oder nur in einem Teil der Datensätze vorhanden sind.

> **Beispiel:**  
> In einem Datensatz, der vier MRT-Sequenzen (T1, T1ce, T2, FLAIR) umfasst, können beispielsweise für 30 % aller Patienten keine T1- und T1ce-Aufnahmen vorliegen. Somit liegen unvollständige Informationen vor.

![VTD->UTD](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/vtd-utd.png)

## Warum ist UTD wichtig?

1. **Praktische Realität:**  
   In vielen Anwendungsbereichen, beispielsweise in der Radiologie oder Videoüberwachung, ist nicht immer jede Modalität verfügbar.

2. **Zeit- und Kostenersparnis:**  
   Bestimmte Diagnostikverfahren (wie Kontrastmittel-MRT) sind aufwändig und kostenintensiv. In der klinischen Praxis wird häufig entschieden, nur bei dringendem Verdacht alle Sequenzen aufzunehmen.

3. **Robuste Modelle:**  
   Modelle, die auch mit teilweise fehlenden Modalitäten umgehen können, zeigen eine höhere Robustheit und sind leichter in der Praxis einsetzbar.

## Wie wird mit UTD umgegangen?

Verschiedene Strategien werden eingesetzt, um trotz unvollständiger Daten valide Modelle zu trainieren. Zu den gängigen Methoden zählen:

1. **Daten-Imputation (Bildrekonstruktion):**  
   Fehlende Modalitäten werden synthetisch erzeugt (z. B. mittels GANs oder Diffusionsmodellen). Dieser Ansatz setzt jedoch oft voraus, dass in einer Vortrainingsphase vollständige Datensätze vorhanden waren.

2. **Wissenstransfer (Distillation):**  
   Zunächst wird ein Modell auf vollständigen Daten trainiert, das als Lehrer fungiert. Anschließend wird ein Schülermodell trainiert, das auf unimodalen Daten basiert. Diese Methode erfordert in der Regel zumindest einige vollständig annotierte Beispiele.

3. **Selbstdistillation und geteilte Repräsentationen:**  
   Ein einziges Netzwerk wird implementiert, das mehrere Encoder (einen pro Modalität) und einen gemeinsamen Fusionspfad umfasst. Das interne Wissen wird zwischen den Pfaden ausgetauscht. Dabei wird darauf geachtet, dass auch Modalitäten, die seltener vorkommen, gezielt unterstützt werden.

Die vorgestellten Ansätze werden kombiniert, um eine robuste Segmentierung zu ermöglichen, selbst wenn Modalitäten fehlen oder in geringer Qualität vorliegen.