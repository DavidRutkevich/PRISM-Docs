---

title: "Methodik: PRISM und PRISMS"
description: "Zusammenführung der mathematischen Grundlagen mit den PRISM- und PRISMS-Konzepten, unter Einbezug der Code-Logik für Adaptive Fusion Transformer (AFT), Spatial Relevance Attention (SRA) und Kanalbezogenen Fusionstransformer (KFT)."
date: 2025-02-05
math: true

---

## 1. Mathematische Grundlagen der Bausteine

In diesem Abschnitt werden die theoretischen Grundlagen der einzelnen Module erläutert, die zusammen das Framework **PRISMS** bilden. Dabei wird auch auf zentrale Implementierungsdetails eingegangen.

### 1.1. Modality Encoder

**Mathematische Beschreibung:**
Der Modality Encoder ist dafür verantwortlich, für jede Modalität \( m \) separate, charakteristische Features zu extrahieren. Dabei wird der Eingang \( x^m_0 \) (z. B. ein 3D-MRT-Bild) schrittweise durch eine Reihe von convolution Operationen transformiert. Formal lässt sich der Prozess für eine Modalität \( m \) durch die rekursive Gleichung darstellen:

\[
x^m_{i+1} = E_m(x^m_i)
\]

wobei \( E_m \) unterschiedliche convolution Blöcke repräsentiert, die in unserem Code beispielsweise mit Reflection Padding (pad_type='reflect') implementiert werden. Diese Architektur, die an 3D-UNet-Strukturen angelehnt ist, erlaubt eine tiefe, schichtweise Extraktion von Features. Besonders wichtig ist die separate Verarbeitung der Modalitäten, da dadurch auch unvollständige Datensätze robust behandelt werden können.

**Codeausschnitt:**

```python
class Encoder(nn.Module):
    def __init__(self):
        super(Encoder, self).__init__()
        # Erster Block: Einfache Convolutionen mit Residual-Verknüpfung
        self.e1_c1 = general_conv3d(1, basic_dims, pad_type='reflect')
        self.e1_c2 = general_conv3d(basic_dims, basic_dims, pad_type='reflect')
        self.e1_c3 = general_conv3d(basic_dims, basic_dims, pad_type='reflect')
```

*Dieser Code demonstriert die initialen Convolutional-Schichten, die den Eingang in erste Feature-Repräsentationen überführen.*

---

### 1.2. Adaptive Fusion Transformer (AFT)

**Mathematische Beschreibung:**
Der AFT ist der Kern des Frameworks, der die Information aller Modalitäten (sowie eines zusätzlichen Fusionsvektors) global integriert. Zunächst werden die modalitätsspezifischen Feature-Tensoren \( x^m \) – beispielsweise aus den Encodern – in eine 2D-Darstellung umgeformt. Für jede Modalität \( m \) wird nach dem Reshaping und Flattening ein Tensor \( F_m \in \mathbb{R}^{N \times c} \) erzeugt, wobei \( N = h \times w \times d \) ist. Anschließend wird ein Satz lernbarer Fusionstokens \( F_{\mathrm{fusion}} \in \mathbb{R}^{N \times c} \) definiert.

Durch die Konkatenation aller modalitätsspezifischen Features und der Fusionstokens entsteht der globale Eingabetensor:

\[
z = \mathrm{Concat}\left( F_1, F_2, F_3, F_4, F_{\mathrm{fusion}} \right) + p
\]

wobei \( p \) ein Positionsvektor ist, der die räumliche Lageinformationen beibehält. Dieser Token-Set \( z \) wird in einen Standard-Transformer (AFT-Block) eingespeist, der unter Berücksichtigung einer Binärmaksse \( M \) (die fehlende Modalitäten ausschließt) globale Abhängigkeiten lernt:

\[
z' = \mathrm{Transformer}(z, M)
\]

Nach der globalen Verarbeitung werden die Features durch das Token-Splitting wieder in die einzelnen Modalitäten aufgeteilt.

**Codeausschnitt:**

```python
embed_cat = torch.cat((embed_flair, embed_t1ce, embed_t1, embed_t2, fusion), dim=1)
embed_cat = embed_cat + pos
embed_cat_trans, attn = self.trans_bottle(embed_cat, mask)
flair_trans, t1ce_trans, t1_trans, t2_trans, fusion_trans = torch.chunk(embed_cat_trans, self.num_cls+1, dim=1)
```

