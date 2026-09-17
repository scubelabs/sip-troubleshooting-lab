# Lab 11 — Missing ACK and Repeated 200 OK

## Symptom

The callee or B2BUA sends `200 OK` to an INVITE repeatedly after the call appears to have been answered. The call may eventually clear because the successful INVITE transaction/dialog establishment was not acknowledged correctly.

## Expected sequence

```text
Caller                     Callee
  | INVITE                    |
  |-------------------------->|
  | 180 Ringing               |
  |<--------------------------|
  | 200 OK                    |
  |<--------------------------|
  | ACK                       |
  |-------------------------->|
```

For a 2xx response to INVITE, ACK is generated as a separate request associated with the established dialog. If the UAS does not receive it, the 2xx can be retransmitted over unreliable transport.

## Investigation

Determine whether:

1. the caller generated ACK;
2. ACK left the caller-side network;
3. route-set/Request-URI was correct;
4. an intermediate proxy forwarded ACK;
5. ACK reached the device retransmitting the 200 OK.

## Evidence pattern

```text
INVITE  ---------------------->
        <---------------------- 200 OK
ACK     ----------X
        <---------------------- 200 OK
        <---------------------- 200 OK
        <---------------------- 200 OK
```

Repeated 200 responses are evidence that the responding UAS has not observed the expected ACK; they do not by themselves identify where ACK was lost.

## Headers to correlate

- Call-ID;
- From/To tags;
- CSeq method/number;
- Contact from the 200 OK;
- Record-Route and resulting Route set;
- ACK Request-URI;
- source/destination addresses.

## Common fault categories

- incorrect Contact address;
- broken Record-Route/Route handling;
- NAT/firewall path issue;
- proxy not forwarding ACK correctly;
- endpoint implementation/configuration error;
- B2BUA leg correlation/configuration problem.

## Root-cause example

> The caller generated ACK, but it targeted the private Contact URI supplied in the 200 OK. The address was unreachable from the caller network, so the UAS never received ACK and retransmitted its 200 response until timeout.

## Production lesson

A high rate of repeated INVITE 2xx responses can be a valuable signal for dialog-establishment path problems. Troubleshooting must locate the ACK loss hop rather than suppress the retransmission symptom.
