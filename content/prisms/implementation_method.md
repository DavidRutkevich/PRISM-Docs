---
**Titel: Methodik: Das PRISMS Framework – Modulare Fusion für Robuste Segmentierung**
**Beschreibung:** Detaillierte Erläuterung der mathematischen Grundlagen und der Implementierung des PRISMS-Frameworks.
**Datum:** 2025-02-05
**math: true**

---

## 1. Grundlagen der PRISMS-Architekturbausteine

In diesem Abschnitt werden die theoretischen Grundlagen und Implementierungsdetails der einzelnen Module erläutert, die das **PRISMS**-Framework bilden. PRISMS ist darauf ausgelegt, multimodale Daten robust zu fusionieren und präzise Segmentierungen zu ermöglichen, auch bei unvollständigen Datensätzen.

### 1.1. Modality Encoder

**Konzept und Mathematische Beschreibung:**
Der Modality Encoder dient der Extraktion tiefgehender, hierarchischer Merkmale für jede einzelne Eingangsmodalität \( m \). Ausgehend von einem Rohdateneingang \( x^m_0 \) (z.B. ein 3D-MRT-Volumen) wird dieser durch eine Sequenz von Faltungsblöcken \( E_m^{(l)} \) auf verschiedenen Ebenen \( l \) transformiert, um eine Repräsentation \( x^m_L \) auf der tiefsten Ebene sowie Skip-Connection-Features \( x^m_l \) für höhere Auflösungen zu generieren:

\[
x^m_{l+1} = E_m^{(l+1)}(x^m_l)
\]

Jeder Block \( E_m^{(l)} \) besteht typischerweise aus Faltungsoperationen, Normalisierung und Aktivierungsfunktionen, oft ergänzt durch Residualverbindungen. In unserer Implementierung werden `general_conv3d`-Blöcke mit `pad_type='reflect'` verwendet. Die strikt separate Verarbeitung der Modalitäten in den Encodern ist fundamental, um die Robustheit gegenüber fehlenden Modalitäten zu gewährleisten.

**Codeausschnitt (Illustrativ für einen Encoder-Block):**

```python
class Encoder(nn.Module):
    def __init__(self):
        super(Encoder, self).__init__()
        self.e1_c1 = general_conv3d(1, basic_dims, pad_type='reflect')
        self.e1_c2 = general_conv3d(basic_dims, basic_dims, pad_type='reflect')
        self.e1_c3 = general_conv3d(basic_dims, basic_dims, pad_type='reflect')
        # ... weitere Blöcke ...

    def forward(self, x):
        x1 = self.e1_c1(x)
        x1 = x1 + self.e1_c3(self.e1_c2(x1))
        # ... Verarbeitung ...
        return x1, x2, x3, x4, x5
```

*Dieser Code demonstriert die initialen Faltungsschichten und eine Residualverknüpfung.*

---

### 1.2. Adaptive Fusion Transformer (AFT) im PRISM Bottleneck

**Konzept und Mathematische Beschreibung:**
Der **Adaptive Fusion Transformer (AFT)** ist das zentrale Modul im sogenannten **PRISM Bottleneck**. Er integriert initial Informationen aus allen verfügbaren Modalitäten und lernbaren Fusions-Tokens. Die tiefsten Merkmals-Tensoren \( x^m_L \) aus den Encodern werden für jede Modalität \( m \) zu Token-Sequenzen \( F_m \in \mathbb{R}^{B \times N \times C'} \) entfaltet. Zusätzlich werden lernbare Fusions-Tokens \( F_{\mathrm{fusion}} \in \mathbb{R}^{B \times N_f \times C'} \) verwendet.

Diese Token-Mengen werden konkateniert und mit lernbaren Positions-Embeddings \( P \) additiv kombiniert, um den Eingabe-Token-Satz \( Z_0 \) für den AFT zu bilden:

\[
Z_0 = \mathrm{Concat}\left( F_1, \dots, F_M, F_{\mathrm{fusion}} \right) + P
\]

Dieser Satz \( Z_0 \) wird dann durch \( L_1 \) Transformer-Blöcke des AFT verarbeitet. Jeder Block wendet Mechanismen wie die **Modalitätsmaskierte Attention (MMA)** (siehe Abschnitt 1.5) an, um globale Abhängigkeiten unter Berücksichtigung einer Eingangsmaske \(\mathcal{M}\) zu lernen:

