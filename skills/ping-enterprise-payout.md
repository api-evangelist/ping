---
name: Send an enterprise payout (Transfer)
description: Create a Ping++ Transfer to pay out funds to a recipient and confirm via webhook.
api: https://api.pingxx.com/v1
operations:
  - POST /v1/transfers
  - GET /v1/transfers/{id}
  - webhook transfer.succeeded
source: https://docs.pingxx.com/api/
---

# Send an enterprise payout with Ping++

Grounded in the Ping++ API reference (https://docs.pingxx.com/api/). Transfers (企业付款) pay funds out to a recipient (e.g. WeChat enterprise payment, Alipay transfer, bank transfer).

## Auth
HTTP Basic auth with your API Key as username (empty password), over HTTPS.

## Steps
1. **Create the Transfer** — `POST /v1/transfers` with `app[id]`, `channel` (e.g. `wx_pub`, `alipay`), `amount` (分), `currency` (`cny`), a unique `order_no`, `type`, `recipient` and channel-specific `extra` fields (see the appendix 支付渠道 extra 参数说明). Some channels require the WeChat `wx_bank_code` for bank transfers.
2. **Confirm asynchronously** — listen for `transfer.succeeded` / `transfer.failed`. Acknowledge with HTTP 2xx.
3. **Verify** — `GET /v1/transfers/{id}` and check the `status` enum.
4. **Batch variant** — for many payouts use `POST /v1/batch_transfers` and handle `batch_transfer.succeeded` (full/partial) and `batch_transfer.failed`.

## Rules
- **Retry-safe by order_no**: on `transfer_system_busy` retry with the SAME `order_no` — Ping++ guarantees only one transfer is sent.
- A `channel_not_support_transfer` error means the WeChat parameter version must be upgraded.
- Reversing a transfer uses the void/reverse transfer operation (added 2025-04-07). See changelog/ping-changelog.yml.
- Errors follow the standard envelope `{type, message, code?, param?}`.
