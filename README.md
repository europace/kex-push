# Push-Mechanism for changes to Vorgänge in KreditSmart

> ⚠️ You'll find German domain-specific terms in the documentation, for translations and further explanations please refer to our [glossary](https://docs.api.europace.de/common/glossary/)

## New Way (Webhook)

`kex push` delivers webhooks whenever a Vorgang or Antrag changes. This
describes the request format, how to verify it, and what we need
from you to enable delivery.

### Request format

Your endpoint must be a publicly reachable HTTPS URL accepting `POST` with a
JSON body. We send one webhook per change with no aggregation or
deduplication, at-least-once, and without ordering guarantees — so the
endpoint must be idempotent.

- **Method:** `POST`
- **Content-Type:** `application/json`
- **Header:** `X-SIGNATURE` — `base64(HMAC-SHA256(shared_secret, raw_body))`.

#### Body

```json
{
  "vorgangsnummer": "VN1245",
  "teilantragsnummer": "VN1245/1/1"
}
```

| Field               | Type              | Notes                                                                                    |
| ------------------- | ----------------- | ---------------------------------------------------------------------------------------- |
| `vorgangsnummer`    | string, required  |                                                                                          |
| `teilantragsnummer` | string \| `null`  | `null` = the change is on the Vorgang; a value = the change is on the referenced Antrag. |

Additional fields will be added over time. Receivers must not fail on unknown
fields.

### Signature verification

Every request carries `X-SIGNATURE = base64(HMAC_SHA256(shared_secret, raw_body))`.
Verifying it is optional but **strongly** recommended — it lets you confirm the request
actually came from us. Reject any request where the header is
missing or does not match.

- Sign the **raw request body bytes** exactly as received — do not
  re-serialize the JSON.

Java example:

```java
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.util.Base64;
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

public static boolean verify(String sharedSecret, byte[] rawBody, String header) throws Exception {
    Mac mac = Mac.getInstance("HmacSHA256");
    mac.init(new SecretKeySpec(sharedSecret.getBytes(StandardCharsets.UTF_8), "HmacSHA256"));
    String expected = Base64.getEncoder().encodeToString(mac.doFinal(rawBody));
    return MessageDigest.isEqual(
        expected.getBytes(StandardCharsets.UTF_8),
        header.getBytes(StandardCharsets.UTF_8)
    );
}
```

### What we need from you

Registration is per `Datenkontext` (`TESTUMGEBUNG` or
`ECHTGESCHAEFT`). You may register one independent set of URL and secret per context.

Per data context you want to enable, send us following information to kredit.helpdesk@europace2.de:

- **Webhook URL** — the HTTPS URL to POST to.
- **Shared secret** — a high-entropy random string (≥ 32 bytes of entropy,
  e.g. 32 random bytes base64- or hex-encoded. Example command to generate: `openssl rand -base64 32`).
- **Datenkontext** — `TESTUMGEBUNG` and/or `ECHTGESCHAEFT`.
- **Partner ID** — your partner ID; we can look this up if unsure.

Transmit the secret through a secure channel (encrypted attachment, password
manager share) — not plain email or chat.

To rotate the secret or change the URL later, send us the new value and we
replace it. There is no dual-secret overlap window, so plan for a short
cutover.

## Legacy Way (AWS)

The legacy approach for receiving push notifications via MQTT will be deprecated soon (exact date tbd) and is no longer supported for new customers. New customers should use webhooks instead, as described above.

An external client (public subscriber) registers at EUROPACE und receives certificates to establish a connection to the broker in AWS. For Echtgeschäft and Testumgebung you will receive separate certificates.

You can get the certificates from your KreditSmart contact.

[AWS](https://docs.aws.amazon.com/iot/latest/developerguide/aws-iot-how-it-works.html) notifies the client based on:

- [Topic Subscription](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html)
- configuration changes ([Device Shadow](https://docs.aws.amazon.com/iot/latest/developerguide/iot-device-shadows.html))

### The partner tree as topic

A Subscriber is allowed to receive all topics for Plaketten in the partner tree underneath their own Plakette and for themselves. In addition the client is allowed to request a shadow document. This document povides you with the highest possible topic. Should a topic change - e.g. due to restructuring inside the partner tree - the client has to react appropriately. It is recommended to then `unsubscribe` to the old topic and `subscribe` to the new topic.

For the topics the [AWS Limits](https://docs.aws.amazon.com/general/latest/gr/aws_service_limits.html#iot-protocol-limits) apply, for example the depth can be a maximum of 7 layers.

To receive changes not only for your own Plakette but for all other Plaketten _underneath_ your own Plakette you need to use a hash `#` as a wildcard. For example, if your certificate is valid for the topic `ECHTGESCHAEFT/PARTNER1/PARTNER2/PARTNER3` the subscriber needs to be configured for the topic `ECHTGESCHAEFT/PARTNER1/PARTNER2/PARTNER3/#`. You can find more details in the developer guide about [Topic Subscription](https://docs.aws.amazon.com/iot/latest/developerguide/topics.html).

### Payload Message Format

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

### Trigger and frequency of push notifications

A message will be published for arbitrary changes in a Vorgang or Antrag. Therefore the activity in the Vorgang is responsible for the frequency of push notifications.

Currently we do not aggregate or deduplicate any messages, so the client is responsible for handling the potential load of messages.

KreditSmart and the AWS broker don't store any push notifications. Consequently, a subscriber won't receive messages during unconnected time intervals.

### Client-Implementation

There are a number of different examples of how to implement and configure an MQTT client.
AWS provides a Java-SDK which is documented at [github.com/aws/aws-iot-device-sdk-java-v2](https://github.com/aws/aws-iot-device-sdk-java-v2) including sample code.

## Terms of use
The APIs are made available under the following [Terms of Use](https://docs.api.europace.de/terms/).
