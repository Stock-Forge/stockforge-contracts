# OrderCancelled

**Producer:** order-service · **Topic:** `orders` · **Consumers:** matching-engine, notification

Raised when an open order is cancelled by the user. Only resting (unfilled) orders can
be cancelled.

## Schema (schemaVersion 1)

```yaml
eventId: uuid
schemaVersion: 1
timestamp: epochMillis
payload:
  orderId: uuid
  accountId: uuid
  symbol: string
  cancelledQuantity: int   # quantity that was still resting
  reason: USER_CANCELLED | EXPIRED
```

## Downstream effects

- matching-engine: removes the order from the order book.
- notification: pushes "order cancelled" to the UI.

## Failure handling

- **Race: cancel vs fill.** The order may fill at almost the same instant a cancel
  arrives. Resolution: cancel is best-effort; whoever commits first wins, and the
  system must emit a final consistent state (FILLED or CANCELLED, never both visible
  as terminal). This race is a classic production bug — we will reproduce it later.
- Duplicate delivery: deduplicate on `eventId`.
