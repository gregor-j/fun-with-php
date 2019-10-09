---
layout: post
title: "PHP late static binding"
bait: "Hilfe, ich kann trotz Vererbung meine statische Funktion nicht aufrufen!"
date: 2018-01-24
author: "Gregor J."
tags: php
---

_Hilfe, ich kann trotz Vererbung meine statische Funktion nicht aufrufen!_

```php
<?php
class One
{
    const TEST = 'foo';
    public static function testSelf()
    {
        return self::TEST;
    }
    public static function testStatic()
    {
        return static::TEST;
    }
}
class Two extends One
{
    const TEST = 'bar';
}
printf("Mit 'self' kommt '%s' heraus.%s", Two::testSelf(), PHP_EOL);
printf("Mit 'static' kommt '%s' heraus.%s", Two::testStatic(), PHP_EOL);
```

**Ergebnis**

```
Mit 'self' kommt 'foo' heraus.
Mit 'static' kommt 'bar' heraus.
```

Ob Konstante, Variable oder Funktion: `self` referenziert die Klasse in der es ausgeführt wird und kennt keine Vererbung. `static` hingegen verwendet [late static binding](http://php.net/manual/en/language.oop5.late-static-bindings.php) und ist sich der Vererbung bewusst.
