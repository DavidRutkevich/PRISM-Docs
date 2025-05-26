---
title: "Ablation"
description: "Quantitative und qualitative Auswertung von PRISMS"
date: 2025-02-05
math: true
---


Um den Beitrag einzelner Bausteine zu verstehen, wurde eine schrittweise Ablation durchgeführt:

* **Baseline (nur Encoder+Decoder, keine Regularisierung):**
  Die Segmentierung ist in allen drei Klassen (WT, TC, ET) deutlich zu schwach – ein Decoder ohne Ausgleichsmechanismen neigt dazu, nur die dominanten Modalitäten zu nutzen.

* **+Geteilter Decoder (Reg):**
  Der gemeinsame Decoder erzwingt ein modalitätsunabhängiges Feature‑Learning. Dadurch sinkt der Bias gegenüber stark vertretenen Modalitäten und die Gesamtleistung steigt moderat.

* **+Adaptive Fusion Transformer (AFT):**
  Mit dem AFT erscheinen erstmals deutliche Sprünge, besonders in der schwierigen ET‑Klasse. Damit wird die Wirksamkeit einer globalen, maskierten Attention auf fehlende Modalitäten empirisch bestätigt.

* **+SRA bzw. KFT:**
  Sowohl die Spatial Relevance Attention (SRA) als auch der Kanalbezogene Fusion‑Transformer (KFT) verbessern die Balance der Feature‑Fusion, ohne andere Modalitäten nennenswert zu beeinträchtigen.

* **Vollausbau:**
  Werden alle Komponenten kombiniert, ergibt sich eine mittlere Dice‑Steigerung von **2,49%/3,53%/6,99%** (WT/TC/ET) gegenüber der reinen AFT‑Baseline.

### KFT‑Einsatz in Skip‑Connections

PRISMS wendet die räumliche Gewichtung (SRA) auf allen Skip‑Connections an. KFT‑Module werden dagegen nur in den **unteren beiden** Ebenen platziert:

* Eine gezielte Ablation (Tab.2) zeigt, dass bereits **ein** KFT‑Layer in der tiefsten Skip‑Connection den DSC im Schnitt um **0,82%** hebt.
* Werden KFTs in die beiden tiefsten Ebenen eingefügt, steigt die mittlere Verbesserung auf **0,35%** gegenüber einem Einzel‑KFT.
* Eine aggressive Ausweitung auf alle Skip‑Connections verschlechtert die Leistung – hochaufgelöste Features der oberen Ebenen sind weniger diskriminativ, was eine kanalweise Neugewichtung erschwert.




<table>
    <caption>Tabelle 1 Komponenten Ablation</caption>
  <thead>
    <tr>
      <th colspan="4"><strong>Komponenten</strong></th>
      <th colspan="4"><strong>∅DSC(%)</strong></th>
      <th colspan="2"><strong>Komplexität</strong></th>
    </tr>
    <tr>
      <th><strong>Reg</strong></th>
      <th><strong>AFT</strong></th>
      <th><strong>KFT</strong></th>
      <th><strong>SRA</strong></th>
      <th><strong>WT</strong></th>
      <th><strong>TC</strong></th>
      <th><strong>ET</strong></th>
      <th><strong>∅</strong></th>
      <th><strong>Params</strong></th>
      <th><strong>GFLOPs</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>○</td><td>○</td><td>○</td><td>○</td>
      <td>85.37</td><td>75.67</td><td>59.78</td><td>73.61</td>
      <td>29.00</td><td>197.818</td>
    </tr>
    <tr>
      <td>●</td><td>○</td><td>○</td><td>○</td>
      <td>86.16</td><td>77.68</td><td>61.85</td><td>75.23</td>
      <td>+0.00</td><td>+0.000</td>
    </tr>
    <tr>
      <td>●</td><td>●</td><td>○</td><td>○</td>
      <td>87.15</td><td>78.50</td><td>64.24</td><td>76.63</td>
      <td>+10.44</td><td>+7.851</td>
    </tr>
    <tr>
      <td>●</td><td>●</td><td>●</td><td>○</td>
      <td>87.38</td><td>78.63</td><td>65.53</td><td>77.18</td>
      <td>+13.87</td><td>+11.750</td>
    </tr>
    <tr>
      <td>●</td><td>●</td><td>○</td><td>●</td>
      <td>87.45</td><td>78.95</td><td>64.55</td><td>76.98</td>
      <td>+10.44</td><td>+7.856</td>
    </tr>
    <tr>
      <td>●</td><td>●</td><td>●</td><td>●</td>
      <td>87.86</td><td>79.20</td><td>66.77</td><td>77.94</td>
      <td>+13.87</td><td>+11.754</td>
    </tr>
  </tbody>
