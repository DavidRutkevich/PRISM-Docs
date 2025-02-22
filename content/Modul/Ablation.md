---

title: "PRISM – Ablation"  
description: "Eine Ablation der PRISM Methode und kurze Kommentare und Anmerkungen."  
date: 2025-02-05  
math: true

---

# Ablationsstudie zu PRISM

In diesem Abschnitt betrachte ich die Ablationsstudie, um die Wirkung einzelner Komponenten sowie verschiedener Distanz- und Hyperparameter-Einstellungen von PRISM genauer zu untersuchen. Die Ablation ist entscheidend, um nachvollziehen zu können, welche Teilschritte in der **Methodik** den stärksten Einfluss auf das Endergebnis haben und wie sich das Zusammenspiel einzelner Module auswirkt.

---

## 1. Komponenten-Ablation

Im Folgenden betrachte ich zunächst die einzelnen Bausteine von PRISM. Dazu dient die Tabelle **Komponenten Ablation auf BraTS2020 und MyoPS2020**. Die Zeilen unterscheiden sich darin, ob sie den **pixelweisen Distillation-Term** $\ell_{\text{pixel}}$, den **Prototyp-basierten Term** $\ell_{\text{proto}}$, die **Rebalancierungsmaske** $\omega$ und/oder den **gradientengewichteten Koeffizienten** $E$ aktivieren. Aus der Tabelle geht hervor, wie jeder dieser Bausteine die Ergebnisse jeweils verbessert.

<table>
  <caption>Komponenten Ablation auf BraTS2020 und MyoPS2020.</caption>
  <thead>
    <tr>
      <th colspan="4" rowspan="2">Komponenten</th>
      <th colspan="8">BraTS2020 FR=(0.2, 0.4, 0.6, 0.8)</th>
      <th colspan="8">MyoPS2020 FR=(0.3,0.5,0.7)</th>
    </tr>
    <tr>
      <th colspan="4">DICE [%] ↑</th>
      <th colspan="4">HD [mm] ↓</th>
      <th colspan="4">DICE [%] ↑</th>
      <th colspan="4">HD [mm] ↓</th>
    </tr>
    <tr>
      <th>L<sub>pixel</sub></th>
      <th>L<sub>proto</sub></th>
      <th>ω</th>
      <th>E</th>
      <th>WT</th>
      <th>TC</th>
      <th>ET</th>
      <th>Avg.</th>
      <th>WT</th>
      <th>TC</th>
      <th>ET</th>
      <th>Avg.</th>
      <th>LVB</th>
      <th>RVB</th>
      <th>MYO</th>
      <th>Avg.</th>
      <th>LVB</th>
      <th>RVB</th>
      <th>MYO</th>
      <th>Avg.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>○</td>
      <td>○</td>
      <td>○</td>
      <td>○</td>
      <td>80.03</td>
      <td>68.26</td>
      <td>50.29</td>
      <td>66.19</td>
      <td>27.06</td>
      <td>22.30</td>
      <td>13.09</td>
      <td>20.82</td>
      <td>77.69</td>
      <td>56.94</td>
      <td>71.81</td>
      <td>68.81</td>
      <td>19.38</td>
      <td>22.62</td>
      <td>21.32</td>
      <td>21.11</td>
    </tr>
    <tr>
      <td>●</td>
      <td>○</td>
      <td>○</td>
      <td>○</td>
      <td>83.67</td>
      <td>70.47</td>
      <td>51.67</td>
      <td>68.60</td>
      <td>12.61</td>
      <td>12.62</td>
      <td>9.40</td>
      <td>11.54</td>
      <td>80.72</td>
      <td>53.60</td>
      <td>77.02</td>
      <td>70.45</td>
      <td>14.29</td>
      <td>28.73</td>
      <td>18.52</td>
      <td>20.51</td>
    </tr>
    <tr>
      <td>○</td>
      <td>●</td>
      <td>○</td>
      <td>○</td>
      <td>82.30</td>
      <td>69.96</td>
      <td>52.19</td>
      <td>68.15</td>
      <td>19.11</td>
      <td>18.22</td>
      <td>10.99</td>
      <td>16.11</td>
      <td>78.75</td>
      <td>57.25</td>
      <td>74.13</td>
      <td>70.04</td>
      <td>15.43</td>
      <td>26.79</td>
      <td>16.25</td>
      <td>19.49</td>
    </tr>
    <tr>
      <td>●</td>
      <td>●</td>
      <td>○</td>
      <td>○</td>
      <td>83.39</td>
      <td>70.37</td>
      <td>52.15</td>
      <td>68.64</td>
      <td>12.59</td>
      <td>13.85</td>
      <td>9.25</td>
      <td>11.90</td>
      <td>80.89</td>
      <td>57.69</td>
      <td>76.99</td>
      <td>71.86</td>
      <td>12.79</td>
      <td>25.64</td>
      <td>13.10</td>
      <td>17.18</td>
    </tr>
    <tr>
      <td>●</td>
      <td>●</td>
      <td>●</td>
      <td>○</td>
      <td>83.73</td>
      <td>71.10</td>
      <td>52.03</td>
      <td>68.95</td>
      <td>12.48</td>
      <td>12.80</td>
      <td>9.35</td>
      <td>11.54</td>
      <td>80.52</td>
      <td>59.61</td>
      <td>76.36</td>
      <td>72.16</td>
      <td>13.25</td>
      <td>21.55</td>
      <td>13.22</td>
      <td>16.01</td>
    </tr>
    <tr>
      <td>●</td>
      <td>●</td>
      <td>●</td>
      <td>●</td>
      <td><strong>83.91</strong></td>
      <td><strong>71.15</strong></td>
      <td><strong>52.77</strong></td>
      <td><strong>69.28</strong></td>
      <td><strong>11.92</strong></td>
      <td><strong>11.78</strong></td>
      <td><strong>8.42</strong></td>
      <td><strong>10.71</strong></td>
      <td><strong>81.44</strong></td>
      <td><strong>60.97</strong></td>
      <td><strong>77.44</strong></td>
      <td><strong>73.28</strong></td>
      <td><strong>11.36</strong></td>
      <td><strong>20.49</strong></td>
      <td><strong>11.64</strong></td>
      <td><strong>14.50</strong></td>
    </tr>
  </tbody>
