---
title: "PRISM – Methodik"  
description: "Die detailierte Erklärung des PRISM Moduls."  
date: 2025-02-05  
math: true
---

![Allgemeiness Beispiel von PRISM](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/new_figures/Prism%20%7C%20Modul/prism_example_method.webp)

## Multimodaler Trainingsdatensatz und Baseline

Gegeben sei ein multimodaler Trainingsdatensatz mit $N$ Proben, wobei jede Probe $M$ unterschiedliche Modalitäten (z. B. medizinische Bildserien, Audio-/Videodaten oder Sensordaten) umfasst. Für eine positive ganze Zahl $P$ definieren wir

$$
[P] = \{1,2,\dots,P\}.
$$

Jede Trainingsprobe wird durch das Paar 

$$
\Bigl(\{x_n^m\}_{m\in[M]},\,\{y_n^m\}_{m\in[M]}\Bigr)
$$

beschrieben, wobei $n\in[N]$ den Probenindex kennzeichnet. Für eine Segmentierungsaufgabe soll jedem Pixel ein individuelles Label aus $K$ Kategorien zugewiesen werden. Zur Notation definieren wir daher $(x_{n,i}^m,\, y_{n,i}^m)$ als den rohen Input und das zugehörige Label des $i$-ten Pixels, das zur Modalität $m$ und zur Probe $n$ gehört. In vielen multimodalen Segmentierungsaufgaben teilen alle Modalitäten ein gemeinsames Label, sodass gilt:

$$
y_{n,i} = y_{n,i}^1 = y_{n,i}^2 = \dots = y_{n,i}^M.
$$

Um den häufig vorkommenden Fall unvollständiger Daten – etwa im medizinischen Kontext – abzubilden, definieren wir eine Indikationsmatrix

$$
C \in \mathbb{R}^{N\times M},
$$

wobei der Eintrag $C_{nm}$ den Status der Verfügbarkeit der Modalität $m$ in Probe $n$ angibt (1, falls vorhanden, 0 sonst). Die Fehlrate einer Modalität $m$ berechnen wir als

$$
\textit{FR}^m = \frac{N-\sum_{n\in[N]} C_{nm}}{N}.
$$

Dabei stellen wir sicher, dass $\textit{FR}^m \in [0,1)$ gilt, sodass für jede Modalität mindestens eine Probe verfügbar ist.

In der Regel werdenfür jede Modalität $m$ separate, modalitätsspezifische Encoder $E_m$ verwendet, deren Ausgaben in einem gemeinsamen Fusionsdecoder $D_f$ fusioniert werden – ähnlich dem U-Net-Ansatz, bei dem Encoder- und Decoderbereiche mittels Skip-Verbindungen verknüpft sind. Zur Vereinheitlichung der Notation definieren wir für die Features in Layer $l$:

$$
z^l_n = D^l_f(x_n) \quad (\text{fusionierte Merkmale aller verfügbaren Modalitäten})
$$

und

$$
z^{m,l}_n = D^l_f(x^m_n) \quad (\text{Merkmale der einzelnen Modalität } m).
$$

Wir setzen $l=0$ als Ausgabeschicht. Das Gesamttrainingsziel der Baseline gliedert sich in zwei Teile:

$$
\mathcal{L}_{task} = \sum_{l=0}^{L}\ell_{dice+ce}\Bigl({\bf Up}_{2^{l}}(z^{l}_n),\, y_n\Bigr) + \sum_{m\in[M]} \ell_{dice+ce}(z^{m,0}_n,\, y_n).
$$


Hierbei steht $\ell_{dice+ce}$ für den kombinierten Dice- und gewichteten Cross-Entropy-Loss, während $Up_{2^l}$ ein $2^{l}$-faches Upsampling durchführt. Im Fall der unvollständigen Trainingsphase (UTD) werden nur die Modalitäten berücksichtigt, für die $C_{nm} = 1$ gilt.
Ein Problem hierbei ist, dass Modalitäten mit hohen Fehlraten $\textit{FR}^m \approx 1$ seltener geupdatet werden – was zu einem erheblichen Ungleichgewicht führen kann.

