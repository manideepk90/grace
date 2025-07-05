# MPGS Connector Integration Implementation Plan

## Initial Setup and Configuration

- [x] Step 1: Generate boilerplate code and move test file
  - **Task**: Run the connector generation script and organize files according to Hyperswitch standards
  - **Files**: 
    - Execute: `sh scripts/add_connector.sh mpgs https://test-gateway.mastercard.com`
    - Move: `crates/hyperswitch_connectors/src/connectors/mpgs/test.rs` → `crates/router/tests/connectors/mpgs.rs`
  - **Step Dependencies**: None
  - **User Instructions**: Run the script from the root directory of the project

- [x] Step 2: Add MPGS to connector enums and configuration
  - **Task**: Register MPGS in the connector enums and add configuration entry
  - **Files**:
    - `crates/common_enums/src/connector_enums.rs`: Add Mpgs variant to Connector enum
    - `crates/connector_configs/toml/development.toml`: Add MPGS configuration section
    - `crates/connector_configs/toml/production.toml`: Add MPGS configuration section
    - `crates/connector_configs/toml/sandbox.toml`: Add MPGS configuration section
  - **Step Dependencies**: Step 1
  - **User Instructions**: None

## Authentication Implementation

- [x] Step 3: Implement authentication types and transformers
  - **Task**: Create MPGS authentication structure for HTTP Basic Auth
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Define MpgsAuthType struct and TryFrom implementation
  - **Step Dependencies**: Step 2
  - **User Instructions**: None

- [x] Step 4: Implement ConnectorCommon trait with authentication
  - **Task**: Implement base connector methods including auth header generation
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorCommon trait with HTTP Basic Auth
  - **Step Dependencies**: Step 3
  - **User Instructions**: None

## Core Data Structures

- [x] Step 5: Define core request/response structures
  - **Task**: Create fundamental MPGS API structures for payments
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add MpgsApiOperation, MpgsOrder, MpgsSourceOfFunds, MpgsCard structures
  - **Step Dependencies**: Step 4
  - **User Instructions**: None

- [x] Step 6: Define payment response structures
  - **Task**: Create response structures for payment operations
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add MpgsPaymentResponse, MpgsTransaction, MpgsResponseCode structures
  - **Step Dependencies**: Step 5
  - **User Instructions**: None

## Payment (Pay) Flow

- [x] Step 7: Implement pay request transformer
  - **Task**: Create request transformation for the Pay flow.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom for MpgsPaymentRequest from PaymentsRouterData.
  - **Step Dependencies**: Step 6
  - **User Instructions**: None

- [x] Step 8: Implement pay response transformer
  - **Task**: Create response transformation for the Pay flow.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom ResponseRouterData for the Pay response.
  - **Step Dependencies**: Step 7
  - **User Instructions**: None

- [x] Step 9: Implement ConnectorIntegration for Pay
  - **Task**: Complete the Pay flow integration.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<Pay> trait.
  - **Step Dependencies**: Step 8
  - **User Instructions**: None

## Payment Authorization Flow

- [x] Step 10: Implement authorize request transformer
  - **Task**: Create request transformation for authorization flow
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom for MpgsPaymentRequest from PaymentsAuthorizeRouterData
  - **Step Dependencies**: Step 9
  - **User Instructions**: None

- [x] Step 11: Implement authorize response transformer
  - **Task**: Create response transformation for authorization flow
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom ResponseRouterData for authorization response
  - **Step Dependencies**: Step 10
  - **User Instructions**: None

- [x] Step 12: Implement ConnectorIntegration for Authorize
  - **Task**: Complete the authorization flow integration
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<Authorize> trait
  - **Step Dependencies**: Step 11
  - **User Instructions**: None

## Payment Capture Flow

- [x] Step 13: Implement capture request transformer
  - **Task**: Create request transformation for capture flow
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add MpgsCaptureRequest struct and TryFrom implementation
  - **Step Dependencies**: Step 12
  - **User Instructions**: None

- [x] Step 14: Implement capture response transformer
  - **Task**: Create response transformation for capture flow
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom ResponseRouterData for capture response
  - **Step Dependencies**: Step 13
  - **User Instructions**: None

- [x] Step 15: Implement ConnectorIntegration for Capture
  - **Task**: Complete the capture flow integration
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<Capture> trait
  - **Step Dependencies**: Step 14
  - **User Instructions**: None

## Payment Void/Cancel Flow

- [x] Step 16: Implement void request transformer
  - **Task**: Create request transformation for void/cancel flow
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add MpgsVoidRequest struct and TryFrom implementation
  - **Step Dependencies**: Step 15
  - **User Instructions**: None

- [x] Step 17: Implement ConnectorIntegration for Void
  - **Task**: Complete the void flow integration
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<Void> trait
  - **Step Dependencies**: Step 16
  - **User Instructions**: None

