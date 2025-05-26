---
title: "Prisms Fusion"
description: "Infrarot und RGB fusion"
date: 2025-02-05
math: true
---

Die Fusion von Bilddaten aus unterschiedlichen Sensoren, wie RGB-Kameras und Infrarotkameras, ist ein etabliertes Verfahren zur Erzeugung umfassenderer und robusterer Szenenrepräsentationen. Während RGB-Bilder detaillierte Farb- und Texturinformationen bei adäquaten Lichtverhältnissen liefern, ermöglichen Infrarotbilder die Erfassung thermischer Signaturen und sind auch bei Dunkelheit oder eingeschränkten Sichtbedingungen operabel. Die Herausforderung besteht darin, diese komplementären Informationen effektiv zu kombinieren, insbesondere wenn einer der Datenströme qualitativ beeinträchtigt ist oder gänzlich fehlt. Herkömmliche Fusionsalgorithmen erreichen hier oft ihre Grenzen.

**PRISMS-HybridFusion: Ein Framework zur adaptiven RGB-Infrarot-Fusion**

PRISMS-HybridFusion ist ein Architekturkonzept, das auf den Prinzipien des PRISM-Frameworks aufbaut und diese um spezialisierte Komponenten für die Fusion von RGB- und Infrarotbildern erweitert. Ziel ist eine robuste Fusion auch bei unvollständigen oder qualitativ variierenden Eingangsdaten.

1.  **Modalitätsspezifische Merkmalsextraktion und -aufbereitung (*prisms_fus*):**
    Eine effektive Fusion erfordert eine optimierte Vorbereitung der modalitätsspezifischen Merkmale. Die Komponente *prisms_fus* ist für diese Aufgabe zuständig:
    *   **RGB-Aufbereitung:** Für das sichtbare Bild (RGB) fokussiert sich *prisms_fus* auf die Disentanglement von Beleuchtungsinformationen. Inspiriert von Ansätzen wie der Retinex-Theorie, wird versucht, die Reflektanzeigenschaften der Objekte von den Umgebungslichtbedingungen zu trennen. Dies ist insbesondere bei geringer Beleuchtung vorteilhaft, um die inhärenten visuellen Details zu verstärken, bevor sie in den Fusionsprozess einfließen. Der Luminanzkanal (Y) des RGB-Bildes wird hierfür typischerweise extrahiert und verarbeitet.
    *   **Infrarot-Aufbereitung:** Für das Infrarotbild zielt *prisms_fus* darauf ab, die strukturelle Integrität der Merkmale zu maximieren. Dies kann durch spezialisierte Faltungsnetzwerke geschehen. Zusätzlich kann eine Rekonstruktionsaufgabe (z.B. die Rekonstruktion des Eingangs-IR-Bildes aus seinen extrahierten Merkmalen) als Regularisierungsmechanismus dienen, um die Qualität und Robustheit der gelernten IR-Features sicherzustellen.
    Die resultierenden aufbereiteten Merkmale beider Pfade bilden die Grundlage für die nachfolgende Fusionsstufe.

2.  **Kernfusion mittels Adaptive Fusion Transformer (AFT):**
    Der AFT ist die zentrale Komponente für die Verschmelzung der aufbereiteten RGB- und Infrarot-Merkmale.
    *   **Lernbare Fusionstokens:** Anstelle fester Fusionsregeln nutzt der AFT lernbare Parametervektoren, sogenannte "Fusionstokens". Diese Tokens agieren als dynamische Repräsentationen, die während des Trainings lernen, die relevantesten und komplementärsten Informationen aus beiden Modalitäten zu extrahieren und zu einer kohärenten, fusionierten Merkmalsdarstellung zu integrieren.
    *   **Modalitätsmaskierte Attention (MMA):** Eine Schlüsselkomponente des AFT ist die MMA. Diese erlaubt es dem Transformer, flexibel auf die Verfügbarkeit und Qualität der Eingangssignale zu reagieren. Eine binäre Maske informiert den Transformer über den Status jeder Modalität (z.B. RGB stark verrauscht, Infrarot nicht verfügbar). Die MMA stellt sicher, dass der Attention-Mechanismus des Transformers seine "Aufmerksamkeit" primär auf die verlässlichen Informationsquellen lenkt. Fehlende oder stark gestörte Daten werden so in ihrem Einfluss auf die Fusion gedämpft oder ignoriert. Die Fusionstokens aggregieren somit effektiv Informationen aus den *tatsächlich nutzbaren* Anteilen der RGB- und Infrarot-Features.

3.  **Nachverfeinerung der fusionierten Merkmale (SRA & KFT):**
    Die vom AFT erzeugte fusionierte Merkmalsrepräsentation wird durch weitere spezialisierte Transformer-basierte Module optimiert:
    *   **Spatial Relevance Attention (SRA):** Dieses Modul analysiert die räumliche Verteilung der fusionierten Merkmale und gewichtet Regionen basierend auf ihrer Relevanz für das Gesamtverständnis der Szene. Dies ermöglicht eine Fokussierung auf wichtige Objekte oder Strukturen im fusionierten Merkmalsraum.
    *   **Kanalbezogener Fusionstransformer (KFT):** Der KFT operiert auf der Kanalebene der fusionierten Merkmale. Seine Aufgabe ist es, Redundanzen zwischen den verschiedenen Feature-Kanälen zu reduzieren und diejenigen Kanäle zu verstärken, die die aussagekräftigsten und komplementärsten fusionierten Informationen tragen.

