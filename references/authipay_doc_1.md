AuthiPay Payment Gateway Integration for Hyperswitch
Overview

AuthiPay: Fiserv Payments Gateway
Website: https://www.fiserv.com/
API Documentation: https://docs.fiserv.dev/public/reference/
Description: AuthiPay (Fiserv Payments Gateway) is a robust payment processing platform supporting card payments, tokenization, 3DS authentication, and alternative payment methods. This guide details the integration of AuthiPay into Hyperswitch, an open-source payments switch written in Rust, focusing on card payment flows for production-ready implementation.

Supported Features



Feature
Supported
Notes



Card Payments
Yes
Supports Visa, Mastercard, Maestro, Amex, etc.


3DS Authentication
Yes
Supports 3DS 1.0 and 2.1, configurable via redirectAttributes


Tokenization
Yes
Supports secure card storage for recurring payments


Refunds
Yes
Full and partial refunds via secondary transactions


Disputes/Chargebacks
Yes
Handled via VoidTransaction or secondary endpoints


Webhooks
No
Not mentioned in the provided OpenAPI spec


Authentication

Method: API Key with HMAC-SHA256 signature
Credentials Required:
Api-Key: Merchant-specific key from Fiserv developer portal
Client-Request-Id: Unique UUID for request tracking and idempotency
Timestamp: Epoch timestamp in milliseconds
Message-Signature: Base64-encoded HMAC-SHA256 hash of the request