\[
Z_{l+1} = \mathrm{AFTBlock}_l(Z_l, \mathcal{M})
\]

Der resultierende Tensor \( Z_{L_1} \) wird in transformierte Modalitäts-Features \( F'_m \), transformierte Fusions-Tokens \( F'_{\mathrm{fusion}} \) und Attention-Matrizen (insbesondere \(\text{Attn}_1\) für die SRA) aufgeteilt.

**Codeausschnitt (Illustrativ für die AFT-Verarbeitung):**

```python
embed_cat = torch.cat((embed_flair, embed_t1ce, embed_t1, embed_t2, fusion), dim=1)
embed_cat = embed_cat + pos
# self.trans_bottle ist der MaskedTransformer (AFT-Block)
embed_cat_trans, attn_matrices = self.trans_bottle(embed_cat, mask)
flair_trans, t1ce_trans, t1_trans, t2_trans, fusion_trans = \
    torch.chunk(embed_cat_trans, num_modals + 1, dim=1)
```

*Der Code illustriert die Token-Zusammenführung, Positions-Embedding-Addition, Transformer-Verarbeitung (AFT) und die Aufteilung der resultierenden Tokens.*

---

### 1.3. Spatial Weight Attention (SRA)

**Konzept und Mathematische Beschreibung:**
Die **Spatial Relevnce Attention (SRA)** quantifiziert und appliziert die räumliche Wichtigkeit von Merkmalsregionen. Sie nutzt die im ersten Layer des AFT berechnete Attention-Matrix \(\text{Attn}_1\). Für jeden ursprünglichen Modalitäts-Token \( j \) wird dessen räumliche Relevanz als Summe seiner Attention-Gewichte zu allen Fusions-Tokens \( i \) berechnet:

\[
\text{Relevance}(j) = \sum_{H_a} \sum_{i \in \text{FusionTokens}} \text{Attn}_1(i, j)
\]
Diese Relevanzwerte werden für jede Modalität \( m \) zu einer räumlichen Wichtigkeitskarte \( I_m \in \mathbb{R}^{B \times 1 \times H \times W \times D} \) umgeformt.