</table>

<table>
  <caption>Tabelle 2 KFT-Abaltion: Die Stage gibt an, über wie viele Skip‑Connections der Encoder mit dem KFT verwendet wird.</caption>
  <thead>
    <tr>
      <th rowspan="2"><strong>Stage</strong></th>
      <th rowspan="2"><strong>SRA</strong></th>
      <th colspan="4"><strong>∅DSC(%)</strong></th>
      <th colspan="2"><strong>Komplexität</strong></th>
    </tr>
    <tr>
      <th><strong>WT</strong></th>
      <th><strong>TC</strong></th>
      <th><strong>ET</strong></th>
      <th><strong>∅</strong></th>
      <th><strong>Params</strong></th>
      <th><strong>GFLOPs</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>○</td>
      <td>87.15</td>
      <td>78.50</td>
      <td>64.24</td>
      <td>76.63</td>
      <td>39.44</td>
      <td>205.670</td>
    </tr>
    <tr>
      <td>0</td>
      <td>●</td>
      <td>87.45</td>
      <td>78.95</td>
      <td>64.55</td>
      <td>76.98</td>
      <td>+0.00</td>
      <td>+0.005</td>
    </tr>
    <tr>
      <td>1</td>
      <td>●</td>
      <td>87.62</td>
      <td>78.88</td>
      <td>65.86</td>
      <td>77.45</td>
      <td>+2.63</td>
      <td>+1.151</td>
    </tr>
    <tr>
      <td>2</td>
      <td>●</td>
      <td>87.86</td>
      <td>79.20</td>
      <td>66.77</td>
      <td>77.94</td>
      <td>+3.43</td>
      <td>+3.903</td>
    </tr>
    <tr>
      <td>3</td>
      <td>●</td>
      <td>87.60</td>
      <td>78.99</td>
      <td>65.39</td>
      <td>77.33</td>
      <td>+3.69</td>
      <td>+11.241</td>
    </tr>
    <tr>
      <td>4</td>
      <td>●</td>
      <td>87.39</td>
      <td>78.54</td>
      <td>66.11</td>
      <td>77.35</td>
      <td>+3.79</td>
      <td>+33.253</td>
    </tr>
    <tr>
      <td>5</td>
      <td>●</td>
      <td>87.32</td>
      <td>78.72</td>
      <td>66.14</td>
      <td>77.39</td>
      <td>+3.83</td>
      <td>+106.621</td>
    </tr>
  </tbody>
</table>

## Hyperparameter

Zur Bewertung des **Adaptive Fusion Transformer (AFT)** wurde eine Ablationsstudie mit unterschiedlicher Layer‑Tiefe durchgeführt (Tab.3). Bereits ein einziger AFT‑Layer verbessert die Baseline deutlich – im Mittel um **≈3,1 Dice‑Prozentpunkte**. Werden zusätzliche AFT‑Layer hinzugefügt, steigt die Leistung dagegen kaum weiter an. Der Grund liegt in der datenintensiven Natur von Transformer‑Schichten: Ab einer gewissen Tiefe tendieren die Self‑Attention‑Matrizen zu „uniformen“ Verteilungen[1], sodass zusätzliche Layer nur noch begrenzten Mehrwert bieten.