</table>

### 1.1 Pixel- vs. Prototyp-basierte Distillation

- **Pixelweise Selbstdistillation** ($\ell_{\text{pixel}}$):  
  Hier vergleiche ich die **Logits** des **multimodalen Modells** und des **unimodalen** Teilmodells. Dies glättet die lokalen Klassifikationsgrenzen und verbessert die Dice-Werte merklich (vgl. Zeile „●○○○“ gegenüber „○○○○“).  
- **Prototyp-basierte Selbstdistillation** ($\ell_{\text{proto}}$):  
  Mithilfe prototypischer Klassenrepräsentationen tausche ich **globale** semantische Informationen zwischen den multi- und unimodalen Pfaden aus. Dies steigert unter anderem die Genauigkeit der Segmentierungsgrenzen und reduziert die HD (Hausdorff-Distanz).

### 1.2 Präferenzbasierte Rebalancierung

- **Maske** $\omega$ (Spalte "ω"):  
  Entscheide, ob eine Modalität bei negativer relativer Präferenz ($RP^m_n$) zusätzlich verstärkt wird. Das heißt: **vernachlässigte** Modalitäten werden gezielt gefördert.  
- **Gradientengewichteter Koeffizient** $E$ (Spalte "E"):  
  Dieser wird dynamisch an den Mittelwert der relativen Präferenzen angepasst. Modalitäten mit besonders hoher Fehlrate oder geringem Lerneffekt erhalten dadurch eine höhere Gewichtung im Training.

Aus der Tabelle erkenne ich, dass erst **mit allen Komponenten** (Zeile „●●●●“) die besten Ergebnisse für alle Metriken (DICE und HD) erzielt werden. Insbesondere steigern sich die **durchschnittlichen Dice-Werte** im Vergleich zur Baseline (erster Eintrag, „○○○○“) deutlich, und die Hausdorff-Distanz sinkt spürbar.

---

## 2. Vergleich verschiedener Distanzmethoden

Als Nächstes untersuche ich in einer separaten Ablation, wie sich unterschiedliche Distanzbegriffe oder Loss-Funktionen auf die **Distillation** auswirken. Dafür liegen folgende Einträge vor:  

