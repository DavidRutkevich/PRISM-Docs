---

title: "Ablationsstudie: Detaillierte Analyse der Verlustfunktionen und Hyperparameter"  
description: "In diesem Artikel betrachte ich die Ablationsstudie zu meinem vorgeschlagenen PRISMS-Framework im Detail. Dabei zeige ich, wie sich verschiedene Hyperparameter auf die Leistung (gemessen in mIoU) auswirken, und diskutiere den Einfluss der einzelnen Komponenten auf das Gesamtergebnis. Im Fokus stehen insbesondere die Modality-Agnostic Distillation (\(\mathcal{L}_{mad}\)), die Unimodal Distillation (\(\mathcal{L}_{umd}\)) sowie die Cross-Modal Distillation (\(\mathcal{L}_{cmd}\))."  
math: true  

---

### 1. Überblick: Ergebnisse auf MUSES und DELIVER

Wie ich bereits im Hauptartikel beschrieben habe, zeigt mein Ansatz auf den Datensätzen **MUSES** (real) und **DELIVER** (synthetisch) deutliche Verbesserungen gegenüber dem aktuellen Stand der Technik:

- **MUSES**: mIoU von **40.23** (+6.37 % gegenüber vorherigem SoTA).  
- **DELIVER**: mIoU von **46.64** (+6.15 % gegenüber MAGIC).

Diese Steigerung belege ich damit, dass mein Modell insbesondere bei fehlenden Modalitäten oder in komplexen Szenarien stabiler bleibt als andere Methoden, die stark von einzelnen Modalitäten (z. B. RGB oder Tiefe) abhängig sind.

---

### 2. Ablation: Modality-Agnostic Distillation \(\mathcal{L}_{mad}\)

Im Folgenden betrachte ich, wie sich unterschiedliche Gewichtungen \(\lambda\) für die Modality-Agnostic Distillation (\(\mathcal{L}_{mad}\)) auf die Performance (mIoU) auswirken. Die Tabelle ist im HTML-Format gehalten und verzichtet auf farbige Hinterlegungen, damit sie in **Dark Mode** und **Light Mode** gleichermaßen gut lesbar bleibt.

<table>
  <thead>
    <tr>
      <th rowspan="2">\(\lambda\)</th>
      <th colspan="2">F</th>
      <th colspan="2">E</th>
      <th colspan="2">L</th>
      <th colspan="2">FE</th>
      <th colspan="2">FL</th>
      <th colspan="2">EL</th>
      <th colspan="2">FEL</th>
      <th colspan="2">Mean</th>
    </tr>
    <tr>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>43.97</td>
      <td>–</td>
      <td>22.33</td>
      <td>–</td>
      <td>31.90</td>
      <td>–</td>
      <td>44.82</td>
      <td>–</td>
      <td>48.61</td>
      <td>–</td>
      <td>35.14</td>
      <td>–</td>
      <td>48.33</td>
      <td>–</td>
      <td>39.30</td>
      <td>–</td>
    </tr>
    <tr>
      <td>10</td>
      <td>43.84</td>
      <td>-0.13</td>
      <td>23.21</td>
      <td>+0.88</td>
      <td>32.71</td>
      <td>+0.81</td>
      <td>44.08</td>
      <td>-0.74</td>
      <td>49.16</td>
      <td>+0.55</td>
      <td>34.97</td>
      <td>-0.17</td>
      <td>48.08</td>
      <td>-0.25</td>
      <td>39.44</td>
      <td>+0.14</td>
    </tr>
    <tr>
      <td>20</td>
      <td>44.08</td>
      <td>+0.11</td>
      <td>22.76</td>
      <td>+0.43</td>
      <td>32.35</td>
      <td>+0.45</td>
      <td>44.37</td>
      <td>-0.45</td>
      <td>49.33</td>
      <td>+0.72</td>
      <td>34.73</td>
      <td>-0.41</td>
      <td>48.79</td>
      <td>+0.46</td>
      <td>39.49</td>
      <td>+0.19</td>
    </tr>
    <tr>
      <td><strong>50</strong></td>
      <td>43.71</td>
      <td>-0.26</td>
      <td>23.00</td>
      <td>+0.67</td>
      <td><strong>34.70</strong></td>
      <td>+2.80</td>
      <td>44.18</td>
      <td>-0.64</td>
      <td>49.13</td>
      <td>+0.52</td>
      <td><strong>37.23</strong></td>
      <td>+2.09</td>
      <td><strong>48.79</strong></td>
      <td>+0.46</td>
      <td><strong>40.11</strong></td>
      <td>+0.81</td>
    </tr>
    <tr>
      <td>60</td>
      <td>44.02</td>
      <td>+0.05</td>
      <td>22.74</td>
      <td>+0.41</td>
      <td>33.82</td>
      <td>+1.92</td>
      <td>44.29</td>
      <td>-0.53</td>
      <td>49.36</td>
      <td>+0.75</td>
      <td>36.69</td>
      <td>+1.55</td>
      <td>48.54</td>
      <td>+0.21</td>
      <td>39.92</td>
      <td>+0.62</td>
    </tr>
    <tr>
      <td>80</td>
      <td>43.84</td>
      <td>-0.13</td>
      <td>22.86</td>
      <td>+0.53</td>
      <td>33.78</td>
      <td>+1.88</td>
      <td>44.25</td>
      <td>-0.57</td>
      <td>49.43</td>
      <td>+0.82</td>
      <td>36.57</td>
      <td>+1.43</td>
      <td>48.72</td>
      <td>+0.39</td>
      <td>39.92</td>
      <td>+0.62</td>
    </tr>
    <tr>
      <td>100</td>
      <td>43.75</td>
      <td>-0.22</td>
      <td>22.87</td>
      <td>+0.54</td>
      <td>34.00</td>
      <td>+2.10</td>
      <td>44.17</td>
      <td>-0.65</td>
      <td>49.36</td>
      <td>+0.75</td>
      <td>36.60</td>
      <td>+1.46</td>
      <td>48.64</td>
      <td>+0.31</td>
      <td>39.91</td>
      <td>+0.61</td>
    </tr>
  </tbody>
