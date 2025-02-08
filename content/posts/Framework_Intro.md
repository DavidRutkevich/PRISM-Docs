---
title: "PRISMS - Das Framework"
description: "Erläuterung zur Problemdefinition, Multi-Uni-Selbstdistillation und präferenzbasierten Regulierung in PRISM."
date: 2025-02-05
type: post
draft: false
translationKey: "methodik_prism_katex_storytelling"
coffee: 1
tags: ['PRISM','Methodik']
categories: ['Framework']
---

### 1. Problemdefinition: PRISMS und fehlende Modalitäten

Stellen wir uns vor, wir haben ein **multimodales** Szenario, in dem wir z. B. für jede Szene (oder jeden Patienten, jedes Fahrzeug etc.) mehrere Datenquellen besitzen: RGB-Bilder, Tiefenkarten, LiDAR-Daten, MRT-Sequenzen und so weiter. Jede Datenquelle bezeichnet man als eigene **Modalität**.  
Das **Ziel** ist eine robuste *Segmentierung*: jedem Pixel (oder Voxelknoten) soll eine Klasse zugeordnet werden (z. B. „Objekt A“, „Hintergrund“, „Tumor“, „gesundes Gewebe“, …). Idealerweise wollen wir alle verfügbaren Modalitäten für ein Modell nutzen – so kann man verschiedene Signalquellen zusammenführen und bessere Ergebnisse erzielen.

Doch die Praxis sieht oft anders aus:

1. **Inkomplette Modalitäten**: Häufig fehlen einzelne Modalitäten für bestimmte Beispiele. Etwa ist bei einigen Patienten keine T1-Sequenz des MRT vorhanden, oder beim autonomen Fahren ist mal die Tiefenkamera ausgefallen.  
2. **Ungleichverteilung**: Manche Modalitäten sind sehr häufig vorhanden, andere nur selten. Oder eine Modalität ist so „dominant“, dass ein naiver Multimodalfusions-Ansatz sich fast nur auf diese verlässt (Stichwort *unimodaler Bias*).

Ein **PRISMS**-System (als Kurzform für ein „Präferenzbasiertes, robusteres Anymodal-Segmentierungssystem“) soll damit umgehen können – also:

- **Beliebige Kombinationen** an verfügbaren Modalitäten nutzen,  
- Keinen drastischen Performance-Abfall zeigen, wenn bestimmte Modalitäten fehlen (selten oder nie verfügbar),  
- Im Idealfall sogar dann noch sinnvoll segmentieren, wenn nur **eine** Modalität vorhanden ist.

---

### 2. Warum ist das schwierig? (Unimodaler Bias)

Viele Systeme, die explizit mehrere Modalitäten fusionieren (z. B. ein Encoder pro Modalität + gemeinsamer Decoder), funktionieren gut, **sofern** alle Modalitäten vorliegen. Sobald jedoch wichtige Modalitäten wegfallen, sinkt die Genauigkeit häufig stark.  
Ursachen:

1. **Bias hin zu „einfachen“ Modalitäten**:  
   Manche Signale (z. B. RGB-Bilder) sind für das Netzwerk leichter zu verarbeiten. Modelle tendieren, „einfach Gelerntes“ zu übergewichten und andere Quellen zu vernachlässigen. Fehlt dann diese „dominante“ Quelle, wirkt sich das fatal aus.

2. **Kein Update für seltene Modalitäten**:  
   Wenn bestimmte Modalitäten selten sind (hohe Fehlrate \(\textit{FR}^m\)), erhalten sie im Training kaum Lerneinheiten. So entsteht eine *Ungleichverteilung* der Lernerfahrung pro Modalität.

3. **Unklare Kombinationslogik**:  
   Ein System, das nur „alles zusammen“ trainiert, weiß nicht, wie es reagieren soll, wenn einzelne Modalitäten plötzlich fehlen. Es hat keine Möglichkeit zu beurteilen, ob Informationen durch andere Sensoren kompensiert werden können.

---

