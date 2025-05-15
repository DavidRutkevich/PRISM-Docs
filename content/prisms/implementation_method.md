--

**Titel: Methodik: Das PRISMS Framework – Modulare Fusion für Robuste Segmentierung**
**Beschreibung:** Detaillierte Erläuterung der mathematischen Grundlagen und der Implementierung des PRISMS-Frameworks.
**Datum:** 2025-02-05
**math: true**

---

## 1. Grundlagen der PRISMS-Architekturbausteine

In diesem Abschnitt werden die theoretischen Grundlagen und Implementierungsdetails der einzelnen Module erläutert, die das **PRISMS**-Framework (PRISM-enhanced Modality-masked Segmentation) bilden. PRISMS ist darauf ausgelegt, multimodale Daten robust zu fusionieren und präzise Segmentierungen zu ermöglichen, auch bei unvollständigen Datensätzen.

### 1.1. Modality Encoder

**Konzept und Mathematische Beschreibung:**
Der Modality Encoder dient der Extraktion tiefgehender, hierarchischer Merkmale für jede einzelne Eingangsmodalität \( m \). Ausgehend von einem Rohdateneingang \( x^m_0 \) (z.B. ein 3D-MRT-Volumen) wird dieser durch eine Sequenz von Faltungsblöcken \( E_m^{(l)} \) auf verschiedenen Ebenen \( l \) transformiert, um eine Repräsentation \( x^m_L \) auf der tiefsten Ebene sowie Skip-Connection-Features \( x^m_l \) für höhere Auflösungen zu generieren:

\[
x^m_{l+1} = E_m^{(l+1)}(x^m_l)
\]

Jeder Block \( E_m^{(l)} \) besteht typischerweise aus Faltungsoperationen, Normalisierung und Aktivierungsfunktionen, oft ergänzt durch Residualverbindungen zur Stabilisierung des Trainings. In unserer Implementierung werden `general_conv3d`-Blöcke mit `pad_type='reflect'` verwendet, was an Architekturen wie 3D-UNet angelehnt ist. Die strikt separate Verarbeitung der Modalitäten in den Encodern ist fundamental, um die Robustheit gegenüber fehlenden Modalitäten im Datensatz zu gewährleisten, da jede Modalität zunächst unabhängig ihre spezifischen Charakteristika lernt.

**Codeausschnitt (Illustrativ für einen Encoder-Block):**

```python
class Encoder(nn.Module):
    def __init__(self):
        super(Encoder, self).__init__()
        # Beispielhafter erster Block mit Residualverknüpfung
        self.e1_c1 = general_conv3d(1, basic_dims, pad_type='reflect')
        self.e1_c2 = general_conv3d(basic_dims, basic_dims, pad_type='reflect')
        self.e1_c3 = general_conv3d(basic_dims, basic_dims, pad_type='reflect')
        # ... weitere Blöcke für tiefere Ebenen ...

    def forward(self, x):
        x1 = self.e1_c1(x)
        x1 = x1 + self.e1_c3(self.e1_c2(x1)) # Residualverknüpfung
        # ... Verarbeitung durch weitere Blöcke ...
        return x1, x2, x3, x4, x5 # Features verschiedener Ebenen
```

*Dieser Code demonstriert die initialen Faltungsschichten und eine Residualverknüpfung, die den Eingang in erste Merkmalsrepräsentationen überführen und für Skip-Connections bereitstellen.*

---

### 1.2. Adaptive Fusion Transformer (AFT) im PRISM Bottleneck

