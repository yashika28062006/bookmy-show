# Database Schema

## Venues

```sql
CREATE TABLE venues (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    capacity INT NOT NULL CHECK (capacity > 0)
);

CREATE INDEX idx_venues_city ON venues(city);
```

## Events

```sql
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    venue_id INT NOT NULL REFERENCES venues(id) ON DELETE RESTRICT,
    name VARCHAR(255) NOT NULL,
    start_time TIMESTAMP NOT NULL,
    status VARCHAR(20) NOT NULL CHECK (
        status IN ('upcoming','on_sale','sold_out','cancelled')
    ),
    total_seat_count INT NOT NULL CHECK (total_seat_count > 0)
);

CREATE INDEX idx_events_status ON events(status);
```

## Users

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Seats

```sql
CREATE TABLE seats (
    id SERIAL PRIMARY KEY,
    event_id INT NOT NULL REFERENCES events(id) ON DELETE CASCADE,
    section VARCHAR(50) NOT NULL,
    row_name VARCHAR(20) NOT NULL,
    seat_number VARCHAR(20) NOT NULL,
    category VARCHAR(20) NOT NULL CHECK (
        category IN ('VIP','Premium','General')
    ),
    price NUMERIC(10,2) NOT NULL CHECK (price > 0),
    status VARCHAR(20) NOT NULL CHECK (
        status IN ('available','held','booked')
    ),
    held_until TIMESTAMP,
    held_by UUID REFERENCES users(id) ON DELETE SET NULL,
    version INT DEFAULT 0 NOT NULL
);

CREATE INDEX idx_seats_event_status
ON seats(event_id,status);
```

## Bookings

```sql
CREATE TABLE bookings (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE RESTRICT,
    event_id INT NOT NULL REFERENCES events(id) ON DELETE RESTRICT,
    status VARCHAR(20) NOT NULL CHECK (
        status IN ('pending','confirmed','failed','refunded')
    ),
    total_amount NUMERIC(10,2) NOT NULL CHECK (total_amount > 0),
    payment_reference VARCHAR(255),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_bookings_user
ON bookings(user_id, created_at DESC);

CREATE INDEX idx_bookings_unresolved
ON bookings(status)
WHERE status IN ('pending','failed');
```

## Booking Seats

```sql
CREATE TABLE booking_seats (
    booking_id UUID NOT NULL REFERENCES bookings(id) ON DELETE CASCADE,
    seat_id INT NOT NULL REFERENCES seats(id) ON DELETE RESTRICT,
    PRIMARY KEY (booking_id, seat_id)
);
```

## Commentary

### Why UUID?

UUID prevents booking enumeration and supports distributed systems.

### Why version column?

Supports optimistic locking and conflict detection.

### Why held_until?

Automatically releases abandoned seats after hold expiration.

### Why partial index?

Keeps unresolved booking lookups extremely fast.

```
```
