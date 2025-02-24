---

title: "PRISM - Evaluierung"  
description: "Eine Analyse der Segmentierungsergebnisse bei unvollständigen multimodalen MRT-Daten unter Verwendung der PRISM-Methodik."  
date: 2025-01-26  
math: true  

-----------------------------  

## Ergebnisse für BraTS2020

Im Folgenden werden die Ergebnisse auf dem **BraTS2020-Datensatz** vorgestellt, wobei unterschiedliche Modalitätskombinationen untersucht wurden. Die Spaltenreihenfolge gibt die verschiedenen Ansätze an: **Baseline, ModDrop, PMR und PRISM**. Mittels PRISM wird ein selbstdistillierendes Netzwerk eingesetzt, um robuste Segmentierungsergebnisse zu erzielen.

### Qualitative Ergebnisse (BraTS2020)
![Qualitative Ergebnisse BraTS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/e090271a8e24c9725f1692590e3c487a2ae84cc0/qual_brats.svg)

#### Analyse der Ergebnisse

- **Baseline**:  
  Bei der Baseline treten zahlreiche Fehlklassifikationen auf. Sowohl falsch positive als auch falsch negative Bereiche sind festzustellen. Insbesondere zeigt sich eine inkonsistente Abgrenzung der Tumorregionen, wenn unvollständige Modalitäten vorliegen.
- **ModDrop**:  
  Durch den Einsatz von ModDrop können die falsch positiven Bereiche leicht reduziert werden, jedoch führt dies teilweise zu fragmentierten Segmentierungen. Einige Tumorregionen erscheinen unvollständig oder verzerrt.
- **PMR**:  
  Mit PMR erfolgt eine Priorisierung bestimmter Modalitäten, was zu einer besseren Abdeckung der Tumorkerne führt. Dennoch bleiben größere falsch negative Bereiche bestehen, vor allem wenn essentielle Modalitäten fehlen.
- **PRISM**:  
  Der Einsatz von PRISM ermöglicht die beste Balance zwischen Sensitivität und Spezifität. Die Tumorregionen werden klar abgegrenzt, falsch positive Bereiche werden minimiert und die Tumorkontur bleibt selbst bei reduzierten Modalitäten erhalten.

**Einfluss der Modalitätskombinationen:**  
- **T1 allein**:  
  Die alleinige Nutzung von T1 führt zu einer inkonsistenten Erkennung der Tumorkontur.  
- **T1ce und FLAIR**:  
  Diese Kombination resultiert in besseren Ergebnissen, da die Enhancing-Region präziser erfasst wird.  
- **T2 allein**:  
  Der alleinige Einsatz von T2 liefert unbefriedigende Ergebnisse, da wichtige Informationen zu den Enhancement-Regionen fehlen.  
- **T1ce und FLAIR**:  
  Es wird festgestellt, dass die Kombination aus T1ce und FLAIR eine der besten Alternativen darstellt, wenn nicht alle Modalitäten verfügbar sind.

#### Quantitative Auswertung (BraTS2020)

| Methode   | WT Dice (%) ↑ | TC Dice (%) ↑ | ET Dice (%) ↑ | Avg Dice (%) ↑ | HD Avg (mm) ↓ |
| --------- | ------------- | ------------- | ------------- | -------------- | ------------- |
| Baseline  | 76.89         | 64.36         | 51.72         | 64.32          | 19.35         |
| ModDrop   | 76.31         | 63.75         | 50.53         | 63.53          | 21.03         |
| PMR       | 77.59         | 65.44         | **52.86**     | 65.30          | 20.58         |
| **PRISM** | **83.42**     | **71.74**     | 52.78         | **69.31**      | **10.52**     |

Die Ergebnisse zeigen, dass PRISM in allen Metriken signifikante Verbesserungen erzielt. Insbesondere steigen die durchschnittlichen Dice-Werte deutlich an, während die Hausdorff-Distanz (HD) erheblich reduziert wird – ein Hinweis auf eine präzisere Segmentierung.

---

## Ergebnisse für MyoPS2020

Für den **MyoPS2020-Datensatz** wird eine ähnliche Auswertung präsentiert. Die folgende Abbildung zeigt Segmentierungsergebnisse für verschiedene Modalitätskombinationen.

### Qualitative Ergebnisse (MyoPS2020)
![Qualitative Ergebnisse MyoPS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/e090271a8e24c9725f1692590e3c487a2ae84cc0/qual_myops.svg)

#### Analyse der Ergebnisse

- **Baseline**:  
  Bei der Baseline ergeben sich Schwierigkeiten bei der Abgrenzung der Herzmuskelregion. Insbesondere in den Randbereichen der Struktur sind falsch positive Bereiche zu erkennen.
- **ModDrop**:  
  Der Einsatz von ModDrop führt zu einer Verbesserung der Gesamtform der Segmentierung, jedoch kommt es in einigen Bereichen zu Fragmentierungen.
- **PMR**:  
  Bei PMR werden die Umrisse verbessert, es kommt jedoch zu einer unvollständigen Erfassung einiger relevanter Strukturen.
- **PRISM**:  
  Der Einsatz von PRISM führt zu einer höchsten Präzision. Sowohl falsch positive als auch falsch negative Bereiche werden erheblich reduziert.

**Einfluss der Modalitätskombinationen:**  
- **bSSFP allein**:  
  Der alleinige Einsatz von bSSFP führt zu einer schwachen Abgrenzung der Myokardstruktur.  
- **LGE**:  
  Die Nutzung von LGE verbessert die Identifikation von Infarktregionen, birgt jedoch ein höheres Risiko für falsch positive Ergebnisse.  
- **T2**:  
  T2 liefert zusätzliche Kontraste, ist jedoch allein nicht ausreichend für eine präzise Segmentierung.  
- **LGE und bSSFP**:  
  Es wird festgestellt, dass die Kombination aus LGE und bSSFP eine der stabilsten Segmentierungslösungen darstellt.

#### Quantitative Auswertung (MyoPS2020)

| Methode   | LVBP Dice (%) ↑ | RVBP Dice (%) ↑ | MYO Dice (%) ↑ | Avg Dice (%) ↑ | HD Avg (mm) ↓ |
| --------- | --------------- | --------------- | -------------- | -------------- | ------------- |
| Baseline  | 72.69           | 51.94           | 66.81          | 63.81          | 22.11         |
| ModDrop   | 75.63           | 46.42           | 70.70          | 64.25          | 23.46         |
| PMR       | 73.05           | 52.61           | 69.32          | 64.99          | 20.54         |
| **PRISM** | **81.44**       | **60.97**       | **77.44**      | **73.28**      | **14.50**     |

Die quantitativen Ergebnisse bestätigen, dass PRISM alle anderen Ansätze in allen Metriken übertrifft. Insbesondere wird die Hausdorff-Distanz deutlich reduziert, was auf eine präzisere Segmentierung hinweist.

---

Die vorliegenden Ergebnisse demonstrieren, dass die PRISM-Methode sowohl bei den Gehirntumorsegmentierungen (BraTS2020) als auch bei den kardiologischen Segmentierungen (MyoPS2020) zu signifikanten Verbesserungen führt. Durch den Einsatz der selbstdistillierenden Architektur werden die lokalen und globalen Repräsentationen optimal angepasst, sodass auch bei unvollständigen Modalitäten robuste und präzise Segmentierungsergebnisse erzielt werden.
