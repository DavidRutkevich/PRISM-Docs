---

title: "Ablationsstudie: Detaillierte Analyse der Verlustfunktionen und Hyperparameter"  
description: "In diesem Artikel wird eine Ablationsstudie zum vorgeschlagenen PRISMS-Framework im Detail untersucht."  
math: true  

---

### 1. Überblick: Ergebnisse auf MUSES und DELIVER

Wie bereits im Hauptartikel dargelegt, zeigen die Ergebnisse auf den Datensätzen **MUSES** (real) und **DELIVER** (synthetisch) signifikante Verbesserungen gegenüber dem aktuellen Stand der Technik:

- **MUSES**: mIoU von **40.23** (+6.37 % gegenüber dem bisherigen SoTA).  
- **DELIVER**: mIoU von **46.64** (+6.15 % gegenüber MAGIC).

Diese Verbesserungen belegen, dass das Modell insbesondere in Szenarien mit fehlenden Modalitäten oder in komplexen Umgebungen stabiler reagiert als Methoden, die stark auf einzelne Modalitäten (z. B. RGB oder Tiefe) angewiesen sind.

---

### 2. Ablation: Modality-Agnostic Distillation \(\mathcal{L}_{mad}\)

Im Folgenden wird untersucht, wie sich unterschiedliche Gewichtungen \(\lambda\) für die Modality-Agnostic Distillation (\(\mathcal{L}_{mad}\)) auf die Performance (mIoU) auswirken. Die Tabelle ist im HTML-Format gehalten, um in **Dark Mode** und **Light Mode** gleichermaßen gut lesbar zu sein.

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
      <td>48.79</td>
      <td>+0.46</td>
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

**Wichtigste Erkenntnisse:**  
- Eine moderate Erhöhung von \(\lambda\) (bis etwa 50) führt zu einer deutlichen Steigerung der Ergebnisse (Mean mIoU bis 40.11).  
- Insbesondere profitieren die Modalitäten **L** (LiDAR) und **EL** (Event + LiDAR), da diesen Modalitäten im Training eine verstärkte Kompensation und Betonung zukommt.  
- Über das Optimum hinaus (\(\lambda > 50\)) nimmt der Zugewinn ab, teils kehrt sich der Effekt sogar um.

---

### 3. Ablation: Unimodal Distillation \(\mathcal{L}_{umd}\)

Die **Unimodal Distillation** wird gezielt für die einzelnen Modalitäten verstärkt. Die folgende Tabelle zeigt, wie sich verschiedene \(\alpha\)-Werte auf die Performance auswirken, wenn alle anderen Parameter fixiert bleiben.

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

**Wichtigste Erkenntnisse:**  
- Niedrige \(\alpha\)-Werte (\(\alpha=1\)) führen zu moderaten Verbesserungen in einzelnen Modalitäten (z. B. F: +0.83), während andere Bereiche (E, L) beeinträchtigt werden.  
- Bei \(\alpha=10\) werden sehr hohe Werte für bestimmte Modalitäten (F, FL) erreicht, allerdings sinkt die Performance bei anderen Modalitäten (E: -7.26).  
- Ein klarer Trade-off besteht zwischen der Optimierung einzelner Modalitäten und der Gesamtfusion.

---

### 4. Cross-Modal Distillation \(\mathcal{L}_{cmd}\) und Kombinationen

Die gemeinsame Anwendung der **Unimodal Distillation** (\(\mathcal{L}_{umd}\)) und der **Cross-Modal Distillation** (\(\mathcal{L}_{cmd}\)) führt zu einer weiteren Reduktion des unimodalen Bias. Dabei muss der Faktor \(\beta\) für \(\mathcal{L}_{cmd}\) moderat gewählt werden, da zu hohe Werte zu einer Überkompensation einzelner Modalitäten führen können.

---

### 5. Keine Distillation auf „fused Features“

Weitere Experimente zeigten, dass das direkte Distillieren der fusionierten Features (also einer gemeinsamen, gemittelten Repräsentation aller Modalitäten) kaum Vorteile bringt. Vielmehr verschlechtert sich die Performance, da wichtige Informationen in den vermischten Merkmalen verloren gehen. Daher ist es effektiver, die unimodalen und cross-modalen Repräsentationen separat zu distillieren.

---

## Fazit

Die vorliegende Ablationsstudie verdeutlicht, wie wichtig ein fein abgestimmtes Zusammenspiel der verschiedenen Distillationsverluste ist:

1. **Modality-Agnostic Distillation** (\(\mathcal{L}_{mad}\)) fängt semantische Unterschiede fehlender Modalitäten ab.  
2. **Unimodal Distillation** (\(\mathcal{L}_{umd}\)) verbessert gezielt die Leistung einzelner Modalitäten und muss durch cross-modale Verfahren ergänzt werden.  
3. **Cross-Modal Distillation** (\(\mathcal{L}_{cmd}\)) gleicht intermodale Unterschiede aus und verhindert eine Überdominanz „leichter“ Modalitäten.  
4. **Fused-Feature-Distillation** erweist sich als weniger effektiv, da wichtige Details in den zusammengeführten Merkmalen verloren gehen.

Insgesamt belegen die Experimente, dass das Framework in sowohl einmodalen als auch multimodalen Szenarien überzeugende Ergebnisse erzielt – vorausgesetzt, dass alle Komponenten sorgfältig aufeinander abgestimmt werden. Dadurch wird eine robuste und qualitativ hochwertige Segmentierung auch in realen Anwendungen, in denen Daten häufig unvollständig oder variabel sind, erreicht.