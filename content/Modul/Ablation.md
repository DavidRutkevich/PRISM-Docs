---

title: "PRISM – Ablation"  
description: "Eine Ablation der PRISM-Methode sowie kurze Kommentare und Anmerkungen."  
date: 2025-02-05  
math: true

---

# Ablationsstudie zu PRISM

In diesem Abschnitt wird eine Ablationsstudie vorgestellt, mit der die Wirkung einzelner Komponenten sowie verschiedener Distanz- und Hyperparameter-Einstellungen von PRISM genauer untersucht wird. Die Analyse dient dazu, nachzuvollziehen, welche Teilschritte der Methodik den stärksten Einfluss auf das Endergebnis haben und wie das Zusammenspiel der einzelnen Module sich auswirkt.

---

## 1. Komponenten-Ablation

Zunächst werden die einzelnen Bausteine von PRISM betrachtet. Dazu wurde die Tabelle **Komponenten Ablation auf BraTS2020 und MyoPS2020** erstellt. Die einzelnen Zeilen differenzieren danach, ob der **pixelweise Distillation-Term** \(\ell_{\text{pixel}}\), der **prototyp-basierte Term** \(\ell_{\text{proto}}\), die **Rebalancierungsmaske** \(\omega\) und/oder der **gradientengewichtete Koeffizient** \(E\) aktiviert werden. Die Tabelle zeigt, inwiefern jeder dieser Bausteine zu einer Verbesserung der Ergebnisse beiträgt.

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

- **Pixelweise Selbstdistillation** (\(\ell_{\text{pixel}}\)):  
  Im Rahmen der pixelweisen Distillation werden die Logits des multimodalen Modells mit denen des unimodalen Teilmodells verglichen. Durch diese Herangehensweise werden die lokalen Klassifikationsgrenzen geglättet, was sich in einer signifikanten Steigerung der Dice-Werte widerspiegelt (siehe Vergleich „●○○○“ gegenüber „○○○○“).

- **Prototyp-basierte Selbstdistillation** (\(\ell_{\text{proto}}\)):  
  Mittels prototypischer Klassenrepräsentationen werden globale semantische Informationen zwischen dem multimodalen und dem unimodalen Pfad ausgetauscht. Dies führt zu einer Verbesserung der Genauigkeit der Segmentierungsgrenzen sowie zu einer Reduktion der Hausdorff-Distanz (HD).

---

## 2. Vergleich verschiedener Distanzmethoden

Im Folgenden wird der Einfluss unterschiedlicher Distanzbegriffe bzw. Loss-Funktionen auf den Distillationsprozess untersucht. Die verwendeten Methoden umfassen:

- **Keine**: Es wird keine zusätzliche Distanzbestrafung angewendet (nur die Baseline).  
- **Dice Loss**: Ein Loss, der auf dem Würfelkoeffizienten basiert, wird zur Matching-Berechnung eingesetzt.  
- **KL Loss**: Es erfolgt ein Vergleich der Wahrscheinlichkeitsverteilungen von Lehrer- und Schülerlogits mittels Kullback-Leibler-Divergenz.  
- **Proto Loss**: Es wird ein prototypbasierter Abgleich durchgeführt, jedoch ohne die explizite Verwendung eines L2-Strafterms.  
- **L2-Proto-Distance**: Die finale Implementierung verwendet eine L2-Norm zur Berechnung der Differenz zwischen den prototypischen Repräsentationen.

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

- **„Keine“ (Baseline):**  
  Erzielte solide Dice-Werte, jedoch wurde eine höhere mittlere HD (11.90 mm) beobachtet.

- **Dice Loss / KL Loss:**  
  Beide Methoden erzielen vergleichbare Ergebnisse; der Dice Loss führt zu einem leichten Vorteil bei den durchschnittlichen Dice-Werten, während die HD-Werte ähnlich bleiben.

- **Proto Loss:**  
  Eine Erhöhung der Dice-Werte, insbesondere im TC-Bereich, wird festgestellt, wobei die HD in WT und TC etwas höher bleiben.

