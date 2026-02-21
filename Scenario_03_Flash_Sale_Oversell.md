# Scenario 03: Overselling During Flash Sale (Race Condition)

## Scenario
During a massive flash sale, thousands of users add the same limited-stock product to their carts. Some users complete payment but later receive cancellation messages saying "Out of Stock," while others who paid later successfully receive the product.

---

## Assumptions
- Inventory is stored in a central database.
- The system uses concurrent API requests to reduce inventory on purchase.
- Payment and inventory deduction may be in separate transactions or services.
- The platform is designed for high throughput during flash sales.

---

## Test Ideas

### Concurrency & Race Condition Testing
- Simulate 1000+ simultaneous "add to cart" and "purchase" requests for a product with stock of 10.
- Use load testing tools (JMeter, k6, Locust) to send parallel purchase requests.
- Verify that only the exact stock quantity is sold — no more.
- Check if "check then act" inventory logic (read stock → if > 0, decrement) is unprotected.

### Inventory Locking
- Test if the database uses pessimistic locking (SELECT FOR UPDATE) during purchase.
- Test if optimistic locking (version/timestamp check) is used and properly handles conflicts.
- Verify that failed lock acquisitions return proper errors (not silent success).
- Test atomic decrement operations (e.g., UPDATE inventory SET stock = stock - 1 WHERE stock > 0).

### Transaction Atomicity
- Verify payment and inventory deduction are wrapped in a single atomic transaction.
- Test rollback: if payment fails after inventory is deducted, is stock restored?
- Test rollback: if inventory deduction fails after payment succeeds, is payment refunded?
- Check if the order confirmation is sent only after both payment and inventory are committed.

### Queue-Based Purchase Flow
- If a queue is used (e.g., order queue), test that queue processing is sequential for same-product orders.
- Verify that the queue does not allow more items than available stock to proceed to payment.

---

## Edge Cases
- Two transactions read stock = 1 simultaneously, both proceed, stock becomes -1.
- Distributed cache (Redis) used for stock count becomes out of sync with the database.
- Cart reservation does not lock inventory — user holds item but stock is not reduced until payment.
- Retry logic on payment failure re-triggers inventory deduction twice.
- A cancelled order does not properly restore inventory, leading to phantom lost stock.

---

## Risks
- **Business Risk:** Overselling damages brand reputation and leads to costly refunds.
- **Financial Risk:** Users are charged for products that cannot be fulfilled.
- **Legal Risk:** Consumer protection laws may require penalties for confirmed orders not fulfilled.
- **Inventory Integrity Risk:** Stock count drifts from actual physical inventory.
- **Scalability Risk:** Locking strategies that work at low volume may deadlock at high concurrency.
