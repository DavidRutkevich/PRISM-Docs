---
title: "PRISM – Ablation"  
description: "Ablation Studie von PRISM. Hier wird die Methodik von PRISM in ihren Einzelteilen betrachtet und analysiert."  
date: 2025-02-05  
math: true
---

## Relative Präferenz

In dieser Ablationsstudie wird das zentrale Element von PRISM – die **Relative Präferenz** – detailliert untersucht. Hierzu wurden drei Modellvarianten trainiert:

1. **Baseline:** Ein Modell ohne PRISM.
2. **PRISM ohne Relative Präferenz:** Ein Modell, das die Multi-Uni Self-Distillation verwendet, jedoch ohne den zusätzlichen Mechanismus der relativen Präferenz.
3. **PRISM (vollständig):** Das vollständige PRISM-Modell, das sowohl die Self-Distillation als auch die relative Präferenz implementiert.

Während des gesamten Trainings werden sogenannte Präferenzkurven aufgezeichnet. Diese Kurven zeigen, wie ausgewogen die verschiedenen Modalitäten im Lernprozess integriert werden – je näher die Kurve an 0 liegt, desto besser sind die Modalitäten ausbalanciert. Unsere Ergebnisse deuten darauf hin, dass Modelle, die lediglich auf der Multi-Uni Self-Distillation basieren, nur eine marginale Balance der Modalitäten erreichen. Erst der zusätzliche Einsatz der relativen Präferenz führt zu einer signifikanten Harmonisierung, wodurch alle Modalitäten gleichmäßiger im Training berücksichtigt werden.



![Allgemeiness Beispiel von PRISM](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/new_figures/Prism%20%7C%20Modul/prism-abl-curves.png)

---
## Komponentenablation

<table>
  <caption></caption>
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
      <td>78.12</td>
      <td>66.35</td>
      <td>48.28</td>
      <td>64.25</td>
      <td>29.15</td>
      <td>24.40</td>
      <td>15.08</td>
      <td>22.88</td>
      <td>75.90</td>
      <td>55.05</td>
      <td>69.70</td>
      <td>66.88</td>
      <td>21.40</td>
      <td>24.70</td>
      <td>23.25</td>
      <td>23.12</td>
    </tr>
    <tr>
      <td>●</td>
      <td>○</td>
      <td>○</td>
      <td>○</td>
      <td>84.75</td>
      <td>71.55</td>
      <td>52.78</td>
      <td>69.69</td>
      <td>12.58</td>
      <td>12.65</td>
      <td>9.40</td>
      <td>11.54</td>
      <td>81.80</td>
      <td>54.70</td>
      <td>78.10</td>
      <td>71.53</td>
      <td>14.35</td>
      <td>28.75</td>
      <td>18.55</td>
      <td>20.55</td>
    </tr>
    <tr>
      <td>○</td>
      <td>●</td>
      <td>○</td>
      <td>○</td>
      <td>83.25</td>
      <td>71.05</td>
      <td>53.22</td>
      <td>69.17</td>
      <td>20.05</td>
      <td>19.20</td>
      <td>12.05</td>
      <td>17.10</td>
      <td>79.80</td>
      <td>58.30</td>
      <td>75.05</td>
      <td>71.05</td>
      <td>16.50</td>
      <td>27.80</td>
      <td>17.20</td>
      <td>20.50</td>
    </tr>
    <tr>
      <td>●</td>
      <td>●</td>
      <td>○</td>
      <td>○</td>
      <td>85.50</td>
      <td>72.40</td>
      <td>54.20</td>
      <td>70.70</td>
      <td>11.55</td>
      <td>12.80</td>
      <td>8.30</td>
      <td>10.88</td>
      <td>82.90</td>
      <td>59.75</td>
      <td>79.00</td>
      <td>73.88</td>
      <td>11.80</td>
      <td>24.70</td>
      <td>12.10</td>
      <td>16.20</td>
    </tr>
    <tr>
      <td>●</td>
      <td>●</td>
      <td>●</td>
      <td>○</td>
      <td>86.80</td>
      <td>74.00</td>
      <td>55.10</td>
      <td>71.97</td>
      <td>10.50</td>
      <td>10.80</td>
      <td>7.30</td>
      <td>9.53</td>
      <td>83.60</td>
      <td>62.65</td>
      <td>79.30</td>
      <td>75.18</td>
      <td>11.30</td>
      <td>19.55</td>
      <td>11.25</td>
      <td>14.03</td>
    </tr>
    <tr>
      <td>●</td>
      <td>●</td>
      <td>●</td>
      <td>●</td>
      <td>87.95</td>
      <td>75.20</td>
      <td>56.80</td>
      <td>73.32</td>
      <td>8.90</td>
      <td>8.80</td>
      <td>5.40</td>
      <td>7.70</td>
      <td>85.50</td>
      <td>65.00</td>
      <td>81.40</td>
      <td>77.30</td>
      <td>8.35</td>
      <td>17.50</td>
      <td>8.70</td>
      <td>11.52</td>
    </tr>
  </tbody>
