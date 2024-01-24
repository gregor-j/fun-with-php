---
layout: post
title: "LUKS aus der Ferne entschlüsseln"
bait: "Deine Workstation steht im Büro am andere Ende Welt und die Festplatte muss beim Booten entschlüsselt werden?"
date: 2021-03-25
author: "Gregor J."
tags: ssh linux ubuntu luks
---

Diese Anleitung erklärt, wie eine mit LUKS verschlüsselte Festplatte an einem
in einem physisch nicht erreichbaren Ort, aus der Ferne entschlüsselt werden
kann. Die Anleitung wurde für Ubuntu 22.04 LTS geschrieben.

_Danke an Josef für das Zusammentragen der Änderungen für Ubuntu 22.04 LTS!_

Dazu wird ein minimaler SSH Server namens [dropbear] ins initramfs installiert
und ein script zur Eingabe des Passworts bereitgestellt. Das initramfs ist
ein virtuelles Dateisystem im RAM mit einem minimalen Kernel, das sowieso zum
Entschlüsseln des Systems verwendet wird.

_Achtung_: Die Anleitung geht davon aus, dass eine schlüsselbasierte Anmeldung
über RSA keys verwendet wird. Wenn Du zur Anmeldung am SSH Server ein Passwort
brauchst, dann musst Du das jetzt beheben. Du musst Dich am laufenden
System mit einem SSH Schlüssel anmelden können.

Um Fehlermeldungen des SSH clients über einen vermeintlichen man-in-the-middle
Angriff zu vermeiden, soll dropbear einen anderen Port als `22` benutzen. Die
Fehlermeldungen kommen dadurch zustande, weil dropbear einen anderen host key
hat, als der auf deinem entschlüsselten System laufende OpenSSH Server.

## Installation fehlender Pakete

```shell
sudo apt install dropbear-initramfs
```

## Busybox und Dropbear aktivieren

In der initramfs Konfiguration müssen `BUSYBOX` und `DROPBEAR` aktiviert
werden. Es ist wahrscheinlich `BUSYBOX` auf `auto` gesetzt, das reicht nicht
aus.

```shell
sudo vi /etc/initramfs-tools/initramfs.conf
```

Die Konfigurationsparameter sollten folgendermaßen aussehen.

```shell
BUSYBOX=y
```

```shell
DROPBEAR=y
```

## Dropbear konfigurieren

Die host keys und die Konfigurationsdatei von Dropbear liegt in 
`/etc/dropbear/initramfs/`, also wechseln wir dorthin.

```shell
cd /etc/dropbear/initramfs/
```

Die host keys sollten bereits automatisch bei der Installation generiert worden
sein. Wir konvertieren sie lediglich ins _openssh_ Format.

```shell
sudo /usr/lib/dropbear/dropbearconvert dropbear openssh dropbear_rsa_host_key id_rsa
sudo dropbearkey -y -f dropbear_rsa_host_key | grep "^ssh-rsa " | sudo tee id_rsa.pub
```

Nun kopieren wir die Liste der erlaubten SSH Schlüssel für deinen user in das
Konfigurationsverzeichnis von Dropbear.

```shell
cp ~/.ssh/authorized_keys .
```

Die Start-Optionen von Dropbear `DROPBEAR_OPTIONS` gehören noch angepasst:

```shell
sudo vi dropbear.conf
```

Folgende Parameter müssen gesetzt werden:

* `-s` Passwort-basierte Logins deaktivieren.
* `-j` Lokale Port-Weiterleitungen deaktivieren.
* `-k` Entfernte Port-Weiterleitungen deaktivieren.
* `-E` Fehlermeldungen auf STDERR ausgeben anstatt auf syslog.
* `-I 60` Trenne Verbindungen auf denen 60 Sekunden lang nichts übermittelt wurde.
* `-p 59522` Dropbear soll Port `59522` verwenden.

```shell
DROPBEAR_OPTIONS="-s -j -k -E -I 60 -p 59522"
```

## Dropbear starten

In der dropbear Default Konfiguration gehört `NO_START` deaktiviert, damit
dropbear auch startet.

```shell
sudo vi /etc/default/dropbear
```

Das sollte in der Konfiguration so aussehen:

```shell
NO_START=0
```

## LUKS unlock script

Um LUKS nach dem Login über dropbear von der busybox aus entschlüsseln zu
können, braucht es ein script.

```shell
sudo vi /etc/initramfs-tools/hooks/crypt_unlock.sh
```

Kopiere den Inhalt von [gusenneans initramfs-hook] in die Datei und mache die
Datei ausführbar.

```shell
sudo chmod +x /etc/initramfs-tools/hooks/crypt_unlock.sh
```

## Initramfs aktualisieren

Jetzt gehört das initramfs aktualisiert.

```shell
sudo update-initramfs -u
```

## System Service deaktivieren

Da dropbear im entschlüsselten System nicht benötigt wird, weil ja schon
OpenSSH läuft, muss das Service noch deaktiviert werden.

```shell
sudo systemctl disable dropbear
```

## GRUB Splash Screen deaktivieren

Weil der Splash Screen von GRUB die Passworteingabe blockieren würde, muss er
deaktiviert werden.

```shell
sudo vi /etc/default/grub
```

Aus dem Parameter `GRUB_CMDLINE_LINUX_DEFAULT` gehört `splash` gelöscht.
Der Parameter sollte dann folgendermaßen aussehen.

```shell
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```

GRUB gehört danach aktualisiert.

```shell
sudo update-grub
```

## System reboot

```shell
sudo reboot
```

Ein reboot kann je nach System etwas länger dauern. Mit `ping` kann das System
aus der Ferne beobachtet werden.

Nun kannst Du Dich mit `root` und Deinem Schlüssel anmelden, sobald die
Workstation gestartet ist, und auf die Eingabe des LUKS Passworts wartet.

```shell
ssh -p 59522 root@<yourworkstation>
```

Danach kann mit dem Befehl `unlock` das Passwort für LUKS eingegeben werden.
Sobald das Passwort erfolgreich eingegeben wurde, wird die Verbindung getrennt,
weil ja das entschlüsselte System nun startet.

## Sicherheitshinweis

Auch wenn es verlockend ist, sollten die host Schlüssel von OpenSSH nicht für
dropbear verwendet werden, weil das initramfs unverschlüsselt auf der
Festplatte liegt und somit die host Schlüssel von Personen mit physischem
Zugriff ausgelesen werden können.

[dropbear]: https://matt.ucc.asn.au/dropbear/dropbear.html
[gusenneans initramfs-hook]: https://gist.github.com/gusennan/712d6e81f5cf9489bd9f
