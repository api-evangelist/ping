---
name: Accept a payment (create and confirm a Charge)
description: Create a Ping++ Charge, drive the client through the channel, and confirm success via webhook.
api: https://api.pingxx.com/v1
operations:
  - POST /v1/charges
  - GET /v1/charges/{id}
  - webhook charge.succeeded
source: https://docs.pingxx.com/api/
---

# Accept a payment with Ping++

Grounded in the Ping++ API reference (https://docs.pingxx.com/api/). Ping++ has no OpenAPI spec; the operations below are the documented REST endpoints.

## Auth
- HTTP Basic auth: API Key as the username, empty password. Use a `sk_test_` key while building; switch to `sk_live_` for production. HTTPS is required.
- Example: `curl https://api.pingxx.com/v1/charges -u sk_test_XXXX:`

## Steps
1. **Create the Charge** — `POST /v1/charges` with form fields: `order_no` (unique merchant order number), `amount` (in the smallest unit / 分), `app[id]` (your app id, `app_...`), `channel` (e.g. `wx`, `alipay`, `upacp`), `currency` (`cny`), `client_ip`, `subject`, `body`. Pass business context in `metadata[<key>]` (max 20 pairs, ≤1000 chars).
2. **Hand the response to the client SDK** — the returned Charge contains the channel `credential` the iOS/Android/JS SDK uses to launch the WeChat/Alipay/UnionPay flow.
3. **Confirm asynchronously** — listen for the `charge.succeeded` webhook (POST JSON Event). Acknowledge with HTTP 2xx; a non-2xx makes Ping++ retry (up to 10 times over ~25h). Do not rely solely on the client returning.
4. **Verify** — optionally `GET /v1/charges/{id}` and check `paid: true` before fulfilling. Treat the webhook or this query as the source of truth, never the client callback alone.

## Rules
- **No duplicate charges**: reusing an `order_no` returns error code `charge_order_no_used`. Generate a fresh `order_no` per attempt; this order-number uniqueness is the dedup mechanism (Ping++ has no Idempotency-Key header).
- **Errors**: inspect `error.type` (`invalid_request_error` / `api_error` / `channel_error` / `card_error`) and `error.code`. See errors/ping-error-codes.yml.
- **Rate/plan**: HTTP 403 means the plan concurrency limit was hit — throttle or upgrade.
- **Test webhooks**: set `metadata[pingpp_tc]=001` (duplicate) or `002` (5-min delay) in test mode to harden your receiver.
