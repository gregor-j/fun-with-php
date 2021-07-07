---
layout: post
title: "CakePHP 2.x Containable Behavior"
bait: "Wer CakePHP 2.x models mit zig `belongsTo` und `hasMany` Verknüpfungen hat,
und `recursive = 2` an einen Wert zwei Verknüpfungen weiter kommt, sollte sich
das [Containable Behavior] mal genauer ansehen."
date: 2021-07-07
author: "Gregor J."
tags: php cakephp
---

In diesem Beispiel geht es darum, aus der Tabelle `timezones` den Namen der
Zeitzone zur Umrechnung der Uhrzeit aus der Tabelle `appointments` zu
selektieren.
![er-diagram]

An den `appointments` und `events` models hängen noch zig weitere models, die
mit `recursive = 2` ebenfalls alle abgefragt werden würden. Leichter ist es im
`AppModel` die Klassen Variable `$actsAs = ['Containable']` zu setzen. Dann
benötigt die Abfrage aller Teilnehmer:innen an einem Termin mit der korrekt
berechneten Uhrzeit kein `recursive` mehr.

```php
$participants = $this->AppointmentsUser->find('all', [
    'conditions' => [
        'AppointmentsUser.appointment_id' => $appointmentId
    ],
    'contain' => [
        'Appointment' => [
            'fields' => ['Appointment.starttime'],
            'Event' => [
                'fields' => ['Event.title'],
                'Timezone' => ['fields' => 'Timezone.timezone']
            ]
        ],
        'User' => ['fields' => ['User.displayname']]
    ],
    'fields' => ['AppointmentsUser.id']
]);
```

Ergebnis:

```php
$participants = [
    [
        'AppointmentsUser' => [
            'id' => '10000'
        ],
        'User' => [
            'displayname' => 'Vorname Nachname',
            'id' => '2000'
        ],
        'Appointment' => [
            'starttime' => '2021-07-07 08:00:00',
            'event_id' => '40',
            'id' => '300',
            'Event' => [
                'title' => 'Event-Titel',
                'timezone_id' => '5',
                'Timezone' => [
                    'timezone' => 'Europe/Vienna'
                ]
            ]
        ]
    ]
    //...
];
```

[Containable Behavior]: https://book.cakephp.org/2/en/core-libraries/behaviors/containable.html
[er-diagram]: https://www.planttext.com/api/plantuml/svg/TP91ZeCm34NtFeLt8p62C_GcA4FSOas8eyIbQWjtdq1RLAQGLUpuFzjVEIGrKS-TWozisH9gIvY2ACnHmx5n1FdHzC8MGvGVHrt22skBnfySMMoYN18UYHJIB_jWekdGiIUz1aA9sif4IARFQEaclca871qLLJ3ogLMq7AbH5Wz0NbclU4uK4zu1yocGxNmfoqTSM4x1cQit7HN5dAkg2iP5RK8GzjhbCDM193Zwp_f-BwgNvDItNhVRHtyogN-ZBUFEDYll33jhDbYJu2QUFd1PvRPHByc-Sw-AjVpXzY1bETotp8CjGyUXB6LObQ_gS9numtC_UAuvgN9ocHJWGDxcF_W7
