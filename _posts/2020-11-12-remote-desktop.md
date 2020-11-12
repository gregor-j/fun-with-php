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

## SSH

Installation des SSH Servers am Server:
```
sudo apt install ssh-server
```

Wenn die Berechtigungen fehlen, um [PuTTY] zu installieren, kann auch `putty.zip` heruntergeladen und in ein beliebiges Verzeichnis entpackt werden.
 
Mit `puttygen.exe` einen 4096 Bit großen RSA Schlüssel erzeugen und speichern. Den public key aus dem Feld `Public key for pasting into OpenSSH authorized_keys file` heraus kopieren und in die Datei `~/.ssh/authorized_keys` auf dem Server speichern.

Abgesehen von der IP-Adresse des Server müssen folgende Einstellungen in PuTTY vorgenommen und abschließend als `vnctunnel` Session abgespeichert werden:
* Connection → Data → Auto-login username: hier kommt der Benutzername des Linux Servers
* Connection → SSH → Auth → Private key file for authentication: hier muß der zuvor erzeugte private key ausgewählt werden.
* Connection → SSH → Tunnels: unter Source port gehört `5903` und unter Destination `127.0.0.1:5903` eingetragen. Dann mit _Add_ den Tunnel hinzufügen.
* Connection → SSH Remote command (optional):
```
echo -n "Waiting for client..."; until [ $(ss sport = :5903 | grep -nc "ESTAB") -gt 0 ]; do sleep 1; echo -n "."; done; echo "OK"
```

Am Server muss noch die Passwort-Authentifizierung deaktiviert werden:
```
sudo sed -i '/^PasswordAuthentication.*/d' /etc/ssh/sshd_config
echo "PasswordAuthentication no" | sudo tee -a /etc/ssh/sshd_config
```

## VNC

Installation von [TigerVNC] und [Xfce] am Server:
```
sudo apt install tigervnc-standalone-server xfce4
```

In `~/.vnc/xstartup` wird der Start des VNC Servers festgelegt:
```
#!/bin/sh

export XKL_XMODMAP_DISABLE=1
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
eval `dbus-launch --exit-with-session --sh-syntax`
startxfce4 &
```

VNC Server mit einer Auflösung von 1280x1024 auf Port 5903 (also `:3`) starten:
```
vncserver -geometry 1280x1024 -localhost yes -SecurityType None -name HomeOffice :3
```
Die Auflösung jener des Windows Clients anpassen.

Der VNC Server kann folgendermaßen wieder gestoppt werden:
```
vncserver -kill :3
```

Auf dem Windows Client den VNC viewer von [TigerVNC] herunterladen, folgendes konfigurieren und dann abspeichern:
* VNC Server: `127.0.0.1:5903`
* Options → Input → Menu key: F1 (im Gegensatz zur voreingestellten F8 Taste kommt F1 beim Programmieren nicht so häufig zum Einsatz...)
* Options → Security: Sowohl _Encryption_ als auch _Authentication_ None, weil die Verbindung durch SSH getunnelt wird und damit niemals localhost verlässt.
* Options → Screen: _Full-screen mode_ anhaken. Mit F1 kann der Full-screen mode wieder verlassen werden.

## Windows Script

Schließlich folgendes Script am Windows Client ablegen und auf den Desktop, das Startmenü oder wohin auch immer verlinken:
```
@ECHO OFF
start /b cmd /c "C:\PuTTY\plink.exe" -load vnctunnel -batch && "C:\TigerVnc\vncviewer.exe" "C:\TigerVnc\vnctunnel.tigervnc"
```
Das Script geht davon aus, dass
1. `plink.exe` aus dem [PuTTY] ZIP unter `C:\PuTTY\plink.exe`,
2. der VNC Viewer von [TigerVNC] unter `C:\TigerVnc\vncviewer.exe` und
3. die VNC Konfigurationsdatei für TigerVNC unter `C:\TigerVnc\vnctunnel.tigervnc` gespeichert wurde.
Die Pfade bitte realen Bedingungen anpassen.

## Abschließende Bemerkungen

Ein Klick auf die Verknüpfung zum Windows Script und der remote desktop wird gestartet. Bequemer wird es nicht.

Sobald die VNC Verbindung getrennt wird, schließt sich auch das Windows Kommandozeilen Fenster. Dafür sorgt das optional in die PuTTY Session eingetragene _SSH Remote command_, das einfach nur wartet, bis der VNC Client die Verbindung hergestellt hat und sich dann beendet, was es aber nicht kann, solange der VNC Client läuft.

Weil bei der SSH Verbindung das Öffnen einer TTY mit `-N` nicht unterdrückt wird, kann unter Linux mit `last` eine Liste der letzten Anmeldungen am VNC ausgegeben werden.

Weil keine Passwort-Anmeldung am Linux Server möglich ist, schlagen alle brute-force Angriffe auf die Anmeldung fehl. Nicht, dass Script-kiddies es nicht trotzdem versuchen.

Anstatt in PuTTY ein _Private key file for authentication_ auszuwählen, kann auch der SSH Agent `pageant.exe` gestartet werden, der dann z.B. einen SSH Schlüssel von einem Yubikey anstelle eines lokal gespeicherten verwendet. 

[PuTTY]: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
[TigerVNC]: https://tigervnc.org/
[Xfce]: https://xfce.org/
