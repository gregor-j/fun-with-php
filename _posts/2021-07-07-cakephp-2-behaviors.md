---
layout: post
title: "CakePHP 2.x Containable Behavior"
bait: "Wer CakePHP 2.x models mit zig `belongsTo`, `hasMany`, etc. Verknüpfungen
hat, und `recursive = 2` an einen Wert zwei Verknüpfungen weiter kommt, sollte
sich das Containable Behavior mal genauer ansehen."
date: 2021-07-07
author: "Gregor J."
tags: php cakephp
---

In diesem Beispiel geht es darum, aus der Tabelle `timezones` den Namen der
Zeitzone zur Umrechnung der in UTC gespeicherten Uhrzeit aus der Tabelle
`appointments` zu selektieren.

![er-diagram]

An den `appointments` und `events` models hängen noch zig weitere models, die
mit `recursive = 2` ebenfalls alle abgefragt werden würden. Leichter ist es im
`AppModel` das [Containable Behavior] zu aktivieren. Das geschieht in der
Klassen-Variable `$actsAs = ['Containable']`. Dann benötigt die Abfrage aller
Teilnehmer:innen an einem Termin mit der korrekt berechneten Uhrzeit kein
`recursive` mehr.

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
                'Timezone' => [
                    'fields' => 'Timezone.timezone'
                ]
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
    0 => [
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

Mehr Details finden sich im CakePHP 2.x Cookbook unter [Containable Behavior].

[Containable Behavior]: https://book.cakephp.org/2/en/core-libraries/behaviors/containable.html
[er-diagram]: https://www.planttext.com/api/plantuml/svg/TP91ZeCm34NtFeLt8p62C_GcA4FSOas8eyIbOWftdq1RL5J8AdR-Vt6-Safgebux1r_OiYNKbZ05KLYgXcFZ2FAXwOKjXYW_Zhg4DzOMZRyuijX4kIKy4ocaN_R1HDEXOqyw14A9sifCIARFQEbclca879qLLJ3ofLMq7AbH5Wz0Nba7l2QA2U-0UHN8TjcKvQCkBAVWpE1DHqMnzAggWcbrcn24lNQvJFKkwljTKw_DMLjlRpicVpAfVwEjqoufAn_CkdXsMBFXJpnzuBBBxQDUbkFWNegj_P7GA6KvtdUoWvqufx4iPTo8DpASPjhuV1qy7pcxSdET4-14tkOF-0y0