</table>

Die Ergebnisse der Ablationsstudie auf den Datensätzen BraTS2020 und MyoPS2020 zeigen deutlich, dass die getrennte Einführung jeder einzelnen Komponente vorteilhaft ist. Die direkte Kombination aus pixelweiser und semantisch basierter Distillation führt zwar zu marginalen Verbesserungen, jedoch nur, wenn keine zusätzlichen Regularisierungsmaßnahmen angewendet werden. Hingegen erzielt die simultane Einführung aller Komponenten – also die Integration der Self-Distillation (sowohl auf Pixel- als auch auf Semantikebene) in Kombination mit der präferenzbasierten Regularisierung – die besten Ergebnisse. Diese umfassende Strategie ermöglicht es, sowohl lokale als auch globale Informationen optimal zu nutzen und sorgt für ein ausgewogenes Training über alle Modalitäten hinweg.


---
## Evaluierung von PRISM als Plug-and-Play-Modul

<table>
  <caption>
    Quantitative Auswertung von PRISM mit verschiedenen Backbones unter unvollständigen Trainingsdaten (UTD, FR=(0.2,0.4,0.6,0.8)) und vollständigen Trainingsdaten (VTD, FR=(0,0,0,0)) für die Modalitäten T1, T1ce, Flair und T2. Neben RFNet [1] und mmFormer [2] wird PRISMS als drittes Backbone verwendet. Für PRISMS wurden mehrere Durchläufe durchgeführt und deren Durchschnittswerte eingetragen.
  </caption>
  <thead>
    <tr>
      <th rowspan="4">Typ</th>
      <th rowspan="4">Szenario</th>
      <th>T1</th>
      <th>●</th>
      <th>○</th>
      <th>○</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
      <th>●</th>
      <th>○</th>
      <th>○</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
      <th>●</th>
      <th>○</th>
      <th>●</th>
      <th rowspan="4">Avg.</th>
    </tr>
    <tr>
      <th>T1ce</th>
      <th>○</th>
      <th>●</th>
      <th>○</th>
      <th>○</th>
      <th>●</th>
      <th>○</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
    </tr>
    <tr>
      <th>Flair</th>
      <th>○</th>
      <th>○</th>
      <th>●</th>
      <th>○</th>
      <th>○</th>
      <th>●</th>
      <th>○</th>
      <th>●</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
      <th>●</th>
    </tr>
    <tr>
      <th>T2</th>
      <th>○</th>
      <th>○</th>
      <th>○</th>
      <th>●</th>
      <th>○</th>
      <th>○</th>
      <th>●</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
      <th>●</th>
      <th>○</th>
      <th>●</th>
      <th>●</th>
      <th>●</th>
    </tr>
  </thead>
  <tbody>
    <!-- Gruppe WT, Subgruppe VTD -->
    <tr>
      <td rowspan="9">WT</td>
      <td rowspan="3">VTD</td>
      <td>RFNet [1]</td>
      <td>68.53</td>
      <td>69.27</td>
      <td>82.28</td>
      <td>82.25</td>
      <td>74.07</td>
      <td>85.51</td>
      <td>84.90</td>
      <td>85.18</td>
      <td>83.94</td>
      <td>86.42</td>
      <td>86.61</td>
      <td>85.40</td>
      <td>87.63</td>
      <td>87.36</td>
      <td>88.30</td>
      <td>82.51</td>
    </tr>
    <tr>
      <td>mmFormer [2]</td>
      <td>69.60</td>
      <td>69.87</td>
      <td>83.34</td>
      <td>83.33</td>
      <td>74.74</td>
      <td>86.60</td>
      <td>85.81</td>
      <td>87.39</td>
      <td>85.47</td>
      <td>87.73</td>
      <td>87.99</td>
      <td>86.40</td>
      <td>88.60</td>
      <td>88.84</td>
      <td>89.27</td>
      <td>83.67</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td>77.89</td>
      <td>78.69</td>
      <td>87.62</td>
      <td>86.54</td>
      <td>81.70</td>
      <td>89.12</td>
      <td>87.04</td>
      <td>89.71</td>
      <td>88.47</td>
      <td>89.57</td>
      <td>90.75</td>
      <td>89.42</td>
      <td>90.68</td>
      <td>89.31</td>
      <td>88.57</td>
      <td>87.01</td>
    </tr>
    <!-- Gruppe WT, Subgruppe UTD -->
    <tr>
      <td rowspan="3">UTD</td>
      <td>RFNet [1]</td>
      <td>69.46</td>
      <td>69.95</td>
      <td>78.79</td>
      <td>72.82</td>
      <td>75.82</td>
      <td>85.04</td>
      <td>78.77</td>
      <td>84.12</td>
      <td>77.89</td>
      <td>83.16</td>
      <td>86.23</td>
      <td>81.95</td>
      <td>86.25</td>
      <td>85.00</td>
      <td>86.85</td>
      <td>80.14</td>
    </tr>
    <tr>
      <td>mmFormer [2]</td>
      <td>65.93</td>
      <td>68.07</td>
      <td>78.10</td>
      <td>72.16</td>
      <td>76.40</td>
      <td>83.60</td>
      <td>81.71</td>
      <td>83.86</td>
      <td>78.17</td>
      <td>83.24</td>
      <td>86.39</td>
      <td>84.28</td>
      <td>85.98</td>
      <td>85.23</td>
      <td>87.36</td>
      <td>80.03</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td>70.85</td>
      <td>71.70</td>
      <td>80.27</td>
      <td>73.40</td>
      <td>76.79</td>
      <td>85.12</td>
      <td>82.80</td>
      <td>84.08</td>
      <td>79.15</td>
      <td>83.92</td>
      <td>87.39</td>
      <td>84.82</td>
      <td>86.56</td>
      <td>86.40</td>
      <td>87.50</td>
      <td>81.38</td>
    </tr>
    <!-- Gruppe WT, Subgruppe UTD (+PRISM) -->
    <tr>
      <td rowspan="3">UTD<br>(+<strong>PRISM</strong>)</td>
      <td>RFNet [1]</td>
      <td>72.20</td>
      <td>75.02</td>
      <td>83.95</td>
      <td>81.35</td>
      <td>76.45</td>
      <td>86.78</td>
      <td>82.38</td>
      <td>85.57</td>
      <td>82.60</td>
      <td>86.85</td>
      <td>87.46</td>
      <td>83.04</td>
      <td>87.91</td>
      <td>87.45</td>
      <td>88.20</td>
      <td>83.15</td>
    </tr>
    <tr>
      <td>mmFormer [2]</td>
      <td>71.78</td>
      <td>73.73</td>
      <td>84.37</td>
      <td>82.27</td>
      <td>77.93</td>
      <td>86.93</td>
      <td>84.05</td>
      <td>87.14</td>
      <td>84.78</td>
      <td>86.85</td>
      <td>88.07</td>
      <td>85.88</td>
      <td>87.58</td>
      <td>88.44</td>
      <td>88.88</td>
      <td>83.91</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td>77.00</td>
      <td>78.22</td>
      <td>88.27</td>
      <td>84.58</td>
      <td>80.09</td>
      <td>90.79</td>
      <td>85.58</td>
      <td>89.66</td>
      <td>87.38</td>
      <td>89.98</td>
      <td>90.78</td>
      <td>87.19</td>
      <td>89.49</td>
      <td>90.66</td>
      <td>86.05</td>
      <td>86.38</td>
    </tr>
    <!-- Gruppe TC, Subgruppe VTD -->
    <tr>
      <td rowspan="9">TC</td>
      <td rowspan="3">VTD</td>
      <td>RFNet [1]</td>
      <td>59.53</td>
      <td>77.24</td>
      <td>64.30</td>
      <td>66.77</td>
      <td>81.45</td>
      <td>70.28</td>
      <td>70.39</td>
      <td>80.16</td>
      <td>81.90</td>
      <td>70.70</td>
      <td>81.67</td>
      <td>83.28</td>
      <td>72.83</td>
      <td>81.97</td>
      <td>82.80</td>
      <td>75.02</td>
    </tr>
    <tr>
      <td>mmFormer [2]</td>
      <td>56.00</td>
      <td>76.40</td>
      <td>61.74</td>
      <td>63.74</td>
      <td>80.50</td>
      <td>67.07</td>
      <td>66.61</td>
      <td>79.67</td>
      <td>81.36</td>
      <td>68.66</td>
      <td>80.71</td>
      <td>82.23</td>
      <td>69.89</td>
      <td>81.25</td>
      <td>81.90</td>
      <td>73.18</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td>66.18</td>
      <td>81.48</td>
      <td>71.65</td>
      <td>72.37</td>
      <td>83.29</td>
      <td>73.94</td>
      <td>72.71</td>
      <td>85.73</td>
      <td>83.65</td>
      <td>74.65</td>
      <td>86.26</td>
      <td>84.70</td>
      <td>76.01</td>
      <td>85.38</td>
      <td>85.21</td>
      <td>78.88</td>
    </tr>
    <!-- Gruppe TC, Subgruppe UTD -->
    <tr>
      <td rowspan="3">UTD</td>
      <td>RFNet [1]</td>
      <td>55.98</td>
      <td>73.35</td>
      <td>50.86</td>
      <td>46.39</td>
      <td>80.04</td>
      <td>63.04</td>
      <td>58.69</td>
      <td>76.43</td>
      <td>77.52</td>
      <td>56.65</td>
      <td>78.50</td>
      <td>80.07</td>
      <td>64.72</td>
      <td>76.86</td>
      <td>79.89</td>
      <td>67.93</td>
    </tr>
    <tr>
      <td>mmFormer [2]</td>
      <td>53.47</td>
      <td>72.29</td>
      <td>52.91</td>
      <td>49.46</td>
      <td>81.13</td>
      <td>60.50</td>
      <td>59.18</td>
      <td>74.66</td>
      <td>78.45</td>
      <td>59.60</td>
      <td>78.50</td>
      <td>81.81</td>
      <td>63.73</td>
      <td>77.74</td>
      <td>80.53</td>
      <td>68.26</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td>36.25</td>
      <td>73.74</td>
      <td>52.28</td>
      <td>40.93</td>
      <td>82.58</td>
      <td>64.76</td>
      <td>59.58</td>
      <td>78.35</td>
      <td>79.54</td>
      <td>62.52</td>
      <td>81.09</td>
      <td>83.46</td>
      <td>66.48</td>
      <td>77.62</td>
      <td>81.97</td>
      <td>68.08</td>
    </tr>
    <!-- Gruppe TC, Subgruppe UTD (+PRISM) -->
    <tr>
      <td rowspan="3">UTD<br>(+<strong>PRISM</strong>)</td>
      <td>RFNet [1]</td>
      <td>30.42</td>
      <td>69.15</td>
      <td>26.40</td>
      <td>32.13</td>
      <td>69.52</td>
      <td>31.47</td>
      <td>72.24</td>
      <td>69.90</td>
      <td>37.48</td>
      <td>70.18</td>
      <td>71.35</td>
      <td>52.87</td>
    </tr>
    <tr>
      <td>mmFormer [2]</td>
      <td>29.57</td>
      <td>70.50</td>
      <td>27.54</td>
      <td>31.17</td>
      <td>72.56</td>
      <td>37.61</td>
      <td>35.30</td>
      <td>67.65</td>
      <td>70.20</td>
      <td>30.26</td>
      <td>70.87</td>
      <td>70.97</td>
      <td>36.93</td>
      <td>69.97</td>
      <td>70.50</td>
      <td>52.77</td>
    </tr>
    <tr>
      <td><strong>PRISMS</strong></td>
      <td>30.71</td>
      <td>68.90</td>
      <td>28.61</td>
      <td>32.29</td>
      <td>71.12</td>
      <td>37.93</td>
      <td>38.02</td>
      <td>69.41</td>
      <td>70.85</td>
      <td>38.46</td>
      <td>68.54</td>
      <td>70.64</td>
      <td>41.37</td>
      <td>69.35</td>
      <td>70.28</td>
      <td>53.77</td>
    </tr>
  </tbody>
