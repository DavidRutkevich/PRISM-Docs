---
title: "PRISM – Einfach erklärt"
description: "Übersichtliche Erläuterung zur Problemdefinition, Multi-Uni-Selbstdistillation und präferenzbasierten Regulierung in PRISM."
date: 2025-02-05
math: true
---

### 1. Problemdefinition

Stellen wir uns vor, wir haben einen **multimodalen Datensatz** mit $N$ Beispielen (z. B. Patienten). Für jedes Beispiel gibt es bis zu $M$ verschiedene **Modalitäten** (etwa unterschiedliche MRT-Sequenzen). Formal schreiben wir:

- **Eingabedaten**: $\{x^m_n\}_{m=1,\dots,M}$  
- **Labels**: $\{y^m_n\}_{m=1,\dots,M}$

Bei einer *Segmentierungsaufgabe* möchte man jedem Pixel im Bild (bzw. Volumen) eine Klasse (z. B. „Tumor“ oder „Gesund“) zuordnen. Häufig existiert **eine gemeinsame** Label-Annotierung, d. h. alle Modalitäten eines Patienten teilen sich die gleichen Pixel-Labels. Oder mathematisch:


$$
y_{n,i} = y^1_{n,i} = y^2_{n,i} = \dots = y^M_{n,i}.
$$


Doch in der Praxis **fehlen** oft Modalitäten. Um das erfassbar zu machen, definieren wir eine **Indikationsmatrix** $C$ mit den Einträgen:


$$
C_{nm} =
\begin{cases}
1, & \text{wenn Modalität $m$ für Beispiel $n$ vorliegt},\\
0, & \text{wenn Modalität $m$ fehlt}.
\end{cases}
$$


Die *Fehlrate* einer Modalität $m$ sagt dann aus, wie oft diese im Datensatz fehlt. Wir nennen sie $ \textit{FR}^m $:


$$
\textit{FR}^m = \frac{N - \sum_{n=1}^N C_{nm}}{N}.
$$


Wenn $\textit{FR}^m$ nahe 1 ist, dann ist diese Modalität fast immer abwesend.  

---

### 2. Unvollständige Multimodale Segmentierungs-Baseline

Viele aktuelle Forschungsarbeiten (z. B. \cite{dingDyh127RFNet2024}) verwenden ein Schema, bei dem **jeder Modalität** ein eigener Encoder $E_m$ zugeordnet wird. Anschließend werden die Feature-Karten in einem **gemeinsamen Decoder** $D_f$ zusammengeführt. Damit kann das Modell alle verfügbaren Informationen fusionieren.

- **Notation**:  
  $ z^l_n = D_f^l(x_n) $ bezeichnet die zusammengeführten Merkmale aus **allen** Modalitäten (falls vorhanden).  
  $ z^{m,l}_n = D_f^l(x^m_n) $ bezeichnet die Merkmale nur der Modalität $m$.  

Für die Segmentierung nutzt man häufig eine Kombination aus **Dice**- und **Cross-Entropy**-Verlust. Man kann es sich vereinfacht so vorstellen:


$$
L_{\text{Obj.}} 
= 
\sum_{l=0}^{L}
  \ell_{\mathit{dice+ce}}(\text{Up}_{2^l}(z^l_n),\,y_n)
+
\sum_{m=1}^{M}
  \ell_{\mathit{dice+ce}}(z^{m,0}_n,\,y_n).
$$


- Erster Teil (links): **Segmentierung** selbst (ggf. mit „Deep Supervision“).  
- Zweiter Teil (rechts): **Regulierung** jedes Encoders, damit jeder Modalitätspfad solide trainiert wird.

**UTD-Problem**: Fehlt eine Modalität ($C_{nm}=0$), erhält diese beim Training kein Update. Modalitäten mit hoher Fehlrate ($\textit{FR}^m \approx 1$) werden somit kaum berücksichtigt, was zu einer *Ungleichverteilung* führt.  

---

### 3. Multi-Uni Selbstdistillierung

Um **alle** Modalitäten einzubeziehen, selbst wenn manche selten oder gar nicht da sind, setzt **PRISM** auf *Selbstdistillation*. Das heißt:

1. **Multimodaler Pfad** (Decoder) = „Lehrer“  
2. **Unimodale Pfade** (einzelne Encoder-Ausgänge) = „Schüler“

Damit kann das Modell sein eigenes Lehrer-Schüler-System in sich tragen, ohne dass wir ein zweites „Lehrer-Netz“ brauchen.

#### (a) Pixel-Ebene

Wir schauen uns die *Logits* der unimodalen Ausgänge $z_n^{m,l}$ an und vergleichen sie mit den Logits der multimodalen Fusion $z_n^l$. Das geschieht mittels Kullback-Leibler-Divergenz und einer Temperatur $\mu$. Einfach gesagt: Wir bringen die Vorhersage der Einzelfeeds (unimodale „Schüler“) näher an die Vorhersage des gemeinsamen „Lehrers“.