*Der Code illustriert, wie die modalitätsspezifischen Embeddings und die Fusionstokens zunächst zusammengeführt, mit Positionsinformationen versehen und dann durch den Transformer verarbeitet werden. Anschließend erfolgt das Aufteilen in einzelne Modalitäten.*

---

### 1.3. Spatial Relevance Attention (SRA)

**Mathematische Beschreibung:**
Die Spatial Relevance Attention (SRA) dient dazu, die räumliche Wichtigkeit einzelner Tokens innerhalb der Feature Maps zu quantifizieren. Obwohl die modality-masked Attention bereits dafür sorgt, dass Tokens überwiegend ihre eigene Modalität repräsentieren, weist nicht jede räumliche Position den gleichen Informationsgehalt auf.
Im ersten AFT-Layer wird hierfür die Attention-Matrix \(\text{MA}_1\) berechnet. Für jeden Token \( j \) wird dessen räumliche Relevanz als Summe der Werte der Zeilen berechnet, die zu den Fusionstokens gehören (d. h. \( i \) von \( 4N+1 \) bis \( 5N \), wobei \( N = h \times w \times d \)):

\[
\text{Col}(j) = \sum_{H} \sum_{i=4N+1}^{5N} \text{MA}_1(i,j), \quad j \in [1,4N]
\]

Der resultierende Vektor \(\text{Col} \in \mathbb{R}^{1 \times 4N}\) wird dann in vier Teilsegmente \( \text{Col}_m \in \mathbb{R}^{1 \times N} \) für jede Modalität \( m \in M \) aufgeteilt. Diese Teilsegmente werden anschließend in räumliche Wichtigkeitskarten \( I_m \in \mathbb{R}^{1 \times h \times w \times d} \) reshaped und mittels elementweiser Multiplikation mit den modalitätsspezifischen Features \(\hat{F}_m\) kombiniert:

\[
\widetilde{F}_m = \hat{F}_m \odot I_m
\]

*Hier sorgt \(\odot\) dafür, dass nur die wichtigen räumlichen Bereiche (mit \( I_m \neq 0 \)) verstärkt werden. Fehlende Modalitäten führen zu einem \( I_m \) aus Nullen, sodass deren Beitrag ausgeschlossen wird.*

Zusätzlich werden diese Gewichtskarten über Upsampling in den Skip-Connections integriert, um auch auf höheren Auflösungsebenen relevante Details zu erhalten.

---

### 1.4. Kanalbezogener Fusionstransformer (KFT)

**Mathematische Beschreibung:**
Während SRA auf die räumlichen Dimensionen abzielt, bearbeitet der Kanalbezogene Fusionstransformer (KFT) die Kanaldimension, um redundante oder dominierende Kanäle auszubalancieren. Zunächst wird aus der Feature Map \( x \in \mathbb{R}^{B \times K \times C \times H \times W \times D} \) der Durchschnitt über die räumlichen Dimensionen berechnet:

\[
\bar{x} = \frac{1}{HWD} \sum_{h,w,d} x_{:, :, :, h, w, d}
\]

Dieser Durchschnitt wird dann durch einen Feed-Forward-Block \( f(\cdot) \) und eine Sigmoid-Aktivierung transformiert, um kanalweise Gewichtungen \( w \) zu erhalten:

\[
w = \sigma\Bigl(f(\bar{x})\Bigr)
\]

Die finalen, neu gewichteten Features werden durch eine gewichtete Summe über die Kanäle berechnet:

\[
y = \sum_{m=1}^{K} w_m \cdot x_m
\]

**Codeausschnitt:**

```python
feat_avg = torch.mean(x, dim=(3,4,5), keepdim=False)
feat_avg = feat_avg.view(B, K * C, 1, 1, 1)
weight = torch.reshape(self.weight_layer(feat_avg), (B, K, 1))
weight = self.sigmoid(weight)
```

*Der Code zeigt, wie aus den räumlich aggregierten Features kanalweise Gewichte berechnet werden, die dann zur Neugewichtung der Feature-Maps genutzt werden. Dadurch wird sichergestellt, dass weniger starke Modalitäten verstärkt und dominierende Modalitäten gedämpft werden.*

---

### 1.5. Modalitätsmaskierte Attention (MMA)

