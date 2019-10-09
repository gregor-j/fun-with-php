---
layout: post
title: "PHPUnit code coverage"
bait: "Woher weiß ich, dass meine Tests den gesamten Code prüfen?"
date: 2018-01-25
author: "Gregor J."
tags: phpunit php codequality
---

_Woher weiß ich, dass meine Tests den gesamten Code prüfen?_

Um zu sehen, zu wie viel Prozent der eigene Code von unit tests abgedeckt wird, 
Erst in der `phpunit.xml` die Verzeichnisse mit dem eigenen Code als Filter angeben:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit colors="true" bootstrap="vendor/autoload.php">
    <testsuites>
        <testsuite name="Project test suite">
            <directory suffix="Tests.php">tests/</directory>
        </testsuite>
    </testsuites>
    <filter>
        <whitelist processUncoveredFilesFromWhitelist="true">
            <directory suffix=".php">src/</directory>
        </whitelist>
    </filter>
</phpunit>
```
Dann folgenden Parameter an `phpunit` anfügen: `--coverage-text`.

**Ergebnis**
```
                        
 Summary:
  Classes: 100.00% (1/1)
  Methods: 100.00% (4/4)
  Lines:   100.00% (89/89)

\Johamg\password_hashing::Johamg\password_hashing\PasswordStorage
  Methods: 100.00% ( 4/ 4)   Lines: 100.00% ( 89/ 89)
```

Für etwas mehr Details kann man sich das Ergebnis auch als HTML Seite nach `coverage/` exportieren lassen: `--coverage-html ./coverage/`, und die darin enthaltene `index.html` öffnen.

In GitLab kann die Ausgabe von `--coverage-text` im Projekt unter "Settings" -> "CI/CD Pipelines" analysiert, und das Ergebnis dieser Analyse kann dann als Plakette ins Readme eingefügt werden.
