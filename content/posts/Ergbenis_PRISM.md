---

title: "PRISM - Evaluierung"
description: "Eine Analyse der Segmentierungsergebnisse bei unvollständigen multimodalen MRT-Daten unter Verwendung der PRISM-Methodik."
date: 2025-01-26
type: post
draft: false
translationKey: prism_eval
coffee: 1
tags: ['PRISM', 'Evaluierung']
categories: ['Modul']
-----------------------------
## **Ergebnisse für BraTS2020**

Die folgenden Abbildungen zeigen qualitative Ergebnisse auf dem **BraTS2020-Datensatz**, wobei unterschiedliche Modalitätskombinationen betrachtet wurden. Die Reihenfolge der Spalten zeigt die verschiedenen Ansätze: **Baseline, ModDrop, PMR und PRISM**. PRISM nutzt dabei ein selbstdistillierendes Netzwerk, um robuste Segmentierungsergebnisse zu erzielen.

### **Qualitative Ergebnisse (BraTS2020)**
![Qualitative Ergebnisse BraTS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/e090271a8e24c9725f1692590e3c487a2ae84cc0/qual_brats.svg)

#### **Analyse der Ergebnisse:**

- **Baseline**: Die Segmentierung weist zahlreiche Fehlklassifikationen auf. Sowohl falsch positive als auch falsch negative Bereiche sind erkennbar. Besonders problematisch ist die inkonsistente Abgrenzung der Tumorregionen bei unvollständigen Modalitäten.
- **ModDrop**: Reduziert die falsch positiven Bereiche leicht, führt jedoch zu fragmentierten Segmentierungen. Einige Tumorregionen erscheinen unvollständig oder verzerrt.
- **PMR**: Die Methode priorisiert bestimmte Modalitäten, was zu einer besseren Abdeckung der Tumorkerne führt. Allerdings treten weiterhin größere falsch negative Bereiche auf, insbesondere wenn essentielle Modalitäten fehlen.
- **PRISM**: Die Segmentierungen zeigen die beste Balance zwischen Sensitivität und Spezifität. Die Tumorregionen sind klar abgegrenzt, falsch positive Bereiche sind minimiert und selbst bei reduzierten Modalitäten bleibt die Tumorkontur erhalten.

**Einfluss der Modalitätskombinationen:**
- **T1 allein** zeigt eine inkonsistente Erkennung der Tumorkontur.
- **T1ce und Flair** führen zu besseren Ergebnissen, da sie die Enhancing-Region besser erfassen.
- **T2 allein liefert schlechte Ergebnisse**, da wichtige Informationen zu Enhancement-Regionen fehlen.
- **Die Kombination aus T1ce und Flair bietet eine der besten Alternativen**, wenn nicht alle Modalitäten verfügbar sind.

#### **Quantitative Auswertung (BraTS2020)**

| Methode   | WT Dice (%) ↑ | TC Dice (%) ↑ | ET Dice (%) ↑ | Avg Dice (%) ↑ | HD Avg (mm) ↓ |
| --------- | ------------- | ------------- | ------------- | -------------- | ------------- |
| Baseline  | 76.89         | 64.36         | 51.72         | 64.32          | 19.35         |
| ModDrop   | 76.31         | 63.75         | 50.53         | 63.53          | 21.03         |
| PMR       | 77.59         | 65.44         | **52.86**     | 65.30          | 20.58         |
| **PRISM** | **83.42**     | **71.74**     | 52.78         | **69.31**      | **10.52**     |

PRISM zeigt signifikante Verbesserungen in allen Metriken, insbesondere bei der Gesamt-Dice-Score und der Hausdorff-Distanz (HD), was auf eine präzisere Segmentierung hindeutet.

---

## **Ergebnisse für MyoPS2020**

Für den **MyoPS2020-Datensatz** betrachten wir eine ähnliche Auswertung. Die folgende Abbildung zeigt Segmentierungsergebnisse für verschiedene Modalitätskombinationen.

### **Qualitative Ergebnisse (MyoPS2020)**
![Qualitative Ergebnisse MyoPS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/e090271a8e24c9725f1692590e3c487a2ae84cc0/qual_myops.svg)

#### **Analyse der Ergebnisse:**

- **Baseline**: Schwierigkeiten bei der Abgrenzung der Herzmuskelregion. Falsch positive Bereiche treten insbesondere in den Randbereichen der Struktur auf.
- **ModDrop**: Verbessert die Gesamtform der Segmentierung, führt jedoch zu Fragmentierung in einigen Bereichen.
- **PMR**: Liefert bessere Umrisse, neigt jedoch dazu, einige relevante Strukturen nicht zu erfassen.
- **PRISM**: Zeigt die höchste Präzision und reduziert sowohl falsch positive als auch falsch negative Bereiche erheblich.

**Einfluss der Modalitätskombinationen:**
- **bSSFP allein liefert eine schwache Abgrenzung der Myokardstruktur**.
- **LGE verbessert die Identifikation von Infarktregionen**, jedoch mit hohem Risiko für falsch positive Ergebnisse.
- **T2 liefert zusätzliche Kontraste**, ist aber allein unzureichend für eine präzise Segmentierung.
- **Die Kombination aus LGE und bSSFP bietet eine der stabilsten Segmentierungslösungen**.

#### **Quantitative Auswertung (MyoPS2020)**

| Methode   | LVBP Dice (%) ↑ | RVBP Dice (%) ↑ | MYO Dice (%) ↑ | Avg Dice (%) ↑ | HD Avg (mm) ↓ |
| --------- | --------------- | --------------- | -------------- | -------------- | ------------- |
| Baseline  | 72.69           | 51.94           | 66.81          | 63.81          | 22.11         |
| ModDrop   | 75.63           | 46.42           | 70.70          | 64.25          | 23.46         |
| PMR       | 73.05           | 52.61           | 69.32          | 64.99          | 20.54         |
| **PRISM** | **81.44**       | **60.97**       | **77.44**      | **73.28**      | **14.50**     |

Auch hier übertrifft PRISM alle anderen Methoden in allen Metriken und reduziert die Hausdorff-Distanz erheblich, was auf eine genauere Segmentierung hinweist.

---

## **Fazit**

Die PRISM-Methodik ermöglicht eine signifikante Verbesserung der Segmentierungsqualität und reduziert Fehler, die bei unvollständigen Modalitätsdaten auftreten. Besonders in klinischen Anwendungen, in denen Modalitäten variabel oder unvollständig sind, zeigt PRISM eine hohe Robustheit.

