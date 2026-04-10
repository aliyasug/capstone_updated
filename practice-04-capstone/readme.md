# Enterprise Order Processing Integration System

## Overview

This project demonstrates an Enterprise Application Integration (EAI) solution for orchestrating order processing using Node-RED.

The system coordinates multiple microservices involved in e-commerce order fulfillment:

- Order Service
- Payment Service
- Inventory Service
- Notification Service

Node-RED acts as the central integration/orchestration layer.

---

## Architecture Decision

This implementation uses **Node-RED as the main entry point** for incoming client requests.

Architecture Flow:

Client → Node-RED → Order Service → Payment Service → Inventory Service → Notification Service

### Why this approach?

Node-RED was selected as the entry point because:

- it centralizes orchestration logic;
- simplifies compensation handling;
- allows visual monitoring of business flow;
- reduces coupling between client and internal services.

---

## Enterprise Integration Patterns Used

| Pattern | Problem It Solves | Where Applied | Why Chosen |
|--------|-----------------|--------------|-----------|
| Content-Based Router | Routes requests based on validation result | Input validation step | To separate valid/invalid orders |
| Correlation Identifier | Tracks request through all services | correlationId generated at flow start | For traceability |
| Request-Reply | Synchronous communication with services | HTTP Request nodes | Needed for immediate responses |
| Dead Letter Channel | Handles failed requests | Error output / failed branch | Captures invalid/failed messages |

---

## Business Logic

### Happy Path

1. Client sends POST /order request
2. Node-RED validates payload
3. Order Service creates order
4. Payment Service authorizes payment
5. Inventory Service reserves stock
6. Notification Service sends confirmation
7. Final response returned with trace log

---

### Failure Scenario 1 — Payment Failure

If payment authorization fails:

- process stops immediately;
- inventory reservation is skipped;
- notification is skipped;
- response status becomes **failed**.

---

### Failure Scenario 2 — Inventory Failure

If inventory reservation fails after successful payment:

- payment refund compensation is triggered;
- status becomes **compensated**.

---

## API Endpoint

### Main Endpoint

```bash
POST http://localhost:1880/order
```

Example Body:

```json
{
  "product": "phone"
}
```

---

## Example Success Response

```json
{
  "orderId": "12345",
  "correlationId": "abc-123",
  "status": "completed"
}
```

---

## Running the Project

### Start all services

```bash
docker compose up -d --build
```

### Open Node-RED

```bash
http://localhost:1880
```

---

## Testing

### Successful Order Test

```bash
curl -X POST http://localhost:1880/order \
-H "Content-Type: application/json" \
-d '{"product":"phone"}'
```

---

### Invalid Order Test

```bash
curl -X POST http://localhost:1880/order \
-H "Content-Type: application/json" \
-d '{}'
```

---

## Documentation

Detailed diagrams are located in:

- docs/system-context.md
- docs/integration-architecture.md
- docs/orchestration-flow.md

---

## AI Usage Disclosure

AI tools were used to assist with:

- architecture refinement;
- documentation formatting;
- EIP explanation.

All implementation and configuration were manually reviewed and adapted.