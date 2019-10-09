---
layout: post
title: "APT upgrade vs. dist-upgrade"
bait: "Aus der Kategorie quälende Fragen zwischen Tür und Angel: Was ist der Unterschied zwischen apt-get upgrade und apt-get dist-upgrade?"
date: 2018-12-11
author: "Gregor J."
tags: linux debian
---

Aus der Kategorie "quälende Fragen zwischen Tür und Angel": _Was ist der Unterschied zwischen `apt-get upgrade` und `apt-get dist-upgrade`?_

`upgrade` wird unter keinen Umständen zusätzliche Pakete installieren, oder nicht mehr benötigte deinstallieren. Dadurch können Paketaktualisierungen auch zurückgehalten werden, weil diese eben entweder zusätzliche Pakete benötigen, oder bisher installierte eben nicht mehr. `dist-upgrade` hingegen löst alle Abhängigkeiten auf und installiert alle Paketaktualisierungen mit allen Abhängigkeiten.

Quelle: `man apt-get`
```
upgrade
   upgrade wird benutzt, um die neusten Versionen aller aktuell auf
   dem System installierten Pakete aus den in /etc/apt/sources.list
   aufgezählten Quellen zu installieren. Aktuell installierte Pakete
   mit verfügbaren neuen Versionen werden heruntergeladen und das
   Upgrade durchgeführt. Unter keinen Umständen werden derzeit
   installierte Pakete entfernt oder nicht installierte Pakete
   heruntergeladen und installiert. Neue Versionen von aktuell
   installierten Paketen von denen kein Upgrade durchgeführt werden
   kann, ohne den Installationsstatus eines anderen Paketes zu ändern,
   werden in ihrer aktuellen Version bleiben. Zuerst muss ein update
   durchgeführt werden, so dass apt-get die neuen Versionen der
   verfügbaren Pakete kennt.

dist-upgrade
   dist-upgrade führt zusätzlich zu der Funktion von upgrade
   intelligente Handhabung von Abhängigkeitsänderungen mit neuen
   Versionen von Paketen durch. apt-get hat ein »intelligentes«
   Konfliktauflösungssystem und es wird versuchen, Upgrades der
   wichtigsten Pakete, wenn nötig zu Lasten der weniger wichtigen, zu
   machen. So könnte der dist-upgrade-Befehl einige Pakete entfernen.
   Die /etc/apt/sources.list-Datei enthält eine Liste mit Orten, von
   denen gewünschte Paketdateien abgerufen werden. Siehe auch
   apt_preferences(5) für einen Mechanismus zum überschreiben der
   allgemeinen Einstellungen für einzelne Pakete.
```
