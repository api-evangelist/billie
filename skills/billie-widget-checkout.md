---
name: Accept a Billie payment via the checkout widget
description: Initialize the Billie checkout widget, confirm the authorization, and capture the order.
api: https://docs.billie.io/reference
operations: [oauth_token_create, create_checkout_session_v2, get_checkout_session_authorization_v2, confirm_authorization_v2, invoice_create]
generated: '2026-07-18'
method: generated
---

# Accept a Billie payment via the checkout widget

Use this flow to collect buyer consent in your storefront checkout with Billie's
embedded widget (see `components/billie-components.yml`).

## Auth
Obtain a Bearer JWT from `oauth/token` (client_credentials) and send it as
`Authorization: Bearer <token>` on every server call.

## Steps
1. **Create a session** — `create_checkout_session_v2` returns a session `id`.
   Sessions expire after 48 hours.
2. **Render the widget** — initialize the Billie widget front-end with the
   session `id`; the buyer supplies company details and consents.
3. **(Optional) Read the authorization** — `get_checkout_session_authorization_v2`
   returns the last authorization for the session.
4. **Confirm** — `confirm_authorization_v2` with the values returned by the
   widget. Rebuild the widget's `debtor_company` into the request `debtor`
   hierarchy with `company_address`; the `amount` must match the widget exactly.
   Success returns `HTTP 202`, `state=created`, and the order `uuid`.
5. **Capture on shipment** — `invoice_create` against the order `uuid` (see the
   backend-order skill).

## Alternative
For a fully hosted flow use `create_hpp_checkout_session_v2`, redirect the buyer
to the returned `hpp_url`, and continue from the confirmation step.
