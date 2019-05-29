---
layout: post
title: "Merging arrays"
date: 2019-05-29
author: "Gregor J."
tags: php
---

_RTFM, oder die Untiefen beim Zusammenführen zweier Arrays in PHP erklärt._

Das Zusammenführen (_merge_) zweier Arrays in PHP kann zu unerwarteten Ergebnissen führen.

Gehen wir von folgenden zwei eigentlich assoziativ verwendeten Arrays mit numerischen Schlüsseln aus:
```php
<?php
$tcp = [
    25      => 'SMTP',
    110     => 'POP3',
    143     => 'IMAP',
    'hello' => 'world'
];
$tcp_s = [
    465     => 'SMTPS',
    995     => 'POP3S',
    993     => 'IMAPS',
    'hello' => 'php'
];
$result1 = array_merge($tcp, $tcp_s);
$result2 = $tcp + $tcp_s;
```

Bei `array_merge()` gehen die numerischen Schlüssel verloren:
```php
$result1 = [
    [0] => SMTP
    [1] => POP3
    [2] => IMAP
    [hello] => php
    [3] => SMTPS
    [4] => POP3S
    [5] => IMAPS
];
```
Nachzulesen im [PHP.net Manual von array_merge() unter Beispiel #2][array_merge]:

> Vergessen Sie nicht, dass numerische Schlüssel neu nummeriert werden! 

Damit die Schlüssel nicht verloren gehen, kann das zweite Array mittels `+` an das erste angehängt werden:
```php
$result2 = [
    [25] => SMTP
    [110] => POP3
    [143] => IMAP
    [hello] => world
    [465] => SMTPS
    [995] => POP3S
    [993] => IMAPS
];
```

... allerdings werden bereits existierende Schlüssel dann nicht mehr überschrieben.

Ebenso nachzulesen im [PHP.net Manual von array_merge() unter Beispiel #2][array_merge]:

> Die Schlüssel des ersten Arrays werden beibehalten. Falls ein Schlüssel in beiden Arrays vorkommt, wird das Element des ersten Arrays verwendet und das entsprechende Element des zweiten Arrays ignoriert.

Conclusio: _In PHP sollte selbst bei vermeintlich simplen Funktionen die Bedienungsanleitung gelesen werden._

[array_merge]: https://www.php.net/manual/de/function.array-merge.php#example-6208 "PHP: array_merge - Manual"