**Konzept und Mathematische Beschreibung:**
Der Adaptive Fusion Transformer (AFT) ist das zentrale Modul im sogenannten **PRISM Bottleneck**, verantwortlich für die initiale globale Integration von Informationen aus allen verfügbaren Modalitäten sowie einem Satz lernbarer Fusions-Tokens. Die tiefsten Merkmals-Tensoren \( x^m_L \) aus den Encodern (mit Dimensionen \( B \times C \times H \times W \times D \)) werden für jede Modalität \( m \) zunächst räumlich entfaltet ("flattened") und transponiert, um Token-Sequenzen \( F_m \in \mathbb{R}^{B \times N \times C'} \) zu erzeugen, wobei \( N = H \cdot W \cdot D \) die Anzahl der räumlichen Tokens und \( C' \) die Feature-Dimension ist. Zusätzlich werden lernbare Fusions-Tokens \( F_{\mathrm{fusion}} \in \mathbb{R}^{B \times N_f \times C'} \) initialisiert, wobei \( N_f \) die Anzahl der Fusions-Tokens ist (in unserem Code entspricht \(N_f\) der Anzahl der Patches, \( \text{patch_size}^3 \)).

Diese Token-Mengen werden konkateniert und mit lernbaren Positions-Embeddings \( P \in \mathbb{R}^{B \times (M \cdot N + N_f) \times C'} \) additiv kombiniert, um den finalen Eingabe-Token-Satz \( Z_0 \) für den Transformer zu bilden:

\[
Z_0 = \mathrm{Concat}\left( F_1, \dots, F_M, F_{\mathrm{fusion}} \right) + P
\]

Dieser Satz \( Z_0 \) wird dann durch eine Serie von \( L_1 \) Transformer-Blöcken (AFT-Block) verarbeitet. Jeder Block wendet Mechanismen wie die Modalitätsmaskierte Attention (MMA, siehe Abschnitt 1.5) an, um globale Abhängigkeiten unter Berücksichtigung einer binären Eingangsmaske \(\mathcal{M}\) (die angibt, welche Modalitäten vorhanden sind) zu lernen:

\[
Z_{l+1} = \mathrm{TransformerBlock}_l(Z_l, \mathcal{M})
\]

Der resultierende Tensor \( Z_{L_1} \) wird anschließend wieder in modalitätsspezifische transformierte Features \( F'_m \) und transformierte Fusions-Tokens \( F'_{\mathrm{fusion}} \) aufgeteilt. Die Attention-Matrizen aus diesen Transformer-Blöcken, insbesondere \(\text{Attn}_1\) aus dem ersten Layer, werden für die SRA (siehe Abschnitt 1.3) weiterverwendet.

**Codeausschnitt (Illustrativ für die AFT-Verarbeitung):**

```python
# embed_flair, ..., embed_t2 sind die geflatteten Encoder-Outputs
# fusion sind die lernbaren Fusions-Tokens
embed_cat = torch.cat((embed_flair, embed_t1ce, embed_t1, embed_t2, fusion), dim=1)
embed_cat = embed_cat + pos # pos sind die Positions-Embeddings
# self.trans_bottle ist der MaskedTransformer (AFT-Block)
embed_cat_trans, attn_matrices = self.trans_bottle(embed_cat, mask) # mask ist die Eingangsmaske M
# Aufteilung der transformierten Tokens
flair_trans, t1ce_trans, t1_trans, t2_trans, fusion_trans = \
    torch.chunk(embed_cat_trans, num_modals + 1, dim=1)
```

*Der Code illustriert die Konkatenation, Addition von Positions-Embeddings, Verarbeitung durch den Transformer unter Maskierung und die anschließende Aufteilung der resultierenden Tokens.*

---

### 1.3. Spatial Weight Attention (SRA)

**Konzept und Mathematische Beschreibung:**
Die Spatial Weight Attention (SRA), in früheren Versionen als Spatial Relevance Attention bezeichnet, dient der Quantifizierung und Anwendung der räumlichen Wichtigkeit einzelner Merkmalsregionen. Sie nutzt die im ersten Layer des AFT (innerhalb des PRISM Bottlenecks) berechnete Attention-Matrix \(\text{Attn}_1\). Für jeden ursprünglichen Modalitäts-Token \( j \) (aus den \( M \cdot N \) Modalitäts-Tokens) wird dessen räumliche Relevanz als die Summe seiner Attention-Gewichte zu allen Fusions-Tokens \( i \) (aus den \( N_f \) Fusions-Tokens) berechnet:

\[
\text{Relevance}(j) = \sum_{H_a} \sum_{i \in \text{FusionTokens}} \text{Attn}_1(i, j)
\]
wobei \( H_a \) die Anzahl der Attention-Heads ist. Diese Relevanzwerte werden für jede Modalität \( m \) zu einer räumlichen Wichtigkeitskarte \( I_m \in \mathbb{R}^{B \times 1 \times H \times W \times D} \) umgeformt.

Diese Karten \( I_m \) werden dann elementweise mit den entsprechenden transformierten Modalitäts-Features \( F'_m \) (Ausgabe des AFT) sowie den ursprünglichen Encoder-Skip-Connection-Features \( x^m_l \) auf verschiedenen Ebenen multipliziert (nach entsprechendem Upsampling der \( I_m \)-Karten):

\[
\widetilde{F}'_m = F'_m \odot I_m \quad \text{(für die tiefste Ebene)}
\]
\[
\widetilde{x}^m_l = x^m_l \odot \mathrm{Upsample}(I_m, \text{scale}_l) \quad \text{(für Skip-Connections)}
\]
Fehlende Modalitäten (gemäß Eingangsmaske \(\mathcal{M}\)) führen zu einer Null-Karte \( I_m \), wodurch ihr Beitrag effektiv eliminiert wird. Die resultierenden gewichteten Features \(\widetilde{F}'_m\) und \(\widetilde{x}^m_l\) werden dann weiterverarbeitet.

**Codeausschnitt (Konzeptuell für die Gewichtung):**

```python
# attn_matrices[0] ist Attn_1 aus dem ersten AFT-Layer
attn_fusion_to_modalities = attn_matrices[0][:, :, num_modals*N_patches : , : num_modals*N_patches]
# Summation und Reshaping zu I_m für jede Modalität
# ... (Details der Indexierung und Summierung wie im Code Weight_Attention) ...
# Beispielhafte Anwendung:
weighted_feature_map_mod_m = feature_map_mod_m * spatial_importance_map_m
```

*Die SRA gewichtet sowohl die direkt aus dem AFT kommenden Features als auch die Skip-Connections, um räumlich relevante Informationen für die nachfolgende Fusion und Dekodierung hervorzuheben.*

---

### 1.4. Cross-Modal Fusion Transformer (CMFT)

**Konzept und Mathematische Beschreibung:**
Der Cross-Modal Fusion Transformer (CMFT), in früheren Versionen als Kanalbezogener Fusionstransformer (KFT) bezeichnet und im Code durch `MultiCrossToken`-Blöcke (`self.CTx`) realisiert, ist für die fortgeschrittene Fusion und Re-Kalibrierung der modalitätsspezifischen Informationen auf verschiedenen Ebenen des Decoders zuständig. Im Gegensatz zu einer einfachen Kanalgewichtung führt der CMFT eine tiefere Interaktion zwischen den Token-Strömen der verschiedenen Modalitäten und den transformierten Fusions-Tokens durch.

Auf jeder Decoder-Ebene \( d \) nimmt ein CMFT-Block \( \text{CMFT}_d \) die (ggf. upgesampelten und mit Skip-Connections kombinierten) Fusions-Features \( \mathcal{F}_{\text{fusion}}^{(d-1)} \) von der vorherigen Ebene und die SRA-gewichteten Skip-Connection-Features der einzelnen Modalitäten \( \widetilde{x}^m_l \) (die der aktuellen Auflösung \( d \) entsprechen) als Eingabe. Diese werden typischerweise zu Token-Sequenzen verarbeitet. Der CMFT-Block besteht aus \( L_2 \) Schichten von Cross-Attention- und Self-Attention-Mechanismen:

1.  **Cross-Attention:** Die Fusions-Features \( \mathcal{F}_{\text{fusion}}^{(d-1)} \) dienen als Queries, während die konkatenierten modalitätsspezifischen Features \( \mathrm{Concat}(\widetilde{x}^1_l, \dots, \widetilde{x}^M_l) \) als Keys und Values dienen. Dies erlaubt den Fusions-Features, relevante Informationen aus den einzelnen Modalitäten zu aggregieren.
2.  **Self-Attention:** Anschließend verfeinern Self-Attention-Layer innerhalb des CMFT-Blocks die neu aggregierten Fusions-Features.

Die Operationen innerhalb des CMFT werden ebenfalls durch die Eingangsmaske \(\mathcal{M}\) gesteuert, um nur verfügbare Modalitäten zu berücksichtigen. Die Ausgabe ist ein Satz verfeinerter Fusions-Features \( \mathcal{F}_{\text{fusion}}^{(d)} \) für die aktuelle Decoder-Ebene:

\[
\mathcal{F}_{\text{fusion}}^{(d)}, \text{UpdatedModalFeatures}^{(d)} = \mathrm{CMFT}_d(\mathcal{F}_{\text{fusion}}^{(d-1)}, \{\widetilde{x}^m_l\}_M, \mathcal{M})
\]
Die `UpdatedModalFeatures` können optional auch weitergegeben oder für andere Zwecke verwendet werden. In PRISMS liegt der Fokus darauf, \( \mathcal{F}_{\text{fusion}}^{(d)} \) zu verfeinern.

**Codeausschnitt (Illustrativ für einen `MultiCrossToken`-Block):**

```python
class MultiCrossToken(nn.Module):
    # ... init mit MultiMaskCrossBlock Layers ...
    def forward(self, inputs_modal_features, kernel_fusion_features, mask):
        feature_maps = inputs_modal_features # z.B. SRA-gewichtete Skip-Features
        kernels = kernel_fusion_features    # z.B. upgesampelte Fusions-Features von tieferer Ebene
        for layer in self.layers: # self.layers enthält MultiMaskCrossBlock Instanzen
            # MultiMaskCrossBlock führt Cross-Attention und FFN durch
            kernels, feature_maps = layer(kernels, feature_maps, mask)
        return kernels # Verfeinerte Fusions-Features
```

*Der Code zeigt, wie `MultiCrossToken` (CMFT-Block) modalitätsspezifische Features (`inputs_modal_features`) und Fusions-Features (`kernel_fusion_features`) unter Berücksichtigung der Maske verarbeitet, um die Fusions-Features zu aktualisieren. Dies geschieht typischerweise auf mehreren Ebenen im Decoder.*

---

### 1.5. Modalitätsmaskierte Attention (MMA)

**Konzept und Mathematische Beschreibung:**
Die Modalitätsmaskierte Attention (MMA) ist ein fundamentaler Mechanismus, der in verschiedenen Transformer-Blöcken von PRISMS (insbesondere im AFT und CMFT) eingesetzt wird. Sie stellt sicher, dass Attention-Berechnungen nur zwischen Tokens von tatsächlich vorhandenen Modalitäten oder zwischen Modalitäts-Tokens und Fusions-Tokens stattfinden. Die Standard-Scaled-Dot-Product-Attention ist definiert als:

\[
\text{Attention}(Q, K, V) = \mathrm{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
\]
Die MMA modifiziert die Berechnung der Attention-Scores \( A = \frac{QK^T}{\sqrt{d_k}} \) vor der Softmax-Operation. Eine binäre Attention-Maske \(\mathcal{A_M}\) wird dynamisch basierend auf der Eingangsmaske \(\mathcal{M}\) generiert. Diese Maske \(\mathcal{A_M}\) hat Einträge von \(0\) für ungültige (maskierte) Interaktionen und \(1\) für gültige. Die Scores für ungültige Interaktionen werden auf \(-\infty\) gesetzt:

\[
A'_{ij} = \begin{cases} A_{ij} & \text{if } \mathcal{A_M}(i,j) = 1 \\ -\infty & \text{if } \mathcal{A_M}(i,j) = 0 \end{cases}
\]
\[
\text{MMA}(Q, K, V) = \mathrm{softmax}(A')V
\]
Dadurch wird effektiv verhindert, dass Tokens von fehlenden Modalitäten die Repräsentationen der vorhandenen Modalitäten oder der Fusions-Tokens beeinflussen.

**Codeausschnitt (Illustrativ für MMA im AFT/`MaskedAttention`):**

```python
# q, k, v sind abgeleitet von den Eingabe-Tokens
attn_scores = (q @ k.transpose(-2, -1)) * self.scale
# self_mask wird basierend auf der Eingangsmaske 'mask' generiert (z.B. durch mask_gen_fusion)
# Diese Maske bestimmt, welche Token-Paare interagieren dürfen.
# Für AFT: Interaktion innerhalb derselben Modalität erlaubt,
#          Interaktion zwischen Fusions-Token und allen vorhandenen Modalitäts-Tokens erlaubt.
self_mask_for_attn = mask_gen_fusion(B, self.num_heads, N_tokens_per_modal, num_modals, mask)
attn_scores = attn_scores.masked_fill(self_mask_for_attn == 0, float("-inf"))
attn_probs = attn_scores.softmax(dim=-1)
# ... attn_probs @ v ...
```

*Der Code zeigt, wie die Attention-Scores mit `masked_fill` modifiziert werden, bevor die Softmax-Funktion angewendet wird, um die MMA zu realisieren.*

---

## 2. Der PRISM-Distillationsmechanismus in PRISMS

Das **PRISMS**-Framework integriert und erweitert den PRISM-Distillationsansatz, um die Robustheit und Generalisierungsfähigkeit des Modells, insbesondere bei unvollständigen Daten, weiter zu steigern. Die Kernidee ist, Wissen von einem starken "Lehrer" (der multimodalen Fusionsrepräsentation) auf "Schüler" (effektiv unimodale Verarbeitungspfade innerhalb des Fusionsnetzwerks) zu übertragen. Dies geschieht durch zwei komplementäre Verlustfunktionen:

### 2.1. Pixel-weiser KL Divergence Loss

**Konzept:** Diese Komponente zielt darauf ab, die Vorhersageverteilungen auf Pixelebene zwischen dem Lehrer und jedem Schüler anzugleichen.
Der Lehrerpfad (die Haupt-Fusionspipeline von PRISMS, die alle verfügbaren Modalitäten nutzt) erzeugt Logits \( z^t_n \) für jedes Pixel \( n \). Für jeden Schülerpfad \( m \) (konzeptionell erzeugt durch Verarbeitung nur der Modalität \( m \) durch die Fusionspipeline mit einer unimodalen Maske) werden ebenfalls Logits \( z^m_n \) erzeugt. Der Verlust für Schüler \( m \) ist dann:

\[
L^{m}_{\text{pixel-KL}} = \sum_n D_{\mathrm{KL}}\!\left[\sigma(z^t_n/\tau)\,\middle\|\,\sigma(z^m_n/\tau)\right]
\]
wobei \( \sigma \) die Softmax-Funktion und \( \tau \) ein Temperaturparameter ist. Dieser Verlust wird sowohl für die finalen Vorhersagen als auch für die Ausgaben der Aux Heads auf verschiedenen Decoder-Ebenen berechnet.

### 2.2. Prototype Alignment Loss (Semantische Distillation)

**Konzept:** Um sicherzustellen, dass die unimodalen Pfade nicht nur pixelweise, sondern auch semantisch mit der reichhaltigeren multimodalen Repräsentation übereinstimmen, wird ein Prototype Alignment Loss verwendet.
Für jede Klasse \( k \) werden Prototyp-Vektoren berechnet, indem die aktivierten Merkmale des Lehrers (\(f^t_n\), z.B. `de_f_avg[0]` aus dem Code) und des Schülers \( m \) (\(f^m_n\), z.B. `de_f_flair[0]`) für Pixel, die zur Klasse \( k \) gehören, räumlich aggregiert (z.B. gemittelt) werden:
\[
S^{t}_{k} = \mathrm{Pool}_{\text{class } k}(f^t_n), \quad S^{m}_{k} = \mathrm{Pool}_{\text{class } k}(f^m_n)
\]
Der Verlust minimiert dann die \(L_2\)-Distanz zwischen den Lehrer- und Schüler-Prototypen für jede Klasse:
\[
L^{m}_{\text{align}} = \sum_{k} \bigl\|S^t_{k} - S^m_{k}\bigr\|_2^2
\]

**Rationale und Implementierung in PRISMS:**
Diese Distillationsverluste (\(L_{\text{pixel-KL}}\) und \(L_{\text{align}}\)) werden zu einem Gesamt-PRISM-Verlust \(L_{\text{prism}}\) kombiniert. Die Gradienten aus \(L_{\text{prism}}\) fließen durch die Schülerpfade zurück und aktualisieren die Gewichte der gemeinsam genutzten Komponenten (insbesondere AFT, SRA, CMFT und den Haupt-Fusionsdecoder). Dies zwingt das Modell, robuste unimodale Repräsentationen zu lernen, die denen des multimodalen Lehrers ähneln.
Zusätzlich können Mechanismen zur "Preference-Aware Re-Weighting" (wie in der ursprünglichen PRISM-Beschreibung erwähnt, mit \(\delta_m\) und \(\beta_m\)) integriert werden, um das Training bei unausgewogenen Fehlraten der Modalitäten weiter zu optimieren, indem der Fokus auf unterrepräsentierte oder schwächere Modalitäten gelenkt wird.

**Codeausschnitt (Illustrativ für die Berechnung des Prototype Alignment Loss):**

```python
# Annahme: fuse_pred ist die Vorhersage des Lehrers (multimodale Fusion)
#          fuse_pred_flair ist die Vorhersage des Flair-Schülers
#          de_f_avg[0] sind die Merkmale des Lehrers für Prototypen
#          de_f_flair[0] sind die Merkmale des Flair-Schülers für Prototypen

align_loss_flair, dist_flair = prototype_alignment_loss(
    student_features=de_f_flair[0],
    teacher_features=de_f_avg[0].detach(), # Lehrer-Merkmale ohne Gradientenfluss
    target_labels=target,
    student_logits=fuse_pred_flair,
    teacher_logits=fuse_pred.detach(),    # Lehrer-Logits ohne Gradientenfluss
    num_cls=num_cls,
    temp=temp
)
# Ähnliche Verluste für KL-Divergenz und für andere Schülerpfade
# L_prism = Summe aller align_loss_m und kl_loss_m (ggf. gewichtet)
```

*Dieser Code zeigt, wie der Prototype Alignment Loss für einen Schülerpfad berechnet wird. Die `.detach()`-Aufrufe stellen sicher, dass Gradienten nur durch den Schülerpfad fließen und nicht versuchen, den Lehrer während dieses Distillationsschritts zu verändern.*

---

## 3. Zusammenfassung der PRISMS-Komponenten und des PRISM-Mechanismus

Das **PRISMS**-Framework kombiniert spezialisierte Module für eine effektive multimodale Fusion und Segmentierung:

*   **Modality Encoder:** Extrahiert unabhängige, hierarchische Merkmale für jede Eingangsmodalität.
*   **Adaptive Fusion Transformer (AFT) im PRISM Bottleneck:** Integriert initial globale Informationen von allen verfügbaren Modalitäten und lernbaren Fusions-Tokens mittels Modalitätsmaskierter Attention. Liefert transformierte Features und Attention-Scores.
*   **Spatial Weight Attention (SRA):** Nutzt die AFT-Attention-Scores, um die räumliche Relevanz von Modalitätsmerkmalen (sowohl direkte AFT-Ausgaben als auch Skip-Connections) zu gewichten und hervorzuheben.
*   **Cross-Modal Fusion Transformer (CMFT):** Besteht aus `MultiCrossToken`-Blöcken, die auf verschiedenen Decoder-Ebenen eine tiefe Interaktion und Re-Kalibrierung zwischen Fusions-Features und modalitätsspezifischen Skip-Connection-Features durchführen, gesteuert durch Maskierung.
*   **Modalitätsmaskierte Attention (MMA):** Ein Kernmechanismus in AFT und CMFT, der sicherstellt, dass Attention nur zwischen validen, vorhandenen Token-Paaren berechnet wird.
*   **Haupt-Fusionsdecoder:** Eine U-Net-ähnliche Architektur, die die fusionierten Ausgaben des CMFT und die SRA-gewichteten Skip-Connections verarbeitet, um die finale Segmentierung und Aux-Head-Vorhersagen zu erzeugen. Dient als "Lehrer" im PRISM-Mechanismus.
*   **PRISM-Distillationsmechanismus:**
    *   Nutzt den Haupt-Fusionsdecoder als **Lehrer**.
    *   Generiert **Schüler**-Pfade, indem die zentrale Fusionspipeline (AFT, SRA, CMFT, Haupt-Fusionsdecoder) konzeptionell mit unimodalen Masken durchlaufen wird.
    *   Wendet **Pixel-weisen KL Divergence Loss** und **Prototype Alignment Loss** an, um das Wissen vom Lehrer auf die Schüler zu übertragen und so die Robustheit der unimodalen Repräsentationen innerhalb des Fusionsnetzwerks zu stärken.
*   **Regulation Decoders:** Separate Decoder für jede Modalität, die direkt von den Encodern gespeist werden und einen zusätzlichen Regularisierungsverlust (\(L_{\text{reg}}\)) liefern.

Durch das Zusammenspiel dieser Komponenten und des PRISM-Distillationsmechanismus erreicht PRISMS eine hohe Segmentierungsgenauigkeit und Robustheit, insbesondere im Umgang mit der Herausforderung unvollständiger multimodaler medizinischer Bilddatensätze.

---
