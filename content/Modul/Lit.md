---

title: "Quantitativer Vergleich auf BraTS2020 und MyoPS2020"  
date: 2025-02-19  
draft: false  
math: true  
---
In diesem Artikel präsentiere ich einen quantitativen Vergleich der Segmentierungsergebnisse auf den Datensätzen **BraTS2020** und **MyoPS2020**. Ich habe die Ergebnisse mit unterschiedlichen Einstellungen ermittelt, wobei ich verschiedene Fehlraten (FR) für die Modalitäten angewandt habe. Für die Darstellung der Ergebnisse unterscheide ich zwischen zwei Szenarien:

- **VTD (vollständige Trainingsdaten):** Hier sind alle Modalitäten vorhanden (FR = (0,0,0)).
- **UTD (unvollständige Trainingsdaten):** Ich wende definierte Fehlraten an. Für BraTS2020 stehen beispielsweise die Werte \\(k = 0.2,\\, m = 0.5,\\, g = 0.8\\) zur Verfügung, während für MyoPS2020 die Werte \\(k = 0.3,\\, m = 0.5,\\, g = 0.7\\) gelten.

Die Tabelle umfasst folgende Metriken:
- **Dice [%] \\(\\uparrow\\):** Höhere Werte deuten auf eine bessere Überlappung zwischen Vorhersage und Ground Truth hin.
- **HD [mm] \\(\\downarrow\\):** Eine geringere Hausdorff-Distanz weist auf präzisere Segmentierungsgrenzen hin.

