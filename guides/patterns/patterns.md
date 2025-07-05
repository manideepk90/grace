# Connector Implementation Patterns

## Authentication Patterns

### HTTP Basic Auth
```rust
let auth_string = format!("{}:{}", username.expose(), password.expose());
let encoded = base64::engine::general_purpose::STANDARD.encode(auth_string.as_bytes());
Ok(vec![(headers::AUTHORIZATION.to_string(), format!("Basic {encoded}").into_masked())])
```

### HMAC Signature (Dlocal Pattern)
```rust
// Centralized in ConnectorCommonExt::build_headers
fn build_headers(&self, req: &RouterData<Flow, Req, Resp>, connectors: &Connectors) -> CustomResult<Vec<(String, Maskable<String>)>, errors::ConnectorError> {
    let request_content = self.get_request_body(req, connectors)?;
    let body_string = match request_content {
        Some(RequestContent::Json(body)) => body.peek().to_string(),
        Some(RequestContent::FormUrlEncoded(form)) => form.peek().to_string(),
        _ => String::new(),
    };
    
    let timestamp = date_time::now();
    let auth = DlocalAuthType::try_from(&req.connector_auth_type)?;
    
    // Signature: X-Login + X-Date + RequestBody
    let to_sign = format!("{}{}{}", auth.x_login, timestamp, body_string);
    let signature = crypto::HmacSha256::sign_message(
        auth.secret_key.peek().as_bytes(),
        to_sign.as_bytes(),
    )?;
    
    Ok(vec![
        ("X-Login".to_string(), auth.x_login.into_masked()),
        ("X-Trans-Key".to_string(), auth.x_trans_key.into_masked()),
        ("X-Date".to_string(), timestamp.to_string().into()),
        ("X-Version".to_string(), "2.1".into()),
        ("Authorization".to_string(), format!("V2-HMAC-SHA256, Signature: {}", hex::encode(signature)).into()),
    ])
}
```

## Data Transformation Patterns

### TryFrom for Auth Response (Avoiding Orphan Rule)
```rust
// Use local ResponseRouterData wrapper
impl TryFrom<ResponseRouterData<AccessTokenAuth, ConnectorAuthResponse, AccessTokenRequestData, AccessToken>>
    for RouterData<AccessTokenAuth, AccessTokenRequestData, AccessToken>
{
    fn try_from(item: ResponseRouterData<...>) -> Result<Self, Self::Error> {
        Ok(Self {
            response: Ok(AccessToken {
                token: item.response.token_field,
                expires: item.response.expires_field,
            }),
            ..item.data // Spread original RouterData
        })
    }
}
```

### TryFrom for Payment Response
```rust
impl TryFrom<ResponseRouterData<Authorize, ConnectorPaymentResponse, PaymentsAuthorizeData, PaymentsResponseData>>
    for RouterData<Authorize, PaymentsAuthorizeData, PaymentsResponseData>
{
    fn try_from(item: ResponseRouterData<...>) -> Result<Self, Self::Error> {
        let status = match item.response.status.as_str() {
            "approved" => AttemptStatus::Charged,
            "failed" => AttemptStatus::Failure,
            _ => AttemptStatus::Pending,
        };
        
        Ok(Self {
            status,
            response: Ok(PaymentsResponseData::TransactionResponse {
                resource_id: ResponseId::ConnectorTransactionId(item.response.transaction_id),
                redirection_data: item.response.redirect_url.map(|url| {
                    Box::new(RedirectForm::Form {
                        endpoint: url,
                        method: Method::Get,
                        form_fields: HashMap::new(),
                    })
                }),
                ..Default::default()
            }),
            ..item.data
        })
    }
}
```

## Amount Handling

### Minor to Major Unit Conversion
```rust
// For connectors expecting f64 major units
let minor_amount_i64 = item.amount.get_amount_as_i64()?;
let major_amount_f64 = utils::to_currency_base_unit(minor_amount_i64, currency)?;
```

