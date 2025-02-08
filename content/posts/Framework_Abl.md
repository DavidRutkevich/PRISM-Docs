---
title: "PRISMS - Ablation des Frameworks"
description: "Eine teifgehende Ablation des Frameworks"
date: 2025-02-05
type: post
draft: false
translationKey: "prisms_abl_framework"
coffee: 1
tags: ['PRISMs','Ablation']
categories: ['Framework']
---


**Ablation Study**  
Im Folgenden präsentieren wir unsere Ablation-Experimente, in denen wir zentrale Module und Hyperparameter untersuchen. Anhand dieser Studien lässt sich erkennen, wie jedes einzelne Modul zum Gesamterfolg beiträgt und wie sensibel das Modell auf unterschiedliche Gewichtungen reagiert. Alle Versuche wurden auf der MUSES-Benchmark \cite{brodermann2024muses} durchgeführt, sofern nicht anders angegeben.

---

### 1. Einfluss einzelner Modulverluste
Wir beginnen mit einer Untersuchung, welche Auswirkung das Hinzunehmen der verschiedenen Verluste $L_{\text{sup}}, L_{\text{mad}}, L_{\text{umd}}$ und $L_{\text{cmd}}$ hat. Dabei nehmen wir jeweils auf die in der Arbeit erläuterten Einstellungen Bezug (beispielsweise wird das *Parallel Multimodal Learning* (PML) genutzt, und der multimodale Lehrer ist bereits vortrainiert).

<table>
  <caption>Ablation verschiedener Verlust-Kombinationen auf dem MUSES-Datensatz. Die Spalten F, E, L entsprechen Ein-Modus-Evaluationen (Frame, Event, LiDAR), während FE, FL, EL Zwei-Modus-Kombinationen bezeichnen. FEL steht für alle drei Modalitäten zusammen. Δ gibt die Abweichung gegenüber dem jeweiligen Baseline-Wert an.</caption>
  <thead>
    <tr>
      <th>Verlust-Kombination</th>
      <th>F</th><th>Δ ↑</th>
      <th>E</th><th>Δ ↑</th>
      <th>L</th><th>Δ ↑</th>
      <th>FE</th><th>Δ ↑</th>
      <th>FL</th><th>Δ ↑</th>
      <th>EL</th><th>Δ ↑</th>
      <th>FEL</th><th>Δ ↑</th>
      <th>Mean</th><th>Δ ↑</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>L<sub>sup</sub></td>
      <td>43.69</td><td>-</td>
      <td>22.35</td><td>-</td>
      <td>32.14</td><td>-</td>
      <td>44.58</td><td>-</td>
      <td>48.53</td><td>-</td>
      <td>35.40</td><td>-</td>
      <td>48.35</td><td>-</td>
      <td>39.29</td><td>-</td>
    </tr>
    <tr>
      <td>L<sub>sup</sub> + λL<sub>mad</sub></td>
      <td>43.71</td><td style="background-color: #FFD580;">+0.02</td>
      <td>23.00</td><td style="background-color: #FFA07A;">+0.65</td>
      <td>34.70</td><td style="background-color: #FF6347;">+2.56</td>
      <td>44.18</td><td style="background-color: #87CEFA;">-0.40</td>
      <td>49.13</td><td style="background-color: #FFA07A;">+0.60</td>
      <td>37.23</td><td style="background-color: #FF4500;">+1.83</td>
      <td>48.79</td><td style="background-color: #FFD580;">+0.44</td>
      <td>40.11</td><td style="background-color: #FF6347;">+0.82</td>
    </tr>
    <tr>
      <td>L<sub>sup</sub> + λL<sub>mad</sub> + αL<sub>umd</sub></td>
      <td>45.82</td><td style="background-color: #FF6347;">+2.13</td>
      <td>19.26</td><td style="background-color: #87CEEB;">-3.09</td>
      <td>31.79</td><td style="background-color: #87CEFA;">-0.35</td>
      <td>45.88</td><td style="background-color: #FFA07A;">+1.30</td>
      <td>51.11</td><td style="background-color: #FF4500;">+2.58</td>
      <td>33.56</td><td style="background-color: #87CEEB;">-1.84</td>
      <td>50.60</td><td style="background-color: #FF6347;">+2.25</td>
      <td>39.72</td><td style="background-color: #FF4500;">+0.43</td>
    </tr>
    <tr>
      <td>L<sub>sup</sub> + λL<sub>mad</sub> + αL<sub>umd</sub> + βL<sub>cmd</sub></td>
      <td><strong>46.01</strong></td><td style="background-color: #FF4500;"><strong>+2.32</strong></td>
      <td><strong>19.57</strong></td><td style="background-color: #87CEFA;"><strong>-2.78</strong></td>
      <td><strong>32.13</strong></td><td style="background-color: #FFD580;"><strong>-0.01</strong></td>
      <td><strong>46.29</strong></td><td style="background-color: #FFA07A;"><strong>+1.71</strong></td>
      <td><strong>51.25</strong></td><td style="background-color: #FF4500;"><strong>+2.72</strong></td>
      <td><strong>35.21</strong></td><td style="background-color: #FFD580;"><strong>-0.21</strong></td>
      <td><strong>51.14</strong></td><td style="background-color: #FF6347;"><strong>+2.79</strong></td>
      <td><strong>40.23</strong></td><td style="background-color: #FF4500;"><strong>+0.94</strong></td>
    </tr>
  </tbody>
