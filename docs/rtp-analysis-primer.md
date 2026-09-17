# RTP Analysis Primer

## SIP success does not prove media success

SIP/SDP negotiates where and how media should flow. RTP is the actual media transport. A `200 OK` and ACK can coexist with silence, one-way audio, wrong codec payloads, severe loss, or media sent to an unreachable address.

## Start with SDP

For each media leg record:

```text
c=IN IP4 <connection-address>
m=audio <port> RTP/AVP <payload-types>
a=rtpmap:<pt> <codec>/<clock-rate>
```

Build an expected matrix before inspecting packets.

| Direction | Expected source | Expected destination | Codec/PT | Observed? |
|---|---|---|---|---|
| A → B | A media socket | B SDP target | negotiated | TBD |
| B → A | B media socket | A SDP target | negotiated | TBD |

## RTP fields

Important RTP header fields include payload type, sequence number, timestamp and SSRC.

### Sequence number

Normally increments per RTP packet and is useful for identifying loss, reordering and stream continuity.

### Timestamp

Represents sampling time according to the media clock. It is not wall-clock time.

### SSRC

Identifies a synchronization source within an RTP session. SSRC changes can help identify stream/source changes but should be interpreted in context.

### Payload type

Maps RTP packets to the negotiated media format. Dynamic payload types derive meaning from SDP rather than from the number alone.

## Troubleshooting patterns

### No RTP in either direction

Investigate whether media was negotiated, whether endpoints actually started transmission, media addresses/ports, firewall/NAT behavior, and whether the capture point is on the media path.

### RTP only one direction

Treat each direction independently. Compare SDP target with actual destination and move capture points across boundaries until the loss is localized.

### RTP exists but no intelligible audio

Investigate payload/codec interpretation, transcoding, SRTP expectations, DTMF vs audio payloads, malformed media, or endpoint playback/device issues. Packet presence alone does not prove usable audio.

### Quality degradation

Inspect loss, reordering, jitter, latency and burst-loss patterns. Correlate with network-interface, host, virtualization and media-server telemetry.

## Capture-point discipline

A packet absent from one capture does not prove it was never sent. State exactly where the capture was taken.

```text
Endpoint → LAN → Firewall → WAN → SBC/Media
    C1              C2              C3
```

Comparing C1/C2/C3 can distinguish endpoint generation, firewall traversal and remote receipt.

## RTCP

Where present, RTCP can provide reception and quality information useful for diagnosing packet loss/jitter and stream health. Do not assume RTCP exists in every deployment.

## Encrypted media

With SRTP, packet headers still provide some transport-level evidence, but payload inspection requires appropriate authorized keying context. Never weaken production encryption merely to simplify troubleshooting.

## Operational rule

For every audio incident, be able to state: **what SDP requested, what packets actually did, at which capture point, and in which direction**.
