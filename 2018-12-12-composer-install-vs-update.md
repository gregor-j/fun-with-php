---
layout: post
title: "Composer install vs. update"
date: 2018-12-12
author: "Gregor J."
tags: composer
---

_Was ist der Unterschied zwischen `composer install` und `composer update`, und warum soll ich die `composer.lock` in das Repository einchecken?_

Ein `composer update` sollte man nur machen, wenn man _alle_ Pakete in dem Projekt auf die aktuellste Version (gem. Einschränkungen aus der `composer.json`) bringen möchte. 
Danach muss das Projekt sehr genau getestet werden, um zu sehen, ob sich durch das Update, etwas geändert hat, das nun _side effects_ verursacht.

Es gibt noch die Möglichkeit ein `composer update <paketname>` durchzuführen, um _nur ein bestimmtes Paket_ zu aktualisieren.

Ein `composer install` ohne `composer.lock` ist, wie ein `composer update` mit all seinen Nachteilen.

Hier der Ablauf der beiden `composer` Befehle:

[![composer install vs. update][res-svg]][res-svg]

Wie erkennbar ist, braucht ist ein `composer install` mit vorhandener `composer.lock` sehr einfach und schnell.

<small>Der [Quelltext der Grafik][res-txt], wie er mit [Planttext][planttext] erzeugt wurde.</small>

[res-svg]: ressource/composer-install-vs-update.svg
[res-txt]: ressource/composer-install-vs-update.txt
[planttext]: https://www.planttext.com/
