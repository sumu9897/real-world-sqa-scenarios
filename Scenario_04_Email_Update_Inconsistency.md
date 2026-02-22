# Scenario 04: Email Update Not Reflected Across All Systems

## Scenario
A user updates their email address in their profile successfully. The UI shows the new email everywhere. However, password reset links and promotional emails are still being sent to the old email address days later.

---

## Assumptions
- The platform uses multiple services: User Profile Service, Auth/Password Service, Email Marketing Service (e.g., Mailchimp, SendGrid).
- Email data may be cached at various layers.
- Third-party email services may sync user data asynchronously.
- The user profile update triggers an event or API call to downstream systems.

---

## Test Ideas

### Data Consistency Validation
- After updating email, trigger a password reset and verify it goes to the new email.
- Subscribe/unsubscribe from promotional emails and check which address receives them.
- Query all databases/services directly to confirm the new email is stored everywhere.
- Check the Auth Service database separately — does it have its own email field?

### Cache Invalidation Testing
- After updating email, immediately trigger a password reset — does the old email receive it?
- Check if cache TTL (time-to-live) is causing the old email to be served for a period.
- Force cache invalidation and re-test to confirm the cache was the root cause.
- Test with cache disabled to confirm the underlying data is correct.

### Third-Party Email Service Sync
- Check if the CRM/email marketing tool syncs via webhook, scheduled job, or real-time API.
- Verify the sync job runs correctly and maps the right fields.
- Check the sync logs for errors or skipped records.
- Manually trigger a sync and verify promotional emails switch to the new address.

### Event/Notification Propagation
- Verify that the profile update event is published to all subscribing services.
- Check if the Password Service and Email Service are subscribed to the profile update event.
- Test what happens if the event is published but a service fails to process it.

---

## Edge Cases
- The user updates the email but does not verify the new email — which email is used?
- A scheduled promotional email job was queued before the update — uses the old email.
- Multiple profile updates in quick succession cause race conditions in event processing.
- Third-party email service has a rate limit causing sync delays.
- The old email address is stored in a separate analytics/tracking system not updated.

---

## Risks
- **Security Risk:** Password reset sent to old email could allow account takeover by previous owner.
- **Privacy Risk:** If email was transferred to another person, they receive private communications.
- **Compliance Risk:** GDPR/data protection laws require accurate personal data across all systems.
- **User Trust Risk:** Users feel the platform is unreliable when changes don't propagate.
- **Operational Risk:** Customer support cannot identify users correctly if emails are inconsistent.
