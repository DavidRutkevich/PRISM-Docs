---
title: "GSS"
description: "Guided Self-Supervision für robuste multimodale Hirntumorsegmentierung"
date: 2025-02-05
math: true
---

In der klinischen Praxis der Hirntumor-Bildgebung werden häufig mehrere MRT-Sequenzen (FLAIR, T1, T1ce, T2) verwendet, um komplementäre Informationen über Tumorgewebe zu erfassen. Leider sind in realen klinischen Szenarien oft nicht alle Modalitäten verfügbar oder weisen Qualitätsprobleme auf. Dies stellt eine erhebliche Herausforderung für automatisierte Segmentierungsalgorithmen dar, die typischerweise auf vollständige Datensätze trainiert wurden.

Die Guided Self-Supervision (GSS) Methode adressiert dieses Problem, indem sie ein innovatives Wissenstransfer-Paradigma zwischen verfügbaren und fehlenden Modalitäten implementiert. GSS ist Teil des PRISM-Frameworks zur multimodalen Segmentierung unter inhärent unvollständigen Trainingsdaten.

## Kernkonzept: Konfidenzbasierte Lehrersignalgenerierung

Das zentrale Innovationselement von GSS ist die intelligente Erzeugung von Lehrersignalen aus verfügbaren Modalitäten, die dann zur Anleitung der Vorhersagen für fehlende Modalitäten dienen. Die Methode unterscheidet dabei zwischen verschiedenen Konfidenzniveaus:

### 1. Region-spezifische Signalgenerierung

GSS teilt das Bild in drei verschiedene Regionen basierend auf der Konfidenz der verfügbaren Modalitäten:

```python
# Regionen mit hoher Konfidenz (bei verfügbarem FLAIR)
pred_mask_l = flair_pred > 0.65

# Regionen mit mittlerer Konfidenz
pred_mask_m = (((t1ce_pred > 0.75)*mask[0,1].float() +
               (t1_pred > 0.75)*mask[0,2].float() +
               (t2_pred > 0.75)*mask[0,3].float()) ==
              (mask[0,1].float()+mask[0,2].float()+mask[0,3].float()))*(flair_pred <= 0.65)

# Regionen mit niedriger Konfidenz
pred_mask_s = (all_one - pred_mask_l - pred_mask_m) == all_one
```

Die Schwellenwerte wurden empirisch optimiert: 0,65 für FLAIR und 0,75 für andere Modalitäten.

### 2. FLAIR-priorisierender Ansatz

FLAIR wird in der Methode bevorzugt behandelt, da diese Sequenz besonders wichtige Informationen für bestimmte Tumorregionen liefert:

```python
if mask[0,0]:  # Wenn FLAIR verfügbar ist
    if mask[0,1]|mask[0,2]|mask[0,3]:  # Wenn auch andere Modalitäten verfügbar sind
        # Für Regionen mit hoher Konfidenz: Direktes FLAIR verwenden
        # Für Regionen mit mittlerer Konfidenz: Gewichteter Durchschnitt anderer Modalitäten
        # Für Regionen mit niedriger Konfidenz: Minimum über alle Modalitäten
        teacher_pred = pred_mask_l*flair_pred + pred_mask_m*(gewichteter_durchschnitt) + pred_mask_s*pred_s
    else:
        # Nur FLAIR ist verfügbar
        teacher_pred = flair_pred
else:  # FLAIR nicht verfügbar
    # Nur mittlere und niedrige Konfidenzregionen berücksichtigen
    # Verwende gewichteten Durchschnitt und Minimum der verfügbaren Modalitäten
    # ...
```

### 3. Konservative Behandlung unsicherer Regionen

Für Regionen mit niedriger Vorhersagekonfidenz verwendet GSS einen konservativen Ansatz, indem das Minimum der Vorhersagewahrscheinlichkeiten über alle verfügbaren Modalitäten berechnet wird:

```python
for j in range(4):  # Für jede Tumorregion (Hintergrund, WT, TC, ET)
    pred_s[0,j,:] = torch.amin(torch.cat((pred_flair[0,j,:], pred_t1ce[0,j,:], pred_t1[0,j,:], pred_t2[0,j,:]), dim=1), dim=1)
```

Dies reduziert falsch-positive Vorhersagen in unsicheren Bereichen und erhöht die Robustheit der Segmentierung.

## Wissenstransfer durch Knowledge Distillation

Nach der Generierung der Lehrervorhersagen implementiert GSS einen Wissenstransfer zu den modalitätsspezifischen Vorhersagen mittels Kullback-Leibler-Divergenz:

```python
# Anwendung von KL-Divergenz für jede Modalität
kl0 = criterions.notemp_kl_loss_bs(flair_pred, teacher_pred.detach(), target, num_cls=num_cls, temp=temp)
kl1 = criterions.notemp_kl_loss_bs(t1ce_pred, teacher_pred.detach(), target, num_cls=num_cls, temp=temp)
kl2 = criterions.notemp_kl_loss_bs(t1_pred, teacher_pred.detach(), target, num_cls=num_cls, temp=temp)
kl3 = criterions.notemp_kl_loss_bs(t2_pred, teacher_pred.detach(), target, num_cls=num_cls, temp=temp)
```

