# Design — Wallet Transfer Service

## Stack
Go, net/http + chi router, PostgreSQL.

## Schema
- wallets(id, balance, version, created_at)
- transfers(id, idempotency_key, from_wallet_id, to_wallet_id, amount, status, created_at, updated_at)
- ledger_entries(id, transfer_id FK, wallet_id, type, amount, created_at)
- idempotency_records(idempotency_key PK, transfer_id, response_body, created_at)

## API
POST /transfers — see assignment spec for request/response shape.

## Idempotency strategy
Unique constraint on idempotency_key. On request: look up key first; if found, return stored response. If not, proceed inside a transaction and persist the record atomically with the transfer.

## Concurrency strategy
Row-level locking: SELECT ... FOR UPDATE on both wallets, always locked in a fixed order (e.g. sorted by wallet id) to avoid deadlocks, inside a single DB transaction.

## State machine
PENDING -> PROCESSED | FAILED. Transition happens once per transfer, guarded by the idempotency check so retries never re-trigger a transition.

## Testing plan
Unit tests per layer, integration test for duplicate idempotency key, concurrency test firing parallel transfers against the same wallet.
