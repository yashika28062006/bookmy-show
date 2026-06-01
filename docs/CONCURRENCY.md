# Concurrency Strategy

## Option A - PostgreSQL FOR UPDATE

Pros:

* ACID compliant
* No extra infrastructure
* Easy to reason about

Cons:

* Limited by DB connection pool
* Lock contention under extreme traffic

### Pool Exhaustion Math

Formula:

Connections Held =
(RPS × 80% × 0.02)
+
(RPS × 20% × 0.8)

500 = RPS × 0.176

RPS ≈ 2840

PostgreSQL pool exhausts around 2800 RPS.

## Option B - Redis SETNX

Lock Key:

seat_lock:{event_id}:{seat_id}

TTL = 30 seconds

Pros:

* Sub-millisecond locking
* 100k+ operations/sec
* Offloads DB

Cons:

* Additional infrastructure
* Lock expiry management

## Chosen Strategy: Hybrid

We use:

1. Redis SETNX for seat reservation.
2. PostgreSQL optimistic locking for booking confirmation.

Reason:

* 5 lakh concurrent users exceed PostgreSQL locking capacity.
* Redis absorbs hot-path contention.
* PostgreSQL remains final source of truth.

Limitations:

* Redis outage impacts booking initiation.
* Requires Redis cluster maintenance.

Migration Trigger:
If traffic remains below ~5k concurrent booking attempts, pure PostgreSQL locking would be sufficient and cheaper.
