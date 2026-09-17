# Lab 13 — One-Way Audio

## Symptom

SIP call setup succeeds and both endpoints show an established call, but only one participant can hear the other.

## Key principle

**One-way audio is directional.** Do not start with "RTP is broken." Determine exactly which media direction is missing.

```text
A ---------------- RTP ----------------> B   ✓
A <--------------- RTP ----------------- B   ✗
```

The failure above is specifically **B → A media delivery**.

## Candidate causes

Possible causes include:

- private/unreachable IP advertised in SDP;
- NAT mapping mismatch;
- firewall/ACL blocking one UDP direction or port range;
- media relay not inserted where required;
- asymmetric routing interacting with stateful firewall policy;
- endpoint sends RTP to a different address/port than negotiated;
- incorrect media-server external/internal address configuration;
- SRTP/keying mismatch in secured deployments.

The existence of these candidates is not evidence that any one is the root cause.

## Investigation procedure

### 1. Verify signaling success

Confirm INVITE/200/ACK completes. Record both SDP offer and answer.

### 2. Build the expected media matrix

| Direction | Sender | Expected destination from SDP | Packets observed? |
|---|---|---|---|
| A → B | A | B media IP:port | TBD |
| B → A | B | A media IP:port | TBD |

### 3. Inspect SDP

For each leg identify:

```text
c=IN IP4 <media-address>
m=audio <port> RTP/AVP ...
```

Then compare the advertised address/port with the actual network topology.

### 4. Inspect packet capture

Do not merely use Wireshark's RTP player. Determine whether packets physically cross each capture point.

Useful evidence includes:

- UDP 5-tuple
- RTP SSRC
- sequence progression
- payload type
- packet count
- first/last packet timestamp

### 5. Localize the loss

With captures at multiple boundaries:

```text
Endpoint B → Firewall → Media Relay/SBC → Firewall → Endpoint A
             ✓              ✓              ✗
```

The first point where expected packets disappear sharply reduces the search space.

## Example root-cause statement

> SIP signaling completed successfully. Endpoint A advertised `10.x.x.x` as its media connection address in SDP. Endpoint B transmitted RTP toward that private address, which was not routable from B's network. B→A RTP therefore never reached A, producing one-way audio.

This is materially better than "NAT issue."

## Remediation categories

Depending on the proven cause, remediation may involve SDP/media-address correction, NAT traversal, media anchoring, firewall policy, RTP port configuration, or endpoint configuration.

## Production detection

Useful production controls can include:

- RTP packet counters per direction;
- no-RTP / media-timeout events;
- synthetic voice calls;
- media quality telemetry;
- SDP sanity validation;
- alarms for calls established without bidirectional media.

## Lab completion criteria

This lab is complete only when it contains a controlled failure capture and a corrected capture demonstrating restored bidirectional RTP.
