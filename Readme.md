# ShowTime Ticketing Platform

## Constraints Analysis

### Constraint 1 - 5 Lakh Concurrent Users

Expected peak traffic is extremely high during ticket launches.

The system must support massive concurrent booking attempts without crashing.

### Constraint 2 - Zero Double Booking

A seat must never be assigned to multiple users.

Redis distributed locks and PostgreSQL optimistic locking are used to guarantee correctness.

### Constraint 3 - $2000 AWS Budget

Infrastructure choices must be cost-efficient.

Redis, PostgreSQL, EC2, SQS, and caching are selected to stay within budget.