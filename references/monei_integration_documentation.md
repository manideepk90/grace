# Monei Integration Documentation

We are integrating **MONEI** into **Hyperswitch** – an open-source payment orchestrator. This documentation provides complete technical details for integrating MONEI’s Payments API with Hyperswitch, focusing exclusively on card payments. It covers API flows, request/response structures, URLs, and authentication details based on the provided MONEI API information.

## Connector URLs

**Urls**
- **baseUrl**: `https://api.monei.com` (Production Base URL for MONEI Payments API)
- **sandboxUrl**: Not explicitly provided in the documentation. MONEI supports a test mode (indicated by `livemode: false` in responses), which uses the same base URL. Sandbox access requires a test API key configured via the MONEI dashboard.
- **Other Important URLs**:
  - **connect_url**: Not applicable. MONEI does not require a specific account connection URL; configuration is done via the MONEI dashboard using an API key.
  - **token_url**: Not applicable. MONEI does not use OAuth2; authentication is handled via API key. Payment tokens for card payments are generated using `monei.js` on the frontend or via the `generatePaymentToken` parameter in the Create Payment API.
  - **documentation_url**: `https://docs.monei.com/` (Official MONEI API documentation)
  - **status_url**: Not explicitly provided. Merchants can monitor service status via the MONEI dashboard or contact MONEI support.
  - **callbackUrl**: Configurable per payment (e.g., `https://example.com/checkout/callback`). Specified in the Create Payment request to receive asynchronous payment status updates via webhook.
  - **completeUrl**: Configurable per payment (e.g., `https://example.com/checkout/complete`). Redirect URL after transaction completion.
  - **failUrl**: Configurable per payment (e.g., `https://example.com/checkout/fail`). Redirect URL for failed transactions.
  - **cancelUrl**: Configurable per payment (e.g., `https://example.com/checkout/cancel`). Redirect URL if the customer cancels the payment.

## Authentication

- **Authentication Type**: API Key (Bearer Token)
- **Steps to Configure**:
  1. Log in to the MONEI dashboard.
  2. Navigate to the API section to generate an API key for production or test mode.
  3. Store the API key securely and include it in the `Authorization` header as a Bearer token for all API requests.
  4. For card payments, use `monei.js` on the frontend to generate temporary `paymentToken`s, ensuring PCI compliance by avoiding direct handling of card details.
- **Example Authentication Headers**:
  ```http
  Authorization: Bearer <API_KEY>
  Content-Type: application/json
  Accept: application/json
  ```
- **Example Curl Command** (for Create Payment):
  ```bash
  curl -L 'https://api.monei.com/v1/payments' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <API_KEY>' \
  --data-raw '{...}'
  ```

## Supported Flows with Request/Response Structures

### 1. Authorize / Payment Intent Creation
- **Endpoint URL**: `https://api.monei.com/v1/payments`
- **HTTP Method**: POST
- **Required Headers**:
  ```http
  Content-Type: application/json
  Accept: application/json
  Authorization: Bearer <API_KEY>
  ```
- **Request Payload**:
  ```json
  {
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "callbackUrl": "https://example.com/checkout/callback",
    "completeUrl": "https://example.com/checkout/complete",
    "failUrl": "https://example.com/checkout/fail",
    "cancelUrl": "https://example.com/checkout/cancel",
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "sessionId": "39603551437913",
    "generatePaymentToken": false,
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "allowedPaymentMethods": ["card"],
    "transactionType": "AUTH",
    "description": "Test Shop - #84370745531439",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "metadata": {
      "systemId": "12345"
    }
  }
  ```
- **Response Payload**:
  ```json
  {
    "id": "af6029f80f5fc73a8ad2753eea0b1be0",
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "description": "Test Shop - #84370745531439",
    "accountId": "aa9333ba-82de-400c-9ae7-087b9f8d2242",
    "authorizationCode": "475816",
    "livemode": false,
    "status": "AUTHORIZED",
    "statusCode": "E000",
    "statusMessage": "Transaction approved",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "refundedAmount": null,
    "lastRefundAmount": null,
    "lastRefundReason": null,
    "cancellationReason": null,
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "metadata": {
      "systemId": "12345"
    },
    "createdAt": 1636366897,
    "updatedAt": 1636366897
  }
  ```
