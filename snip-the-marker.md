# 1 Introduction

\[<mark title="Ephemeral region marking">All text is normative unless otherwise labeled</mark>\]

## 1.0 Intellectual property rights policy

This specification is provided under the [[Non-Assertion]](https://www.oasis-open.org/policies-guidelines/ipr#Non-Assertion-Mode) Mode of the [[OASIS IPR Policy]](https://www.oasis-open.org/policies-guidelines/ipr), the mode chosen when the Technical Committee was established. For information on whether any patents have been disclosed that may be essential to implementing this specification, and any offers of patent licensing terms, refer to the Intellectual Property Rights section of the TC's web page ([[https://www.oasis-open.org/committees/mqtt/ipr.php]](https://www.oasis-open.org/committees/mqtt/ipr.php)).

## 1.1 Changes from earlier Versions

Here is a description of significant differences from previously published, differently numbered Versions of this specification.

### 1.1.1 MQTT-SN 1.2

- Some terminology has been changed to match MQTT 5.0. For example:

  - Topic Id becomes Topic Alias

  - Message Id becomes Packet Identifier

  - Message Type becomes Packet Type

- The concept of Virtual Connection has been introduced, corresponding to the TCP connection of MQTT.

- The Will Message Packets are removed - setting the Will Message is now done in the CONNECT Packet, as in MQTT.

- The Enhanced Authentication of MQTT 5.0 is supported in CONNECT and CONNACK, and the introduction of the AUTH Packet.

- The *going to sleep* function of DISCONNECT is now the responsibility of a separate Packet - SLEEPREQ, and the response SLEEPRESP. As a result, DISCONNECT never has a response.

- The Short Topic Name has been removed - all publish packets now support full Topic Names as well as Topic Aliases.

- All responses allow a Reason Code to be returned. DISCONNECT also allows a Reason String for enhanced diagnostics. A set of Reason Codes and their use is included.

- Inspired by OSCORE, a Protection Encapsulation is introduced to provide lightweight authentication and encryption.

- Session Expiry, Maximum Packet Size, Assigned Client Identifier and Subscribe Options (No Local, Retain Handling, Retain as Published) have been adopted from MQTT.

## 1.2 Organization of the MQTT-SN specification

The specification is split into six chapters:

- [Chapter](#_heading=h.2bxgwvm) 1 -- Introduction

- Chapter 2 -- MQTT-SN Control Packet format

- Chapter 3 -- MQTT-SN Control Packets

- Chapter 4 -- Operational Behavior

- Chapter 5 - Security

- Chapter 6 -- Conformance

## 1.3 Terminology

The keywords \"MUST\", \"MUST NOT\", \"REQUIRED\", \"SHALL\", \"SHALL NOT\", \"SHOULD\", \"SHOULD NOT\", \"RECOMMENDED\", \"MAY\", and \"OPTIONAL\" in this specification are to be interpreted as described in IETF RFC 2119 \[RFC2119\], except where they appear in text that is marked as non-normative.

**Datagram:**

An independent, self-contained sequence of bytes. If received, the contents of a datagram must be correct.

**Underlying Network:**

The underlying network which provides the means to send datagrams from one network endpoint to another.

**Network Address:**

A unique label provided by the Underlying Network to identify a network endpoint.

To receive datagrams, an MQTT-SN Client or Server listens to the network for packets addressed to a specific Network Address.

**Network Identity:**

The identity used to establish that a sequence of datagrams originates from the same sender. This could be, for example:

- A Network Address

- A DTLS connection ID

- An MQTT-SN Protection Packet Sender Identifier

**Virtual Connection:**

An MQTT-SN construct corresponding to the network connection in MQTT. It associates a Network Identity with a Session, by means of the Client Identifier.

**Application Message:**

The data carried by the MQTT-SN (or MQTT) protocols across the network for the application. When an Application Message is transported by MQTT-SN (or MQTT) it contains payload data, a Quality of Service (QoS), and a Topic Name.

**Client:**

A program or device that uses MQTT-SN. An MQTT-SN Client does one or more of the following:

- creates a Virtual Connection to a Server, then:

  - publishes Application Messages that other Clients might be interested in.

  - subscribes to request Application Messages that it is interested in receiving.

  - unsubscribes to remove a request for Application Messages.

  - deletes the Virtual Connection to the Server.

- without using a Virtual Connection

  - publishes Application Messages to one or more recipients.

**Server:**

A program or device that acts as an intermediary between Clients which publish Application Messages and Clients which have made Subscriptions.

A Server does one or more of the following:

- accepts CONNECT requests from Clients and then:

  - accepts Application Messages published by Clients.

  - processes Subscribe and Unsubscribe requests from Clients.

  - forwards Application Messages that match Client Subscriptions.

  - accepts DISCONNECT requests from connected Clients.

- without using a Virtual Connection:

  - accepts Application Messages.

- opens an MQTT Network Connection to an MQTT Server, then:

  - accepts Application Messages from the MQTT Server and forwards some or all to MQTT-SN Clients.

  - accepts Application Messages from MQTT-SN Clients and forwards some or all to the MQTT Server.

- opens an MQTT Network Connection to an MQTT Server when an MQTT-SN CONNECT request is received, then:

  - forwards equivalent MQTT packets to the MQTT Server for each MQTT-SN packet received

  - forwards equivalent MQTT-SN packets to the MQTT-SN Client for each MQTT packet received

  - closes the MQTT Network Connection when the MQTT-SN Virtual Connection is deleted

<!-- -->

-

- accepts Application Messages from MQTT-SN Clients and forwards some or all to the MQTT Server.

**Gateway:**

An MQTT-SN Server that uses one or more TCP connections to communicate with an MQTT Server.

**MQTT Client:**

A program or device that uses MQTT. An MQTT Client:

- opens the Network Connection to the MQTT Server.

- publishes Application Messages that other MQTT (or MQTT-SN) Clients might be interested in.

- subscribes to request Application Messages that it is interested in receiving.

- unsubscribes to remove a request for Application Messages.

- closes the Network Connection to the Server.

**MQTT Server:**

A program or device that acts as an intermediary between MQTT Clients which publish Application Messages and MQTT Clients which have made Subscriptions.

Also known informally as an MQTT **Broker**.

An MQTT Server:

- accepts Network Connections from MQTT Clients.

- accepts Application Messages published by MQTT Clients.

- processes Subscribe and Unsubscribe requests from MQTT Clients.

- forwards Application Messages that match MQTT Client Subscriptions.

- closes the Network Connection from the MQTT Client.

**Client Identifier:**

A UTF-8 encoded character string which uniquely identifies every Client connecting to a Server.

**Session:**

A stateful interaction between a Client and a Server which is associated with a Client Identifier. Some Sessions last only as long as the Virtual Connection, others can span multiple consecutive Virtual Connections between a Client and a Server.

**Session State:**

The set of data that describes a Session. The Session State held by a Client is different to that held by a Server. See [[4.1 Session state]](#session-state) for details.

**Subscription:**

A Subscription comprises a Topic Filter and a maximum QoS. A Subscription is associated with a single Session. A Session can contain more than one Subscription. Each Subscription within a Session has a different Topic Filter.

**Wildcard Subscription:**

A Wildcard Subscription is a Subscription with a Topic Filter containing one or more wildcard characters. This allows the subscription to match more than one Topic Name. Refer to [[4.7.1.1 Topic wildcards]](#topic-wildcards) for a description of wildcard characters in a Topic Filter.

**Topic Name:**

A label attached to an Application Message which is matched against the Subscriptions known to the Server.

**Topic Alias:**

A Topic Alias is a Two Byte Integer value that is used to identify the Topic instead of using the Topic Name. This reduces Packet sizes, and is useful when the Topic Names are long and the same Topic Names are used repetitively within a Virtual Connection.

**Topic Filter:**

An expression contained in a Subscription to indicate an interest in one or more topics. A Topic Filter can include wildcard characters and can match more than one Topic Name.

**MQTT-SN Control Packet:**

A packet of information that is sent to a Network Address.

**Malformed Packet:**

A Control Packet that cannot be parsed according to this specification. Refer to [[4.12 Handling errors]](#handling-errors) for information about error handling.

**Protocol Error:**

An error that is detected after the packet has been parsed and found to contain data that is not allowed by the protocol or is inconsistent with the state of the Client or Server. Refer to [[4.12 Handling errors]](#handling-errors) for information about error handling.

**Will Message:**

An Application Message which is published by the Server after the Virtual Connection is deleted in cases where the Virtual Connection is not deleted normally. Refer to [[3.1.3 Will Flags]](#will-flags) for information about Will Messages.

**Retained Message:**

An Application Message which is stored by the Server for a Topic Name. When a Client subscribes to a topic which has a Retained Message set, the Server sends the Retained Message to the Client, depending on the setting of the Retain Handling Subscribe Flags. Refer to [[3.7.2 SUBSCRIBE Flags]](#subscribe-flags) and [[4.13 Retained Messages]](#retained-messages) for more information about Retained Messages.

**Disallowed Unicode code point:**

The set of Unicode Control Codes and Unicode Noncharacters which should not be included in a UTF-8 Encoded String. Refer to [[1.7.4 UTF-8 Encoded String]](#utf-8-encoded-string) for more information about the Disallowed Unicode code points.

## 1.4 Normative references

<mark title="Ephemeral region marking">\[Required section.\]</mark>

<mark title="Ephemeral region marking">This appendix contains the normative and informative references that are used in this document.</mark>

<mark title="Ephemeral region marking">While any hyperlinks included in this appendix were valid at the time of publication, OASIS cannot guarantee their long-term validity.</mark>

<mark title="Ephemeral region marking">Note: Any normative work cited in the body of the text as needed to implement the work product must be listed in the Normative References section below. Each reference to a separate document or artifact in this work must be listed here and must be identified as either a Normative or an Informative Reference.</mark>

<mark title="Ephemeral region marking">For all References -- Normative and Informative:</mark>

<mark title="Ephemeral region marking">Recommended approach: Set up **\[Reference\]** label elements as \"Bookmarks\", then create hyperlinks to them within the document at locations from which the references are cited. Citations in the body of the text should be hyperlinked to the appropriate Reference entry, not directly to targets which are not a part of this Work Product.</mark>

<mark title="Ephemeral region marking">The proper format for citation of technical work produced by an OASIS TC (whether Standards Track or Non-Standards Track) is:</mark>

<mark title="Ephemeral region marking">**\[Citation Label\]**</mark>

<mark title="Ephemeral region marking">Work Product title (italicized). Edited by Albert Alston, Bob Ballston, and Calvin Carlson. Approval date (DD Month YYYY). OASIS Stage Identifier and Revision Number (e.g., OASIS Committee Specification Draft 01). Principal URI (stage-specific URI, e.g., with stage component: somespec-v1.0-csd01.html). Latest stage: (static URI, without stage identifiers, used as a symbolic link to most recently published stage of this Version).</mark>



<mark title="Ephemeral region marking">For example:</mark>



**<mark title="Ephemeral region marking">\[OpenDoc-1.2\]</mark>**

<mark title="Ephemeral region marking">Open Document Format for Office Applications (OpenDocument) Version 1.2. Edited by Patrick Durusau and Michael Brauer. 19 January 2011. OASIS Committee Specification Draft 07. https://docs.oasis-open.org/office/v1.2/csd07/OpenDocument-v1.2-csd07.html. Latest stage: https://docs.oasis-open.org/office/v1.2/OpenDocument-v1.2.html.</mark>



<mark title="Ephemeral region marking">Reference sources:</mark>

<mark title="Ephemeral region marking">For references to IETF RFCs, use the approved citation formats at:</mark>

<mark title="Ephemeral region marking">[[https://docs.oasis-open.org/templates/ietf-rfc-list/ietf-rfc-list.html]](https://docs.oasis-open.org/templates/ietf-rfc-list/ietf-rfc-list.html).</mark>

<mark title="Ephemeral region marking">The most recent IETF RFC references are listed by the IETF at [[https://www.rfc-editor.org/in-notes/rfc-ref.txt]](https://www.rfc-editor.org/in-notes/rfc-ref.txt).</mark>

<mark title="Ephemeral region marking">For references to W3C Recommendations, use the approved citation formats at:</mark>

<mark title="Ephemeral region marking">[[https://docs.oasis-open.org/templates/w3c-recommendations-list/w3c-recommendations-list.html]](https://docs.oasis-open.org/templates/w3c-recommendations-list/w3c-recommendations-list.html).</mark>

<mark title="Ephemeral region marking">Remove this note before submitting for publication.</mark>

**\[RFC2119\]**

Bradner, S., \"Key words for use in RFCs to Indicate Requirement Levels\", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997,

[[http://www.rfc-editor.org/info/rfc2119]](http://www.rfc-editor.org/info/rfc2119)

<mark title="Ephemeral region marking">\[RFC8174\]</mark>

<mark title="Ephemeral region marking">Leiba, B., \"Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words\", BCP 14, RFC 8174, DOI 10.17487/RFC8174, May 2017, \<[[https://www.rfc-editor.org/info/rfc8174]](https://www.rfc-editor.org/info/rfc8174)\>.</mark>

**\[RFC3629\]**

Yergeau, F., \"UTF-8, a transformation format of ISO 10646\", STD 63, RFC 3629, DOI 10.17487/RFC3629, November 2003,

[[http://www.rfc-editor.org/info/rfc3629]](http://www.rfc-editor.org/info/rfc3629)

**\[RFC6455\]**

Fette, I. and A. Melnikov, \"The WebSocket Protocol\", RFC 6455, DOI 10.17487/RFC6455, December 2011,

[[http://www.rfc-editor.org/info/rfc6455]](http://www.rfc-editor.org/info/rfc6455)

**\[Unicode\]**

The Unicode Consortium. The Unicode Standard,

[[http://www.unicode.org/versions/latest/]](http://www.unicode.org/versions/latest/)

## 1.5 Informative References

<mark title="Ephemeral region marking">\[RFC3552\]</mark>

<mark title="Ephemeral region marking">Rescorla, E. and B. Korver, \"Guidelines for Writing RFC Text on Security Considerations\", BCP 72, RFC 3552, DOI 10.17487/RFC3552, July 2003, \<[[https://www.rfc-editor.org/info/rfc3552]](https://www.rfc-editor.org/info/rfc3552)\>.</mark>

<mark title="Ephemeral region marking">\[Reference\]</mark>

<mark title="Ephemeral region marking">\[Full reference citation\]</mark>

## 1.6 MQTT For Sensor Networks (MQTT-SN)

Sensor Networks are simple, low cost and easy to deploy. They are typically used to provide event detection, monitoring, automation, process control and more. Sensor Networks often comprise many battery-powered sensors and actuators, each containing a limited amount of storage and processing capability. They usually communicate wirelessly.

Sensor Networks are typically self-forming, continually changing, and do not have any central control. The wireless network connections and processing nodes will fail, and the batteries will run out. The nodes will be replaced, added or removed in an unplanned way. The identities of the devices are usually created when they are manufactured, this avoids the need for specialist configuration when they are deployed. Applications running outside the Sensor Network do not need to know the details of the devices in it. The applications consume information from the sensors and send instructions to actuators based only on labels created by the application designers. The labels are called Topic Names in the MQTT and MQTT-SN protocols. The MQTT-SN implementation carries information between a set of applications and the correct set of devices based on its knowledge of the network and the applications designer's choice of Topic Names.

Consider an example of a medicine tracking application. The application needs to know the location and temperature of the medicine, but it does not want to concern itself with the network details of the devices providing the data. It may be that the number and types of the devices changes over time. There may also be other applications using the same sensor data for other purposes. The model is that the devices and applications produce and consume data addressed by the Topics rather than the other devices and applications.

This MQTT-SN specification is a variant of the MQTT version 5 specification. It is adapted to exploit low power and low bandwidth wireless networks. Low power wireless radio links typically have higher numbers of transmission errors compared to more powerful networks because they are more susceptible to interference and fading of the radio signals. They also have lower transmission rates.

For example, wireless networks based on the IEEE 802.15.4 standard used by Zigbee have a maximum bandwidth of 250 kbit/s in the 2.4 GHz band. To reduce transmission errors the packets are kept short. The maximum packet length at the physical layer is 128 bytes and half of these may be used for Media Access Control and security.

The MQTT-SN protocol is optimized for implementation on low-cost, battery-operated devices with limited processing and storage resources. The capabilities are kept simple and the specification allows partial implementations.

### 1.6.1 Differences Between MQTT-SN and MQTT

To facilitate interoperation MQTT-SN is similar in many ways to MQTT, but the two are independent of each other.

MQTT-SN can work isolated from other networks or in conjunction with MQTT. The main differences between MQTT-SN and MQTT are:

1.  In addition to Topic Alias and long Topic Names MQTT-SN allows Predefined Topic Aliases.

2.  Support for sleeping clients allows battery operated devices to enter a low power mode. In this state, Application Messages for the Client are buffered by the Server and delivered when the client wakes.

3.  A new Quality of Service level (WITHOUT SESSION) is introduced in MQTT-SN, allowing devices to publish without a session having been established.

4.  MQTT-SN has fewer requirements on the underlying transport and it can use connectionless network transports such as User Datagram Protocol (UDP).

5.  MQTT-SN introduces the PROTECTION packet for packet-based security based on symmetric-key cryptography.

6.  If the network supports sending messages to more than one recipient at once, Gateway Advertisement and Discovery can be implemented.

## 1.7 Data representation

### 1.7.1 Bits (Byte)

Bits in a byte are labeled 7 to 0. Bit number 7 is the most significant bit, the least significant bit is assigned bit number 0.

### 1.7.2 Two Byte Integer

Two Byte Integer data values are 16-bit unsigned integers in big-endian order: the high order byte precedes the lower order byte. This means that a 16-bit word is presented on the network as Most Significant Byte (MSB), followed by Least Significant Byte (LSB).

### 1.7.3 Four Byte Integer

Four Byte Integer data values are 32-bit unsigned integers in big-endian order: the high order byte precedes the successively lower order bytes. This means that a 32-bit word is presented on the network as Most Significant Byte (MSB), followed by the next most Significant Byte (MSB), followed by the next most Significant Byte (MSB), followed by Least Significant Byte (LSB).

### 1.7.4 UTF-8 Encoded String

Text fields within the MQTT-SN Control Packets are encoded as fixed length UTF-8 strings. UTF-8 [\[RFC3629\]](#RFC3629) is an efficient encoding of Unicode [\[Unicode\]](#Unicode) characters that optimizes the encoding of ASCII characters in support of text-based communications.

Unless stated otherwise all variable length UTF-8 encoded strings can have any length in the range 0 to 65,535 bytes.

*Figure 1-1 -- Structure of UTF-8 Encoded Strings*

<img src="media/image6.png" alt="" title="" width="650" />

«<mark title="Requirement MQTT-SN-1.7.4-1"><a name="MQTT-SN-1.7.4-1"></a>The character data in a UTF-8 Encoded String MUST be well-formed UTF-8 as defined by the Unicode specification [\[Unicode\]](#Unicode) and restated in RFC 3629 [\[RFC3629\]](#RFC3629). In particular, the character data MUST NOT include encodings of code points between U+D800 and U+DFFF</mark>»\[MQTT‑SN‑1.7.4‑1].

If the Client or Server receives an MQTT-SN Control Packet containing ill-formed UTF-8 it is a Malformed Packet. Refer to [[4.12 Handling errors]](#handling-errors) for information about handling errors.

«<mark title="Requirement MQTT-SN-1.7.4-2"><a name="MQTT-SN-1.7.4-2"></a>A UTF-8 Encoded String MUST NOT include an encoding of the null character U+0000</mark>»\[MQTT‑SN‑1.7.4‑2]. If a receiver (Server or Client) receives an Control Packet containing U+0000 in a UTF-8 Encoded String it is a Malformed Packet.

UTF-8 Encoded Strings SHOULD NOT include the Unicode \[Unicode\] code points listed below. If a receiver (Server or Client) receives an MQTT-SN Control Packet with UTF-8 Encoded Strings containing any of them it MAY treat it as a Malformed Packet. These are the Disallowed Unicode code points.

- U+0001..U+001F control characters

- U+007F..U+009F control characters

- Code points defined in the Unicode specification [\[Unicode\]](#Unicode) to be non-characters (for example U+0FFFF)

«<mark title="Requirement MQTT-SN-1.7.4-3"><a name="MQTT-SN-1.7.4-3"></a>A UTF-8 encoded sequence 0xEF 0xBB 0xBF is always interpreted as U+FEFF (\"ZERO WIDTH NO-BREAK SPACE\") wherever it appears in a string and MUST NOT be skipped over or stripped off by a packet receiver</mark>»\[MQTT‑SN‑1.7.4‑3].

> **Informative example**
>
> For example, the string A𪛔 which is LATIN CAPITAL Letter A followed by the code point U+2A6D4 (which represents a CJK IDEOGRAPH EXTENSION B character) is encoded as follows:

*Figure 1-2 -- Fixed Length UTF-8 Encoded String informative example*

> <img src="media/image31.png" alt="" title="" width="6.5in" height="2.5972222222222223in" />