Ein zweiter zentraler Hyperparameter ist die Anzahl der **Kanalbezogenen Fusionstransformer (KFT)**. Tab.4 zeigt, dass ein einzelner KFT‑Layer bereits merklich hilft, indem er Redundanzen entlang der Kanäle reduziert und kompaktere, modalitätsspezifische Features lernt. Fügt man jedoch zu viele KFT‑Layer hinzu, kippt der Effekt: Die fortlaufende Re‑Gewichtung entlang der Kanaldimension dämpft zunehmend die räumlichen Informationen und führt zu Leistungseinbußen.

Interessant ist zudem die Modellkomplexität: Ein einzelnes KFT‑Modul ersetzt die konventionellen Convolution‑Skip‑Connections im mmFormer‑Backbone und **reduziert** dadurch die Parameterzahl – trotz seines Transformer‑Anteils. Damit bietet die Kombination „1×AFT+1×KFT“ einen guten Kompromiss aus Genauigkeit und Effizienz.

<table>
  <caption>Tabelle 3 Anzahl AFT Layer</caption>
  <thead>
    <tr>
      <th><strong>L₁</strong></th>
      <th><strong>WT</strong></th>
      <th><strong>TC</strong></th>
      <th><strong>ET</strong></th>
      <th><strong>∅DSC(%)</strong></th>
      <th><strong>Params</strong></th>
      <th><strong>GFLOPs</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>86.16</td>
      <td>77.68</td>
      <td>61.85</td>
      <td>75.23</td>
      <td>29.0</td>
      <td>197.78</td>
    </tr>
    <tr>
      <td>1</td>
      <td>87.69</td>
      <td>79.11</td>
      <td>65.35</td>
      <td>77.38</td>
      <td>+9.14</td>
      <td>+5.851</td>
    </tr>
    <tr>
      <td>2</td>
      <td>87.77</td>
      <td>79.20</td>
      <td>66.34</td>
      <td>77.77</td>
      <td>+11.50</td>
      <td>+8.803</td>
    </tr>
    <tr>
      <td>3</td>
      <td>87.86</td>
      <td>79.20</td>
      <td>66.77</td>
      <td>77.94</td>
      <td>+13.87</td>
      <td>+11.754</td>
    </tr>
  </tbody>
</table>

<table>
  <caption>Tabelle 4: Anzahl KFT layer</caption>
  <thead>
    <tr>
      <th><strong>L₂</strong></th>
      <th><strong>WT</strong></th>
      <th><strong>TC</strong></th>
      <th><strong>ET</strong></th>
      <th><strong>∅DSC(%)</strong></th>
      <th><strong>Params</strong></th>
      <th><strong>GFLOPs</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>87.45</td>
      <td>78.95</td>
      <td>64.55</td>
      <td>76.98</td>
      <td>39.44</td>
      <td>205.674</td>
    </tr>
    <tr>
      <td>1</td>
      <td>87.58</td>
      <td>79.05</td>
      <td>65.93</td>
      <td>77.52</td>
      <td>-0,01</td>
      <td>+0.076</td>
    </tr>
    <tr>
      <td>2</td>
      <td>87.86</td>
      <td>79.20</td>
      <td>66.77</td>
      <td>77.94</td>
      <td>+3.43</td>
      <td>+3.899</td>
    </tr>
    <tr>
      <td>3</td>
      <td>87.68</td>
      <td>79.20</td>
      <td>66.10</td>
      <td>77.66</td>
      <td>+6.86</td>
      <td>+7.721</td>
    </tr>
    <tr>
      <td>4</td>
      <td>87.46</td>
      <td>79.24</td>
      <td>65.65</td>
      <td>77.51</td>
      <td>+10.30</td>
      <td>+11.544</td>
    </tr>
  </tbody>
</table>