- **Curl**:
  ```bash
  curl -L 'https://api.monei.com/v1/payments' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <API_KEY>' \
  --data-raw '{
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "callbackUrl": "https://example.com/checkout/callback",
    "completeUrl": "https://example.com/checkout/complete",
    "failUrl": "https://example.com/checkout/fail",
    "cancelUrl": "https://example.com/checkout/cancel",
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "sessionId": "39603551437913",
    "generatePaymentToken": false,
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "allowedPaymentMethods": ["card"],
    "transactionType": "AUTH",
    "description": "Test Shop - #84370745531439",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "metadata": {
      "systemId": "12345"
    }
  }'
  ```

### 2. Capture
- **Endpoint URL**: `https://api.monei.com/v1/payments/:id/capture`
- **HTTP Method**: POST
- **Required Headers**:
  ```http
  Content-Type: application/json
  Accept: application/json
  Authorization: Bearer <API_KEY>
  ```
- **Request Payload**:
  ```json
  {
    "amount": 110
  }
  ```
- **Response Payload**:
  ```json
  {
    "id": "af6029f80f5fc73a8ad2753eea0b1be0",
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "description": "Test Shop - #84370745531439",
    "accountId": "aa9333ba-82de-400c-9ae7-087b9f8d2242",
    "authorizationCode": "475816",
    "livemode": false,
    "status": "SUCCEEDED",
    "statusCode": "E000",
    "statusMessage": "Transaction approved",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "refundedAmount": null,
    "lastRefundAmount": null,
    "lastRefundReason": null,
    "cancellationReason": null,
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "metadata": {
      "systemId": "12345"
    },
    "createdAt": 1636366897,
    "updatedAt": 1636366897
  }
  ```
- **Curl**:
  ```bash
  curl -L 'https://api.monei.com/v1/payments/af6029f80f5fc73a8ad2753eea0b1be0/capture' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <API_KEY>' \
  -d '{
    "amount": 110
  }'
  ```

### 3. Refund
- **Endpoint URL**: `https://api.monei.com/v1/payments/:id/refund`
- **HTTP Method**: POST
- **Required Headers**:
  ```http
  Content-Type: application/json
  Accept: application/json
  Authorization: Bearer <API_KEY>
  ```
- **Request Payload**:
  ```json
  {
    "amount": 110,
    "refundReason": "requested_by_customer"
  }
  ```
- **Response Payload**:
  ```json
  {
    "id": "af6029f80f5fc73a8ad2753eea0b1be0",
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "description": "Test Shop - #84370745531439",
    "accountId": "aa9333ba-82de-400c-9ae7-087b9f8d2242",
    "authorizationCode": "475816",
    "livemode": false,
    "status": "REFUNDED",
    "statusCode": "E000",
    "statusMessage": "Transaction approved",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "refundedAmount": 110,
    "lastRefundAmount": 110,
    "lastRefundReason": "requested_by_customer",
    "cancellationReason": null,
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "metadata": {
      "systemId": "12345"
    },
    "createdAt": 1636366897,
    "updatedAt": 1636366897
  }
  ```
- **Curl**:
  ```bash
  curl -L 'https://api.monei.com/v1/payments/af6029f80f5fc73a8ad2753eea0b1be0/refund' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <API_KEY>' \
  -d '{
    "amount": 110,
    "refundReason": "requested_by_customer"
  }'
  ```

### 4. Sync / Psync
- **Endpoint URL**: `https://api.monei.com/v1/payments/:id`
- **HTTP Method**: GET
- **Required Headers**:
  ```http
  Accept: application/json
  Authorization: Bearer <API_KEY>
  ```
