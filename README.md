# Push-Mechanism for changes to Vorgänge in KreditSmart

> ⚠️ You'll find German domain-specific terms in the documentation, for translations and further explanations please refer to our [glossary](https://docs.api.europace.de/common/glossary/)

An external client (public subscriber) registers at EUROPACE und receives certificates to establish a connection to the broker in AWS. For Echtgeschäft and Testumgebung you will receive separate certificates.

You can get the certificates from your KreditSmart contact.

[AWS](https://docs.aws.amazon.com/iot/latest/developerguide/aws-iot-how-it-works.html) notifies the client based on:

- [Topic Subscription](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html)
- configuration changes ([Device Shadow](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html))

## The partner tree as topic

A Subscriber is allowed to receive all topics for Plaketten in the partner tree underneath their own Plakette and for themselves. In addition the client is allowed to request a shadow document. This document povides you with the highest possible topic. Should a topic change - e.g. due to restructuring inside the partner tree - the client has to react appropriately. It is recommended to then `unsubscribe` to the old topic and `subscribe` to the new topic.

For the topics the [AWS Limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#iot-protocol-limits) apply, for example the depth can be a maximum of 7 layers.

To receive changes not only for your own Plakette but for all other Plaketten _underneath_ your own Plakette you need to use a hash `#` as a wildcard. For example, if your certificate is valid for the topic `ECHTGESCHAEFT/PARTNER1/PARTNER2/PARTNER3` the subscriber needs to be configured for the topic `ECHTGESCHAEFT/PARTNER1/PARTNER2/PARTNER3/#`. You can find more details in the developer guide about [Topic Subscription](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html).

## Payload Message Format

The message body contains the payload as JSON:

```
{
    "vorgangsnummer": "VN1245",
    "kundenbetreuerPartnerbaum": "ECHTGESCHAEFT/PARTNER1/PARTNER2/PARTNER3",
    "letztesAenderungsDatum": "2019-01-02",
    "quellsystem": "KREDITSMART"
}
```

More details about the attributes:

- `vorgangsnummer`:

  For changes in either Vorgang or Antrag the KEX-PUSH message always only contains the Vorgangsnummer.

- `kundenbetreuerPartnerbaum`:

  Because of the [limits for Topics](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#iot-protocol-limits) to a maximium of 7 it can happen that a topic gets cut off if the corresponding partner tree is too deep. To still be able to notify the subscriber about a Kundenbetreuer, the property `kundenbetreuerPartnerbaum` contains the full tree.

- `letztesAenderungsDatum`:

  This property is always the current timestamp at the time the message was sent.

- `quellsystem`:

  Currently only `KREDITSMART` is supported as a possible event source. The property is added to be future proof for possible expansion in other areas of the platform.

## Trigger and frequency of push notifications

A message will be published for arbitrary changes in a Vorgang or Antrag. Therefore the activity in the Vorgang is responsible for the frequency of push notifications.

Currently we do not aggregate or deduplicate any messages, so the client is responsible for handling the potential load of messages.

KreditSmart and the AWS broker don't store any push notifications. Consequently, a subscriber won't receive messages during unconnected time intervals.

## Client-Implementation

There are a number of different examples of how to implement and configure an MQTT client.
AWS provides a Java-SDK which is documented at [github.com/aws/aws-iot-device-sdk-java-v2](https://github.com/aws/aws-iot-device-sdk-java-v2) including sample code.

## Terms of use
The APIs are made available under the following [Terms of Use](https://docs.api.europace.de/terms/).