**Mathematische Beschreibung:**
Die Modalitätsmaskierte Attention (MMA) integriert eine binäre Maske \( M \) direkt in die Berechnung der standardmäßigen skalieren Punktprodukt-Attention. Die unmodifizierte Attention-Funktion lautet:

\[
\text{Attn}(Q, K) = \mathrm{softmax}\left(\frac{QK^T}{\sqrt{d}}\right)
\]

Mit einer binären Maske \( M \), die sicherstellt, dass nur Werte für verfügbare Modalitäten berücksichtigt werden, modifizieren wir diese Funktion zu:

\[
\text{Attn}'(Q, K) = \mathrm{softmax}\left(\frac{QK^T}{\sqrt{d}} \odot M\right)
\]

Hierdurch wird verhindert, dass Tokens, die zu fehlenden Modalitäten gehören, in die Attention-Berechnung einfließen.

**Codeausschnitt:**

```python
key = torch.cat((key_flair, key_t1ce, key_t1, key_t2), dim=1)
attn = (query @ key.transpose(-2, -1)) * (query.shape[-1]) ** -0.5
self_mask = mask_gen_cross4(qb, qc, key.shape[1], mask).cuda(non_blocking=True)
attn = attn.masked_fill(self_mask==0, float("-inf"))
attn = attn.softmax(dim=-1)
```

*Der Code zeigt, wie die standardmäßige Attention mit Hilfe einer generierten Maskierung self_mask angepasst wird, sodass nur gültige Modalitäten in die Berechnung einfließen.*

---

## 2. Integration des PRISM-Mechanismus in PRISMS

**Mathematische Beschreibung:**
Das Framework **PRISMS** baut auf dem PRISM-Ansatz auf und erweitert diesen um zusätzliche Komponenten, die speziell zur robusten Fusion von multimodalen Informationen beitragen. Ein zentraler Bestandteil ist die prototypbasierte Distillation, durch die unimodale Repräsentationen an einen starken, globalen Prototypen angeglichen werden. Für jede Modalität \( m \) wird ein Loss-Term definiert, der die Differenz zwischen den unimodalen Ähnlichkeitswerten \( S^{m,s}(i) \) und den globalen, multimodalen Ähnlichkeitswerten \( S^t(i) \) minimiert:

\[
L^m_{\text{proto}} = \sum_{i} \| S^{m,s}(i) - S^t(i) \|_2^2
\]

Dies gewährleistet, dass Modalitäten, die seltener oder von geringerer Qualität sind, durch den Vergleich an den stärkeren globalen Features ausgerichtet werden.

**Codeausschnitt – Integration in PRISMS:**

```python
proto_loss[0], dist[0] = prototype_prism_loss(
    de_f_flair[0],             # Unimodaler Feature-Output für Flair
    de_f_avg[0].detach(),      # Globaler Prototyp, ohne Gradientenfluss
    target,                    # Ground-Truth
    fuse_pred_flair,           # Segmentierungsvorhersage der Flair-Pfade
    fuse_pred.detach(),        # Gesamtsegmentierung (Lehrer)
    num_cls=num_cls,
    temp=temp
)
```

*Hier wird die Differenz zwischen den unimodalen und globalen Repräsentationen gezielt minimiert, um die Robustheit und Genauigkeit der finalen Segmentierung zu verbessern.*

---

## 3. Zusammenfassung

- **Modality Encoder:**
  Extrahiert durch convolution Schichten modalitätsspezifische Feature-Repräsentationen, die unabhängig voneinander verarbeitet werden können.

- **Adaptive Fusion Transformer (AFT):**
  Kombiniert modalitätsspezifische Features und Fusionstokens durch Konkatenation, Addition von Positionsvektoren und globale Verarbeitung unter Berücksichtigung einer Maskierung für fehlende Modalitäten. Das Ergebnis wird anschließend in die einzelnen Modalitäten zurückgeführt.

- **Spatial Relevance Attention (SRA):**
  Nutzt die Attention-Matrix des ersten AFT-Layers, um aus den Fusionstokens Gewichtungsvektoren zu berechnen, die die relevanten räumlichen Regionen jeder Modalität hervorheben.

- **Kanalbezogener Fusionstransformer (KFT):**
  Rebalanciert kanalweise die Informationen, indem lokale, convolution-basierte Projektionen eingesetzt werden, um den Beitrag einzelner Kanäle adaptiv zu gewichten.

