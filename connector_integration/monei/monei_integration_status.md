# Monei Connector Integration Status

## Completed Tasks ✓

### Phase 1: Planning and Specification
- [x] **Moved .clinerules to root directory** - Completed as per .gracerules initial step
- [x] **Created Technical Specification** - `grace/connector_integration/monei/monei_specs.md`
  - Documented all API endpoints and authentication
  - Defined request/response structures
  - Mapped status and error codes
  - Specified implementation details
- [x] **Created Implementation Plan** - `grace/connector_integration/monei_plan.md`
  - 33 detailed implementation steps
  - Organized into 12 phases
  - Clear dependencies and user instructions

## Key Integration Details

### Authentication
- **Type**: Bearer Token with API Key
- **Header**: `Authorization: Bearer {api_key}`

### Supported Flows
1. **Payment Authorization** - POST /payments (AUTH or SALE)
2. **Payment Capture** - POST /payments/{id}/capture
3. **Payment Sync** - GET /payments/{id}
4. **Refund Execute** - POST /payments/{id}/refund
5. **Refund Sync** - GET /payments/{id} (check refundedAmount)

### Special Considerations
- **Payment Token Required**: Card payments require a paymentToken generated via monei.js
- **Session ID**: Required for temporary tokens validation
- **Transaction Types**: Supports both AUTH (authorize only) and SALE (direct capture)
- **Partial Operations**: Supports partial capture and partial refunds
- **Error Codes**: Comprehensive mapping for E000-E599 codes
- **3DS Support**: Error codes E300-E309 indicate 3DS authentication required

## Next Steps

1. **Execute Implementation Plan**
   - Run `./scripts/add_connector.sh monei https://api.monei.com/v1`
   - Follow the 33 steps in `monei_plan.md`

2. **Update Documentation**
   - Update guides/types/types.md with Monei-specific types
   - Document any patterns discovered in guides/patterns/patterns.md
   - Record errors and solutions in guides/errors/errors.md
   - Add learnings to guides/learning/learning.md

3. **Testing**
   - Configure test API key in sample_auth.toml
   - Implement all 20 sanity tests
   - Test partial capture/refund scenarios

## Reference Documents
- **API Documentation**: [Monei API Docs](https://docs.monei.com/)
- **Integration Spec**: `grace/connector_integration/monei/monei_specs.md`
- **Implementation Plan**: `grace/connector_integration/monei_plan.md`
- **Original Reference**: `grace/references/MONEI Integration Documentation.markdown`

## Status: Ready for Implementation
All planning documents have been created. The implementation can now proceed following the detailed plan.