<table>
  <caption></caption>
  <thead>
    <tr><th>Typ</th><th>Methode</th><th>BraTS2018 (HD)</th><th>BraTS2020 (HD)</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="6">WT</td><td>HeMIS</td><td>26.72</td><td>27.32</td></tr>
    <tr><td>U‑HVED</td><td>25.10</td><td>28.00</td></tr>
    <tr><td>RobustMSeg</td><td>11.37</td><td>13.05</td></tr>
    <tr><td>RFNet</td><td>7.24</td><td>8.42</td></tr>
    <tr><td>mmFormer</td><td>7.30</td><td>7.71</td></tr>
    <tr><td><strong>PRISMS</strong></td><td><strong>6.38</strong></td><td><strong>5.68</strong></td></tr>
    <tr><td rowspan="6">TC</td><td>HeMIS</td><td>27.99</td><td>25.27</td></tr>
    <tr><td>U‑HVED</td><td>25.18</td><td>23.77</td></tr>
    <tr><td>RobustMSeg</td><td>11.74</td><td>12.70</td></tr>
    <tr><td>RFNet</td><td>7.24</td><td>8.42</td></tr>
    <tr><td>mmFormer</td><td>7.30</td><td>7.71</td></tr>
    <tr><td><strong>PRISMS</strong></td><td><strong>6.60</strong></td><td><strong>6.49</strong></td></tr>
    <tr><td rowspan="6">ET</td><td>HeMIS</td><td>15.48</td><td>16.70</td></tr>
    <tr><td>U‑HVED</td><td>13.48</td><td>14.86</td></tr>
    <tr><td>RobustMSeg</td><td>8.28</td><td>9.04</td></tr>
    <tr><td>RFNet</td><td>7.24</td><td>8.42</td></tr>
    <tr><td>mmFormer</td><td>7.30</td><td>7.71</td></tr>
    <tr><td><strong>PRISMS</strong></td><td><strong>5.95</strong></td><td><strong>5.02</strong></td></tr>
  </tbody>
</table>

## Vergleich mit distillations­basierten Multi‑Model‑Ansätzen

Für den direkten Vergleich wurden bewusst ausschließlich **Single‑Model‑Methoden** des aktuellen State of the Art herangezogen. Distillations­basierte Verfahren wie **ACN** [2] und **SMU‑Net** [3] trainieren dagegen für jede Modalitäts­konstellation ein eigenes Modell (insgesamt 15 Modelle). Ihre Ergebnisse sind in Tab. 5 zusammengefasst.

Ein wichtiger Unterschied liegt in der Eingabe­auflösung: ACN und SMU‑Net verarbeiten Volumina mit \(160 \times 192 \times 128\) Voxel, während **PRISMS** aus Hardware‑Gründen mit \(80 \times 80 \times 80\) Voxel operiert. Trotz dieses handfesten Nachteils erzielt PRISMS:

* **38 / 45** bessere Ergebnisse gegenüber ACN
  * durchschnittliche Dice‑Steigerung: **+1.0 % (WT)**, **+4.5 % (TC)**, **+2.1 % (ET)**
* **38 / 45** bessere Ergebnisse gegenüber SMU‑Net
  * durchschnittliche Dice‑Steigerung: **+0.4 % (WT)**, **+3.4 % (TC)**, **+1.0 % (ET)**

Diese Resultate unterstreichen die Effizienz der präferenz­gesteuerten Selbstdistillation von PRISMS selbst bei halbierter Eingabe­größe. Da höhere Auflösungen erfahrungsgemäß zusätzliche Detail­informationen liefern, ist bei einer Skalierung auf größere Volumina eine weitere Leistungs­steigerung zu erwarten.


