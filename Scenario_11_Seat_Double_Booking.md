# Scenario 11: Two Users Book the Same Seat Simultaneously

## Scenario
A movie ticket booking system allows two users to reserve the same seat at the exact same time during peak hours. Both users receive confirmation messages, but only one can enter the hall.

---

## Assumptions
- Seat availability is managed in a central database.
- The booking flow involves: seat selection → reservation → payment → confirmation.
- Peak hours create high concurrent booking requests for popular seats.
- The system may use a temporary reservation/hold mechanism before payment.

---

## Test Ideas

### Concurrency Control Testing
- Simulate two simultaneous booking requests for the same seat using parallel API calls.
- Use load testing tools to replicate peak-hour conditions with hundreds of concurrent users.
- Verify that only one booking succeeds when two requests arrive at the same millisecond.
- Test the behavior under different database isolation levels (READ COMMITTED vs SERIALIZABLE).

### Database Locking Testing
- Verify SELECT FOR UPDATE or equivalent locking is used when reading seat availability.
- Test optimistic locking: check if a version/timestamp conflict is properly detected and rejected.
- Verify that the lock is held for the minimum necessary duration to prevent deadlocks.
- Test deadlock scenarios: two users each locking different seats and then requesting the other's.

### Real-Time Availability Testing
- After User A selects a seat, verify that it immediately shows as unavailable to User B.
- Test WebSocket or polling-based real-time seat updates — are they fast enough to prevent conflicts?
- Verify that "seat held/reserved" status is shown to other users during the payment window.
- Test what happens when User A's reservation times out — is the seat immediately released?

### Confirmation & Notification Testing
- Verify that a confirmation message is only sent after the database write is committed.
- Test the rollback scenario: if payment fails, is the seat correctly released?
- Check if duplicate confirmations can be sent due to retry logic.

---

## Edge Cases
- User selects a seat, pauses on the payment page, and the reservation expires — another user books it.
- Race condition between reservation expiry cleanup job and a new booking request for the same seat.
- Multiple booking attempts by the same user (back button + resubmit) create duplicate reservations.
- Mobile app caches seat availability — shows a seat as available locally even after it was booked.
- Group booking reserves multiple seats — some succeed, some fail, leading to partial group reservations.

---

## Risks
- **Customer Experience Risk:** Guests arriving at the hall and being denied entry is a severe failure.
- **Legal Risk:** Confirmed bookings are contracts — a double booking may have legal consequences.
- **Reputation Risk:** Viral complaints about double bookings can be highly damaging.
- **Scalability Risk:** Strong locking strategies may become bottlenecks during very high concurrency.
- **Revenue Risk:** If both users are refunded, the seat effectively generates no revenue.