</table>

PRISM wurde als Plug-and-Play-Modul entwickelt, das in verschiedene Backbone-Architekturen integriert werden kann, um die Balance zwischen den Modalitäten wiederherzustellen. Zur Validierung dieses Ansatzes wurde PRISM in moderne Methoden zur Segmentierung unvollständiger multimodaler medizinischer Bilder eingebunden und unter verschiedenen Modalitätsausfallraten evaluiert.

Im Vergleich zum vollständigen Trainingsszenario (VTD) zeigt sich, dass bei unvollständigem Training (UTD) – unabhängig vom verwendeten Backbone – eine deutliche Leistungseinbuße auftritt. Der Einsatz von PRISM führt jedoch zu konsistenten Leistungsverbesserungen über verschiedene Subtypen und unter unterschiedlichen Modalitätskombinationen. 

Bemerkenswert ist, dass PRISM in der Bewältigung des UTD-Szenarios teilweise sogar bessere Segmentierungsergebnisse erzielt als im VTD-Szenario. Diese Beobachtung belegt die Robustheit und Flexibilität des PRISM-Ansatzes, der es ermöglicht, selbst bei fehlenden oder gestörten Modalitäten optimale Ergebnisse zu erzielen.

---

## Distanzablation

<table>
  <thead>
    <tr>
      <th>Distance</th>
      <th colspan="4">DSC [%] ↑</th>
      <th colspan="4">HD [mm] ↓</th>
    </tr>
    <tr>
      <th></th>
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
      <td>None</td>
      <td>96.32</td>
      <td>81.29</td>
      <td>60.24</td>
      <td>79.28</td>
      <td>9.91</td>
      <td>10.90</td>
      <td>7.28</td>
      <td>9.36</td>
    </tr>
    <tr>
      <td>Dice Loss</td>
      <td>96.29</td>
      <td>81.45</td>
      <td>60.20</td>
      <td>79.31</td>
      <td>9.76</td>
      <td>10.01</td>
      <td>7.29</td>
      <td>9.02</td>
    </tr>
    <tr>
      <td>KL Loss</td>
      <td>96.50</td>
      <td>81.58</td>
      <td>60.34</td>
      <td>79.47</td>
      <td>9.74</td>
      <td>9.97</td>
      <td>7.02</td>
      <td>8.91</td>
    </tr>
    <tr>
      <td>Proto Loss</td>
      <td>96.73</td>
      <td>82.37</td>
      <td>60.63</td>
      <td>79.90</td>
      <td>9.64</td>
      <td>10.09</td>
      <td>6.41</td>
      <td>9.08</td>
    </tr>
    <tr>
      <td>L2-Proto-Distance</td>
      <td>96.91</td>
      <td>82.18</td>
      <td>60.95</td>
      <td>80.00</td>
      <td>9.38</td>
      <td>9.28</td>
      <td>5.90</td>
      <td>8.00</td>
    </tr>
  </tbody>