- **Keine**: Keine zusätzliche Distanzbestrafung (nur Baseline).  
- **Dice Loss**: Statt prototypischer Abgleichung wird ein Würfelkoeffizient-basiertes Matching genutzt.  
- **KL Loss**: Kullback-Leibler-Divergenz zwischen Lehrer- und Schülerlogits.  
- **Proto Loss**: Prototyp-basierte Distanz ohne explizite L2-Strafterm-Variante.  
- **L2-Proto-Distance**: Meine finale Implementierung der **L2-basierten Prototyp-Distanz**.

<table>
  <caption>Vergleich der Distanzmethoden</caption>
  <thead>
    <tr>
      <th rowspan="2">Distanz</th>
      <th colspan="4">Dice [%]</th>
      <th colspan="4">HD [mm]</th>
    </tr>
    <tr>
      <th>WT</th>
      <th>TC</th>
      <th>ET</th>
      <th>Avg.</th>
      <th>WT</th>
      <th>TC</th>
      <th>ET</th>
      <th>Avg.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Keine</td>
      <td>83.39</td>
      <td>70.37</td>
      <td>52.15</td>
      <td>68.64</td>
      <td>12.59</td>
      <td>13.85</td>
      <td>9.25</td>
      <td>11.90</td>
    </tr>
    <tr>
      <td>Dice Loss</td>
      <td>83.37</td>
      <td>70.53</td>
      <td>52.12</td>
      <td>68.67</td>
      <td>13.22</td>
      <td>12.42</td>
      <td>8.72</td>
      <td>11.45</td>
    </tr>
    <tr>
      <td>KL Loss</td>
      <td>83.56</td>
      <td>70.58</td>
      <td>51.89</td>
      <td>68.68</td>
      <td>12.97</td>
      <td>12.21</td>
      <td>8.85</td>
      <td>11.34</td>
    </tr>
    <tr>
      <td>Proto Loss</td>
      <td>83.55</td>
      <td>71.31</td>
      <td>52.44</td>
      <td>69.10</td>
      <td>13.69</td>
      <td>12.78</td>
      <td>8.14</td>
      <td>11.54</td>
    </tr>
    <tr>
      <td>L2-Proto-Distance</td>
      <td>83.91</td>
      <td>71.15</td>
      <td>52.77</td>
      <td>69.28</td>
      <td>11.92</td>
      <td>11.78</td>
      <td>8.42</td>
      <td>10.71</td>
    </tr>
  </tbody>
</table>

### 2.1 Analyse

- **„Keine“** (Baseline):  
  Liefert bereits solide Dice-Werte, zeigt jedoch eine **höhere** mittlere HD (11.90).  
- **Dice Loss** oder **KL Loss**:  
  Beide erreichen ähnliche Werte, wobei der Dice Loss hier (68.67 avg. Dice) minimal besser abschneidet als KL bei ET, aber die HD ist schwankend.  
- **Proto Loss**:  
  Zeigt eine Steigerung der Dice-Werte (besonders in TC: 71.31). Allerdings bleibt die **HD** in WT und TC etwas höher.  
- **L2-Proto-Distance**:  
  Dieses Verfahren erzielt den besten Gesamtwert (69.28 avg. Dice und 10.71 avg. HD). Die Kombination aus lokalem und globalem Matching (Prototypen + L2) führt also zu einer stabileren Repräsentation.  

Mit anderen Worten: Die prototypische Repräsentation vergleicht Klassenmittelwerte und Pixel-Features im **Feature-Raum** und nutzt hierbei eine L2-Norm, um Abweichungen zu minimieren. So werden **inter-** und **intra-klassenbezogene** Unterschiede deutlicher regularisiert.

---

## 3. Einfluss des Hyperparameters λ

In einer weiteren Ablation betrachte ich den Einfluss von $\lambda$ (Lambda). Dieser Parameter steuert, **wie stark** sich die präferenzbasierte Rebalancierung (insbesondere beim Gradientenabstieg) auswirkt.

