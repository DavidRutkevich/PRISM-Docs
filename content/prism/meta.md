---
title: "PRISM – Ablation"
description: "Ablation Studie von PRISM. Hier wird die Methodik von PRISM in ihren Einzelteilen betrachtet und analysiert."
date: 2025-02-05
math: true
---
## 1. Bi-Level Meta-Learning: Kernidee und Implementierung

### 1.1 Bi-Level Optimierung

Die Meta-Drop-Strategie nutzt eine **zweistufige (bi-level) Optimierung**. Sie trennt die Parameterupdates in zwei Ebenen auf:

- **Inner Loop** (Task-spezifische Anpassung):
  - Die Modellparameter \(\phi\) werden anhand eines Trainings-Batches \(D_{\text{train}}\) aktualisiert.
  - Anpassungsschritt (Gradient Descent auf Trainingsverlust \(\mathcal{L}_{\text{train}}\)):
\[
\phi_{\text{tilde}} \leftarrow \phi - \eta_{\text{in}} \nabla_\phi \mathcal{L}_{\text{train}}(\phi,D_{\text{train}})
\]

- **Outer Loop** (Meta-Level Anpassung):
  - Die Meta-Validierung erfolgt auf einem separaten Batch \(D_{\text{meta}}\) (mit potenziell abweichender Modalitätsverteilung), um die Generalisierungsfähigkeit zu prüfen.
  - Das finale Parameterupdate kombiniert den ursprünglichen Zustand \(\phi_0\) (vor dem Inner Loop) mit der task-spezifischen Anpassung \(\phi_{\text{tilde}}\), gewichtet über den Meta-Verlust:
\[
\phi \leftarrow \phi_0 + \alpha\,(\phi_{\text{tilde}} - \phi_0) - \eta_{\text{out}} \nabla_\phi \mathcal{L}_{\text{meta}}(\phi_{\text{tilde}}, D_{\text{meta}})
\]

Dabei sind \(\eta_{\text{in}}\), \(\eta_{\text{out}}\) Lernraten der jeweiligen Loops, und \(\alpha\) ein Skalierungsfaktor zur Rückführung.

---

## 2. Implementierung der Methode (Code-Detail)

Die Meta-Drop Methode ist in `train_meta_drop.py` realisiert:

**Inner Loop:**
```python
phi_0 = [p.clone() for p in model.parameters()]  # Ursprungsparameter sichern

# Inner Loop: Task-spezifische Anpassung
output_train = model(x_train, mask_train, target_train)
loss_train = calculate_loss(output_train, target_train)
optimizer.zero_grad()
loss_train.backward()
optimizer.step()

phi_tilde = [p.clone() for p in model.parameters()]  # Angepasste Parameter sichern
```

**Outer Loop (Meta-Validation und Parameterupdate):**
```python
# Meta-Validation
output_meta = model(x_meta, mask_meta, target_meta)
loss_meta = calculate_loss(output_meta, target_meta)

# Meta-Gradient berechnen
optimizer.zero_grad()
loss_meta.backward()

# Parameterupdate basierend auf Meta-Gradient
with torch.no_grad():
    for p, p_tilde, p0 in zip(model.parameters(), phi_tilde, phi_0):
        p.copy_(p0 + alpha * (p_tilde - p0) - eta_meta * p.grad)

# Zustand aktualisieren
phi_0 = [p.clone() for p in model.parameters()]
```

Diese Implementierung erlaubt dem Modell, simultan spezifisch (Inner Loop) und generalisierend (Outer Loop) zu lernen.

---

## 3. Verlustfunktionen im Meta-Drop Setting

Meta-Drop nutzt folgende Verluste:

**Fusion-Loss (Segmentierung):**
\[
\mathcal{L}_{\text{fuse}} = \text{CE}(y_{\text{fuse}},y) + \text{Dice}(y_{\text{fuse}},y)
\]

**Uni-Modal und PRM Losses (für Deep Supervision):**
\[
\mathcal{L}_{\text{uni}}^{(m)} = \text{CE}(y^{(m)},y) + \text{Dice}(y^{(m)},y)
\]

\[
\mathcal{L}_{\text{PRM}} = \sum_{s}\gamma_s[\text{CE}(y^{(s)},y) + \text{Dice}(y^{(s)},y)],\quad\gamma_s=2^{-s}
\]

**Self-Distillation und Proto-Regularisierung (PRISM-Komponenten):**
\[
\mathcal{L}_{\text{pixel}}^{(m)} = \sum_{l}\text{KL}(z_l^{(m)}\|z_l^{\text{fuse}})
\]

\[
\mathcal{L}_{\text{proto}}^{(m)} = \sum_{k}\|c_k^{(m)}-c_k^{\text{fuse}}\|_2^2
\]

Gesamtverlust (für Inner- und Meta-Loop):
\[
\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{fuse}} + \sum_{m}\beta_m\mathcal{L}_{\text{pixel}}^{(m)}+\delta_m\mathcal{L}_{\text{proto}}^{(m)}+\mathcal{L}_{\text{uni}}+\mathcal{L}_{\text{PRM}}
\]

---

## 4. Methodischer Vergleich PRISM (Meta-Drop) vs. PRISM v1

| Komponente              | PRISM (ohne Meta-Learning)                               | PRISM (Meta-Drop)                                                | Effekt und Vorteil v2                                     |
|-------------------------|-------------------------------------------------------------|----------------------------------------------------------------------|-----------------------------------------------------------|
| Trainingsprozess        | Ein-Stufen Training (Single-Level)                          | **Bi-Level** Training (Inner Task + Outer Meta)                      | Generalisierung auf unbekannte Modalitätsverteilungen    |
| Modalitäts-Maskierung   | Permanente Maskierung („utd“)                               | **Dynamische Maskierung** (Meta-Drop „utd_drop“)                     | Robuste Behandlung wechselnder Modalitätsmuster          |
| Parameteraktualisierung | Direkte Anpassung über Trainingsbatch                       | Parameteranpassung über Training & **Meta-Validierung**              | Vermeidung von Overfitting auf Trainingsmodalitäten      |
| Loss-Komponenten        | Fusion-, Uni-Modal, PRM, Pixel-, Proto-Loss                 | Gleiche Komponenten, jedoch **Meta-Level Optimierung**               | Anpassung aller Verluste auf Generalisierbarkeit         |
| Regularisierung         | Statische Gewichtung                                        | Adaptive Regularisierung via Meta-Gradient                           | Optimale Balance zwischen spezifischem und allgemeinem Wissen |

---

## 5. Fazit und Nutzen der Meta-Drop Methode

PRISM (Meta-Drop) erweitert die ursprüngliche PRISM-Architektur durch bi-level Meta-Learning, um die Robustheit und Generalisierung bei multimodalen Segmentierungsproblemen mit inhärent unvollständigen Trainingsdaten (ITD) zu erhöhen. Durch Kombination von task-spezifischen Updates (Inner Loop) und generalisierender Meta-Validierung (Outer Loop) lernt das Modell gleichzeitig spezifische Muster und bleibt robust gegenüber unbekannten Modalitätsverteilungen im Testeinsatz.

**Nutzen:**
- Erhöhte Generalisierungsfähigkeit auf klinisch relevante, unbekannte Modalitätskombinationen.
- Verbesserte Robustheit bei ungleichmäßig verteilten Fehlraten und unvollständigen Trainingsdaten.
- Klare methodische Weiterentwicklung gegenüber PRISM v1 mit nachweisbarer Praxisrelevanz.
