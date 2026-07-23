---
name: Send an RTP or Wire payment
description: Validate a counterparty, initiate an instant RTP or high-value wire payment, and confirm its delivery.
api: openapi/keycorp-rtp-wire-payments-openapi.yml
operations: [healthCheck, participant, participantList, Payment-Validate, Payment-Initiate]
---

# Send an RTP or Wire payment

Use the combined KeyBank RTP and Wire Payments service to dispatch high-value or instant payments from a commercial account.

## Preconditions
- OAuth2 bearer token (JWT) + mutual-TLS client certificate.
- `x-fapi-interaction-id` UUID on every request.
- QV sandbox host `https://partner-api-qv.key.com`; production `https://partner-api.key.com`.

## Steps
1. `healthCheck` — verify connectivity.
2. `participant` (or `participantList`) — look up the receiving bank by routing number to confirm RTP/wire reachability before sending.
3. `Payment-Validate` — validate the payment details (amount, accounts, routing) before submission to catch errors early.
4. `Payment-Initiate` — submit the payment. Supply a **`requestReference`** (≤ 32 chars) unique per payment; it provides idempotency so a retry does not double-send.
5. Confirm delivery with the inquiry APIs: `searchRtpTransactions` / `searchRtpTransaction` (openapi/keycorp-rtp-inquiry-openapi.yml) or `searchWireTransactions` / `searchWireTransaction` (openapi/keycorp-wire-inquiry-openapi.yml).

## Rules
- RTP payments are **instant and irrevocable** — always `Payment-Validate` and confirm the participant first.
- Reuse the same `requestReference` when retrying a timed-out `Payment-Initiate`; never mint a new one for the same intended payment.
- Handle `429`/`503` with backoff + `Retry-After`.