Besonders wichtig ist die Verwendung von `.detach()`, wodurch der Gradientenfluss in die Lehrervorhersagen verhindert wird. Dies stellt sicher, dass die Lehrer stabile Lernziele darstellen.

### Selektive Anwendung durch Modalitätsmasken

GSS wendet den Wissenstransfer selektiv nur auf tatsächlich verfügbare Modalitäten an:

```python
# Anwendung der Modalitätsmaske auf KL-Verluste
kl_loss = torch.cat((kl0, kl1, kl2, kl3), dim=1)
kl_loss_m = torch.sum(kl_loss*mask, dim=0)
term_kl = kl_loss_m.sum()
```

Dies verhindert, dass fehlende Modalitäten Gradienten empfangen und ermöglicht effektives Training auch bei stark unausgewogenen Daten.

## Mehrteilige Verlustfunktion

Die Gesamt-Verlustfunktion kombiniert modalitätsspezifische Dice-Verluste mit dem KL-Divergenz-Verlust:

```python
loss = torch.sum(torch.stack(losses, dim=0)) + 0.5 * term_kl
```

Der KL-Divergenz-Verlust wird mit 0,5 gewichtet, um eine Balance zwischen Segmentierungsgenauigkeit (Dice-Verlust) und Wissenskonsistenz zwischen den Modalitäten zu erreichen.

## Vergleich mit anderen Ansätzen

### GSS vs. Meta-Learning (Meta-Drop)

Im Gegensatz zum Meta-Drop-Ansatz, der eine Bi-Level-Optimierung mit inneren Task-Loops und äußeren Meta-Loops verwendet, implementiert GSS einen direkteren Wissenstransfer innerhalb jedes Trainingsschritts:

| Aspekt | Meta-Drop | GSS |
|--------|-----------|-----|
| Kernmethodik | Bi-Level Meta-Learning | Konfidenzbasierte Knowledge Distillation |
| Optimierung | Zwei Optimierungsebenen | Einstufige Optimierung mit geführtem Wissen |
| Adaptivität | Dynamische Modalitätsmasken in Meta-Validierungsphase | Konfidenzbasierte regionsspezifische Fusion |
| Zielsetzung | Generalisierung über Modalitätsverteilungen | Effektiver Wissenstransfer zwischen verfügbaren und fehlenden Modalitäten |

### GSS vs. Shared-Specific Architektur

Die GSS-Methodik kann mit verschiedenen Netzwerkarchitekturen kombiniert werden. Besonders effektiv ist die Kombination mit Shared-Specific-Architekturen (`train_shaspec_gss.py`), die zwischen modalitätsspezifischen und gemeinsamen Merkmalen unterscheiden.

## Technische Implementierungsdetails

### Modalitätsmaskierung

GSS berücksichtigt 15 verschiedene Kombinationen fehlender Modalitäten:

```python
masks_test = [[False, False, False, True], [False, True, False, False], ...]
# [FLAIR, T1ce, T1, T2] - False bedeutet Modalität ist verfügbar
```

### Normalisierung der Lehrervorhersagen

Nach der Generierung werden die Lehrervorhersagen normalisiert, um gültige Wahrscheinlichkeitsverteilungen zu gewährleisten:

```python
teacher_pred = torch.clamp(teacher_pred, min=0.005, max=1)
teacher_pred = teacher_pred / torch.sum(teacher_pred, dim=1).unsqueeze(1)
```

Der Minimalwert von 0,005 verhindert numerische Instabilitäten bei der KL-Divergenzberechnung.

### Anwendung auf verschiedene Modellarchitekturen

Die GSS-Methodik kann mit verschiedenen Backbone-Modellen verwendet werden:

```python
if args.model == 'mmformer':
    model = mmformer.Model(num_cls=num_cls)
elif args.model == 'rfnet':
    model = rfnet.Model(num_cls=num_cls)
elif args.model == 'm2ftrans':
    model = m2ftrans.Model(num_cls=num_cls)
# ...
```

## Fazit und klinische Bedeutung

Die Guided Self-Supervision (GSS) Methode stellt einen bedeutenden Fortschritt für die klinische Anwendbarkeit automatisierter Hirntumorsegmentierung dar. Durch die konfidenzbasierte Fusion und den modalitätsübergreifenden Wissenstransfer ermöglicht GSS:

1. **Robuste Segmentierung bei fehlenden Modalitäten**: Die Methode kann hochwertige Segmentierungen liefern, selbst wenn bestimmte MRT-Sequenzen nicht verfügbar sind.

2. **Verbessertes Training mit unausgewogenen Daten**: GSS nutzt alle verfügbaren Informationen optimal, selbst wenn die Verteilung der Modalitäten im Trainingsdatensatz stark unausgewogen ist.

3. **Regionsspezifische Adaptivität**: Durch die unterschiedliche Behandlung von Regionen mit hoher, mittlerer und niedriger Konfidenz erreicht GSS eine feinkörnige Adaptivität, die besonders bei heterogenen Tumoren wichtig ist.

Die Integration von GSS in klinische Workflows könnte die Verlässlichkeit und Flexibilität automatisierter Segmentierungstools deutlich verbessern und somit zu präziseren Diagnosen und Behandlungsplanungen beitragen.