Setup Instructions:
Register on the Fiserv Developer Portal (https://docs.fiserv.dev) to obtain an Api-Key and API secret.
Generate a unique Client-Request-Id (128-bit UUID) per request.
Include the current epoch timestamp in milliseconds in the Timestamp header.
Compute the Message-Signature using the API secret and SHA256 algorithm (see Fiserv’s signature documentation).
Configure in Hyperswitch’s development.toml:[connector.authipay]
api_key = "<your_api_key>"
api_secret = "<your_api_secret>"




Example Headers:curl -X POST https://prod.emea.api.fiservapps.com/sandbox/ipp/payments-gateway/v2/payments \
-H "Content-Type: application/json" \
-H "Api-Key: <your_api_key>" \
-H "Client-Request-Id: 30dd879c-ee2f-11db-8314-0800200c9a66" \
-H "Timestamp: 1554308829345" \
-H "Message-Signature: <base64_hmac_sha256>"



API Specifications

Base URL:
Sandbox: https://prod.emea.api.fiservapps.com/sandbox/ipp/payments-gateway/v2
Production: https://prod.emea.api.fiservapps.com/ipp/payments-gateway/v2



Supported Endpoints
Authorize/Payment Intent Creation

URL: /payments
Method: POST
Request:{
  "requestType": "PaymentCardSaleTransaction",
  "transactionAmount": {
    "total": 12.04,
    "currency": "EUR"
  },
  "paymentMethod": {
    "paymentCard": {
      "number": "5424180279791732",
      "securityCode": "977",
      "expiryDate": {
        "month": "12",
        "year": "24"
      }
    }
  },
  "merchantTransactionId": "lsk23532djljff3",
  "storeId": "12345500000",
  "storedCredentials": {
    "sequence": "FIRST",
    "scheduled": true
  }
}


Required Fields:
requestType: Must be PaymentCardSaleTransaction or PaymentCardPreAuthTransaction
transactionAmount.total: Amount in minor units (e.g., 12.04 EUR = 1204 cents)
transactionAmount.currency: ISO 4217 currency code
paymentMethod.paymentCard.number: Full card number
paymentMethod.paymentCard.securityCode: CVV/CVC
paymentMethod.paymentCard.expiryDate: Month and year


Response:{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "ipgTransactionId": "123978432",
  "orderId": "123456",
  "transactionTime": 1554308829345,
  "transactionState": "CAPTURED",
  "paymentType": "CREDIT_CARD",
  "transactionOrigin": "ECOM",
  "amount": {
    "total": 12.04,
    "currency": "EUR"
  },
  "storeId": "12345500000"
}


Notes: Use PaymentCardSaleTransaction for direct payments or PaymentCardPreAuthTransaction for pre-authorizations. merchantTransactionId is optional for tracking.

Capture

URL: /payments/{transaction-id}
Method: POST
Request:{
  "requestType": "PostAuthTransaction",
  "transactionAmount": {
    "total": 12.04,
    "currency": "EUR"
  },
  "storeId": "12345500000"
}


Required Fields:
requestType: Must be PostAuthTransaction
transactionAmount.total: Amount to capture
transactionAmount.currency: Must match the original transaction


Response:{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "ipgTransactionId": "123978432",
  "orderId": "123456",
  "transactionTime": 1554308829345,
  "transactionState": "CAPTURED",
  "paymentType": "CREDIT_CARD",
  "amount": {
    "total": 12.04,
    "currency": "EUR"
  }
}


Notes: Captures a pre-authorized transaction using the ipgTransactionId.

Refund

URL: /payments/{transaction-id} or /orders/{order-id}
Method: POST
Request:{
  "requestType": "ReturnTransaction",
  "transactionAmount": {
    "total": 12.04,
    "currency": "EUR"
  },
  "storeId": "12345500000"
}


Required Fields:
requestType: Must be ReturnTransaction
transactionAmount.total: Amount to refund
transactionAmount.currency: Must match the original transaction


Response:{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "ipgTransactionId": "123978432",
  "orderId": "123456",
  "transactionTime": 1554308829345,
  "transactionState": "RETURNED",
  "paymentType": "CREDIT_CARD",
  "amount": {
    "total": 12.04,
    "currency": "EUR"
  }
}


Notes: Supports full or partial refunds. Use /orders/{order-id} for order-based refunds.

Sync/Payment Status

URL: /payments/{transaction-id}
Method: GET
Request: No body required
Required Headers:
Api-Key
Client-Request-Id
Timestamp
Message-Signature


Response:{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "ipgTransactionId": "123978432",
  "orderId": "123456",
  "transactionTime": 1554308829345,
  "transactionState": "CAPTURED",
  "paymentType": "CREDIT_CARD",
  "amount": {
    "total": 12.04,
    "currency": "EUR"
  }
}


Notes: Retrieves the current transaction state using ipgTransactionId.

Dispute Handling

URL: /payments/{transaction-id}
Method: POST
Request:{
  "requestType": "VoidTransaction",
  "storeId": "12345500000"
}


Required Fields:
requestType: Must be VoidTransaction


Response:{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "ipgTransactionId": "123978432",
  "orderId": "123456",
  "transactionTime": 1554308829345,
  "transactionState": "VOIDED",
  "paymentType": "CREDIT_CARD"
}


Notes: Used to cancel transactions before settlement, typically for disputes.

Tokenization

URL: /payment-tokens
Method: POST
Request:{
  "requestType": "PaymentCardPaymentTokenizationRequest",
  "paymentCard": {
    "number": "5424180279791732",
    "securityCode": "977",
    "expiryDate": {
      "month": "12",
      "year": "24"
    }
  },
  "storeId": "12345500000"
}


Required Fields:
requestType: Must be PaymentCardPaymentTokenizationRequest
paymentCard.number: Full card number
paymentCard.expiryDate: Month and year


Response:{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "paymentToken": {
    "value": "1235325235236",
    "function": "DEBIT",
    "cardLast4": "1732",
    "brand": "MASTERCARD",
    "expiryMonth": "12",
    "expiryYear": "24"
  },
  "requestStatus": "SUCCESS"
}


Notes: Creates a token for secure recurring payments. Use paymentToken.value for subsequent transactions.

Preprocessing Flow (Card Verification)

URL: /card-verification
Method: POST
Request:{
  "requestType": "CardVerificationRequest",
  "paymentCard": {
    "number": "5424180279791732",
    "securityCode": "977",
    "expiryDate": {
      "month": "12",
      "year": "24"
    }
  },
  "storeId": "12345500000"
}


Required Fields:
requestType: Must be CardVerificationRequest
paymentCard.number: Full card number
paymentCard.expiryDate: Month and year


Response:{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "transactionTime": 1554308829345,
  "transactionState": "VERIFIED",
  "paymentType": "CREDIT_CARD"
}


Notes: Verifies card validity before processing payments.

Cancel Flow

URL: /payments/{transaction-id}

Method: POST

Request:
{
  "requestType": "VoidTransaction",
  "storeId": "12345500000"
}


Required Fields:

requestType: Must be VoidTransaction


Response:
{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "ipgTransactionId": "123978432",
  "orderId": "123456",
  "transactionTime": 1554308829345,
  "transactionState": "VOIDED",
  "paymentType": "CREDIT_CARD"
}


Notes: Cancels a transaction before settlement.

Rate Limits: Not specified in the OpenAPI spec. Contact Fiserv support for details.

Error Codes:

400: Bad Request (invalid payload)
401: Unauthenticated (invalid Api-Key)
403: Unauthorized (insufficient permissions)
404: Not Found (invalid transaction-id or order-id)
409: Transaction Gateway Declined
422: Transaction Endpoint Declined
500: Server Error
502: Endpoint Communication Error



Payment Flows

Supported Flows:

Direct Authorization (PaymentCardSaleTransaction)
Pre-Authorization (PaymentCardPreAuthTransaction)
Tokenization-Based Authorization (PaymentTokenSaleTransaction, PaymentTokenPreAuthTransaction)
3DS Authentication (PaymentCardPayerAuthTransaction)


Flow Details:

Authorization:
Steps:
Send PaymentCardSaleTransaction or PaymentCardPreAuthTransaction to /payments.
Include required card details and transaction amount.
Store ipgTransactionId and orderId from the response.


Example:curl -X POST https://prod.emea.api.fiservapps.com/sandbox/ipp/payments-gateway/v2/payments \
-H "Content-Type: application/json" \
-H "Api-Key: <your_api_key>" \
-H "Client-Request-Id: 30dd879c-ee2f-11db-8314-0800200c9a66" \
-H "Timestamp: 1554308829345" \
-H "Message-Signature: <base64_hmac_sha256>" \
-d '{"requestType":"PaymentCardSaleTransaction","transactionAmount":{"total":12.04,"currency":"EUR"},"paymentMethod":{"paymentCard":{"number":"5424180279791732","securityCode":"977","expiryDate":{"month":"12","year":"24"}}},"merchantTransactionId":"lsk23532djljff3","storeId":"12345500000"}'




Capture:
Steps:
Use ipgTransactionId from a pre-authorized transaction.
Send PostAuthTransaction to /payments/{transaction-id}.


Example:curl -X POST https://prod.emea.api.fiservapps.com/sandbox/ipp/payments-gateway/v2/payments/123978432 \
-H "Content-Type: application/json" \
-H "Api-Key: <your_api_key>" \
-H "Client-Request-Id: 30dd879c-ee2f-11db-8314-0800200c9a66" \
-H "Timestamp: 1554308829345" \
-H "Message-Signature: <base64_hmac_sha256>" \
-d '{"requestType":"PostAuthTransaction","transactionAmount":{"total":12.04,"currency":"EUR"},"storeId":"12345500000"}'




Refund:
Steps:
Use ipgTransactionId or orderId.
Send ReturnTransaction to /payments/{transaction-id} or /orders/{order-id}.


Example:curl -X POST https://prod.emea.api.fiservapps.com/sandbox/ipp/payments-gateway/v2/payments/123978432 \
-H "Content-Type: application/json" \
-H "Api-Key: <your_api_key>" \
-H "Client-Request-Id: 30dd879c-ee2f-11db-8314-0800200c9a66" \
-H "Timestamp: 1554308829345" \
-H "Message-Signature: <base64_hmac_sha256>" \
-d '{"requestType":"ReturnTransaction","transactionAmount":{"total":12.04,"currency":"EUR"},"storeId":"12345500000"}'




Dispute Handling:
Steps:
Use ipgTransactionId to void a transaction.
Send VoidTransaction to /payments/{transaction-id}.


Example:curl -X POST https://prod.emea.api.fiservapps.com/sandbox/ipp/payments-gateway/v2/payments/123978432 \
-H "Content-Type: application/json" \
-H "Api-Key: <your_api_key>" \
-H "Client-Request-Id: 30dd879c-ee2f-11db-8314-0800200c9a66" \
-H "Timestamp: 1554308829345" \
-H "Message-Signature: <base64_hmac_sha256>" \
-d '{"requestType":"VoidTransaction","storeId":"12345500000"}'




Tokenization:
Steps:
Send PaymentCardPaymentTokenizationRequest to /payment-tokens.
Store paymentToken.value for recurring payments.


Example:curl -X POST https://prod.emea.api.fiservapps.com/sandbox/ipp/payments-gateway/v2/payment-tokens \
-H "Content-Type: application/json" \
-H "Api-Key: <your_api_key>" \
-H "Client-Request-Id: 30dd879c-ee2f-11db-8314-0800200c9a66" \
-H "Timestamp: 1554308829345" \
-H "Message-Signature: <base64_hmac_sha256>" \
-d '{"requestType":"PaymentCardPaymentTokenizationRequest","paymentCard":{"number":"5424180279791732","securityCode":"977","expiryDate":{"month":"12","year":"24"}},"storeId":"12345500000"}'




3DS Handling:
Steps:
Send PaymentCardPayerAuthTransaction to /payments with redirectAttributes.
If redirectURL is returned, redirect the user to complete 3DS authentication.
Update the transaction via /payments/{transaction-id} with Secure3DAuthenticationUpdateRequest.


Request (Initial 3DS):{
  "requestType": "PaymentCardPayerAuthTransaction",
  "transactionAmount": {
    "total": 12.04,
    "currency": "EUR"
  },
  "paymentMethod": {
    "paymentCard": {
      "number": "5424180279791732",
      "securityCode": "977",
      "expiryDate": {
        "month": "12",
        "year": "24"
      }
    }
  },
  "redirectAttributes": {
    "authenticateTransaction": true,
    "challengeIndicator": "01",
    "browserJavaScriptEnabled": true,
    "browserJavaEnabled": true,
    "threeDSEmvCoMessageCategory": "01"
  },
  "storeId": "12345500000"
}


Response (Initial 3DS):{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "ipgTransactionId": "123978432",
  "orderId": "123456",
  "transactionTime": 1554308829345,
  "transactionState": "PENDING",
  "redirectURL": "https://api.firstdata.com/connect/gateway/processing?storename=123456789&oid=R-96cdbaa4-c22e-4598-a2f1-c2b5fed79ef1&redirectRequestId=d3eb74fe-cf63-47e1-b89f-52ba0cc7965c"
}


Request (3DS Update):{
  "requestType": "Secure3DAuthenticationUpdateRequest",
  "authenticationResult": {
    "status": "SUCCESS",
    "cavv": "j9...Pz",
    "eci": "05",
    "authenticationId": "123456"
  }
}


Response (3DS Update):{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0bd552c12342d3448-b-ea-1142-12938318-7",
  "ipgTransactionId": "123978432",
  "orderId": "123456",
  "transactionTime": 1554308829345,
  "transactionState": "CAPTURED"
}


Notes: Supports 3DS 1.0 and 2.1. challengeIndicator values: 01 (no preference), 02 (no challenge), 03 (challenge preferred), 04 (challenge mandated).


Currency Unit: Amounts are in minor units (e.g., 12.04 EUR = 1204 cents). Implement Hyperswitch’s get_currency_unit to handle conversions.



Webhooks

Supported: No
Event Types: Not applicable
Payload Structure: Not applicable
Signature Verification: Not applicable
Setup Instructions: Not applicable
Notes: The OpenAPI spec does not mention webhooks. Contact Fiserv support to confirm availability.

Configuration

Required Parameters:
api_key: Merchant API key
api_secret: Secret for HMAC signatures
store_id: Outlet ID for multi-store merchants


Supported Currencies: EUR, GBP, USD, etc. (use /available-currencies to retrieve full list)
Supported Countries: US, EU countries, etc. (use /available-iso-countries for full list)
Supported Card Networks: Visa, Mastercard, Maestro, Amex (use /card-information for full list)
Additional Settings:
Idempotency Keys: Use Client-Request-Id for idempotency.
Custom Metadata: Support for merchantTransactionId and orderId.
3DS Configuration: Use redirectAttributes for 3DS preferences.



Hyperswitch Compatibility

ConnectorCommon Trait:
Implement api_key, api_secret, and base_url.


ConnectorIntegration Trait:
Map PaymentCardSaleTransaction to PaymentAuthorize.
Map PostAuthTransaction to PaymentCapture.
Map ReturnTransaction to PaymentRefund.
Map VoidTransaction to PaymentCancel.
Map PaymentCardPaymentTokenizationRequest to PaymentTokenization.
Map CardVerificationRequest to PaymentPreprocessing.
Implement get_currency_unit for minor unit conversions.


Notes: Follow Hyperswitch’s add_connector.md for Rust implementation. Use Stripe or Adyen connectors as templates.

Missing Information and Follow-Up Actions

Rate Limits: Not specified. Contact Fiserv support or check the developer portal.
Webhook Support: Verify with Fiserv for real-time transaction notifications.
Dispute Handling: Confirm detailed chargeback processes with Fiserv support.
Currency/Card Lists: Use /available-currencies and /card-information endpoints for complete lists.

