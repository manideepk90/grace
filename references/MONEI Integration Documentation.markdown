Monei Payment Gateway Integration for Hyperswitch
Key Points:

Monei is a European-focused payment gateway supporting card payments, digital wallets, and local methods like Bizum, with a REST API for integration.
It seems likely that Monei supports key payment flows such as authorization, capture, refunds, and tokenization, aligning with Hyperswitch’s requirements.
Authentication uses API keys, easily obtainable from the Monei Dashboard, ensuring straightforward setup.
Webhooks and 3D Secure are supported, enhancing security and real-time payment updates.
Some details, like specific request parameters for authorization vs. capture, are unclear and may require contacting Monei support for clarification.
Overview
Monei is a payment gateway designed for European businesses, offering a robust platform to process card payments, digital wallets, and local payment methods like Bizum. Its REST API allows seamless integration into Hyperswitch, an open-source payments switch written in Rust. This guide outlines the technical steps to integrate Monei as a new connector, focusing on card payment processing, based on available documentation and standard practices.

How to Integrate
To integrate Monei into Hyperswitch, developers need to:

Obtain API Credentials: Sign up at Monei’s website and retrieve the API key from the Monei Dashboard under Settings → API Access.
Configure Hyperswitch: Add the API key to Hyperswitch’s configuration (e.g., development.toml) and implement the connector using Rust traits like ConnectorCommon and ConnectorIntegration.
Implement Payment Flows: Use Monei’s API endpoints for authorization, capture, refunds, and tokenization, ensuring proper handling of 3D Secure and webhooks.
Test Thoroughly: Use Monei’s test mode with provided test card numbers to verify the integration before going live.
What You Need to Know
Monei’s API uses a base URL of https://api.monei.com/v1 and supports card payments with 3D Secure v2.1, tokenization for recurring payments, and webhooks for real-time updates. Amounts are in minor currency units (e.g., cents for EUR), aligning with Hyperswitch’s conventions, so no conversion is needed. However, specific parameters for distinguishing authorization from capture (e.g., transactionType) are not fully documented and may require further clarification from Monei’s support team.

Next Steps
Due to incomplete endpoint details in the available documentation, developers should contact Monei support at support@monei.com to obtain the full API specification, including request/response schemas and webhook event types. Reviewing Hyperswitch’s add_connector.md on GitHub will also guide the implementation process.

Detailed Technical Integration Guide for Monei in Hyperswitch
This report provides a comprehensive guide for integrating the Monei payment gateway as a new connector into the Hyperswitch repository, focusing on card payment processing. It aligns with Hyperswitch’s connector architecture and Rust-based implementation requirements, as outlined in the add_connector.md documentation. The information is derived from Monei’s official resources, Hyperswitch’s guidelines, and related integrations like Gringotts, with notes on missing details and follow-up actions.

Overview
Full Name: Monei Payment Platform
Website: Monei Official Website
API Documentation: Monei API Documentation
Description: Monei is a payment gateway tailored for the European market, offering an AWS EC2 PCI-compliant infrastructure. It supports a wide range of payment methods, including credit/debit cards, digital wallets (e.g., Apple Pay, Google Pay), and local methods like Bizum. Its REST API facilitates integration with e-commerce platforms and custom websites, making it suitable for Hyperswitch’s modular payment orchestration. Monei also supports GraphQL for advanced features, but this guide focuses on the REST API for compatibility with Hyperswitch’s connector architecture.
Supported Features
Monei supports essential payment processing features required for Hyperswitch integration, particularly for card payments. The table below summarizes these features based on available documentation and third-party integrations like Gringotts.


Feature	Supported	Notes
Card Payments	Yes	Supports Visa, Mastercard, JCB, Diners Club, Discover
3DS Authentication	Yes	Supports 3D Secure v2.1 (Challenge, Frictionless, Direct)
Tokenization	Yes	Stores card details for recurring payments using /registrations endpoint
Refunds	Yes	Supports full and partial refunds via /payments/{id}/refund
Disputes/Chargebacks	Yes	Handled through Monei Dashboard; specific API details unclear
Webhooks	Yes	Supports events like payment succeeded, failed, refunded; uses HMAC-SHA256
Authentication
Method: API key-based authentication
Credentials Required:
API key (e.g., pk_test_... for test mode, pk_live_... for production)
Optional: Account ID for Monei Connect (partner integrations)
Setup Instructions:
Create a Monei account at Monei Signup.
Log into the Monei Dashboard and navigate to Settings → API Access.
Generate or retrieve the test and live API keys.
In Hyperswitch, configure the connector by adding the API key to the development.toml file under the Monei connector section, following the structure in Hyperswitch’s add_connector.md .
Example:
bash

Copy
curl -H "Authorization: pk_test_3c140607778e1217f56ccb8b50540e00" \
     -H "Content-Type: application/json" \
     https://api.monei.com/v1/payments
