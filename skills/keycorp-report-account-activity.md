---
name: Report on commercial account activity
description: Retrieve commercial account details, balances, and prior/current-day transactions for reconciliation.
api: openapi/keycorp-commercial-accounts-reporting-openapi.yml
operations: [searchForAccounts, getAccount, searchForPreviousDayAccountTransactionsV1, searchForCurrentDayAccountTransactionsV1]
---

# Report on commercial account activity

Use the KeyBank Commercial Accounts Reporting API to pull account-level information for client-authorized commercial accounts.

## Preconditions
- OAuth2 bearer token (JWT) + mutual-TLS client certificate.
- `x-fapi-interaction-id` UUID on every request.

## Steps
1. `searchForAccounts` — list the payment accounts the caller can access.
2. `getAccount` — retrieve full details and balances for a specific account.
3. `searchForPreviousDayAccountTransactionsV1` — query posted prior-day activity (up to 180 days of history).
4. `searchForCurrentDayAccountTransactionsV1` — query intraday activity.

## Rules
- List/search operations page with an opaque `pageKey` cursor; follow `nextPageKey` and keep `pageSize` ≤ 1000 (see conventions/keycorp-conventions.yml).
- These are read operations — safe to retry on transient `5xx` with backoff.
