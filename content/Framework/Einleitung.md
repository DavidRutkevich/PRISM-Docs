---
title: Einleitung
description: "Einführung in die multimodale Segmentierung und die Herausforderungen bei unvollständigen Daten."
math: true
---

## Einleitung

Die multimodale Segmentierung hat in den letzten Jahren erheblich an Bedeutung gewonnen, da sie in vielen Anwendungsbereichen – von der medizinischen Bildgebung über autonomes Fahren bis hin zur industriellen Qualitätskontrolle – essenziell ist. Im Kern geht es darum, Informationen aus unterschiedlichen Sensoren oder Bildquellen (z. B. RGB, Tiefenkarten, LiDAR, MRT) zu kombinieren, um eine detaillierte und robuste Klassifizierung von Pixeln (oder Voxeln) in einer Szene zu ermöglichen. Dabei werden jedem Pixel oder Voxelknoten semantische Labels wie „Hintergrund“, „Objekt“ oder spezifische Klassen (z. B. „Tumor“, „gesunde Struktur“) zugeordnet.

### Wichtige Begriffe und Konzepte

- **Modalität**: Jede Datenquelle, wie ein RGB-Bild, eine Tiefenkarte oder LiDAR-Daten, wird als eigene Modalität bezeichnet. In multimodalen Systemen ist das Ziel, diese unterschiedlichen Informationsströme optimal zu kombinieren, um ein umfassenderes Verständnis der Szene zu erhalten.
- **Unimodaler Bias**: Ein häufig beobachtetes Phänomen ist, dass Modelle dazu neigen, dominante Modalitäten – beispielsweise leicht interpretierbare RGB-Bilder – zu bevorzugen. Dies führt dazu, dass andere Modalitäten, die zwar wichtige, aber schwerer zu extrahierende Informationen liefern, vernachlässigt werden. Fehlt die dominante Modalität, verschlechtert sich oft die Gesamtleistung des Systems erheblich.
- **Distillation**: In unserem Ansatz spielen verschiedene Distillationsverfahren eine zentrale Rolle. Hierbei wird Wissen von einem „Lehrer“-Modell, das auf vollständigen, multimodalen Daten trainiert wurde, an ein „Schüler“-Modell weitergegeben, das auch bei unvollständigen Eingaben robuste Vorhersagen treffen soll.
  - **Modality-Agnostic Distillation** (\(\mathcal{L}_{mad}\)): Stellt sicher, dass das Schüler-Modell auch bei variierenden und fehlenden Modalitäten semantisch konsistente Vorhersagen trifft.
  - **Unimodal Distillation** (\(\mathcal{L}_{umd}\)): Unterstützt die einzelnen Modalitäten, indem es das Schüler-Modell anleitet, die spezifischen Repräsentationen jeder Modalität zu erlernen.
  - **Cross-Modal Distillation** (\(\mathcal{L}_{cmd}\)): Gleicht die Beziehungen zwischen verschiedenen Modalitäten an, um den unimodalen Bias zu reduzieren.

### Literaturüberblick

Zahlreiche Studien haben sich mit der Fusion multimodaler Daten beschäftigt. Klassische Ansätze setzen auf die Kombination von separaten Encodern für jede Modalität, gefolgt von einem gemeinsamen Decoder, um die unterschiedlichen Informationen zusammenzuführen [vgl. Zhang et al., 2023; Zheng et al., 2024]. Obwohl diese Methoden bei vollständigen Datensätzen gute Ergebnisse erzielen, zeigen sie oft gravierende Leistungseinbußen, wenn einzelne Modalitäten fehlen.

Aktuelle Ansätze wie MAGIC und Any2Seg haben versucht, den unimodalen Bias zu adressieren, indem sie speziell darauf ausgelegte Lernstrategien verwenden, die den Einfluss dominanter Modalitäten reduzieren. Trotz dieser Fortschritte bleibt die Herausforderung bestehen, ein robustes System zu entwickeln, das flexibel auf unterschiedliche, teilweise unvollständige Modalitäten reagiert. Hier setzt unser PRISMS-Framework an, das durch den Einsatz von parallelem multimodalem Lernen (PML) und einer Kombination aus unimodaler, cross-modaler sowie modality-agnostischer Distillation eine deutliche Verbesserung erzielt.

### Motivation und Zielsetzung

Die praktische Relevanz der multimodalen Segmentierung liegt vor allem in der Realwelt, in der nicht immer alle gewünschten Datenquellen verfügbar sind. Sensorfehler, unterschiedliche Erfassungsraten oder Umgebungsbedingungen führen häufig zu unvollständigen Datensätzen. Ein robustes Segmentierungsmodell muss daher in der Lage sein, auch unter solchen Bedingungen zuverlässige Ergebnisse zu liefern. Ziel unserer Arbeit ist es, ein Framework zu entwickeln, das:
- Beliebige Kombinationen von Modalitäten verarbeiten kann,
- Bei fehlenden Modalitäten keinen signifikanten Leistungseinbruch zeigt,
- Sowohl auf Einmodal- als auch auf Multimodal-Daten konsistente, semantisch fundierte Vorhersagen trifft.

Mit PRISMS wird ein Ansatz vorgestellt, der diese Ziele durch einen mehrstufigen Distillationsprozess erreicht und damit den Herausforderungen in der realen Anwendung gerecht wird.

---

Diese Arbeit liefert somit einen umfassenden Beitrag zur Weiterentwicklung der multimodalen Segmentierung, indem sie nicht nur bestehende Ansätze kritisch beleuchtet, sondern auch durch detaillierte Ablationsstudien zeigt, wie ein robustes und flexibles System konstruiert werden kann.