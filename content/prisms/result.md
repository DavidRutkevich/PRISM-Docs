---
title: "Ergebnisse"
description: "Ergebnisse der Auswertung von PRISMS. Qualitativ und Quantitativ"
date: 2025-02-05
math: true
---
### Datensätze

Für die Evaluation werden zwei Datensätze aus der "Multimodal Brain Tumor Segmentation Challenge (BRATS)" herangezogen. Beide Datensätze enthalten MRT-Bilder aus vier Modalitäten: Flair, T1ce, T2 und T1. Wie in [1–3] beschrieben, wurden zunächst die schwarzen Hintergrundregionen entfernt, und jede Modalität wurde anschließend auf einen Nullmittelwert und eine Einheitsvarianz normalisiert.
Für den BRATS2018-Datensatz wurden die Datenaufteilungen aus [1] und [3] übernommen, sodass die Fälle in 199 Trainings-, 29 Validierungs- und 57 Testbeispiele unterteilt wurden. Beim BRATS2020-Datensatz, der 369 Fälle umfasst, erfolgte eine Aufteilung in 219 Trainings-, 50 Validierungs- und 100 Testbeispiele – hier wurde die Methodik aus [3] strikt eingehalten. Zur Evaluierung wurden sowohl der Dice Similarity Coefficient (DSC) als auch die Hausdorff-Distanz (HD) verwendet.



### Implementierungsdetails

Das Framework wurde in PyTorch implementiert und mit dem AdamW-Optimierer trainiert. Dabei kam eine initiale Lernrate von \(2 \times 10^{-4}\) sowie ein Weight Decay von \(1 \times 10^{-4}\) zum Einsatz; die Batch-Größe betrug 2. Das Training fand über 1000 Epochen auf Google Colab Premium statt. Zur Anpassung der Lernrate wurde eine adaptive Strategie („why-uop learning rate scheduling“) genutzt, die mit einem Poly-Decay-Verfahren (mit \(p = 0.9\)) kombiniert wurde.

Gemäß [1–3] erfolgte das Training im VTD-Szenario, wobei modalitiespezifische Maskierung zur Simulation fehlender Modalitäten eingeführt wurde. Jedes Volumen wurde dabei zufällig auf eine Größe von \(80 \times 80 \times 80\) Pixeln zugeschnitten und zusätzlich durch zufällige Rotationen, Intensitätsänderungen sowie Mirror-Flips augmentiert. In Bezug auf die Positional Embeddings wurden, analog zu dem Inter-Modal Transformer im mmFormer, lernbare Positions-Embeddings den Fusionstokens hinzugefügt – ein Konzept, das dem großen Class-Token im ViT ähnelt.

## Vergleich mit state-of-the-art Methoden

Basierend auf der Verfügbarkeit von Open-Source-Code sowie den öffentlich zugänglichen Datenaufteilungen wurden fünf moderne Ansätze für die unvollständige multimodale Hirntumorsegmentierung ausgewählt. Für den Vergleich wurden identische Datensplits verwendet. Zu den ausgewählten Methoden zählen sowohl CNN-basierte Ansätze – namentlich HeMIS [A], U-HVED [B], RobustMSeg [C] und RFNet [D] – als auch der transformerbasierte Ansatz mmFormer [E].




### 1) Quantitativer Vergleich

In Tab. 1 sind die Ergebnisse aller 15 multimodalen Modalitätskombinationen auf dem BraTS2018-Datensatz aufgeführt. Unter den etablierten State‑of‑the‑Art-Methoden erzielt **RFNet** die besten Mittelwerte für Whole Tumor (WT), Tumor Core (TC) und Enhancing Tumor (ET) und übertrifft **mmFormer** in 40 von 45 Kombinationen. **PRISMS** hingegen liefert in **allen** 45 Konstellationen durchgängig höhere Dice-Werte als sowohl mmFormer als auch RFNet – und zwar für WT, TC und ET gleichermaßen.