</table>


Aus \autoref{tab:Ablation-Losses} wird ersichtlich:
- Die Kombination $\mathcal{L}_{sup} + \lambda \mathcal{L}_{mad}$ verbessert zunächst die Performance bei Event (E, +0.65) und LiDAR (L, +2.56), hebt jedoch RGB (F) nur minimal an.  
- Fügt man zusätzlich den **Unimodal Distillation**-Term ($\alpha L_{\text{umd}}$) hinzu, wird besonders der Single-Modus RGB stark gesteigert (+2.13), allerdings leiden Event und LiDAR (Übergewichtung eines „einfach zu lernenden“ Modalpfads).  
- Erst durch den **Cross-Modal Distillation**-Term ($\beta L_{\text{cmd}}$) wird die Balance im System wiederhergestellt: FL und FEL weisen weitere Steigerungen auf (+2.72 bzw. +2.79), während Event (E) sich gegenüber dem vorigen Setting geringfügig erholt.

---

### 2. Hyperparameter-Sensitivität
Für den Verlust $L_{\text{mad}}$, $L_{\text{umd}}$ und $L_{\text{cmd}}$ wurden verschiedene Skalarfaktoren ($\lambda, \alpha, \beta$) getestet. Wie in \autoref{tab:Ablation-Losses} ersichtlich, kann eine zu starke Gewichtung einzelner Distillationspfade zu Performance-Einbrüchen bei anderen Modalitäten führen. Optimalwerte liegen hier in Bereichen, wo weder unimodale noch cross-modale Distillation übertrieben stark gewichtet werden.

---

### 3. Fusion vs. Individuelle Distillation
In weiteren Experimenten prüften wir, ob eine **Distillation auf bereits fusionierten Features** besser funktioniert (ähnlich \cite{chen2021spatial}). Hierzu haben wir die Feature-Maps sämtlicher Modalitäten gemittelt und erst dann eine Wissensdistillation durchgeführt. Die Resultate belegen jedoch, dass dieser **Fusion-first**-Ansatz dem Modell teils schadet, da wichtige Unterschiede zwischen den Modalitäten verschleiert werden. Eine **individuelle** Behandlung jeder Modalität bzw. die Cross-Modal-Kopplung mit $\mathcal{L}_{cmd}$ erweisen sich als effektiver.

---

### 4. Fazit der Ablation
Die Ablationsanalyse untermauert die Notwendigkeit aller Kernkomponenten:
1. **Modality-Agnostic Distillation** ($L_{\text{mad}}$) gleicht fehlende Modalitäten aus,  
2. **Unimodal Distillation** ($L_{\text{umd}}$) stärkt schwächere Einzelpfade, kann aber zu Bias führen,  
3. **Cross-Modal Distillation** ($L_{\text{cmd}}$) mitigiert diesen Bias und stabilisiert die Gesamtleistung.

Eine reine **Fusion-first Distillation** erweist sich als suboptimal, da sie kaum Rücksicht auf modalitätsspezifische Lernkurven nimmt. Somit ergibt sich eine robuste **Anymodal**-Konfiguration nur durch das Zusammenspiel der Distillationsformen und einer feinen Abstimmung ihrer Gewichtungen.