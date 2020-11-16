---
layout: post
title: "Mein Netzwerk ist langsam!"
bait: "Im HomeOffice Latenzzeiten zwischen Windows und Linux messen."
date: 2020-11-16
author: "Gregor J."
tags: network latency homeoffice windows linux
---

Wenn nach jedem Klick und jeder Tastatureingabe eine gefühlte Ewigkeit vergeht bis der VNC client reagiert, wird es Zeit für eine Überprüfung der Latenzzeiten. Leider ist ein ~~Windoof~~ Windows Laptop ohne Administrator Berechtigungen von Haus aus nicht mit den geeigneten Mitteln ausgestattet. Abhilfe schafft das Tool [Microsoft/Ethr](https://github.com/Microsoft/Ethr) das es für Windows, Linux und macOS gibt.

Unter Linux eine Server Instanz starten:
```
ethr -s
```

Der Client unter Windows verbindet sich dann zur Server-Instanz unter Linux `-c <server-ip>` und misst die Latenz über TCP `-t l`. Aussagekräftige Daten gibt es nur über einen längeren Zeitraum, z.B. für acht Stunden `-d 8h`.
```
ethr -c <server-ip> -t l -d 0
```

Die Ausgabe der Latenzzeiten erfolgt dann tabellarisch:

```
     Avg      Min      50%      90%      95%      99%    99.9%   99.99%      Max
 58.73ms  40.34ms  55.87ms  69.08ms  76.28ms 102.63ms 140.91ms 140.91ms 186.78ms
 60.87ms  42.74ms  56.29ms  72.51ms  84.50ms 131.57ms 253.01ms 253.01ms 277.35ms
 65.02ms  39.21ms  57.14ms  80.35ms  98.63ms 209.45ms 334.64ms 334.64ms 364.31ms
```

Das Ergebnis lässt sich in einer Tabellenkalkulation, wie z.B. jener von LibreOffice, als CSV mit fixen Abständen importieren und in eine hübsche Grafik verwandeln.

![Latency over 8h](/assets/latency.png)   