<table>
  <caption>Tabelle 5: KD-Verfahren</caption>
  <thead>
    <tr>
      <th rowspan="4">Typ</th>
      <th>Flair</th>
      <th>○</th><th>○</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th>
      <th>○</th><th>●</th><th>●</th><th>●</th><th>●</th><th>●</th><th>○</th><th>●</th>
      <th rowspan="4">Avg.</th>
    </tr>
    <tr>
      <th>T1</th>
      <th>○</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th>
      <th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>●</th><th>●</th>
    </tr>
    <tr>
      <th>T1ce</th>
      <th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th>
      <th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>●</th><th>●</th><th>●</th>
    </tr>
    <tr>
      <th>T2</th>
      <th>●</th><th>○</th><th>○</th><th>○</th><th>●</th><th>○</th><th>○</th>
      <th>●</th><th>●</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th>
    </tr>
  </thead>
  <tbody>
    <tr><td rowspan="3">WT</td><td>ACN</td><td>85.4</td><td>79.8</td><td>78.7</td><td>87.3</td><td>84.9</td><td>79.6</td><td>86.0</td><td>84.4</td><td>86.9</td><td>87.8</td><td>88.4</td><td>87.4</td><td>87.2</td><td>86.6</td><td>89.1</td><td><strong>85.30</strong></td></tr>
    <tr><td>SMU‑Net</td><td>85.7</td><td>80.3</td><td>78.6</td><td>87.5</td><td>86.1</td><td>80.3</td><td>87.3</td><td>85.6</td><td>87.9</td><td>88.4</td><td>88.2</td><td>88.3</td><td>88.2</td><td>86.5</td><td>88.9</td><td><strong>85.85</strong></td></tr>
    <tr><td>PRISMS</td><td>83.5</td><td>78.9</td><td>78.0</td><td>87.7</td><td>86.3</td><td>82.5</td><td>88.9</td><td>86.0</td><td>88.7</td><td>89.4</td><td>89.5</td><td>89.2</td><td>89.6</td><td>87.1</td><td>89.6</td><td><strong>86.33</strong></td></tr>
    <tr><td rowspan="3">TC</td><td>ACN</td><td>66.8</td><td>83.3</td><td>70.9</td><td>66.4</td><td>83.2</td><td>83.9</td><td>70.4</td><td>72.8</td><td>70.7</td><td>82.9</td><td>83.3</td><td>67.7</td><td>82.9</td><td>83.2</td><td>84.8</td><td><strong>76.88</strong></td></tr>
    <tr><td>SMU‑Net</td><td>67.2</td><td>84.1</td><td>69.5</td><td>71.8</td><td>85.0</td><td>84.4</td><td>71.2</td><td>73.5</td><td>71.2</td><td>84.1</td><td>84.2</td><td>67.9</td><td>82.5</td><td>84.4</td><td>87.3</td><td><strong>77.89</strong></td></tr>
    <tr><td>PRISMS</td><td>70.8</td><td>87.7</td><td>71.0</td><td>70.8</td><td>88.2</td><td>88.4</td><td>75.4</td><td>74.6</td><td>73.7</td><td>88.3</td><td>88.5</td><td>75.8</td><td>88.2</td><td>88.6</td><td>88.4</td><td><strong>81.23</strong></td></tr>
    <tr><td rowspan="3">ET</td><td>ACN</td><td>41.7</td><td>78.0</td><td>41.8</td><td>42.2</td><td>74.9</td><td>75.3</td><td>42.5</td><td>46.5</td><td>44.3</td><td>77.5</td><td>75.1</td><td>42.8</td><td>73.8</td><td>75.9</td><td>78.2</td><td><strong>60.70</strong></td></tr>
    <tr><td>SMU‑Net</td><td>43.1</td><td>78.3</td><td>42.8</td><td>46.1</td><td>75.7</td><td>75.1</td><td>44.0</td><td>47.7</td><td>46.0</td><td>77.3</td><td>76.2</td><td>43.1</td><td>75.4</td><td>76.2</td><td>79.3</td><td><strong>61.75</strong></td></tr>
    <tr><td>PRISMS</td><td>47.5</td><td>79.0</td><td>39.7</td><td>32.8</td><td>79.4</td><td>79.8</td><td>41.0</td><td>48.6</td><td>48.4</td><td>79.2</td><td>79.5</td><td>49.1</td><td>79.4</td><td>79.6</td><td>79.4</td><td><strong>62.83</strong></td></tr>
  </tbody>
