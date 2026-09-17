# Lab 10 — INVITE Retransmissions

## Symptom

A packet trace contains repeated INVITEs and may appear at first glance to show duplicate call attempts.

## Core concept

With unreliable transport such as UDP, SIP transactions use retransmission timers. Repeated INVITEs with the same transaction identity can therefore indicate that the sender has not received the response it expects—not that the application intentionally originated multiple calls.

## What to compare

For repeated messages compare:

- Call-ID;
- CSeq number and method;
- top Via branch;
- From tag;
- Request-URI;
- source/destination;
- message body;
- timing between transmissions.

## Typical diagnostic patterns

### Request never reaches downstream

```text
UAC → INVITE → network drop
UAC → INVITE → network drop
UAC → INVITE → network drop
```

### Downstream responds but response cannot return

```text
UAC → INVITE → UAS
               UAS → response → X
UAC → INVITE → UAS
```

The packet pattern at the caller can look similar, but a capture at the UAS distinguishes the cases.

### Application-level duplicate origination

Two genuinely separate attempts may have different Call-IDs/branches or other transaction identifiers. Do not classify them as SIP retransmissions solely because they occur close together.

## Troubleshooting procedure

1. Group messages by transaction identity.
2. Plot timestamps and retransmission intervals.
3. Find the expected response.
4. Capture at the next hop.
5. Determine whether request delivery or response return is failing.
6. Check NAT/firewall state and Via/response routing.
7. Compare against a successful transaction.

## Operational impact

Retransmissions consume signaling capacity and can amplify an outage. A degraded downstream system can create more upstream signaling work precisely when the platform has the least spare capacity.

Monitor retransmission rates alongside transaction latency, timeout rates, CPU, queueing and downstream health.

## Production lesson

"Duplicate INVITEs" is a symptom description. A defensible diagnosis identifies whether the duplicates are protocol retransmissions, application retries, forked branches, or genuinely independent calls.
