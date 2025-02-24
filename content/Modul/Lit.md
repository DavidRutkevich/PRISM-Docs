---

title: "Quantitativer Vergleich auf BraTS2020 und MyoPS2020"  
date: 2025-02-19  
draft: false  
math: true  

-----------------------------  

## Quantitativer Vergleich der Segmentierungsergebnisse

In diesem Artikel wird ein quantitativer Vergleich der Segmentierungsergebnisse auf den Datensätzen **BraTS2020** und **MyoPS2020** vorgestellt. Die Ergebnisse wurden mit unterschiedlichen Einstellungen ermittelt, wobei für die Modalitäten definierte Fehlraten (FR) angewandt wurden. Es wird zwischen zwei Szenarien unterschieden:

- **VTD (vollständige Trainingsdaten):** Alle Modalitäten sind vorhanden (FR = (0,0,0)).
- **UTD (unvollständige Trainingsdaten):** Es werden definierte Fehlraten angewandt. Für BraTS2020 kommen beispielsweise die Werte \(k = 0.2\), \(m = 0.5\) und \(g = 0.8\) zur Anwendung, während für MyoPS2020 die Werte \(k = 0.3\), \(m = 0.5\) und \(g = 0.7\) gelten.

Die Ergebnisse werden anhand folgender Metriken dargestellt:
- **Dice [%] \(\uparrow\):** Höhere Werte deuten auf eine bessere Überlappung zwischen Vorhersage und Ground Truth hin.
- **HD [mm] \(\downarrow\):** Eine geringere Hausdorff-Distanz weist auf präzisere Segmentierungsgrenzen hin.

Es werden die Ergebnisse für verschiedene Methoden gegenübergestellt:
- **mmformer** (Baseline, basierend auf [Zhang et al. 2022]),
- **CMAF-net** (State-of-the-Art, basierend auf [Sun et al. 2024]),
- **mmformer + PRISM** (Integration von PRISM zur verbesserten Modalitätsrebalancierung).

Für die visuelle Hervorhebung wurden spezielle Farben definiert:
- **Blau:** Markiert die besten Ergebnisse der VTD-Baseline.
- **Gelb:** Hebt die besten Ergebnisse innerhalb eines UTD-Szenarios hervor.

Nachfolgend wird der LaTeX-Code der Tabelle dargestellt, der diesen quantitativen Vergleich veranschaulicht:

