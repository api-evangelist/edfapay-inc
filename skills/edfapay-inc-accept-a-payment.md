---
name: Accept an EdfaPay payment
description: Take a card payment via the EdfaPay Server-to-Server gateway, handle 3D Secure, and verify via webhook + STATUS.
api: EdfaPay Payment Gateway (Server-to-Server)
host: https://api.edfapay.com
auth: client_key / password MD5 hash signature
operations:
  - SALE     # Server-to-Server sale
  - STATUS   # transaction status lookup
  - REFUND   # refund a transaction
generated: '2026-07-19'
method: generated
source: https://docs.edfapay.com/docs/overview + https://docs.edfapay.com/docs/webhook-operation-types
---

# Accept an EdfaPay payment

Take a card payment server-to-server, handle 3D Secure, and confirm the result.

## Steps

1. **Build the signed request.** Compute the `hash`:
   `MD5( reverse(payer_email) + password + reverse(first6 + last4 of card number) )`
   uppercased (see authentication/edfapay-inc-authentication.yml). Include your
   `client_key`.
2. **Submit the SALE** with `action: Initiate` to create a SALE (or AUTH)
   transaction.
3. **Handle the response `result`:**
   - `SUCCESS` -> proceed; final `status` is `SETTLED`.
   - `REDIRECT` / `3DS` -> redirect the customer to `redirect_url` to complete
     3D Secure, then wait for the webhook.
   - `DECLINED` / `ERROR` -> read the decline outcome
     (errors/edfapay-inc-decline-codes.yml).
4. **Receive the webhook.** EdfaPay POSTs the outcome to your configured
   callback URL. Verify the `hash` field before trusting it
   (asyncapi/edfapay-inc-webhooks.yml).
5. **Confirm authoritatively.** Because webhooks are asynchronous and not final,
   call `STATUS` (`action: STATUS`) to confirm the transaction before fulfilling.
6. **Refund if needed** with `action: Refund` (REFUND), full or partial.

## Rules

- Distinguish `result` (immediate action outcome) from `status` (lifecycle
  state) — a `SUCCESS` result does not guarantee `SETTLED`.
- The gateway accepts no idempotency-key header; dedupe webhook handling on
  `trans_id` and confirm with STATUS before re-charging on TIMED_OUT/UNKNOWN.
- Always use HTTPS; never log full card numbers, CVV, or the password.
