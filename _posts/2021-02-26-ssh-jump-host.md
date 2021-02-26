---
layout: post
title: "SSH Sprünge"
bait: "Wenn ein SSH Server nur über einen zweiten SSH Server erreichbar ist."
date: 2021-02-26
author: "Gregor J."
tags: network ssh linux
---

## Das Problem

Der eigentliche `server1` ist nur über `server2` erreichbar. Somit müssen auf
`server2` wieder eine Konfiguration und ein Schlüssel gepflegt werden.

```shell
gregor@workstation $ ssh server2
Last login: Thu Feb 26 08:31:26 2021 from workstation
gregor@server2 $ ssh server1
Last login: Thu Feb 26 08:31:45 2021 from server2
gregor@server1 $ 
```

Zwei Konfigurationen, zwei Schlüssel und keine Möglichkeit eine Smartcard als
Schlüssel zu verwenden. Das ist so … 1995.

## Die Lösung

Der `server2` kann in der Konfiguration `~/.ssh/config` als _JumpHost_
definiert werden.

```
Host *
    User gregor
    IdentityFile ~/.ssh/id_ecdsa

Host server2
    HostName server2.localdomain

Host server1
    HostName server1.localdomain
    JumpHost server2 
```

Und schon erfolgt die Verbindung in einem Schritt.

```shell
gregor@workstation $ ssh server1
Last login: Thu Feb 26 08:32:17 2021 from server2
gregor@server1 $ 
```

Auf einem mobilen Computer, der mal vor und mal hinter `server2` steht, macht
ein fixer Eintrag des _JumpHost_ in der Konfiguration keinen Sinn. Der
_JumpHost_ kann mit dem Parameter `-J` auch auf der Kommandozeile eingegeben
werden.

```shell
gregor@workstation $ ssh -J server2 server1
Last login: Thu Feb 26 08:33:01 2021 from workstation
gregor@server1 $ 
```

Eine Konfiguration, ein Schlüssel. Danke, openssh!