</table>

**Wichtigste Erkenntnisse**:  
- Eine **moderate Erhöhung** von \(\lambda\) (bis etwa 50) steigert die Ergebnisse deutlich (Mean mIoU bis 40.11).  
- Besonders **L** (LiDAR) und **EL** (Event + LiDAR) profitieren, da ich diese Modalitäten im Training öfter kompensiere und betone.  
- Über das Optimum hinaus (\(\lambda > 50\)) nimmt der Zugewinn ab, teils kehrt sich der Effekt sogar um.

---

### 3. Ablation: Unimodal Distillation \(\mathcal{L}_{umd}\)

Die **Unimodal Distillation** (\(\alpha\)) stärke ich gezielt für die einzelnen Modalitäten. Die folgende Tabelle zeigt, wie sich verschiedene \(\alpha\)-Werte auswirken, wenn alle anderen Faktoren fixiert sind.

<table>
  <thead>
    <tr>
      <th rowspan="2">\(\alpha\)</th>
      <th colspan="2">F</th>
      <th colspan="2">E</th>
      <th colspan="2">L</th>
      <th colspan="2">FE</th>
      <th colspan="2">FL</th>
      <th colspan="2">EL</th>
      <th colspan="2">FEL</th>
      <th colspan="2">Mean</th>
    </tr>
    <tr>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
      <th>mIoU</th>
      <th>\(\Delta\uparrow\)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>w/o</td>
      <td>43.71</td>
      <td>–</td>
      <td>23.00</td>
      <td>–</td>
      <td>34.70</td>
      <td>–</td>
      <td>44.18</td>
      <td>–</td>
      <td>49.13</td>
      <td>–</td>
      <td>37.23</td>
      <td>–</td>
      <td>48.79</td>
      <td>–</td>
      <td>40.11</td>
      <td>–</td>
    </tr>
    <tr>
      <td>1</td>
      <td>44.54</td>
      <td>+0.83</td>
      <td>22.02</td>
      <td>-0.98</td>
      <td>31.67</td>
      <td>-3.03</td>
      <td>44.66</td>
      <td>+0.48</td>
      <td>49.55</td>
      <td>+0.42</td>
      <td>33.93</td>
      <td>-3.30</td>
      <td>48.89</td>
      <td>+0.10</td>
      <td>39.32</td>
      <td>-0.79</td>
    </tr>
    <tr>
      <td>3</td>
      <td>45.38</td>
      <td>+1.67</td>
      <td>20.64</td>
      <td>-2.36</td>
      <td>31.37</td>
      <td>-3.33</td>
      <td>45.43</td>
      <td>+1.25</td>
      <td>50.53</td>
      <td>+1.40</td>
      <td>33.65</td>
      <td>-3.58</td>
      <td>49.93</td>
      <td>+1.14</td>
      <td>39.56</td>
      <td>-0.55</td>
    </tr>
    <tr>
      <td><strong>5</strong></td>
      <td>45.82</td>
      <td>+2.11</td>
      <td>19.26</td>
      <td>-3.74</td>
      <td>31.79</td>
      <td>-2.91</td>
      <td>45.88</td>
      <td>+1.70</td>
      <td>51.11</td>
      <td>+1.98</td>
      <td>33.56</td>
      <td>-3.67</td>
      <td>50.60</td>
      <td>+1.81</td>
      <td>39.72</td>
      <td>-0.39</td>
    </tr>
    <tr>
      <td>7</td>
      <td>46.09</td>
      <td>+2.38</td>
      <td>17.84</td>
      <td>-5.16</td>
      <td>31.81</td>
      <td>-2.89</td>
      <td>46.18</td>
      <td>+2.00</td>
      <td>51.36</td>
      <td>+2.23</td>
      <td>33.43</td>
      <td>-3.80</td>
      <td>51.01</td>
      <td>+2.22</td>
      <td>39.67</td>
      <td>-0.44</td>
    </tr>
    <tr>
      <td>10</td>
      <td>46.17</td>
      <td>+2.46</td>
      <td>15.74</td>
      <td>-7.26</td>
      <td>31.95</td>
      <td>-2.75</td>
      <td>46.37</td>
      <td>+2.19</td>
      <td>51.17</td>
      <td>+2.04</td>
      <td>33.26</td>
      <td>-3.97</td>
      <td>51.08</td>
      <td>+2.29</td>
      <td>39.39</td>
      <td>-0.72</td>
    </tr>
  </tbody>
