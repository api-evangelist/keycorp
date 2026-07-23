---
name: Originate and track an ACH payment
description: Submit an ACH credit or debit from a commercial KeyBank account, then confirm and, if needed, cancel it.
api: openapi/keycorp-ach-originations-openapi.yml
operations: [healthCheck, achPaymentCCD, achPaymentPPD, achPaymentCTX, achPaymentTEL, achPaymentWEB, achPaymentStatus, achPaymentAddendasStatus, achPaymentUndo]
---

# Originate and track an ACH payment

Use the KeyBank ACH Origination API to move funds from a commercial account.

## Preconditions
- OAuth2 bearer token (JWT) in `Authorization: Bearer {token}`.
- KeyBank-issued mutual-TLS client certificate on the connection.
- A fresh `x-fapi-interaction-id` UUID header on every request.
- Test against `https://partner-api-qv.key.com`; go live on `https://partner-api.key.com`.

## Steps
1. `healthCheck` — confirm connectivity and that the token is accepted.
2. Choose the Standard Entry Class for the transaction and call the matching operation:
   - `achPaymentCCD` — corporate credit/debit (business-to-business).
   - `achPaymentPPD` — consumer deposits/withdrawals.
   - `achPaymentCTX` — corporate trade exchange with remittance addenda.
   - `achPaymentTEL` — telephone-authorized payment.
   - `achPaymentWEB` — internet-authorized payment.
   Stamp a **unique `uuid`** on every batch and detail record — KeyBank dedupes on it, so retries are safe and idempotent.
3. `achPaymentStatus` — poll by `uuid` to see whether each item was accepted, failed, or is awaiting addenda; read `traceNumber` for the settled item.
4. `achPaymentAddendasStatus` — if the payment carries addenda, confirm all addenda were received.
5. `achPaymentUndo` — cancel a request by `uuid` before it settles if it must be reversed.

## Rules
- Idempotency is carried in-body via `uuid`, not an `Idempotency-Key` header. Reuse the same `uuid` when retrying a failed submit.
- On `429`/`503`, back off with jitter and honor `Retry-After`; do not resubmit with a new `uuid`.
- Errors arrive as the custom `exception` envelope (`ErrorMessage`, `TransactionId`, `X-CorrelationId`); log `X-CorrelationId` for support (see errors/keycorp-error-codes.yml).
