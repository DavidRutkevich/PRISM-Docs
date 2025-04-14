---
title: "PRISM – Evaluierung"  
description: "Analyse der Segmentierungsergebnisse unvollständiger multimodaler MRT-Daten mithilfe der PRISM-Methodik."  
date: 2025-01-26  
math: true
---

## Implementierung und Hyperparameter

Zur Evaluierung des Standes der Technik im unbalancierten multimodalen Lernen wurden zwei moderne Backbones verwendet:  
- **mmFormer** von Zhan et al. [3]  
- **RFNet** von Ding et al. [2] (als Plug-and-Play-Modul)

Alle Modelle wurden in PyTorch implementiert und hinsichtlich der Eingabedimensionen vereinheitlicht.

### Optimierungsstrategie

- **Optimierer:** AdamW  
- **Initiale Lernrate:** $2 \times 10^{-4}$  
- **Weight Decay:** $1 \times 10^{-4}$  
- **Batch-Größe:** 1  
- **Epochen:** 300  
- **Hardware:** NVIDIA GeForce RTX 4090 GPU  
- **Lernraten-Plan:** Poly-Decay mit $\rho = 0.9$

### Hyperparameter der Self-Distillation und Regularisierung

- **Temperatur für pixelweise Self-Distillation ($\mu$)**: 4  
- **Hyperparameter für den Selbstdistillations-Loss:**  
  - Pixelbasierter Loss: $\gamma_1 = 0.5$  
  - Semantischer Loss: $\gamma_2 = 0.1$
- **Hyperparameter für die Rebalancierungs-Supervision:** $\lambda = 0.01$

### Datenvorverarbeitung und Simulation fehlender Modalitäten

- **Brain Scans:**  
  - Zufälliger Zuschnitt auf Patchgrößen von $80 \times 80 \times 80$
- **Herzscans:**  
  - Skalierung auf $256 \times 256$ Pixel
- **Datenaugmentation:**  
  - Zufällige Rotationen und Intensitätsveränderungen
- **Simulation fehlender Modalitäten:**  
  - Modalitäten werden während des Trainings anhand definierter Fehlraten ($FR$) zufällig maskiert.  
  - Die maskierten Modalitäten werden vollständig entfernt (als *unsichtbar* behandelt) und ihre enkodierten Merkmalsvektoren auf Null gesetzt.

### Reproduzierbarkeit

Das zugehörige Colab Notebook zur Reproduzierbarkeit der Ergebnisse kann auf Anfrage bereitgestellt werden. Bitte beachten Sie, dass der Code nicht als Open Source verfügbar ist, da PRISM(S) wirtschaftlich genutzt wird.


## Ergebnisse für BraTS2020

Auf dem **BraTS2020-Datensatz** wurden unterschiedliche Modalitätskombinationen untersucht. Die Spalten in den folgenden Tabellen repräsentieren die Ergebnisse der einzelnen Ansätze: Baseline, ModDrop, PMR und PRISM. PRISM verwendet ein selbstdistillierendes Netzwerk, das multimodales Wissen effizient auf unimodale Pfade überträgt.

### Qualitative Ergebnisse

![Qualitative Ergebnisse BraTS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/new_figures/Prism%20%7C%20Modul/qual_brats.svg)

#### Analyse

- **Baseline:**  
  Die herkömmliche Baseline zeigt zahlreiche Fehlklassifikationen. Sowohl falsch positive als auch falsch negative Bereiche treten auf, und die Abgrenzung der Tumorregionen ist inkonsistent – insbesondere bei unvollständigen Modalitäten.

- **ModDrop:**  
  Mit ModDrop können falsch positive Bereiche reduziert werden, allerdings führt dies gelegentlich zu fragmentierten Segmentierungen, bei denen einige Tumorregionen unvollständig erscheinen.

- **PMR:**  
  Der Einsatz von PMR verbessert die Erfassung der Tumorkerne, zeigt jedoch weiterhin größere falsch negative Bereiche, vor allem wenn wesentliche Modalitäten fehlen.

- **PRISM:**  
  PRISM erzielt die beste Balance: Die Tumorregionen sind klar abgegrenzt, falsch positive Bereiche werden minimiert und die Tumorkontur bleibt auch bei reduzierten Modalitäten weitgehend erhalten.

**Einfluss der Modalitätskombinationen:**  
- **T1 allein:**  
  Die alleinige Nutzung von T1 führt zu einer unsicheren Segmentierung der Tumorkontur.  
- **T1ce und FLAIR:**  
  Diese Kombination verbessert die Erfassung der Enhancing-Regionen erheblich.  
- **T2 allein:**  
  T2 liefert weniger präzise Ergebnisse, da wichtige Informationen zu den Enhancement-Regionen fehlen.

### Quantitative Auswertung

| Methode   | WT Dice (%) ↑ | TC Dice (%) ↑ | ET Dice (%) ↑ | Avg Dice (%) ↑ | HD Avg (mm) ↓ |
| --------- | ------------- | ------------- | ------------- | -------------- | ------------- |
| Baseline  | 76.89         | 64.36         | 51.72         | 64.32          | 19.35         |
| ModDrop   | 76.31         | 63.75         | 50.53         | 63.53          | 21.03         |
| PMR       | 77.59         | 65.44         | **52.86**     | 65.30          | 20.58         |
| **PRISM** | **83.42**     | **71.74**     | 52.78         | **69.31**      | **10.52**     |

