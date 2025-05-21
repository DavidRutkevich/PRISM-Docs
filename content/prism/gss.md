---
title: "GSS"
description: "Guided Self-Supervision für robuste multimodale Hirntumorsegmentierung"
date: 2025-02-05
math: true
---

```mermaid
flowchart TD
    subgraph InputData["Multi-Modale Eingabe"]
        I1["Multi-modale MRT<br/>(FLAIR, T1, T1ce, T2)"] --> I2["Modalitätsmaske<br/>(Verwaltung fehlender Modalitäten)"]
        I2 --> I3["Binäre Masken verfolgen<br/>verfügbare Modalitäten"]
    end
    
    subgraph ForwardPass["Encoder-Decoder Netzwerk"]
        FP1["Modalitätsspezifische<br/>Encoder"] --> FP2["Merkmalsextraktion<br/>& Fusion"]
        FP2 --> FP3["Individuelle Modalitäts-<br/>vorhersagen"]
        FP3 --> FP4["Softmax-Anwendung<br/>für jede Modalität"]
    end
    
    subgraph GSS["Gruppenspezifische Selbstüberwachung"]
        direction TB
        G1["Erzeugung von Konfidenzmasken"]
        
        subgraph RegionDetection["Regionsklassifikation"]
            RD1["Hohe Konfidenz<br/>(Vorh. > 0,65)"]
            RD2["Mittlere Konfidenz<br/>(Vorh. > 0,75 in anderen<br/>verfügbaren Modalitäten)"]
            RD3["Niedrige Konfidenz<br/>(Verbleibende Regionen)"]
        end
        
        subgraph RegionFusion["Adaptive Fusionsstrategie"]
            RF1["Verwendung von FLAIR<br/>Primäre Modalität"]
            RF2["Gewichteter Durchschnitt<br/>verfügbarer Modalitäten"]
            RF3["Minimale Vorhersage<br/>(Konservative Fusion)"]
        end
        
        subgraph TeacherGen["Lehrer-Modell-Generierung"]
            TG1["Fusionierung basierend auf<br/>Konfidenzregionen"]
            TG2["Wertebegrenzung<br/>(0,005, 1,0)"]
            TG3["Normalisierung der<br/>Wahrscheinlichkeitsverteilung"]
        end
        
        G1 --> RegionDetection
        RD1 --> RF1
        RD2 --> RF2
        RD3 --> RF3
        RF1 --> TG1
        RF2 --> TG1
        RF3 --> TG1
        TG1 --> TG2 --> TG3
    end
    
    subgraph LossCalc["Mehrkomponenten-Verlustfunktion"]
        L1["Dice-Verlust<br/>(pro Modalität<br/>Segmentierung)"]
        L2["KL-Divergenz-Verlust<br/>(Wissensdestillation<br/>Gewicht: 0,5)"]
        L3["Kombinierter Verlust"]
        
        L1 --> L3
        L2 --> L3
    end
    
    subgraph GMDOpt["GMD-Optimierung"]
        GMD1["Berechnung aufgaben-<br/>spezifischer Gradienten"]
        GMD2["Erkennung & Lösung<br/>von Gradientenkonflikten"]
        GMD3["Parameter-Aktualisierung"]
        
        GMD1 --> GMD2 --> GMD3
    end
    
    InputData --> ForwardPass
    ForwardPass --> FP4
    FP4 --> G1
    GSS --> |"Lehrer-Modell"| L2
    FP4 --> |"Schüler-Modelle"| L1
    L3 --> GMDOpt
```

### Schlüsselkomponenten

