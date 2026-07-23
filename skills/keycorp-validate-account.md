---
name: Validate an account before payment
description: Verify account details and ownership against the National Shared Database Resource before originating a payment.
api: openapi/keycorp-account-validation-openapi.yml
operations: [healthCheck, verifyAccount]
---

# Validate an account before payment

Use the KeyBank Account Validation v2 API to reduce fraud and return risk before you originate an ACH, wire, or RTP payment.

## Preconditions
- OAuth2 bearer token (JWT) + mutual-TLS client certificate.
- A KeyBank-issued `secondaryId` scoping you to your client relationship.
- `x-fapi-interaction-id` UUID on every request.

## Steps
1. `healthCheck` — confirm connectivity and credentials.
2. `verifyAccount` — submit the account and owner details. The API matches them against the National Shared Database Resource (the industry's collaborative account-status source) and returns account status and ownership match.
3. Gate your payment origination (ACH / RTP / Wire skills) on a passing validation result.

## Rules
- Always send the `secondaryId`; requests without it are rejected.
- Treat validation as a pre-flight step — do not originate to an account that fails status/ownership checks.
- Errors return the custom `exception` envelope; log `X-CorrelationId`.