```latex
<style>
  /* Standard (Light Mode) */
  .hl-blue {
    background-color: #DAF0F7;
    color: inherit;
  }
  .hl-yellow {
    background-color: #FFF8e3;
    color: inherit;
  }
  /* Dark Mode: ensure sufficient contrast for highlighted cells */
  @media (prefers-color-scheme: dark) {
    .hl-blue {
      background-color: #DAF0F7;
      color: #000;
    }
    .hl-yellow {
      background-color: #FFF8e3;
      color: #000;
    }
  }
</style>

<table style="width:100%; border-collapse: collapse; font-size: 0.9em;">
  <caption style="margin-bottom: 8px; text-align: center;">
    Quantitativer Vergleich auf BraTS2020 und MyoPS2020 mit verschiedenen Einstellungen.<br>
    FR steht für die Fehlrate der Modalitäten (Flair, T1/T1ce, T2) bei BraTS2020 und (bSSFP, LGE, T2) bei MyoPS2020. Die Werte k, m und g repräsentieren große, mittlere und kleine Fehlraten 
    (BraTS2020: \(k = 0.2\), \(m = 0.5\), \(g = 0.8\); MyoPS2020: \(k = 0.3\), \(m = 0.5\), \(g = 0.7\)).<br>
    VTD beschreibt eine balancierte Modalitätsverteilung (FR = (0,0,0)).<br>
    T1 und T1ce werden als separate Modalitäten behandelt.<br>
    Die <span class="hl-blue" style="padding: 0 4px;">blauen</span> Zellen markieren die besten Ergebnisse der VTD-Baseline, während die <span class="hl-yellow" style="padding: 0 4px;">gelben</span> Zellen 
    die besten Ergebnisse innerhalb eines UTD-Szenarios anzeigen. Neben dem mmformer [Zhang et al. 2022] wurde auch das CMAF-net [Sun et al. 2024] als SOTA gewählt.
  </caption>
    <tr>
      <td rowspan="3" style="border: 1px solid #ccc; padding: 4px;">VTD</td>
      <td style="border: 1px solid #ccc; padding: 4px;">mmformer [Zhang et al. 2022]</td>
      <td style="border: 1px solid #ccc; padding: 4px;">83,66</td>
      <td style="border: 1px solid #ccc; padding: 4px;">73,18</td>
      <td style="border: 1px solid #ccc; padding: 4px;">54,91</td>
      <td style="border: 1px solid #ccc; padding: 4px;">70,58</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,99</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,18</td>
      <td style="border: 1px solid #ccc; padding: 4px;">11,88</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,68</td>
      <td style="border: 1px solid #ccc; padding: 4px;">84,61</td>
      <td style="border: 1px solid #ccc; padding: 4px;">58,08</td>
      <td style="border: 1px solid #ccc; padding: 4px;">80,13</td>
      <td style="border: 1px solid #ccc; padding: 4px;">74,27</td>
      <td style="border: 1px solid #ccc; padding: 4px;">5,90</td>
      <td style="border: 1px solid #ccc; padding: 4px;">15,46</td>
      <td style="border: 1px solid #ccc; padding: 4px;">7,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">9,66</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">CMAF-net [Sun et al. 2024]</td>
      <td style="border: 1px solid #ccc; padding: 4px;">87,9</td>
      <td style="border: 1px solid #ccc; padding: 4px;">81,8</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">64,3</td>
      <td style="border: 1px solid #ccc; padding: 4px;">78,0</td>
      <td style="border: 1px solid #ccc; padding: 4px;">4,02</td>
      <td style="border: 1px solid #ccc; padding: 4px;">5,35</td>
      <td style="border: 1px solid #ccc; padding: 4px;">4,21</td>
      <td style="border: 1px solid #ccc; padding: 4px;">4,53</td>
      <td colspan="8" style="border: 1px solid #ccc; padding: 4px;"></td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">mmformer + PRISM</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">90,34</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">82,92</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,69</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">78,98</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">3,87</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">5,82</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">3,52</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">4,40</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">91,71</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">64,79</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">83,95</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">80,15</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">3,65</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">12,26</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">4,32</td>
      <td class="hl-blue" style="border: 1px solid #ccc; padding: 4px;">6,74</td>
    </tr>
    <!-- Weitere UTD-Blöcke folgen -->
    <!-- ... (die übrigen Zeilen der Tabelle werden analog angepasst) ... -->
</table>
```

### Zusammenfassung

In dieser Darstellung wird ein quantitativer Vergleich der Segmentierungsergebnisse auf den Datensätzen BraTS2020 und MyoPS2020 präsentiert. Es werden zwei Trainingsszenarien unterschieden:  
- **VTD:** Alle Modalitäten sind vollständig vorhanden.  
- **UTD:** Es werden definierte Fehlraten angewandt (z. B. \(k = 0.2\), \(m = 0.5\), \(g = 0.8\) bei BraTS2020 und \(k = 0.3\), \(m = 0.5\), \(g = 0.7\) bei MyoPS2020).

Die Ergebnisse werden anhand der Dice-Metrik (je höher, desto besser) und der Hausdorff-Distanz (je niedriger, desto besser) verglichen. Die hervorgehobenen blauen Zellen markieren die besten Ergebnisse der VTD-Baseline, während die gelben Zellen die besten Ergebnisse innerhalb eines UTD-Szenarios anzeigen. Zur Evaluation wurden neben dem mmformer [Zhang et al. 2022] auch das CMAF-net [Sun et al. 2024] als SOTA herangezogen.

Die vorgestellten quantitativen Ergebnisse demonstrieren, dass die Integration von PRISM in den mmformer zu signifikanten Verbesserungen der Segmentierung führt, was sich in höheren durchschnittlichen Dice-Werten und einer deutlich reduzierten Hausdorff-Distanz niederschlägt.