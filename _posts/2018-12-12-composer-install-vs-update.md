---
layout: post
title: "Composer install vs. update"
bait: "Was ist der Unterschied zwischen composer install und composer update, und warum soll ich die composer.lock in das Repository einchecken?"
date: 2018-12-12
author: "Gregor J."
tags: composer
---

Ein `composer update` sollte man nur machen, wenn man _alle_ Pakete in dem Projekt auf die aktuellste Version (gem. Einschränkungen aus der `composer.json`) bringen möchte. Danach muss das Projekt sehr genau getestet werden, um zu sehen, ob sich durch das Update, etwas geändert hat, das nun _Nebenwirkungen_ verursacht.

Es gibt die Möglichkeit mit `composer.phar update <package name>` _nur ein bestimmtes Paket_ zu aktualisieren, die anderen Pakete aber unangetastet zu lassen. Um _ein bestimmtes Paket mit all seinen Abhängigkeiten_ zu aktualisieren, kann noch die Option `--with-dependencies` hinzugefügt werden.

Der Befehl `composer update` erstellt die Datei `composer.lock`. Dort werden die Versionen und commit IDs aller Pakete und all ihrer Abhängigkeiten gespeichert. 

Beim Aufruf von `composer install` wird die `composer.lock` abgearbeitet. Wäre, die Datei `composer.lock` nicht vorhanden, könnte nicht mit Sicherheit gesagt werden, welche Versionen und welche abhängigen Pakete installiert werden. Ein `composer install` ohne `composer.lock` ist also wie ein `composer update`.

Hier der Ablauf der beiden `composer` Befehle:

[![composer install vs. update][res-svg]][res-svg]

<small>Der [Quelltext der Grafik][res-txt], wie er mit [Planttext][planttext] erzeugt wurde.</small>

[res-svg]: {{ site.url }}/ressource/composer-install-vs-update.svg
[res-txt]: {{ site.url }}/ressource/composer-install-vs-update.txt
[planttext]: https://www.planttext.com/
