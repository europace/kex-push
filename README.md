# Push-Mechanismus für Vorgangsänderungen in KreditSmart

Ein externer Client (public subscriber) registriert sich bei EUROPACE und erhält im Gegenzug AWS-Credentials, um im Folgenden eine Verbindung zu AWS aufnehmen zu können. [AWS](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/aws-iot-how-it-works.html) versorgt den Client mit Nachrichten anhand von:

- [Topic Subscription](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/topics.html)
- Konfigurationsänderungen ([Device Shadow](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/iot-device-shadows.html))

## Der Partnerbaum als Topic

Ein Subscriber darf mit seinem Zertifikat alle Topics für Plaketten im Partnerbaum unterhalb seiner eigenen Plakette und für sich selbst empfangen. Zusätzlich darf ein Client sein Shadow-Document abfragen. Dieses informiert über das größtmöglich erlaubte Topic. Sollte sich das Topic ändern - z.B. durch Umorganisation im Partnerbaum - muss der Client geeignet reagieren. Hier empfiehlt sich ein `unsubsribe` und ein `subscribe` mit dem neuen Topic.

Für Echtgeschäft und Testumgebung werden getrennte Zertifikate ausgestellt.

## Payload Message Format

Im Body wird eine Payload als JSON in folgendem Format verschickt:

```
{
    "vorgangsnummer": "VN1245",
    "kundenbetreuerPartnerBaum": "PARTNER1/PARTNER2/PARTNER3",
    "letztesAenderungsDatum": "2019-01-02",
    "quellsystem": "KREDITSMART"
}
```
