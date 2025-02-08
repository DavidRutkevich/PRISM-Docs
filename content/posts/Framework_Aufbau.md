---
title: "PRISMS – Innovation und Methodik"
description: "Detaillierte Erklärung zur mathematischen Herangehensweise und Architektur hinter PRISMS (ehemals AnySeg)."
date: 2025-02-06
type: post
draft: false
translationKey: "prisms_methodik_storytelling"
coffee: 1
tags: ['PRISMS','Innovation','Methodik']
categories: ['Framework']
---


## Einleitung
In vielen realen Anwendungsszenarien (Medizin, autonomes Fahren, Industrie 4.0) stehen mehrere **Modalitäten** (RGB-Bilder, Tiefendaten, LiDAR, Events etc.) zur Verfügung, die aber selten lückenlos vorliegen. Genau hier setzt **PRISMS** (zuvor “AnySeg”) an: Es handelt sich um ein Verfahren für *Anymodal-Semantic-Segmentation*, das robuste Segmentierungen ermöglicht – **unabhängig davon**, welche Teilmenge an Modalitäten in einem konkreten Fall verfügbar ist.

Dieses Dokument erläutert die **Innovationen** von PRISMS im Detail. Zur besseren Veranschaulichung können Abbildungen eingefügt werden, die das **Lehrer–Student-Prinzip**, die **multiskalige Verarbeitung** sowie die **Distillationsmechanismen** illustrieren. Bitte füge die entsprechenden Abbildungen an den gekennzeichneten Stellen ein.

---

## 1. Motivation: Fehlende Modalitäten und Unimodaler Bias
Obwohl mehrere Sensoren gleichzeitig genutzt werden **könnten**, fehlen in der Praxis oft Daten (z. B. eventbasiertes Kamerasignal defekt, Tiefe nicht aufgezeichnet). Zudem ist bei klassischer Multimodalfusion häufig ein *Bias* zu beobachten: Ein Modell „versteift“ sich auf eine dominante Modalität, was bei deren Ausfall zu drastischem Performance-Verlust führt.

**PRISMS** adressiert diese Probleme mit:
- **Multiskaliger Repräsentation** in allen Modalitäten,  
- **Selbstdistillation** (Lehrer–Schüler-Prinzip),  
- **Cross-Modaler** Abstimmung, damit jede Modalität ihre Stärken einbringen kann.

---

## 2. Architekturüberblick

### 2.1 Mehrskalige Transformer-Blöcke

![Abbildung Multiskalige Fusion](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/Model_detail.png)

- **Idee**: Jede verfügbare Modalität wird in einem Transformer-basierten Netzwerk verarbeitet.  
- **Mittelwertoperation**: Nach jedem Block werden die Feature-Karten über alle aktiven Modalitäten gemittelt. Mathematisch:

  ```math
  z_i^l = \frac{1}{M_l} \sum_{m=1}^{M_l} h_{i,m}^l,
  ```
  
  wobei $h_{i,m}^l$ das Feature der Modalität $m$ (von insgesamt $M_l$ verfügbaren Modalitäten) im $l$-ten Block beschreibt. Das resultierende $z_i^l$ spiegelt ein **multiskaliges** Merkmal wider, das – wenn verfügbar – Informationen aus allen Modalitäten enthält.

- **Segmentierung**: Diese gemittelten Features werden anschließend an einen Segmentation-Head weitergegeben, der die semantische Klassifikation vornimmt (z. B. via Pixelwise Prediction).

### 2.2 Lehrer–Studenten-Netzwerke

![Abbildung Lehrer–Schüler-Prinzip](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/refs/heads/figures/Model_general.png)

1. **Lehrer**  
   - Erhält im Idealfall alle Modalitäten (oder so viele wie möglich) und lernt daher eine **reiche** Repräsentation.
   - Dient als starker Referenzpunkt, von dem das Studentenmodell lernen kann.

