---

title: "PRISM - Evaluierung"  
description: "Eine Analyse der Segmentierungsergebnisse bei unvollständigen multimodalen MRT-Daten unter Verwendung der PRISM-Methodik."  
date: 2025-01-26  
math: true  

-----------------------------  

## **Ergebnisse für BraTS2020**

In diesem Abschnitt präsentiere ich meine Ergebnisse auf dem **BraTS2020-Datensatz**, wobei ich unterschiedliche Modalitätskombinationen betrachte. Die Reihenfolge der Spalten zeigt die verschiedenen Ansätze: **Baseline, ModDrop, PMR und PRISM**. Mit PRISM setze ich ein selbstdistillierendes Netzwerk ein, um robuste Segmentierungsergebnisse zu erzielen.

### **Qualitative Ergebnisse (BraTS2020)**
![Qualitative Ergebnisse BraTS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/e090271a8e24c9725f1692590e3c487a2ae84cc0/qual_brats.svg)

#### **Analyse der Ergebnisse:**

- **Baseline**:  
  Bei der Baseline stelle ich zahlreiche Fehlklassifikationen fest. Sowohl falsch positive als auch falsch negative Bereiche treten auf. Besonders problematisch finde ich die inkonsistente Abgrenzung der Tumorregionen bei unvollständigen Modalitäten.
- **ModDrop**:  
  Mit ModDrop reduziere ich die falsch positiven Bereiche leicht, allerdings führt dies zu fragmentierten Segmentierungen. Einige Tumorregionen erscheinen unvollständig oder verzerrt.
- **PMR**:  
  Bei PMR priorisiere ich bestimmte Modalitäten, was zu einer besseren Abdeckung der Tumorkerne führt. Dennoch treten weiterhin größere falsch negative Bereiche auf, insbesondere wenn essentielle Modalitäten fehlen.
- **PRISM**:  
  Mit PRISM erziele ich die beste Balance zwischen Sensitivität und Spezifität. Die Tumorregionen sind klar abgegrenzt, falsch positive Bereiche werden minimiert und selbst bei reduzierten Modalitäten bleibt die Tumorkontur erhalten.

**Einfluss der Modalitätskombinationen:**  
- **T1 allein**:  
  Ich beobachte, dass T1 allein eine inkonsistente Erkennung der Tumorkontur liefert.  
- **T1ce und Flair**:  
  Diese Kombination führt zu besseren Ergebnissen, da sie die Enhancing-Region präziser erfasst.  
- **T2 allein**:  
  T2 allein liefert unbefriedigende Ergebnisse, da wichtige Informationen zu Enhancement-Regionen fehlen.  
- **T1ce und Flair**:  
  Ich komme zu dem Schluss, dass die Kombination aus T1ce und Flair eine der besten Alternativen darstellt, wenn nicht alle Modalitäten verfügbar sind.

#### **Quantitative Auswertung (BraTS2020)**

| Methode   | WT Dice (%) ↑ | TC Dice (%) ↑ | ET Dice (%) ↑ | Avg Dice (%) ↑ | HD Avg (mm) ↓ |
| --------- | ------------- | ------------- | ------------- | -------------- | ------------- |
| Baseline  | 76.89         | 64.36         | 51.72         | 64.32          | 19.35         |
| ModDrop   | 76.31         | 63.75         | 50.53         | 63.53          | 21.03         |
| PMR       | 77.59         | 65.44         | **52.86**     | 65.30          | 20.58         |
| **PRISM** | **83.42**     | **71.74**     | 52.78         | **69.31**      | **10.52**     |

Ich zeige, dass PRISM signifikante Verbesserungen in allen Metriken aufweist – insbesondere bei der Gesamt-Dice-Score und der Hausdorff-Distanz (HD) – was auf eine präzisere Segmentierung hindeutet.

---

## **Ergebnisse für MyoPS2020**

Für den **MyoPS2020-Datensatz** präsentiere ich eine ähnliche Auswertung. In der folgenden Abbildung zeige ich Segmentierungsergebnisse für verschiedene Modalitätskombinationen.

### **Qualitative Ergebnisse (MyoPS2020)**
![Qualitative Ergebnisse MyoPS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/e090271a8e24c9725f1692590e3c487a2ae84cc0/qual_myops.svg)

#### **Analyse der Ergebnisse:**

- **Baseline**:  
  Bei der Baseline habe ich Schwierigkeiten bei der Abgrenzung der Herzmuskelregion festgestellt. Falsch positive Bereiche treten insbesondere in den Randbereichen der Struktur auf.
- **ModDrop**:  
  Mit ModDrop verbessere ich die Gesamtform der Segmentierung, jedoch kommt es in einigen Bereichen zu Fragmentierungen.
- **PMR**:  
  Bei PMR erziele ich bessere Umrisse, neige jedoch dazu, einige relevante Strukturen nicht vollständig zu erfassen.
- **PRISM**:  
  Mit PRISM erreiche ich die höchste Präzision und reduziere sowohl falsch positive als auch falsch negative Bereiche erheblich.

**Einfluss der Modalitätskombinationen:**  
- **bSSFP allein**:  
  Ich stelle fest, dass bSSFP allein eine schwache Abgrenzung der Myokardstruktur liefert.
- **LGE**:  
  Ich beobachte, dass LGE die Identifikation von Infarktregionen verbessert, jedoch mit einem hohen Risiko für falsch positive Ergebnisse.
- **T2**:  
  T2 liefert zusätzliche Kontraste, ist aber allein unzureichend für eine präzise Segmentierung.
- **LGE und bSSFP**:  
  Ich komme zu dem Schluss, dass die Kombination aus LGE und bSSFP eine der stabilsten Segmentierungslösungen bietet.

#### **Quantitative Auswertung (MyoPS2020)**

| Methode   | LVBP Dice (%) ↑ | RVBP Dice (%) ↑ | MYO Dice (%) ↑ | Avg Dice (%) ↑ | HD Avg (mm) ↓ |
| --------- | --------------- | --------------- | -------------- | -------------- | ------------- |
| Baseline  | 72.69           | 51.94           | 66.81          | 63.81          | 22.11         |
| ModDrop   | 75.63           | 46.42           | 70.70          | 64.25          | 23.46         |
| PMR       | 73.05           | 52.61           | 69.32          | 64.99          | 20.54         |
| **PRISM** | **81.44**       | **60.97**       | **77.44**      | **73.28**      | **14.50**     |

Auch hier wird gezeigt, dass PRISM alle anderen Methoden in allen Metriken übertrifft und die Hausdorff-Distanz erheblich reduziert – ein klarer Hinweis auf eine genauere Segmentierung.