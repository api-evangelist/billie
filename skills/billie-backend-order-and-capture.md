---
name: Create a backend order and capture it
description: Authorize a B2B buyer with Billie from your backend, then capture (invoice) the shipment.
api: https://docs.billie.io/reference
operations: [oauth_token_create, order_create_v2, order_get_details_v2, invoice_create, invoice_payment_confirm]
generated: '2026-07-18'
method: generated
---

# Create a backend order and capture it

Use this flow when buyer consent is collected outside the Billie widget and you
create the order server-to-server.

## Auth
1. POST to `oauth/token` at `https://paella.billie.io/api/v2/oauth/token`
   (sandbox: `https://paella-sandbox.billie.io/api/v2/oauth/token`) with
   `grant_type=client_credentials`, `client_id`, `client_secret`.
2. Use the returned JWT as `Authorization: Bearer <token>` on every call. Tokens
   expire after 8 hours; on `401` fetch a new token and retry.

## Steps
1. **Create the order** — `order_create_v2`. Body requires `amount`
   (net/gross/tax), `duration` (1-120 days), `debtor` (company + registered
   address), `debtor_person` (email), and `line_items`. A success returns
   `state=created` and the order `uuid`. If financing is refused the order is
   `declined` with a `decline_reason` (see `errors/billie-decline-codes.yml`).
2. **(Optional) Read it back** — `order_get_details_v2` with the order `uuid`.
3. **Capture on shipment** — `invoice_create` against the order `uuid`. Provide
   the invoice number as `external_code` and a reachable `invoice_url`. Partial
   captures are separate requests until the full amount is reached; the order
   moves to `partially_shipped` then `shipped`.
4. **Reconcile direct payments** — if the buyer pays you instead of Billie, call
   `invoice_payment_confirm` with the `captureId` and amount so Billie reduces
   the outstanding balance.

## Rules
- De-duplicate orders with your own `external_code`; Billie rejects a changed
  `external_code` on an existing order (no Idempotency-Key header exists).
- Order amounts may only be lowered after creation, never raised (a raise is a
  new authorization). See `conventions/billie-conventions.yml`.
