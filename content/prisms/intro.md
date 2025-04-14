---
title: "Einleitung zu PRISMS"
description: "Einführung in das PRISMS Framework zur robusten multimodalen Segmentierung."
date: 2025-02-05
math: true
---

Die zunehmende Verbreitung unvollständiger multimodaler Datensätze, insbesondere in der medizinischen Bildverarbeitung, stellt eine zentrale Herausforderung dar. Um robuste und präzise Segmentierungsergebnisse zu erzielen, wurde das Framework **PRISMS** entwickelt – eine Erweiterung des ursprünglichen PRISM-Ansatzes, die transformerbasierte Mechanismen und adaptive Fusionsstrategien integriert.

## Motivation

In realen Anwendungsszenarien, wie etwa bei der Magnetresonanztomographie (MRT), sind nicht alle Modalitäten stets in gleicher Qualität oder vollständig vorhanden. Dies führt zu einer ungleichen Verteilung der Informationen, wodurch traditionelle Fusionsansätze oft an ihre Grenzen stoßen. PRISMS adressiert dieses Problem, indem es fehlende Modalitäten intelligent kompensiert und eine robuste, globale Informationsintegration ermöglicht.

## Kernkomponenten von PRISMS

- **Adaptive Fusion Transformer (AFT):**  
  Die tiefsten Merkmalsrepräsentationen $x_5^m$ jeder Modalität werden zusammen mit lernbaren Fusionstokens $F_{\text{fusion}}$ in den AFT eingespeist. Mithilfe einer binären Indikationsmatrix $\mathbf{M} \in \{0,1\}^{N \times M}$ wird sichergestellt, dass nur tatsächlich vorhandene Modalitäten in den Fusionsprozess einfließen – so wird verhindert, dass fehlende Daten die Berechnung der Attention stören.

- **Spatial Relevance Attention (SRA):**  
  Da nicht alle räumlichen Regionen einer Modalität gleich wichtig sind – beispielsweise erfordern Tumorareale eine verstärkte Beachtung – erzeugt SRA Gewichtungsmasken $W_m$, die relevante Bereiche hervorheben und so die Genauigkeit der Segmentierung verbessern.

- **Kanalbezogener Fusionstransformer (KFT):**  
  Um redundante Informationen entlang der Kanaldimension zu reduzieren und wichtige Feature-Kanäle zu betonen, wird der KFT eingesetzt. Dies optimiert die Fusion der räumlich gewichteten Merkmale und trägt zur Effizienz der gesamten Architektur bei.

## Mathematischer Rahmen

Die Eingabedaten $x^m$ der einzelnen Modalitäten werden zunächst durch spezialisierte Encoder verarbeitet. Die resultierenden tiefen Merkmale $x_5^m$ dienen als Basis für die multimodale Fusion im AFT, in dem auch lernbare Tokens $F_{\text{fusion}}$ zum Einsatz kommen, um den Informationsaustausch zu steuern. Ein zentraler Aspekt ist die Modalitätsmaskierung, welche durch die folgende Attention-Formel verdeutlicht wird:

$$
\mathrm{Attn}(Q,K,i,j) =
\begin{cases}
\exp\left(\dfrac{Q_{i} K_{j}^\top}{\sqrt{d}}\right), & \text{wenn } M_{ij} = 1, \\
0, & \text{wenn } M_{ij} = 0.
\end{cases}
$$

Durch diese gezielte Maskierung wird verhindert, dass fehlende Modalitäten in den Fusionsprozess einfließen und somit den Informationsaustausch stören. Die Kombination der AFT-, SRA- und KFT-Komponenten ermöglicht so eine robuste, globale Integration der vorhandenen Modalitäten.

## Zielsetzung

PRISMS verfolgt das Ziel, ein flexibles und skalierbares Framework bereitzustellen, das:
- **Adaptiv** fehlende Modalitäten erkennt und kompensiert,
- **Effizient** Ressourcen schont, indem es zusätzliche Bildaufnahmezyklen vermeidet,
- **Generalisiert** zuverlässige Segmentierungsergebnisse in verschiedensten Anwendungsszenarien liefert.

Durch den Einsatz von Selbstdistillation, adaptiven Lernraten und transformerbasierten Fusionsmechanismen schafft PRISMS ein System, das den Herausforderungen unvollständiger und unbalancierter multimodaler Daten erfolgreich begegnet.