2. **Student (Anymodal)**  
   - Verarbeitet nur diejenigen Modalitäten, die in einem Mini-Batch tatsächlich vorhanden sind.
   - Muss daher *robust* gegen fehlende Daten sein.  
   - Lässt sich so trainieren, dass selbst bei nur einer verfügbaren Modalität noch sinnvolle Segmentierungsergebnisse entstehen.

---

## 3. Mathematische Kernelemente

### 3.1 Modalitätsagnostische Distillation (MAD)

Um den Studenten-Pfad robust zu machen, wird eine **Modalitäts-Agnostische** Distillation angewendet. Das heißt, der Lehrer gibt eine „Zielfunktion“ vor, die unabhängig von der jeweiligen Modalität ist. Auf **Pixel- bzw. Logit-Ebene** kann man dies formulieren als:

```math
L_{\text{MAD}} 
= 
\sum_{i=1}^{N} \sum_{k=1}^{K}
\mathrm{KL}\Bigl(
  P_{\text{stud}}^{i,k} \,\Big\|\,
  P_{\text{teach}}^{i,k}
\Bigr),
```

- $P_{\text{stud}}^{i,k}$ und $P_{\text{teach}}^{i,k}$ sind die Wahrscheinlichkeitsvorhersagen (Logits nach Softmax) für Klasse $k$ an Pixel $i$.  
- Durch die **Kullback-Leibler-Divergenz** wird der Studierende dazu gebracht, die Lehrer-Verteilung nachzuahmen, ungeachtet der aktuell verfügbaren Modalitäten.

### 3.2 Weighted-Distribution (WD)
Fehlen in vielen Trainingsiterationen bestimmte Modalitäten, können deren Gewichte leicht „untergehen“. Daher führt PRISMS häufig eine Gewichtung $\omega_m$ ein:

```math
\omega_m = \frac{1}{1 + \text{FR}^m},
```

wobei $\text{FR}^m$ die Fehlrate (Missing Rate) von Modalität $m$ angibt. Eine seltene Modalität ($\text{FR}^m$ nahe 1) erhält so eine **höhere** Lernrate, um in ihren wenigen Vorkommen besonders gut genutzt zu werden.

### 3.3 Cross-Modale PRISMS-Distillation
Neben dem distanzbasierten Vergleich zwischen Lehrer und Student integriert PRISMS eine **cross-modale** Distillation innerhalb des Studentenmodells. Damit lernt jede Modalität, ähnliche Klassendiskriminierung zu liefern. Eine Beispiel-Formel könnte lauten:

```math
L_{\text{cross}} 
= 
\sum_{m \neq m'} \omega_m \,\omega_{m'}\,
\mathrm{dist}\Bigl(\phi(z^m), \phi(z^{m'})\Bigr),
```

- $z^m$ bezeichnet das Feature aus Modalität $m$.  
- $\mathrm{dist}$ misst die Ähnlichkeit/Differenz (z. B. via Cosine Similarity, KL oder L2).  
- $\phi$ ist eine Projektionsebene, z. B. ein MLP oder eine Prototyp-Layer.

Die cross-modale Distillation sorgt dafür, dass alle Modalitäten **konsistent** werden und sich gegenseitig ergänzen können, statt dass eine Modalität das System dominiert.

### 3.4 Gesamt-Loss
Typischerweise fasst man alle Verluste in einer Summe zusammen:

```math
L_{\text{total}} 
=
L_{\text{seg}}
+
\lambda_{\text{mad}}\,L_{\text{MAD}}
+
\lambda_{\text{cross}}\,L_{\text{cross}}
+
\lambda_{\text{reg}}\,L_{\text{reg}},
```

- $L_{\text{seg}}$: klassischer Segmentierungsverlust (z. B. Dice + CE).  
- $L_{\text{MAD}}$: modalitätsagnostische Distillation vom Lehrer.  
- $L_{\text{cross}}$: cross-modale Abstimmung zwischen den Modalitäten des Studenten.  
- $L_{\text{reg}}$: optionale Regulierungen (z. B. Weighted-Distribution).

---

## 4. Zusammenspiel der Komponenten

