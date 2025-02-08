---
title: "PRISM – Modellübersicht"
description: "Wie das mehrstufiges Modell aufgebaut ist und wie die einzelnen Teile ineinandergreifen."
date: 2025-02-05
type: post
draft: false
translationKey: prism_modell
coffee: 1
tags: ['Modell','multimodal','PRISM']
categories: ['Modul']
---

<span class="letterine"><i>D</i>as **PRISM-Modell** ist so entworfen</span>, dass es in Szenarien funktioniert, in denen *nicht alle* Modalitäten – beispielsweise verschiedene MRT-Sequenzen – verfügbar sind. Die grundlegende Idee:  
- Wir haben **mehrere Enkodierzweige** (Encoder) – jeder Zweig kümmert sich um eine Modalität.  
- Alle Zweige sind mit **einem gemeinsamen Decoder** verbunden.  
- Auf diese Weise fließen Informationen aller (aktuell vorhandenen) Modalitäten in einer einzigen Fusionsschicht zusammen.

Im Folgenden siehst du zwei Abbildungen, die den Kern des Modells skizzieren und erklären, wie wir die Konzepte „Lehrer“ und „Schüler“ innerhalb ein und desselben Netzwerks umsetzen.  

---

## Abbildung 1: *Relative Präferenz* – Überblick

![Relative Präferenz (Figur 1)](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/8d1a65675a64e6fc6c43bad46dfbbf8bb38a8e75/Relative%20Pref(1).svg)

**Was du siehst**  
1. **Kreis mit Pfeilen**:  
   - Jeder farbige Pfeil repräsentiert eine bestimmte Modalität.  
   - Der Abstand zur Mitte (bzw. ein Winkel) zeigt, ob und wie stark eine Modalität vom Modell bevorzugt oder vernachlässigt wird.  
   - „Beschleunigen“ heißt, dass das Modell diese Modalität stärker berücksichtigt; „Verlangsamen“ bedeutet, dass man sie im Training etwas drosselt, damit sich keine Modalität zu sehr in den Vordergrund spielt.

2. **Logit-Diagramm** (unten):  
   - Die verschiedenfarbigen Kurven sind *Ausgabe-Verteilungen* (Logits) der jeweiligen Modalitäten.  
   - Die horizontalen Linien/Markierungen (dashed lines) symbolisieren, wie die Lernrate in Abhängigkeit der *Modalität* variiert.  
   - Weiter rechts heißt: „Stärkere Vorhersage“ oder z. T. „dominierende“ Modalität. Man kann aber auch herleiten, wo unsere *Regulierung* (Sprich: Anpassung der Lernrate) ansetzt.

**Wieso ist das wichtig?**  
- Jede Modalität kann ihr eigenes „Gewicht“ im Netzwerk haben.  
- Ist z. B. T2-MRT sehr informativ, könnte es alle anderen Modalitäten überstrahlen.  
- Die **relative Präferenz** sorgt für *Ausgleich*. Wir wollen sicherstellen, dass keine Modalität komplett untergeht oder das Training dominiert.  

---

## Abbildung 2: *Geteiltes Modell* – Architekturdiagramm

![Geteiltes Modell (Figur 2)](https://raw.githubusercontent.com/DavidRutkevich/PRISM-Docs/986647d02bce43bfd247aa9760de520054dcf7de/Model_vis_web.svg)

**Aufbau**  
1. **Encoder (gelb, rosa, lila, …)**  
   - Jeder Encoder bekommt *eine* MRT-Sequenz als Eingabe.  
   - Die Encoder-Blöcke extrahieren Merkmale (Features) aus der jeweiligen Modalität.  

2. **Gemeinsamer Decoder (grau/weiß)**  
   - Die Ausgaben aller Encoder werden zusammengeführt (Fusion).  
   - Dieser geteilte Decoder verarbeitet die kombinierten Merkmale und erzeugt eine *multimodale* Vorhersage (hier als „Lehrerpfad“ bezeichnet).  

3. **Einzelausgänge („Schüler“) pro Modalität**  
   - Neben dem großen Decoder gibt es für jede Modalität noch einen kleinen Decoder-Zweig.  
   - Damit erhält jeder Encoder auch einen *eigenen* Output.  

**Warum diese Aufteilung?**  
- *Multimodaler Pfad* (großer Decoder): Nutzt alle verfügbaren Modalitäten gleichzeitig und kann somit „am meisten“ sehen.  
- *Unimodale Pfade* (kleine Decoderausgänge):  
  - Trainieren gezielt auf einer einzigen Modalität.  
  - Sie bekommen während des Trainings Feedback vom großen Decoder.  

> **Wichtig:** Fehlt eine Modalität komplett (z. B. T1ce ist nicht vorhanden), wird der entsprechende Encoder bei der Fusionsstufe einfach *nicht* genutzt. Das Modell kann also jederzeit auch mit nur zwei oder drei Modalitäten weiterarbeiten, ohne dass wir Code ändern müssen.

**Was ist „SIM“ und „L2-Distanz“ in der Grafik?**  
- **SIM** (Similarity) deutet an, dass wir die Feature-Darstellungen von *einzelnen* Modalitäten mit dem *multimodalen* Pfad vergleichen.  
- **L2-Distanz** bedeutet einfach, dass wir zwischen zwei Features (z. B. logitbasiert oder embeddingsbasiert) den L2-Abstand ausrechnen. Dieser Wert sagt aus, wie nah oder fern ein unimodaler Decoder von der „multimodalen Lehrervorhersage“ liegt.

**Fazit der Grafik**  
- Der komplette orangefarbene Kasten (rechts) bezeichnet den *Lehrer*: Alles was aus dem gemeinsamen Decoder (grau) kommt, gilt als „multimodal“ und dient den Einzeldisziplinen (den Encodern samt ihrer Ausgänge) als *Referenz*.  
- Jeder einzelne Encoder-Ausgang (gelb, rosa, lila) ist quasi *Schüler* und wird motiviert, sich an der multimodalen Sichtweise zu orientieren.  

---

## Fazit zum Modell

- **Vielseitigkeit**: Durch das Baukasten-Prinzip (mehrere Encoder + gemeinsamer Decoder) kann das Modell auch bei fehlenden Modalitäten gute Ergebnisse liefern.  
- **Ausbalancierte Nutzung**: Die *relative Präferenz* (Abbildung 1) stellt sicher, dass keine Modalität im Training total über- oder unterbewertet wird.  
- **Lehrer–Schüler-Prinzip intern**: Statt ein externes Lehrermodell zu bauen, übernimmt der *Fusionsteil* diese Aufgabe gleich mit. So spart man Rechenzeit und bekommt ein kompakteres Design.

In weiteren Abschnitten wird genauer beschrieben, **wie** die Methode zur Regelung und Selbstdistillation formal funktioniert und welche Formeln im Detail verwendet werden (siehe „Methodik & Loss-Funktionen“). Für den Einstieg hilft aber schon ein Blick auf diese beiden Abbildungen, um das *große Ganze* zu verstehen:
- *Viele Eingänge* (Modalitäten) → *ein gemeinsamer Ausgang* + *unimodale Ausgänge*.  
- Interner Lehrer (Fusionspfad) → interne Schüler (Einzelmodalitäten).  
- *Relative Präferenz* → verhindert Ungleichgewicht im Training.

So lässt sich das PRISM-Modell strukturiert aufbauen und an verschiedene Datensituationen anpassen.
