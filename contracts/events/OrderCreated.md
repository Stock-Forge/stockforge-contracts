# OrderCreated

**Producer:** order-service · **Topic:** `orders` · **Consumers:** matching-engine, notification

Raised when a new order is accepted into the system and needs matching. This is the
first event in the order lifecycle.

## Schema (schemaVersion 1)

```yaml
eventId: uuid                 # idempotency key for consumers
schemaVersion: 1
timestamp: epochMillis        # when the order was created
payload:
  orderId: uuid
  clientOrderId: string       # idempotency key from the client (may be empty)
  accountId: uuid
  symbol: string              # e.g. AAPL
  side: BUY | SELL
  orderType: MARKET | LIMIT
  quantity: int
  limitPrice: double | null   # required for LIMIT, null for MARKET
```

## Downstream effects

- matching-engine: inserts the order into its order book (per symbol).
- notification: pushes "order received" to the UI via WebSocket.

## Failure handling

- If matching-engine is down, the event stays in the topic (consumer lag) — it is
  replayed when the consumer returns. Order state must not be considered final until
  OrderAccepted/OrderRejected is emitted.
- Duplicate delivery: consumers deduplicate on `eventId`.
