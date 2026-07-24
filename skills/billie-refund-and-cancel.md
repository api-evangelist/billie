---
name: Cancel an order or refund a capture
description: Cancel an uncaptured order, refund a capture in full, or issue a partial credit note.
api: https://docs.billie.io/reference
operations: [oauth_token_create, order_cancel_v2, invoice_cancel_v2, create_credit_note, get_invoice]
generated: '2026-07-18'
method: generated
---

# Cancel an order or refund a capture

## Auth
Send `Authorization: Bearer <token>` (client_credentials JWT) on every call.

## Choose the operation
- **Cancel a whole order before capture** — `order_cancel_v2` with the order
  `id`. Returns `HTTP 204`. For cancellations after capture, use a refund
  instead.
- **Full refund of a capture** — `invoice_cancel_v2` with the `captureId`
  refunds the full captured amount (`HTTP 204`).
- **Partial or full credit note** — `create_credit_note` with the `captureId`
  and the `amount` to refund (`HTTP 201`). The amount must not exceed the
  capture amount.
- **Inspect a capture first** — `get_invoice` with the `captureId`.

## Rules
- Refund amounts can never exceed the referenced capture amount.
- Order state transitions and decline reasons are documented in
  `errors/billie-decline-codes.yml`; error/response semantics in
  `errors/billie-problem-types.yml`.
