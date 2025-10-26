«<mark title="Requirement MQTT-SN-1.7.4-1"><a name="MQTT-SN-1.7.4-1"></a>The character data in a UTF-8 Encoded String MUST be well-formed UTF-8 as defined by the Unicode specification [\[Unicode]](#Unicode) and restated in RFC 3629 [\[RFC3629]](#RFC3629). In particular, the character data MUST NOT include encodings of code points between U+D800 and U+DFFF</mark>»\[MQTT‑SN‑1.7.4‑1].

If the Client or Server receives an MQTT-SN Control Packet containing ill-formed UTF-8 it is a Malformed Packet. Refer to [[4.12 Handling errors]](#handling-errors) for information about handling errors.

«<mark title="Requirement MQTT-SN-1.7.4-2"><a name="MQTT-SN-1.7.4-2"></a>A UTF-8 Encoded String MUST NOT include an encoding of the null character U+0000</mark>»\[MQTT‑SN‑1.7.4‑2]. If a receiver (Server or Client) receives an Control Packet containing U+0000 in a UTF-8 Encoded String it is a Malformed Packet.

UTF-8 Encoded Strings SHOULD NOT include the Unicode \[Unicode\] code points listed below. If a receiver (Server or Client) receives an MQTT-SN Control Packet with UTF-8 Encoded Strings containing any of them it MAY treat it as a Malformed Packet. These are the Disallowed Unicode code points.

- U+0001..U+001F control characters

- U+007F..U+009F control characters

- Code points defined in the Unicode specification [\[Unicode]](#Unicode) to be non-characters (for example U+0FFFF)

«<mark title="Requirement MQTT-SN-1.7.4-3"><a name="MQTT-SN-1.7.4-3"></a>A UTF-8 encoded sequence 0xEF 0xBB 0xBF is always interpreted as U+FEFF (\"ZERO WIDTH NO-BREAK SPACE\") wherever it appears in a string and MUST NOT be skipped over or stripped off by a packet receiver</mark>»\[MQTT‑SN‑1.7.4‑3].

> **Informative example**
