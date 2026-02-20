# Plan Review Analysis Guide

This guide provides detailed methodology for analyzing Claude Code plan files and conducting comprehensive reviews using OpenAI Codex CLI.

## Table of Contents

1. [Review Workflow Overview](#review-workflow-overview)
2. [Phase 1: Plan Ingestion & Parsing](#phase-1-plan-ingestion--parsing)
3. [Phase 2: Multi-Perspective Analysis](#phase-2-multi-perspective-analysis)
4. [Phase 3: Findings Synthesis](#phase-3-findings-synthesis)
5. [Phase 4: Plan Enhancement](#phase-4-plan-enhancement)
6. [Review Quality Checklist](#review-quality-checklist)

## Review Workflow Overview

The plan review process follows four phases:

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Phase 1     │    │  Phase 2     │    │  Phase 3     │    │  Phase 4     │
│  Ingestion   │───▶│  Analysis    │───▶│  Synthesis   │───▶│  Enhancement │
│  & Parsing   │    │  (7 views)   │    │  & Scoring   │    │  & Merging   │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

1. **Ingestion** - Locate, read, and parse the plan file
2. **Analysis** - Review from 7 perspectives via Codex CLI
3. **Synthesis** - Classify and prioritize findings
4. **Enhancement** - Merge review results into the plan file

## Phase 1: Plan Ingestion & Parsing

### 1.1 Locate Plan File

```bash
# Auto-detect: find most recent .md file in plans directory
ls -t ~/.claude/plans/*.md 2>/dev/null | head -1

# Check current project plans
ls -t .claude/plans/*.md 2>/dev/null | head -1
```

### 1.2 Parse Plan Structure

Identify key elements in the plan:

- **Title and description** - What is being built
- **Context section** - Background and motivation
- **File list** - Files to create or modify
- **Implementation steps** - Ordered sequence of actions
- **Design decisions** - Key architectural choices
- **Error handling strategy** - How failures are addressed
- **Verification method** - How to validate the result

### 1.3 Extract Core Components

```
Plan Element          │ What to Look For
──────────────────────┼─────────────────────────────
Goal                  │ What the plan aims to achieve
Scope                 │ Boundaries of the implementation
Dependencies          │ External tools, libraries, services
File Changes          │ New files, modified files, deleted files
Architecture          │ Design patterns, data flow
Security Boundary     │ Auth, data access, input handling
Performance Concern   │ Scale, latency, resource usage
Test Coverage         │ What is tested and how
```

### 1.4 Identify Plan Completeness Signals

A well-structured plan should have:
- Clear objective statement
- Ordered implementation steps
- File-level change descriptions
- Error handling considerations
- Verification/testing approach

Missing elements are noted as findings in Phase 2.

## Phase 2: Multi-Perspective Analysis

### 2.1 Technical Feasibility

#### Checklist
- [ ] All implementation steps are present and ordered correctly
- [ ] No circular dependencies between steps
- [ ] Technology choices are appropriate for the use case
- [ ] File paths and naming conventions are consistent
- [ ] No implicit assumptions left unstated
- [ ] Edge cases are identified and handled

#### Key Questions
1. Can each step be executed with the information provided?
2. Are there missing intermediate steps?
3. Is the order of operations correct?
4. Are the proposed APIs/interfaces well-defined?
5. Does the plan account for existing codebase constraints?
6. Are there hidden complexity areas?

#### Example Findings
```
[High] Step 3 depends on a database table that is created in Step 5.
       Recommendation: Reorder to create the table before referencing it.

[Medium] Plan assumes Redis is available but does not include setup steps.
         Recommendation: Add Redis installation/configuration step or
         document it as a prerequisite.
```

### 2.2 Security

#### Checklist
- [ ] OWASP Top 10 vulnerabilities addressed
- [ ] Authentication mechanisms are secure
- [ ] Authorization checks are in place
- [ ] User input is validated and sanitized
- [ ] No injection vulnerabilities (SQL, XSS, command)
- [ ] Sensitive data is encrypted at rest and in transit
- [ ] Secrets are not hardcoded
- [ ] CORS and CSP policies considered
- [ ] Rate limiting implemented where needed
- [ ] Audit logging for sensitive operations

#### Key Questions
1. Where does user input enter the system?
2. How is authentication handled?
3. Are there privilege escalation risks?
4. Is data encrypted appropriately?
5. Are there exposed secrets or credentials?
6. Are third-party dependencies secure?

#### Example Findings
```
[Critical] API endpoint accepts raw SQL in query parameter without
           sanitization. Recommendation: Use parameterized queries.

[High] JWT tokens are stored in localStorage, vulnerable to XSS.
       Recommendation: Use httpOnly cookies instead.
```

### 2.3 Performance

#### Checklist
- [ ] No N+1 query patterns
- [ ] Caching strategy defined for frequently accessed data
- [ ] Async/parallel processing for independent operations
- [ ] Database indexes planned for query patterns
- [ ] Pagination for large result sets
- [ ] Connection pooling configured
- [ ] No unbounded memory growth patterns
- [ ] Resource cleanup (file handles, connections) ensured
- [ ] Bundle size / payload size considered
- [ ] CDN usage for static assets

#### Key Questions
1. What are the expected data volumes?
2. Where are the potential bottlenecks?
3. Is there unnecessary data fetching?
4. Can operations be batched or parallelized?
5. Are there memory-intensive operations?
6. What is the expected response time requirement?

#### Example Findings
```
[High] Loading all user records into memory for filtering.
       Recommendation: Move filtering to database query with
       appropriate indexes.

[Medium] No caching for API responses that rarely change.
         Recommendation: Add Redis cache with TTL for
         configuration endpoints.
```

### 2.4 Test Strategy

#### Checklist
- [ ] Unit tests for business logic
- [ ] Integration tests for component interactions
- [ ] E2E tests for critical user journeys
- [ ] Edge case scenarios covered
- [ ] Error/failure scenarios tested
- [ ] Mock strategy for external dependencies
- [ ] Test data management strategy
- [ ] Coverage targets defined
- [ ] CI pipeline integration planned
- [ ] Performance/load test consideration

#### Key Questions
1. Are all critical paths tested?
2. What happens when external services fail?
3. Are boundary conditions tested?
4. Is the mocking strategy realistic?
5. Can tests run independently and in parallel?
6. What is the minimum acceptable coverage?

#### Example Findings
```
[High] No integration tests for the payment processing flow.
       Recommendation: Add integration tests with mocked payment
       gateway to cover success, failure, and timeout scenarios.

[Medium] Test plan does not include load testing for the new API.
         Recommendation: Add k6 or Artillery load test for
         expected peak traffic.
```

### 2.5 Alternative Approaches

#### Checklist
- [ ] Simpler solutions considered
- [ ] Established design patterns evaluated
- [ ] Existing libraries/frameworks compared
- [ ] Build vs. buy analysis done
- [ ] Migration path complexity assessed
- [ ] Team expertise alignment checked

#### Key Questions
1. Is this the simplest solution that works?
2. Are there well-tested libraries for this use case?
3. Is the custom implementation justified?
4. Could a different architecture reduce complexity?
5. Are there proven patterns in the ecosystem?
6. Does the approach align with team capabilities?

#### Example Findings
```
[Medium] Custom authentication system being built when
         established solutions exist. Recommendation: Consider
         using Auth0, Supabase Auth, or NextAuth.js to reduce
         implementation effort and improve security.

[Low] Custom date formatting utility when date-fns is already
      in dependencies. Recommendation: Use date-fns format()
      instead of building custom formatter.
```

### 2.6 Dependencies

#### Checklist
- [ ] All dependencies explicitly listed with versions
- [ ] No known critical vulnerabilities (CVE)
- [ ] License compatibility verified
- [ ] Bundle size impact assessed
- [ ] Maintenance status of dependencies checked
- [ ] Version conflicts identified and resolved
- [ ] Lock file strategy defined
- [ ] Peer dependency requirements met

#### Key Questions
1. Are all required dependencies identified?
2. Are there version conflicts between dependencies?
3. Are the licenses compatible with the project?
4. Are dependencies actively maintained?
5. Could lighter alternatives reduce bundle size?
6. Are there known security advisories?

#### Example Findings
```
[High] Using lodash (full bundle) for only 2 utility functions.
       Recommendation: Import specific functions (lodash/debounce)
       or use native alternatives.

[Medium] Dependency X has not been updated in 18 months and
         has 3 open security advisories. Recommendation: Evaluate
         alternative library Y which is actively maintained.
```

### 2.7 Error Handling

#### Checklist
- [ ] All failure modes identified
- [ ] Recovery strategy for each failure mode
- [ ] User-friendly error messages defined
- [ ] Logging strategy for debugging
- [ ] Monitoring and alerting considered
- [ ] Timeout values defined for external calls
- [ ] Retry strategy with backoff for transient failures
- [ ] Circuit breaker pattern for dependent services
- [ ] Graceful degradation for non-critical features
- [ ] Data consistency in failure scenarios

#### Key Questions
1. What happens when the database is unavailable?
2. What happens when an external API times out?
3. How are partial failures handled?
4. Are error messages helpful for debugging?
5. Is there sufficient logging for production issues?
6. Are there cascading failure risks?

#### Example Findings
```
[Critical] No error handling for database connection failure in
           the startup sequence. Application will crash without
           meaningful error message. Recommendation: Add connection
           retry with exponential backoff and clear error logging.

[High] External API calls have no timeout configured.
       Recommendation: Set 30-second timeout with retry
       (max 3 attempts, exponential backoff).
```

## Phase 3: Findings Synthesis

### Severity Classification

| Severity | Definition | Action Required |
|----------|-----------|----------------|
| **Critical** | Blocks implementation or causes data loss/security breach | Must fix before proceeding |
| **High** | Significant risk to quality, security, or performance | Should fix before implementation |
| **Medium** | Improvement opportunity with moderate impact | Recommended to address |
| **Low** | Minor improvement or style suggestion | Nice to have |
| **Info** | Observation or context, no action needed | For awareness |

### Prioritization Matrix

```
              High Impact         Low Impact
           ┌─────────────────┬─────────────────┐
High Effort│   Schedule       │   Deprioritize   │
           │   (plan for it)  │   (skip or later) │
           ├─────────────────┼─────────────────┤
Low Effort │   Do First       │   Quick Win       │
           │   (immediate)    │   (if time allows) │
           └─────────────────┴─────────────────┘
```

### Counting Rules

- **Critical Issues**: Findings that block implementation or pose severe risk
- **High Issues**: Findings that significantly affect quality or security
- **Medium Issues**: Findings that improve the plan but are not blocking
- **Recommendations**: All actionable suggestions (Low + Info with suggestions)

## Phase 4: Plan Enhancement

### Merge Strategy

1. **Preserve Original** - Never modify existing plan content
2. **Append Review** - Add review section at the end with horizontal rule separator
3. **Structured Format** - Use consistent heading structure
4. **Attribution** - Include Codex model and timestamp
5. **Actionable Items** - End with prioritized checklist

### Merge Format Rules

- Use `---` (horizontal rule) to separate original plan from review
- Start with `## Codex Review (Generated: YYYY-MM-DD HH:MM)` heading
- Include blockquote with model attribution
- Use `### ` (h3) for each review perspective
- Use severity tags in brackets: `[Critical]`, `[High]`, `[Medium]`, `[Low]`, `[Info]`
- End with `### Action Items` as prioritized checklist

### Post-Merge Verification

After merging review results:
1. Original plan content is unchanged
2. Review section is properly formatted
3. All 7 perspectives are included (even if "No issues found")
4. Summary counts match actual findings
5. Action items are prioritized correctly
6. Timestamp and model info are accurate

## Review Quality Checklist

Before finalizing the review:

- [ ] All 7 review perspectives are addressed
- [ ] Severity levels are consistently applied
- [ ] Summary counts are accurate
- [ ] Recommendations are actionable and specific
- [ ] No duplicate findings across perspectives
- [ ] Action items are prioritized (Critical > High > Medium)
- [ ] Original plan content is preserved
- [ ] Review section format is correct
- [ ] Timestamp and model attribution are present
- [ ] Language is professional and constructive