API Specifications
Base URL: https://api.monei.com/v1 (production); https://test.monei-api.net/v1 (test mode)
Supported Endpoints:
Authorize/Payment Intent Creation:
URL: POST /payments
Method: POST
Request:
json

Copy
{
  "amount": 1000,
  "currency": "EUR",
  "orderId": "14379133960355",
  "description": "Test Shop - #14379133960355",
  "customer": {
    "email": "john.doe@microapps.com"
  },
  "paymentMethod": {
    "card": {
      "number": "4200000000000000",
      "expMonth": 12,
      "expYear": 2025,
      "cvc": "123"
    }
  },
  "transactionType": "PA" // Assumed for authorization; needs confirmation
}
Response:
json

Copy
{
  "id": "pay_123456789",
  "amount": 1000,
  "currency": "EUR",
  "status": "AUTHORIZED",
  "authorizationCode": "475816",
  "nextAction": {
    "redirectUrl": "https://api.monei.com/v1/payments/pay_123456789/redirect"
  }
}
Notes: The transactionType parameter (e.g., "PA" for pre-authorization) is assumed based on PayPal’s transactionType (SALE or AUTH). Confirmation from Monei support is needed. The response may include a nextAction.redirectUrl for 3DS authentication.
Capture:
URL: POST /payments/{payment_id}/capture
Method: POST
Request:
json

Copy
{
  "amount": 1000
}
Response:
json

Copy
{
  "id": "pay_123456789",
  "status": "SUCCEEDED",
  "capturedAmount": 1000
}
Notes: Supports partial captures; requires a 32-byte payment_id.
Refund:
URL: POST /payments/{payment_id}/refund
Method: POST
Request:
json

Copy
{
  "amount": 500,
  "reason": "customer_request"
}
Response:
json

Copy
{
  "id": "pay_123456789",
  "status": "PARTIALLY_REFUNDED",
  "refundedAmount": 500
}
Notes: Supports full and partial refunds; requires a 32-byte payment_id.
Sync/Payment Status:
URL: GET /payments/{payment_id}
Method: GET
Response:
json

Copy
{
  "id": "pay_123456789",
  "status": "SUCCEEDED",
  "amount": 1000,
  "currency": "EUR"
}
Notes: Retrieves the current status of a payment (e.g., AUTHORIZED, SUCCEEDED, FAILED).
Dispute Handling:
Details: Disputes are managed via the Monei Dashboard. Specific API endpoints for disputes are not documented; contact Monei Support for details.
Tokenization:
URL: POST /registrations
Method: POST
Request:
json

Copy
{
  "card": {
    "number": "4200000000000000",
    "expMonth": 12,
    "expYear": 2025,
    "cvc": "123"
  }
}
Response:
json

Copy
{
  "id": "reg_123456789",
  "token": "tok_123456789"
}
Notes: Stores card details for recurring payments; token can be used in paymentToken field for future payments.
Preprocessing Flow:
Details: Create a payment without immediate payment details to obtain a nextAction.redirectUrl for customer input (e.g., 3DS or Bizum). Confirm the payment later using /payments/{id}/confirm.
Tokenization Flow:
Details: Use /registrations to store card details, then reference the token in payment requests for one-click or recurring payments.
Cancel Flow:
URL: POST /payments/{payment_id}/void
Method: POST
Request: Empty or minimal payload
Response:
json

Copy
{
  "id": "pay_123456789",
  "status": "CANCELED"
}
Notes: Voids an authorized payment, clearing held funds.
Rate Limits: Not explicitly documented. Contact Monei Support for details.
Error Codes:
400: Bad Request (invalid parameters)
401: Unauthorized (invalid API key)
403: Forbidden (e.g., attempting to unstore a card)
404: Not Found (invalid payment ID)
Additional codes may exist; refer to Monei’s API documentation.
Payment Flows
Supported Flows: DirectAuthorization, TokenizationBasedAuthorization, Capture, Refund, Void
Flow Details:
Authorization:
Steps:
Send a POST /payments request with card details and transactionType: "PA" (assumed).
Handle nextAction.redirectUrl for 3DS authentication if required.
Receive an AUTHORIZED status and payment ID for capture or void.
Example:
bash

Copy
curl -X POST https://api.monei.com/v1/payments \
     -H "Authorization: pk_test_3c140607778e1217f56ccb8b50540e00" \
     -H "Content-Type: application/json" \
     -d '{"amount": 1000, "currency": "EUR", "paymentMethod": {"card": {"number": "4200000000000000", "expMonth": 12, "expYear": 2025, "cvc": "123"}}, "transactionType": "PA"}'
Capture:
Steps:
Use the payment ID from authorization.
Send a POST /payments/{id}/capture request with the amount to capture.
Example:
bash

Copy
curl -X POST https://api.monei.com/v1/payments/pay_123456789/capture \
     -H "Authorization: pk_test_3c140607778e1217f56ccb8b50540e00" \
     -H "Content-Type: application/json" \
     -d '{"amount": 1000}'
