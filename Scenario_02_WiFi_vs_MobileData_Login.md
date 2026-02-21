# Scenario 02: Login Works on Wi-Fi But Fails on Mobile Data

## Scenario
After releasing a new mobile app version, users report they can log in successfully when connected to Wi-Fi but fail when using mobile data. The backend team says the authentication service is working fine and sees no errors in normal logs.

---

## Assumptions
- The authentication service is accessible via both Wi-Fi and mobile data networks.
- The mobile app may have different behavior depending on network type detection.
- SSL/TLS certificates and API security rules are in place.
- The backend uses standard HTTP/HTTPS APIs for authentication.
- The issue appeared after a new app version release.

---

## Test Ideas

### Network Configuration Testing
- Test login on multiple carriers and network types (4G, 5G, LTE).
- Use a proxy tool (Charles, Fiddler) to capture requests on both Wi-Fi and mobile data.
- Compare request headers, host resolution, and IP routing on both networks.
- Check if the app uses a hardcoded IP or DNS name — DNS resolution may differ on mobile data.
- Test if carrier-level firewalls or DPI (deep packet inspection) blocks certain requests.

### Timeout Handling
- Simulate high-latency mobile conditions and observe if login requests time out silently.
- Check if the app has different timeout configurations for different network types.
- Test what happens when authentication takes longer than expected on slower networks.

### API Security Rules
- Check if the backend has IP whitelisting or rate limiting that blocks mobile carrier IPs.
- Verify if API gateway rules differ for requests coming from mobile networks vs ISPs.
- Check if SSL certificate pinning is configured and failing on mobile data.

### Client-Side Logic
- Check if the app detects network type and changes API endpoints or behavior accordingly.
- Verify if any network-type-specific code was introduced in the new release.
- Test with network type simulation on emulators (set to mobile data mode).

---

## Edge Cases
- VPN usage on mobile data changes the apparent network and may succeed where normal mobile fails.
- IPv6 only mobile networks vs IPv4 Wi-Fi causing connection differences.
- Carrier proxy modifies headers (e.g., X-Forwarded-For) that trigger backend security rules.
- Certificate pinning failure on mobile network due to carrier MITM or transparent proxy.
- The new app version has a bug in network state detection logic.

---

## Risks
- **Security Risk:** Certificate pinning issues could expose users to MITM attacks.
- **User Experience Risk:** Users on mobile data cannot log in, reducing accessibility.
- **Carrier Dependency Risk:** App behavior tied to carrier infrastructure outside dev control.
- **Regression Risk:** New release introduced a network-type-sensitive bug undetected in testing.
- **Observability Risk:** Backend sees no errors because the request never reaches the server.
