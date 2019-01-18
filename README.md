# Push-Mechanismus für Vorgangsänderungen in KreditSmart

Ein externer Client (public subscriber) registriert sich bei EUROPACE und erhält im Gegenzug Zertifikate, um damit eine Verbindung zu AWS herstellen zu können. Für Echtgeschäft und Testumgebung werden getrennte Zertifikate ausgestellt.

Die Zertifikate erhalten Sie von Ihrem Ansprechpartner im **Kredit**Smart-Team. 

[AWS](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/aws-iot-how-it-works.html) versorgt den Client mit Nachrichten anhand von:

- [Topic Subscription](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/topics.html)
- Konfigurationsänderungen ([Device Shadow](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/iot-device-shadows.html))

## Der Partnerbaum als Topic

Ein Subscriber darf mit seinem Zertifikat alle Topics für Plaketten im Partnerbaum unterhalb seiner eigenen Plakette und für sich selbst empfangen. Zusätzlich darf ein Client sein Shadow-Document abfragen. Dieses informiert über das größtmöglich erlaubte Topic. Sollte sich das Topic ändern - z.B. durch Umorganisation im Partnerbaum - muss der Client geeignet reagieren. Hier empfiehlt sich ein `unsubsribe` und ein `subscribe` mit dem neuen Topic.

Für das Topic gelten in [AWS Limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#iot-protocol-limits), die beispielsweise eine Tiefe von maximal 7 Ebenen erlaubt.

## Payload Message Format

Im Body wird eine Payload als JSON in folgendem Format verschickt:

```
{
    "vorgangsnummer": "VN1245",
    "kundenbetreuerPartnerbaum": "ECHTGESCHAEFT/PARTNER1/PARTNER2/PARTNER3",
    "letztesAenderungsDatum": "2019-01-02",
    "quellsystem": "KREDITSMART"
}
```

Zu den Attributen im Detail:

- `vorgangsnummer`:

  Sowohl bei Änderungen im Vorgang als auch im Antrag wird in einer KEX-PUSH Message stets nur die Vorgangsnummer übermittelt.

- `kundenbetreuerPartnerbaum`: 

  Wegen der [Limitierung des Topics](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#iot-protocol-limits) auf maximal 7 Ebenen kann je nach Tiefe der Partnerstruktur ein Topic abgeschnitten werden. Um dem Subscriber dennoch Auskunft über den aktuellen Kundenbetreuer geben zu können, enthält der `kundenbetreuerPartnerbaum` die vollständige Partnerstruktur.

- `letztesAenderungsDatum`:

  Das übermittelte Änderungsdatum entspricht dem aktuellen Zeitstempel beim Versand der Message.

- `quellsystem`:

  Derzeit wird als Quellsystem stets `KREDITSMART` eingetragen. Das Attribut ist für zukünftige Erweiterungen über **Kredit**Smart hinaus vorgesehen.

## Auslöser und Frequenz von Push-Nachrichten

Eine Nachricht wird unabhängig von der Art der Änderung eines Vorgangs oder eines Antrags verschickt. Je nach Aktivität am Vorgang ergibt sich daraus eine entsprechende Frequenz an Push-Nachrichten.

Wir aggregieren derzeit keine Nachrichten, d.h. Clients sind selbst dafür verantwortlich die über AWS potentiell eingehende Last zu verarbeiten.

In **Kredit**Smart werden keine Push-Nachrichten aufbewahrt. Daher gehen Nachrichten in den Zeiträumen verloren, in denen kein Subscriber für die jeweiligen Topics mit AWS verbunden ist.
