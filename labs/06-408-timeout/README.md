# Lab 06 — 408 Request Timeout

## Symptom

A SIP transaction fails with `408 Request Timeout`, or an upstream element reports timeout behavior after receiving no usable response from the next hop.

## Important distinction

A 408 is an outcome, not a complete root cause. Determine **which component generated it and which expected response did not arrive**.

## Failure model

```text
UA-A → Proxy A → Proxy B → UA-B
                 |
                 | INVITE forwarded
                 | ... no qualifying response ...
                 ▼
              timeout
```

Possible causes include unreachable destination, firewall drop, incorrect route, DNS/resolution issue, response-path failure, overloaded downstream system, transport failure, or an application that received the request but failed to respond.

## Investigation

1. Find the 408 and identify its source IP/component.
2. Trace backward to the request/transaction it terminates.
3. Correlate Call-ID, CSeq and Via branch.
4. Determine the last hop where the INVITE is observed.
5. Determine whether any provisional response returned.
6. Inspect retransmission timing for UDP.
7. Check routing/DNS/transport evidence.
8. Capture on both sides of suspected network boundaries.

## Diagnostic fork

```text
Was INVITE transmitted by upstream?
 ├─ No  → routing/application decision before network send
 └─ Yes
     ↓
Was INVITE received downstream?
 ├─ No  → network/path/address/transport investigation
 └─ Yes
     ↓
Did downstream send response?
 ├─ No  → downstream application/resource investigation
 └─ Yes
     ↓
Did response reach upstream?
 ├─ No  → response-path/NAT/firewall/routing investigation
 └─ Yes → transaction/correlation/application behavior
```

## Evidence to record

- component generating 408;
- request source/destination;
- Via branch and CSeq;
- retransmission sequence;
- last known successful hop;
- whether downstream received request;
- whether downstream generated response;
- network/firewall evidence;
- healthy comparison after remediation.

## Production lesson

Aggregate 408 rates are useful, but dimensions matter. Break them down by carrier, destination, route, region, node and downstream dependency so a localized failure is not hidden in platform-wide averages.
