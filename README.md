# Checkout gateway
Microservice handling payments and order confirmation for the e-commerce site.
# Payments & Order Confirmation Microservice

A mock microservice responsible for processing payments and confirming customer orders for an e-commerce platform.

This service is designed to sit between the storefront, payment provider, order management system, and notification service. It validates checkout requests, processes payments, updates order status, and emits confirmation events.

---

## Overview

The Payments & Order Confirmation Microservice handles the final stage of the checkout flow:

1. Receives a payment request from the e-commerce checkout service.
2. Validates customer, cart, pricing, and payment details.
3. Sends the payment request to a payment gateway.
4. Updates the order status based on payment outcome.
5. Publishes an order confirmation event.
6. Triggers confirmation notifications such as email or SMS.

---

## Key Features

- Payment authorization and capture
- Order confirmation after successful payment
- Payment failure handling
- Idempotent payment processing
- Order status updates
- Event publishing for downstream services
- Basic audit logging
- Health check endpoint
- Mock-ready local development setup

---

## Tech Stack

> This is a mock README. Update these values to match the final implementation.

- **Language:** Node.js / TypeScript
- **Framework:** Express.js
- **Database:** PostgreSQL
- **Message Broker:** Kafka or RabbitMQ
- **Payment Provider:** Stripe, Adyen, PayPal, or mock gateway
- **Containerization:** Docker
- **Testing:** Jest

---

## Service Responsibilities

This microservice is responsible for:

- Accepting payment requests for existing orders
- Preventing duplicate charges through idempotency keys
- Communicating with external payment providers
- Recording payment transaction status
- Confirming orders after successful payment
- Publishing events for email, inventory, fulfillment, and analytics services

This microservice is not responsible for:

- Cart management
- Product catalog management
- Inventory reservation
- Shipping calculation
- Customer authentication
- Tax calculation

---

## Example Architecture

```text
[Checkout Service]
        |
        v
[Payments & Order Confirmation Service]
        |
        +--> [Payment Gateway]
        |
        +--> [Orders Database]
        |
        +--> [Message Broker]
                  |
                  +--> [Notification Service]
                  +--> [Inventory Service]
                  +--> [Fulfillment Service]
```

---

## API Endpoints

### Health Check

```http
GET /health
```

#### Example Response

```json
{
  "status": "ok",
  "service": "payments-order-confirmation",
  "timestamp": "2026-05-07T12:00:00Z"
}
```

---

### Process Payment

```http
POST /api/v1/payments
```

Processes payment for an order and confirms the order if payment succeeds.

#### Request Body

```json
{
  "orderId": "ord_123456",
  "customerId": "cus_987654",
  "amount": 129.99,
  "currency": "USD",
  "paymentMethodId": "pm_card_visa",
  "idempotencyKey": "checkout_abc123"
}
```

#### Success Response

```json
{
  "paymentId": "pay_123456",
  "orderId": "ord_123456",
  "status": "confirmed",
  "paymentStatus": "succeeded",
  "amount": 129.99,
  "currency": "USD",
  "confirmedAt": "2026-05-07T12:00:00Z"
}
```

#### Failure Response

```json
{
  "orderId": "ord_123456",
  "status": "payment_failed",
  "paymentStatus": "declined",
  "reason": "Insufficient funds"
}
```

---

### Get Payment Status

```http
GET /api/v1/payments/{paymentId}
```

#### Example Response

```json
{
  "paymentId": "pay_123456",
  "orderId": "ord_123456",
  "paymentStatus": "succeeded",
  "orderStatus": "confirmed",
  "amount": 129.99,
  "currency": "USD"
}
```

---

## Events

### Published Events

#### `payment.succeeded`

Published when a payment is successfully processed.

```json
{
  "eventType": "payment.succeeded",
  "paymentId": "pay_123456",
  "orderId": "ord_123456",
  "customerId": "cus_987654",
  "amount": 129.99,
  "currency": "USD",
  "timestamp": "2026-05-07T12:00:00Z"
}
```

#### `payment.failed`

Published when a payment attempt fails.

