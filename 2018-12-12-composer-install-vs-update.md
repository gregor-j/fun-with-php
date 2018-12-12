---
layout: post
title: "Composer install vs. update"
date: 2018-12-12
author: "Gregor J."
categories: php
tags: composer
---

_Was ist der Unterschied zwischen `composer install` und `composer update`, und warum soll ich die `composer.lock` in das Repository einchecken?_

Ein `composer update` sollte man nur machen, wenn man _alle_ Pakete in dem Projekt auf die aktuellste Version (gem. Einschränkungen aus der `composer.json`) bringen möchte. 
Danach muss das Projekt sehr genau getestet werden, um zu sehen, ob sich durch das Update, etwas geändert hat, das nun _side effects_ verursacht.

Es gibt noch die Möglichkeit ein `composer update <paketname>` durchzuführen, um _nur ein bestimmtes Paket_ zu aktualisieren.

Ein `composer install` ohne `composer.lock` ist, wie ein `composer update` mit all seinen Nachteilen.

Hier der Ablauf der beiden `composer` Befehle:

[![composer install vs. update][planttext-svg]][planttext-svg]

Wie erkennbar ist, braucht ist ein `composer install` mit vorhandener `composer.lock` sehr einfach und schnell.

[planttext-svg]: https://www.plantuml.com/plantuml/svg/tLKzRzim4DtvAswC0STkLtj8WA13jg93WOuCTRBOIAH0dkpet_ST9N-aJjiKtMfaYCYxT-_T8u_tMMoIeMkD4IyjaujT7pCdyW5RRKlxlAAXhwomCINOc125AVbT8uRm-yAs8ccOgkY6ZeDOyJ4G_ifs8zBdpNgzOpc2hsBQhH6z_l3nzibcxsw7R7UywYh3eIB5DsOpLQNMPw5OKs_TCu8oMOJiEMKKoLIdsCrRYDiQuskwGPqEKGjb5UXc0beecyBeBv0Z418b1cqlCNdEsm8nuK3IUq1Eih_d62xo3ur7nsy-7pDRheKaKf-Yu_nMyjs2VAQRaNDHGoSe_QdontnEQADx1XPdM79txCMlKZLpba213n0jZGt5rXWOf-4rxX_mBhjyH9rxibPplDpwwWfq-oh_zC5jHfT2N5miXShNs-kwfz3hhKs6MKSk5nwzYyzl2sDjTXA5TSzCDEORj33cdTvClF-x8sh-sO2tsrpMfrRFrKxkcFa_ipcEiRDsvsAgVDYfZ9-glPvi3uMNbD06ccejZgIl3whG8pE93K9FxEm05bxeu55g-h9Xr8jpQ291U0RUq9VhXDPzPrUwg76wn2yM2zbsVBO5VfMPoZsqS73mLQ-aq1W7ieaLC8FRYhx58T738yeaP2-rG3MuCeCxwneVE2_dKrBndjlOeuRsUjH4OTijIoUHF48evAexvLCLf47PJAEWKBoajDGJuXf2e_mE1pQNCA3rw6GGRrkgOwAdOOVErpLqAtuCxKWvTZeSabngRUEeBmZUd4zxN0bS6vKBl9avsHQSbie--nVgOf0cWmDY4yzUJuQT6wq83hEiPcuLoiLGU_2NwmRF25nfXocBsdc8wLaX-KmeJOmk4XEtR3FoyEwCEa3FHi8aLsmEgulNBKliZJ7tMC7h-mC0