4.  **Generierung des fusionierten Bildes mittels spezialisiertem Decoder:**
    Ein finaler Decoder-Teil ist für die Rekonstruktion des fusionierten Bildes aus den optimierten Merkmalen zuständig. Dieser Decoder ist strukturell an Architekturen wie dem U-Net angelehnt und nutzt Skip-Connections, um Detailinformationen aus früheren Verarbeitungsstufen der *prisms_fus*-Encoder einzubinden:
    *   **Eingang (Bottleneck):** Die vom AFT (und ggf. SRA/KFT) prozessierten, fusionierten Merkmale, die aus den Fusionstokens abgeleitet wurden, bilden den Eingang im tiefsten Punkt (Bottleneck) des Decoders.
    *   **Skip-Connections und adaptive Gewichtung:** Merkmale aus den Encoder-Stufen der *prisms_fus*-Komponente (sowohl vom RGB- als auch vom IR-Pfad) werden über Skip-Connections zu den entsprechenden Auflösungsstufen im Decoder geleitet. Entscheidend ist hierbei eine adaptive Gewichtung dieser Skip-Merkmale. Inspiriert durch Mechanismen wie SWA (Spatial Weight Attention), kann die Zuverlässigkeit oder Verfügbarkeit jeder Modalität (ggf. abgeleitet aus der MMA im AFT oder einer expliziten Qualitätsabschätzung) genutzt werden, um den Beitrag der jeweiligen Skip-Merkmale zu steuern. Ist eine Modalität als unzuverlässig oder fehlend markiert, werden ihre Skip-Merkmale entsprechend heruntergewichtet oder ignoriert.
    *   **Progressive Fusion und Upsampling:** In jeder Decoderstufe werden die hochgesampelten Merkmale aus der tieferen Ebene mit den (adaptiv gewichteten) Skip-Merkmalen der aktuellen Auflösungsebene konkateniert und durch Faltungsschichten weiterverarbeitet.
    *   **Ausgabe des Luminanzkanals:** Der Decoder generiert typischerweise den fusionierten Luminanzkanal (Y-Kanal) des Bildes, da dieser die wesentlichen Struktur- und Helligkeitsinformationen enthält.
    *   **Finale RGB-Bildsynthese:** Der fusionierte Y-Kanal wird anschließend mit den Chrominanzkanälen (Cb, Cr) kombiniert. Diese Chrominanzkanäle können entweder direkt vom ursprünglichen RGB-Eingangsbild stammen oder, für potenziell bessere Farbkonsistenz, von dem durch die *prisms_fus*-Komponente aufbereiteten (z.B. beleuchtungsoptimierten) RGB-Bild. Das Ergebnis ist das finale fusionierte Farbbild.

**Technische Vorteile von PRISMS-HybridFusion:**

*   **Verbesserte Bildqualität:** Erzeugung fusionierter Bilder mit hoher Detailgenauigkeit und Informationsdichte durch spezialisierte Aufbereitung und adaptive Fusion.
*   **Robustheit bei Datenvariabilität:** Konsistente Fusionsergebnisse auch bei signifikanten Qualitätsunterschieden oder dem Fehlen einer der Eingangsmethoden (RGB oder Infrarot).
*   **Adaptive Fusionsstrategie:** Das System lernt dynamisch, die Beiträge der verfügbaren Modalitäten optimal zu gewichten und zu integrieren, basierend auf ihrer aktuellen Verlässlichkeit.
*   **Erhöhte Systemzuverlässigkeit:** Von besonderer Bedeutung für sicherheitskritische Anwendungen, die eine durchgängig verlässliche Perzeption erfordern.

**Anwendungsfelder:**

Die technischen Eigenschaften von PRISMS-HybridFusion prädestinieren es für anspruchsvolle Anwendungen:

*   **Autonomes Fahren:** Zuverlässige Umfelderkennung unter variablen Licht- und Wetterbedingungen.
*   **Sicherheits- und Überwachungstechnik:** Effektive Detektion und Verfolgung von Objekten.
*   **Industrielle Inspektion und Qualitätskontrolle:** Kombination visueller Oberflächenanalyse mit thermischer Prozessüberwachung.
*   **Robotik und Mensch-Maschine-Interaktion:** Verbesserte Szeneninterpretation für autonome Systeme.

PRISMS-HybridFusion stellt einen methodischen Ansatz dar, um die Herausforderungen der RGB-Infrarot-Bildfusion, insbesondere bei unvollständigen Daten, durch adaptive, Transformer-basierte Mechanismen und modalitätsspezifische Vorverarbeitung zu adressieren.
