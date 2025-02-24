---

title: "PRISM – erklärt"  
description: "Übersichtliche Erläuterung zur Problemdefinition, Multi-Uni-Selbstdistillation und präferenzbasierten Regulierung in PRISM."  
date: 2025-02-05  
math: true  

---

### 1. Problemdefinition

Es liegt ein **multimodaler Datensatz** mit \(N\) Beispielen (z. B. Patienten) vor, wobei für jedes Beispiel bis zu \(M\) verschiedene **Modalitäten** (etwa unterschiedliche MRT-Sequenzen) verfügbar sein können. Formal werden die Eingabedaten wie folgt notiert:

- **Eingabedaten**: \(\{x^m_n\}_{m=1,\dots,M}\)  
- **Labels**: \(\{y^m_n\}_{m=1,\dots,M}\)

Bei einer Segmentierungsaufgabe soll jedem Pixel (bzw. Voxel) eines Bildes oder Volumens eine Klasse (z. B. „Tumor“ oder „Gesund“) zugeordnet werden. Häufig wird eine **gemeinsame** Label-Annotierung verwendet, d. h. alle Modalitäten eines Beispiels teilen dieselben Pixel-Labels. Dies wird durch folgende Gleichung ausgedrückt:

\[
y_{n,i} = y^1_{n,i} = y^2_{n,i} = \dots = y^M_{n,i}.
\]

Da in der Praxis jedoch häufig Modalitäten fehlen, wird eine **Indikationsmatrix** \(C\) definiert, deren Einträge wie folgt lauten:

\[
C_{nm} =
\begin{cases}
1, & \text{wenn Modalität \(m\) für Beispiel \(n\) vorliegt},\\[1mm]
0, & \text{wenn Modalität \(m\) fehlt}.
\end{cases}
\]

Die *Fehlrate* einer Modalität \(m\) wird durch

\[
\textit{FR}^m = \frac{N - \sum_{n=1}^N C_{nm}}{N}
\]

beschrieben. Ein Wert von \(\textit{FR}^m\) nahe 1 zeigt an, dass diese Modalität nahezu immer fehlt.

---

### 2. Unvollständige Multimodale Segmentierungs-Baseline

In zahlreichen aktuellen Forschungsarbeiten (z. B. RFNet) wird ein Schema verwendet, bei dem jeder Modalität ein eigener Encoder \(E_m\) zugeordnet wird. Anschließend werden die resultierenden Feature-Karten im **gemeinsamen Decoder** \(D_f\) fusioniert, sodass alle verfügbaren Informationen integriert werden können.

- **Notation**:  
  \( z^l_n = D_f^l(x_n) \) bezeichnet die zusammengeführten Merkmale aller Modalitäten (sofern vorhanden).  
  \( z^{m,l}_n = D_f^l(x^m_n) \) bezeichnet die Merkmale der einzelnen Modalität \(m\).

Für die Segmentierung wird häufig eine Kombination aus **Dice-** und **Cross-Entropy-Verlust** verwendet. Dies kann vereinfacht dargestellt werden als:

\[
L_{\text{Obj.}} = 
\sum_{l=0}^{L}
  \ell_{\mathit{dice+ce}}(\text{Up}_{2^l}(z^l_n),\,y_n) + \sum_{m=1}^{M}\ell_{\mathit{dice+ce}}(z^{m,0}_n,\,y_n).
\]

- Der erste Term bezieht sich auf die Segmentierungsleistung des gesamten Netzwerks (ggf. mit „Deep Supervision“).  
- Der zweite Term reguliert die Leistung der einzelnen Encoders, sodass jeder Modalitätspfad adäquat trainiert wird.

Im **UTD-Problem** führt das Fehlen einer Modalität (\(C_{nm}=0\)) dazu, dass kein Update für diesen Pfad erfolgt. Modalitäten mit hoher Fehlrate (\(\textit{FR}^m \approx 1\)) werden somit kaum berücksichtigt, was zu einer ungleichen Behandlung der Modalitäten führt.

---

### 3. Multi-Uni Selbstdistillierung

Um alle Modalitäten zu berücksichtigen – selbst wenn einzelne selten oder gar nicht vorhanden sind – wird in PRISM auf eine *Selbstdistillation* gesetzt. Dabei übernimmt das multimodale Netzwerk die Funktion eines „Lehrers“ für die einzelnen unimodalen Pfade (Schüler), sodass ein integriertes Lehrer-Schüler-System entsteht, ohne dass ein separates Lehrermodell benötigt wird.

#### (a) Pixel-Ebene

Die *Logits* der unimodalen Ausgänge \(z_n^{m,l}\) werden mit denen der multimodalen Fusion \(z_n^l\) verglichen. Dieser Vergleich erfolgt über die Kullback-Leibler-Divergenz, wobei eine Temperatur \(\mu\) zur Glättung der Softmax-Ausgaben verwendet wird:

\[
L_{\text{pixel}}^m = 
\sum_{l=0}^{L}
  KL\Bigl[
    \sigma\Bigl(\frac{z_n^{m,l}}{\mu}\Bigr)
    \,\Big\|\,
    \sigma\Bigl(\frac{z_n^l}{\mu}\Bigr)
  \Bigr].
\]

Durch diesen Loss wird angestrebt, dass die Vorhersagen der unimodalen Pfade in die Nähe der multimodalen Ausgabe rücken.

