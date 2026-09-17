# Lab 17 — RTP Port Blocked by Firewall

## Symptom

SIP signaling completes successfully but media is absent in one or both directions. SDP advertises plausible addresses and ports, yet packet capture shows RTP disappearing at a network boundary.

## Failure topology

```text
Endpoint A → Firewall A → Media Server → Firewall B → Endpoint B
   RTP ✓        RTP ✓          RTP ✓        RTP X
```

## Investigation

Build a media tuple from SDP for each direction:

```text
source IP : source UDP port
        → destination IP : destination UDP port
payload / codec
```

Then capture as close as practical to both sides of the suspected boundary.

### Evidence required

A strong firewall diagnosis demonstrates:

- RTP leaving the sender;
- expected destination derived from SDP;
- packets arriving on one side of the firewall;
- packets absent on the other side;
- firewall/ACL/session logs consistent with the drop where available.

"No audio, therefore firewall" is not a diagnosis.

## Dynamic media ranges

Voice platforms commonly allocate RTP from configured UDP ranges. Operational failures occur when the application/media range and network-security policy diverge—for example after a configuration change, scale-out, or migration.

Document for each media component:

- configured RTP range;
- advertised address strategy;
- ingress/egress policy;
- whether media is direct or anchored;
- stateful firewall/NAT behavior.

## Production lesson

Configuration-as-code validation can compare expected media ranges against firewall/security policy before deployment. Synthetic calls and directional RTP counters can detect failures that SIP-only health checks miss.