### Helper Struct Pattern
```rust
pub struct ConnectorRouterData<T> {
    pub amount: MinorUnit, // Or String/f64 based on API
    pub router_data: T,
}

impl<T> From<(MinorUnit, T)> for ConnectorRouterData<T> {
    fn from((amount, router_data): (MinorUnit, T)) -> Self {
        Self { amount, router_data }
    }
}
```

## Request Construction

### Signature-Safe Request Body
```rust
// 1. Create concrete request
let connector_req = ConnectorPaymentsRequest::try_from(&connector_router_data)?;

// 2. Serialize for signature
let request_str = serde_json::to_string(&connector_req)?;

// 3. Box for request body
let request_content = RequestContent::Json(Box::new(connector_req));
```

### Required ConnectorIntegration Implementations

Don't forget to implement these connector flows if applicable:
- `PaymentReject`
- `PaymentApprove`
- `PaymentAuthorizeSessionToken`

### Empty Request Bodies
```rust
// GET requests: Don't call set_body()

// POST with empty body
Ok(RequestContent::Empty)

// POST with empty JSON
#[derive(Debug, Serialize)]
pub struct EmptyRequest {}
Ok(RequestContent::Json(Box::new(EmptyRequest {})))
```

## Configuration & Metadata

### Merchant-Specific Config
```rust
let gateway_token = req.connector_meta_data
    .as_ref()
    .and_then(|meta| meta.peek().as_object())
    .and_then(|obj| obj.get("gateway_token"))
    .and_then(|token| token.as_str())
    .ok_or(errors::ConnectorError::MissingRequiredField { 
        field_name: "gateway_token" 
    })?;
```

### Transaction Token Management
```rust
// Store in response
resource_id: ResponseId::ConnectorTransactionId(response.transaction.token),

// Retrieve for capture/refund/sync
let token = req.request.connector_transaction_id.clone();
// Or for sync
let token = req.request.get_connector_transaction_id()?;

// URL pattern
format!("{}/v1/transactions/{}/capture.json", base_url, token)
```

## Field Processing

### Name Splitting
```rust
let parts: Vec<&str> = name.peek().split_whitespace().collect();
first_name: parts.first()
    .map(|s| Secret::new(s.to_string()))
    .unwrap_or_else(|| Secret::new("".to_string())),
last_name: if parts.len() > 1 {
    Some(Secret::new(parts[1..].join(" ")))
} else {
    None
}.unwrap_or_else(|| Secret::new("".to_string())),
```

### Field and Method Names
- Use `customer_id` not `connector_customer`
- Use `get_ip_address_as_optional()` not `get_optional_ip()`

### JSON to HashMap<String, String>
```rust
let form_fields = if let Value::Object(map) = json_data {
    map.into_iter()
        .filter_map(|(k, v)| match v {
            Value::String(s) => Some((k, s)),
            Value::Number(n) => Some((k, n.to_string())),
            Value::Bool(b) => Some((k, b.to_string())),
            _ => None,
        })
        .collect()
} else {
    HashMap::new()
};
```

## Status Mapping

### Multi-Field Status
```rust
let status = match (response.transaction_type.as_str(), response.succeeded) {
    ("Authorize", true) => AttemptStatus::Authorized,
    ("Capture", true) => AttemptStatus::Charged,
    ("Void", true) => AttemptStatus::Voided,
    (_, false) => AttemptStatus::Failure,
    _ => AttemptStatus::Pending,
};
```

### Enum-Based Status
```rust
impl From<ConnectorStatus> for AttemptStatus {
    fn from(status: ConnectorStatus) -> Self {
        match status {
            ConnectorStatus::Succeeded => Self::Charged,
            ConnectorStatus::Authorized => Self::Authorized,
            ConnectorStatus::Failed => Self::Failure,
            ConnectorStatus::Pending => Self::Pending,
        }
    }
}
```

## Webhook Patterns

