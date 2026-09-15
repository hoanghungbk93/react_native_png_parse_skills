---
name: race-safe-points-and-money
description: Use when code touches points, coins, wallets, ledgers, vouchers, one-time grants, approvals, or bulk imports in an async Python/SQLAlchemy backend — especially before declaring a race condition "fixed", when a parallel test passes suspiciously easily, or when the test suite runs on SQLite while production is Postgres.
---

# Race-safe points & money

## Overview
Every "read state → check → write" on something money-like runs at least twice in
production: mobile apps fire 5–10 requests in parallel on launch, humans double-click,
each request has its own DB session. Python-level checks do not serialize them.
Core principle: **the database decides who wins, and the test must be able to fail.**

## Audit checklist — find every money-like effect
grep for these and treat each as a race site until proven otherwise:
- one-time grants in auth dependencies (`welcome`, `first_login`, `*_at is None`)
- status transitions that create value (`status = APPROVED`, ledger/`delta` inserts)
- balance checks before spending (`balance < cost`), stock/voucher hand-out
- bulk imports (HR double-click / F5 → whole batch twice)
- period limits ("1 per quarter", "3 per day") and streak/badge awards

Ask per site: is the balance **derived** (SUM of rows) or **stored** (counter)? Derived
survives a duplicate approval (no double points, only duplicate side effects); stored
counters and ledger inserts multiply real value → fix those first.

## Patterns (pick by shape)
| Shape | Fix |
|---|---|
| Do X once per user/entity | Claim first: `UPDATE t SET flag=now() WHERE id=? AND flag IS NULL` → `rowcount == 1` wins; others return. Never `if flag is None:` then write. |
| Approve/reject/cancel a row | `SELECT … FOR UPDATE` on that row *before* reading its status (Postgres only; skip on SQLite). |
| Checks spanning two users (giver+receiver, sender+wallet) | Lock both rows in **ascending id order** in one statement — crossed order A→B / B→A deadlocks. |
| Hand out one of N codes/items | `FOR UPDATE SKIP LOCKED` + `LIMIT 1`. |
| "Only one badge/entry per (user, kind)" | UNIQUE constraint; catch `IntegrityError` → treat as already done. |
| Import the same file twice | Hash normalized rows, store hash in audit log, refuse same hash within N minutes (offer "undo old batch"). |

Central helper (one place, dialect-guarded):
```python
async def lock_rows(db, model, ids):
    ids = sorted({int(i) for i in ids if i is not None})
    if ids and db.bind.dialect.name == "postgresql":
        await db.execute(select(model.id).where(model.id.in_(ids)).order_by(model.id).with_for_update())
```

## Proving the fix — the test must be able to fail
A parallel test that passes is **not evidence** until you have seen it fail:
1. **Run on real Postgres** (`TEST_DATABASE_URL=postgresql+asyncpg://…`; SQLite skips FOR UPDATE, so mark these tests skip-on-sqlite). A throwaway server: `pg_ctl initdb -D pgtest && pg_ctl -D pgtest -o "-p 5499" start`.
2. **Widen the race window.** On a fast local DB, `asyncio.gather` of 6 requests still runs the critical section back-to-back and passes *with the lock removed*. Monkeypatch an `await asyncio.sleep(0.2)` right after the state read (inside/after the locked fetch so the lock still covers it).
3. **Mutation-test the lock**: make `lock_rows` a no-op → the test MUST go red; restore → green. Only then commit.
4. Assert on **side-effect counts** (approval rows, notifications, ledger rows == 1), not only on HTTP status codes.

## After the fact
Duplicates already in prod: keep `min(id)` per (user, kind), delete the rest in one
transaction, print the affected users first; derived balances heal themselves once rows are gone.

## Red flags
- "The status check already prevents it" — only inside one process with no await in between.
- "Test passes on Postgres" without step 3.
- Locking user rows in request order instead of id order.
