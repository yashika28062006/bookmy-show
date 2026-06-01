# Cache Design

## Event Details

Key:

event:{event_id}

TTL:

3600 seconds

Invalidate:

On event update.

---

## Seat Availability Count

Key:

availability:{event_id}:{category}

TTL:

30 seconds

Invalidate:

Whenever any seat changes status.

Reason:

High-read, low-write data.

---

## Seat Map Layout

Key:

seatmap:{event_id}

TTL:

86400 seconds

Invalidate:

Event cancellation only.

---

## What We Do NOT Cache

### Individual Seat Status

Reason:

Can cause stale reads and race conditions.

Always read from source of truth.

---

## Cache Invalidation Strategy

Pattern:

Cache Aside

Flow:

1. Update PostgreSQL
2. Delete Redis key
3. Next request rebuilds cache

Pseudo Flow:

When seat status changes:

* Update DB
* Delete availability:{event_id}:{category}
* Future read reloads cache
