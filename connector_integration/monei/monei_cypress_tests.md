# Monei Connector Cypress Tests Implementation

## Completed Tasks ✓

- [x] **Created Monei configuration file**: `cypress-tests/cypress/e2e/configs/Payment/Monei.js`
  - Defined test card details
  - Configured test data for various payment flows
  - Set up expected response templates for different scenarios

- [x] **Updated Utils.js**: Added Monei to the connector list
  - Imported Monei connector details
  - Added to connectorDetails object

- [x] **Created Monei connector test file**: `cypress-tests/cypress/e2e/spec/Payment/connectors/Monei.cy.js`
  - Implemented account setup tests
  - Added No3DS Auto Capture payment flow tests
  - Added No3DS Manual Capture payment flow tests
  - Added refund flow tests
  - Added partial refund flow tests
  - Added partial capture flow tests
  - Added saved card flow tests

## Test Coverage

The following payment flows are covered in the Monei connector tests:

### Basic Payment Flows
1. **Card Payment - No 3DS with Auto Capture**
   - Creates payment intent
   - Lists payment methods
   - Confirms payment
   - Verifies payment status

2. **Card Payment - No 3DS with Manual Capture**
   - Creates payment intent
   - Confirms payment
   - Captures payment
   - Verifies payment status

### Refund Flows
3. **Full Refund**
   - Creates and confirms payment
   - Refunds payment
   - Verifies refund status

4. **Partial Refund**
   - Creates and confirms payment
   - Partially refunds payment
   - Verifies refund status

### Capture Flows
5. **Partial Capture**
   - Creates payment intent
   - Confirms payment
   - Partially captures payment
   - Verifies payment status

### Saved Card Flows
6. **Save Card for Future Use**
   - Creates customer
   - Creates payment with saved card setup
   - Confirms payment with saved card
   - Verifies payment status

## Test Execution

To run the Monei connector tests:

1. Ensure the Monei connector implementation is complete according to `monei_plan.md`
2. Add Monei test credentials to the test configuration
3. Run the tests with:

```bash
CYPRESS_CONNECTOR="monei" npm run cypress:run
```

## Notes

- The tests expect the Monei connector to be fully implemented as specified in the `monei_plan.md` and `monei_specs.md` documents.
- The currency used in tests is EUR, as specified in the Monei technical specs.
- Test card numbers are chosen to match Monei's test environment expectations.
- Both authentication flows (No 3DS) and authorization flows (manual capture) are tested to ensure complete coverage.
- Partial operations (capture, refund) are tested to verify support for partial amount processing.