```json
{
  "eventType": "payment.failed",
  "orderId": "ord_123456",
  "customerId": "cus_987654",
  "reason": "Insufficient funds",
  "timestamp": "2026-05-07T12:00:00Z"
}
```

#### `order.confirmed`

Published when an order is confirmed after successful payment.

```json
{
  "eventType": "order.confirmed",
  "orderId": "ord_123456",
  "customerId": "cus_987654",
  "paymentId": "pay_123456",
  "timestamp": "2026-05-07T12:00:00Z"
}
```

---

## Environment Variables

Create a `.env` file in the project root.

```env
NODE_ENV=development
PORT=3000

DATABASE_URL=postgresql://user:password@localhost:5432/payments_db

PAYMENT_PROVIDER=mock
PAYMENT_API_KEY=replace_me
PAYMENT_WEBHOOK_SECRET=replace_me

MESSAGE_BROKER_URL=localhost:9092

LOG_LEVEL=info
```

---

## Getting Started

### Prerequisites

- Node.js 20+
- npm or yarn
- Docker and Docker Compose
- PostgreSQL

---

### Installation

```bash
git clone https://github.com/your-org/payments-order-confirmation-service.git
cd payments-order-confirmation-service
npm install
```

---

### Run Locally

```bash
npm run dev
```

The service should be available at:

```text
http://localhost:3000
```

---

### Run with Docker

```bash
docker compose up --build
```

---

## Testing

Run the test suite:

```bash
npm test
```

Run tests with coverage:

```bash
npm run test:coverage
```

---

## Example cURL Request

```bash
curl -X POST http://localhost:3000/api/v1/payments \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: checkout_abc123" \
  -d '{
    "orderId": "ord_123456",
    "customerId": "cus_987654",
    "amount": 129.99,
    "currency": "USD",
    "paymentMethodId": "pm_card_visa",
    "idempotencyKey": "checkout_abc123"
  }'
```

---

## Idempotency

Payment requests must include an `idempotencyKey`.

If the same request is submitted more than once with the same key, the service should return the original response instead of charging the customer again.

This protects against:

- Network retries
- Browser refreshes
- Duplicate checkout submissions
- Payment gateway timeout retries

---

## Error Handling

Common error responses include:

| Status Code | Error | Description |
| --- | --- | --- |
| 400 | `INVALID_REQUEST` | Missing or invalid request fields |
| 401 | `UNAUTHORIZED` | Missing or invalid credentials |
| 404 | `ORDER_NOT_FOUND` | Order does not exist |
| 409 | `DUPLICATE_PAYMENT` | Payment already processed |
| 422 | `PAYMENT_DECLINED` | Payment provider declined the transaction |
| 500 | `INTERNAL_ERROR` | Unexpected server error |

---

## Security Considerations

- Do not store raw card details.
- Use tokenized payment methods only.
- Store secrets in a secure secret manager.
- Validate webhook signatures from payment providers.
- Use HTTPS in production.
- Log payment metadata, not sensitive payment details.
- Apply rate limiting to payment endpoints.

---

## Observability

Recommended logs and metrics:

- Payment request count
- Payment success rate
- Payment failure rate
- Payment provider latency
- Order confirmation latency
- Duplicate idempotency key count
- Error rate by endpoint

---

## Suggested Project Structure

```text
.
├── src
│   ├── controllers
│   ├── services
│   ├── repositories
│   ├── events
│   ├── middleware
│   ├── config
│   └── index.ts
├── tests
├── docker-compose.yml
├── Dockerfile
├── package.json
└── README.md
```

---

## Roadmap

- [ ] Add real payment gateway integration
- [ ] Add webhook handling
- [ ] Add retry strategy for failed provider calls
- [ ] Add distributed tracing
- [ ] Add fraud detection hooks
- [ ] Add refund support
- [ ] Add admin payment reconciliation endpoint

---

## Contributing

1. Create a feature branch.
2. Commit your changes with clear messages.
3. Add or update tests.
4. Open a pull request.
5. Request review from the payments or platform team.

---

## License

This project is currently provided as a mock internal service template. Update this section with the appropriate license before publishing.