- **Modalitätsmaskierte Attention (MMA):**
  Modifiziert den standardmäßigen Attention-Mechanismus durch eine binäre Maske, sodass nur die verfügbaren Modalitäten in die Berechnung einfließen.

- **PRISM-Mechanismus:**
  Durch prototypbasierte Distillation werden unimodale Repräsentationen an globale Prototypen angepasst, um die Robustheit insbesondere bei unvollständigen Datensätzen zu verbessern.

Diese Bausteine bilden die Grundlage von **PRISMS**, einem modularen Framework, das klassische convolution Methoden mit modernen Transformer-Techniken kombiniert, um eine robuste und präzise Segmentierung multimodaler Daten zu ermöglichen.

Hier folgt eine überarbeitete Passage, die die Integration des PRISM-Moduls in das PRISMS-Framework detailliert erläutert – mit aktualisierten Namenskonventionen, sodass veraltete Bezeichnungen nicht erwähnt werden:

---

## Integration des PRISM-Moduls in PRISMS

In **PRISMS** wird das ursprüngliche PRISM-Konzept weiterentwickelt, um den Informationsaustausch zwischen den Modalitäten noch robuster zu gestalten – insbesondere in Szenarien mit unvollständigen Datensätzen. Dabei kommen zwei wesentliche Neuerungen zum Einsatz:

1. **Prototype Alignment Loss**
   Anstelle des alten Ansatzes wird nun ein **Prototype Alignment Loss** eingesetzt, um die unimodalen Feature-Repräsentationen gezielt an einen starken, globalen Prototypen anzupassen. Für jede Modalität \( m \) werden die unimodalen Ähnlichkeitswerte \( S^{m,s}(i) \) mit den globalen, multimodalen Ähnlichkeitswerten \( S^t(i) \) verglichen. Dieser Vergleich wird durch folgenden Verlustterm formuliert:

   \[
   L^m_{\text{align}} = \sum_{i} \| S^{m,s}(i) - S^t(i) \|_2^2
   \]

   Die Minimierung dieses Terms zwingt auch Modalitäten, die seltener oder qualitativ schwächer repräsentiert sind, dazu, ihre Repräsentationen an den starken, durch alle Modalitäten aggregierten globalen Prototyp anzupassen. Dadurch wird eine ausgewogene und konsistente Fusion der Informationen erreicht.

   **Codebeispiel:**

   ```python
   align_loss[0], dist[0] = prototype_alignment_loss(
       de_f_flair[0],             # Unimodaler Feature-Output für die Flair-Modalität
       de_f_avg[0].detach(),      # Globaler Prototyp, der als "Lehrer" fungiert (ohne Gradientenfluss)
       target,                    # Ground-Truth
       fuse_pred_flair,           # Segmentierungsvorhersage der Flair-Modalität
       fuse_pred.detach(),        # Gesamtsegmentierung – als Referenz für starke Features
       num_cls=num_cls,
       temp=temp
   )
   ```

   In diesem Ausschnitt wird der Prototype Alignment Loss berechnet und in den Gesamtverlust integriert, um die Differenz zwischen den unimodalen Vorhersagen und dem globalen, fusionierten Ergebnis zu minimieren.

2. **Cross-Modal Fusion Transformer (CMFT)**
   Zusätzlich wird die Integration der Modalitäten durch einen speziell entwickelten Cross-Modal Fusion Transformer optimiert. Module wie `MultiCrossToken` (z. B. implementiert als `self.CT5` und `self.CT4`) sorgen dafür, dass die Fusionstokens und modalitätsspezifischen Token-Streams in mehreren Schichten interagieren. Durch diese Interaktion werden relevante Informationen aus den unterschiedlichen Modalitäten zusammengeführt und kanalweise neu gewichtet – wodurch der Einfluss dominanter Modalitäten abgeschwächt und schwächere Modalitäten gestärkt werden.

Diese beiden Komponenten – der Prototype Alignment Loss und der Cross-Modal Fusion Transformer – sind integraler Bestandteil von **PRISMS**. Gemeinsam tragen sie dazu bei, dass die Ergebnisse der globalen Fusion (repräsentiert durch die finalen Segmentierungsvorhersagen) konsistent und robust sind, selbst wenn einzelne Datenquellen fehlen oder von unterschiedlicher Qualität sind.
