# Analysis Guide for Commit Message Generation

This guide provides detailed methodology for analyzing staged changes and generating appropriate Conventional Commits messages.

## Table of Contents

1. [Initial Analysis](#initial-analysis)
2. [Type Determination](#type-determination)
3. [Scope Identification](#scope-identification)
4. [Subject Generation](#subject-generation)
5. [Body Composition](#body-composition)
6. [Footer Construction](#footer-construction)
7. [Special Cases](#special-cases)

## Initial Analysis

### 1. Check Staged Changes

```bash
# Get list of staged files
git diff --staged --name-only

# Get detailed changes
git diff --staged

# Get statistics
git diff --staged --stat
```

### 2. Categorize Files by Change Type

Group files into categories:

```
Added files:    New functionality
Modified files: Changes to existing functionality
Deleted files:  Removed functionality
Renamed files:  Refactoring or reorganization
```

### 3. Read File Contents

For each modified file:
- Read the file to understand context
- Identify the purpose of the code
- Understand what the changes do
- Note any breaking changes

## Type Determination

### Decision Tree

```
Is this a new feature visible to users?
├─ YES → feat
└─ NO
   ├─ Does it fix a bug?
   │  ├─ YES → fix
   │  └─ NO
   │     ├─ Does it only change documentation?
   │     │  ├─ YES → docs
   │     │  └─ NO
   │     │     ├─ Does it only change formatting/style?
   │     │     │  ├─ YES → style
   │     │     │  └─ NO
   │     │     │     ├─ Does it restructure code without changing behavior?
   │     │     │     │  ├─ YES → refactor
   │     │     │     │  └─ NO
   │     │     │     │     ├─ Does it improve performance?
   │     │     │     │     │  ├─ YES → perf
   │     │     │     │     │  └─ NO
   │     │     │     │     │     ├─ Does it add/modify tests?
   │     │     │     │     │     │  ├─ YES → test
   │     │     │     │     │     │  └─ NO
   │     │     │     │     │     │     ├─ Build/CI related?
   │     │     │     │     │     │     │  ├─ YES → build/ci
   │     │     │     │     │     │     │  └─ NO → chore
```

### Detailed Type Guidelines

#### feat (Feature)

**When to use:**
- New functionality added
- New user-facing feature
- New API endpoint
- New component/module

**Examples:**
```
feat(auth): add social login support
feat(dashboard): add real-time notifications
feat(api): add pagination to list endpoints
```

**Indicators in diff:**
- New files created
- New functions/methods added
- New routes/endpoints
- New UI components

#### fix (Bug Fix)

**When to use:**
- Fixing a bug
- Correcting wrong behavior
- Patching security vulnerability
- Fixing regression

**Examples:**
```
fix(login): prevent password field autocomplete
fix(api): handle null values in response
fix(ui): correct button alignment on mobile
```

**Indicators in diff:**
- Error handling added
- Conditional logic modified
- Edge cases handled
- Validation added

#### docs (Documentation)

**When to use:**
- README updates
- Code comments added/updated
- API documentation changes
- Migration guides
- Only documentation changes (no code)

**Examples:**
```
docs(README): add installation instructions
docs(api): update authentication examples
docs: add contributing guidelines
```

**Indicators in diff:**
- Only `.md` files changed
- Only comments changed
- JSDoc/docstrings updated
- No functional code changes

#### style (Code Style)

**When to use:**
- Formatting changes
- Whitespace fixes
- Missing semicolons
- Code style improvements
- No functional changes

**Examples:**
```
style: format code with prettier
style(components): add consistent spacing
style: fix indentation in config files
```

**Indicators in diff:**
- Only whitespace changes
- Only formatting changes
- Linter fixes
- No logic changes

#### refactor (Code Refactoring)

**When to use:**
- Code restructuring
- Renaming variables/functions
- Extracting functions/components
- Improving code quality
- No behavior change

**Examples:**
```
refactor(auth): extract validation logic to helper
refactor(api): simplify error handling middleware
refactor: migrate class components to hooks
```

**Indicators in diff:**
- Functions moved/renamed
- Code extracted to new files
- Logic reorganized
- Same functionality, different structure

#### perf (Performance)

**When to use:**
- Performance optimizations
- Reducing memory usage
- Faster algorithms
- Database query optimization
- Bundle size reduction

**Examples:**
```
perf(images): implement lazy loading
perf(api): add database query caching
perf: optimize bundle size with code splitting
```

**Indicators in diff:**
- Caching added
- Memoization added
- Optimized algorithms
- Lazy loading implemented

#### test (Testing)

**When to use:**
- Adding tests
- Updating tests
- Fixing failing tests
- Improving test coverage

**Examples:**
```
test(auth): add unit tests for login flow
test: increase coverage to 80%
test(api): add integration tests for endpoints
```

**Indicators in diff:**
- Test files (`.test.js`, `.spec.ts`) modified
- New test cases added
- Test utilities updated
- Only test code changed

#### build (Build System)

**When to use:**
- Build configuration changes
- Dependency updates
- Build tool modifications
- Compilation changes

**Examples:**
```
build: upgrade webpack to v5
build: add TypeScript compilation
build(deps): update dependencies
```

**Indicators in diff:**
- `package.json` changes
- Build config files modified
- Webpack/Rollup/Vite config changes
- `tsconfig.json` updates

#### ci (Continuous Integration)

**When to use:**
- CI/CD configuration changes
- GitHub Actions updates
- Pipeline modifications
- Deployment scripts

**Examples:**
```
ci: add automated testing workflow
ci: update deployment pipeline
ci(github): add pull request template
```

**Indicators in diff:**
- `.github/workflows/` changes
- CI config files modified
- Deployment scripts updated

#### chore (Maintenance)

**When to use:**
- Other changes not fitting above types
- Tooling changes
- Configuration updates
- Dependency management
- Repository maintenance

**Examples:**
```
chore: update .gitignore
chore(deps): update devDependencies
chore: configure VSCode settings
```

**Indicators in diff:**
- Config files changed
- Dev dependencies updated
- Tooling modified
- No production code impact

## Scope Identification

### Determining Scope

Scope indicates what part of the codebase was affected.

### Common Scope Patterns

#### By Feature/Module
```
feat(auth): ...
feat(payment): ...
feat(profile): ...
```

#### By Component
```
feat(button): ...
feat(modal): ...
feat(navbar): ...
```

#### By Layer
```
refactor(api): ...
fix(database): ...
perf(cache): ...
```

#### By Technology
```
build(webpack): ...
ci(github): ...
style(eslint): ...
```

### Scope Analysis Steps

1. **Identify primary affected area**
   - Look at file paths
   - Identify main module/component
   - Find common directory

2. **Check for existing scopes**
   ```bash
   # View recent commit scopes
   git log --oneline -20 | grep -oP '\(\K[^\)]+' | sort | uniq
   ```

3. **Use consistent naming**
   - lowercase
   - kebab-case for multi-word scopes
   - Singular nouns (not plural)

4. **Omit scope if change is global**
   ```
   docs: update README
   style: format all files
   chore: update dependencies
   ```

### Scope Examples by File Pattern

```
src/components/Button.tsx          → scope: button
src/features/auth/Login.tsx        → scope: auth
src/api/users.ts                   → scope: api or users
src/utils/validation.ts            → scope: utils
src/hooks/useAuth.ts               → scope: hooks or auth
docs/api.md                        → scope: docs or api
.github/workflows/test.yml         → scope: ci or github
package.json                       → scope: deps or build
```

## Subject Generation

### Rules

1. **Use imperative mood**
   - ✅ "add feature"
   - ❌ "added feature"
   - ❌ "adds feature"

2. **Start with lowercase**
   - ✅ "add user authentication"
   - ❌ "Add user authentication"

3. **No period at end**
   - ✅ "fix memory leak"
   - ❌ "fix memory leak."

4. **Maximum 50 characters**
   - Keep it concise
   - Save details for body

5. **Be specific**
   - ✅ "fix password validation regex"
   - ❌ "fix bug"

### Subject Templates

```
add {what}
remove {what}
fix {issue}
update {what}
refactor {what}
improve {what}
implement {what}
extract {what} to {where}
migrate {what} to {what}
rename {what} to {what}
move {what} to {where}
replace {what} with {what}
prevent {issue}
ensure {behavior}
```

### Analysis for Subject

1. **Identify the main action**
   - What is the primary change?
   - What verb describes it best?

2. **Identify the object**
   - What was changed?
   - Be specific but concise

3. **Combine action + object**
   ```
   Action: "add"
   Object: "JWT token validation"
   Subject: "add JWT token validation"
   ```

4. **Check length**
   ```bash
   echo "add JWT token validation" | wc -c
   # Should be ≤ 50
   ```

## Body Composition

### When to Include Body

Include body when:
- Subject alone is not clear
- Need to explain rationale
- Multiple related changes
- Breaking changes
- Complex implementation

Omit body when:
- Change is obvious from subject
- Simple one-line change
- Self-explanatory diff

### Body Structure

```
<blank line after subject>

<paragraph 1: What changed>

<paragraph 2: Why it changed>

<paragraph 3: Implementation details (if needed)>
```

### Body Guidelines

1. **Wrap at 72 characters**
   ```
   Implement JWT token validation middleware for protected
   routes. Validates token signature, expiration, and user
   permissions using jsonwebtoken library.
   ```

2. **Explain what and why, not how**
   - ✅ "Need to prevent unauthorized access to admin routes"
   - ❌ "Added if statement to check token"

3. **Use present tense**
   - ✅ "This change prevents..."
   - ❌ "This change prevented..."

4. **Be thorough but concise**
   - Include relevant details
   - Omit obvious information

### Body Templates

#### For Features
```
Implement {feature} to {reason}.

Users can now {action}. This provides {benefit}.

Implementation uses {technology/approach}.
```

#### For Bug Fixes
```
Fix {issue} that occurred when {condition}.

The problem was caused by {root cause}. This fix
{solution approach}.

Tested by {testing approach}.
```

#### For Refactoring
```
Refactor {component} to improve {quality aspect}.

Previous implementation had {issues}. New structure
provides {benefits}.
```

### Example Body Analysis

**Code change:**
```diff
+ const validateToken = (token) => {
+   const decoded = jwt.verify(token, SECRET);
+   if (decoded.exp < Date.now()) {
+     throw new Error('Token expired');
+   }
+   return decoded;
+ };
```

**Body:**
```
Implement JWT token validation middleware for protected routes.

Previous implementation only checked token presence, not validity.
This caused security issues where expired or invalid tokens were
accepted.

New middleware validates token signature, expiration, and user
permissions using jsonwebtoken library. Invalid tokens now return
401 Unauthorized.
```

## Footer Construction

### Footer Components

1. **Issue References**
2. **Breaking Changes**
3. **Co-authors**
4. **Other metadata**

### Issue References

#### Format
```
Fixes #123
Closes #456, #789
Refs #111
See also #222
```

#### Keywords

**Closes issue:**
- `Fixes #123`
- `Closes #123`
- `Resolves #123`

**References issue:**
- `Refs #123`
- `Related to #123`
- `See also #123`

#### Multiple Issues
```
Fixes #123, #456
Refs #789, #101
```

### Breaking Changes

#### Format 1: BREAKING CHANGE Footer
```
BREAKING CHANGE: API response format changed to JSON:API spec.
All endpoints now return data in {data: ...} wrapper.
Previous format is no longer supported.
```

#### Format 2: ! in Type
```
feat(api)!: change response format
```

#### What Qualifies as Breaking

- API changes incompatible with previous version
- Removed functionality
- Changed behavior of existing features
- Required migration steps
- Dependency version requirements

#### Breaking Change Template
```
BREAKING CHANGE: {what changed}

{why it changed}

{how to migrate}
```

### Co-authors

```
Co-authored-by: John Doe <john@example.com>
Co-authored-by: Jane Smith <jane@example.com>
```

### Complete Footer Example

```
feat(api): add GraphQL endpoint

Implement GraphQL API alongside existing REST API.
Provides more flexible data fetching and reduces
over-fetching.

BREAKING CHANGE: REST API deprecated. Will be removed in v2.0.
Migrate to GraphQL endpoint at /graphql.

Migration guide: docs/graphql-migration.md

Fixes #123
Refs #456
Co-authored-by: John Doe <john@example.com>
```

## Special Cases

### Multiple Unrelated Changes

**Problem:** Staged changes include unrelated modifications

**Solution:** Suggest splitting into multiple commits

```bash
# Reset staged changes
git reset

# Stage only related changes
git add src/auth/*
git commit -m "feat(auth): add OAuth2 support"

git add src/api/*
git commit -m "refactor(api): simplify error handling"
```

### Mixed Feature and Fix

**Problem:** One commit contains both feature and bug fix

**Solution:** Prioritize the most significant change

If feature is primary:
```
feat(auth): add OAuth2 support

Also fixes password reset token expiration bug.

Fixes #123
```

If fix is primary:
```
fix(auth): correct password reset token expiration

Also adds OAuth2 support for convenience.

Fixes #123
```

Better: Split into two commits

### Reverts

**Format:**
```
revert: feat(auth): add OAuth2 support

This reverts commit abc123def456.

Reason: OAuth2 implementation causes login failures
in production. Reverting to investigate issue.

Refs #789
```

### Merge Commits

**Avoid custom messages for merges:**
```bash
# Use default merge message
git merge feature-branch

# Result:
# Merge branch 'feature-branch' into main
```

### WIP Commits

**Don't use WIP in main/production branches:**
```
# In feature branch (acceptable)
git commit -m "WIP: working on auth feature"

# Before merging, squash and create proper commit:
git rebase -i main
# Create proper conventional commit
```

## Analysis Checklist

Before generating commit message:

- [ ] Analyzed `git diff --staged`
- [ ] Read modified files for context
- [ ] Identified primary change type
- [ ] Determined appropriate scope
- [ ] Checked for breaking changes
- [ ] Found related issues/PRs
- [ ] Verified changes are related
- [ ] Considered splitting if needed
- [ ] Checked for sensitive information in diff
- [ ] Ensured tests pass (if applicable)

## Common Pitfalls

### Too Vague
❌ `fix: fix bug`
✅ `fix(auth): prevent token refresh race condition`

### Too Detailed
❌ `feat(button): add loading state by adding isLoading prop and showing spinner component when true and disabling button`
✅ `feat(button): add loading state`

### Wrong Type
❌ `chore: add user authentication` (should be `feat`)
❌ `feat: fix typo in README` (should be `docs`)

### Wrong Mood
❌ `feat: added login feature` (should be "add")
❌ `fix: fixing memory leak` (should be "fix")

### Capitalization
❌ `feat: Add login feature` (should be lowercase)
✅ `feat: add login feature`

### Period at End
❌ `feat: add login feature.`
✅ `feat: add login feature`

### Missing Scope When Needed
❌ `feat: add validation` (which component?)
✅ `feat(form): add email validation`

## Examples by Scenario

### New Feature with Tests
```
feat(auth): add two-factor authentication

Implement TOTP-based 2FA for enhanced security.
Users can enable 2FA in account settings.

- Add QR code generation for authenticator apps
- Add backup codes for account recovery
- Add tests for TOTP validation

Closes #234
```

### Performance Improvement
```
perf(images): implement lazy loading

Reduce initial page load time by deferring image loading
until they enter viewport. Uses Intersection Observer API.

Improves Lighthouse score from 65 to 89.

Refs #456
```

### Security Fix
```
fix(api): prevent SQL injection in search query

Sanitize user input before including in SQL queries.
Previous implementation concatenated strings directly.

Uses parameterized queries to prevent injection attacks.

Security advisory: docs/security/CVE-2024-001.md

Fixes #789
```

### Breaking API Change
```
feat(api)!: migrate to REST conventions

Change API endpoint structure to follow REST standards.

BREAKING CHANGE: Endpoint URLs have changed:
- /getUser → /users/:id
- /createUser → POST /users
- /updateUser → PUT /users/:id
- /deleteUser → DELETE /users/:id

Migration guide: docs/api-v2-migration.md

Closes #345
```

### Dependency Update
```
build(deps): upgrade React to v18

Update React from v17 to v18 for improved performance
and new features.

- Update all React-related dependencies
- Migrate to new root API
- Update tests for new rendering behavior

No breaking changes for end users.

Refs #567
```