Ich stelle die Ergebnisse für verschiedene Methoden vor:
- **mmformer** (Baseline, basierend auf [Zhang et al. 2022](https://doi.org/10.1145/XXXXXX)),
- **CMAF-net** (SOTA, basierend auf [Sun et al. 2024](https://doi.org/10.1145/YYYYYY)),
- **mmformer + PRISM** (Integration von PRISM zur verbesserten Modalitätsrebalancierung).

Zur besseren visuellen Hervorhebung habe ich spezielle Farben definiert:
- **Blau:** Markiert die besten Ergebnisse für die VTD-Baseline.
- **Gelb:** Hebt die besten Ergebnisse innerhalb eines UTD-Szenarios hervor.

Nachfolgend finden Sie den LaTeX-Code der Tabelle, der diesen quantitativen Vergleich darstellt:

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
      color: #000; /* Set text to black for contrast in dark mode */
    }
    .hl-yellow {
      background-color: #FFF8e3;
      color: #000; /* Set text to black for contrast in dark mode */
    }
  }
</style>

<table style="width:100%; border-collapse: collapse; font-size: 0.9em;">
  <caption style="margin-bottom: 8px; text-align: center;">
    Quantitativer Vergleich auf BraTS2020 und MyoPS2020 mit verschiedenen Einstellungen.<br>
    FR steht für die Fehlrate der Modalitäten (Flair, T1/T1ce, T2) bei BraTS2020 und (bSSFP, LGE, T2) bei MyoPS2020. Die Werte k, m und g repräsentieren große, mittlere und kleine Fehlraten 
    (BraTS2020: k = 0.2, m = 0.5, g = 0.8; MyoPS2020: k = 0.3, m = 0.5, g = 0.7).<br>
    VTD beschreibt eine balancierte Modalitätsverteilung (FR = (0,0,0)).<br>
    T1 und T1ce werden als separate Modalitäten behandelt.<br>
    Die <span class="hl-blue" style="padding: 0 4px;">blauen</span> Zellen markieren die besten Ergebnisse der VTD-Baseline, während die <span class="hl-yellow" style="padding: 0 4px;">gelben</span> Zellen 
    die besten Ergebnisse innerhalb eines UTD-Szenarios anzeigen. Neben dem mmformer [Zhang et al. 2022] habe ich auch das CMAF-net [Sun et al. 2024] als SOTA gewählt.
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
    <!-- UTD Block: ($k$, $m$, $g$) -->
    <tr>
      <td rowspan="4" style="border: 1px solid #ccc; padding: 4px;">($k$,$m$,$g$)</td>
      <td style="border: 1px solid #ccc; padding: 4px;">Baseline</td>
      <td style="border: 1px solid #ccc; padding: 4px;">76,89</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,36</td>
      <td style="border: 1px solid #ccc; padding: 4px;">51,72</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,32</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,51</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,41</td>
      <td style="border: 1px solid #ccc; padding: 4px;">15,14</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,35</td>
      <td style="border: 1px solid #ccc; padding: 4px;">72,69</td>
      <td style="border: 1px solid #ccc; padding: 4px;">51,94</td>
      <td style="border: 1px solid #ccc; padding: 4px;">66,81</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,81</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,38</td>
      <td style="border: 1px solid #ccc; padding: 4px;">23,62</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,32</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,11</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+ModDrop</td>
      <td style="border: 1px solid #ccc; padding: 4px;">76,31</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,75</td>
      <td style="border: 1px solid #ccc; padding: 4px;">50,53</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,53</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,24</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,94</td>
      <td style="border: 1px solid #ccc; padding: 4px;">15,91</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,03</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,42</td>
      <td style="border: 1px solid #ccc; padding: 4px;">70,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,25</td>
      <td style="border: 1px solid #ccc; padding: 4px;">16,42</td>
      <td style="border: 1px solid #ccc; padding: 4px;">32,84</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,13</td>
      <td style="border: 1px solid #ccc; padding: 4px;">23,46</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+PMR</td>
      <td style="border: 1px solid #ccc; padding: 4px;">77,59</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,44</td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>52,86</strong></td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,30</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,84</td>
      <td style="border: 1px solid #ccc; padding: 4px;">23,22</td>
      <td style="border: 1px solid #ccc; padding: 4px;">16,68</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,58</td>
      <td style="border: 1px solid #ccc; padding: 4px;">73,05</td>
      <td style="border: 1px solid #ccc; padding: 4px;">52,61</td>
      <td style="border: 1px solid #ccc; padding: 4px;">69,32</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,99</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,16</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,04</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,42</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,54</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+<strong>PRISM</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>83,42</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>71,74</strong></td>
      <td style="border: 1px solid #ccc; padding: 4px;">52,78</td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>69,31</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>11,98</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>11,78</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>7,80</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>10,52</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>81,44</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>60,97</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>77,44</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>73,28</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>11,36</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>20,49</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>11,64</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>14,50</strong></td>
    </tr>
    <!-- UTD Block: ($k$, $g$, $m$) -->
    <tr>
      <td rowspan="4" style="border: 1px solid #ccc; padding: 4px;">($k$,$g$,$m$)</td>
      <td style="border: 1px solid #ccc; padding: 4px;">Baseline</td>
      <td style="border: 1px solid #ccc; padding: 4px;">76,53</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,74</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,02</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,43</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,88</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,71</td>
      <td style="border: 1px solid #ccc; padding: 4px;">16,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,10</td>
      <td style="border: 1px solid #ccc; padding: 4px;">69,25</td>
      <td style="border: 1px solid #ccc; padding: 4px;">49,82</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,79</td>
      <td style="border: 1px solid #ccc; padding: 4px;">60,29</td>
      <td style="border: 1px solid #ccc; padding: 4px;">26,18</td>
      <td style="border: 1px solid #ccc; padding: 4px;">33,92</td>
      <td style="border: 1px solid #ccc; padding: 4px;">26,86</td>
      <td style="border: 1px solid #ccc; padding: 4px;">28,99</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+ModDrop</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,54</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,10</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,64</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,09</td>
      <td style="border: 1px solid #ccc; padding: 4px;">31,98</td>
      <td style="border: 1px solid #ccc; padding: 4px;">30,12</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,05</td>
      <td style="border: 1px solid #ccc; padding: 4px;">27,72</td>
      <td style="border: 1px solid #ccc; padding: 4px;">73,57</td>
      <td style="border: 1px solid #ccc; padding: 4px;">49,93</td>
      <td style="border: 1px solid #ccc; padding: 4px;">69,83</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,44</td>
      <td style="border: 1px solid #ccc; padding: 4px;">11,91</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,52</td>
      <td style="border: 1px solid #ccc; padding: 4px;">13,12</td>
      <td style="border: 1px solid #ccc; padding: 4px;">15,52</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+PMR</td>
      <td style="border: 1px solid #ccc; padding: 4px;">77,32</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,39</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,88</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,20</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,39</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">15,76</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,59</td>
      <td style="border: 1px solid #ccc; padding: 4px;">72,67</td>
      <td style="border: 1px solid #ccc; padding: 4px;">52,38</td>
      <td style="border: 1px solid #ccc; padding: 4px;">68,98</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,68</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,84</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,12</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,68</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,55</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+<strong>PRISM</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>83,47</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>71,57</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>53,06</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>69,37</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>10,52</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>11,21</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>7,20</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>9,64</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>80,20</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>58,22</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>77,56</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>71,99</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>16,23</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>18,13</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>13,01</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>15,79</strong></td>
    </tr>
    <!-- UTD Block: ($m$, $k$, $g$) -->
    <tr>
      <td rowspan="4" style="border: 1px solid #ccc; padding: 4px;">($m$,$k$,$g$)</td>
      <td style="border: 1px solid #ccc; padding: 4px;">Baseline</td>
      <td style="border: 1px solid #ccc; padding: 4px;">76,56</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,22</td>
      <td style="border: 1px solid #ccc; padding: 4px;">47,06</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,95</td>
      <td style="border: 1px solid #ccc; padding: 4px;">26,00</td>
      <td style="border: 1px solid #ccc; padding: 4px;">27,98</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,33</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,44</td>
      <td style="border: 1px solid #ccc; padding: 4px;">72,16</td>
      <td style="border: 1px solid #ccc; padding: 4px;">49,44</td>
      <td style="border: 1px solid #ccc; padding: 4px;">71,08</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,23</td>
      <td style="border: 1px solid #ccc; padding: 4px;">16,78</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,13</td>
      <td style="border: 1px solid #ccc; padding: 4px;">25,56</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,49</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+ModDrop</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,40</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,74</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,58</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,57</td>
      <td style="border: 1px solid #ccc; padding: 4px;">32,80</td>
      <td style="border: 1px solid #ccc; padding: 4px;">34,57</td>
      <td style="border: 1px solid #ccc; padding: 4px;">25,67</td>
      <td style="border: 1px solid #ccc; padding: 4px;">31,01</td>
      <td style="border: 1px solid #ccc; padding: 4px;">74,07</td>
      <td style="border: 1px solid #ccc; padding: 4px;">50,30</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,69</td>
      <td style="border: 1px solid #ccc; padding: 4px;">66,69</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,46</td>
      <td style="border: 1px solid #ccc; padding: 4px;">23,32</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,27</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,68</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+PMR</td>
      <td style="border: 1px solid #ccc; padding: 4px;">76,73</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,29</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,76</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,93</td>
      <td style="border: 1px solid #ccc; padding: 4px;">23,62</td>
      <td style="border: 1px solid #ccc; padding: 4px;">25,27</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,75</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,55</td>
      <td style="border: 1px solid #ccc; padding: 4px;">72,49</td>
      <td style="border: 1px solid #ccc; padding: 4px;">52,62</td>
      <td style="border: 1px solid #ccc; padding: 4px;">70,90</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,34</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,57</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,96</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,02</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,52</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+<strong>PRISM</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>83,41</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>70,49</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>53,27</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>69,06</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,12</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>14,42</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>10,10</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,21</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>75,15</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>49,54</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>71,39</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>65,36</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>17,85</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,77</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>20,60</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,41</strong></td>
    </tr>
    <!-- UTD Block: ($g$, $k$, $m$) -->
    <tr>
      <td rowspan="4" style="border: 1px solid #ccc; padding: 4px;">($g$,$k$,$m$)</td>
      <td style="border: 1px solid #ccc; padding: 4px;">Baseline</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,77</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,15</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,85</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,59</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,68</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,84</td>
      <td style="border: 1px solid #ccc; padding: 4px;">15,96</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,49</td>
      <td style="border: 1px solid #ccc; padding: 4px;">72,74</td>
      <td style="border: 1px solid #ccc; padding: 4px;">47,86</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,97</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,19</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,72</td>
      <td style="border: 1px solid #ccc; padding: 4px;">29,28</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,91</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,97</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+ModDrop</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,22</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,50</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,69</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,47</td>
      <td style="border: 1px solid #ccc; padding: 4px;">30,93</td>
      <td style="border: 1px solid #ccc; padding: 4px;">30,47</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,78</td>
      <td style="border: 1px solid #ccc; padding: 4px;">28,06</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,99</td>
      <td style="border: 1px solid #ccc; padding: 4px;">48,16</td>
      <td style="border: 1px solid #ccc; padding: 4px;">70,90</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,02</td>
      <td style="border: 1px solid #ccc; padding: 4px;">16,70</td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>22,17</strong></td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,34</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,40</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+PMR</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,94</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,18</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,54</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,89</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,97</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,29</td>
      <td style="border: 1px solid #ccc; padding: 4px;">15,94</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,07</td>
      <td style="border: 1px solid #ccc; padding: 4px;">74,72</td>
      <td style="border: 1px solid #ccc; padding: 4px;">49,21</td>
      <td style="border: 1px solid #ccc; padding: 4px;">67,72</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,88</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,57</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,96</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,02</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,52</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+<strong>PRISM</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>83,41</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>70,49</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>53,27</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>69,06</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,12</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>14,42</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>10,10</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,21</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>75,15</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>49,54</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>71,39</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>65,36</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>17,85</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,77</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>20,60</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,41</strong></td>
    </tr>
    <!-- UTD Block: ($g$, $m$, $k$) -->
    <tr>
      <td rowspan="4" style="border: 1px solid #ccc; padding: 4px;">($g$,$m$,$k$)</td>
      <td style="border: 1px solid #ccc; padding: 4px;">Baseline</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,33</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,06</td>
      <td style="border: 1px solid #ccc; padding: 4px;">47,01</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,80</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,43</td>
      <td style="border: 1px solid #ccc; padding: 4px;">23,87</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,57</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,62</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,44</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">70,14</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,09</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,06</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,65</td>
      <td style="border: 1px solid #ccc; padding: 4px;">16,38</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,70</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+ModDrop</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,60</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,75</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,33</td>
      <td style="border: 1px solid #ccc; padding: 4px;">25,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">27,19</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,84</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,24</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,59</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,95</td>
      <td style="border: 1px solid #ccc; padding: 4px;">71,41</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,65</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,86</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,75</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,75</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+PMR</td>
      <td style="border: 1px solid #ccc; padding: 4px;">76,13</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,96</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,99</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,69</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,61</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,05</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,79</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,15</td>
      <td style="border: 1px solid #ccc; padding: 4px;">49,54</td>
      <td style="border: 1px solid #ccc; padding: 4px;">71,39</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,36</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,85</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,77</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,60</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,41</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+<strong>PRISM</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>83,41</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>70,49</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>53,27</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>69,06</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,12</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>14,42</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>10,10</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,21</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>75,15</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>49,54</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>71,39</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>65,36</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>17,85</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,77</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>20,60</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,41</strong></td>
    </tr>
    <!-- UTD Block: ($g$, $m$, $k$) -->
    <tr>
      <td rowspan="4" style="border: 1px solid #ccc; padding: 4px;">($g$,$m$,$k$)</td>
      <td style="border: 1px solid #ccc; padding: 4px;">Baseline</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,33</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,06</td>
      <td style="border: 1px solid #ccc; padding: 4px;">47,01</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,80</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,43</td>
      <td style="border: 1px solid #ccc; padding: 4px;">23,87</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,57</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,62</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,44</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">70,14</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,09</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,06</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,65</td>
      <td style="border: 1px solid #ccc; padding: 4px;">16,38</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,70</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+ModDrop</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,60</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,75</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,33</td>
      <td style="border: 1px solid #ccc; padding: 4px;">25,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">27,19</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,84</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,24</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,59</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,95</td>
      <td style="border: 1px solid #ccc; padding: 4px;">71,41</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,65</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,86</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,75</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,75</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+PMR</td>
      <td style="border: 1px solid #ccc; padding: 4px;">76,13</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,96</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,99</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,69</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,61</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,05</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,79</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,15</td>
      <td style="border: 1px solid #ccc; padding: 4px;">49,54</td>
      <td style="border: 1px solid #ccc; padding: 4px;">71,39</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,36</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,85</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,77</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,60</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,41</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+<strong>PRISM</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>83,41</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>70,49</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>53,27</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>69,06</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,12</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>14,42</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>10,10</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,21</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>75,15</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>49,54</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>71,39</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>65,36</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>17,85</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,77</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>20,60</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,41</strong></td>
    </tr>
    <!-- UTD Block: ($g$, $m$, $k$) -->
    <tr>
      <td rowspan="4" style="border: 1px solid #ccc; padding: 4px;">($g$,$m$,$k$)</td>
      <td style="border: 1px solid #ccc; padding: 4px;">Baseline</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,33</td>
      <td style="border: 1px solid #ccc; padding: 4px;">63,06</td>
      <td style="border: 1px solid #ccc; padding: 4px;">47,01</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,80</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,43</td>
      <td style="border: 1px solid #ccc; padding: 4px;">23,87</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,57</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,62</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,44</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">70,14</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,09</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,06</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,65</td>
      <td style="border: 1px solid #ccc; padding: 4px;">16,38</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,70</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+ModDrop</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,60</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,75</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,33</td>
      <td style="border: 1px solid #ccc; padding: 4px;">25,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">27,19</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,84</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,24</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,59</td>
      <td style="border: 1px solid #ccc; padding: 4px;">46,95</td>
      <td style="border: 1px solid #ccc; padding: 4px;">71,41</td>
      <td style="border: 1px solid #ccc; padding: 4px;">64,65</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,63</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,86</td>
      <td style="border: 1px solid #ccc; padding: 4px;">14,75</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,75</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+PMR</td>
      <td style="border: 1px solid #ccc; padding: 4px;">76,13</td>
      <td style="border: 1px solid #ccc; padding: 4px;">62,96</td>
      <td style="border: 1px solid #ccc; padding: 4px;">45,99</td>
      <td style="border: 1px solid #ccc; padding: 4px;">61,69</td>
      <td style="border: 1px solid #ccc; padding: 4px;">22,70</td>
      <td style="border: 1px solid #ccc; padding: 4px;">24,61</td>
      <td style="border: 1px solid #ccc; padding: 4px;">18,05</td>
      <td style="border: 1px solid #ccc; padding: 4px;">21,79</td>
      <td style="border: 1px solid #ccc; padding: 4px;">75,15</td>
      <td style="border: 1px solid #ccc; padding: 4px;">49,54</td>
      <td style="border: 1px solid #ccc; padding: 4px;">71,39</td>
      <td style="border: 1px solid #ccc; padding: 4px;">65,36</td>
      <td style="border: 1px solid #ccc; padding: 4px;">17,85</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,77</td>
      <td style="border: 1px solid #ccc; padding: 4px;">20,60</td>
      <td style="border: 1px solid #ccc; padding: 4px;">19,41</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ccc; padding: 4px;">+<strong>PRISM</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>83,41</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>70,49</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>53,27</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>69,06</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,12</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>14,42</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>10,10</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>12,21</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>75,15</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>49,54</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>71,39</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>65,36</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>17,85</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,77</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>20,60</strong></td>
      <td class="hl-yellow" style="border: 1px solid #ccc; padding: 4px;"><strong>19,41</strong></td>
    </tr>
  </tbody>
</table>
