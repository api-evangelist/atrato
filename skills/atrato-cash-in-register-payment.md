---
name: Register an in-store cash-in payment
description: Authenticate, look up an Atrato borrower by CIE reference, pick the receiving store, and register one or more payments against the customer's active credits.
api: openapi/atrato-partners-openapi.yml
operations:
  - autenticación
  - get_api-v4-integration-cash-in-search
  - get_api-v4-integration-cash-in-available-stores
  - post_api-v4-integration-cash-in-register-payment
generated: '2026-07-18'
method: generated
---

# Register an in-store cash-in payment (Atrato)

Use this flow to record a customer's in-store payment against their Atrato buy-now-pay-later credits.

## Auth
All calls send the API key in the `x-auth-token` header. Tokens expire after **5 minutes** — re-authenticate as needed.

1. **`autenticación`** — `POST /api/v4/integration/login` with `{ "username", "password" }` (partner-dashboard credentials). Response: `{ "token" }`. Send that token as `x-auth-token` on every subsequent call.

## Steps
2. **`get_api-v4-integration-cash-in-search`** — `GET /api/v4/integration/cash-in/search?reference=U123456789` (CIE reference starts with `U`) **or** `?applicationId=12345`. At least one is required. Returns `userId` and `credits[]` (`creditId`, `debtToDate`, `settleAmount`, `installmentAmount`).
3. **`get_api-v4-integration-cash-in-available-stores`** — `GET /api/v4/integration/cash-in/available-stores`. Returns authorized `stores[]` (`storeId`, `name`). Pick the branch receiving the payment.
4. **`post_api-v4-integration-cash-in-register-payment`** — `POST /api/v4/integration/cash-in/register-payment` with `{ "userId", "storeId", "payments":[{ "creditId", "amount" }] }`. Returns `globalPaymentId`, `status` (`success` | `pending_conciliation` | `cancelled`), `totalAmount`, `appliedPayments`.

## Rules
- Each `amount` must not exceed the credit's `settleAmount` from step 2 (else `400`: "El monto a pagar no puede ser mayor al monto a liquidar").
- `creditId`s must belong to the user and be active.
- `storeId` must be one of the authorized stores; some stores only accept payments 09:00–18:00 Mexico time (else `500`).
- Errors are `application/json` with `message`/`details` (not RFC 9457). See `errors/atrato-problem-types.yml`.
- Reconcile asynchronously via the `UPDATED_STATUS_PAYMENT_CASH_IN_AT_STORE` webhook (dedupe on `paymentId` + `status`). See `asyncapi/atrato-webhooks.yml`.
