---
title: "Gradient Merging Descent (GMD) – Optimierung bei konkurrierenden Lernzielen"
description: "Detaillierte Analyse und Implementierung von GMD zur Auflösung von Gradientenkonflikten im Multi-Objective- und Multi-Task-Learning."
date: 2025-05-02
math: true
---

In vielen modernen Deep-Learning-Szenarien wie **Multi-Task-Learning** oder **Deep-Supervision** entstehen mehrere Verlustfunktionen \(L_1, L_2, \dots, L_K\), die gleichzeitig minimiert werden sollen. Ein gängiges Beispiel ist die multimodale medizinische Segmentierung, bei der pro Modalität eine eigene Loss-Komponente berechnet wird. 

**Problem:**  
Die Gradienten \(\nabla L_i\) können in Konflikt geraten—ein Schritt, der einen Loss reduziert, kann zeitgleich einen anderen Loss erhöhen. Naive Gradientenaggregation (Summe oder Mittel) ignoriert diese Konflikte und führt zu instabilen Trainingstrajektorien oder zur Dominanz einzelner Ziele.

**Lösung:**  
Gradient Merging Descent (GMD) erkennt und beseitigt solche Konflikte **vor** der Parameteraktualisierung durch:

1. **Paarweise Projektion** konfliktierender Gradienten  
2. **Reduktion** der projektierten Gradienten zu einem einzigen, konsistenten Update  

---

## 1. Algorithmusübersicht

Der Ablauf von GMD gliedert sich in fünf Schritte:

1. **Separates Backpropagation**  
   Für jedes Loss-Objektiv \(L_i\) wird der Gradient \(\mathbf{g}_i\) einzeln berechnet:

   \[
   \mathbf{g}_i = \text{flatten}\bigl(\nabla_\theta L_i(\theta)\bigr)\quad i=1\ldots K.
   \]

2. **Konflikterkennung**  
   Zwei Gradienten \(\mathbf{g}_i\) und \(\mathbf{g}_j\) stehen im Konflikt, wenn

   \[
   \mathbf{g}_i^\top \mathbf{g}_j < 0.
   \]

3. **Konfliktprojektion**  
   Bei einem negativen Skalarprodukt wird der Anteil von \(\mathbf{g}_i\) entlang \(\mathbf{g}_j\) entfernt:

   \[
   \alpha_{ij} = \frac{\mathbf{g}_i^\top \mathbf{g}_j}{\|\mathbf{g}_j\|^2},\quad
   \mathbf{g}_i \leftarrow \mathbf{g}_i - \operatorname{clamp}(\alpha_{ij}, -2, 1)\,\mathbf{g}_j.
   \]

4. **Gradient Merging**  
   Definiere eine Binärmaske \(m_k\in\{0,1\}\), die anzeigt, ob Parameter \(k\) von **allen** Objectives betroffen ist. Dann:

   \[
   \tilde g_k = 
   \begin{cases}
     \displaystyle\frac{1}{K}\sum_{i=1}^K g_{i,k}, & \text{if }m_k=1,\\[1ex]
     \displaystyle\sum_{i=1}^K g_{i,k},        & \text{if }m_k=0.
   \end{cases}
   \]

5. **Parameter-Update**  
   Der so entstandene Gesamtgradient \(\tilde{\mathbf{g}}\) wird in die Gradientenfelder zurückgeschrieben und der Basis-Optimizer (z. B. Adam) nimmt den Schritt:

   ```python
   self._set_grad(unflatten(tilde_g, shapes))
   optimizer.step()
   ```

## 2. Detaillierte Implementierung

### 2.1 Gradienten-Packing

```python
def _pack_grad(self, objectives, ddp_model=None):
    grads, shapes, has_grads = [], [], []
    for obj in objectives:
        self._optim.zero_grad(set_to_none=True)
        obj.backward(retain_graph=True)
        grad_list, shape_list, mask_list = self._retrieve_grad()
        grads.append(self._flatten_grad(grad_list, shape_list))
        has_grads.append(self._flatten_grad(mask_list, shape_list))
        shapes.append(shape_list)
    return grads, shapes, has_grads
```

- **`_retrieve_grad()`** liefert pro Parameter Tensor, deren Gradienten und eine Maske, ob ein Grad existiert.
- **`_flatten_grad()`** konkateniert alle Parameter in einen Vektor.

### 2.2 Konfliktauflösung

```python
def _project_conflicting(self, grads, has_grads):
    shared = torch.stack(has_grads).prod(0).bool()
    pc_grad = copy.deepcopy(grads)
    for i, g_i in enumerate(pc_grad):
        for j, g_j in enumerate(grads):
            dot = torch.dot(g_i, g_j)
            if dot < 0:
                coef = dot / (g_j.norm()**2)
                coef = torch.clamp(coef, min=-2.0, max=1.0)
                g_i -= coef * g_j
    # Logging der Koeffizienten optional über self.writer
    # ...
    # Merging:
    merged = torch.zeros_like(grads[0])
    merged[shared] = torch.stack([g[shared] for g in pc_grad]).mean(dim=0)
    merged[~shared] = torch.stack([g[~shared] for g in pc_grad]).sum(dim=0)
    return merged
```

- **Randomisierung** der Aktualisierungsreihenfolge kann eingebaut werden, um Bias zu reduzieren.
- **Clamp** begrenzt extrem große Projektionen.

### 2.3 Gradienten zurückschreiben

```python
def _set_grad(self, grads):
    idx = 0
    for group in self._optim.param_groups:
        for p in group['params']:
            p.grad = grads[idx].view(p.shape).clone()
            idx += 1
```

## 3. Vorteile und Einsatzszenarien

- **Robustheit gegen Gradientenkonflikte**  
  Verhindert, dass starker Fortschritt bei \(L_i\) den Fortschritt bei \(L_j\) komplett aufhebt.
- **Balanciertes Training**  
  Alle Objectives tragen gleichberechtigt zum Update bei.
- **Anwendungsfälle**  
  - Multi-Task-Netzwerke mit geteilten Backbone-Parametern  
  - Deep-Supervision (z. B. PRISM-Strukturen)  
  - Multi-Modal-/Multimodal-Segmentierung  

---

## 4. Fazit

Gradient Merging Descent (GMD) ist eine effektive Erweiterung klassischer Optimierer, um **konkurrierende Lernziele** harmonisch zu vereinen. Durch paarweises Projektions- und Zusammenführungsverfahren wird ein stabiler, konfliktarmer Gesamtgradient erzeugt, der in realen Multi-Task- und Multi-Objective-Szenarien zu konsistenteren und oft besseren Konvergenzergebnissen führt.