---
name: Refund a charge and reconcile
description: Issue a full or partial refund against a Ping++ Charge and confirm it via webhook.
api: https://api.pingxx.com/v1
operations:
  - POST /v1/charges/{id}/refunds
  - GET /v1/charges/{id}/refunds/{refund_id}
  - webhook refund.succeeded
source: https://docs.pingxx.com/api/
---

# Refund a Ping++ charge

Grounded in the Ping++ API reference (https://docs.pingxx.com/api/).

## Auth
HTTP Basic auth with your API Key as username (empty password), over HTTPS. Use `sk_test_` in test mode.

## Steps
1. **Create the Refund** — `POST /v1/charges/{charge_id}/refunds` with `amount` (omit for a full refund; provide a value in 分 for partial) and an optional `description`. Attach `metadata[<key>]` if you need reconciliation keys.
2. **Confirm asynchronously** — listen for the `refund.succeeded` webhook (or `refund.failed`). Acknowledge with HTTP 2xx.
3. **Verify** — `GET /v1/charges/{charge_id}/refunds/{refund_id}` and check `succeed: true`.

## Rules
- A refund cannot exceed the charge amount, and total refunds across attempts cannot exceed it.
- **Channel refund states** to handle (from errors/ping-error-codes.yml): `refund_wait_operation`, `refund_pending`, `refund_retry`, `refund_refused`, `refund_manual_intervention`. On `refund_pending` re-query later rather than re-issuing.
- A charge that is already closed returns `charge_closed`.
- Errors follow the standard envelope `{type, message, code?, param?}`.
