---
name: Subscribe to Defacto webhooks
description: Register a webhook subscription and react to Defacto loan, invoice, payment, and credit-line events in real time.
api: openapi/defacto-openapi-original.json
operations:
- post_webhooks
- get_webhooks
- patch_webhook-webhook-id
- delete_webhook-webhook-id
---

# Subscribe to Defacto webhooks

Defacto notifies you of resource state changes by **POSTing** a JSON payload to a URL you register. Authenticate management calls with `Authorization: Bearer <API_KEY>`.

## Steps

1. **Create a subscription** — `POST /webhooks` (`post_webhooks`) with `to_url` (your endpoint) and the `event_types` you want (e.g. `Invoice.SUBMITTED`, `Invoice.VERIFIED`, `Loan.VALIDATED`, `Loan.DECLINED`, `Payment.PAID`).
2. **List subscriptions** — `GET /webhooks` (`get_webhooks`) to recover subscription URLs/ids.
3. **Update / remove** — `PATCH /webhook/{webhook_id}` (`patch_webhook-webhook-id`) or `DELETE /webhook/{webhook_id}` (`delete_webhook-webhook-id`).

## Handling notifications

- Your callback **must accept the POST method** — Defacto always calls POST on `to_url`.
- Payload shape: `entity_type`, `transition_name`, `status`, `timestamp`, `id`, `event_type`, and `entity_state` (the same payload the entity's GET endpoint returns).
- The full event-type catalog (32 types across Borrower/CreditLine/Invoice/Loan/Payment) and payload schema are in `asyncapi/defacto-webhooks-asyncapi.yml`.
- Known caveat: the `CreditLine.CREATED` event sends a `borrower_id` that is an internal business id, not the borrower id from `POST /borrowers`; on receipt, re-fetch `GET /credit_lines?borrower=<company_number>`.
