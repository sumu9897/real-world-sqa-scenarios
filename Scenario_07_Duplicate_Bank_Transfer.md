# Scenario 07: Duplicate Credit — Receiver Gets Amount Twice

## Scenario
A banking app processes money transfers and shows "Transfer Successful" instantly. Later, some users complain that the receiver received the same amount twice, while the sender's balance was deducted only once.

---

## Assumptions
- The transfer system uses an external payment processor or banking API.
- Retry logic is implemented for handling network timeouts.
- Transactions are supposed to be atomic and idempotent.
- The backend has a transaction ledger/audit log.

---

## Test Ideas

### Idempotency Testing
- Submit the same transfer request twice with the same request ID — does the second one get rejected?
- Test if the API enforces unique transaction/idempotency keys per transfer.
- Simulate a network timeout after the bank processes the transaction but before the response reaches the app — does the retry create a duplicate?
- Verify the idempotency key is stored and checked before processing any new transaction.

### Retry Mechanism Testing
- Test automatic retry logic: block the response after processing and let the retry fire.
- Check if retries use the same transaction ID or generate a new one each time.
- Verify that the retry mechanism checks transaction status before re-submitting.
- Test the behavior when retry occurs during a partial failure (DB wrote, notification failed).

### Rollback & Failure Recovery
- Simulate a crash mid-transaction and verify rollback restores both accounts to original balances.
- Test two-phase commit or saga pattern if used — verify compensation actions work.
- Test what happens if the debit succeeds but the credit fails — is debit rolled back?
- Verify idempotent recovery: replaying a failed transaction only executes once.

### Ledger & Audit Validation
- Query transaction logs to check if the duplicate credit has two separate entries or one.
- Verify that each transaction entry has a unique ID and is linked to the original request.
- Check if the accounting balance is consistent with the transaction ledger.

---

## Edge Cases
- User double-taps the "Transfer" button — two requests sent simultaneously.
- Payment processor returns a timeout but internally processed the transaction.
- A network glitch causes the app to resend the request, which the server treats as a new transaction.
- Retry logic runs after a server restart that lost in-memory idempotency cache.
- Webhook from bank is received twice (duplicate delivery) and both are processed.

---

## Risks
- **Financial Risk:** Duplicate credits cause direct monetary loss to the platform or bank.
- **Regulatory Risk:** Banking systems have strict regulations; duplicate transactions may trigger audits.
- **Legal Risk:** Users may exploit this as a bug to gain unauthorized funds.
- **Reputation Risk:** Financial errors severely damage trust in a banking application.
- **Data Integrity Risk:** Inconsistent ledger entries make financial reconciliation impossible.