<table>
  <caption>Hyperparameter λ – Auswirkung auf DICE und HD</caption>
  <thead>
    <tr>
      <th rowspan="2">λ</th>
      <th colspan="4">DICE [%]</th>
      <th colspan="4">HD [mm]</th>
    </tr>
    <tr>
      <th>WT</th>
      <th>TC</th>
      <th>ET</th>
      <th>Avg.</th>
      <th>WT</th>
      <th>TC</th>
      <th>ET</th>
      <th>Avg.</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>83.01</td>
      <td>69.12</td>
      <td>51.82</td>
      <td>67.98</td>
      <td>14.5</td>
      <td>13.03</td>
      <td>8.73</td>
      <td>12.09</td>
    </tr>
    <tr>
      <td>2</td>
      <td>83.51</td>
      <td>70.63</td>
      <td>51.91</td>
      <td>68.68</td>
      <td>12.22</td>
      <td>12.17</td>
      <td>8.07</td>
      <td>10.82</td>
    </tr>
    <tr>
      <td>3</td>
      <td>83.66</td>
      <td>70.99</td>
      <td>52.46</td>
      <td>69.04</td>
      <td>11.8</td>
      <td>10.87</td>
      <td>8.01</td>
      <td>10.23</td>
    </tr>
    <tr>
      <td>4</td>
      <td>83.91</td>
      <td>71.15</td>
      <td>52.77</td>
      <td>69.28</td>
      <td>12.59</td>
      <td>13.85</td>
      <td>8.42</td>
      <td>11.62</td>
    </tr>
    <tr>
      <td>5</td>
      <td>82.73</td>
      <td>69.8</td>
      <td>52.43</td>
      <td>68.32</td>
      <td>14.06</td>
      <td>12.95</td>
      <td>9</td>
      <td>12</td>
    </tr>
  </tbody>
</table>

### 3.1 Interpretation

- **Geringe Werte (λ = 1, 2)**:  
  Ich erkenne, dass bei zu kleinem $\lambda$ die Rebalancierung eher schwach ausfällt, wodurch besonders seltene Modalitäten nicht genügend verstärkt werden. Dies zeigt sich in leicht schlechteren Dice-Werten (67.98 bis 68.68) und höheren HDs (zwischen 10.82 und 12.09).  
- **Optimale Werte (λ = 3, 4)**:  
  Hier sehe ich **Steigerungen** im Dice (bis zu 69.28) bei akzeptabler HD. Besonders $\lambda=4$ liefert einen guten Kompromiss zwischen DICE und HD.  
- **Hohe Werte (λ = 5)**:  
  Die Lernraten-Anpassung kann zu stark werden, was Overcompensation bewirkt. Die Dice-Werte fallen wieder (68.32), und die HD steigt.

Daraus lasse ich den Schluss zu, dass $\lambda$ – abhängig vom Datensatz, der Modalitätsverteilung und weiteren Hyperparametern – gut **abgestimmt** werden muss, um die **Effekte der Rebalancierung** weder zu unterschätzen noch zu übersteuern.

---

## 4. Fazit der Ablationen

1. **Komponenten-Betrachtung:**  
   - Die Kombination aus pixelweiser und prototypbasierter Distillation führt zu einer deutlich verbesserten Segmentierung.  
   - Die Einführung der präferenzbasierten Rebalancierung ($\omega$ & $E$) ist besonders relevant, wenn einige Modalitäten hohe Fehlraten aufweisen oder seltener verfügbar sind.  

2. **Distanzmethoden:**  
   - Ein prototypbasierter Abgleich (L2-Proto) schlägt herkömmliche Verfahren wie Dice- oder KL-basiertes Matching in den meisten Metriken.  
   - Global-semantische Informationen (Prototypen) ergänzen lokale Pixelinformationen ideal, was in einer höheren Robustheit gegenüber unbalancierten Szenarien resultiert.

3. **Hyperparameter λ:**  
   - Die Rebalancierungsintensität muss **weder zu schwach noch zu stark** ausfallen. Im Bereich von $\lambda = 3$ oder 4 erziele ich typischerweise die besten Ergebnisse.  
   - Eine zu starke Gewichtung kann zu Overcompensation führen, während eine zu geringe Gewichtung kaum etwas an der Modalitätsimbalance ändert.

**Gesamtfazit:**  
Die Ablationen demonstrieren die Wirksamkeit der in der **Methodik** (PRISM) eingeführten Module: Die Multi-Uni Selbstdistillation bringt eine engere Kopplung zwischen mono- und multimodalen Repräsentationen, während das präferenzbasierte Lernraten-Management sicherstellt, dass alle Modalitäten – auch seltene oder schwache – angemessen in das Endmodell einfließen. Die passenden Distanzfunktionen und Hyperparameter sind dabei entscheidend, um ein **ausgewogenes** und zugleich **leistungsstarkes** Training zu erreichen.