| Komponente | Implementierungsdetails |
|------------|-------------------------|
| **Datenvorbereitung** | - Binäre Masken verfolgen verfügbare Modalitäten<br>- Verarbeitet beliebige Kombinationen fehlender Modalitäten<br>- Unterstützt unausgewogene Datensätze mit unterschiedlicher Verfügbarkeit |
| **Multi-Branch-Netzwerk** | - Modalitätsspezifische Encoder-Zweige<br>- Gemeinsame Cross-Attention-Fusionsmodule<br>- Spezialisierte Decoder für jede Modalität |
| **Konfidenzbasierte Regionsklassifikation** | - **Regionen mit HOHER Konfidenz (>0,65)**: Bereiche, in denen die FLAIR-Modalität starke Vorhersagen hat<br>- **Regionen mit MITTLERER Konfidenz (>0,75)**: Bereiche, in denen andere verfügbare Modalitäten stärkere Vorhersagen haben<br>- **Regionen mit NIEDRIGER Konfidenz**: Bereiche mit Unsicherheit über alle Modalitäten hinweg |
| **Adaptive Fusionsstrategie** | - HOHE Konfidenz: Verwendung der primären Modalität (FLAIR)<br>- MITTLERE Konfidenz: Gewichteter Durchschnitt verfügbarer Modalitäten<br>- NIEDRIGE Konfidenz: Minimale Vorhersage (konservativster Ansatz) |
| **Wissensdestillations-Pipeline** | - Lehrer-Modell durch optimale Fusion kombinierter Regionen erstellt<br>- Begrenzung und Normalisierung gewährleisten gültige Wahrscheinlichkeitsverteilung<br>- Schülermodelle lernen vom Lehrer durch KL-Divergenz-Verlust |
| **Verlustkomponenten** | - Dice-Verlust für direkte Segmentierungsüberwachung<br>- KL-Divergenz-Verlust für Wissensdestillation (Gewicht: 0,5)<br>- Kombinierte Optimierung durch GMD (Lösung von Gradientenkonflikten) |

## Gradient-Manipulation für Deep Learning (GMD)

Zur Optimierung unseres Multi-Task-Lernansatzes verwenden wir GMD, eine Technik zur effektiven Handhabung von Gradientenkonflikten:

1. **Erkennung von Konflikten**: Identifiziert widersprüchliche Gradienten zwischen verschiedenen Aufgaben durch negative Skalarprodukte
2. **Projektionsbasierte Lösung**: Projiziert konfliktbehaftete Gradientenkomponenten, um destruktive Interferenz zu verhindern
3. **Spezifische Parameterbehandlung**: Unterschiedliche Handhabung von gemeinsam genutzten vs. aufgabenspezifischen Parametern

## Vorteile unseres Ansatzes

- **Robust bei fehlenden Daten:** Funktioniert effektiv mit jeder Kombination verfügbarer Modalitäten
- **Regionsadaptives Lernen:** Wendet unterschiedliche Fusionsstrategien basierend auf Konfidenzleveln in verschiedenen Regionen an
- **Modalitätsübergreifender Wissenstransfer:** Modalitätsspezifische Zweige profitieren von Informationen aus anderen Modalitäten
- **Konfliktfreie Optimierung:** GMD verhindert destruktive Gradienteninterferenz zwischen Aufgaben
- **Klinische Anwendbarkeit:** Arbeitet mit realistischen klinischen Szenarien, in denen Daten unvollständig sind
- **Leistungserhaltung:** Behält hohe Leistung auch bei erheblich fehlenden Daten bei

## Implementierungsdetails

Die Lehrer-Vorhersage wird durch ein dreistufiges Konfidenzsystem sorgfältig konstruiert:

```python
# Hohe Konfidenz: FLAIR-Vorhersage direkt verwenden
pred_mask_l = flair_pred > 0.65

# Mittlere Konfidenz: Gewichteten Durchschnitt verfügbarer Modalitäten verwenden
pred_mask_m = ((t1ce_pred > 0.75)*mask[0,1] + (t1_pred > 0.75)*mask[0,2] + 
               (t2_pred > 0.75)*mask[0,3]) == (mask[0,1]+mask[0,2]+mask[0,3])

# Niedrige Konfidenz: Minimale Vorhersagen verwenden (konservativster Ansatz)
pred_mask_s = (all_one - pred_mask_l - pred_mask_m) == all_one
teacher_pred = pred_mask_l*flair_pred + pred_mask_m*weighted_average + pred_mask_s*minimum_pred
```

