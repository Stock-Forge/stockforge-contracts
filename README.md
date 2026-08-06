# stockforge-contracts

**StockForge** — contract-first agreements between services.

This repository owns the **interfaces** of the platform: the HTTP API contract
(OpenAPI) and the Kafka event contracts. It prevents one repository from silently
breaking another.

## Why a separate contracts repo (production thinking)

In a 10-service platform, every service is built by different teams/repos at
different times. If `stockforge-order-service` changes its API without telling anyone,
`stockforge-web` breaks in production. Contract-first means:

- The **interface is written and versioned before the code**.
- Services and the UI are built **in parallel against the same spec**.
- CI can run **contract tests** against each implementation (added in a later phase).

This is exactly how enterprises manage API catalogs (OpenAPI/AsyncAPI), event
registries (Confluent Schema Registry), and contract tests.

## Repository layout

```
stockforge-contracts/
│
├── README.md
└── contracts/
    ├── openapi.yaml        # HTTP API contract (auth, market-data, orders, portfolio)
    └── events/
        ├── INDEX.md                 # Topics, partitioning, cross-cutting rules
        ├── OrderCreated.md
        ├── OrderAccepted.md
        ├── OrderRejected.md
        ├── OrderExecuted.md
        ├── OrderCancelled.md
        ├── PositionUpdated.md
        └── MarketPriceUpdated.md
```

## Contract rules

1. **Version, don't break.** Never edit an operation/event in a breaking way — add a
   new version (`v2`) and deprecate the old one. Consumers tolerate one version drift.
2. **Every boundary crossing is documented.** Anything the UI or external clients see
   lives in `openapi.yaml`; every Kafka message lives in `contracts/events/`.
3. **Events are idempotent.** Every event carries `eventId`; consumers deduplicate.
   Kafka redelivers — processing twice must be harmless.
4. **Events carry timestamps** for later order-to-execution latency measurement.

## How to work in this repo

- Change requests go through the same Git workflow as everywhere (branch → commit →
  push). This repo has **no application code** — contracts only.
- Update the README when the contract surface changes.
- This repo is consumed by `stockforge-api`, the services, and `stockforge-web`;
  each consumes the specs, not this repo's Git history.

## Current status

- Phase 1 foundation: API contract (v1.0.0) + 7 event contracts defined.
- No implementation yet — services are built against these contracts in later phases.

## Known limitations

- No schema validation tooling yet (e.g. swagger-cli, confluent schema registry) —
  planned when services arrive.
- No contract tests yet (planned Phase 14 with GitHub Actions).