### HMAC-SHA256 Verification
```rust
fn verify_webhook_source(&self, request: &IncomingWebhookRequestDetails<'_>, 
    secrets: &ConnectorWebhookSecrets) -> CustomResult<(), ConnectorError> {
    let signature = get_header_key_value("x-signature", request.headers)?;
    let webhook_secret = secrets.secret.as_ref()
        .ok_or(ConnectorError::WebhookVerificationSecretNotFound)?;
    
    let expected = crypto::HmacSha256::sign_message(
        webhook_secret.expose().as_bytes(),
        request.body.as_bytes(),
    )?;
    
    if hex::encode(expected) != signature {
        return Err(ConnectorError::WebhookSourceVerificationFailed.into());
    }
    Ok(())
}
```

### Object Reference ID
```rust
fn get_webhook_object_reference_id(&self, request: &IncomingWebhookRequestDetails<'_>) 
    -> CustomResult<ObjectReferenceId, ConnectorError> {
    let webhook: ConnectorWebhook = request.body.parse_struct("ConnectorWebhook")?;
    let payment_id = id_type::PaymentId::try_from(Cow::Owned(webhook.payment_id))?;
    
    Ok(ObjectReferenceId::PaymentId(
        PaymentIdType::PaymentIntentId(payment_id)
    ))
}
```

## Import Patterns

### Request Data Accessor Traits
```rust
// In transformers.rs
use crate::utils::{
    PaymentsAuthorizeRequestData,
    PaymentsPreProcessingRequestData,
    RefundsRequestData,
};
```

### Parse Method
```rust
// For parse_struct on byte slices
use common_utils::ext_traits::ByteSliceExt;
```

## Struct Patterns

### Flat Structures (When API is Simple)
```rust
#[derive(Debug, Serialize)]
pub struct PaymentsRequest {
    transaction: Transaction,
}

#[derive(Debug, Serialize)]
pub struct Transaction {
    amount: StringMinorUnit,
    currency_code: String,
    card_number: cards::CardNumber,
    // Direct fields, not nested
}
```

### Derive Requirements
```rust
// For event_builder.set_response_body()
#[derive(Debug, Clone, Deserialize, Serialize)]
pub struct ConnectorResponse { /* ... */ }

// For enums in Default structs
#[derive(Debug, Clone, Default, Deserialize, Serialize)]
#[serde(rename_all = "SCREAMING_SNAKE_CASE")]
pub enum Status {
    Succeeded,
    Failed,
    #[default]
    Pending,
}
```

## Connector-Specific Auth Patterns

### MPGS Basic Auth Pattern
```rust
// MPGS uses Basic Auth with merchant ID embedded in the API key
// Format: "merchant.{merchantId}:{password}"
fn get_auth_header(&self, auth_type: &ConnectorAuthType) -> CustomResult<Vec<(String, masking::Maskable<String>)>, errors::ConnectorError> {
    let auth = mpgs::MpgsAuthType::try_from(auth_type)
        .change_context(errors::ConnectorError::FailedToObtainAuthType)?;
    
    // MPGS API key contains both merchant ID and password
    let encoded = common_utils::consts::BASE64_ENGINE.encode(auth.api_key.expose());
    
    Ok(vec![(
        headers::AUTHORIZATION.to_string(),
        format!("Basic {}", encoded).into_masked(),
    )])
}

// Extract merchant ID from auth key
let merchant_id = auth.api_key.expose()
    .strip_prefix("merchant.")
    .ok_or(errors::ConnectorError::InvalidConnectorConfig { 
        config: "Invalid merchant ID format. Expected: merchant.{merchantId}".to_string() 
    })?;
```

## URL Building Patterns

### MPGS URL Pattern (Unique Transaction IDs)
```rust
// MPGS requires unique transaction IDs for each operation
fn get_payment_url(
    auth_type: &ConnectorAuthType,
    payment_id: &str,
    capture_method: Option<CaptureMethod>,
) -> Result<String, error_stack::Report<errors::ConnectorError>> {
    let auth = MpgsAuthType::try_from(auth_type)?;
    let merchant_id = extract_merchant_id(&auth)?;
    
    // Generate unique transaction ID
    let transaction_id = format!("txn_{}", uuid::Uuid::new_v4());
    
    // Determine operation type
    let operation = match capture_method {
        Some(CaptureMethod::Automatic) | None => "pay",
        Some(CaptureMethod::Manual) => "authorize",
        _ => "pay",
    };
    
    Ok(format!(
        "/api/rest/version/73/merchant/{}/order/{}/transaction/{}?operation={}",
        merchant_id, payment_id, transaction_id, operation
    ))
}
```