</table>

## Modelleffizents

![Vergleich verschiedener sota Modelle](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/new_figures/Prisms%20%7C%20Framework/model_comparison.png)
| Methode                 | HeMIS | U‑HVED | RobustMSeg | mmFormer | MAML  | RFNet  | RA‑HVED | U‑Net‑MFI | PRISMS |
|-------------------------|------:|-------:|-----------:|---------:|------:|-------:|--------:|----------:|-------:|
| Parameter in Millionen  |  1.17 |   3.79 |      37.58 |    57.61 | 22.71 |   8.40 |    5.89 |     30.91 |  42.87 |
| GFLOPs                  | 77.88 | 284.37 |     848.54 |   206.83 |375.92 | 204.57 |  328.43 |    999.04 | 209.57 |
| Avg DSC [%]             | 64.53 |  66.61 |      71.85 |    74.95 |78.55  |  75.55 |   74.47 |     78.17 |  85.81 |

### Cross‑Training

Um die Generalisierungsfähigkeit von **PRISMS** zu evaluieren, wurde ein Cross‑Training‑Szenario durchgeführt: Ein ausschließlich auf **BraTS2018** trainiertes Modell wurde unverändert auf den **BraTS2020**‑Datensatz angewendet. Damit kein Daten‑Leakage entsteht, wurden alle Fälle, die bereits in BraTS2018 enthalten sind, aus BraTS2020 entfernt.

Die Ergebnisse sind in Tab. 6 dargestellt. Während alle verglichenen Modelle im domänen­fremden Setting an Genauigkeit einbüßen, bleibt der Leistungsabfall bei PRISMS deutlich geringer. Insbesondere vergrößert sich der Abstand zu den konkurrierenden Ansätzen nochmals, was die Robustheit und die bessere Übertragbarkeit der innerhalb von PRISMS gelernten Cross‑Modal‑Repräsentationen unterstreicht.

<table>
  <caption>Tabelle 6: Cross-Training</caption>
  <thead>
    <tr>
      <th>Typ</th>
      <th>Methode</th>
      <th>DSC (%)</th>
      <th>p‑value</th>
      <th>HD (mm)</th>
      <th>p‑value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="6">WT</td>
      <td>HeMIS</td>
      <td>81.89</td>
      <td>&lt;0.001</td>
      <td>23.55</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>U‑HVED</td>
      <td>83.12</td>
      <td>&lt;0.001</td>
      <td>20.01</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RobustMSeg</td>
      <td>86.43</td>
      <td>&lt;0.001</td>
      <td>10.04</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RFNet</td>
      <td>88.63</td>
      <td>&lt;0.001</td>
      <td>5.90</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>mmFormer</td>
      <td>87.93</td>
      <td>&lt;0.001</td>
      <td>5.33</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td><strong>90.00</strong></td>
      <td>&ndash;</td>
      <td><strong>4.31</strong></td>
      <td>&ndash;</td>
    </tr>
    <tr>
      <td rowspan="6">TC</td>
      <td>HeMIS</td>
      <td>71.03</td>
      <td>&lt;0.001</td>
      <td>25.58</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>U‑HVED</td>
      <td>73.38</td>
      <td>&lt;0.001</td>
      <td>19.47</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RobustMSeg</td>
      <td>78.12</td>
      <td>&lt;0.001</td>
      <td>9.87</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RFNet</td>
      <td>83.05</td>
      <td>&lt;0.001</td>
      <td>5.73</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>mmFormer</td>
      <td>82.64</td>
      <td>&lt;0.001</td>
      <td>6.30</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td><strong>84.82</strong></td>
      <td>&ndash;</td>
      <td><strong>4.67</strong></td>
      <td>&ndash;</td>
    </tr>
    <tr>
      <td rowspan="6">ET</td>
      <td>HeMIS</td>
      <td>59.67</td>
      <td>&lt;0.001</td>
      <td>16.08</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>U‑HVED</td>
      <td>62.15</td>
      <td>&lt;0.001</td>
      <td>10.53</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RobustMSeg</td>
      <td>66.54</td>
      <td>&lt;0.001</td>
      <td>6.89</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RFNet</td>
      <td>72.13</td>
      <td>&lt;0.001</td>
      <td>5.10</td>
      <td>0.018</td>
    </tr>
    <tr>
      <td>mmFormer</td>
      <td>71.15</td>
      <td>&lt;0.001</td>
      <td>5.22</td>
      <td>0.002</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td><strong>73.61</strong></td>
      <td>&ndash;</td>
      <td><strong>4.28</strong></td>
      <td>&ndash;</td>
    </tr>
  </tbody>
