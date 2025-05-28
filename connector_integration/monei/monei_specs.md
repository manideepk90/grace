# Monei Connector Integration Technical Specification

## 1. Connector Overview

### 1.1 Basic Information
- **Connector Name**: monei
- **Base URL**: `https://api.monei.com/v1`
- **API Documentation**: https://docs.monei.com/
- **Supported Countries**: Global (subject to merchant configuration)
- **Supported Currencies**: All ISO 4217 three-letter currency codes (EUR, USD, etc.)

### 1.2 Authentication Method
- **Type**: API Key (Bearer Token)
- **Header Format**: `Authorization: Bearer {api_key}`
- **Additional Headers**: 
  - `Content-Type: application/json`
  - `Accept: application/json`

### 1.3 Supported Features
| Feature | Supported | Notes |
|---------|-----------|-------|
| Card Payments | ✓ | Via paymentToken |
| Bank Transfers | ✗ | Not in scope |
| Wallets | ✗ | Not in scope |
| 3DS 2.0 | ✓ | Supported with error codes E300-E309 |
| Recurring Payments | ✓ | Via permanent paymentToken |
| Partial Capture | ✓ | Supported |
| Multiple Captures | ✗ | Not documented |
| Instant Refunds | ✓ | Supported |
| Partial Refunds | ✓ | Supported |
| Webhooks | ✓ | Via callbackUrl |

## 2. API Endpoints

### 2.1 Payment Operations
| Operation | Method | Endpoint | Purpose |
|-----------|---------|----------|---------|
| Create Payment | POST | /payments | Create payment with AUTH or SALE |
| Capture Payment | POST | /payments/{id}/capture | Capture authorized payment |
| Cancel Payment | POST | /payments/{id}/cancel | Cancel/void payment |
| Get Payment | GET | /payments/{id} | Retrieve payment status |

### 2.2 Refund Operations
| Operation | Method | Endpoint | Purpose |
|-----------|---------|----------|---------|
| Create Refund | POST | /payments/{id}/refund | Initiate full or partial refund |
| Get Refund | GET | /payments/{id} | Check refund status (part of payment) |

## 3. Data Models

### 3.1 Payment Request Structure
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

### 3.2 Payment Response Structure
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

### 3.3 Status Mappings
| Connector Status | Hyperswitch Status | Description |
|------------------|-------------------|-------------|
| PENDING | Processing | Payment is being processed |
| AUTHORIZED | Authorized | Payment authorized, ready for capture |
| SUCCEEDED | Succeeded | Payment completed successfully |
| FAILED | Failed | Payment failed |
| REFUNDED | Succeeded | Refund completed (check refundedAmount) |
| PARTIALLY_REFUNDED | PartiallyCaptured | Partial refund completed |
| CANCELED | Voided | Payment canceled/voided |

### 3.4 Error Code Mappings
| Connector Error Code | Hyperswitch Error | Description |
|---------------------|------------------|-------------|
| E000 | Success | Transaction approved |
| E001 | PaymentMethodNotSupported | Unauthorized card type |
| E100-E199 | AuthenticationFailed | Authentication errors |
| E200-E299 | InvalidRequestData | Request validation errors |
| E300-E309 | AuthenticationRequired | 3DS required |
| E400-E499 | InsufficientFunds | Insufficient funds |
| E500-E599 | ProcessingError | Processing errors |
| E508 | PaymentMethodNotSupported | AMEX not supported |

## 4. Implementation Details

### 4.1 Request Transformations

#### 4.1.1 Authorize Request
```rust
// Authorization request transformation
MoneiPaymentRequest {
    amount: req.request.minor_amount.get_amount_as_i64(), // Already in minor units
    currency: req.request.currency.to_string(),
    order_id: req.payment_id.clone(),
    callback_url: req.request.webhook_url.clone(),
    complete_url: req.request.complete_authorize_url.clone(),
    payment_token: get_payment_token(&req.request.payment_method_data)?,
    session_id: req.connector_request_reference_id.clone(),
    generate_payment_token: should_generate_token(&req.request),
    payment_method: transform_payment_method(&req.request.payment_method_data)?,
    allowed_payment_methods: vec!["card".to_string()],
    transaction_type: if req.request.is_auto_capture() { "SALE" } else { "AUTH" },
    description: req.description.clone(),
    customer: transform_customer_details(&req.request)?,
    billing_details: transform_billing_details(&req.address)?,
    metadata: build_metadata(&req.request)?,
}
```

#### 4.1.2 Capture Request
```rust
MoneiCaptureRequest {
    amount: req.request.minor_amount.get_amount_as_i64(), // Partial capture supported
}
```

#### 4.1.3 Refund Request
```rust
MoneiRefundRequest {
    amount: req.request.minor_refund_amount.get_amount_as_i64(),
    refund_reason: map_refund_reason(&req.request.reason),
}
```

### 4.2 Response Transformations

#### 4.2.1 Payment Response
```rust
PaymentsResponseData {
    status: map_payment_status(&response.status, &response.refunded_amount),
    response: Ok(PaymentsResponseData::TransactionResponse {
        resource_id: ResponseId::ConnectorTransactionId(response.id),
        redirection_data: build_redirection_data(&response),
        mandate_reference: None,
        connector_metadata: build_connector_metadata(&response),
        network_txn_id: None,
        connector_response_reference_id: Some(response.order_id),
        incremental_authorization_allowed: None,
        charge_id: None,
    }),
    amount_captured: get_amount_captured(&response),
}
```