Diese Karten \( I_m \) werden dann elementweise mit den transformierten Modalitäts-Features \( F'_m \) (Ausgabe des AFT) und den ursprünglichen Encoder-Skip-Connection-Features \( x^m_l \) auf verschiedenen Ebenen multipliziert (nach Upsampling der \( I_m \)-Karten):

\[
\widetilde{F}'_m = F'_m \odot I_m
\]
\[
\widetilde{x}^m_l = x^m_l \odot \mathrm{Upsample}(I_m, \text{scale}_l)
\]
Fehlende Modalitäten führen zu einer Null-Karte \( I_m \). Die resultierenden gewichteten Features \(\widetilde{F}'_m\) und \(\widetilde{x}^m_l\) werden weiterverarbeitet.

**Codeausschnitt (Konzeptuell für die SRA-Gewichtung):**

```python
# attn_matrices[0] ist Attn_1 aus dem ersten AFT-Layer
# ... (Details der Indexierung und Summierung zur Erzeugung von spatial_importance_map_m) ...
# Beispielhafte Anwendung:
weighted_feature_map_mod_m = feature_map_mod_m * spatial_importance_map_m
```

*Die SRA gewichtet Features aus dem AFT und Skip-Connections, um räumlich relevante Informationen hervorzuheben.*

---

### 1.4. Cross-Modal Fusion Transformer (KFT)

**Konzept und Mathematische Beschreibung:**
Der **Kanalbezogener Fusion Transformer (KFT)**, im Code durch `MultiCrossToken`-Blöcke (`self.CTx`) realisiert, ist für die fortgeschrittene Fusion und Re-Kalibrierung der modalitätsspezifischen Informationen auf verschiedenen Ebenen des Decoders zuständig. Er ermöglicht eine tiefe Interaktion zwischen den Token-Strömen der verschiedenen Modalitäten und den transformierten Fusions-Tokens.

Auf jeder Decoder-Ebene \( d \) nimmt ein KFT-Block \( \text{KFT}_d \) die Fusions-Features \( \mathcal{F}_{\text{fusion}}^{(d-1)} \) von der vorherigen Ebene und die SRA-gewichteten Skip-Connection-Features der einzelnen Modalitäten \( \widetilde{x}^m_l \) als Eingabe. Der KFT-Block besteht aus \( L_2 \) Schichten, die typischerweise Cross-Attention- und Self-Attention-Mechanismen unter Verwendung der **MMA** (siehe Abschnitt 1.5) beinhalten:

1.  **Cross-Attention:** Fusions-Features \( \mathcal{F}_{\text{fusion}}^{(d-1)} \) als Queries, konkatenierte modalitätsspezifische Features \( \mathrm{Concat}(\{\widetilde{x}^m_l\}_M) \) als Keys/Values.
2.  **Self-Attention:** Zur Verfeinerung der aggregierten Fusions-Features.

Die Ausgabe ist ein Satz verfeinerter Fusions-Features \( \mathcal{F}_{\text{fusion}}^{(d)} \) für die aktuelle Decoder-Ebene:

\[
\mathcal{F}_{\text{fusion}}^{(d)}, \text{UpdatedModalFeatures}^{(d)} = \mathrm{KFT}_d(\mathcal{F}_{\text{fusion}}^{(d-1)}, \{\widetilde{x}^m_l\}_M, \mathcal{M})
\]

**Codeausschnitt (Illustrativ für einen `MultiCrossToken`-Block als KFT):**

```python
class MultiCrossToken(nn.Module): # Repräsentiert einen KFT-Block
    # ... init mit MultiMaskCrossBlock Layers ...
    def forward(self, inputs_modal_features, kernel_fusion_features, mask):
        # inputs_modal_features: SRA-gewichtete Skip-Features
        # kernel_fusion_features: upgesampelte Fusions-Features von tieferer Ebene
        # MultiMaskCrossBlock führt MMA-basierte Cross-Attention und FFN durch
        # ... (Loop über Layers) ...
        return kernels # Verfeinerte Fusions-Features
```

*Der KFT (`MultiCrossToken`) verarbeitet modalitätsspezifische und Fusions-Features unter Berücksichtigung der Maske, um die Fusions-Features auf jeder Decoder-Ebene zu aktualisieren.*

---

### 1.5. Modalitätsmaskierte Attention (MMA) (Modality Masked Attention)

**Konzept und Mathematische Beschreibung:**
Die **Modalitätsmaskierte Attention (MMA)** ist ein fundamentaler Mechanismus in den Transformer-Blöcken von PRISMS (AFT und KFT). Sie stellt sicher, dass Attention-Berechnungen nur zwischen Tokens von tatsächlich vorhandenen Modalitäten oder zwischen Modalitäts-Tokens und Fusions-Tokens stattfinden. Die Standard-Attention ist:
\[
\text{Attention}(Q, K, V) = \mathrm{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
\]
Die MMA modifiziert die Attention-Scores \( A = \frac{QK^T}{\sqrt{d_k}} \) vor der Softmax-Operation. Eine binäre Attention-Maske \(\mathcal{A_M}\), basierend auf der Eingangsmaske \(\mathcal{M}\), wird generiert. Scores für ungültige Interaktionen (\(\mathcal{A_M}(i,j) = 0\)) werden auf \(-\infty\) gesetzt:
\[
A'_{ij} = \begin{cases} A_{ij} & \text{if } \mathcal{A_M}(i,j) = 1 \\ -\infty & \text{if } \mathcal{A_M}(i,j) = 0 \end{cases}
\]
\[
\text{MMA}(Q, K, V) = \mathrm{softmax}(A')V
\]
Dies verhindert, dass Tokens fehlender Modalitäten die Repräsentationen beeinflussen.

**Codeausschnitt (Illustrativ für MMA im AFT/`MaskedAttention`):**

```python
attn_scores = (q @ k.transpose(-2, -1)) * self.scale
# self_mask_for_attn wird basierend auf der Eingangsmaske 'mask' generiert
# z.B. durch mask_gen_fusion für AFT oder mask_gen_cross4 für KFT
self_mask_for_attn = mask_generator_function(B, ..., mask)
attn_scores = attn_scores.masked_fill(self_mask_for_attn == 0, float("-inf"))
attn_probs = attn_scores.softmax(dim=-1)
# ... attn_probs @ v ...
```

*Der Code zeigt die Modifikation der Attention-Scores mittels `masked_fill` zur Realisierung der MMA.*

---

## 2. Der PRISM-Distillationsmechanismus in PRISMS

Das **PRISMS**-Framework integriert den PRISM-Distillationsansatz, um Wissen von einem starken "Lehrer" (der multimodalen Fusionsrepräsentation) auf "Schüler" (effektiv unimodale Verarbeitungspfade innerhalb des Fusionsnetzwerks) zu übertragen.

### 2.1. Pixel-weiser KL Divergence Loss

**Konzept:** Angleichung der Vorhersageverteilungen auf Pixelebene.
Der Lehrerpfad (Haupt-Fusionspipeline von PRISMS) erzeugt Logits \( z^t_n \). Für jeden Schülerpfad \( m \) (konzeptionell erzeugt durch Verarbeitung nur der Modalität \( m \) durch die Fusionspipeline mit unimodaler Maske) werden Logits \( z^m_n \) erzeugt. Der Verlust ist:

\[
L^{m}_{\text{pixel-KL}} = \sum_n D_{\mathrm{KL}}\!\left[\sigma(z^t_n/\tau)\,\middle\|\,\sigma(z^m_n/\tau)\right]
\]
Dies wird für finale Vorhersagen und Aux Heads berechnet.

### 2.2. Prototype Alignment Loss (Semantische Distillation)

**Konzept:** Angleichung der semantischen Klassenrepräsentationen.
Für jede Klasse \( k \) werden Prototyp-Vektoren durch räumliche Aggregation der Merkmale des Lehrers (\(f^t_n\)) und des Schülers \( m \) (\(f^m_n\)) berechnet:
\[
S^{t}_{k} = \mathrm{Pool}_{\text{class } k}(f^t_n), \quad S^{m}_{k} = \mathrm{Pool}_{\text{class } k}(f^m_n)
\]
Der Verlust minimiert die \(L_2\)-Distanz zwischen den Prototypen:
\[
L^{m}_{\text{align}} = \sum_{k} \bigl\|S^t_{k} - S^m_{k}\bigr\|_2^2
\]

**Rationale und Implementierung in PRISMS:**
Die Gradienten aus dem kombinierten PRISM-Verlust \(L_{\text{prism}}\) aktualisieren die Gewichte der gemeinsam genutzten Komponenten (AFT, SRA, KFT, Haupt-Fusionsdecoder). Mechanismen zur "Preference-Aware Re-Weighting" können integriert werden.

**Codeausschnitt (Prototype Alignment Loss):**

```python
align_loss_flair, dist_flair = prototype_alignment_loss(
    student_features=de_f_flair[0],       # Merkmale des Flair-Schülers
    teacher_features=de_f_avg[0].detach(),# Merkmale des Lehrers (kein Gradient)
    target_labels=target,
    student_logits=fuse_pred_flair,       # Logits des Flair-Schülers
    teacher_logits=fuse_pred.detach(),    # Logits des Lehrers (kein Gradient)
    # ...
)
# L_prism = Summe aller align_loss_m und kl_loss_m
```

*Der Code zeigt die Berechnung des Prototype Alignment Loss für einen Schülerpfad.*

---

## 3. Zusammenfassung der PRISMS-Komponenten und des PRISM-Mechanismus

*   **Modality Encoder:** Extrahiert unabhängige, hierarchische Merkmale pro Modalität.
*   **Adaptive Fusion Transformer (AFT) im PRISM Bottleneck:** Initiale globale Fusion von Modalitäts- und Fusions-Tokens mittels **MMA**.
*   **Spatial Weight Attention (SRA):** Gewichtung der räumlichen Relevanz von AFT-Ausgaben und Skip-Connections basierend auf AFT-Attention.
*   **Cross-Modal Fusion Transformer (KFT):** Tiefe Interaktion und Re-Kalibrierung von Fusions- und Skip-Connection-Features auf Decoder-Ebenen mittels **MMA** (realisiert durch `MultiCrossToken`-Blöcke).
*   **Modalitätsmaskierte Attention (MMA) (Modality Masked Attention):** Kernmechanismus in AFT und KFT zur Berücksichtigung nur valider Token-Paare.
*   **Haupt-Fusionsdecoder:** Verarbeitet KFT-Ausgaben und SRA-gewichtete Skips; dient als "Lehrer".
*   **PRISM-Distillationsmechanismus:** Überträgt Wissen vom Lehrer auf konzeptionelle Schülerpfade mittels Pixel-KL- und Prototype-Alignment-Loss.
*   **Regulation Decoders:** Separate Decoder pro Modalität für \(L_{\text{reg}}\).
