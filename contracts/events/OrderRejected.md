# OrderRejected

**Producer:** order-service (on behalf of risk-service) · **Topic:** `orders` ·
**Consumers:** notification, api

Raised when an order fails validation or risk checks (balance, margin, position or
quantity limits). The order never reaches the matching engine.

## Schema (schemaVersion 1)

```yaml
eventId: uuid
schemaVersion: 1
timestamp: epochMillis
payload:
  orderId: uuid
  accountId: uuid
  symbol: string
  side: BUY | SELL
  quantity: int
  reason: string          # e.g. INSUFFICIENT_BALANCE, POSITION_LIMIT_EXCEEDED
  rejectedBy: string      # VALIDATION | RISK
```

## Downstream effects

- notification: pushes "order rejected (<reason>)" to the UI.
- api: returns HTTP 422 with the rejection reason.

## Failure handling

- Risk rejection must be **authoritative**: after OrderRejected, the order is terminal.
  No downstream system may later act on it.
- If the rejection event is lost, the client might never learn — so the rejection must
  also be returned synchronously in the API response (event is a copy, not the source).
- Duplicate delivery: deduplicate on `eventId`.
