---
title: "RoQ qlog event definitions"
category: info

docname: draft-engelbart-qlog-roq-events-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - RoQ
 - qlog
 - RTP
 - QUIC
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "mengelbart/draft-engelbart-qlog-roq-events"
  latest: "https://mengelbart.github.io/draft-engelbart-qlog-roq-events/draft-engelbart-qlog-roq-events.html"

author:
 -
    fullname: Mathis Engelbart
    organization: Technical University of Munich
    email: mathis.engelbart@gmail.com

normative:

informative:

--- abstract

This document describes concrete qlog event definitions and their metadata for
RTP over QUIC {{!I-D.draft-ietf-avtcore-rtp-over-quic}} related events. These
events can then be embedded in the higher level schema defined in
{{!I-D.draft-ietf-quic-qlog-main-schema}}.

--- middle

# Introduction

This document describes the values of the qlog name ("category" + "event") and
"data" fields and their semantics for RTP over QUIC
({{!I-D.draft-ietf-avtcore-rtp-over-quic}}).

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The event and data structure definitions in this document are expressed in the
Concise Data Definition Language {{!RFC8610}} and its extensions described in
{{!I-D.draft-ietf-quic-qlog-main-schema}}.

**TODO: Check if we need all these imports:**

The following fields from {{!I-D.draft-ietf-quic-qlog-main-schema}} are imported
and used: name, category, type, data, group_id, protocol_type, importance,
RawInfo, and time-related fields.

As is the case for {{!I-D.draft-ietf-quic-qlog-main-schema}}, the qlog schema
definitions in this document are intentionally agnostic to serialization
formats. The choice of format is an implementation decision.

# Overview

This document describes how RoQ can be expressed in qlog using the schema
defined in {{!I-D.draft-ietf-quic-qlog-main-schema}}. RoQ events are defined
with a category, a name (the concatenation of "category" and "event"), an
"importance", and "data" fields.

When any event from this document is included in a qlog trace, the
"protocol_type" qlog array field MUST contain an entry with the value "RoQ".

## Usage with QUIC

The events described in this document can be used with or without logging the
related QUIC events defined in {{!I-D.draft-ietf-quic-qlog-quic-events}}. If
used with QUIC events, the QUIC document takes precedence in terms of
recommended filenames and trace separation setups.

If used without QUIC events, it is recommended that the implementation assign a
globally unique identifier to each RoQ connection. This identifier can then be
used as the value of the qlog "group_id" field, as well as the qlog filename or
file identifier, potentially suffixed by the vantagepoint type (For example,
abcd1234_server.qlog would contain the server-side trace of the connection with
identifier abcd1234).

# RoQ Event Overview

This document defines events in the `roq` category.

As described in {{Section 3.4.2 of !I-D.draft-ietf-quic-qlog-main-schema}}, the
qlog "name" field is the concatenation of category and type.

{{roq-events-tab}} summarizes the name value of each
event type that is defined in this specification.