### 3. PRISMS: Die Vision eines „jedem Input gewachsenen“ Segmentierers

**PRISMS** (Präferenzbasiertes robustes System für Anymodal-Segmentierung) strebt an:

1. **Robuste Verarbeitung**: Egal ob nur RGB vorliegt, nur Tiefe, nur LiDAR oder eine beliebige Kombination: das Modell soll weiter verlässliche Segmentierungen liefern.  
2. **Flexible Architektur**: Häufig nutzt man *Einzelpfade* pro Modalität plus einen Fusionspfad. Dieser Fusionspfad fungiert als „Lehrer“, während die Einzelpfade „Schüler“ sind (Selbstdistillation).  
3. **Ausbalancierte Gewichtung**: Eine Lernstrategie, die „dominante“ Modalitäten nicht übermäßig bevorzugt. Modalitäten, die seltener vorhanden sind oder schwerer zu lernen, erhalten spezielle **Regulierungs- oder Distillationsmechanismen**. So wird verhindert, dass sie komplett untergehen.

---

### 4. Reale Beispiele für PRISMS

- **Medizinische Bildgebung**: MRI-Scans mit T1-, T2-, FLAIR-Sequenzen. Ist eine Sequenz nicht verfügbar, soll das Modell trotzdem den Tumor segmentieren können.  
- **Autonomes Fahren**: Kamera, LiDAR, Radar, ggf. Wärmebild. Bei Schlechtwetter oder Hardwarefehlern kann die Kamera ausfallen, und der LiDAR-Input springt ein.  
- **Industrie-4.0-Inspektion**: Unterschiedlichste Sensorik (Röntgen, Infrarot, visuelles Licht). Eine Maschine soll defekte Bauteile selbst dann erkennen, wenn nicht alle Sensoren aktiv sind.

---

### 5. Kernelemente der Problemlösung

1. **(Selbst-)Distillation**:  
   Ein starker „multimodaler Pfad“ wird zum Lehrer, der den „unimodalen Pfaden“ (oder Teil-Fusionspfaden) beibringt, ähnliche Vorhersagen zu treffen.  
2. **Dynamische Gewichte**:  
   Wenn eine Modalität selten ist, kann man sie mit höheren Distillations-Gewichten fördern. Häufig verfügbare Modalitäten werden nicht unendlich bevorzugt.  
3. **Prototypen oder semantische Repräsentation**:  
   Neben Pixel-Logits ermöglichen globale Klassen-Prototypen, dass jede Modalität den „globalen Kontext“ einer Klasse versteht.  
4. **Plug-and-Play**:  
   Methoden wie **PRISMS** integrieren sich in Standard-Backbones (U-Net, Transformers, etc.), sodass kein massives Re-Engineering nötig ist.

---

### 6. Fazit: Warum dieses Problem so wichtig ist

**PRISMS** adressiert den wachsenden Bedarf, unvollständige Datenumgebungen zu bewältigen. Unsere moderne Welt ist reich an Sensoren – aber selten perfekt synchron und vollständig. Ein Segmentierungsmodell, das in jedem nur denkbaren Modus weiterlaufen kann, bietet:

- **Zuverlässigkeit**: Fehlende Modalitäten führen nicht zum Erkennungs-Aus.  
- **Kosteneffizienz**: Man muss den Betrieb nicht unterbrechen, sobald einer der Sensoren ausfällt.  
- **Skalierbarkeit**: Neue Sensoren/Modalitäten lassen sich in Zukunft leichter hinzufügen, ohne ein komplett neues Modell aufzubauen.

**Kurz gesagt**: Das Problem „PRISMS“ (im Sinne von einem robusten System für Anymodal-Segmentierung) ist kein Spezialfall, sondern eine Antwort auf die reale Datenwelt, in der viele unterschiedliche und teils fehlende Modalitäten vorliegen. Wer in der Praxis mit vielseitigen, aber lückenhaften Datensätzen arbeitet, profitiert von solchen Ansätzen enorm – ob in der Medizin, im autonomen Fahren oder in industriellen Anwendungsfeldern.