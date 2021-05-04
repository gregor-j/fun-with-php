---
layout: post
title: "Linux remote desktop"
bait: "Kurzanleitung um sich unter Ubuntu Linux einen remote desktop einzurichten."
date: 2020-11-12
author: "Gregor J."
tags: linux ubuntu vnc ssh
---

## Zielsetzung

Ein VNC Server unter Ubuntu Linux soll über SSH erreichbar sein. Um die mangelnde Sicherheit von VNC zu kompensieren, soll die VNC Verbindung durch SSH mit Authentifizierung über Schlüssel erfolgen. Als VNC Server und Client wird [TigerVNC] und als Windowmanager [Xfce] verwendet. 

## Voraussetzungen

Voraussetzung ist ein Ubuntu 18.04 LTS als Server mit einem Account mit sudo-Rechten und Windows 10 als Client. Die Anleitung wird sicherlich auch unter anderen Voraussetzungen funktionieren, das wurde aber nicht getestet.

## VNC Server

Installation von [TigerVNC] und [Xfce] am Server:

```
sudo apt install tigervnc-standalone-server xfce4
```

In `~/.vnc/xstartup` wird der Start des VNC Servers festgelegt:

```
#!/bin/sh

export XKL_XMODMAP_DISABLE=1
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
dbus-update-activation-environment --all --systemd
startxfce4 &
```

Den VNC Server auf Display `:3` (also Port `5903`) starten:

```
vncserver -localhost yes -SecurityType None -name HomeOffice :3
```

Der VNC Server kann folgendermaßen wieder gestoppt werden:
```
vncserver -kill :3
```

Diese beiden Kommandos können entweder als Aliase oder als Shell Scripte angelegt werden, um sie bequem aufrufen zu können.

## Linux VNC wait script

Auf dem Server folgendes Script in `/usr/local/bin/vncclientwait.sh` anlegen und mit `chmod +x /usr/local/bin/vncclientwait.sh` ausführbar machen:
```
#!/bin/bash
until [ $(ss sport = :5903 | grep -nc "ESTAB") -gt 0 ]; do
	sleep 1
done
```

Das Script wartet bis eine Verbindung vom VNC Client zum VNC Server hergestellt wurde und beendet sich dann. Wird dieses Script anstelle einer Shell vom SSH Client ausgeführt, wird die SSH Verbindung beendet, sobald auch die VNC Verbindung beendet wird.

## SSH Server

Sofern das noch nicht geschehen ist, muss ein SSH Server unter Linux installiert werden.
```
sudo apt install openssh-server
```

## SSH Schlüssel erzeugen

Wenn die Berechtigungen fehlen, um [PuTTY] unter Windows zu installieren, kann auch `putty.zip` heruntergeladen, in ein beliebiges Verzeichnis entpackt, und PuTTYs Programme direkt aus diesem Verzeichnis gestartet werden.
 
Mit `puttygen.exe` einen 4096 Bit großen RSA Schlüssel erzeugen und z. B. als `LinuxPC.ppk` passwortgeschützt abspeichern. Eine Verknüpfung zu `pageant.exe` aus dem PuTTY Verzeichnis in den Windows Autostart Ordner legen und dann den Pfad zur eben erzeugten PPK Datei als Parameter übergeben. Das _Ziel_ im _Eigenschaften_ Dialog der Verknüpfung sollte also so aussehen: `"C:\PuTTY\pageant.exe" "C:\Daten\LinuxPC.ppk"`

Den public key aus dem Feld `Public key for pasting into OpenSSH authorized_keys file` heraus kopieren und in die Datei `~/.ssh/authorized_keys` auf dem Linux Server speichern.

## Windows Batch Script

Schließlich folgendes Batch Script z. B. als `LinuxPC.cmd` ablegen, und auf den Desktop, das Startmenü oder wohin auch immer verlinken:

```
@ECHO OFF
START /MIN /B "C:\PuTTY\plink.exe" -batch -ssh -L 5903:127.0.0.1:5903 <USER>@<HOST> vncclientwait.sh 

:checkport
netstat -n -a | findstr 127.0.0.1:5903
IF %ERRORLEVEL% EQU 0 (GOTO finish) ELSE (GOTO wait)

:wait
TIMEOUT /T 1 >nul
GOTO checkport

:finish
TIMEOUT /T 1 >nul
"C:\TigerVnc\vncviewer.exe" -FullScreen -RemoteResize -SecurityTypes None -MenuKey F1 127.0.0.1:3
```

Das Script geht davon aus, dass
1. `plink.exe` aus dem [PuTTY] ZIP unter `C:\PuTTY\plink.exe` und
2. der VNC Viewer von [TigerVNC] unter `C:\TigerVnc\vncviewer.exe` gespeichert wurde.
Die Pfade bitte den realen Bedingungen anpassen.

Die Platzhalter `<USER>` bitte durch den Benutzernamen am Linux Server und `<HOST>` durch die IP-Adresse bzw. Hostname des Linux Servers ersetzen.

Der VNC Viewer startet im Vollbild Modus und passt beim Verbinden die Auflösung des VNC Servers an jene des Clients an. Weil ich die Taste F8 unter Linux zum Debuggen benötige habe ich das Menü, um den VNC Viewer im Vollbild Modus bedienen zu können, auf F1 konfiguriert.

## SSH Server absichern

Abschließend muss noch die Passwort-Authentifizierung des SSH Servers deaktiviert werden:

```
sudo sed -i '/^PasswordAuthentication.*/d' /etc/ssh/sshd_config
echo "PasswordAuthentication no" | sudo tee -a /etc/ssh/sshd_config
sudo service ssh restart
```

## Abschließende Bemerkungen

Ein Klick auf die Verknüpfung zum Windows Script und der remote desktop wird gestartet. Bequemer wird es nicht.

Weil bei der SSH Verbindung das Öffnen einer TTY nicht mit `-N` unterdrückt wird, kann unter Linux mit `last` eine Liste der letzten Anmeldungen ausgegeben werden.

Weil keine Passwort-Anmeldung am SSH Server möglich ist, schlagen alle brute-force Angriffe auf die Anmeldung fehl. Nicht, dass Script-kiddies es nicht trotzdem versuchen.

Weil der VNC Server nur am `localhost` auf Verbindungen wartet, braucht es keine gesonderte Absicherung z. B. durch ein Passwort.

[PuTTY]: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
[TigerVNC]: https://tigervnc.org/
[Xfce]: https://xfce.org/
