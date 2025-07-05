# MPGS Connector Integration Technical Specification

## 1. Connector Overview

### 1.1 Basic Information
- **Connector Name**: mpgs
- **Base URL**: Configured per region (e.g., https://ap-gateway.mastercard.com, https://eu-gateway.mastercard.com)
- **API Version**: 100
- **API Documentation**: https://test-gateway.mastercard.com/api/documentation/apiDocumentation/rest-json/version/100/api.html
- **Supported Countries**: Global (region-specific endpoints)
- **Supported Currencies**: All major currencies

### 1.2 Authentication Method
- **Type**: HTTP Basic Authentication
- **Header Format**: `Authorization: Basic base64(merchant_id:password)`
- **Credentials Required**:
  - `merchant_id`: Merchant identifier
  - `password`: API password
- **Additional Headers**: 
  - `Content-Type: application/json`

### 1.3 Supported Features
| Feature | Supported | Notes |
|---------|-----------|-------|
| Card Payments | ✓ | Credit/Debit cards |
| Bank Transfers | ✗ | Not supported |
| Wallets | ✓ | Via tokenization |
| 3DS 2.0 | ✓ | Via session creation and redirect |
| Recurring Payments | ✓ | Using stored tokens |
| Partial Capture | ✓ | Capture less than authorized amount |
| Multiple Captures | ✗ | Single capture only |
| Instant Refunds | ✓ | Immediate processing |
| Partial Refunds | ✓ | Refund less than captured amount |
| Disbursements | ✓ | Payouts to cards |
| Webhooks | ✓ | Transaction notifications |

## 2. API Endpoints

### 2.1 Payment Operations
| Operation | Method | Endpoint | Purpose |
|-----------|---------|----------|---------|
| Create Session | POST | /api/rest/version/100/merchant/{merchantId}/session | Create checkout session for 3DS |
| Pay | PUT | /api/rest/version/100/merchant/{merchantId}/order/{orderId}/transaction/{transactionId} | Combined auth + capture |
| Authorize | PUT | /api/rest/version/100/merchant/{merchantId}/order/{orderId}/transaction/{transactionId} | Reserve funds |
| Capture | PUT | /api/rest/version/100/merchant/{merchantId}/order/{orderId}/transaction/{transactionId} | Capture authorized funds |
| Void | PUT | /api/rest/version/100/merchant/{merchantId}/order/{orderId}/transaction/{transactionId} | Cancel authorization |
| Verify | PUT | /api/rest/version/100/merchant/{merchantId}/order/{orderId}/transaction/{transactionId} | Validate card |
| Retrieve Order | GET | /api/rest/version/100/merchant/{merchantId}/order/{orderId} | Get order status |
| Retrieve Transaction | GET | /api/rest/version/100/merchant/{merchantId}/order/{orderId}/transaction/{transactionId} | Get transaction status |

### 2.2 Refund and Disbursement Operations
| Operation | Method | Endpoint | Purpose |
|-----------|---------|----------|---------|
| Refund | PUT | /api/rest/version/100/merchant/{merchantId}/order/{orderId}/transaction/{newTransactionId} | Initiate refund |
| Disbursement | PUT | /api/rest/version/100/merchant/{merchantId}/order/{orderId}/transaction/{transactionId} | Payout funds to a card |

### 2.3 Tokenization Operations
| Operation | Method | Endpoint | Purpose |
|-----------|---------|----------|---------|
| Create Token | PUT | /api/rest/version/100/merchant/{merchantId}/token/{tokenId} | Tokenize card details |
| Get Token | GET | /api/rest/version/100/merchant/{merchantId}/token/{tokenId} | Retrieve token details |

## 3. Data Models

### 3.1 Payment Request Structure

#### PAY Request (Purchase)
```json
{
  "apiOperation": "PAY",
  "order": {
    "amount": "10.55",
    "currency": "USD"
  },
  "sourceOfFunds": {
    "type": "CARD",
    "provided": {
      "card": {
        "number": "4111111111111111",
        "expiry": {
          "month": "12",
          "year": "25"
        },
        "securityCode": "123"
      }
    }
  },
  "transaction": {
    "source": "INTERNET"
  }
}
```

#### DISBURSEMENT Request
```json
{
  "apiOperation": "DISBURSEMENT",
  "disbursementType": "GAMING_WINNINGS",
  "order": {
    "amount": "50.00",
    "currency": "USD"
  },
  "sourceOfFunds": {
    "type": "CARD",
    "provided": {
      "card": {
        "number": "4111111111111111",
        "expiry": {
          "month": "12",
          "year": "25"
        }
      }
    }
  }
}
```

### 3.2 Payment Response Structure
```json
{
  "result": "SUCCESS",
  "merchant": "your_merchant_id",
  "order": {
    "amount": 100.00,
    "currency": "USD",
    "id": "ORDER123",
    "status": "CAPTURED",
    "totalAuthorizedAmount": 100.00,
    "totalCapturedAmount": 100.00,
    "totalDisbursedAmount": 0.00,
    "totalRefundedAmount": 0.00
  },
  "transaction": {
    "id": "TXN123",
    "type": "PAYMENT",
    "authorizationCode": "123456"
  },
  "response": {
    "gatewayCode": "APPROVED"
  }
}
```

### 3.3 Status Mappings
| MPGS Status | Hyperswitch Status | Description |
|------------------|-------------------|-------------|
| APPROVED (PAY) | Charged | Payment completed successfully |
| APPROVED (AUTHORIZE) | Authorized | Authorization successful |
| APPROVED (CAPTURE) | Charged | Capture successful |
| APPROVED (VOID) | Voided | Void successful |
| APPROVED (REFUND) | Succeeded | Refund successful |
| APPROVED (DISBURSEMENT) | Succeeded | Disbursement successful |
| DECLINED | Failure | Transaction declined |
| REFERRED | Pending | Requires manual review |
| AUTHENTICATION_IN_PROGRESS | AuthenticationPending | 3DS required |
| AUTHENTICATION_FAILED | Failure | 3DS authentication failed |
| PENDING | Pending | Transaction is pending |
| SUBMITTED | Pending | Transaction submitted to acquirer |
| UNKNOWN | Pending | Transaction result is unknown |

### 3.4 Error Code Mappings
| MPGS Error Code | Hyperswitch Error | Description |
|---------------------|------------------|-------------|
| INSUFFICIENT_FUNDS | InsufficientFunds | Card has insufficient funds |
| EXPIRED_CARD | CardExpired | Card has expired |
| INVALID_CSC | InvalidCardSecurityCode | Security code is invalid |
| DECLINED | TransactionDeclined | Card was declined |
| UNSPECIFIED_FAILURE | TransactionFailed | Transaction could not be processed |
| SYSTEM_ERROR | InternalServerError | Internal system error |
| INVALID_REQUEST | InvalidRequest | The request was invalid |

## 4. Implementation Details

### 4.1 Request Transformations

#### 4.1.1 Authorize Request
```rust
// Amount conversion: Minor units (cents) to Major units (dollars)
let amount = utils::to_currency_base_unit(req.request.amount, req.request.currency)?;

MpgsPaymentRequest {
    api_operation: MpgsApiOperation::Authorize,
    order: MpgsOrder {
        amount: format!("{:.2}", amount), // Format as string with 2 decimals
        currency: req.request.currency.to_string(),
        id: Some(req.connector_request_reference_id.clone()),
    },
    source_of_funds: MpgsSourceOfFunds {
        source_type: "CARD".to_string(),
        provided: MpgsProvidedData {
            card: transform_card_data(&req.request.payment_method_data)?,
        },
    },
    transaction: MpgsTransactionInfo {
        source: "INTERNET".to_string(),
        id: Some(req.payment_id.clone()),
    },
    customer: req.request.email.map(|email| MpgsCustomer {
        email: email.clone(),
    }),
}
```

### 4.2 Response Transformations
```rust
// Status mapping
let status = match (response.result.as_str(), response.response.gateway_code.as_str()) {
    ("SUCCESS", "APPROVED") => {
        match response.transaction.transaction_type.as_str() {
            "AUTHORIZATION" => AttemptStatus::Authorized,
            "PAYMENT" | "CAPTURE" => AttemptStatus::Charged,
            "VOID_AUTHORIZATION" => AttemptStatus::Voided,
            "REFUND" => AttemptStatus::Succeeded,
            "DISBURSEMENT" => AttemptStatus::Succeeded,
            _ => AttemptStatus::Pending,
        }
    },
    ("PENDING", _) => AttemptStatus::Pending,
    (_, "AUTHENTICATION_IN_PROGRESS") => AttemptStatus::AuthenticationPending,
    _ => AttemptStatus::Failure,
};

// Response construction
PaymentsResponseData::TransactionResponse {
    resource_id: ResponseId::ConnectorTransactionId(response.transaction.id),
    redirection_data: response.authentication.redirect_url.map(|url| {
        Box::new(RedirectForm::Form {
            endpoint: url,
            method: Method::Get,
            form_fields: HashMap::new(),
        })
    }),
    mandate_reference: None,
    connector_metadata: None,
    network_txn_id: response.transaction.acquirer.as_ref().map(|a| a.transaction_id.clone()),
    connector_response_reference_id: Some(response.order.id),
}
```

### 4.3 Amount Handling
- **Format**: Major units (dollars) as string with 2 decimal places
- **Conversion**: Use `utils::to_currency_base_unit()` to convert from minor to major units
- **Zero Decimal Currencies**: Handle JPY, KRW, etc. appropriately

## 5. 3DS Authentication Flow

### 5.1 Session Creation
```json
{
  "apiOperation": "CREATE_CHECKOUT_SESSION",
  "interaction": {
    "operation": "PURCHASE"
  },
  "order": {
    "currency": "USD",
    "id": "ORDER123",
    "amount": "100.00"
  }
}
```

### 5.2 Session Response
```json
{
  "result": "SUCCESS",
  "session": {
    "id": "SESSION0002713374821S8232930L51",
    "version": "1.0"
  }
}
```

### 5.3 3DS Redirect Handling
- If authentication required, response includes `authentication.redirectUrl`
- Store session data in connector metadata for completion
- Handle redirect return for final authentication

## 6. Webhook Implementation

### 6.1 Webhook Configuration
- **Endpoint Format**: /webhooks/{merchant_id}/mpgs
- **Signature Header**: Not specified, requires clarification.
- **Signature Algorithm**: Not specified, requires clarification.

### 6.2 Supported Events
| Event Type | Hyperswitch Event | Description |
|------------|------------------|-------------|
| order.status.updated | PaymentIntentSuccess / PaymentIntentFailure | Order status changes |

## 7. Testing Strategy

### 7.1 Test Credentials
- **Test Merchant ID**: Provided by MPGS
- **Test Base URL**: https://ap-gateway.mastercard.com
- **Test Cards**: Provided in MPGS documentation

### 7.2 Test Scenarios
1. **Successful Payment Flow**
   - PAY operation with valid card
   - AUTHORIZE → CAPTURE flow
   - Verify transaction status

2. **Failed Payment Scenarios**
   - Insufficient funds
   - Invalid card number
   - Expired card
   - Invalid CVV

3. **3DS Authentication**
   - Create session
   - Handle redirect
   - Complete authentication

4. **Refund and Disbursement Scenarios**
   - Full refund after capture
   - Partial refund
   - Disbursement for gaming winnings

5. **Void Scenarios**
   - Void authorized transaction
   - Attempt void after capture (should fail)

## 8. Connector-Specific Considerations

### 8.1 Quirks and Limitations
- Transaction IDs must be unique per order
- Order IDs have specific length requirements (1-40 characters)
- Some operations require specific time windows (void within 24 hours)
- Amount format must be string with exactly 2 decimal places

## 9. References

### 9.1 External Documentation
- [MPGS API Reference v100](https://ap-gateway.mastercard.com/api/documentation/apiDocumentation/rest-json/version/100/api.html)
- [MPGS Integration Guide](https://ap-gateway.mastercard.com/api/documentation/integrationGuidelines/index.html)
- [MPGS Test Cards](https://ap-gateway.mastercard.com/api/documentation/integrationGuidelines/supportedFeatures/testAndGoLive.html)

### 9.2 Internal References
- Similar connector implementations: Adyen, Stripe (for 3DS handling)
- Reusable patterns from: patterns.md (HMAC signature, Basic Auth)
- Amount conversion utilities: common_utils
