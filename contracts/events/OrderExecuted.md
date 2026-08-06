# OrderExecuted

**Producer:** matching-engine · **Topic:** `executions` · **Consumers:** portfolio,
notification, order-service

Raised for every fill — a partial fill or a full fill. One order can produce several
OrderExecuted events (partial fills). This is where money moves, so correctness is
paramount.

## Schema (schemaVersion 1)

```yaml
eventId: uuid
schemaVersion: 1
timestamp: epochMillis
payload:
  orderId: uuid
  accountId: uuid
  executionId: uuid         # unique per fill
  symbol: string
  side: BUY | SELL
  price: double             # execution price
  quantity: int             # quantity filled in THIS execution
  remainingQuantity: int    # what is left unfilled on the order
  orderStatus: PARTIALLY_FILLED | FILLED
  sequence: int             # fill number for this order (1, 2, 3…)
```

## Downstream effects

- portfolio: updates positions, cash balance, and P&L (execution is the only source of
  truth for money movement).
- order-service: marks the order filled/partially-filled; publishes final state.
- notification: pushes execution to the UI in real time.

## Failure handling

- **Never lose an execution.** If portfolio is down, the event must be retained and
  replayed; portfolio must be able to rebuild from execution history.
- **Never double-count.** Idempotency on `executionId` is mandatory — replays must not
  credit a fill twice. This is a classic production incident (duplicate fills).
- Ordering: `sequence` lets a consumer detect gaps or out-of-order deliveries.
- Duplicate delivery: deduplicate on `executionId`.
