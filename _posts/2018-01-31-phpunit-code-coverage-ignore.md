---
layout: post
title: "PHPUnit code coverage ignore"
bait: "Die erbittertsten Feinde, um bei PHPUnit eine _code coverage_ von 100% zu erreichen, sind Exceptions, die geworfen werden, wenn Systemressourcen fehlen, oder ebensolche Exceptions in privaten Funktionen."
date: 2018-01-31
author: "Gregor J."
tags: phpunit php codequality
---

Die erbittertsten Feinde, um bei PHPUnit eine _code coverage_ von 100% zu erreichen, sind Exceptions, die geworfen werden, wenn Systemressourcen fehlen, oder ebensolche Exceptions in privaten Funktionen.

PHPUnit hat zu diesem Zweck den _tag_ `@codeCoverageIgnore` mit dem man die Berechnung zur _code coverage_ aussetzen kann.

_Code coverage_ für ein paar untestbare Zeilen aussetzen:

```php
<?php
try {
    $salt_raw = \random_bytes(static::SALT_BYTES);
} catch (\Error $e) {
    $salt_raw = false;
    // @codeCoverageIgnoreStart
} catch (\Exception $e) {
    $salt_raw = false;
} catch (\TypeError $e) {
    $salt_raw = false;
    // @codeCoverageIgnoreEnd
}
```

_Code coverage_ für eine untestbare Funktion aussetzen:

```php
<?php
if (!function_exists('strlen')) {
    /**
     * Calculate the length of a string
     *
     * @codeCoverageIgnore
     *
     * @param string $str
     * @return int
     */
    private function strlen($str)
    {
        //do some crazy stuff
    }
}
```