1. **Multiskalen-Fusion**: Jede Modalität wird parallel im gleichen Netzwerk pro Block verarbeitet. Durch das Mittelwerten der Feature-Karten entsteht eine gemeinsame Repräsentation, die **direkt** ins Segmentierungsmodul fließt.  
2. **Lehrer–Student**: Ein Lehrer-Netz (möglichst vollmodale Eingabe) liefert Zielverteilungen, während das Studenten-Netz (beliebig modale Eingabe) diese Vorhersagen anpasst.  
3. **Distillation**:  
   - *MAD*: Lernt Pixel- bzw. Logit-Ähnlichkeit zur Lehrer-Verteilung.  
   - *Cross-Modale Distillation*: Bringt Modalitäten untereinander in Einklang.  
   - *Weighted-Distribution*: Kompensiert für seltene Modalitäten.

---

## 5. Fazit

**PRISMS** schafft einen innovativen Ansatz, um beliebige (unvollständige) Kombinationen von Sensoren nutzen zu können. Durch eine **geschickte Verzahnung** aus:

- **Multiskaliger Netzwerkstruktur**,  
- **Selbstdistillation** (Lehrer–Student),  
- **Gewichteter Fusion** (Weighted-Distribution)  
- **Cross-Modaler** Feinjustierung

wird ein System entwickelt, das trotz fehlender Modalitäten **robust** Segmentierungen erzeugt. Anwendungen in der Medizin (z. B. MRT-Sequenzen), im autonomen Fahren (Kamera, LiDAR, Radar) oder in industriellen Prozessen (Röntgen, Infrarot, visuell) zeigen das breite Potenzial: **PRISMS** macht Modelle **tatsächlich** praxistauglich für Szenarien, in denen man nie sicher weiß, welche Daten genau ankommen.

**Schlussbemerkung zur Fusion von PRISM und PRISMS**  
Die in diesem Beitrag vorgestellten Ideen zu PRISM und PRISMS ergänzen sich in ihrer Zielsetzung, unvollständige sowie unbalancierte multimodale Daten effizient zu verarbeiten. Während PRISM vor allem durch die Kombination von Multi-Uni-Selbstdistillation und präferenzbasierter Regulierung überzeugt, überträgt PRISMS die Grundprinzipien eines anymodalen Lernens in einen noch universelleren Kontext. Die Verschmelzung beider Konzepte – also die **Fusion** von PRISM und PRISMS – führt zu einem ganzheitlichen Rahmenwerk, in dem sowohl **globale** (Prototyp-, Klassenrepräsentationen) als auch **lokale** (Pixel-basierte) Distillationsmechanismen nahtlos ineinandergreifen. Dadurch entsteht eine robuste End-to-End-Pipeline, die in unterschiedlichen Netzwerkarchitekturen (von klassischen CNN-basierten Encodern bis hin zu modernen Transformer-Strukturen) eingesetzt werden kann.

Diese Fusion erlaubt es, die präferenzbasierte Lernratensteuerung (aus PRISM) mit den anymodalen Strategien (aus PRISMS) zu vereinen, sodass auch seltene Modalitäten durchgängig berücksichtigt werden und das Gesamtsystem seine Robustheit gegenüber unvollständigen und fehlerhaften Datensätzen steigert. Die multiskalige Repräsentation und die darauf aufbauende Selbstdistillation – Eckpfeiler beider Ansätze – sind somit kein isoliertes Lernkonstrukt mehr, sondern werden als zentrale, universell einsetzbare Bausteine eines umfassenden Frameworks verstanden. Dadurch lassen sich verschiedenste Anwendungsdomänen (Medizin, autonome Systeme, Industrie 4.0) flexibel bedienen, ohne bei fehlenden Modalitäten oder stark schwankender Datenqualität an Grenzen zu stoßen. Insgesamt entsteht so ein leistungsfähiges, modular aufgebautes Konzept, das bei steigendem Bedarf an multimodalen Verfahren den Weg für eine robuste und skalierbare Any-Modal-Segmentierung ebnet.