- **L2-Proto-Distance:**  
  Diese Methode erreicht den besten Gesamtwert mit einem durchschnittlichen Dice von 69.28 % und einer HD von 10.71 mm. Die Kombination aus lokalem und globalem Matching unter Verwendung der L2-Norm führt zu einer stabileren Repräsentation.

---

## 3. Einfluss des Hyperparameters \(\lambda\)

Der Einfluss des Hyperparameters \(\lambda\), der die Stärke der präferenzbasierten Rebalancierung steuert, wird im Folgenden untersucht.

<table>
  <caption>Hyperparameter \(\lambda\) – Auswirkung auf Dice und HD</caption>
  <thead>
    <tr>
      <th rowspan="2">\(\lambda\)</th>
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

- **Niedrige Werte (\(\lambda = 1, 2\)):**  
  Bei zu kleinen \(\lambda\)-Werten fällt die Rebalancierung zu schwach aus, wodurch seltene Modalitäten nicht ausreichend verstärkt werden. Dies führt zu etwas niedrigeren Dice-Werten (zwischen 67.98 und 68.68) und höheren HDs (10.82 bis 12.09 mm).

- **Optimale Werte (\(\lambda = 3, 4\)):**  
  Hier wird eine Steigerung der Dice-Werte (bis zu 69.28) bei akzeptablen HD-Werten beobachtet. Insbesondere \(\lambda=4\) liefert einen guten Kompromiss zwischen den Metriken.

- **Hohe Werte (\(\lambda = 5\)):**  
  Eine zu starke Anpassung führt zu einer Overcompensation, wodurch die Dice-Werte wieder sinken (auf 68.32) und die HD ansteigen.

Aus diesen Ergebnissen folgt, dass der Hyperparameter \(\lambda\) – abhängig vom Datensatz, der Modalitätsverteilung und weiteren Parametern – sorgfältig abgestimmt werden muss, um die Effekte der Rebalancierung weder zu unter- noch zu übersteuern.

---

## 4. Fazit der Ablationen

1. **Komponenten-Betrachtung:**  
   Die Kombination aus pixelweiser und prototyp-basierter Distillation führt zu einer signifikanten Verbesserung der Segmentierungsergebnisse. Die Integration der präferenzbasierten Rebalancierung (durch \(\omega\) und \(E\)) ist insbesondere bei Modalitäten mit hohen Fehlraten von großer Bedeutung.

2. **Distanzmethoden:**  
   Der prototypbasierte Abgleich mittels L2-Norm (L2-Proto-Distance) übertrifft herkömmliche Ansätze wie Dice- oder KL-basierte Matching-Methoden in den meisten Metriken. Die Kombination von global-semantischen Informationen mit lokalen Pixelinformationen führt zu einer erhöhten Robustheit gegenüber unbalancierten Szenarien.

3. **Hyperparameter \(\lambda\):**  
   Die Intensität der Rebalancierung muss weder zu schwach noch zu stark sein. Werte im Bereich von \(\lambda = 3\) oder 4 erzielen typischerweise die besten Ergebnisse, während zu hohe Werte zu einer Überkompensation führen.

**Gesamtfazit:**  
Die durchgeführten Ablationen bestätigen die Wirksamkeit der in der Methodik eingeführten Module. Die Multi-Uni Selbstdistillation fördert eine engere Kopplung zwischen mono- und multimodalen Repräsentationen, während das präferenzbasierte Lernraten-Management sicherstellt, dass alle Modalitäten – auch seltene oder schwache – adäquat berücksichtigt werden. Die geeignete Wahl der Distanzfunktionen und Hyperparameter ist dabei entscheidend, um ein ausgewogenes und leistungsstarkes Training zu erreichen.

---

## 5. Zusammenfassung

Die Ablationsstudie zeigt, dass PRISM durch die Kombination von pixelweiser und prototyp-basierter Selbstdistillation sowie durch eine gezielte, präferenzbasierte Rebalancierung eine signifikante Verbesserung der Segmentierungsleistung erzielt. Die ausgewählten Distanzmethoden und Hyperparameter tragen maßgeblich dazu bei, dass auch in Szenarien mit unvollständigen und unbalancierten Modalitäten stabile und präzise Ergebnisse erzielt werden können.