Hier ist dein Artikel in der Ich-Form:

---

title: "Architektur und Aufbau des PRISMS-Frameworks"  
description: "Detaillierte Erläuterung der Gesamtstruktur (PML, Unimodal & Cross-Modal Distillation) anhand zentraler Abbildungen"  
math: true  

---

## 1. Gesamtstruktur: Lehrer–Schüler-Aufbau und Distillation

![PRISMS Framework](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/Overall_light.png "Abbildung 1 zeigt das (a) Overall Framework in einer zweistufigen Struktur, die aus einem multimodalen Segmentor (oben) und einem anymodalen Segmentor (unten) besteht.")

### 1.1 Multimodaler Lehrer

- **Multimodale Eingabe**: Im oberen Teil verarbeite ich alle verfügbaren Modalitäten – beispielsweise RGB, Tiefe, Event und LiDAR – in einem einzigen Batch.  
- **Mehrskalige multimodale Eigenschaften**: Mein Encoder (z. B. ein Transformer-Backbone) extrahiert mehrskalige Repräsentationen aus jeder Modalität. Diese Repräsentationen führe ich anschließend, je nach gewähltem Fusionsprinzip, zu einem „starken“ Feature-Vektor zusammen.  
- **Segmentation Head**: Auf Basis dieser fusionierten Repräsentationen erzeuge ich eine Segmentierungskarte, die als **Lehrer-Vorhersage** fungiert. Parallel dazu setze ich einen klassischen Cross-Entropy-Loss (\(\mathcal{L}_{pre}\)) zur Supervision ein.

Dieser „Teacher“ dient mir als Referenzmodell, das sämtliche Modalitäten kennt und daher die bestmögliche Vorhersagequalität liefern soll. Die Idee dahinter ist, dass ich aus diesem reichhaltigen Modell Wissen (sowohl Feature- als auch Vorhersageinformationen) extrahiere und an ein „Student“-Modell weitergebe.

### 1.2 Anymodaler Schüler

- **Teilweise oder unvollständige Modalitäten**: Im unteren Teil der Abbildung zeige ich, dass die Eingabe **randomisiert** unvollständig sein kann – mal liegt nur eine einzelne Modalität vor (z. B. nur RGB), mal zwei (RGB + Tiefe) oder mal alle vier.  
- **Mehrskalige beliebige modale Eigenschaften**: Ein ähnlicher Encoder-Block extrahiert die Features nur aus den tatsächlich vorhandenen Modalitäten. Mein Ziel ist es, trotz der reduzierten Eingabequalität robuste Segmentierungsergebnisse zu erzielen.  
- **Segmentation Head**: Aus den gewonnenen Teil-Features berechne ich die endgültigen Vorhersagen. Verschiedene Distillations-Mechanismen stellen sicher, dass der Schüler die „Denkmuster“ des Lehrers übernimmt – auch wenn er weniger oder andere Daten erhält.

### 1.3 Unimodale und Cross-Modal Distillation

Rechts in Abbildung 1 illustriere ich zwei wichtige Prozesse, die auf den Feature-Ebenen stattfinden:

1. **Unimodale Distillation**:  
   - Hier vergleiche ich für jede Modalität die Features des Schülers (z. B. \(g_r^{i}\) für RGB) direkt mit den entsprechenden Lehrer-Features (\(f_r^{i}\)).  
   - Ziel ist es, dass der Schüler in derselben Modalität ähnliche Repräsentationen lernt wie der Lehrer, der alle Modalitäten gesehen hat.  
   - Formal setze ich dazu häufig Kullback-Leibler-Divergenz oder ähnliche Verlustfunktionen ein, wie in \(\mathcal{L}_{umd}\) beschrieben.

2. **Cross-Modal Correspondence Distillation**:  
   - Zusätzlich vergleiche ich die Beziehungen **zwischen** zwei Modalitäten im Schüler mit jenen im Lehrer. Beispielsweise messe ich die Kosinusähnlichkeit zwischen den RGB- und Tiefen-Features im Schüler (\(\mathcal{S}(g_r^i, g_d^i)\)) und setze sie mit denen des Lehrers (\(\mathcal{S}(f_r^i, f_d^i)\)) in Beziehung.  
   - Damit verhindere ich, dass der Schüler zu stark auf eine einzelne Modalität fixiert bleibt („unimodaler Bias“), und lernt, wie sich verschiedene Modalitäten gegenseitig ergänzen.  
   - Mathematisch realisiere ich dies über \(\mathcal{L}_{cmd}\), in der ich die Kosinusähnlichkeiten miteinander vergleiche.

