# Scenario 08: Push Notifications Arrive Late or Not At All

## Scenario
Push notifications arrive immediately for some users, but for others they arrive hours later or not at all. The issue is inconsistent across devices, OS versions, and internet speeds.

---

## Assumptions
- The platform uses FCM (Firebase Cloud Messaging) and/or APNs (Apple Push Notification Service).
- A backend queue (e.g., SQS, RabbitMQ) manages notification dispatch.
- Notifications may be sent from multiple services or a centralized notification service.
- Device battery optimization or background restrictions may affect delivery.

---

## Test Ideas

### Device State Testing
- Test notification delivery with the app in foreground, background, and killed/closed state.
- Test on devices with battery saver or data saver mode enabled.
- Test on Android devices from various manufacturers (Samsung, Xiaomi, OnePlus) known for aggressive background restrictions.
- Verify that FCM/APNs tokens are refreshed correctly when the app is reinstalled.

### OS Version Testing
- Test across Android 10, 11, 12, 13, 14 — background processing rules changed significantly.
- Test across iOS 14, 15, 16, 17 — APNs behavior and silent push handling varies.
- Check if notification permissions were granted on the test devices.
- Verify the app correctly handles notification channel priority on Android 8+.

### Backend Queue Testing
- Monitor the backend notification queue for message accumulation (queue backup under load).
- Check queue consumer scaling — does it process all messages timely under high volume?
- Verify message TTL (time-to-live) settings — are notifications expiring before delivery?
- Check if retry logic is in place for failed deliveries to FCM/APNs.

### Network & Latency Testing
- Test on slow connections (3G simulation) and check if notification arrives after reconnect.
- Verify FCM/APNs unreachable behavior — does the backend retry when the service is temporarily down?
- Check notification timestamps: when was it enqueued vs when was it delivered?

---

## Edge Cases
- User's device was offline for hours — all queued notifications arrive simultaneously on reconnect.
- FCM/APNs token becomes invalid (user reinstalled app) but backend still uses old token.
- Notification priority set to "low" — OS delays delivery aggressively.
- High volume event triggers thousands of notifications at once, causing queue backlog.
- Notification collapse key causes multiple notifications to be replaced by one.
- Time zone handling errors cause notifications to be scheduled for the wrong time.

---

## Risks
- **UX Risk:** Delayed notifications make the app feel unreliable.
- **Business Risk:** Time-sensitive notifications (OTPs, order updates) lose value when delayed.
- **Token Management Risk:** Stale FCM/APNs tokens never cleaned up — wasted delivery attempts.
- **Scalability Risk:** Notification system not designed for peak load causes cascading delays.
- **Platform Risk:** Dependency on FCM/APNs means third-party outages affect delivery.
