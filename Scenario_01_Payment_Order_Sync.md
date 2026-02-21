# Scenario 01: Payment Confirmed But Order Never Received

## Scenario
A food delivery app confirms payment and shows "Order Placed Successfully." However, the restaurant dashboard never receives the order, while the customer's money is already deducted. Customer support can see the payment record but no order record in the admin panel.

---

## Assumptions
- The system uses separate microservices for Payment and Order Management.
- Payment gateway triggers an event/webhook to create the order.
- There is an admin panel with access to both payment and order records.
- Logs are available at the service, API gateway, and database levels.
- Message queues (e.g., Kafka, RabbitMQ) may be used between services.

---

## Test Ideas

### Flow Tracing
- Trace the complete API call chain: Payment Gateway → Payment Service → Order Service → Restaurant Notification Service.
- Check if the payment gateway webhook/callback is correctly received by the backend.
- Verify if the order creation API is called after successful payment confirmation.
- Check database transaction logs: does an order record get inserted after payment record?

### Service & API Validation
- Test the order creation API independently with a mock successful payment payload.
- Validate that the Payment Service publishes an event to the message queue after success.
- Validate that the Order Service consumes the event and creates the order atomically.
- Check API response codes between services — is a 200 returned even on internal failure?

### Log Analysis
- Check Payment Service logs for confirmation of event publishing.
- Check Order Service logs for event consumption and any silent failures.
- Check message queue dead-letter queues for failed/unprocessed messages.
- Check restaurant notification service logs for any delivery failures.

### Database Checks
- Verify foreign key or transaction link between payment_id and order_id tables.
- Check if the order table insert is wrapped in the same transaction as payment confirmation.

---

## Edge Cases
- Payment succeeds but the message queue is temporarily down — order event is lost.
- Order Service receives the event but crashes mid-creation (partial write).
- Network timeout between Payment Service and Order Service causes a retry that is not idempotent.
- Restaurant dashboard uses a different data source (replica DB) that is lagging.
- Admin panel reads from a different service than the restaurant dashboard.
- Payment gateway sends duplicate callbacks — one processed, one causes an error that rolls back the order.

---

## Risks
- **Data Loss Risk:** Orders silently lost means customers are charged but never served.
- **Financial Risk:** Refund processes may not be triggered automatically.
- **Trust Risk:** Users lose confidence in the platform.
- **Idempotency Risk:** Retry mechanisms may create duplicate payments without duplicate orders.
- **Observability Gap:** Silent failures with no error logs make bugs hard to detect in production.