---
title: "Unvollständige Trainingsdaten (UTD) in der Multimodalen Segmentierung"
description: "Ein Überblick darüber, was UTD ist, warum es wichtig ist, wie man damit umgeht und welche Herausforderungen zu beachten sind."
date: 2025-01-26
type: post
draft: false
translationKey: utd_article
coffee: 1
tags: ['UTD', 'Trainingsszenario']
categories: ['Modul']
---

<span class="letterine"><i>U</i>nvollständige Trainingsdaten</span> (kurz: UTD) sind in vielen realen Anwendungsfällen die Regel und nicht die Ausnahme. Man spricht von UTD, wenn in einem Datensatz **nicht** alle Modalitäten (oder Merkmale) vorhanden sind, die ursprünglich vorgesehen waren. Im medizinischen Umfeld kann dies beispielsweise bedeuten, dass für einige Patienten bestimmte MRT-Sequenzen nicht erfasst wurden.


![UTDvsVTD](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/VTD_UTD_Comp.png)


## Was ist UTD?

Man unterscheidet **unvollständige** von **vollständigen** Trainingsdaten (VTD).  
- **VTD:** Alle Modalitäten liegen für jeden Datensatzpunkt vor.  
- **UTD:** Mindestens eine oder mehrere Modalitäten fehlen teilweise oder vollständig.

Bei UTD kann es passieren, dass manche Modalitäten *konstant* fehlen (z. B. kostspielige oder zeitaufwändige MRT-Sequenzen) oder lediglich in einem Teil der Datensätze verfügbar sind.  

> **Beispiel**:  
> In einem Datensatz mit vier MRT-Sequenzen (T1, T1ce, T2, Flair) könnte es vorkommen, dass für 30 % aller Patienten keine T1 und T1ce-Aufnahmen angefertigt wurden. Man hat also *unvollständige* Informationen.


![VTD->UTD](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/vtd-utd.png)

## Warum UTD?

1. **Praktische Realität**:  
   In vielen Szenarien (z. B. Radiologie, Video-Surveillance) steht eine Modalität nicht immer zur Verfügung.  

2. **Zeit- und Kostenersparnis**:  
   Bestimmte Diagnostik-Verfahren (z. B. Kontrastmittel-MRT) sind aufwändig. In der Klinik wird oft entschieden, manche Sequenzen nur bei dringendem Verdacht durchzuführen.  

3. **Robuste Modelle**:  
   Ein Modell, das auch mit teils fehlenden Modalitäten umgehen kann, ist allgemein *robuster* und lässt sich leichter in der Praxis anwenden.


## Wie geht man mit UTD um?

Die folgenden Strategien sind verbreitet, um trotz unvollständiger Daten **valide** Modelle zu trainieren:

1. **Daten-Imputation (Bildrekonstruktion)**  
   - Fehlende Modalitäten werden *synthetisch* erzeugt (z. B. per GAN oder Diffusion).  
   - Kann funktionieren, benötigt jedoch oft vollmodale Beispiele (während einer Vortrainingsphase).

2. **Wissenstransfer (Distillation)**  
   - Ein Modell wird auf *vollständigen* Daten trainiert und dient als Lehrer.  
   - Ein Schülermodell lernt anschließend, *unimodale* Daten zu segmentieren.  
   - Problem: Erfordert in der Regel mindestens einige vollständig annotierte Fälle.

3. **Selbstdistillation und geteilte Repräsentationen**  
   - Ein *einziges* Netzwerk enthält mehrere Encoder (einen pro Modalität) und einen Fusionspfad.  
   - Verfügbares Wissen wird intern ausgetauscht.  
   - Fehlende Modalitäten werden seltener, aber gez