- **Request Payload**: None (GET request)
- **Response Payload**:
  ```json
  {
    "id": "af6029f80f5fc73a8ad2753eea0b1be0",
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "description": "Test Shop - #84370745531439",
    "accountId": "aa9333ba-82de-400c-9ae7-087b9f8d2242",
    "authorizationCode": "475816",
    "livemode": false,
    "status": "SUCCEEDED",
    "statusCode": "E000",
    "statusMessage": "Transaction approved",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "refundedAmount": null,
    "lastRefundAmount": null,
    "lastRefundReason": null,
    "cancellationReason": null,
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "metadata": {
      "systemId": "12345"
    },
    "createdAt": 1636366897,
    "updatedAt": 1636366897
  }
  ```
- **Curl**:
  ```bash
  curl -L 'https://api.monei.com/v1/payments/af6029f80f5fc73a8ad2753eea0b1be0' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <API_KEY>'
  ```

### 5. Dispute Handling
- **Note**: The provided MONEI API documentation does not explicitly detail a dedicated endpoint for dispute handling (e.g., chargebacks). Disputes are typically managed through the MONEI dashboard or by contacting MONEI support. For Hyperswitch integration, dispute handling may rely on webhook notifications (via `callbackUrl`) to receive updates on status changes (e.g., `status: REFUNDED` with `lastRefundReason: fraudulent`). Below is a placeholder structure assuming disputes are handled via the Refund endpoint with a `fraudulent` reason.
- **Endpoint URL**: `https://api.monei.com/v1/payments/:id/refund`
- **HTTP Method**: POST
- **Required Headers**:
  ```http
  Content-Type: application/json
  Accept: application/json
  Authorization: Bearer <API_KEY>
  ```
- **Request Payload**:
  ```json
  {
    "amount": 110,
    "refundReason": "fraudulent"
  }
  ```
- **Response Payload**:
  ```json
  {
    "id": "af6029f80f5fc73a8ad2753eea0b1be0",
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "description": "Test Shop - #84370745531439",
    "accountId": "aa9333ba-82de-400c-9ae7-087b9f8d2242",
    "authorizationCode": "475816",
    "livemode": false,
    "status": "REFUNDED",
    "statusCode": "E000",
    "statusMessage": "Transaction approved",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "refundedAmount": 110,
    "lastRefundAmount": 110,
    "lastRefundReason": "fraudulent",
    "cancellationReason": null,
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "metadata": {
      "systemId": "12345"
    },
    "createdAt": 1636366897,
    "updatedAt": 1636366897
  }
  ```
- **Curl**:
  ```bash
  curl -L 'https://api.monei.com/v1/payments/af6029f80f5fc73a8ad2753eea0b1be0/refund' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  -H 'Authorization: Bearer <API_KEY>' \
  -d '{
    "amount": 110,
    "refundReason": "fraudulent"
  }'
  ```

### 6. Tokenization / Vaulting
- **API for Tokenizing Payment Methods**: Tokenization is handled via the Create Payment or Confirm Payment endpoints by setting `generatePaymentToken: true`. This generates a permanent `paymentToken` for card payments, enabling one-click checkout for returning customers.
- **Endpoint URL**: `https://api.monei.com/v1/payments` (or `/payments/:id/confirm` for deferred payments)
- **HTTP Method**: POST
- **Required Headers**:
  ```http
  Content-Type: application/json
  Accept: application/json
  Authorization: Bearer <API_KEY>
  ```
- **Request Payload** (for Create Payment with tokenization):
  ```json
  {
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "callbackUrl": "https://example.com/checkout/callback",
    "completeUrl": "https://example.com/checkout/complete",
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "sessionId": "39603551437913",
    "generatePaymentToken": true,
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "allowedPaymentMethods": ["card"],
    "transactionType": "SALE",
    "description": "Test Shop - #84370745531439",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "metadata": {
      "systemId": "12345"
    }
  }
  ```
