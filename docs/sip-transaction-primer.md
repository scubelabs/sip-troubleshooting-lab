# SIP Transaction & Dialog Primer

## Why this matters for troubleshooting

A SIP trace becomes much easier to reason about when three concepts are separated: **message**, **transaction**, and **dialog**.

## Message

A SIP message is an individual request or response: `INVITE`, `ACK`, `BYE`, `REGISTER`, `OPTIONS`, `180 Ringing`, `200 OK`, and so on.

## Transaction

A transaction groups a request with the responses associated with that request. The top Via `branch` parameter is a key correlation value for RFC 3261 transactions.

Example INVITE transaction:

```text
INVITE  ---------------------->
        <---------------------- 100 Trying
        <---------------------- 180 Ringing
        <---------------------- 200 OK
ACK     ---------------------->
```

The ACK handling differs depending on whether it acknowledges a 2xx or non-2xx final response, which is important when diagnosing retransmissions.

## Dialog

A dialog represents a peer-to-peer SIP relationship that can persist across multiple transactions. A dialog is identified by the combination of:

- Call-ID
- local tag
- remote tag

An established call can therefore contain an initial INVITE transaction followed by re-INVITE/UPDATE, INFO, REFER, and BYE transactions within the same dialog.

## B2BUA implication

A Back-to-Back User Agent terminates one SIP dialog and originates another. FreeSWITCH commonly operates this way.

```text
UA-A        FreeSWITCH          UA-B
 | Dialog A     |                |
 |<------------>|                |
 |              |    Dialog B    |
 |              |<-------------->|
```

Do not assume that one business interaction has one Call-ID across every component. Correlation may require timestamps, custom headers, application UUIDs, or platform-specific identifiers.

## Headers to inspect

### Via
Tracks the SIP response path through proxies and contains the transaction branch identifier.

### Record-Route / Route
A proxy that must remain on the signaling path can Record-Route the initial dialog-forming request. Subsequent in-dialog requests use the resulting route set.

### Contact
Identifies where a SIP user agent wants subsequent requests for the dialog or registration to be sent. NAT and incorrect Contact rewriting are frequent sources of routing failures.

### CSeq
Combines a sequence number and method. It helps identify ordering and associate requests/responses within a dialog.

### Max-Forwards
Protects against routing loops. Each forwarding proxy decrements the value.

## Retransmission clue

Repeated identical UDP INVITEs are not automatically evidence that the caller intentionally placed multiple calls. They may be SIP transaction retransmissions caused by a missing response or an unreachable response path.

Always compare:

- Call-ID
- CSeq
- Via branch
- source/destination
- timing

before concluding that duplicate calls were generated.
