# Push-Mechanismus für Vorgangsänderungen in KreditSmart

Ein externer Client (public subscriber) registriert sich bei EUROPACE und erhält im Gegenzug Zertifikate, um damit eine Verbindung zu AWS herstellen zu können. Für Echtgeschäft und Testumgebung werden getrennte Zertifikate ausgestellt.

Die Zertifikate erhalten Sie von Ihrem Ansprechpartner im KreditSmart-Team. 

[AWS](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/aws-iot-how-it-works.html) versorgt den Client mit Nachrichten anhand von:

- [Topic Subscription](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/topics.html)
- Konfigurationsänderungen ([Device Shadow](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/iot-device-shadows.html))

## Der Partnerbaum als Topic

Ein Subscriber darf mit seinem Zertifikat alle Topics für Plaketten im Partnerbaum unterhalb seiner eigenen Plakette und für sich selbst empfangen. Zusätzlich darf ein Client sein Shadow-Document abfragen. Dieses informiert über das größtmöglich erlaubte Topic. Sollte sich das Topic ändern - z.B. durch Umorganisation im Partnerbaum - muss der Client geeignet reagieren. Hier empfiehlt sich ein `unsubsribe` und ein `subscribe` mit dem neuen Topic.

Für das Topic gelten in [AWS Limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#iot-protocol-limits), die beispielsweise eine Tiefe von maximal 7 Ebenen erlaubt.

Um Vorgangsänderungen nicht nur für die eigene Plakette, sondern auch für alle Personen und Organisationen _unterhalb_ der eigenen Plakette zu empfangen, muss in der Subscriber-Implementierung darauf geachtet werden, dass eine Raute `#` als Platzhalter am Topic ergänzt wird. Wird also ein Zertifikat auf das Topic `ECHTGESCHAEFT/PARTNER1/PARTNER2/PARTNER3` ausgestellt, dann wird im Subscriber das Topic auf `ECHTGESCHAEFT/PARTNER1/PARTNER2/PARTNER3/#` konfiguriert. In der Dokumentation zur [Topic Subscription](https://docs.aws.amazon.com/de_de/iot/latest/developerguide/topics.html) werden die Möglichkeiten im Detail erläutert.

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

  Derzeit wird als Quellsystem stets `KREDITSMART` eingetragen. Das Attribut ist für zukünftige Erweiterungen über KreditSmart hinaus vorgesehen.

## Auslöser und Frequenz von Push-Nachrichten

Eine Nachricht wird unabhängig von der Art der Änderung eines Vorgangs oder eines Antrags verschickt. Je nach Aktivität am Vorgang ergibt sich daraus eine entsprechende Frequenz an Push-Nachrichten.

Wir aggregieren derzeit keine Nachrichten, d.h. Clients sind selbst dafür verantwortlich die über AWS potentiell eingehende Last zu verarbeiten.

In KreditSmart werden keine Push-Nachrichten aufbewahrt. Daher gehen Nachrichten in den Zeiträumen verloren, in denen kein Subscriber für die jeweiligen Topics mit AWS verbunden ist.

## Client-Implementierung

Für die Implementierung und Konfiguration eines MQTT Clients gibt es eine Vielzahl von Beispiel-Implementierungen.
AWS bietet selbst ein passendes Java-SDK an, das unter [github.com/aws/aws-iot-device-sdk-java](https://github.com/aws/aws-iot-device-sdk-java#use-the-sdk) inklusive Beispiel-Code dokumentiert ist.
