# OrderAccepted

**Producer:** order-service · **Topic:** `orders` · **Consumers:** notification, api

Raised after validation and risk checks pass. The order is now live and resting in
the matching engine (or immediately executable for a market order).

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
  orderType: MARKET | LIMIT
  quantity: int
  limitPrice: double | null
  status: ACCEPTED
```

## Downstream effects

- notification: pushes "order accepted" to the UI.
- api: updates the client's order status if the create-order request is still in flight.

## Failure handling

- This event must follow OrderCreated on the same partition (key = orderId) so a
  consumer observes state in order. If ordering is violated, a consumer must reject
  the event (out-of-order guard) rather than process it incorrectly.
- Duplicate delivery: deduplicate on `eventId`.
