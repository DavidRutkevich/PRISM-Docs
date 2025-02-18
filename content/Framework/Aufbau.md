---
title: "Architektur und Aufbau des PRISMS-Frameworks"
description: "Detaillierte Erläuterung der Gesamtstruktur (PML, Unimodal & Cross-Modal Distillation) anhand zentraler Abbildungen"
math: true
---

## 1. Gesamtstruktur: Lehrer–Schüler-Aufbau und Distillation
![PRISMS Framework](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/Overall_light.png "Abbildung 1 zeigt das (a) Overall Framework in einer zweistufigen Struktur, die aus einem multimodalen Segmentor (oben) und einem anymodalen Segmentor (unten) besteht.") 

### 1.1 Multimodaler Lehrer

- **Multimodale Eingabe**: Im oberen Teil werden alle verfügbaren Modalitäten – beispielsweise RGB, Tiefe, Event und LiDAR – in einem einzigen Batch verarbeitet.  
- **Mehrskalige multimodale Eigenschaften**: Der Encoder (hier z. B. ein Transformer-Backbone) extrahiert mehrskalige Repräsentationen aus jeder Modalität. Diese werden anschließend, je nach gewähltem Fusionsprinzip, zu einem „starken“ Feature-Vektor zusammengefasst.  
- **Segmentation Head**: Auf Basis dieser fusionierten Repräsentationen entsteht eine Segmentierungskarte, die als **Lehrer-Vorhersage** fungiert. Parallel dazu wird ein klassischer Cross-Entropy-Loss (\(\mathcal{L}_{pre}\)) zur Supervision genutzt.

Diese „Teacher“-Komponente dient uns als Referenzmodell, das sämtliche Modalitäten kennt und daher die bestmögliche Vorhersagequalität liefern soll. Die Idee dahinter ist, dass wir aus diesem reichen Modell Wissen (Feature- und Vorhersageinformationen) extrahieren und an ein „Student“-Modell weitergeben können.

### 1.2 Anymodaler Schüler

- **Teilweise oder unvollständige Modalitäten**: Im unteren Teil der Abbildung sieht man, dass die Eingabe **randomisiert** unvollständig sein kann – mal liegt nur eine einzelne Modalität vor (z. B. nur RGB), mal zwei (RGB + Tiefe), mal alle vier.  
- **Mehrskalige beliebige modale Eigenschaften**: Ein ähnlicher Encoder-Block extrahiert die Features nur aus den tatsächlich vorhandenen Modalitäten. Das Ziel: Trotz der reduzierten Eingabequalität weiterhin robuste Segmentierungsergebnisse zu erzielen.  
- **Segmentation Head**: Aus den gewonnenen Teil-Features werden die endgültigen Vorhersagen berechnet. Dabei helfen verschiedene Distillations-Mechanismen, die sicherstellen, dass das Schülermodell die „Denkmuster“ des Lehrers übernimmt – auch wenn es weniger oder andere Daten bekommt.

### 1.3 Unimodale (b) und Cross-Modal (c) Distillation

Rechts in Abbildung 1 sind zwei wichtige Prozesse abgebildet, die jeweils auf den Feature-Ebenen stattfinden:

1. **Unimodale Distillation**:  
   - Hier werden für jede Modalität die Features des Schülers (z. B. \(g_r^{i}\) für RGB) direkt mit den entsprechenden Lehrer-Features (\(f_r^{i}\)) verglichen.  
   - Ziel ist es, dass der Schüler in derselben Modalität ähnliche Repräsentationen lernt wie der Lehrer, der mehr Daten gesehen hat.  
   - Formal erfolgt dies meist über Kullback-Leibler-Divergenz oder ähnliche Verluste, wie wir im vorherigen Artikel durch \(\mathcal{L}_{umd}\) dargestellt haben.

2. **Cross-Modal Correspondence Distillation**:  
   - Zusätzlich vergleicht man die Beziehungen **zwischen** zwei Modalitäten im Schüler mit jenen im Lehrer. Beispielsweise die Kosinusähnlichkeit zwischen den RGB- und Tiefen-Features im Schüler (\(\mathcal{S}(g_r^i, g_d^i)\)) gegenüber dem Lehrer (\(\mathcal{S}(f_r^i, f_d^i)\)).  
   - Damit wird verhindert, dass der Schüler zu stark auf eine einzelne Modalität fixiert bleibt („unimodaler Bias“). Stattdessen lernt er, wie verschiedene Modalitäten einander ergänzen können.  
   - Mathematisch erfolgt dies über \(\mathcal{L}_{cmd}\), wo die Kosinusähnlichkeiten miteinander verglichen werden.

### 1.4 Modality-Agnostic Distillation

