# DESIGN DECISIONS

## Decision: Redis SETNX for Seat Locking

**Context:**
The system must prevent double-booking when multiple users try to book the same seat simultaneously during high-demand events.

**Options considered:**

1. PostgreSQL FOR UPDATE row locking

   * Strong consistency
   * Creates heavy database contention during traffic spikes

2. Optimistic locking using version columns

   * Reduces lock contention
   * More retry logic required

3. Redis SETNX distributed locking (Chosen)

   * Fast lock acquisition
   * Reduces database load significantly

**Why chosen:**
Redis SETNX provides low-latency distributed locking and prevents thousands of concurrent booking requests from overwhelming the database. The database remains the source of truth while Redis handles lock management.

**Tradeoffs accepted:**
The system becomes dependent on Redis availability. Additional monitoring and failover handling are required.

**Revision trigger:**
If Redis becomes a bottleneck or traffic exceeds current capacity, consider Redlock or alternative distributed lock mechanisms.

---

## Decision: TTL-Based Cache Invalidation

**Context:**
Event details and seat availability are frequently read but updated less often.

**Options considered:**

1. Event-driven cache invalidation

   * More accurate
   * Higher implementation complexity

2. TTL-only cache expiration (Chosen)

   * Simpler implementation
   * Easier operational maintenance

**Why chosen:**
A short TTL provides a good balance between freshness and reduced database load while keeping the architecture simple.

**Tradeoffs accepted:**
Users may occasionally see slightly stale availability information.

**Revision trigger:**
If stale data impacts user experience significantly, move to event-driven invalidation.

---

## Decision: UUID for Booking IDs

**Context:**
Booking IDs must be unique across all servers and difficult to predict.

**Options considered:**

1. SERIAL auto-increment IDs

   * Simple
   * Predictable and sequential

2. UUID (Chosen)

   * Globally unique
   * Hard to guess
   * Works well in distributed environments

**Why chosen:**
UUIDs prevent enumeration attacks and avoid coordination issues across multiple application servers.

**Tradeoffs accepted:**
Larger storage size and slightly larger indexes.

**Revision trigger:**
If storage efficiency becomes a major concern at very large scale.

---

## Decision: SQS Visibility Timeout = 20 Seconds

**Context:**
Payment processing should not result in duplicate booking confirmations.

**Options considered:**

1. 10 seconds

   * Faster retries
   * Higher risk of duplicate processing

2. 20 seconds (Chosen)

   * Matches expected payment processing duration

3. 60 seconds

   * Fewer duplicate retries
   * Slower recovery from worker failures

**Why chosen:**
Twenty seconds provides sufficient time for payment workers to complete processing while keeping recovery times reasonable.

**Tradeoffs accepted:**
If a worker crashes, message retry is delayed until timeout expires.

**Revision trigger:**
If payment processing time consistently increases or decreases.