## Multi-Uni Self-Distillation (PRISM)

Angelehnt an die Prinzipien der Wissensdistillation [1] – bei der das „dunkle Wissen“ eines großen Modells an ein kleineres Modell weitergegeben wird – wird hier eine Self-Distillation innerhalb eines einheitlichen Netzwerks durchgeführt. Unser Ansatz, den wir **PRISM** nennen, transferiert multimodales Wissen direkt auf die uni-modalen Teilmodelle. Dies erlaubt es, das in der gesamten Modalität enthaltene Wissen zu nutzen, ohne einen separaten vollmodalen Lehrer trainieren zu müssen.

### Pixelweise Self-Distillation

Da die Segmentierung als Pixelklassifikationsaufgabe formuliert werden kann, zielen wir darauf ab, die Vorhersagen einzelner Pixel zwischen dem multimodalen (Lehrer-) und dem uni-modalen (Schüler-) Zweig anzugleichen. Dabei wird die Logit-Ausrichtung mittels der Kullback-Leibler-Divergenz verfolgt:

$$
\mathcal{L}_{pixel}^m = \sum_{l=0}^{L} KL\!\Bigl[\sigma\!\Bigl(\frac{z_n^{m,l}}{\mu}\Bigr)\,\Big\|\, \sigma\!\Bigl(\frac{z^l_n}{\mu}\Bigr)\Bigr],
$$

wobei $\sigma$ die Softmax-Funktion, $\mu$ der Temperaturparameter und $KL$ die Kullback-Leibler-Divergenz bezeichnet. Diese pixelweise Distillation unterstützt die Robustheit gegenüber unbalancierten Modalitäten.

### Semantische Self-Distillation

Neben der lokalen, pixelweisen Information soll auch das globale, klassenbezogene Wissen übertragen werden. Hierzu wird für jede Probe und Klasse ein Prototyp berechnet. Konkret definieren wir für Klasse $k$ und Probe $n$ den multimodalen Lehrer-Prototyp $c_{n,k}^t$ und den uni-modalen Schüler-Prototyp $c_{n,k}^{m,s}$ als

$$
c_{n,k}^t = \frac{\sum_{i} z^0_{n,i}\,\mathbf{1}[y_{n,i}=k]}{\sum_{i} \mathbf{1}[y_{n,i}=k]}, \quad
c_{n,k}^{m,s} = \frac{\sum_{i} z^{m,0}_{n,i}\,\mathbf{1}[y_{n,i}=k]}{\sum_{i} \mathbf{1}[y_{n,i}=k]}.
$$

Hierbei steht $\mathbf{1}[\cdot]$ für die Indikatorfunktion. Zur Bewertung der Repräsentationsqualität berechnen wir Kosinus-Ähnlichkeiten zwischen den Pixelmerkmalen und den Prototypen:

$$
S^t_{n,k} = \sum_i \text{Cos}\bigl(z^0_{n,i},\, c^t_{n,k}\bigr), \quad
S^{m,s}_{n,k} = \sum_i \text{Cos}\bigl(z^{m,0}_{n,i},\, c^{m,s}_{n,k}\bigr).
$$

Diese Maßzahlen erfassen die semantische Übereinstimmung zwischen den Features und den Prototypen. Anschließend wird die „semantische Wissenslücke“ zwischen dem Schüler und dem Lehrer quantifiziert als

$$
D^m_n = \sum_i \sum_{k\in[K]} \left\| S^{m,s}_{n,k}(i)-S^t_{n,k}(i) \right\|_2.
$$

Die Minimierung des daraus abgeleiteten Verlusts,