## Payment Status Sync

- [x] Step 18: Implement payment sync transformers
  - **Task**: Create transformers for payment status retrieval
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add response transformer for payment sync
  - **Step Dependencies**: Step 17
  - **User Instructions**: None

- [x] Step 19: Implement ConnectorIntegration for PSync
  - **Task**: Complete the payment sync flow integration
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<PSync> trait
  - **Step Dependencies**: Step 18
  - **User Instructions**: None

## Refund Implementation

- [x] Step 20: Define refund request/response structures
  - **Task**: Create structures for refund operations
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add MpgsRefundRequest and MpgsRefundResponse structures
  - **Step Dependencies**: Step 19
  - **User Instructions**: None

- [x] Step 21: Implement refund transformers
  - **Task**: Create request and response transformations for refunds
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom for refund request and response
  - **Step Dependencies**: Step 20
  - **User Instructions**: None

- [x] Step 22: Implement ConnectorIntegration for RefundExecute
  - **Task**: Complete the refund execution flow
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<RefundExecute> for refunds
  - **Step Dependencies**: Step 21
  - **User Instructions**: None

- [x] Step 23: Implement ConnectorIntegration for RefundSync
  - **Task**: Complete the refund status sync flow
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<RefundSync> trait
  - **Step Dependencies**: Step 22
  - **User Instructions**: None

## Verify Flow
- [ ] Step 24: Implement Verify request transformer
  - **Task**: Create request transformation for the Verify flow.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom for MpgsVerifyRequest from VerifyRouterData.
  - **Step Dependencies**: Step 23
  - **User Instructions**: None

- [ ] Step 25: Implement Verify response transformer
  - **Task**: Create response transformation for the Verify flow.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom ResponseRouterData for the Verify response.
  - **Step Dependencies**: Step 24
  - **User Instructions**: None

- [ ] Step 26: Implement ConnectorIntegration for Verify
  - **Task**: Complete the Verify flow integration.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<Verify> trait.
  - **Step Dependencies**: Step 25
  - **User Instructions**: None

## Disbursement Flow
- [ ] Step 27: Implement Disbursement request transformer
  - **Task**: Create request transformation for the Disbursement flow.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom for MpgsDisbursementRequest.
  - **Step Dependencies**: Step 26
  - **User Instructions**: None

- [ ] Step 28: Implement Disbursement response transformer
  - **Task**: Create response transformation for the Disbursement flow.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Implement TryFrom ResponseRouterData for the Disbursement response.
  - **Step Dependencies**: Step 27
  - **User Instructions**: None

- [ ] Step 29: Implement ConnectorIntegration for Disbursement
  - **Task**: Complete the Disbursement flow integration.
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<Disbursement> trait.
  - **Step Dependencies**: Step 28
  - **User Instructions**: None

## Error Handling

- [x] Step 30: Implement error response handling
  - **Task**: Create error response structures and mapping
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add MpgsErrorResponse struct
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement build_error_response in ConnectorCommon
  - **Step Dependencies**: Step 29
  - **User Instructions**: None

## Helper Functions and Utilities

- [ ] Step 31: Implement helper functions
  - **Task**: Create utility functions for common operations
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add helper functions for amount conversion, status mapping
  - **Step Dependencies**: Step 30
  - **User Instructions**: None

- [x] Step 32: Implement ConnectorCommonExt
  - **Task**: Add common extension trait implementation
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorCommonExt trait
  - **Step Dependencies**: Step 31
  - **User Instructions**: None

## 3DS Authentication Support

- [ ] Step 33: Define 3DS session structures
  - **Task**: Create structures for 3DS authentication flow
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add MpgsSessionRequest, MpgsSessionResponse structures
  - **Step Dependencies**: Step 32
  - **User Instructions**: None

- [ ] Step 34: Implement preprocessing for 3DS
  - **Task**: Create preprocessing flow for 3DS session creation
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<PreProcessing> trait
  - **Step Dependencies**: Step 33
  - **User Instructions**: None

- [ ] Step 35: Implement complete authorize for 3DS
  - **Task**: Handle 3DS authentication completion
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement ConnectorIntegration<CompleteAuthorize> trait
  - **Step Dependencies**: Step 34
  - **User Instructions**: None

## Webhook Implementation