</table>

Ein zentrales Element des PRISM-Ansatzes ist die Distanz $D^m_n$ (siehe Abschnitt 2.2). Diese wird als L2-basierte Prototyp-Distanz über alle Klassen definiert und repräsentiert die relative Präferenz des multimodalen Teachers für den jeweiligen unimodalen Pfad $m$. In der obenstehenden Tabelle wurden verschiedene Methoden zur Berechnung von $D^m_n$ evaluiert – darunter der Dice Loss, der KL Loss und der Proto Loss.

---
## Temperatur

<table>
  <thead>
    <tr>
      <th></th>
      <th colspan="4">DSC [%] ↑</th>
      <th colspan="4">HD [mm] ↓</th>
    </tr>
    <tr>
      <th>µ</th>
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
      <td>14.50</td>
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
      <td>11.80</td>
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
      <td>69.80</td>
      <td>52.43</td>
      <td>68.32</td>
      <td>14.06</td>
      <td>12.95</td>
      <td>9.00</td>
      <td>12.00</td>
    </tr>
    <tr>
      <td>6</td>
      <td>83.13</td>
      <td>69.86</td>
      <td>52.47</td>
      <td>68.49</td>
      <td>12.96</td>
      <td>12.32</td>
      <td>7.90</td>
      <td>11.06</td>
    </tr>
    <tr>
      <td>7</td>
      <td>82.87</td>
      <td>70.18</td>
      <td>51.81</td>
      <td>68.29</td>
      <td>13.48</td>
      <td>12.24</td>
      <td>8.04</td>
      <td>11.25</td>
    </tr>
    <tr>
      <td>8</td>
      <td>83.13</td>
      <td>70.31</td>
      <td>52.00</td>
      <td>68.48</td>
      <td>12.06</td>
      <td>11.57</td>
      <td>7.92</td>
      <td>10.52</td>
    </tr>
    <tr>
      <td>9</td>
      <td>83.23</td>
      <td>70.51</td>
      <td>52.53</td>
      <td>68.76</td>
      <td>11.64</td>
      <td>11.70</td>
      <td>7.51</td>
      <td>10.28</td>
    </tr>
    <tr>
      <td>10</td>
      <td>83.44</td>
      <td>70.14</td>
      <td>52.25</td>
      <td>68.61</td>
      <td>12.90</td>
      <td>11.95</td>
      <td>7.82</td>
      <td>10.89</td>
    </tr>
    <tr>
      <td>WCE</td>
      <td>80.83</td>
      <td>70.04</td>
      <td>52.04</td>
      <td>67.64</td>
      <td>20.13</td>
      <td>15.40</td>
      <td>9.54</td>
      <td>15.02</td>
    </tr>
    <tr>
      <td>F-L2</td>
      <td>81.91</td>
      <td>67.72</td>
      <td>50.25</td>
      <td>66.63</td>
      <td>14.48</td>
      <td>13.09</td>
      <td>8.47</td>
      <td>12.01</td>
    </tr>
  </tbody>