$$
\mathcal{L}_{proto}^m = \sum_i \sum_{k\in[K]} \left\| S^{m,s}_{n,k}(i)-S^t_{n,k}(i) \right\|_2^2,
$$

fördert eine ausgewogene, klassenbasierte Wissensübertragung, insbesondere zur Stärkung schwächerer Modalitäten.

## Prefärenzbewusste Regularisierung

Obwohl die Multi-Uni Self-Distillation (PRISM) das multimodale Wissen auf alle uni-modalen Zweige überträgt, können Modalitäten mit hohen Fehlraten dennoch im Training zurückfallen. Um dem entgegenzuwirken, wird eine dynamische, präferenzbasierte Regularisierung eingeführt.

**Relative Präferenz.**  
Hierzu berechnen wir für jede Probe $n$ die relative Präferenz der Modalität $m$ basierend auf der semantischen Wissenslücke $D^m_n$. Für eine gegebene Probe wird zunächst der Durchschnittswert

$$
\bar{D}_n = \frac{\sum_{m\in[M]\atop C_{nm}=1} D^m_n}{\sum_{m\in[M]} C_{nm}}
$$

ermittelt. Die relative Präferenz ergibt sich dann als

$$
RP_n^m = 1 - \frac{D^m_n}{\bar{D}_n}.
$$

Ein negativer Wert von $RP_n^m$ weist darauf hin, dass Modalität $m$ im Vergleich zu anderen benachteiligt ist.

**Rebalancing-Regularisierung.**  
Um schwächere Modalitäten zu fördern, werden zwei Regularisierungskomponenten eingeführt:

- ***Aufgaben****bezogene Regularisierung:* Ein Maskierungsfaktor 

  $$
  \omega_n^m = \mathbf{1}[RP_n^m < 0]
  $$

  wird gesetzt, um bei Modalitäten mit negativer relativer Präferenz den semantischen Distillations-Loss $\mathcal{L}_{proto}^m$ zusätzlich zu gewichten.

- ***Gradienten****bezogene Regularisierung:* Ein gradientengewichteter Koeffizient $\mathcal{E}^m$ (initialisiert als $\frac{1}{1-\textit{FR}^m}$) wird eingesetzt, um die pixelweise Self-Distillation dynamisch auszugleichen. Nach jeder Epoche wird der Durchschnitt der relativen Präferenz

  $$
  \bar{RP}^m = \frac{\sum_{n\in[N]} RP_n^m}{\sum_{n\in[N]} C_{nm}}
  $$

  berechnet und $\mathcal{E}^m$ mittels

  $$
  \mathcal{E}^m_{r+1} = \mathcal{E}^m_r - \gamma \cdot \bar{RP}^m
  $$

  aktualisiert ($\gamma$ bezeichnet die Update-Rate). Dadurch wird bei Modalitäten mit negativer $\bar{RP}^m$ die Lernrate erhöht und bei Modalitäten mit positiver $\bar{RP}^m$ entsprechend verringert.

## Gesamtoptimierungsziel

Kombiniert man die oben vorgestellten Komponenten der Multi-Uni Self-Distillation (PRISM) und der präferenzbasierten Regularisierung, ergibt sich das Gesamttrainingsziel zu

$$
\mathcal{L} = \mathcal{L}_{seg} + \sum_{m\in[M]\atop C_{nm}=1} \Bigl( \lambda_1\,\mathcal{E}^m\,\mathcal{L}_{pixel}^m + \lambda_2\,\omega_n^m\,\mathcal{L}_{proto}^m \Bigr),
$$

wobei $\lambda_1$ und $\lambda_2$ als Hyperparameter die Gewichtung der pixelweisen bzw. semantischen Distillation steuern.


–––
### Quellen

[1] Hinton, G., Vinyals, O., & Dean, J. (2015). [Distilling the Knowledge in a Neural Network](https://arxiv.org/abs/1503.02531). *arXiv preprint arXiv:1503.02531*.