Ein ähnliches Bild zeigt sich in Tab. 2 (z. B. auf BraTS2020 oder im UTD-Szenario): Die größten Leistungsgewinne durch PRISMS treten in den schwierigeren Klassen auf (Reihenfolge der Schwierigkeit: WT < TC < ET. Dies unterstreicht, dass PRISMS cross‑modale Informationen besonders effektiv nutzt, was gerade in herausfordernden Settings (UTD vs. VTD) entscheidend ist. Zusätzlich belegen die angeführten p‑Werte die statistische Signifikanz dieser Verbesserungen.

Schließlich werden in Tab. 3 die gleichen Methoden anhand der Hausdorff-Distanz (HD) verglichen. Auch hier zeigt PRISMS durchweg niedrigere HD-Werte als die Konkurrenz, was auf eine präzisere Formwiedergabe der Segmentierungen hinweist.

<table>
  <caption>Tabelle 1 BraTS2018</caption>
  <thead>
    <tr><th rowspan="4">Typ</th><th rowspan="4">Model</th><th>Flair</th><th>○</th><th>○</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>○</th><th>●</th><th>●</th><th>●</th><th>●</th><th>●</th><th>●</th><th>●</th><th rowspan="4">Avg.</th><th rowspan="4">p‑value</th></tr>
    <tr><th>T1</th><th>○</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>●</th><th>●</th></tr>
    <tr><th>T1ce</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>●</th><th>●</th><th>●</th></tr>
    <tr><th>T2</th><th>●</th><th>○</th><th>○</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="6">WT</td><td>HeMIS</td><td>78.31</td><td>55.82</td><td>53.23</td><td>75.16</td><td>80.62</td><td>63.28</td><td>81.14</td><td>80.30</td><td>83.13</td><td>82.24</td><td>83.77</td><td>84.82</td><td>85.19</td><td>81.83</td><td>85.85</td><td>76.98</td><td>76.98</td><td>&lt;0.001</td></tr>
    <tr><td>U‑HVED</td><td>80.11</td><td>61.41</td><td>57.03</td><td>77.30</td><td>82.92</td><td>66.82</td><td>82.59</td><td>82.06</td><td>85.42</td><td>84.07</td><td>85.64</td><td>86.19</td><td>87.37</td><td>83.36</td><td>87.68</td><td>79.33</td><td>79.33</td><td>&lt;0.001</td></tr>
    <tr><td>RobustMSeg</td><td>83.43</td><td>70.66</td><td>67.91</td><td>80.20</td><td>85.56</td><td>74.55</td><td>85.75</td><td>85.21</td><td>87.76</td><td>86.48</td><td>88.73</td><td>88.26</td><td>88.33</td><td>85.96</td><td>88.65</td><td>83.16</td><td>83.16</td><td>0.001</td></tr>
    <tr><td>RFNet</td><td>84.68</td><td>76.33</td><td>76.16</td><td>85.69</td><td>86.69</td><td>79.54</td><td>88.05</td><td>86.60</td><td>88.20</td><td>88.35</td><td>88.76</td><td>89.01</td><td>89.19</td><td>87.17</td><td>89.46</td><td>85.59</td><td>85.59</td><td>0.006</td></tr>
    <tr><td>mmFormer</td><td>84.28</td><td>75.24</td><td>73.36</td><td>85.01</td><td>86.10</td><td>78.60</td><td>87.39</td><td>86.00</td><td>88.00</td><td>88.11</td><td>88.51</td><td>88.49</td><td>89.01</td><td>86.61</td><td>89.19</td><td>84.93</td><td>84.93</td><td>0.002</td></tr>
    <tr><td><strong>PRISMS</strong></td><td>86.92</td><td>77.78</td><td>77.21</td><td>87.15</td><td>88.07</td><td>81.06</td><td>88.37</td><td>87.45</td><td>89.24</td><td>88.85</td><td>88.95</td><td>89.39</td><td>89.78</td><td>88.00</td><td>89.73</td><td>86.53</td><td><strong>86.53</strong></td><td>--</td></tr>
    <tr><td rowspan="6">TC</td><td>HeMIS</td><td>56.68</td><td>62.49</td><td>33.18</td><td>46.25</td><td>74.95</td><td>66.28</td><td>51.90</td><td>58.73</td><td>59.21</td><td>73.60</td><td>75.31</td><td>60.34</td><td>77.11</td><td>75.93</td><td>77.45</td><td>63.29</td><td>63.29</td><td>&lt;0.001</td></tr>
    <tr><td>U‑HVED</td><td>58.83</td><td>67.79</td><td>41.68</td><td>43.66</td><td>76.26</td><td>70.77</td><td>51.88</td><td>60.89</td><td>60.89</td><td>75.23</td><td>76.30</td><td>62.26</td><td>77.95</td><td>76.99</td><td>78.37</td><td>65.32</td><td>65.32</td><td>&lt;0.001</td></tr>
    <tr><td>RobustMSeg</td><td>65.86</td><td>77.43</td><td>55.61</td><td>55.73</td><td>83.91</td><td>80.72</td><td>68.37</td><td>70.45</td><td>70.51</td><td>81.11</td><td>82.26</td><td>72.39</td><td>82.70</td><td>84.02</td><td>83.18</td><td>74.28</td><td>74.28</td><td>0.001</td></tr>
    <tr><td>RFNet</td><td>69.69</td><td>81.88</td><td>65.92</td><td>68.14</td><td>84.02</td><td>82.36</td><td>73.92</td><td>72.54</td><td>73.06</td><td>82.87</td><td>83.89</td><td>74.68</td><td>83.69</td><td>84.77</td><td>84.48</td><td>77.73</td><td>77.73</td><td>0.016</td></tr>
    <tr><td>mmFormer</td><td>67.97</td><td>79.01</td><td>62.06</td><td>64.80</td><td>82.26</td><td>81.37</td><td>72.72</td><td>71.38</td><td>71.93</td><td>82.04</td><td>83.42</td><td>74.09</td><td>83.23</td><td>83.43</td><td>84.00</td><td>76.25</td><td>76.25</td><td>0.002</td></tr>
    <tr><td><strong>PRISMS</strong></td><td>72.37</td><td>82.60</td><td>66.24</td><td>69.89</td><td>85.23</td><td>83.45</td><td>74.08</td><td>74.45</td><td>75.40</td><td>84.78</td><td>85.26</td><td>76.48</td><td>85.29</td><td>85.46</td><td>85.67</td><td>79.11</td><td><strong>79.11</strong></td><td>--</td></tr>
    <tr><td rowspan="6">ET</td><td>HeMIS</td><td>30.06</td><td>57.08</td><td>6.60</td><td>20.63</td><td>63.96</td><td>59.17</td><td>14.83</td><td>29.88</td><td>31.48</td><td>65.62</td><td>68.16</td><td>29.74</td><td>64.66</td><td>64.82</td><td>66.69</td><td>44.89</td><td>44.89</td><td>&lt;0.001</td></tr>
    <tr><td>U‑HVED</td><td>30.85</td><td>59.49</td><td>13.18</td><td>13.40</td><td>64.66</td><td>64.18</td><td>18.98</td><td>32.98</td><td>32.73</td><td>64.29</td><td>66.56</td><td>31.84</td><td>66.60</td><td>67.21</td><td>68.46</td><td>46.36</td><td>46.36</td><td>&lt;0.001</td></tr>
    <tr><td>RobustMSeg</td><td>37.13</td><td>63.99</td><td>26.30</td><td>28.92</td><td>66.93</td><td>67.24</td><td>36.24</td><td>40.54</td><td>40.26</td><td>66.92</td><td>67.90</td><td>42.38</td><td>65.70</td><td>68.87</td><td>69.36</td><td>52.58</td><td>52.58</td><td>0.001</td></tr>
    <tr><td>RFNet</td><td>38.11</td><td>74.47</td><td>36.26</td><td>36.98</td><td>76.72</td><td>73.49</td><td>39.84</td><td>42.09</td><td>42.85</td><td>77.65</td><td>78.26</td><td>44.52</td><td>74.55</td><td>76.84</td><td>76.65</td><td>59.29</td><td>59.29</td><td>0.006</td></tr>
    <tr><td>mmFormer</td><td>37.19</td><td>75.37</td><td>32.45</td><td>31.59</td><td>74.47</td><td>76.30</td><td>38.76</td><td>40.26</td><td>41.09</td><td>76.83</td><td>79.53</td><td>43.01</td><td>77.17</td><td>74.88</td><td>77.69</td><td>58.44</td><td>58.44</td><td>0.002</td></tr>
    <tr><td><strong>PRISMS</strong></td><td>46.41</td><td>78.92</td><td>37.24</td><td>37.98</td><td>80.93</td><td>80.77</td><td>43.48</td><td>47.23</td><td>49.12</td><td>82.05</td><td>82.19</td><td>49.79</td><td>80.56</td><td>80.82</td><td>80.61</td><td>63.87</td><td><strong>63.87</strong></td><td>--</td></tr>
  </tbody>
</table>



---


<table>
  <caption>Tabelle 2 BraTS2020</caption>
  <thead>
    <tr><th rowspan="4">Typ</th><th rowspan="4">Model</th><th>Flair</th><th>○</th><th>○</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>○</th><th>●</th><th>●</th><th>●</th><th>●</th><th>●</th><th>●</th><th>●</th><th rowspan="4">Avg.</th><th rowspan="4">p‑value</th></tr>
    <tr><th>T1</th><th>○</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>●</th><th>●</th><th>○</th></tr>
    <tr><th>T1ce</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>●</th><th>●</th><th>●</th><th>●</th></tr>
    <tr><th>T2</th><th>●</th><th>○</th><th>○</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>○</th><th>●</th><th>○</th><th>○</th><th>●</th><th>●</th><th>●</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="6">WT</td><td>HeMIS</td><td>76.07</td><td>58.26</td><td>51.23</td><td>79.52</td><td>80.69</td><td>64.69</td><td>83.74</td><td>79.46</td><td>84.63</td><td>83.56</td><td>85.55</td><td>85.97</td><td>87.26</td><td>82.35</td><td>88.00</td><td>78.07</td><td>78.07</td><td>&lt;0.001</td></tr>
    <tr><td>U‑HVED</td><td>80.02</td><td>62.31</td><td>55.13</td><td>79.88</td><td>82.30</td><td>64.74</td><td>83.05</td><td>81.59</td><td>86.63</td><td>84.73</td><td>85.80</td><td>87.16</td><td>87.78</td><td>82.77</td><td>88.06</td><td>79.46</td><td>79.46</td><td>&lt;0.001</td></tr>
    <tr><td>RobustMSeg</td><td>83.00</td><td>71.61</td><td>67.73</td><td>82.42</td><td>86.10</td><td>76.31</td><td>87.19</td><td>85.64</td><td>88.34</td><td>87.75</td><td>88.69</td><td>89.02</td><td>89.28</td><td>86.49</td><td>89.57</td><td>83.94</td><td>83.94</td><td>&lt;0.001</td></tr>
    <tr><td>RFNet</td><td>86.55</td><td>76.74</td><td>76.82</td><td>86.97</td><td>88.16</td><td>80.50</td><td>89.37</td><td>87.97</td><td>89.63</td><td>89.43</td><td>90.24</td><td>90.42</td><td>90.38</td><td>88.54</td><td>90.89</td><td>86.84</td><td>86.84</td><td>&lt;0.001</td></tr>
    <tr><td>mmFormer</td><td>85.37</td><td>74.86</td><td>74.91</td><td>86.27</td><td>87.35</td><td>79.61</td><td>88.91</td><td>87.19</td><td>89.04</td><td>89.03</td><td>89.61</td><td>89.96</td><td>89.82</td><td>87.85</td><td>90.31</td><td>86.01</td><td>86.01</td><td>&lt;0.001</td></tr>
    <tr><td><strong>PRISMS</strong></td><td>87.20</td><td>78.80</td><td>79.15</td><td>88.70</td><td>88.67</td><td>82.40</td><td>90.30</td><td>88.34</td><td>90.56</td><td>90.38</td><td>91.00</td><td>90.91</td><td>91.16</td><td>89.01</td><td>91.36</td><td><strong>87.86</strong></td><td>--</td></tr>
    <tr><td rowspan="6">TC</td><td>HeMIS</td><td>56.71</td><td>66.35</td><td>34.81</td><td>53.31</td><td>76.34</td><td>70.49</td><td>60.29</td><td>59.60</td><td>63.82</td><td>73.87</td><td>75.63</td><td>65.10</td><td>77.79</td><td>77.41</td><td>78.34</td><td>65.99</td><td>65.99</td><td>&lt;0.001</td></tr>
    <tr><td>U‑HVED</td><td>62.35</td><td>69.70</td><td>43.57</td><td>51.92</td><td>78.68</td><td>73.50</td><td>58.17</td><td>65.10</td><td>65.31</td><td>76.05</td><td>77.93</td><td>66.89</td><td>80.04</td><td>79.68</td><td>80.49</td><td>68.63</td><td>68.63</td><td>&lt;0.001</td></tr>
    <tr><td>RobustMSeg</td><td>63.87</td><td>77.95</td><td>53.29</td><td>57.28</td><td>83.55</td><td>81.51</td><td>67.01</td><td>69.65</td><td>69.35</td><td>81.93</td><td>82.47</td><td>70.64</td><td>83.17</td><td>84.38</td><td>83.39</td><td>73.96</td><td>73.96</td><td>&lt;0.001</td></tr>
    <tr><td>RFNet</td><td>69.85</td><td>81.72</td><td>64.78</td><td>68.82</td><td>84.75</td><td>82.42</td><td>73.38</td><td>72.03</td><td>73.70</td><td>85.46</td><td>84.62</td><td>74.15</td><td>85.47</td><td>84.10</td><td>85.09</td><td>78.02</td><td>78.02</td><td>0.018</td></tr>
    <tr><td>mmFormer</td><td>70.21</td><td>80.74</td><td>64.24</td><td>67.80</td><td>84.30</td><td>82.00</td><td>71.83</td><td>72.61</td><td>72.82</td><td>84.44</td><td>84.59</td><td>73.90</td><td>84.63</td><td>84.28</td><td>84.49</td><td>77.53</td><td>77.53</td><td>&lt;0.001</td></tr>
    <tr><td><strong>PRISMS</strong></td><td>72.31</td><td>81.85</td><td>66.75</td><td>72.20</td><td>84.62</td><td>83.70</td><td>74.44</td><td>73.56</td><td>75.42</td><td>85.54</td><td>85.82</td><td>76.14</td><td>85.27</td><td>84.90</td><td>85.43</td><td><strong>79.20</strong></td><td>--</td></tr>
    <tr><td rowspan="6">ET</td><td>HeMIS</td><td>30.52</td><td>61.70</td><td>12.47</td><td>26.25</td><td>68.60</td><td>65.40</td><td>29.57</td><td>32.66</td><td>35.87</td><td>67.56</td><td>68.16</td><td>36.73</td><td>68.21</td><td>70.01</td><td>69.39</td><td>49.54</td><td>49.54</td><td>&lt;0.001</td></tr>
    <tr><td>U‑HVED</td><td>37.30</td><td>65.77</td><td>19.95</td><td>19.32</td><td>69.56</td><td>67.70</td><td>28.84</td><td>38.79</td><td>38.18</td><td>68.03</td><td>70.21</td><td>39.07</td><td>70.94</td><td>70.11</td><td>72.41</td><td>51.75</td><td>51.75</td><td>&lt;0.001</td></tr>
    <tr><td>RobustMSeg</td><td>40.14</td><td>74.16</td><td>25.42</td><td>32.67</td><td>74.80</td><td>73.98</td><td>38.61</td><td>42.21</td><td>42.41</td><td>75.62</td><td>76.99</td><td>45.10</td><td>73.39</td><td>74.68</td><td>74.42</td><td>57.64</td><td>57.64</td><td>&lt;0.001</td></tr>
    <tr><td>RFNet</td><td>47.75</td><td>75.65</td><td>34.85</td><td>40.40</td><td>76.02</td><td>78.18</td><td>44.62</td><td>47.82</td><td>47.96</td><td>75.17</td><td>78.42</td><td>48.86</td><td>76.38</td><td>78.27</td><td>76.50</td><td>61.79</td><td>61.79</td><td>&lt;0.001</td></tr>
    <tr><td>mmFormer</td><td>46.12</td><td>76.45</td><td>34.78</td><td>38.39</td><td>75.29</td><td>77.12</td><td>41.17</td><td>48.07</td><td>47.98</td><td>77.22</td><td>77.13</td><td>48.77</td><td>76.36</td><td>76.69</td><td>78.11</td><td>61.31</td><td>61.31</td><td>&lt;0.001</td></tr>
    <tr><td><strong>PRISMS</strong></td><td>51.50</td><td>82.57</td><td>40.87</td><td>43.39</td><td>82.35</td><td>83.81</td><td>47.04</td><td>49.90</td><td>53.87</td><td>83.07</td><td>84.12</td><td>53.33</td><td>81.23</td><td>82.36</td><td>82.17</td><td><strong>66.77</strong></td><td>--</td></tr>
  </tbody>
</table>

---

<table>
  <caption>Tabelle 3 HD auf BraTS2018 und BraTS2020</caption>
  <thead>
    <tr><th>Typ</th><th>Methode</th><th>BraTS2018 (HD)</th><th>BraTS2020 (HD)</th></tr>
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

### 2) Qualitiativer Vergleich

![Qualitiatver Vergleich auf BraTS2018 und BraTS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/new_figures/Prisms%20%7C%20Framework/Qual_PRISMS_1820(2).svg)


![Segmentierungsergebnisse auf BraTS2018 und BraTS2020](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/new_figures/Prisms%20%7C%20Framework/qual_seg_PRISMS(1).svg)

---

Hier sind die Quellenangaben in einem einheitlichen, vollständigen Format:

**[1]** Dorent, R., Joutard, S., Modat, M., Ourselin, S., & Vercauteren, T. (2019). *Hetero‑Modal Variational Encoder‑Decoder for Joint Modality Completion and Segmentation*. In **Medical Image Computing and Computer Assisted Intervention – MICCAI 2019** (pp. 87–95). Springer. https://arxiv.org/abs/1907.11150
**[2]** Chen, C., Dou, Q., Jin, Y., Chen, H., Qin, J., & Heng, P.‑A. (2020). *Robust Multimodal Brain Tumor Segmentation via Feature Disentanglement and Gated Fusion*. _CoRR_, abs/2002.09708. arXiv:2002.09708 citeturn1search0

**[3]** Sun, L., Yang, K., Hu, X., Hu, W., & Wang, K. (2020). *Real‑time Fusion Network for RGB‑D Semantic Segmentation Incorporating Unexpected Obstacle Detection for Road‑driving Images*. _CoRR_, abs/2002.10570. https://github.com/dyh127/RFNet

---

**[A]** Havaei, M., Davy, A., Warde‑Farley, D., et al. (2016). *HeMIS: Hetero‑modal Image Segmentation*. In **Medical Image Computing and Computer Assisted Intervention – MICCAI 2016** (LNCS 9901, pp. 111–119).  https://arxiv.org/abs/1607.05194

**[B]** Dorent, R., Joutard, S., Modat, M., Ourselin, S., & Vercauteren, T. (2019). *Hetero‑Modal Variational Encoder‑Decoder for Joint Modality Completion and Segmentation*. https://arxiv.org/pdf/1907.11150

**[C]** Chen, C., Dou, Q., Jin, Y., Chen, H., Qin, J., & Heng, P.‑A. (2020). *Robust Multimodal Brain Tumor Segmentation via Feature Disentanglement and Gated Fusion*. https://arxiv.org/pdf/2002.09708

**[D]** Sun, L., Yang, K., Hu, X., Hu, W., & Wang, K. (2020). *Real‑time Fusion Network for RGB‑D Semantic Segmentation Incorporating Unexpected Obstacle Detection for Road‑driving Images*. https://arxiv.org/abs/2002.10570

**[E]** Zhang, Y., He, N., Yang, J., Li, Y., Wei, D., Huang, Y., Zhang, Y., He, Z., & Zheng, Y. (2022). *mmFormer: Multimodal Medical Transformer for Incomplete Multimodal Learning of Brain Tumor Segmentation*. In **Medical Image Computing and Computer Assisted Intervention – MICCAI 2022** (LNCS 13332, pp. 71–83). https://arxiv.org/abs/2206.02425