#### (b) Semantik-Ebene (Prototypen)

Auf der semantischen Ebene werden für jede Klasse \(k\) Prototypen berechnet. Dabei wird der Lehrer-Prototyp \(c^t_{n,k}\) als Durchschnitt der Features aller Pixel mit Label \(k\) und der Schüler-Prototyp \(c^{m,s}_{n,k}\) analog berechnet. Anhand der Kosinus-Ähnlichkeit zwischen den Pixelfeatures und den jeweiligen Prototypen wird ein semantischer Distillations-Loss definiert:

\[
L_{\text{proto}}^m=
\sum_i \sum_{k=1}^K \Bigl\|
  S_{n,k}^{m,s}(i) -
  S_{n,k}^t(i)
\Bigr\|_2^2.
\]

Dieser Loss zwingt die unimodalen Repräsentationen dazu, näher an die globale, multimodale Klassenzentrierung zu rücken.

---

### 4. Präferenzbewusste Regularisierung

Es kann vorkommen, dass Modalitäten mit wenigen Daten trotz Distillation vernachlässigt werden. Um diesem Problem entgegenzuwirken, wird eine **relative Präferenz** \(RP_n^m\) definiert, die misst, wie gut eine Modalität im Vergleich zu anderen abschneidet. Dabei wird die semantische Distanz \(D_n^m\) der Modalität \(m\) zum Lehrer mit dem Durchschnittswert \(\bar{D}_n\) aller verfügbaren Modalitäten verglichen:

\[
RP_n^m = 
1 - 
\frac{D_n^m}{\bar{D}_n}.
\]

Liegt \(RP_n^m < 0\), wird der Einfluss dieser Modalität durch zusätzliche Gewichtung verstärkt, während bei \(RP_n^m > 0\) der Einfluss reduziert wird.

---

### 5. Gesamter PRISM-Loss

Die finale Loss-Funktion von PRISM kombiniert den Standard-Segmentierungs-Loss mit den Distillations- und Regularisierungstermen:

\[
L = 
L_{\text{seg}} +
\sum_{\substack{m=1,\dots,M \\ C_{nm}=1}}
\Bigl(
  \gamma_1 \,E^m\, L_{\text{pixel}}^m +
  \gamma_2 \,\omega_n^m\, L_{\text{proto}}^m
\Bigr),
\]

wobei  
- \(L_{\text{seg}}\) den Standard-Segmentierungs-Loss (Dice + Cross-Entropy) darstellt,  
- \(L_{\text{pixel}}^m\) und \(L_{\text{proto}}^m\) die Distillations-Verluste auf Pixel- und Semantikebene sind,  
- \(E^m\) und \(\omega_n^m\) dynamische Faktoren zur Anpassung der Lernrate bzw. zur Maskierung vernachlässigter Modalitäten darstellen,  
- \(\gamma_1\) und \(\gamma_2\) als Gewichtungsparameter fungieren.

---

### 6. Plug-and-Play-Charakter

PRISM wurde so konzipiert, dass es problemlos in bestehende Segmentierungsnetzwerke (wie U-Net oder transformerbasierte Modelle) integriert werden kann. Die Vorgehensweise umfasst:

1. Zuweisung eines **Encoders** pro Modalität, sofern vorhanden.  
2. Verwendung eines **gemeinsamen Decoders** als Lehrer-Pfad.  
3. Bereitstellung der **unimodalen Ausgänge** als Schüler-Pfade.  
4. Anwendung der **Selbstdistillation** (auf Pixel- und Prototyp-Ebene) kombiniert mit der **präferenzbasierten Regularisierung**.

Diese modulare Herangehensweise gewährleistet, dass auch bei vielen fehlenden Modalitäten stabile Ergebnisse erzielt werden.

---

### 7. Relevanz in der Praxis

In der realen Anwendung – etwa in der klinischen Bildverarbeitung, Industrie oder im IoT – liegen oft nicht alle Datentypen vollständig vor. Das Phänomen der unvollständigen Multimodalität wird in der Praxis häufig beobachtet. PRISM bietet hier folgende Vorteile:

- Es wird nicht von vollständigen Daten ausgegangen, sodass ein separates, vollmodal trainiertes Lehrer-Netz entfällt.  
- Die dynamische Anpassung der Lernraten über die relative Präferenz verhindert, dass seltene Modalitäten ignoriert werden.  
- Durch den modularen Aufbau lässt sich PRISM einfach in bestehende Systeme integrieren.

---

### 8. Fazit

PRISM kombiniert *Selbstdistillation* und *präferenzbasierte Regularisierung* zu einem Ansatz, der unvollständige multimodale Datensätze bestmöglich nutzt. Dabei werden lokale (pixelweise) und globale (prototypbasierte) Repräsentationen so angeglichen, dass jedes Teilmodell von der gesamten Information profitiert – ohne dass separate vollmodale Datensätze oder Lehrer-Netze benötigt werden. Dieser Ansatz ermöglicht zuverlässige Segmentierungsergebnisse, selbst wenn nicht alle Modalitäten vollständig vorliegen.

Kurz zusammengefasst: **PRISM bietet einen praktischen Ansatz, um trotz unvollständiger Datensätze präzise Segmentierungen zu realisieren.**