---
layout: post
title: "PHPUnit Data Provider"
date: 2019-10-09
author: "Gregor J."
tags: phpunit php
---

_Wie kann ich mit PHPUnit effizient unterschiedliche Parameter für meine Funktion testen?_

In folgendem Fall wird das [Beispiel zu Data Provider von der PHPUnit Website][1] verwendet.

Es soll getestet werden, ob die Addition in PHP das gewünschte Ergebnis bringt (spoiler: ja, tut es). Dazu wird in
 einer Testfunktion `testAdditions()` das Ergebnis der Addition mittels `assertSame()` validiert. Die Parameter der
 Testfunktion werden mittels `@dataprovider` befüllt. Dazu retourniert im konkreten Fall die in `@dataProvider`
 angegebene Funktion `provideAdditions` drei Arrays mit Werten für die Parameter der Testfunktion in der korrekten
 Reihenfolge - also an Position 0 den Wert für `$a`, an Position 1 den Wert für `$b` und an dritter Position den Wert
 für `$expected`. 

```php
<?php

class AdditionTest extends PHPUnit_Framework_TestCase
{
    /**
     * Data provider for additions.
     * @return array
     */
    public function provideAdditions()
    {
        return [
            [1, 2, 3],
            [2, 2, 4],
            [12, 30, 42]
        ];
    }

    /**
     * Test additions.
     * @param int $a
     * @param int $b
     * @param int $expected
     * @dataProvider provideAdditions
     */
    public function testAdditions($a, $b, $expected)
    {
        static::assertSame($expected, $a+$b);
    }
}
```

Nehmen wir für das nächste Beispiel eine zu testende Klasse `Integer` an.

```php
<?php

namespace GregorJ\StrictTypes;

class Integer
{
    private $value;
    
    public function __construct($value)
    {
        if (!is_int($value)) {
            throw new \InvalidArgumentException('Expected an integer!');
        }
        $this->value = $value;
    }
}
```

Niemand hat gesagt, dass die _data provider_ Funktion, welche die Parameter für die Testfunktion bereit stellt, in
 derselben Klasse wie die Testfunktion zu sein hat.

```php
<?php

namespace Tests\GregorJ\StrictTypes;

class DataProviders
{
    public static function allExceptInteger()
    {
        return [
            ['1'],
            ['Hello!'],
            [1.23],
            [true],
            [false],
            [null],
            [[1]],
            [new \stdClass()]
        ];
    }
}
```

Bei der Gelegenheit sehen wir auch gleich, wie mit [PHPUnit Exceptions mittels `@expectedException` validiert][2]
 werden können.

```php
<?php

namespace Tests\GregorJ\StrictTypes;

use GregorJ\StrictTypes\Integer;

class IntegerTest extends PHPUnit_Framework_TestCase
{
    /**
     * @dataProvider \Tests\GregorJ\StrictTypes\DataProviders::allExceptInteger
     * @expectedException \InvalidArgumentException
     * @expectedExceptionMessage Expected an integer value!
     */
    public function testInvalidParametersForConstructor($value)
    {
        new Integer($value);
    }
}
```

[1]: https://phpunit.de/manual/6.5/en/writing-tests-for-phpunit.html#writing-tests-for-phpunit.data-providers
[2]: https://phpunit.de/manual/6.5/en/writing-tests-for-phpunit.html#writing-tests-for-phpunit.exceptions