## Request Structure Patterns

### MPGS Nested Request Structure
```rust
#[derive(Debug, Serialize)]
#[serde(rename_all = "camelCase")]
pub struct MpgsPaymentsRequest {
    api_operation: MpgsApiOperation,  // Determines flow type
    order: MpgsOrder,
    source_of_funds: MpgsSourceOfFunds,
    transaction: MpgsTransaction,
    #[serde(skip_serializing_if = "Option::is_none")]
    customer: Option<MpgsCustomer>,
}

// Enum for API operations
#[derive(Debug, Serialize)]
#[serde(rename_all = "SCREAMING_SNAKE_CASE")]
pub enum MpgsApiOperation {
    Pay,        // Automatic capture
    Authorize,  // Manual capture
    Capture,
    Void,
    Refund,
    Verify,
}
```

## Response Handling Patterns

### MPGS Multi-Field Status Mapping
```rust
// MPGS requires checking both 'result' and 'response.gateway_code'
let status = match (item.response.result.as_str(), item.response.response.gateway_code.as_str()) {
    ("SUCCESS", "APPROVED") => {
        match item.response.transaction.transaction_type.as_str() {
            "AUTHORIZATION" => common_enums::AttemptStatus::Authorized,
            "PAYMENT" | "CAPTURE" => common_enums::AttemptStatus::Charged,
            "VOID" => common_enums::AttemptStatus::Voided,
            _ => common_enums::AttemptStatus::Pending,
        }
    },
    ("PENDING", _) => common_enums::AttemptStatus::Pending,
    (_, "AUTHENTICATION_REQUIRED") => common_enums::AttemptStatus::AuthenticationPending,
    _ => common_enums::AttemptStatus::Failure,
};
```

## Error Response Patterns

### MPGS Nested Error Structure
```rust
#[derive(Debug, Serialize, Deserialize)]
pub struct MpgsErrorResponse {
    pub error: MpgsError,  // Nested error object
}

#[derive(Debug, Serialize, Deserialize)]
pub struct MpgsError {
    pub cause: String,
    pub explanation: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub field: Option<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub validation_error: Option<String>,
}

// Error response transformation
fn build_error_response(&self, res: Response, event_builder: Option<&mut ConnectorEvent>) -> CustomResult<ErrorResponse, errors::ConnectorError> {
    let response: mpgs::MpgsErrorResponse = res.response.parse_struct("MpgsErrorResponse")?;
    
    Ok(ErrorResponse {
        status_code: res.status_code,
        code: response.error.cause.clone(),
        message: response.error.explanation.clone(),
        reason: response.error.field.clone(),
        attempt_status: None,
        connector_transaction_id: None,
        network_advice_code: None,
        network_decline_code: None,
        network_error_message: response.error.validation_error,
    })
}
```

## Key Principles

1. **Use Local Wrappers**: Always use `crate::types::ResponseRouterData` for TryFrom to avoid orphan rule
2. **Centralize Common Logic**: Like HMAC generation in `build_headers`
3. **Handle Sensitive Data**: Use `peek()` for references, `expose()` only when necessary
4. **Simplify When Possible**: Match API simplicity with code simplicity
5. **Clear Error Messages**: Always provide context in errors
6. **Type Safety**: Use appropriate types (Secret, Email, etc.)
7. **Consistent Patterns**: Follow established patterns from other connectors
8. **Unique Transaction IDs**: Some APIs (like MPGS) require unique IDs per operation
9. **Nested Response Handling**: Check for nested error/status structures
10. **Multi-Field Status Logic**: Some APIs require checking multiple fields for status
