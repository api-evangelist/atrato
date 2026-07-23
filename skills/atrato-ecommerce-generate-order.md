---
name: Generate an ecommerce BNPL checkout order
description: Authenticate an ecommerce partner, generate an Atrato credit order at checkout, poll its status, and register delivery once the credit is authorized.
api: openapi/atrato-partners-openapi.yml
operations:
  - obtener-token
  - generar-orden
  - consulta-de-orden
  - registrar-entrega
generated: '2026-07-18'
method: generated
---

# Generate an ecommerce BNPL checkout order (Atrato)

Use this flow to offer Atrato buy-now-pay-later as a checkout payment method in an online store. Every ecommerce path includes a `{partner-key}` segment obtained from the partner dashboard (Development > API).

## Auth
1. **`obtener-token`** — `POST /api/v4/ecommerce/integration/{partner-key}/login` with `{ "username", "password" }`. Returns a token; send it as the `x-auth-token` header on subsequent calls (5-minute TTL).

## Steps
2. **`generar-orden`** — `POST /api/v4/ecommerce/integration/{partner-key}/application` with `{ "orderId", "requestedAmount", "productDescription", "customerName", "customerEmail", "customerPhone" }`. Creates the Atrato credit application/order and returns the redirect/reference to continue the customer through Atrato's flow.
3. **`consulta-de-orden`** — `GET /api/v4/ecommerce/integration/{partner-key}/order/{order-id}`. Poll (or rely on webhooks) until the application reaches `CREDITOAUTORIZADO` (status_id 29) or a terminal state (`DENEGADO`, `CANCELADO`, `CADUCADO`).
4. **`registrar-entrega`** — `POST /api/v4/ecommerce/integration/{partner-key}/order/{order-id}/delivery` once the goods are delivered, to confirm the order and trigger merchant disbursement.

## Rules
- Prefer the `status_updated` webhook over polling; dedupe on `atrato_application_id` + `status_code`. Disbursements arrive as `disbursement_updated`. See `asyncapi/atrato-webhooks.yml`.
- Cancel unfulfilled orders with `solicitar-cancelacion` (`POST .../order/{order-id}/cancel`).
- `generar-orden` can return `422` for unprocessable business data; all errors are `application/json` with a `message`. See `errors/atrato-problem-types.yml`.
- Track disbursements with `consulta-de-desembolsos` / `consulta-de-desembolsos-orden`.
