---
layout: post
title: "systemd VNC Service"
bait: "Erstellen eines systemd Service um den VNC Server beim Booten zu starten."
date: 2023-12-21
author: "Gregor J."
tags: linux ubuntu vnc systemd
---

## Zielsetzung

Der VNC-Server, den wir bereits eingerichtet haben, soll beim Booten automatisch starten.

## Voraussetzung

Der VNC-Server, wie z.B. TigerVNC, muss auf einem Ubuntu 20.04 oder neuer installiert sein und funktionieren. Die Ausgabe von `vncserver -list` sollte keine laufenden VNC-Server anzeigen.

## Die Unit-Datei anlegen

Wir legen eine [Unit-Datei] im Verzeichnis `~/.config/systemd/user/vnc@.service` an.

```unit file (systemd)
[Unit]
# Die Beschreibung des Services. 
# Der Platzhalter `%i` steht dabei für den Parameter, der an den Service-Namen
# angehängt werden wird. In diesem Fall ist es das Display, das der VNC-Server
# belegen wird. 
# Die Platzhalter `%u` und `%U` stehen für den Login-Namen bzw. die UID.
Description=VNC-Server for %u (%U) on display :%i
# Das Service soll nach dem erreichen des `multi-user.target` gestartet werden.
After=multi-user.target

[Service]
# Das Service soll als _fork_ gestartet werden.
Type=forking
# Das Arbeitsverzeichnis in dem das Service aufgerufen werden soll. Das muss
# das Home-Verzeichnis, Platzhalter `%h`, sein damit der VNC-Server und alle
# Prozesse darunter im Home-Verzeichnis starten und nicht irgendwo.
WorkingDirectory=%h
# Weil das Service als _fork_ gestartet wird, braucht es eine Prozess-ID,
# anhand der `systemd` den Status des Services prüfen kann. Der VNC-Server
# legt diese Datei unter `~/.vnc/<hostname>:<display>.pid` an.
# Platzhalter `%h`: Home-Verzeichnis
# Platzhalter `%H`: Host-Name
# Platzhalter `%i`: das Display- siehe oben.
PIDFile=%h/.vnc/%H:%i.pid
# Alle VNC-Server auf dem Display `%i` müssen gestoppt werden, bevor der
# VNC-Server starten kann. Das `=-` ist übrigens kein Tippfehler, sondern
# notwendig damit der Exit-Status des Befehlt ignoriert wird.
ExecStartPre=-/usr/bin/vncserver -kill :%i
# Der Befehl zum Starten des VNC-Servers auf Display `%i`.
# Der VNC-Server läuft nur am localhost und benötigt daher keine Absicherung.
# Eine Verbindung kann nur über SSH hergestellt werden.
ExecStart=/usr/bin/vncserver -localhost yes -SecurityTypes none :%i
# Der Befehl zum Stoppen des VNC-Servers auf Display `%i`.
ExecStop=/usr/bin/vncserver -kill :%i

[Install]
# Wo das Service installiert werden soll: im `default.target`.
WantedBy=default.target
```

Als Nächstes machen wir `systemd` auf die neue Unit-Datei aufmerksam:

```shell
systemd --user daemon-reload
```

Dann starten wir das Service auf Display 1, das als Parameter nach dem `@` übergeben wird.

```shell
systemd --user start vnc@1.service
```

Anschließend sehen wir nach, ob es beim Starten Probleme gab:

```shell
systemd --user status vnc@1.service
```

Im Status sollte `running` und im anschließenden Log keine Fehlermeldungen stehen:

```
myuser@myhostname:~$ systemd --user status vnc@1
● vnc@1.service - VNC-Server for myuser (1000) on display :1
     Loaded: loaded (/home/myuser/.config/systemd/user/vnc@.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-12-21 09:02:59 CET; 1h 02min ago
    Process: 1230 ExecStartPre=/usr/bin/vncserver -kill :1 || : > /dev/null 2>&1 (code=exited, status=0/SUCCESS)
    Process: 1231 ExecStart=/usr/bin/vncserver -localhost yes -SecurityTypes none :1 (code=exited, status=0/SUCCESS)
   Main PID: 1232 (Xtigervnc)
   #...Prozessbaum...
Dez 21 09:02:59 myhostname vncserver[1231]: Log file is /home/myuser/.vnc/myhostname:1.log
Dez 21 09:02:59 myhostname vncserver[1231]: Use xtigervncviewer -SecurityTypes none :1 to connect to the VNC server.
Dez 21 09:02:59 myhostname systemd[1233]: Started VNC Server for myuser (1000) on display :1.
```

Wir sollten jedenfalls einmal zu dem soeben gestarteten VNC-Server verbinden und nachsehen, ob alles passt.
Anpassungen sollten nur in der Datei `~/.vnc/xstartup` gemacht werden, nicht in der Unit-Datei.

Somit können wir das Service aktivieren:

```shell
systemd --user enable vnc@1
```

Der letzte Test besteht nun darin, den Computer neu zu starten. Der VNC-Server sollte automatisch gestartet werden.

Wenn der VNC-Server einmal hängen bleibt können wir ganz ohne `sudo` das Service einfach neu starten:

```shell
systemd --user restart vnc@1
```

[Unit-Datei]: https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html
