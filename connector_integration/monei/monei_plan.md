# Monei Connector Integration Implementation Plan

## Phase 1: Project Setup and Boilerplate Generation
- [ ] Step 1: Generate Monei connector boilerplate
  - **Task**: Run the add_connector.sh script to generate initial connector structure with proper base URL
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Generated main connector file
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Generated transformers file
    - `crates/hyperswitch_connectors/src/connectors/monei/test.rs`: Generated test file
  - **Step Dependencies**: None
  - **User Instructions**: Run command: `./scripts/add_connector.sh monei https://api.monei.com/v1`

- [ ] Step 2: Move test file to correct location
  - **Task**: Move the generated test file from hyperswitch_connectors to router tests directory
  - **Files**: 
    - `crates/router/tests/connectors/monei.rs`: Move test file here
  - **Step Dependencies**: Step 1
  - **User Instructions**: Run command: `mv crates/hyperswitch_connectors/src/connectors/monei/test.rs crates/router/tests/connectors/monei.rs`

- [ ] Step 3: Update connector module imports
  - **Task**: Add Monei connector to the connectors module and ensure proper imports
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors.rs`: Add monei module export
    - `crates/router/src/connector.rs`: Add Monei to connector list
  - **Step Dependencies**: Step 1
  - **User Instructions**: None

## Phase 2: Authentication Implementation
- [ ] Step 4: Implement MoneiAuthType structure
  - **Task**: Define the authentication type structure for API key authentication
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Add MoneiAuthType struct with api_key field
  - **Step Dependencies**: Step 1
  - **User Instructions**: None

- [ ] Step 5: Implement authentication conversion and headers
  - **Task**: Implement TryFrom trait for MoneiAuthType and get_auth_header method in ConnectorCommon
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Implement auth conversion and header generation
  - **Step Dependencies**: Step 4
  - **User Instructions**: None

## Phase 3: Core Connector Traits Implementation
- [ ] Step 6: Implement ConnectorCommon trait
  - **Task**: Complete ConnectorCommon trait implementation with id, currency unit, content type, base URL, and error response methods
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Complete ConnectorCommon implementation
  - **Step Dependencies**: Step 5
  - **User Instructions**: None

- [ ] Step 7: Implement ConnectorCommonExt trait
  - **Task**: Implement build_headers method to construct common headers with authentication
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Add ConnectorCommonExt implementation
  - **Step Dependencies**: Step 6
  - **User Instructions**: None

## Phase 4: Request/Response Type Definitions
- [ ] Step 8: Define payment request structures
  - **Task**: Create MoneiPaymentRequest structure with all required fields for authorization
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Define payment request types
  - **Step Dependencies**: Step 1
  - **User Instructions**: None

- [ ] Step 9: Define payment response structures
  - **Task**: Create MoneiPaymentResponse structure matching API response format
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Define payment response types
  - **Step Dependencies**: Step 8
  - **User Instructions**: None

- [ ] Step 10: Define capture and refund structures
  - **Task**: Create MoneicaptureRequest, MoneiRefundRequest, and error response structures
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Define capture, refund, and error types
  - **Step Dependencies**: Step 9
  - **User Instructions**: None

## Phase 5: Payment Authorization Flow
- [ ] Step 11: Implement authorize request transformation
  - **Task**: Implement TryFrom trait to convert RouterData to MoneiPaymentRequest
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Implement payment request transformation
  - **Step Dependencies**: Step 8, Step 9
  - **User Instructions**: None

- [ ] Step 12: Implement authorize response transformation
  - **Task**: Implement TryFrom trait to convert MoneiPaymentResponse to RouterData response
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Implement payment response transformation
  - **Step Dependencies**: Step 11
  - **User Instructions**: None

- [ ] Step 13: Implement PaymentAuthorize connector integration
  - **Task**: Complete ConnectorIntegration trait for PaymentAuthorize with URL, headers, request body, and response handling
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Implement PaymentAuthorize trait
  - **Step Dependencies**: Step 12
  - **User Instructions**: None

## Phase 6: Payment Capture Flow
- [ ] Step 14: Implement capture request transformation
  - **Task**: Implement TryFrom trait for MoneiCaptureRequest
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Implement capture request transformation
  - **Step Dependencies**: Step 10
  - **User Instructions**: None

- [ ] Step 15: Implement capture response transformation
  - **Task**: Implement response transformation for capture operation
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Implement capture response transformation
  - **Step Dependencies**: Step 14
  - **User Instructions**: None

- [ ] Step 16: Implement PaymentCapture connector integration
  - **Task**: Complete ConnectorIntegration trait for PaymentCapture
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Implement PaymentCapture trait
  - **Step Dependencies**: Step 15
  - **User Instructions**: None

## Phase 7: Payment Sync Flow
- [ ] Step 17: Implement payment sync transformation
  - **Task**: Reuse payment response transformation for sync operation
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Add sync-specific status handling
  - **Step Dependencies**: Step 12
  - **User Instructions**: None

- [ ] Step 18: Implement PaymentSync connector integration
  - **Task**: Complete ConnectorIntegration trait for PaymentSync with GET method
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Implement PaymentSync trait
  - **Step Dependencies**: Step 17
  - **User Instructions**: None

## Phase 8: Refund Implementation
- [ ] Step 19: Implement refund request transformation
  - **Task**: Implement TryFrom trait for MoneiRefundRequest with reason mapping
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Implement refund request transformation
  - **Step Dependencies**: Step 10
  - **User Instructions**: None

- [ ] Step 20: Implement refund response transformation
  - **Task**: Implement response transformation for refund operation with status mapping
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Implement refund response transformation
  - **Step Dependencies**: Step 19
  - **User Instructions**: None

- [ ] Step 21: Implement RefundExecute connector integration
  - **Task**: Complete ConnectorIntegration trait for RefundExecute
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Implement RefundExecute trait
  - **Step Dependencies**: Step 20
  - **User Instructions**: None

- [ ] Step 22: Implement RefundSync connector integration
  - **Task**: Complete ConnectorIntegration trait for RefundSync
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Implement RefundSync trait
  - **Step Dependencies**: Step 20
  - **User Instructions**: None

## Phase 9: Error Handling and Edge Cases
- [ ] Step 23: Implement comprehensive error mapping
  - **Task**: Create error code mapping function for all E000-E599 error codes
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Add error mapping utilities
  - **Step Dependencies**: Step 10
  - **User Instructions**: None

- [ ] Step 24: Add status mapping with refund handling
  - **Task**: Implement status mapping that considers refunded amounts for accurate status determination
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Add enhanced status mapping
  - **Step Dependencies**: Step 23
  - **User Instructions**: None

## Phase 10: Helper Functions and Utilities
- [ ] Step 25: Implement payment token extraction
  - **Task**: Create helper function to extract payment token from payment method data
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Add payment token utilities
  - **Step Dependencies**: Step 11
  - **User Instructions**: None

- [ ] Step 26: Implement metadata and address transformations
  - **Task**: Create helper functions for transforming customer details, billing address, and metadata
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Add transformation helpers
  - **Step Dependencies**: Step 11
  - **User Instructions**: None

## Phase 11: Testing Implementation
- [ ] Step 27: Configure test authentication
  - **Task**: Add Monei test credentials to sample_auth.toml
  - **Files**: 
    - `crates/router/tests/connectors/sample_auth.toml`: Add Monei API key configuration
  - **Step Dependencies**: Step 2
  - **User Instructions**: Add your Monei test API key to the sample_auth.toml file

- [ ] Step 28: Implement authorization tests
  - **Task**: Implement tests for authorization flow including AUTH and SALE transaction types
  - **Files**: 
    - `crates/router/tests/connectors/monei.rs`: Implement authorization test cases
  - **Step Dependencies**: Step 27, Step 13
  - **User Instructions**: None

- [ ] Step 29: Implement capture and sync tests
  - **Task**: Implement tests for capture flow and payment sync operations
  - **Files**: 
    - `crates/router/tests/connectors/monei.rs`: Implement capture and sync test cases
  - **Step Dependencies**: Step 28, Step 16, Step 18
  - **User Instructions**: None

- [ ] Step 30: Implement refund tests
  - **Task**: Implement tests for full and partial refund flows
  - **Files**: 
    - `crates/router/tests/connectors/monei.rs`: Implement refund test cases
  - **Step Dependencies**: Step 29, Step 21, Step 22
  - **User Instructions**: None

## Phase 12: Final Integration and Cleanup
- [ ] Step 31: Add Monei to supported connectors configuration
  - **Task**: Update configuration files to include Monei as a supported connector
  - **Files**: 
    - `config/config.example.toml`: Add Monei configuration section
    - `config/development.toml`: Add Monei configuration for development
  - **Step Dependencies**: All implementation steps
  - **User Instructions**: Update your local configuration files with Monei settings

- [ ] Step 32: Implement webhook support (optional)
  - **Task**: Add webhook signature verification if documentation becomes available
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Add webhook verification
  - **Step Dependencies**: Step 31
  - **User Instructions**: Skip if webhook signature details not available

- [ ] Step 33: Documentation and final testing
  - **Task**: Add inline documentation and run full test suite
  - **Files**: 
    - `crates/hyperswitch_connectors/src/connectors/monei.rs`: Add documentation
    - `crates/hyperswitch_connectors/src/connectors/monei/transformers.rs`: Add documentation
  - **Step Dependencies**: All previous steps
  - **User Instructions**: Run: `cargo test --package router --test connectors -- monei --test-threads=1`

# Summary

This implementation plan for the Monei connector integration follows a systematic approach:

1. **Initial Setup**: Generate boilerplate code and organize files
2. **Authentication**: Implement API key-based Bearer token authentication
3. **Core Traits**: Implement fundamental connector traits
4. **Type Definitions**: Define all request/response structures
5. **Payment Flows**: Implement authorize, capture, and sync operations
6. **Refund Flows**: Implement refund creation and synchronization
7. **Error Handling**: Comprehensive error code and status mapping
8. **Utilities**: Helper functions for common transformations
9. **Testing**: Complete test coverage for all flows
10. **Integration**: Final configuration and documentation

Key considerations:
- Monei requires payment tokens for card payments (generated client-side)
- Support for both AUTH and SALE transaction types
- Comprehensive error code mapping (E000-E599)
- Partial capture and refund support
- Status determination considers refunded amounts
- Test mode uses same base URL with test API key

The implementation follows Hyperswitch patterns and reuses existing utilities for amount conversion and common transformations.
