# Lab 15 — Codec / SDP Negotiation Failure

## Objective

Diagnose failures where endpoints cannot agree on a usable audio format or where RTP payload interpretation does not match the negotiated SDP.

## SDP example

```text
m=audio 24000 RTP/AVP 0 8 101
a=rtpmap:0 PCMU/8000
a=rtpmap:8 PCMA/8000
a=rtpmap:101 telephone-event/8000
```

The `m=` line advertises payload types; `a=rtpmap` defines mappings, especially for dynamic payload types.

## Failure categories

- no common codec between offer and answer;
- codec disabled on one call leg;
- transcoding required but unavailable/disabled;
- incorrect dynamic payload mapping;
- SDP manipulated incorrectly by an intermediary;
- RTP sent using a payload type inconsistent with negotiated SDP;
- DTMF telephone-event confused with audio codec handling.

## Investigation

1. Extract every SDP offer/answer across each B2BUA leg.
2. List codecs and payload mappings per leg.
3. Identify the selected/common codec.
4. Determine whether a B2BUA/media server must transcode between legs.
5. Inspect actual RTP payload type.
6. Verify media-server codec modules/configuration where relevant.
7. Compare with a known-good call using the same route/endpoints.

## B2BUA example

```text
Carrier leg             Media server             Agent leg
PCMU only       <---->  transcode?      <---->   Opus only
```

A media server can make two otherwise incompatible legs interoperable if the required codecs/transcoding capability are available. This consumes additional resources and should be reflected in capacity planning.

## Root-cause quality

Weak:

> Codec issue.

Precise:

> The carrier offered PCMU only while the agent leg was constrained to Opus. The media server configuration did not provide the required transcoding path, so no compatible end-to-end media format could be established.

## Production lesson

Track codec distribution and transcoding utilization. Codec policy changes can alter CPU capacity requirements even when call concurrency remains unchanged.
