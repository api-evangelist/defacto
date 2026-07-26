---
name: Finance an invoice with Defacto
description: Onboard a borrower, share an invoice, request a loan against it, and confirm repayment via Defacto's embedded financing API.
api: openapi/defacto-openapi-original.json
operations:
- get_hello
- post_borrowers
- post_invoices
- post_loans
- post_loan-loan-id-validate
- get_loan-loan-id
---

# Finance an invoice with Defacto

Use the Defacto API to advance cash against a business invoice. All calls are authenticated with an HTTP Bearer API key (`Authorization: Bearer <API_KEY>`). Test against the sandbox host `https://api-sandbox.getdefacto.com` before production `https://api.getdefacto.com`. Sandbox and production use **distinct** keys.

## Steps

1. **Verify connectivity** — `GET /hello` (`get_hello`). A welcome message confirms the key works.
2. **Enroll the borrower** — `POST /borrowers` (`post_borrowers`). This creates the borrower and opens a credit line. Set `wait_for_ready: true` to get a synchronous decision, or subscribe to the `CreditLine.CREATED` webhook. For direct-debit repayment you must supply the director's identity and the borrower IBAN.
3. **Create the invoice** — `POST /invoices` (`post_invoices`) with the base64-encoded invoice PDF, or `POST /invoices/upload` (`post_invoices-upload`) to extract fields from the PDF automatically.
4. **Request the loan** — `POST /loans` (`post_loans`) referencing the borrower and `invoice_ids`. **Always set a unique `salt_id`** so retries don't create duplicate loans (idempotency is keyed on `salt_id`). Use `wait_for_validation: true` for a synchronous decision.
5. **Validate the offer** — if the loan returns status `TO_VALIDATE`, call `POST /loan/{loan_id}/validate` (`post_loan-loan-id-validate`) to accept it (or pass `auto_validate: true` on the loan request to skip this).
6. **Track status** — poll `GET /loan/{loan_id}` (`get_loan-loan-id`) or subscribe to `Loan.*` webhooks (`Loan.VALIDATED`, `Loan.SCHEDULED`, `Loan.TO_REPAY`, `Loan.CLOSED`, `Loan.DECLINED`).

## Rules

- **Idempotency**: `salt_id` on `POST /loans` dedups; a repeated `salt_id` returns the existing loan.
- **Async**: without `wait_for_validation`, a loan is `PENDING_VALIDATION` until a webhook reports the decision (seconds to a couple of business days). Communicate declines to the end user.
- **Declines**: a declined loan carries a `denial_reason`; resolve human text via `GET /translation/denial-code/{code}`. Do not hard-code the reason list — enumerate it with `GET /eligibility/reasons`.
- **Timeouts**: long operations return 504 after 30s; retry with the same `salt_id`.
- **Errors**: standard HTTP status codes with JSON bodies; `POST /invoice/{invoice_id}/submit` returns 422 listing missing fields.
