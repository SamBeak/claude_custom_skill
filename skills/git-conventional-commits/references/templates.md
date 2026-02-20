# Commit Message Templates

Comprehensive collection of commit message templates following Conventional Commits specification for various scenarios.

## Table of Contents

1. [Basic Templates](#basic-templates)
2. [Feature Templates](#feature-templates)
3. [Bug Fix Templates](#bug-fix-templates)
4. [Refactoring Templates](#refactoring-templates)
5. [Documentation Templates](#documentation-templates)
6. [Performance Templates](#performance-templates)
7. [Testing Templates](#testing-templates)
8. [Build & CI Templates](#build--ci-templates)
9. [Breaking Changes](#breaking-changes)
10. [Special Scenarios](#special-scenarios)

## Basic Templates

### Minimal (Subject Only)
```
<type>(<scope>): <subject>
```

Example:
```
feat(auth): add login functionality
```

### With Body
```
<type>(<scope>): <subject>

<body>
```

Example:
```
feat(auth): add login functionality

Implement email/password authentication with JWT tokens.
```

### Complete (Subject + Body + Footer)
```
<type>(<scope>): <subject>

<body>

<footer>
```

Example:
```
feat(auth): add login functionality

Implement email/password authentication with JWT tokens.
Users can now log in using their registered credentials.

Closes #123
```

## Feature Templates

### Simple Feature
```
feat(<scope>): add <feature>
```

Examples:
```
feat(button): add ripple effect animation
feat(api): add rate limiting middleware
feat(ui): add dark mode toggle
```

### Feature with Details
```
feat(<scope>): add <feature>

Implement <description of feature>. Users can now <action>.

- <detail 1>
- <detail 2>
- <detail 3>
```

Example:
```
feat(dashboard): add real-time notifications

Implement WebSocket-based notification system.
Users can now receive instant updates for new messages.

- Add WebSocket connection management
- Add notification badge on navbar
- Add notification preferences in settings
```

### Feature with User Story
```
feat(<scope>): add <feature>

As a <user type>, I want <goal> so that <benefit>.

Implementation:
- <technical detail 1>
- <technical detail 2>
```

Example:
```
feat(export): add CSV export for reports

As a data analyst, I want to export reports to CSV
so that I can analyze data in Excel.

Implementation:
- Add export button to reports page
- Generate CSV with all visible columns
- Include filters in filename
```

### Feature with Dependencies
```
feat(<scope>): add <feature>

<description>

Depends on:
- <dependency 1>
- <dependency 2>

Closes #<issue>
```

Example:
```
feat(payment): add Stripe integration

Implement payment processing using Stripe API.
Supports credit cards and digital wallets.

Depends on:
- Backend API for payment endpoints
- User authentication system

Closes #456
```

## Bug Fix Templates

### Simple Bug Fix
```
fix(<scope>): <what was fixed>
```

Examples:
```
fix(login): prevent password field autocomplete
fix(api): handle null response from database
fix(ui): correct button alignment on mobile
```

### Bug Fix with Root Cause
```
fix(<scope>): <what was fixed>

<description of bug>
<root cause>
<solution>

Fixes #<issue>
```

Example:
```
fix(auth): prevent token refresh race condition

Multiple simultaneous API calls triggered multiple token
refresh requests, causing auth failures.

Root cause: No mutex lock on token refresh logic.

Solution: Add semaphore to ensure only one refresh
request executes at a time.

Fixes #789
```

### Bug Fix with Reproduction Steps
```
fix(<scope>): <what was fixed>

Bug occurred when:
1. <step 1>
2. <step 2>
3. <step 3>

Fixed by: <solution>

Fixes #<issue>
```

Example:
```
fix(cart): prevent duplicate items on rapid clicks

Bug occurred when:
1. User clicked "Add to Cart" multiple times rapidly
2. Each click sent separate API request
3. Cart showed duplicate items

Fixed by: Debounce button clicks with 500ms delay

Fixes #234
```

### Security Fix
```
fix(<scope>): prevent <vulnerability>

<description of security issue>
<impact>
<solution>

Security advisory: <link>
Fixes #<issue>
```

Example:
```
fix(api): prevent SQL injection in search query

User input was concatenated directly into SQL queries,
allowing potential SQL injection attacks.

Impact: Attackers could access or modify database records.

Solution: Use parameterized queries for all user input.

Security advisory: docs/security/CVE-2024-001.md
Fixes #567
```

## Refactoring Templates

### Simple Refactor
```
refactor(<scope>): <what was refactored>
```

Examples:
```
refactor(auth): extract validation logic to helper
refactor(api): simplify error handling middleware
refactor(ui): migrate class components to hooks
```

### Refactor with Rationale
```
refactor(<scope>): <what was refactored>

Previous implementation: <description>
Issues: <what was wrong>

New implementation: <description>
Benefits: <improvements>
```

Example:
```
refactor(database): extract query builders to separate module

Previous implementation: Query building logic scattered
across model classes.
Issues: Hard to maintain, duplicated code, difficult to test.

New implementation: Dedicated QueryBuilder classes in
database/builders directory.
Benefits: Centralized logic, easier testing, reusable builders.
```

### Refactor for Performance
```
refactor(<scope>): optimize <what>

Refactor <component> to improve <metric>.

Before: <previous performance>
After: <new performance>
```

Example:
```
refactor(images): optimize loading strategy

Refactor image component to use progressive loading.

Before: All images loaded on mount, 5s initial load time
After: Lazy loading with blur placeholder, 1.5s initial load

Improves Lighthouse score from 68 to 92.
```

## Documentation Templates

### README Update
```
docs(README): <what was updated>
```

Examples:
```
docs(README): add installation instructions
docs(README): update API endpoint documentation
docs(README): add troubleshooting section
```

### API Documentation
```
docs(api): <what was documented>

- <endpoint 1>: <description>
- <endpoint 2>: <description>
```

Example:
```
docs(api): document authentication endpoints

- POST /auth/login: User login with credentials
- POST /auth/refresh: Refresh access token
- POST /auth/logout: Invalidate user session

Include request/response examples and error codes.
```

### Code Comments
```
docs(<scope>): add comments to <component>

Improve code documentation for better maintainability.

- <what was commented 1>
- <what was commented 2>
```

Example:
```
docs(validation): add JSDoc comments to validators

Improve code documentation for better maintainability.

- Document all validation functions
- Add parameter types and return values
- Include usage examples
```

### Migration Guide
```
docs: add migration guide for <version>

Document breaking changes and migration steps from
<old version> to <new version>.

Covers:
- <topic 1>
- <topic 2>
```

Example:
```
docs: add migration guide for v2.0

Document breaking changes and migration steps from
v1.x to v2.0.

Covers:
- API endpoint changes
- Authentication flow updates
- Database schema migrations
- Configuration changes
```

## Performance Templates

### Performance Optimization
```
perf(<scope>): <what was optimized>

Optimize <component> by <technique>.

Impact: <performance improvement>
```

Example:
```
perf(dashboard): optimize data fetching

Optimize dashboard by implementing data pagination
and caching strategy.

Impact: Initial load time reduced from 8s to 2s.
```

### Bundle Size Reduction
```
perf: reduce bundle size by <amount>

- <optimization 1>
- <optimization 2>
- <optimization 3>

Before: <old size>
After: <new size>
```

Example:
```
perf: reduce bundle size by 40%

- Implement code splitting for routes
- Replace moment.js with date-fns
- Remove unused lodash imports
- Lazy load chart libraries

Before: 850KB
After: 510KB
```

### Database Optimization
```
perf(database): optimize <query/table>

<description of problem>
<optimization technique>

Impact: <improvement>
```

Example:
```
perf(database): optimize user search query

Previous query scanned entire users table for each search,
causing slow performance with large datasets.

Add composite index on (email, username, status) columns.

Impact: Search time reduced from 2.5s to 150ms for 100K users.
```

## Testing Templates

### Add Tests
```
test(<scope>): add tests for <component>

- <test type 1>: <description>
- <test type 2>: <description>

Coverage: <percentage>%
```

Example:
```
test(auth): add tests for login flow

- Unit tests: Token validation, password hashing
- Integration tests: Login API endpoint
- E2E tests: Complete login flow in browser

Coverage: 92%
```

### Fix Failing Tests
```
test(<scope>): fix failing <test name>

Test failed because: <reason>
Fixed by: <solution>
```

Example:
```
test(api): fix failing user creation test

Test failed because: Test database not properly reset
between test runs, causing unique constraint violations.

Fixed by: Add beforeEach hook to truncate users table.
```

### Update Tests
```
test(<scope>): update tests for <change>

Update tests to reflect changes from <related commit/PR>.

- <updated test 1>
- <updated test 2>
```

Example:
```
test(auth): update tests for new token expiration

Update tests to reflect changes from OAuth2 implementation.

- Update token TTL expectations
- Add tests for refresh token rotation
- Update mock token generation
```

## Build & CI Templates

### Dependency Update
```
build(deps): update <dependency> to <version>

Update <dependency> from <old version> to <new version>.

Changes:
- <notable change 1>
- <notable change 2>
```

Example:
```
build(deps): update React to v18.2.0

Update React from v17.0.2 to v18.2.0.

Changes:
- New automatic batching improves performance
- Concurrent features available
- Strict mode improvements

No breaking changes for our codebase.
```

### Build Configuration
```
build: <what was configured>

<description>

Benefits:
- <benefit 1>
- <benefit 2>
```

Example:
```
build: add TypeScript compilation

Configure TypeScript compiler for production builds.

Benefits:
- Type checking at build time
- Better IDE support
- Smaller bundle size with tree shaking
```

### CI/CD Pipeline
```
ci: <what was changed>

<description>

- <change 1>
- <change 2>
```

Example:
```
ci: add automated testing workflow

Add GitHub Actions workflow to run tests on every PR.

- Run unit tests on Node 16, 18, 20
- Run integration tests with test database
- Upload coverage to Codecov
- Block merge if coverage drops below 80%
```

## Breaking Changes

### API Breaking Change
```
feat(<scope>)!: <change description>

<detailed description>

BREAKING CHANGE: <what broke>

Migration:
- <step 1>
- <step 2>

Before:
<code example>

After:
<code example>
```

Example:
```
feat(api)!: change response format to JSON:API spec

Migrate all API endpoints to follow JSON:API specification
for consistency and better tooling support.

BREAKING CHANGE: Response structure has changed for all endpoints.

Migration:
- Update client code to expect data in {data: ...} wrapper
- Handle errors in {errors: [...]} array format
- Update relationship parsing logic

Before:
{
  "user": {...},
  "posts": [...]
}

After:
{
  "data": {
    "type": "user",
    "attributes": {...},
    "relationships": {
      "posts": {
        "data": [...]
      }
    }
  }
}

Migration guide: docs/api-v2-migration.md
```

### Removed Feature
```
feat(<scope>)!: remove <feature>

Remove <feature> due to <reason>.

BREAKING CHANGE: <what was removed>

Alternative: <what to use instead>
```

Example:
```
feat(api)!: remove legacy authentication endpoints

Remove v1 authentication endpoints that have been deprecated
since v1.5.0 (6 months ago).

BREAKING CHANGE: /api/v1/auth/* endpoints no longer exist.

Alternative: Use /api/v2/auth/* endpoints with JWT tokens.
Migration guide: docs/auth-migration.md
```

### Behavior Change
```
fix(<scope>)!: change <behavior>

Previous behavior: <description>
Issue: <why it was wrong>

New behavior: <description>

BREAKING CHANGE: <impact on users>
```

Example:
```
fix(validation)!: change email validation to be case-sensitive

Previous behavior: Email addresses were case-insensitive
Issue: Allowed duplicate accounts with different casing

New behavior: Email validation is now case-sensitive

BREAKING CHANGE: Users with emails differing only in case
need to merge accounts or choose one.

Data migration: npm run migrate:email-dedupe
```

## Special Scenarios

### Revert Commit
```
revert: <type>(<scope>): <original subject>

This reverts commit <hash>.

Reason: <why reverting>
```

Example:
```
revert: feat(auth): add OAuth2 support

This reverts commit abc123def456.

Reason: OAuth2 implementation causes login failures in
production. Reverting to investigate and fix issue.

Refs #789
```

### Merge Commit
```
Merge branch '<branch-name>' into <target-branch>
```

Example:
```
Merge branch 'feature/oauth2' into main
```

Or with details:
```
Merge branch 'feature/oauth2' into main

Add OAuth2 authentication support

Closes #456
```

### Initial Commit
```
chore: initial commit

Initialize project with <description>.

Includes:
- <item 1>
- <item 2>
- <item 3>
```

Example:
```
chore: initial commit

Initialize React project with TypeScript and Vite.

Includes:
- Project structure
- Base configuration
- Development dependencies
- README with setup instructions
```

### Release Commit
```
chore: release v<version>

Release version <version> with <summary of changes>.

See CHANGELOG.md for full details.
```

Example:
```
chore: release v2.0.0

Release version 2.0.0 with new authentication system
and performance improvements.

See CHANGELOG.md for full details.
```

### Hotfix
```
fix(<scope>): <critical fix> [hotfix]

Critical production issue: <description>
Impact: <who/what affected>
Fix: <solution>

Deployed to production: <timestamp>
```

Example:
```
fix(payment): prevent charge duplication [hotfix]

Critical production issue: Payment API called multiple times
for single transaction, causing duplicate charges.

Impact: 23 customers charged twice in last hour.

Fix: Add idempotency key to payment requests.

Deployed to production: 2024-01-15 14:30 UTC
Fixes #999
```

### WIP Commit (Feature Branch Only)
```
WIP: <description>
```

Example:
```
WIP: implementing OAuth2 authentication

Not ready for review. Need to:
- [ ] Add tests
- [ ] Handle edge cases
- [ ] Update documentation
```

Note: Squash WIP commits before merging to main.

### Multiple Authors
```
<type>(<scope>): <subject>

<body>

Co-authored-by: <Name> <<email>>
Co-authored-by: <Name> <<email>>
```

Example:
```
feat(api): add GraphQL endpoint

Implement GraphQL API with Apollo Server.

Co-authored-by: John Doe <john@example.com>
Co-authored-by: Jane Smith <jane@example.com>
```

## Language-Specific Templates

### JavaScript/TypeScript
```
feat(components): add <ComponentName> component

Implement <ComponentName> component with <features>.

Props:
- <prop1>: <type> - <description>
- <prop2>: <type> - <description>

Usage:
<ComponentName prop1="value" prop2={value} />
```

### Python
```
feat(module): add <function_name> function

Implement <function_name> to <purpose>.

Parameters:
- <param1> (<type>): <description>
- <param2> (<type>): <description>

Returns:
- <type>: <description>
```

### Java
```
feat(service): add <ServiceName> service

Implement <ServiceName> service for <purpose>.

Methods:
- <method1>(<params>): <description>
- <method2>(<params>): <description>

Dependencies:
- <dependency1>
- <dependency2>
```

## Framework-Specific Templates

### React
```
feat(hooks): add use<HookName> hook

Implement use<HookName> custom hook for <purpose>.

Returns:
- <return1>: <description>
- <return2>: <description>

Example usage:
const {value, setValue} = use<HookName>();
```

### Next.js
```
feat(pages): add <page-name> page

Implement <page-name> page at /<route>.

Features:
- Server-side rendering
- SEO optimization
- Loading states

Route: /<route>
```

### Spring Boot
```
feat(controller): add <ControllerName> controller

Implement REST endpoints for <resource> management.

Endpoints:
- GET /<resource>: List all
- POST /<resource>: Create new
- GET /<resource>/:id: Get by ID
- PUT /<resource>/:id: Update
- DELETE /<resource>/:id: Delete
```

## Commit Message Checklist

Before finalizing commit message:

- [ ] Type is correct (feat/fix/docs/etc)
- [ ] Scope is appropriate and consistent
- [ ] Subject uses imperative mood
- [ ] Subject is lowercase (except proper nouns)
- [ ] Subject has no period at end
- [ ] Subject is ≤ 50 characters
- [ ] Body wraps at 72 characters
- [ ] Body explains what and why (not how)
- [ ] Footer references issues if applicable
- [ ] Breaking changes noted with BREAKING CHANGE
- [ ] Co-authors added if applicable
- [ ] No sensitive information in message
