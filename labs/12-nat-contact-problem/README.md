# Lab 12 — NAT and Contact Routing Failure

## Objective

Diagnose signaling failures caused by SIP addresses that describe an endpoint's local network view rather than a reachable address from the signaling peer.

## Example topology

```text
SIP UA (10.0.0.25)
       |
       | NAT
       v
Public network
       |
       v
SIP Proxy / Peer
```

A SIP endpoint can transmit from a translated public source while placing a private address in Via, Contact, SDP, or other signaling fields. SIP-aware infrastructure must distinguish packet-source information from protocol-advertised routing information.

## Example symptom

Registration succeeds because REGISTER reaches the registrar, but a subsequent inbound INVITE is sent toward an unreachable private Contact address.

```text
REGISTER source: 203.0.113.x:62000
Contact: <sip:1001@10.0.0.25:5060>
```

The exact correction depends on architecture; blindly replacing every private address is not a safe general rule.

## Investigation matrix

| Item | Advertised value | Observed packet source | Reachable from peer? |
|---|---|---|---|
| Via sent-by | TBD | n/a | TBD |
| Contact | TBD | n/a | TBD |
| SIP source | n/a | TBD | TBD |
| SDP `c=` | TBD | n/a | TBD |
| RTP source | n/a | TBD | TBD |

## Distinguish signaling NAT from media NAT

A call can have:

- working SIP + broken RTP;
- broken SIP + no opportunity for RTP;
- both signaling and media NAT problems.

Do not treat Contact rewriting and SDP/media traversal as the same problem.

## Production design considerations

Depending on topology, mechanisms can include received-source tracking, registrar NAT flags, connection reuse, outbound-aware endpoint behavior, SBC topology/NAT handling, media anchoring, ICE for WebRTC-capable endpoints, and explicit external-address configuration.

## Completion criteria

A future executable version of this lab should include a controlled NAT topology, failure capture, corrected capture, and explanation of exactly which routing datum changed.
