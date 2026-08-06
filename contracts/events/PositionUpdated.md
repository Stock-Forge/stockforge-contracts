# PositionUpdated

**Producer:** portfolio · **Topic:** `positions` · **Consumers:** notification, api

Raised whenever an account's position or balance changes (after executions).
This is a state snapshot for the account, not a transaction log.

## Schema (schemaVersion 1)

```yaml
eventId: uuid
schemaVersion: 1
timestamp: epochMillis
payload:
  accountId: uuid
  cash: double
  positions:
    - symbol: string
      quantity: int
      averagePrice: double
  unrealisedPnl: double
  totalValue: double
  changeReason: EXECUTION | FEE_ADJUSTMENT
```

## Downstream effects

- notification: pushes updated portfolio/P&L to the UI.
- api: refreshes cached portfolio for authenticated clients.

## Failure handling

- Latest-wins semantics: the most recent PositionUpdated for an account supersedes
  older ones. Ordering per account (partition key = accountId) is therefore required.
- If a consumer processes out of order, it must detect staleness (event time < last
  applied time) and ignore.
- Duplicate delivery: deduplicate on `eventId`.