- **Response Payload**:
  ```json
  {
    "id": "af6029f80f5fc73a8ad2753eea0b1be0",
    "amount": 110,
    "currency": "EUR",
    "orderId": "14379133960355",
    "description": "Test Shop - #84370745531439",
    "accountId": "aa9333ba-82de-400c-9ae7-087b9f8d2242",
    "authorizationCode": "475816",
    "livemode": false,
    "status": "SUCCEEDED",
    "statusCode": "E000",
    "statusMessage": "Transaction approved",
    "customer": {
      "email": "john.doe@example.com",
      "name": "John Doe"
    },
    "billingDetails": {
      "name": "John Doe",
      "email": "john.doe@example.com",
      "address": {
        "country": "ES",
        "city": "Málaga",
        "line1": "Fake Street 123",
        "zip": "1234",
        "state": "Málaga"
      }
    },
    "refundedAmount": null,
    "lastRefundAmount": null,
    "lastRefundReason": null,
    "cancellationReason": null,
    "paymentToken": "7cc38b08ff471ccd313ad62b23b9f362b107560b",
    "paymentMethod": {
      "card": {
        "cardholderName": "John Doe",
        "cardholderEmail": "john.doe@monei.com"
      }
    },
    "metadata": {
      "systemId": "12345"
    },
    "createdAt": 1636366897,
    "updatedAt": 1636366897
  }
  ```
- **Supported Tokens or Vault Options**:
  - MONEI supports permanent `paymentToken`s for card payments, which do not expire and are used server-side for recurring payments or one-click checkouts.
  - Temporary `paymentToken`s are generated client-side using `monei.js` Components, requiring a `sessionId` for validation.
  - Tokens are linked to the card payment method and can be reused for subsequent payments.

## Configuration & Setup

- **Required Configuration Parameters**:
  - **API Key**: Obtained from the MONEI dashboard for authentication.
  - **AccountId**: Unique MONEI account identifier (e.g., `aa9333ba-82de-400c-9ae7-087b9f8d2242`).
  - **Merchant ID (MID)**: Required for payment processor configuration, verified via the MONEI dashboard.
  - **callbackUrl**: Webhook URL for asynchronous payment status updates.
- **Environment-Specific Variables**:
  - **Sandbox**: Use a test API key with `livemode: false`. All endpoints use the same base URL (`https://api.monei.com/v1`). Enable test mode in the MONEI dashboard.
  - **Production**: Use a production API key with `livemode: true`. Ensure payment processor configuration is complete in the MONEI dashboard.
- **Supported Currencies**: Any ISO 4217 three-letter currency code (e.g., EUR, USD), subject to MONEI’s supported currencies. Verify via the MONEI dashboard or support.
- **Supported Countries**: Not explicitly listed; MONEI supports global card payments, but specific restrictions may apply based on the merchant’s configuration.
- **Supported Card Networks**: Includes major networks (e.g., Visa, MasterCard). AMEX may not be supported by some processors (see error code E508).
- **Supported Payment Methods**: Card payments (focus of this integration). Other methods (e.g., Bizum, PayPal) are supported but excluded here.

## Additional Information

- **Steps for Enabling PCI-Compliant Integration**:
  1. **Use monei.js for Card Input**: Implement `monei.js` Components on the frontend to collect card details securely. This generates a temporary `paymentToken`, ensuring no sensitive card data (e.g., card number, CVC) is handled by your server.
  2. **Secure API Key Storage**: Store the MONEI API key securely in your backend environment (e.g., using environment variables or a secrets manager). Never expose it in client-side code.
  3. **HTTPS for All Requests**: Ensure all API calls and webhook endpoints (`callbackUrl`, `completeUrl`, etc.) use HTTPS to encrypt data in transit.
  4. **Validate SessionId**: When using temporary `paymentToken`s, include the corresponding `sessionId` in the Create Payment or Confirm Payment request to prevent tampering.
  5. **Enable 3D Secure**: MONEI supports 3D Secure for card payments (see error codes E300–E309). Ensure your merchant account is configured for 3D Secure to comply with Strong Customer Authentication (SCA) requirements.
  6. **Webhook Security**: Validate incoming webhook notifications (via `callbackUrl`) by checking the payment `id` and `statusCode` to ensure authenticity.
  7. **PCI DSS Compliance**: Since `monei.js` handles card data client-side, your backend avoids direct card data processing, reducing PCI DSS scope. Ensure compliance with PCI DSS requirements for any stored payment tokens or customer data.
  8. **Test in Sandbox**: Use MONEI’s test mode (`livemode: false`) with test card numbers (available in MONEI’s documentation) to validate the integration before going live.