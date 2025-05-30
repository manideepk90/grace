You are an expert code reviewer tasked with analyzing the {{connector_name}} connector implementation and creating a detailed optimization plan. Your review should focus on code quality, performance, security, and integration with the Hyperswitch platform.
Please review the following context and implementation:

<project_request>
Integration of the {{connector_name}} connector to Hyperswitch
</project_request>

<project_rules>
1. **Type Safety**: Use `types.rs` in `hyperswitch_domain_models` for type definitions
2. **Code Standards**: Follow existing connector patterns and maintain consistent code standards
3. **No Assumptions**: Do not assume implementation details; refer to documentation
4. **Reuse Components**: 
   - Use existing amount conversion utilities from common utils
   - Do not create new amount conversion code
5. **Boilerplate Generation**: Use `add_connector.sh {{connector_name}} {{connector_base_url}}` to generate initial code
6. **File Organization**: Move `crates/hyperswitch_connectors/src/connectors/{{connector_name}}/test.rs` to `crates/router/tests/connectors/{{connector_name}}.rs`
7. **API Types**: Define connector-specific request/response types and conversions based on the connector's actual API
8. **Implementation**: Complete all `todo!()` markers in the boilerplate code
9. Should not fix errors based on the rust compiler suggestions [REMEMBER]
10. Fix errors from the reference docs [REMEMBER]
11. Always use cargo build in root folder. rather than using cargo build in other folder.
12. After every phase it has to do cargo build/check and fix the errors.
13. For Testing, create the cypress test as the other connector in the repo for the implemented flows.
14. For Fixing issues of rust, follow the types.md, errors.md then rust suggestion. [MUST]
15. After every step update the doc and mark as done if completed [MUST REMEMBER]
16. Always Follow full request/response struct from the {{connector_name}} doc. don't modify the payloads.
</project_rules>

<reference_docs>
| Document | Purpose |
| `grace/guides/types/types.md` | Type definitions and data structures |
| `grace/guides/integrations/integrations.md` | Connector implementation patterns |
| `grace/guides/learning/learning.md` | Lessons from previous integrations |
| `grace/guides/patterns/patterns.md` | Common implementation patterns |
| `grace/guides/errors/errors.md` | Error handling strategies |
</reference_docs>
<connector_information>
`grace/references/{{connector_name}}_doc_*.md` -> Connector-specific API documentation
</connector_information>

<technical_specification>
`grace/connector_integration/{{connector_name}}_specs.md` -> Technical specifications
</technical_specification>
<implementation_plan>
`grace/connector_integration/{{connector_name}}_plan.md` -> Implementation Plan
</implementation_plan>

<existing_code>
hyperswitch_connectors/src/connectors/
├── {{connector_name}}/
│   └── transformers.rs    # Request/Response transformations
└── {{connector_name}}.rs   # Main connector implementation
</existing_code>
First, analyze the implemented code against the original requirements and plan. Consider the following areas:

### 1. Implementation Completeness
- Are all required payment flows implemented?
- Are all `todo!()` markers resolved?
- Does implementation match the original plan?

### 2. Code Structure and Organization
- Is code properly modularized?
- Is there clear separation of concerns?
- Are files organized according to project standards?

### 3. Code Quality
- Does the code follow Rust best practices?
- Are there any anti-patterns present?
- Is naming consistent and descriptive?
- Is code DRY (Don't Repeat Yourself)?

### 4. Type Safety and Error Handling
- Are appropriate types used throughout?
- Is error handling comprehensive and consistent?
- Are error messages descriptive and actionable?

### 5. Performance and Security
- Are there any potential performance bottlenecks?
- Are there security vulnerabilities (e.g., missing validation)?
- Is sensitive data properly handled?

### 6. Testing
- Are all flows adequately tested?
- Are edge cases and error scenarios covered?
- Are tests clear and maintainable?

## Analysis and Optimization Plan Format

First, provide your analysis within <analysis> tags:

```
<analysis>
Detailed assessment of the current implementation, highlighting strengths and areas for improvement.
Include specific examples and references to the codebase.
</analysis>
```

Then, create a detailed optimization plan using the following format:

```md
# Optimization Plan
## [Category Name]
- [ ] Step 1: [Brief title]
  - **Task**: [Detailed explanation of what needs to be optimized/improved]
  - **Files**: [List of files]
    - `path/to/file1.rs`: [Description of changes]
  - **Step Dependencies**: [Any steps that must be completed first]
  - **User Instructions**: [Any manual steps required]
[Additional steps...]
```

For each step in your plan:
1. Focus on specific, concrete improvements
2. Keep changes manageable (no more than 20 files per step, ideally less)
3. Ensure steps build logically on each other
4. Preserve starter template code and patterns
5. Maintain existing functionality
6. Follow project rules and technical specifications

Your plan should be detailed enough for a code generation AI to implement each step in a single iteration. Order steps by priority and dependency requirements.

Remember:
- Focus on implemented code, not starter template code
- Maintain consistency with existing patterns
- Ensure each step is atomic and self-contained
- Include clear success criteria for each step
- Consider the impact of changes on the overall system

Begin your response with your analysis of the current implementation, then proceed to create your detailed optimization plan.