| Name value                  | Importance | Definition                  |
| --------------------------- | ---------- | --------------------------- |
| roq:stream_opened           | Base       | {{stream-opened}}           |
| roq:stream_packet_created   | Core       | {{stream-packet-created}}   |
| roq:stream_packet_parsed    | Core       | {{stream-packet-parsed}}    |
| roq:datagram_packet_created | Core       | {{datagram-packet-created}} |
| roq:datagram_packet_parsed  | Core       | {{datagram-packet-parsed}}  |
{: #roq-events-tab title="RoQ Events"}


# RoQ Events

RoQ events extend the `$ProtocolEventData` extension point defined in
{{!I-D.draft-ietf-quic-qlog-main-schema}}.

~~~ cddl
RoQEventData = RoQStreamOpened /
               RoQStreamPacketCreated /
               RoQStreamPacketParsed /
               RoQDatagramPacketCreated /
               RoQDatagramPacketParsed
~~~
{: #roq-event-data-def title="RoQEventData definition and ProtocolEventData extension" }

RoQ events are logged when a certain condition happens at the application layer,
and there isn't always a one to one mapping between RoQ and QUIC events. The
exchange of data between the RoQ and QUIC layer is logged via the
"stream_data_moved" and "datagram_data_moved" events in
{{!I-D.draft-ietf-quic-qlog-quic-events}}.

## RoQ Stream Opened {#stream-opened}

The `stream_opened` event is emitted when a new QUIC stream for sending RoQ
packets is opened. It has Base importance level; see {{Section 9.2 of
!I-D.draft-ietf-quic-qlog-main-schema}}.

~~~ cddl
RoQStreamOpened = {
    flow_id: uint64
    stream_id: uint64
}
~~~
{: #roq-stream-opened-def title="RoQStreamOpened definition" }

## RoQ Stream Packet Created {#stream-packet-created}

The `stream_packet_created` event is emitted when a RoQ packet to be sent on a
QUIC stream is created. It has Core importance level; see {{Section 9.2 of
!I-D.draft-ietf-quic-qlog-main-schema}}.

This event is not necessarily the same as when the packet is passed to the QUIC
layer. For that, see the `stream_data_moved` event in
{{!I-D.draft-ietf-quic-qlog-quic-events}}.

~~~ cddl
RoQStreamPacketCreated = {
    ~RoQPacket
    stream_id: uint64
}
~~~
{: #roq-stream-packet-created-def title="RoQStreamPacketCreated definition" }

## RoQ Stream Packet Parsed {#stream-packet-parsed}

The `stream_packet_parsed` event is emitted when a RoQ packet received from a
QUIC stream is parsed. It has Core importance level; see {{Section 92. of
!I-D.draft-ietf-quic-qlog-main-schema}}.

The event is not necessarily the same as when the packet is received from the
QUIC layer. For that see `stream_data_moved` event in
{{!I-D.draft-ietf-quic-qlog-quic-events}}.

~~~ cddl
RoQStreamPacketParsed = {
    ~RoQPacket
    stream_id: uint64
}
~~~
{: #roq-stream-packet-parsed-def title="RoQStreamPacketParsed definition" }

## RoQ Datagram Packet Created {#datagram-packet-created}

The `roq_datagram_packet_created` event is emitted when a RoQ packet to be sent
in a QUIC Datagram is created. It has Core importance level; see {{Section 9.2
of !I-D.draft-ietf-quic-qlog-main-schema}}.

This event is not necessarily the same as when the packet is passed to the QUIC
layer. For that, see the `datagram_data_moved` event in
{{!I-D.draft-ietf-quic-qlog-quic-events}}.

~~~ cddl
RoQDatagramPacketCreated = {
    ~RoQPacket
}
~~~
{: #roq-datagram-packet-created-def title="RoQDatagramPacketCreated definition" }

## RoQ Datagram Packet Parsed {#datagram-packet-parsed}

The `datagram_packet_parsed` event is emitted when a RoQ packet received in a
QUIC Datagram is parsed. It has Core importance level; see {{Section 92. of
!I-D.draft-ietf-quic-qlog-main-schema}}.

The event is not necessarily the same as when the packet is received from the
QUIC layer. For that see `datagram_data_moved` event in
{{!I-D.draft-ietf-quic-qlog-quic-events}}.

~~~ cddl
RoQDatagramPacketParsed = {
    ~RoQPacket
}
~~~
{: #roq-datagram-packet-parsed-def title="RoQDatagramPacketParsed definition" }

# RoQ Data Field Definitions {#data-fields}

The following data field definitions can be used in RoQ events.

## RoQ Packet {#roq-packet}

~~~ cddl
RoQPacket = {
  flow_id: uint64
  length: uint64
}
~~~
{: #roq-packet-def title="RoQPacket definition"}

## RoQ Application Error {#application-error}

~~~ cddl
RoQApplicationError = "roq_no_error" /
                      "roq_general_error" /
                      "roq_internal_error" /
                      "roq_packet_error" /
                      "roq_stream_creation_error" /
                      "roq_frame_cancelled_error" /
                      "roq_unknown_flow_id_error" /
                      "roq_expectation_unmet_error"
~~~
{: #roq-application-error-def title="RoQApplicationError definition"}

The `RoQApplicationError` extends the general `$ApplicationError` definition in
the qlog QUIC definition, see {{!I-D.draft-ietf-quic-qlog-main-schema}}.

~~~ cddl
$ApplicationError /= RoQApplicationError
~~~

# Security Considerations

The security and privacy considerations discussed in
{{!I-D.draft-ietf-quic-qlog-main-schema}} apply to this document as well.

# IANA Considerations

TODO

--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank Lucas Pardue for their valuable comments and
suggestions contributing to this document.