Die Tabelle zeigt, dass PRISM in allen Metriken signifikant bessere Ergebnisse liefert. Besonders auffällig ist der Anstieg der durchschnittlichen Dice-Werte sowie die Reduktion der Hausdorff-Distanz, was auf eine präzisere Segmentierung hinweist.

## Ergebnisse für MyoPS2020

Für den **MyoPS2020-Datensatz** wurden ebenfalls verschiedene Modalitätskombinationen evaluiert. Die folgenden Ergebnisse veranschaulichen den Einfluss der unterschiedlichen Ansätze auf die Segmentierungsgenauigkeit.

### Qualitative Ergebnisse

![Qualitative Ergebnisse MyoPS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/64ce7eb4b98fa0a90b9236f6d4711a44b9709aa3/Prism%20%7C%20Modul/qual_myops.svg)

#### Analyse

- **Baseline:**  
  Die Baseline weist Schwierigkeiten bei der Abgrenzung der Herzmuskelregion auf. Besonders in den Randbereichen treten falsch positive Segmentierungen auf.

- **ModDrop:**  
  ModDrop verbessert die Gesamtform, führt jedoch in einigen Bereichen zu Fragmentierungen der Segmentierung.

- **PMR:**  
  PMR verbessert die Konturendarstellung, erfasst aber nicht alle relevanten Strukturen vollständig.

- **PRISM:**  
  Mit PRISM werden sowohl falsch positive als auch falsch negative Bereiche deutlich reduziert, was zu einer präziseren Segmentierung der Myokardregion führt.

**Einfluss der Modalitätskombinationen:**  
- **bSSFP allein:**  
  Der alleinige Einsatz von bSSFP führt zu einer unklaren Abgrenzung der Myokardstruktur.  
- **LGE:**  
  LGE verbessert die Identifikation von Infarktregionen, birgt jedoch ein höheres Risiko für falsch positive Ergebnisse.  
- **T2 allein:**  
  T2 bietet zusätzliche Kontraste, reicht aber nicht aus, um eine präzise Segmentierung zu erzielen.  
- **LGE und bSSFP:**  
  Diese Kombination erweist sich als eine der stabilsten Lösungen, insbesondere wenn nicht alle Modalitäten verfügbar sind.

### Quantitative Auswertung

| Methode   | LVBP Dice (%) ↑ | RVBP Dice (%) ↑ | MYO Dice (%) ↑ | Avg Dice (%) ↑ | HD Avg (mm) ↓ |
| --------- | --------------- | --------------- | -------------- | -------------- | ------------- |
| Baseline  | 72.69           | 51.94           | 66.81          | 63.81          | 22.11         |
| ModDrop   | 75.63           | 46.42           | 70.70          | 64.25          | 23.46         |
| PMR       | 73.05           | 52.61           | 69.32          | 64.99          | 20.54         |
| **PRISM** | **81.44**       | **60.97**       | **77.44**      | **73.28**      | **14.50**     |

Auch hier bestätigt die quantitative Auswertung, dass PRISM alle anderen Ansätze übertrifft – insbesondere die signifikant niedrigere Hausdorff-Distanz weist auf eine sehr präzise Segmentierung hin.

---

## Weitere Implementierungsdetails
Die gradientenbasierte Rebalancierungsregularisierung wird mittels Gradientenabstieg implementiert, indem der durchschnittliche relative Präferenzwert (epoch-average) in die Aktualisierung der gewichteten Koeffizienten einfließt. Da der Gradient jedoch nicht direkt mit dem pixelbezogenen Optimierungsziel der Selbstdistillation verknüpft ist, wurden zusätzliche Beschränkungen eingeführt. Konkret wird der Gradientenabstieg auf eine zulässige Domäne projiziert, was wie folgt formuliert werden kann:

$$
\mathcal{E}^m_{r+1} = \mathcal{E}_r^m - \gamma \cdot \bar{RP}^m,
$$

$$
[\mathcal{E}^1_{r+1}, \mathcal{E}^2_{r+1}, \dots, \mathcal{E}^M_{r+1}] = \operatorname{Proj}_{\tau_{\mathcal{E}}}\Bigl( [\mathcal{E}^1_{r+1}, \mathcal{E}^2_{r+1}, \dots, \mathcal{E}^M_{r+1}] \Bigr).
$$

Hierbei indexiert $r$ die Epoche, und $\tau_{\mathcal{E}}$ bezeichnet den zulässigen Bereich der Koeffizienten

$$
\mathcal{E} = [\mathcal{E}^1, \mathcal{E}^2, \dots, \mathcal{E}^M].
$$

Wir definieren die zulässige Domäne als

$$
\tau_{\mathcal{E}} = \Bigl\{\mathcal{E} \in \mathbb{R}^M \;\Big|\; \mathcal{E}^m \geq 0.1 \text{ und } \|\mathcal{E}\|_2 = \|\mathcal{E}_0\|_2\Bigr\},
$$

wobei $Proj_{\tau_{\mathcal{E}}}[\cdot]$ die Projektion auf diese Domäne bezeichnet und $\gamma$ die Lernrate für die Aktualisierung der gewichteten Koeffizienten darstellt.