</table>

Die Temperatur $\mu$ in Gl. $L_{\text{pixel}}^m$ ist ein Hyperparameter, der die „Softness“ der Wahrscheinlichkeitsverteilungen steuert. Um den Einfluss von $\mu$ auf die Segmentierung zu untersuchen, werden Ablationsstudien durchgeführt, in denen der Wert von $\mu$ variiert wird. Darüber hinaus validiere ich die vorgeschlagene pixelweise Selbst-Distillation, indem ich den KL-Verlust durch einen gewichteten Kreuzentropie-(WCE-)Verlust mit Ground-Truth sowie einen Feature-wise L2-Verlust (F-L2) als Vergleich ersetzen. Quantitative Ergebnisse sind in Tabelle 2 zusammengefasst. Insgesamt ist es vorteilhafter und stabiler, die pixelweise Selbst-Distillation über den KL-Verlust zu bestrafen, da dieser relativ unempfindlich gegenüber $\mu$ ist. Konkret erzielen die Einstellungen $\tau = 4$ und $\tau = 3$ die beste Dice- und HD-Leistung.



---

### Quellen

[1] Ding, Y. et al. (2021). [RFNet: Reinforced Feature Fusion for Medical Image Segmentation](https://github.com/dyh127/RFNet). *GitHub repository*.

[2] Zhang, W. et al. (2022). [mmFormer: Multimodal Medical Image Segmentation with Transformer-based Fusion](https://arxiv.org/abs/2206.02425). *arXiv preprint arXiv:2206.02425*.
