# Async Order Processing

## Why Async?

Payment APIs take 200ms–2000ms.

If API waits:

Connections Held =
(RPS × Payment Time)

DB pool exhausts quickly.

Async processing reduces API latency to ~50ms.

---

## SQS Message Format

```json
{
  "bookingId": "uuid",
  "userId": "uuid",
  "eventId": 101,
  "seatIds": [1,2,3],
  "totalAmount": 2500,
  "paymentToken": "token123",
  "idempotencyKey": "booking-uuid"
}
```

### Fields

bookingId → booking lookup

userId → notifications

eventId → validation

seatIds → seat update

totalAmount → payment charge

paymentToken → gateway transaction

idempotencyKey → prevents duplicate processing

---

## Success Flow

1. Read message from SQS
2. Call payment gateway
3. Update booking = confirmed
4. Update seats = booked
5. Send email/SMS
6. Delete message

---

## Failure Flow

1. Read message
2. Payment fails
3. Update booking = failed
4. Release seats
5. Notify user
6. Delete message

---

## Edge Cases

### API crashes after SQS publish

Booking exists.

Message exists.

Worker still processes payment.

User can check booking status later.

### Payment Timeout

Retry automatically.

After 3 failures:

Move message to DLQ.

Manual investigation required.

---

## SQS Configuration

Visibility Timeout:

30 seconds

Reason:

2× maximum expected payment duration.

Max Receive Count:

3

Reason:

Avoid infinite retries.

After 3 failures move to DLQ.