</table>

<table>
  <caption>Tabelle 7: BraTS2021</caption>
  <thead>
    <tr>
      <th>Typ</th>
      <th>Methode</th>
      <th>DSC (%)</th>
      <th>p‑value</th>
      <th>HD (mm)</th>
      <th>p‑value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="6">WT</td>
      <td>HeMIS</td>
      <td>78.75</td>
      <td>&lt;0.001</td>
      <td>24.62</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>U‑HVED</td>
      <td>80.75</td>
      <td>&lt;0.001</td>
      <td>24.54</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RobustMSeg</td>
      <td>83.64</td>
      <td>&lt;0.001</td>
      <td>22.17</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RFNet</td>
      <td>87.16</td>
      <td>&lt;0.001</td>
      <td>9.64</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>mmFormer</td>
      <td>87.13</td>
      <td>&lt;0.001</td>
      <td>7.51</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td><strong>88.33</strong></td>
      <td>&ndash;</td>
      <td><strong>6.37</strong></td>
      <td>&ndash;</td>
    </tr>
    <tr>
      <td rowspan="6">TC</td>
      <td>HeMIS</td>
      <td>66.05</td>
      <td>&lt;0.001</td>
      <td>23.62</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>U‑HVED</td>
      <td>70.50</td>
      <td>&lt;0.001</td>
      <td>24.62</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RobustMSeg</td>
      <td>74.25</td>
      <td>&lt;0.001</td>
      <td>12.54</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RFNet</td>
      <td>80.56</td>
      <td>&lt;0.001</td>
      <td>7.50</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>mmFormer</td>
      <td>80.90</td>
      <td>&lt;0.001</td>
      <td>6.84</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td><strong>83.20</strong></td>
      <td>&ndash;</td>
      <td><strong>5.38</strong></td>
      <td>&ndash;</td>
    </tr>
    <tr>
      <td rowspan="6">ET</td>
      <td>HeMIS</td>
      <td>54.93</td>
      <td>&lt;0.001</td>
      <td>17.61</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>U‑HVED</td>
      <td>60.41</td>
      <td>&lt;0.001</td>
      <td>18.03</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RobustMSeg</td>
      <td>64.23</td>
      <td>&lt;0.001</td>
      <td>10.07</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>RFNet</td>
      <td>70.13</td>
      <td>&lt;0.001</td>
      <td>6.18</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td>mmFormer</td>
      <td>70.40</td>
      <td>&lt;0.001</td>
      <td>5.78</td>
      <td>&lt;0.001</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td><strong>73.76</strong></td>
      <td>&ndash;</td>
      <td><strong>4.88</strong></td>
      <td>&ndash;</td>
    </tr>
  </tbody>
</table>

### Quellen

[1] **Touvron, H., Cord, M., Sablayrolles, A., Synnaeve, G., & Jégou, H. (2021).**
   *Going Deeper with Image Transformers*.
   *arXiv preprint arXiv:2103.17239*. doi:[10.48550/arXiv.2103.17239](https://doi.org/10.48550/arXiv.2103.17239)
[2] https://arxiv.org/abs/2106.14591
[3] https://arxiv.org/abs/2204.02961
