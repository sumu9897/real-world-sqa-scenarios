# Scenario 05: Rides Start Automatically Without Driver Action (GPS/Network Bug)

## Scenario
In a ride-sharing app, some rides start automatically without the driver clicking the "Start Trip" button. This only happens when GPS signal is weak or when the driver switches between mobile data and Wi-Fi.

---

## Assumptions
- The app uses GPS for location tracking and network events trigger state changes.
- The trip state machine has states: Accepted → Arrived → Started → Completed.
- Background services handle location updates and network reconnection.
- The "Start Trip" action sends an API call that updates trip state in the backend.

---

## Test Ideas

### GPS Signal Testing
- Test trip state behavior in a GPS-denied environment (underground, tunnels).
- Simulate GPS signal loss and recovery using emulator tools or GPS spoofing apps.
- Check if GPS accuracy threshold triggers any automatic state transitions.
- Verify the app does not interpret "location = pickup point" as "trip started."

### Network Reconnection Testing
- Toggle between Wi-Fi and mobile data while in "Arrived" state.
- Simulate network drop and reconnection during the pre-trip phase.
- Check if a pending or queued "Start Trip" API call fires on reconnection.
- Test if reconnection replays buffered events unintentionally.

### Background Service Testing
- Check if background location services fire events that are misinterpreted as trip start.
- Test with the app backgrounded and foregrounded during the Arrived state.
- Verify that background network state change receivers do not trigger trip state changes.

### State Machine Validation
- Audit the client-side state machine for invalid transitions (Arrived → Started without explicit action).
- Test rapid state changes: accept, arrive, then network drops — what state is restored on reconnect?
- Verify the backend validates state transitions and rejects invalid ones.

---

## Edge Cases
- Driver presses "Arrived" while GPS is weak — coordinates are inaccurate, triggering auto-start logic.
- App receives a delayed acknowledgment of an earlier "Start Trip" that was thought to have failed.
- Network switch causes the app to re-authenticate and re-sync state, pulling a stale "Started" state from the backend.
- Background process interprets geofence entry (arriving at pickup) as trip start confirmation.
- Race condition between a manual cancel and an auto-start event.

---

## Risks
- **Legal Risk:** A trip started without consent could create fraudulent billing.
- **Safety Risk:** Driver or passenger may not be ready when trip is prematurely started.
- **Financial Risk:** Riders are charged from the wrong time/location.
- **Trust Risk:** Drivers may receive negative ratings for trips they did not control.
- **State Corruption Risk:** Incorrect trip state may prevent proper completion or cancellation.