</table>

**Wichtigste Erkenntnisse**:  
- Geringe \(\alpha\)-Werte (\(\alpha=1\)) verbessern oft einzelne Modalitäten (z. B. F: +0.83), beeinträchtigen aber andere (E, L).  
- Bei \(\alpha=10\) erreiche ich sehr hohe Werte für **F** und **FL**, allerdings sinkt die Performance der Events stark (E: -7.26).  
- Ein **Trade-off** entsteht zwischen der Optimierung einzelner Modalitäten und der Gesamtfusion.

---

### 4. Cross-Modal Distillation \(\mathcal{L}_{cmd}\) und Kombinationen

Wenn ich die Unimodal Distillation (\(\mathcal{L}_{umd}\)) und die Cross-Modal Distillation (\(\mathcal{L}_{cmd}\)) gemeinsam einsetze, kann ich den **unimodalen Bias** weiter reduzieren. Allerdings muss ich den Faktor \(\beta\) für \(\mathcal{L}_{cmd}\) moderat wählen, denn zu hohe Werte können einzelne Modalitäten überkompensieren und dadurch andere verschlechtern.

---

### 5. Keine Distillation auf „fused Features“

Aus weiteren Experimenten habe ich festgestellt, dass das **direkte Distillieren fusionierter Features** (also ein gemeinsames, gemitteltes Feature aller Modalitäten) kaum Vorteile bringt. Im Gegenteil: Die Performance leidet, da die für das Training relevanten Informationen in den „vermischten“ Merkmalen schwerer zu extrahieren sind. Daher ist es für mich effektiver, **unimodale** und **cross-modale** Repräsentationen separat zu distillieren.

---

## Fazit

Die Ablationsstudie unterstreicht für mich, wie wichtig ein **fein abgestimmtes Zusammenspiel** der einzelnen Distillationsverluste ist:

1. **Modality-Agnostic Distillation** (\(\mathcal{L}_{mad}\)) fängt fehlende Modalitäten semantisch ab.  
2. **Unimodal Distillation** (\(\mathcal{L}_{umd}\)) verbessert gezielt die Leistung einzelner Modalitäten, muss aber durch cross-modale Verfahren ausgeglichen werden.  
3. **Cross-Modal Distillation** (\(\mathcal{L}_{cmd}\)) gleicht intermodale Beziehungen an und verhindert eine Überdominanz „leichter“ Modalitäten.  
4. **Fused-Feature-Distillation** bringt für mich keine Vorteile, da wichtige Details in den zusammengeführten Merkmalen verloren gehen.

Insgesamt zeigen die Experimente, dass das Framework sowohl in Einmodal- als auch in multimodalen Szenarien überzeugt – vorausgesetzt, alle Komponenten werden sorgfältig aufeinander abgestimmt. So erreicht PRISMS auch bei realen Anwendungen, in denen Daten häufig unvollständig oder variabel sind, eine robuste und qualitativ hochwertige Segmentierungsleistung.