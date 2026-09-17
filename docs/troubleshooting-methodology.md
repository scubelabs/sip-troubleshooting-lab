# SIP/RTP Troubleshooting Methodology

## 1. Start with the symptom, not the assumption

Classify what is actually failing:

- call setup
- registration
- authentication
- routing
- ringing/early media
- answer
- in-dialog signaling
- media
- teardown

Do not diagnose an RTP problem from a SIP response code or a SIP routing problem from an audio symptom without evidence.

## 2. Draw the expected topology

Example:

```text
UA-A → SIP Proxy/SBC → Media/Application Server → SIP Proxy/SBC → UA-B
```

For each hop identify whether the component is:

- stateless/stateful SIP proxy
- B2BUA
- registrar/location service
- media anchor/relay
- endpoint
- NAT/firewall boundary

This determines which headers, Call-IDs, dialogs, and RTP endpoints should change.

## 3. Identify the transaction/dialog

Correlate at minimum:

- Call-ID
- From tag
- To tag
- CSeq
- Via branch
- Request-URI
- Contact
- Route / Record-Route

A B2BUA can create separate dialogs, so one end-to-end business call may contain multiple SIP Call-IDs.

## 4. Establish the SIP ladder

Determine:

1. Who generated the request?
2. Which hop received it?
3. Which responses were returned?
4. Where did forwarding stop?
5. Were retransmissions generated?
6. Did ACK arrive for the final response where required?
7. Did route-set behavior remain correct for in-dialog requests?

## 5. Analyze SDP independently

For every offer/answer inspect:

- `c=` connection address
- `m=` media port
- codec payload types
- `a=rtpmap`
- direction attributes (`sendrecv`, `sendonly`, etc.)
- ICE candidates when applicable
- DTLS/SRTP attributes when applicable

A successful SIP dialog does not prove a viable media path.

## 6. Prove the RTP path

For each media direction answer:

- What IP/port did SDP advertise?
- What IP/port actually sent packets?
- Where were packets received?
- Are sequence numbers increasing?
- Are timestamps plausible?
- Is the payload type negotiated?
- Is loss/jitter significant?

For one-way audio, explicitly create two hypotheses: **A→B failed** and **B→A failed**. Prove which direction is absent before changing configuration.

## 7. Correlate protocol evidence with logs

Packet capture establishes what crossed an interface. Application logs establish internal decisions. Neither alone necessarily explains the complete failure.

Correlate using timestamps, Call-ID/dialog identifiers, endpoint identifiers, transaction branches, and application correlation IDs where available.

## 8. State the root cause precisely

Weak root cause:

> Network issue.

Useful root cause:

> The callee SDP advertised a private RFC1918 media address that was unreachable from the media peer; SIP signaling completed successfully, but RTP sent toward the advertised address never reached the callee.

## 9. Verify the fix

A troubleshooting exercise is incomplete until the corrected behavior is captured and compared with the failure case.

## 10. Extract the production lesson

Every lab ends by asking what would detect or prevent the same fault in production: health checks, synthetic calls, SIP response metrics, RTP quality telemetry, configuration validation, alerting, topology controls, or automated failover.
