# Lab 03 — 401 vs 407 Authentication Challenges

## Objective

Understand where SIP authentication challenges originate, which header carries credentials, and why a challenge is often normal protocol behavior rather than a call failure.

## Core distinction

| Response | Challenger | Credential header on retry |
|---|---|---|
| `401 Unauthorized` | UAS / registrar | `Authorization` |
| `407 Proxy Authentication Required` | SIP proxy | `Proxy-Authorization` |

Both commonly use digest authentication. The endpoint receives a challenge containing parameters such as realm and nonce, calculates a digest response, increments CSeq as required for the retried request, and resubmits with credentials.

## Healthy REGISTER example

```text
UA                         Registrar
 | REGISTER                    |
 |---------------------------->|
 | 401 + WWW-Authenticate      |
 |<----------------------------|
 | REGISTER + Authorization    |
 |---------------------------->|
 | 200 OK                      |
 |<----------------------------|
```

A trace containing a 401 is therefore not sufficient evidence of an authentication problem.

## Failure hypotheses

If authentication never succeeds, inspect:

- username/authentication identity;
- realm;
- password/secret configuration;
- nonce freshness;
- algorithm compatibility;
- stale credential caching;
- incorrect `Authorization` vs `Proxy-Authorization` behavior;
- intermediate devices modifying relevant headers;
- repeated challenge loops.

## Packet-analysis checklist

Compare the first request and authenticated retry:

- Call-ID;
- CSeq;
- From/To;
- challenge realm;
- nonce;
- URI used in digest calculation;
- presence of `Authorization` or `Proxy-Authorization`;
- final response.

Do not publish traces containing reusable credentials or secrets. Digest fields should still be treated as sensitive operational material.

## Root-cause precision

Weak:

> SIP authentication failed.

Better:

> The registrar challenged the REGISTER with realm `lab.example`. The UA retried with an Authorization header using a different configured realm, so the registrar rejected the digest and issued another challenge. Correcting the account realm allowed the authenticated REGISTER to receive 200 OK.

## Production lesson

Authentication metrics should distinguish expected challenge/response behavior from terminal authentication failures. Alerting on every 401 would create noise and obscure genuine repeated-challenge loops.