### 1.4 Modality-Agnostic Distillation

Abgerundet wird das Konzept durch die **Vorhersage-Distillation** (unten rechts in Abbildung 1). Dort vergleiche ich die finale Segmentierungskarte des Schülers (\(P_{am}\)) mit der des Lehrers (\(P_{mm}\)). Der entsprechende Verlust \(\mathcal{L}_{mad}\) misst die Diskrepanz zwischen beiden Wahrscheinlichkeitsverteilungen:

\[
\mathcal{L}_{mad} = \frac{1}{N} \sum_{i=1}^{N} \sum_{k=1}^{K} P_{am}^{i,k} \log\left(\frac{P_{am}^{i,k}}{P_{mm}^{i,k}}\right).
\]

So „sieht“ der Schüler, welche semantischen Vorhersagen der Lehrer in komplexen, multimodalen Situationen trifft, und passt seine eigene Verteilung entsprechend an – selbst wenn ihm nicht alle Modalitäten zur Verfügung stehen.

---

## 2. PML (Parallel Multimodal Learning) – Ein starker Lehrer

![PML](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/PML%2Bfeature%20distil.png)

Die zweite Abbildung zeigt den **PML-Prozess**, der den zentralen Baustein zum Aufbau meines Lehrer-Netzwerks bildet. Hier werden alle Modalitäten **parallel** in einem einzigen Mini-Batch verarbeitet.

### 2.1 Gemeinsame Verarbeitung pro Transformer-Block

- **Eingabe**: Das System erhält pro Batch für jede Szene sämtliche Modalitäten.  
- **Parallele Feature-Extraktion**: In jedem Transformer-Block berechne ich zunächst die Feature Maps aller Modalitäten unabhängig voneinander. Anschließend führe ich sie über eine einfache Mittelung (Min- bzw. Mean-Operation) zu einem gemeinsamen, multimodalen Feature zusammen.  
- **Skalenniveaus**: Typischerweise durchlaufe ich vier Stufen (Level 1 bis 4), in denen ich die Eingangsmodalitäten schrittweise zusammenführe.

### 2.2 Ziel: Maximaler Informationsgehalt

Durch das PML erhalte ich einen **starken „Multimodal Encoder“**, der in der Lage ist, alle Signalquellen zu vereinen. Das finale Feature, das an den Decoder (Segmentation Head) weitergegeben wird, vereint also Wissen aus RGB, Tiefe, Event, LiDAR etc. Das **Segmentierungsergebnis** des Lehrer-Modells wird dann über einen klassischen Kreuzentropie-Verlust (\(\mathcal{L}_{pre}\)) trainiert:

\[
\mathcal{L}_{pre} = -\sum_{i=1}^{N} \sum_{k=1}^{K} y_{i,k} \log(p_{i,k}).
\]

Dieses Vorgehen stellt sicher, dass mein Lehrer das bestmögliche Verständnis der Szene erlangt – ideal, um danach als Distillationsquelle für den anymodalen Schüler zu dienen.

---

## 3. Zusammenspiel der Komponenten

1. **Lehrertraining mit PML**:  
   Zuerst trainiere ich den **Lehrer** mit allen Modalitäten im Parallel-Modus. Daraus entsteht ein robustes, umfassend informiertes Modell.

2. **Anymodales Schülertraining**:  
   Danach trainiere ich den **Schüler** (Anymodal Segmentor) mithilfe von:
   - **Unimodaler Distillation** (\(\mathcal{L}_{umd}\)) für jede Modalität,  
   - **Cross-Modal Distillation** (\(\mathcal{L}_{cmd}\)) zur Angleichung der Beziehungen zwischen Modalitäten,  
   - **Modality-Agnostic Distillation** (\(\mathcal{L}_{mad}\)) auf der Vorhersageebene.

3. **Ziel: Robustheit bei fehlenden Modalitäten**:  
   Während des Schülertrainings werden die Modalitäten zufällig maskiert oder weggelassen, sodass der Schüler lernt, auch mit unvollständigen Eingaben umzugehen. Dank des zuvor trainierten Lehrers kann er sich dennoch an den „vollen“ Kontext annähern.
