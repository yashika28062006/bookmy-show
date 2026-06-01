# Panel Q&A Notes

## Q1: What if Redis crashes during seat hold?
Answer:
Return 503 instead of proceeding without a lock.
DB optimistic locking prevents double booking.

## Q2: Read replica lag causes wrong availability?
Answer:
Availability shown is approximate.
Booking validation always happens on primary DB.

## Q3: User holds 200 seats?
Answer:
Redis per-user counter.
Maximum 8 active seat holds.

## Q4: SQS outage?
Answer:
Status polling endpoint.
Circuit breaker after 60s failures.

## Q5: Infrastructure cost increases?
Answer:
Peak-event costs are budgeted separately.
Use Spot Instances and better CDN caching.