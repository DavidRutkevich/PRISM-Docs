---

title: "Einleitung"  
description: "Einführung in die multimodale Segmentierung und die Herausforderungen bei unvollständigen Daten."  
math: true  

---

## Einleitung

Die multimodale Segmentierung hat in den letzten Jahren erheblich an Bedeutung gewonnen, da sie in vielen Anwendungsbereichen – von der medizinischen Bildgebung über autonomes Fahren bis hin zur industriellen Qualitätskontrolle – essenziell ist. Im Kern geht es darum, Informationen aus unterschiedlichen Sensoren oder Bildquellen (z. B. RGB, Tiefenkarten, LiDAR, MRT) zu kombinieren, um eine detaillierte und robuste Klassifizierung von Pixeln (oder Voxeln) in einer Szene zu ermöglichen. Dabei wird jedem Pixel bzw. Voxelknoten ein semantisches Label wie „Hintergrund“, „Objekt“ oder spezifische Klassen (z. B. „Tumor“, „gesunde Struktur“) zugeordnet.

### Wichtige Begriffe und Konzepte

- **Modalität**: Jede Datenquelle, wie ein RGB-Bild, eine Tiefenkarte oder LiDAR-Daten, wird als eigene Modalität bezeichnet. In multimodalen Systemen besteht das Ziel darin, diese unterschiedlichen Informationsströme optimal zu kombinieren, um ein umfassenderes Verständnis der Szene zu erhalten.
- **Unimodaler Bias**: Es wird häufig beobachtet, dass Modelle dominante Modalitäten – beispielsweise leicht interpretierbare RGB-Bilder – bevorzugen. Dies führt dazu, dass andere Modalitäten, die zwar wichtige, aber schwerer zu extrahierende Informationen liefern, vernachlässigt werden. Fehlt die dominante Modalität, kann sich die Gesamtleistung des Systems erheblich verschlechtern.
- **Distillation**: Verschiedene Distillationsverfahren spielen in diesem Ansatz eine zentrale Rolle. Dabei wird Wissen von einem „Lehrer“-Modell, das auf vollständigen, multimodalen Daten trainiert wurde, an ein „Schüler“-Modell weitergegeben, das auch bei unvollständigen Eingaben robuste Vorhersagen treffen soll.
  - **Modality-Agnostic Distillation** (\(\mathcal{L}_{mad}\)): Stellt sicher, dass das Schüler-Modell auch bei variierenden und fehlenden Modalitäten semantisch konsistente Vorhersagen trifft.
  - **Unimodal Distillation** (\(\mathcal{L}_{umd}\)): Unterstützt die einzelnen Modalitäten, indem das Schüler-Modell angeleitet wird, die spezifischen Repräsentationen jeder Modalität zu erlernen.
  - **Cross-Modal Distillation** (\(\mathcal{L}_{cmd}\)): Gleicht die Beziehungen zwischen verschiedenen Modalitäten an, um den unimodalen Bias zu reduzieren.

### Literaturüberblick

Zahlreiche Studien haben sich mit der Fusion multimodaler Daten beschäftigt. Klassische Ansätze setzen auf die Kombination von separaten Encodern für jede Modalität, gefolgt von einem gemeinsamen Decoder, um die unterschiedlichen Informationen zusammenzuführen [vgl. Zhang et al., 2023; Zheng et al., 2024]. Obwohl diese Methoden bei vollständigen Datensätzen gute Ergebnisse erzielen, zeigen sie oft gravierende Leistungseinbußen, wenn einzelne Modalitäten fehlen.

Aktuelle Ansätze wie MAGIC und Any2Seg haben versucht, den unimodalen Bias zu adressieren, indem speziell darauf ausgelegte Lernstrategien verwendet werden, die den Einfluss dominanter Modalitäten reduzieren. Trotz dieser Fortschritte bleibt die Herausforderung bestehen, ein robustes System zu entwickeln, das flexibel auf unterschiedliche, teilweise unvollständige Modalitäten reagiert. Hier setzt das PRISMS-Framework an, das durch den Einsatz von parallelem multimodalem Lernen (PML) und einer Kombination aus unimodaler, cross-modaler sowie modality-agnostischer Distillation deutliche Verbesserungen erzielt.

### Motivation und Zielsetzung

Die praktische Relevanz der multimodalen Segmentierung liegt vor allem in der Realität, in der nicht immer alle gewünschten Datenquellen verfügbar sind. Sensorfehler, unterschiedliche Erfassungsraten oder Umgebungsbedingungen führen häufig zu unvollständigen Datensätzen. Ein robustes Segmentierungsmodell muss daher in der Lage sein, auch unter solchen Bedingungen zuverlässige Ergebnisse zu liefern. Ziel dieser Arbeit ist es, ein Framework zu entwickeln, das:
- Beliebige Kombinationen von Modalitäten verarbeiten kann,
- Bei fehlenden Modalitäten keinen signifikanten Leistungseinbruch zeigt,
- Sowohl auf einmodalen als auch auf multimodalen Daten konsistente, semantisch fundierte Vorhersagen trifft.

Mit PRISMS wird ein Ansatz vorgestellt, der diese Ziele durch einen mehrstufigen Distillationsprozess erreicht und damit den Herausforderungen in der realen Anwendung gerecht wird.