$$
L_{\text{pixel}}^m 
= 
\sum_{l=0}^{L}
  KL\Bigl[
    \sigma\bigl(z_n^{m,l}/\mu\bigr)
    \,\big\|\,
    \sigma\bigl(z_n^l/\mu\bigr)
  \Bigr].
$$


#### (b) Semantik-Ebene (Prototypen)

Neben Pixel-Logits können „globale“ Klassenzentren (Prototypen) helfen. Für jede Klasse $k$ mitteln wir die Features aller zugehörigen Pixel. So entsteht ein Lehrer-Prototyp $c^t_{n,k}$ und ein Schüler-Prototyp $c^{m,s}_{n,k}$. Je nachdem, wie sehr diese Prototypen abweichen, gibt es einen Distillations-Verlust:


$$
L_{\text{proto}}^m
=
\sum_i \sum_{k=1}^K
\Bigl\|
  S_{n,k}^{m,s}(i)
  -
  S_{n,k}^t(i)
\Bigr\|_2^2.
$$


Wenn ein unimodaler Pfad weit weg von der „multimodalen Sicht“ ist, wird er im Training stärker angepasst.

---

### 4. Präferenzbewusste Regularisierung

Leider kann es passieren, dass Modalitäten mit wenigen Daten trotz Distillation untergehen. Deshalb braucht es noch eine „Präferenz“:

- Wir messen, wie gut oder schlecht eine Modalität $m$ im Vergleich zu den anderen abschneidet.  
- *Vernachlässigte* Modalitäten werden gezielt gestärkt.  
- *Dominante* Modalitäten werden eher gedrosselt, damit sie nicht alle anderen überstimmen.

Dazu definiert PRISM eine **relative Präferenz** $RP_n^m$. Die Idee: Wenn Modalität $m$ bei Beispiel $n$ eine große Abweichung zum Lehrer hat, deutet das auf Vernachlässigung hin. Das wird mathematisch so ausgedrückt:


$$
RP_n^m 
= 
1 
- 
\frac{D_n^m}{\bar{D}_n}.
$$


- $D_n^m$ = semantische Distanz der Modalität $m$ zum Lehrer  
- $\bar{D}_n$ = Durchschnitt dieser Distanzen über alle verfügbaren Modalitäten

Liegt $RP_n^m < 0$, bekommt Modalität $m$ im Distillationsterm mehr Gewicht. Liegt $RP_n^m > 0$, wird sie etwas „gebremst“.  

---

### 5. Gesamter PRISM-Loss

Kombiniert man **Selbstdistillation** (Pixel + Prototyp) mit **Präferenzregulierung**, erhält man den finalen Trainings-Loss:


$$
L 
= 
L_{\text{seg}}
+
\sum_{\substack{m=1,\dots,M \\ C_{nm}=1}}
\Bigl(
  \gamma_1 \,E^m\, L_{\text{pixel}}^m
  +
  \gamma_2 \,\omega_n^m\, L_{\text{proto}}^m
\Bigr),
$$


- $L_{\text{seg}}$: Standard-Segmentierung (Dice+CE).  
- $L_{\text{pixel}}^m$, $L_{\text{proto}}^m$: Distillation-Verluste (lokal + semantisch).  
- $E^m, \omega_n^m$: Dynamische Faktoren, die die Lernrate für Modalität $m$ anpassen.  
- $\gamma_1, \gamma_2$: Gewichtungen.

---

### 6. Plug-and-Play-Charakter

**PRISM** ist so entwickelt, dass man es leicht zu bestehenden Segmentierungsnetzwerken (z. B. U-Net, Transformers) hinzufügen kann. Das Grundprinzip:

1. **Encoder** pro Modalität (ggf. wegfallen, wenn Modalität nicht vorhanden).  
2. **Ein gemeinsamer Decoder** als Lehrer-Pfad.  
3. **Unimodale Ausgänge** als Schüler-Pfade.  
4. **Distillation** (Pixel & Prototyp) + **Präferenz** (dynamische Rebalancierung).

Dadurch kann PRISM auch bei vielen fehlenden Modalitäten stabil bleiben.

---

### 7. Relevanz in der Praxis

In der realen Welt – sei es Medizin, Industrie oder IoT – liegen oft **nicht** immer alle Datentypen komplett vor. Dies nennt man „unvollständige Multimodalität“. Hier glänzt PRISM:

- Es **muss** keine Vollständigkeit annehmen (kein externes Lehrer-Netz).  
- Die **lernratebasierte** Präferenzsteuerung verhindert, dass seltene Modalitäten ignoriert werden.  
- Das Verfahren ist **modular** und simpel in bestehende Prozesse integrierbar.

---

### 8. Fazit

**PRISM** liefert eine clevere Kombination aus *Selbstdistillation* und *Präferenzregulierung*, um unvollständige Datenquellen bestmöglich zu nutzen. So kann jedes Teilmodell (jede Modalität) das globale Wissen erwerben, ohne dass man teure vollmodale Datensätze oder extra Lehrernetze braucht.  

Kurz: **Ein praktischer Ansatz, um trotz fehlender Daten zuverlässige Segmentierungen zu ermöglichen.**
