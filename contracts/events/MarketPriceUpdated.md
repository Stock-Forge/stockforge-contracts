# MarketPriceUpdated

**Producer:** market-data · **Topic:** `market.prices` · **Consumers:** matching-engine, api

Raised on every simulated price tick for a symbol. This is the **highest-volume**
event in the system — it is also streamed to the UI over WebSocket rather than REST.

## Schema (schemaVersion 1)

```yaml
eventId: uuid
schemaVersion: 1
timestamp: epochMillis
payload:
  symbol: string
  bid: double
  ask: double
  last: double
  volume: int64
  sequence: int             # per-symbol tick counter
```

## Downstream effects

- matching-engine: evaluates resting limit orders against the new price
  (e.g. a resting SELL LIMIT fills when bid crosses the limit price).
- api: forwards the tick to connected UI clients via WebSocket.

## Failure handling

- **Latest-wins, latency-sensitive:** stale ticks must be dropped, not queued.
  Unlike order events, no one rebuilds state from the full tick history.
- High volume means the consumer must keep up (bounded lag). If a consumer falls
  behind, it should skip to the latest tick rather than replay thousands.
- Duplicate delivery: deduplicate on `eventId`; staleness guard on `sequence`.
