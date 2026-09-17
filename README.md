# SIP Troubleshooting Lab

> Hands-on SIP, SDP, and RTP troubleshooting scenarios for engineers operating real-time voice platforms.

## Purpose

SIP problems rarely stop at a response code. A `408`, one-way audio, repeated INVITEs, or a failed transfer is the observable symptom of behavior across endpoints, proxies, SBCs, media servers, NAT/firewalls, DNS, SDP negotiation, and RTP.

This repository builds a systematic troubleshooting discipline around **evidence from the wire**.

Each lab follows the same model:

**Topology → Symptom → Evidence → SIP/SDP/RTP Analysis → Root Cause → Fix → Production Lesson**

## Troubleshooting workflow

1. Establish the expected call flow and component ownership.
2. Identify the failing call leg and transaction/dialog.
3. Correlate Call-ID, tags, branches, CSeq and route headers.
4. Inspect SIP response timing and retransmissions.
5. Validate SDP offer/answer and codec negotiation.
6. Validate RTP addresses, ports, payload types and packet direction.
7. Separate signaling failure from media failure.
8. Form a hypothesis and prove or disprove it with packet/log evidence.
9. Correct the fault and capture the healthy comparison.

## Lab catalog

| # | Scenario | Primary layer |
|---:|---|---|
| 01 | Baseline successful SIP call | SIP / RTP |
| 02 | Registration and authentication | SIP |
| 03 | 401 vs 407 authentication challenge | SIP |
| 04 | 403 Forbidden | SIP / policy |
| 05 | 404 User Not Found | SIP / routing |
| 06 | 408 Request Timeout | SIP / network |
| 07 | 480 Temporarily Unavailable | SIP / endpoint state |
| 08 | 486 Busy Here | SIP / endpoint state |
| 09 | 503 Service Unavailable | SIP / infrastructure |
| 10 | INVITE retransmissions | SIP / transport |
| 11 | Missing ACK | SIP / transaction/dialog |
| 12 | NAT / Contact routing failure | SIP / NAT |
| 13 | One-way audio | RTP / NAT / firewall |
| 14 | No audio | RTP |
| 15 | Codec mismatch | SDP / RTP |
| 16 | Incorrect SDP connection address | SDP / NAT |
| 17 | RTP port blocked | RTP / firewall |
| 18 | Session timer failure | SIP dialog |
| 19 | SIP proxy failure | infrastructure |
| 20 | Media server failure | infrastructure / RTP |

## Repository structure

```text
.
├── README.md
├── docs/
│   ├── troubleshooting-methodology.md
│   ├── sip-transaction-primer.md
│   └── rtp-analysis-primer.md
├── labs/
│   ├── 01-baseline-call/
│   ├── 02-registration-auth/
│   └── ...
├── diagrams/
├── captures/
└── tools/
```

Packet captures will be added only when they are generated from controlled lab traffic and reviewed for credentials, phone numbers, IP information, audio, and other sensitive content.

## Tools

The labs are designed around tools commonly used in production voice troubleshooting:

- Wireshark
- sngrep
- tcpdump / tshark
- Kamailio logging/tracing
- FreeSWITCH CLI/logs
- SIPp for controlled traffic generation in later labs

## Scope

This is an engineering lab and reference, not a collection of memorized SIP response codes. The objective is to explain **why** the protocol behaves as observed and how to isolate faults efficiently in multi-component voice systems.

## Status

🚧 Initial lab framework under development.