### 4.3 Amount Handling
- **Format**: Minor units (cents)
- **Decimal Places**: 2 for most currencies
- **Zero Decimal Currencies**: Standard handling (JPY, KRW, etc.)

### 4.4 Payment Method Transformations
```rust
// Card payment via paymentToken
match payment_method_data {
    PaymentMethodData::Card(card) => {
        // Note: Card details are tokenized via monei.js frontend
        // Only paymentToken is sent to API
        MoneiPaymentMethod {
            card: Some(MoneiCardDetails {
                cardholder_name: card.card_holder_name.clone(),
                cardholder_email: customer_email.clone(),
            }),
        }
    }
    _ => Err(errors::ConnectorError::NotImplemented(
        "Payment method not supported".to_string(),
    )),
}
```

## 5. Webhook Implementation

### 5.1 Webhook Configuration
- **Endpoint Format**: Configured via callbackUrl per payment
- **Signature Header**: Not documented (implementation may vary)
- **Signature Algorithm**: To be determined from Monei documentation

### 5.2 Supported Events
| Event Type | Hyperswitch Event | Description |
|------------|------------------|-------------|
| payment.authorized | PaymentIntentProcessing | Payment authorized |
| payment.succeeded | PaymentIntentSuccess | Payment completed |
| payment.failed | PaymentIntentFailure | Payment failed |
| payment.refunded | RefundSuccess | Refund completed |
| payment.canceled | PaymentIntentCancelled | Payment canceled |

### 5.3 Webhook Payload
```json
{
  "id": "af6029f80f5fc73a8ad2753eea0b1be0",
  "status": "SUCCEEDED",
  "orderId": "14379133960355",
  // Full payment object as response
}
```

## 6. Error Handling

### 6.1 API Error Response Format
```json
{
  "code": 400,
  "message": "Bad Request",
  "error": "Invalid payment token"
}
```

### 6.2 Error Handling Strategy
- Map statusCode to appropriate Hyperswitch errors
- Use statusMessage for detailed error information
- Handle network timeouts with appropriate retry
- Log all errors with connector_response

## 7. Testing Strategy

### 7.1 Test Credentials
- **Test Mode**: Use test API key with `livemode: false`
- **Test Cards**: As per Monei documentation
- **Environment**: Same base URL for test and production

### 7.2 Test Scenarios
1. **Successful Payment Flow**
   - Create payment (AUTH) → Capture → Success
   - Create payment (SALE) → Direct success

2. **Failed Payment Scenarios**
   - Invalid payment token
   - Insufficient funds
   - 3DS authentication required

3. **Refund Scenarios**
   - Full refund after capture
   - Partial refund
   - Multiple partial refunds

4. **Edge Cases**
   - Timeout handling
   - Invalid currency
   - Missing required fields

### 7.3 Integration Test Structure
```rust
#[test]
async fn test_monei_payment_create_success() {
    let connector = Monei;
    let auth_type = MoneiAuthType {
        api_key: Secret::new("test_api_key".to_string()),
    };
    // Test implementation
}
```

## 8. Connector-Specific Considerations

### 8.1 Quirks and Limitations
- Payment token required for card payments (generated via monei.js)
- SessionId validation for temporary tokens
- AMEX may not be supported by some processors (E508)
- Single capture only (no multiple partial captures documented)

### 8.2 Best Practices
- Always include sessionId with temporary payment tokens
- Use generatePaymentToken for recurring payments
- Include comprehensive billing details for better approval rates
- Monitor callbackUrl for async status updates

### 8.3 PCI Compliance
- Use monei.js Components for card data collection
- Never handle raw card data server-side
- Temporary tokens expire; permanent tokens for recurring use

## 9. Implementation Checklist

### 9.1 Core Implementation
- [ ] Run `add_connector.sh monei https://api.monei.com/v1`
- [ ] Implement MoneiAuthType with api_key field
- [ ] Complete ConnectorCommon trait implementation
- [ ] Implement Payment trait with Authorize flow
- [ ] Implement Capture flow
- [ ] Implement PaymentSync flow
- [ ] Implement Refund and RefundSync flows
- [ ] Add proper error response parsing

### 9.2 Additional Features
- [ ] Handle 3DS responses (E300-E309 errors)
- [ ] Implement payment token handling
- [ ] Add webhook signature verification
- [ ] Support partial captures and refunds

### 9.3 Testing & Documentation
- [ ] Move test file to correct location
- [ ] Implement all 20 sanity tests
- [ ] Add test cases for error scenarios
- [ ] Test partial capture/refund flows
- [ ] Verify webhook handling
- [ ] Update connector documentation

## 10. References

### 10.1 External Documentation
- [Official API Documentation](https://docs.monei.com/)
- [monei.js Integration Guide](https://docs.monei.com/)

### 10.2 Internal References
- Similar connector implementations: Checkout, Stripe (for payment token handling)
- Reusable patterns from: Common utils for amount conversion
- Error handling patterns: Standard connector error mappings