Refund:
Steps:
Use the payment ID from a captured payment.
Send a POST /payments/{id}/refund request with the refund amount.
Example:
bash

Copy
curl -X POST https://api.monei.com/v1/payments/pay_123456789/refund \
     -H "Authorization: pk_test_3c140607778e1217f56ccb8b50540e00" \
     -H "Content-Type: application/json" \
     -d '{"amount": 500, "reason": "customer_request"}'
Dispute Handling:
Steps: Monitor disputes via the Monei Dashboard. API support for disputes is unclear; contact Monei Support for programmatic handling.
Tokenization:
Steps:
Send a POST /registrations request to store card details.
Use the returned token in future payment requests.
Example:
bash

Copy
curl -X POST https://api.monei.com/v1/registrations \
     -H "Authorization: pk_test_3c140607778e1217f56ccb8b50540e00" \
     -H "Content-Type: application/json" \
     -d '{"card": {"number": "4200000000000000", "expMonth": 12, "expYear": 2025, "cvc": "123"}}'
3DS Handling:
Monei supports 3D Secure v2.1 (Challenge, Frictionless, Direct).
If 3DS is required, the payment creation response includes a nextAction.redirectUrl for customer authentication.
Hyperswitch must handle redirection and parse the final status via webhooks or GET /payments/{id}.
Currency Unit: Monei uses minor units (e.g., 1000 = 10.00 EUR), matching Hyperswitch’s convention. In ConnectorCommon, set get_currency_unit to Minor.
Webhooks
Supported: Yes
Event Types: Likely include payment_succeeded, payment_failed, refund_processed, payment_authorized, payment_canceled (exact list requires confirmation from Monei).
Payload Structure:
json

Copy
{
  "id": "evt_123",
  "type": "payment_succeeded",
  "data": {
    "object": {
      "id": "pay_123456789",
      "status": "SUCCEEDED",
      "amount": 1000,
      "currency": "EUR"
    }
  }
}
Signature Verification:
Method: HMAC-SHA256
Example:
javascript

Copy
const signature = req.headers['MONEI-Signature'];
const isValid = monei.verifySignature(req.body.toString(), signature);
if (isValid) {
  // Process webhook
}
Setup Instructions:
In the Monei Dashboard, navigate to Settings → Webhooks.
Add the Hyperswitch webhook URL (e.g., https://your-hyperswitch-instance.com/webhooks/monei).
In Hyperswitch, implement a webhook handler to verify signatures using the webhook secret and process events (e.g., update payment status).
Return a 200 HTTP status code to acknowledge receipt; otherwise, Monei retries delivery.
Configuration
Required Parameters:
api_key: Obtained from Monei Dashboard
account_id: Required for Monei Connect (partner integrations)
Supported Currencies: Over 230 currencies, including EUR, USD, GBP (full list at Monei Currencies).
Supported Countries: Primarily Spain and Andorra; global expansion ongoing.
Supported Card Networks: Visa, Mastercard, JCB, Diners Club, Discover.
Additional Settings:
Idempotency Keys: Use orderId for idempotency to prevent duplicate payments.
Custom Metadata: Include in description or custom fields for reconciliation.
3DS Settings: Configure in Monei Dashboard under Settings → Payment Methods → Card Payments.
Hyperswitch Compatibility
To align with Hyperswitch’s architecture, the Monei connector must implement the following traits as per add_connector.md:

ConnectorCommon:
id: Set to "monei".
get_currency_unit: Return Minor (no conversion needed).
base_url: Return https://api.monei.com/v1 (or test URL).
get_auth_header: Construct Authorization: <api_key>.
ConnectorIntegration:
Implement get_url, get_headers, get_request_body, build_request, handle_response, and get_error_response for flows like PaymentAuthorize, PaymentCapture, PaymentSync, Refund, RefundExecute, RefundSync.
Use todo!() for unimplemented flows (e.g., disputes if not supported via API).
Testing:
Use Monei’s test mode with test card numbers (e.g., 4444444444444406 for Visa 3DS v2.1 Challenge, expiration 12/34, CVC 123).
Configure test API keys in crates/router/tests/connectors/sample_auth.toml.
Run tests using cargo test --package router --test connectors -- monei --test-threads=1.
Missing Information and Follow-Up Actions
Exact Parameter for Authorization: The transactionType parameter (e.g., "PA" for pre-authorization) is assumed based on PayPal’s implementation. Confirm with Monei support.
Webhook Event Types: The full list of webhook events is not documented; obtain from Monei’s API reference or support.
Dispute Handling API: Dispute management appears to be dashboard-based; verify if programmatic access is available.
Rate Limits: Not specified in documentation; request details from Monei support.
Action: Contact Monei Support or check the Monei API Reference for complete endpoint schemas and additional details.