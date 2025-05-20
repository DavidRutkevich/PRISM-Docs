---
title: "PRISM – Einleitung"  
description: "Eine Einführung in das PRISM-Framework: Herausforderungen unvollständiger multimodaler Daten und meine Lösung."  
date: 2025-02-05  
math: true
---

In der Computer Vision bezeichnet der Begriff *Modalität* unterschiedliche Datenquellen – beispielsweise visuelle, auditive oder textuelle Informationen, die mit Kameras, Mikrofonen oder anderen Sensoren erfasst werden. Die Kombination verschiedener Modalitäten ermöglicht es, fehlende Informationen auszugleichen – ähnlich wie unser Tastsinn und Gehör uns in dunklen Räumen helfen, uns zurechtzufinden.

In vielen praktischen Anwendungen, etwa in der medizinischen Bildverarbeitung, sind die verfügbaren Daten jedoch oft unvollständig. Bei der Magnetresonanztomographie (MRT) etwa werden verschiedene Sequenzen (z. B. T1, T1ce, T2, FLAIR) erzeugt, die unterschiedliche Gewebekontraste liefern. In der Realität fehlen aber häufig einzelne Sequenzen oder sie liegen in unterschiedlicher Qualität vor.

Um diese Problematik zu adressieren, unterscheide ich zwei Trainingsszenarien:

- **VTD (Vollständiges Training):**  
  Hier sind in jeder Trainingsprobe alle Modalitäten vorhanden. Unter idealen Bedingungen kann das Netzwerk in diesem Szenario multimodale Zusammenhänge optimal lernen, da alle Informationsquellen vollständig vorliegen.

- **UTD (Unvollständiges Training):**  
  In diesem realitätsnahen Szenario fehlen in einzelnen Proben einige Modalitäten. Das Training erfolgt nur mit den tatsächlich verfügbaren Modalitäten, was zu einem Ungleichgewicht führen kann – Modalitäten mit hohen Fehlraten werden seltener aktualisiert und können dadurch im Training benachteiligt sein.

Um diesen Herausforderungen zu begegnen, habe ich **PRISM** entwickelt. Das Framework nutzt ein einzelnes Netzwerk, das gleichzeitig als Teacher und Student fungiert. Dabei wird das in den vorhandenen Modalitäten enthaltene Wissen auf zwei Ebenen weitergegeben:

1. **Pixelbasierte Distillation:**  
   Lokale (pixelweise) Informationen werden zwischen dem multimodalen Teacher und den unimodalen Studentn abgeglichen, sodass selbst bei fehlenden Modalitäten konsistente Vorhersagen möglich sind.

2. **Semantische Distillation:**  
   Globale (klassenbasierte) Informationen werden über Prototypen übertragen. Hierbei wird für jede Klasse ein Prototyp (eine typische Repräsentation) berechnet, der das in den Daten enthaltene Wissen bündelt.

Ergänzt wird dieser Ansatz durch eine **präferenzbasierte Regularisierung**. Da Modalitäten mit hohen Fehlraten im UTD-Szenario weniger oft trainiert werden, wird deren Lernrate dynamisch angepasst. So wird sichergestellt, dass alle Modalitäten – unabhängig von ihrer Häufigkeit – gleichmäßig in den Lernprozess einfließen.

Die zentralen Vorteile von PRISM lassen sich wie folgt zusammenfassen:

- **Realitätsnahe Trainingsszenarien:**  
  Durch die explizite Unterscheidung zwischen VTD und UTD spiegelt PRISM die tatsächlichen Bedingungen in Anwendungen wider, in denen Daten unvollständig vorliegen.

- **Integrierte Self-Distillation:**  
  Ein einzelnes Netzwerk teilt effizient sowohl lokales als auch globales Wissen zwischen den Modalitäten, ohne dass ein separater vollmodaler Teacher benötigt wird.

- **Ausgewogene Regularisierung:**  
  Durch adaptive Anpassung der Lernraten wird verhindert, dass Modalitäten mit hohen Fehlraten im UTD-Szenario benachteiligt werden.

Mit PRISM schließe ich die Lücke zwischen idealen (VTD) und realen (UTD) multimodalen Trainingsszenarien. Das Framework passt die internen Repräsentationen dynamisch an und sorgt dafür, dass der Lernprozess aller Modalitäten ausgewogen abläuft – selbst unter herausfordernden Bedingungen.