Abgerundet wird das Konzept durch die **Vorhersage-Distillation** (rechts in Abbildung 1, im unteren Teil). Dort wird die finale Segmentierungskarte des Schülers (\(P_{am}\)) mit der des Lehrers (\(P_{mm}\)) verglichen. Der entsprechende Verlust \(\mathcal{L}_{mad}\) misst die Diskrepanz zwischen beiden Verteilungen:

\[
\mathcal{L}_{mad} = \frac{1}{N} \sum_{i=1}^{N} \sum_{k=1}^{K} P_{am}^{i,k} \log\left(\frac{P_{am}^{i,k}}{P_{mm}^{i,k}}\right).
\]

Dadurch „sieht“ der Schüler, welche semantischen Vorhersagen der Lehrer in komplexen, multimodalen Situationen trifft und passt seine eigene Verteilung entsprechend an. Auch wenn der Schüler nicht alle Modalitäten zur Verfügung hat, kann er so semantisch konsistent bleiben.

---

## 2. PML (Parallel Multimodal Learning) – Ein starker Lehrer
![PML](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/PML%2Bfeature%20distil.png)
Die zweite Abbildung zeigt den **PML-Prozess**, der den zentralen Baustein zum Aufbau des Lehrer-Netzwerks bildet. Hier werden alle Modalitäten **parallel** in einem einzigen Mini-Batch verarbeitet.

### 2.1 Gemeinsame Verarbeitung pro Transformer-Block

- **Eingabe**: Das System erhält pro Batch für jede Szene sämtliche Modalitäten.  
- **Parallele Feature-Extraktion**: In jedem Transformer-Block werden die Feature Maps aller Modalitäten zunächst unabhängig voneinander berechnet. Anschließend erfolgt eine einfache **Mittelung** (Min- bzw. Mean-Operation), sodass ein gemeinsames, multimodales Feature entsteht.  
- **Skalenniveaus**: Typischerweise durchläuft man vier Stufen (Level 1 bis 4). Auf jedem Level werden die Eingangsmodalitäten zusammengeführt.  

### 2.2 Ziel: Maximaler Informationsgehalt

Durch das PML entsteht ein **starker „Multimodal Encoder“**, der in der Lage ist, alle Signalquellen zu vereinen. Das finale Feature, das an den Decoder (Segmentation Head) weitergegeben wird, vereint also Wissen aus RGB, Tiefe, Event, LiDAR etc. Das **Segmentierungsergebnis** dieses Lehrer-Modells wird über die bereits erwähnte Kreuzentropie-Verlustfunktion \(\mathcal{L}_{pre}\) trainiert:

\[
\mathcal{L}_{pre} = -\sum_{i=1}^{N} \sum_{k=1}^{K} y_{i,k} \log(p_{i,k}).
\]

Dieses Vorgehen stellt sicher, dass unser Lehrer das bestmögliche Verständnis über die Szene erlangt – ideal, um danach als Distillationsquelle für den anymodalen Schüler zu dienen.

---

## 3. Zusammenspiel der Komponenten

1. **Lehrertraining mit PML**:  
   Zuerst trainieren wir den **Lehrer** mit allen Modalitäten im Parallel-Modus. Daraus entsteht ein robustes, umfassend informiertes Modell.

2. **Anymodales Schülertraining**:  
   Danach trainieren wir den **Schüler** (Anymodal Segmentor) mithilfe von:
   - **Unimodaler Distillation** (\(\mathcal{L}_{umd}\)) für jede Modalität,  
   - **Cross-Modal Distillation** (\(\mathcal{L}_{cmd}\)) für die Beziehungen zwischen Modalitäten,  
   - **Modality-Agnostic Distillation** (\(\mathcal{L}_{mad}\)) auf der Vorhersageebene.  

3. **Ziel: Robustheit bei fehlenden Modalitäten**:  
   Während des Schülertrainings werden die Modalitäten zufällig maskiert oder weggelassen, sodass der Schüler lernt, auch mit unvollständigen Eingaben umzugehen. Dank der vorher gelernten Lehrer-Struktur kann er sich trotzdem an den „vollen“ Kontext annähern.

---

## 4. Fazit

Die beiden Abbildungen zeigen anschaulich, wie das PRISMS-Framework systematisch vom **vollständigen, parallelen Multimodal-Lernen** (PML) hin zu einem **robusten, anymodalen Schüler** gelangt. Durch **Unimodale**, **Cross-Modal** und **Modality-Agnostic Distillation** übertragen wir das in der Lehrer-Architektur angereicherte Wissen auf ein Modell, das auch bei fehlenden Sensoren oder unregelmäßigen Datenquellen eine präzise Segmentierung liefert.

Diese Strategie – eine Kombination aus umfassendem Lehrer und gezielter Distillation – ist der Schlüssel, um in realen Anwendungsszenarien (etwa medizinische Bildgebung, autonomes Fahren, Industrie-4.0-Inspektion) die nötige Zuverlässigkeit und Flexibilität zu erreichen.