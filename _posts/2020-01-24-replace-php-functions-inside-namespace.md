---
layout: post
title: "PHP Funktionen in Namespaces ersetzen"
bait: "Wie kann ich eine Methode testen, die globale PHP Funktionen verwendet, wie z.B. fsockopen?"
date: 2020-01-24
author: "Gregor J."
tags: phpunit php
---

Angenommen, du hast eine Methode, die ein `fsockopen()` verwendet und du möchtest in deinem Unit-Test natürlich auch
den Fehlerfall testen.

```php
<?php
namespace vendor\library;
class MySocket
{
    public $socket;
    public function __construct($host, $port) {
        if (($this->socket = fsockopen($host, $port, $errCode, $errMsg)) === false) {
            throw new \RuntimeException($errMsg, $errCode);        
        }
    }
    public function __destruct(){
        fclose($this->socket);
    }
}
```

Hier kommt uns die Fehlertoleranz von PHP entgegen: Eingentlich sollte im Code nämlich `\fsockopen()` stehen und nicht
bloss `fsockopen()`. So sucht PHP zuerst die funktion `\vendor\library\fsockopen()`, die ja nicht existiert, und sucht
erst dann im root-Namespace weiter, wo sie eigentlich existiert.

Das nutzen wir nun aus, indem wir der in der PHP-Datei für den dazugehörigen Unit-Test zwei namespaces definieren:
Einmal `vendor\library` in dem wir eine funktion namens `fsockopen()` definieren und einmal `test\vendor\library` für
den eigentlichen Unit-Test. Achtung: in diesem Fall müssen die Namespaces mit `{` und `}` umschlossen werden!
So schieben wir dem Namespace `vendor\library` zwei neue Funktionen `fsockopen()` und `fclose()` unter.

```php
<?php
namespace vendor\library
{
    function fsockopen($hostname, $port, &$code = null, &$message = null, $timeout = null)
    {
        $code = 111;
        $message = 'Connection denied!';
        return false;
    }
    function fclose()
    {
        return true;
    }
}
namespace test\vendor\library
{
    use \vendor\library\MySocket;
    class MySocketTest extends \PHPUnit_Framework_TestCase
    {
        /**
         * Test exception thrown by constructor.
         * @expectedException \RuntimeException
         * @expectedExceptionMessage Connection denied!
         * @expectedExceptionCode 111
         */
        public function testConstructorException()
        {
            new MySocket('example.com', 1234);
        }
    }
}
```

Der Unit-Test ist etwas einseitig, weil nun die von `\vendor\library\MySocket::__constructor()` verwendete PHP Funktion
`\vendor\library\fsockopen()` immer `false` retourniert. Also bringen wir etwas mehr Flexibilität in das Verhalten der
überschriebenen Funktion `fsockopen()`.

```php
<?php
namespace vendor\library
{
    use test\vendor\library\MySocketTest;
    function fsockopen($hostname, $port, &$code = null, &$message = null, $timeout = null)
    {
        if (is_callable(MySocketTest::$fsockopenResponse)) {
            $func = MySocketTest::$fsockopenResponse;
            return $func($hostname, $port, $code, $message, $timeout);
        }
        return MySocketTest::$fsockopenResponse;
    }
    function fclose()
    {
        return true;
    }
}
namespace test\vendor\library
{
    use \vendor\library\MySocket;
    class MySocketTest extends \PHPUnit_Framework_TestCase
    {
        public static $fsockopenResponse;
        public function setUp()
        {
            parent::setUp();
            static::$fsockopenResponse = null;
        }
        public function testConstructor()
        {
            static::$fsockopenResponse = 'THIS IS A RESOURCE';
            $socket = new MySocket('example.com', 1234);
            static::assertInstanceOf(MySocket::class, $socket);
            static::assertSame('THIS IS A RESOURCE', $socket->socket);
        }
        /**
         * Test exception thrown by constructor.
         * @expectedException \RuntimeException
         * @expectedExceptionMessage Connection denied!
         * @expectedExceptionCode 111
         */
        public function testConstructorException()
        {
            static::$fsockopenResponse = function($host, $port, &$errNo, &$errMessage, $timeout)
            {
                 $errNo = 111;
                 $errMessage = 'Connection denied!';
                 return false;
            };
            new MySocket('example.com', 1234);
        }
    }
}
```

Wichtig ist, dass die statische Variable `$fsockopenResponse` für jeden Test neu initialisiert wird. Das geschieht in
der von PHPUnit bereit gestellten Methode `setUp()`.

Ich habe absichtlich die Funktion `fsockopen()` verwendet, weil sie Rückgabewerte über die referenzierten Parameter
`&$errNo` und `&$errMessage` verwendet. Würde anstelle der einzelnen Übergabe der referenzierten Parameter, die
callable Funktion mittels `return call_user_func_array(MySocketTest::$fsockopenResponse, func_get_args());` aufrufen,
könnte man die relevanten Rückgabewerte `&$errNo` und `&$errMessage` nicht, wie oben, retournieren.

Wie eingangs erwähnt unterbindet die Angabe des Namespaces der gewünschten Funktion, wie beispielsweise `\fsockopen()`
dieses Verhalten.
