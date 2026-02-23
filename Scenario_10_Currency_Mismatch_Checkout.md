# Scenario 10: Currency Mismatch Between Product Listing and Checkout

## Scenario
An e-commerce platform allows users to change currency (USD, EUR, BDT). After switching currency, product listing prices update correctly, but checkout total remains in the old currency — causing users to pay wrong amounts.

---

## Assumptions
- Currency preference is stored client-side (localStorage, app state) and/or server-side (user profile).
- The pricing engine returns prices in the requested currency.
- The checkout service has its own pricing calculation, possibly independent of the listing service.
- Payment gateway charges in the currency provided by the checkout service.

---

## Test Ideas

### Currency Propagation Testing
- Switch currency and verify the product detail page, cart, and checkout all reflect the new currency.
- Switch currency mid-session (after adding items to cart) and verify the cart recalculates correctly.
- Switch currency on the checkout page after items are in cart — does checkout update?
- Test currency switching immediately before clicking "Pay" — what currency does the payment use?

### Frontend-Backend Integration Testing
- Capture API requests on the checkout page and verify the currency parameter is included.
- Confirm the checkout API receives and uses the correct currency from the session/request.
- Test if the checkout service reads currency from the user's session vs. accepting it from the request.
- Verify the checkout summary page and the actual payment request use the same currency.

### Payment Gateway Integration
- Confirm the payment gateway receives the correct currency code in the charge request.
- Test that the charged amount and currency match what the user saw on the checkout page.
- Verify the payment gateway's response includes the currency, and it is validated against what was expected.

### Pricing Engine Validation
- Test exchange rate consistency between the listing page and checkout — are the same rates applied?
- Check if exchange rates are cached — could stale rates cause discrepancies?
- Verify rounding behavior in currency conversion is consistent between listing and checkout.

---

## Edge Cases
- User switches from USD to BDT and checkout still shows USD (frontend state not synced to checkout component).
- User switches currency after the checkout page loads — page does not react to the state change.
- Browser back-navigation to checkout after currency switch restores old cached checkout state.
- Multiple browser tabs open with different currencies — session currency is ambiguous.
- A/B testing or feature flags cause different components to use different currency sources.

---

## Risks
- **Financial Risk:** Users are charged in the wrong currency, leading to unexpected conversion fees.
- **Legal Risk:** Charging a different amount or currency than displayed may violate consumer laws.
- **Trust Risk:** Currency inconsistency makes the platform appear untrustworthy.
- **Fraud Risk:** Users may exploit the gap to pay less by switching to a weaker currency at checkout.
- **Reconciliation Risk:** Payment records in one currency vs. displayed prices in another complicates accounting.
