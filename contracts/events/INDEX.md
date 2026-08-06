# StockForge — Event Contracts (Kafka)

**Purpose:** every message that crosses a service boundary over Kafka is defined here,
contract-first, exactly like `openapi.yaml` defines HTTP. Events are the backbone of
the order lifecycle: matching, risk, portfolio and notifications all react to these
events rather than calling each other synchronously.

## Topics and partitioning

| Topic | Producer | Consumers | Partition key | Ordering requirement |
|---|---|---|---|---|
| `orders` | order-service | matching-engine, notification | `orderId` | Per order (state transitions) |
| `executions` | matching-engine | portfolio, notification, order-service | `orderId` | Per order |
| `positions` | portfolio | notification, api | `accountId` | Per account |
| `market.prices` | market-data | matching-engine, api | `symbol` | Per symbol (latest wins) |

## The 7 event types

| Event | File | Producer → Consumer |
|---|---|---|
| OrderCreated | `events/OrderCreated.md` | order-service → matching-engine, notification |
| OrderAccepted | `events/OrderAccepted.md` | order-service → notification, api |
| OrderRejected | `events/OrderRejected.md` | order-service / risk-service → notification, api |
| OrderExecuted | `events/OrderExecuted.md` | matching-engine → portfolio, notification, order-service |
| OrderCancelled | `events/OrderCancelled.md` | order-service → matching-engine, notification |
| PositionUpdated | `events/PositionUpdated.md` | portfolio → notification, api |
| MarketPriceUpdated | `events/MarketPriceUpdated.md` | market-data → matching-engine, api |

## Cross-cutting rules (applied to every event)

- **Idempotency:** every event carries `eventId` (UUID). Consumers deduplicate on it —
  Kafka can redeliver, so "process twice" must be harmless.
- **Versioning:** every event carries `schemaVersion`. Consumers must tolerate one
  version ahead/behind during rollout.
- **Failure handling:** poison messages (unparseable, schema mismatch) go to a dead
  letter topic, never silently dropped. Retries are safe because processing is idempotent.
- **Time:** every event carries `timestamp` (epoch millis, UTC) — used for
  order-to-execution latency measurement later.

See `events/<EventName>.md` for each event's schema and notes.