- [x] Step 36: Define webhook structures
  - **Task**: Create webhook event structures
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs/transformers.rs`: Add MpgsWebhookEvent, MpgsWebhookBody structures
  - **Step Dependencies**: Step 35
  - **User Instructions**: None

- [x] Step 37: Implement webhook handling
  - **Task**: Implement webhook verification and event processing
  - **Files**:
    - `crates/hyperswitch_connectors/src/connectors/mpgs.rs`: Implement IncomingWebhook trait
  - **Step Dependencies**: Step 36
  - **User Instructions**: None

## Testing Implementation

- [ ] Step 38: Create test configuration
  - **Task**: Add MPGS test credentials to test configuration
  - **Files**:
    - `crates/router/tests/connectors/sample_auth.toml`: Add MPGS test credentials section
  - **Step Dependencies**: Step 37
  - **User Instructions**: Add test merchant credentials to the auth file

- [ ] Step 39: Implement integration tests
  - **Task**: Create comprehensive integration tests for all flows
  - **Files**:
    - `crates/router/tests/connectors/mpgs.rs`: Implement test cases for authorize, capture, void, refund flows
  - **Step Dependencies**: Step 38
  - **User Instructions**: None

## Build and Error Resolution

- [ ] Step 40: Run initial build and fix compilation errors
  - **Task**: Compile the project and resolve any type or syntax errors
  - **Files**:
    - Various files based on compilation errors
  - **Step Dependencies**: Step 39
  - **User Instructions**: Run `cargo build` from root directory and fix errors based on errors.md guide

- [ ] Step 41: Run clippy and fix warnings
  - **Task**: Run clippy linter and resolve all warnings
  - **Files**:
    - Various files based on clippy warnings
  - **Step Dependencies**: Step 40
  - **User Instructions**: Run `cargo clippy --all-features` and fix warnings

## Cypress Test Implementation

- [ ] Step 42: Create Cypress test configuration
  - **Task**: Add MPGS configuration for Cypress tests
  - **Files**:
    - `cypress-tests/cypress/fixtures/connector-auth.json`: Add MPGS test configuration
    - `cypress-tests/cypress/fixtures/payment-method-data.json`: Add MPGS payment method data
  - **Step Dependencies**: Step 41
  - **User Instructions**: None

- [ ] Step 43: Implement Cypress tests for payment flows
  - **Task**: Create E2E tests for MPGS payment operations
  - **Files**:
    - `cypress-tests/cypress/e2e/connectors/mpgs.cy.js`: Create tests for authorize, capture, void flows
  - **Step Dependencies**: Step 42
  - **User Instructions**: None

- [ ] Step 44: Implement Cypress tests for refund flows
  - **Task**: Create E2E tests for MPGS refund operations
  - **Files**:
    - `cypress-tests/cypress/e2e/connectors/mpgs-refund.cy.js`: Create tests for refund scenarios
  - **Step Dependencies**: Step 43
  - **User Instructions**: None

## Documentation and Final Validation

- [ ] Step 45: Update connector documentation
  - **Task**: Add MPGS to the list of supported connectors
  - **Files**:
    - `docs/connectors/mpgs.md`: Create MPGS connector documentation
    - `README.md`: Update supported connectors list
  - **Step Dependencies**: Step 44
  - **User Instructions**: None

- [ ] Step 46: Run full test suite
  - **Task**: Execute all tests to ensure proper integration
  - **Files**: None (testing only)
  - **Step Dependencies**: Step 45
  - **User Instructions**: Run `cargo test --package router --test connectors -- mpgs` for integration tests

- [ ] Step 47: Final build and validation
  - **Task**: Perform final build and ensure no errors or warnings
  - **Files**: None (validation only)
  - **Step Dependencies**: Step 46
  - **User Instructions**: Run `cargo build --release` from root directory

## Summary

This implementation plan breaks down the MPGS connector integration into 47 manageable steps. The approach follows these key phases:

1. **Setup and Configuration** (Steps 1-2): Initial boilerplate and configuration
2. **Authentication** (Steps 3-4): HTTP Basic Auth implementation
3. **Core Structures** (Steps 5-6): Request/response data models
4. **Payment Flows** (Steps 7-19): Pay, Authorization, capture, void, and sync
5. **Refund Flows** (Steps 20-23): Refund execution and status sync
6. **Verify Flow** (Steps 24-26): Account verification
7. **Disbursement Flow** (Steps 27-29): Payouts
8. **Error Handling** (Steps 30-32): Error response mapping and utilities
9. **3DS Support** (Steps 33-35): Authentication flow handling
10. **Webhooks** (Steps 36-37): Event processing and verification
11. **Testing** (Steps 38-44): Integration and E2E tests
12. **Documentation** (Steps 45-47): Final documentation and validation

Each step is designed to be atomic and builds upon previous steps. The implementation follows Hyperswitch patterns and reuses existing utilities where possible.

## Key Considerations

1. **Amount Conversion**: MPGS uses major units (dollars) while Hyperswitch uses minor units (cents)
2. **Transaction IDs**: Each MPGS operation requires unique transaction IDs
3. **Status Mapping**: Careful mapping between MPGS and Hyperswitch status codes
4. **3DS Flow**: Session-based authentication with redirect handling
5. **Regional URLs**: Base URL varies by merchant region
