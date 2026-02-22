# Scenario 09: User Auto-Logged In After Logout and App Close

## Scenario
A user logs out of the mobile app and closes it completely. When reopening the app later, they are automatically logged in again without entering credentials.

---

## Assumptions
- The app uses JWT or session tokens for authentication.
- Tokens are stored in local device storage (SharedPreferences, Keychain, AsyncStorage).
- Logout should invalidate the session server-side and clear all local tokens.
- The app checks for a valid token on startup to determine login state.

---

## Test Ideas

### Session & Token Invalidation
- After logout, verify the server-side session is invalidated (token blacklisted or deleted).
- After logout, try making an authenticated API call with the old token — it should return 401.
- Verify the logout API call actually completes before the app clears local storage.
- Test logout when the device is offline — does it properly clear local tokens?

### Local Storage Clearance
- After logout, inspect device storage (Keychain/SharedPreferences) to confirm tokens are deleted.
- Check if refresh tokens are also deleted on logout, not just access tokens.
- Verify that "Remember Me" or biometric login tokens are also cleared on explicit logout.
- Test if closing the app (before logout) and reopening does NOT auto-login — it should not.

### Auto-Login Logic Testing
- Review the app startup flow: what conditions trigger auto-login?
- Test if a persistent/refresh token remains after logout and is being used to re-authenticate.
- Verify that token expiry checks are enforced on startup.
- Check if a separate credential store (OS-level SSO, Google Account) is being used to bypass app logout.

### Security Testing
- Test on a shared/rooted device whether tokens can be extracted and reused.
- Verify tokens are stored in secure storage (not plain SharedPreferences on Android).
- Test session behavior after force-stopping the app vs proper logout.

---

## Edge Cases
- Network error during logout prevents the server-side session from being invalidated.
- The app caches the user profile and auto-populates UI, giving the appearance of being logged in.
- OS-level auto-fill or password manager re-injects credentials on app open.
- A background refresh token exchange completed just before logout, creating a new valid session.
- Multiple devices logged in — logout from one device does not invalidate other sessions.

---

## Risks
- **Security Risk:** Unauthorized access if the device is shared, lost, or stolen.
- **Privacy Risk:** Personal data accessible without authentication.
- **Compliance Risk:** Financial and healthcare apps have strict session management requirements (PCI-DSS, HIPAA).
- **Regulatory Risk:** GDPR requires users to have control over their sessions.
- **Trust Risk:** Users who explicitly log out expect their session to be terminated completely.
