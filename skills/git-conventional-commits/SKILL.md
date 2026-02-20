---
name: git-conventional-commits
description: Generate Conventional Commits formatted commit messages by analyzing git diff and codebase context. Automatically creates semantic, standardized commit messages following the Conventional Commits specification. Use when users request: (1) Creating commit messages, (2) Analyzing staged changes for commits, (3) Generating CHANGELOG files, (4) Setting up commit message templates, (5) Enforcing commit conventions, or (6) Any task involving git commit message standardization.
---

# Git Conventional Commits Generator

Automatically generate Conventional Commits formatted commit messages by analyzing staged changes, understanding code context, and following semantic versioning principles.

## Quick Start

When user requests commit message generation:

1. **Analyze staged changes**:
   ```bash
   git diff --staged
   ```

2. **Understand the scope and type**:
   - Read affected files to understand context
   - Identify the type of change (feat, fix, refactor, etc.)
   - Determine the scope (component, module, file)

3. **Generate commit message** following format:
   ```
   <type>(<scope>): <subject>

   <body>

   <footer>
   ```

4. **Ask user for confirmation** before committing

## Commit Types

Following the Conventional Commits specification:

- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation only changes
- **style**: Code style changes (formatting, missing semicolons, etc.)
- **refactor**: Code refactoring without changing functionality
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **build**: Changes to build system or dependencies
- **ci**: CI/CD configuration changes
- **chore**: Other changes that don't modify src or test files
- **revert**: Reverts a previous commit

## Message Structure

### Type and Scope
```
feat(user-auth): add JWT token validation
```

- **Type**: Must be one of the types above
- **Scope**: Optional, indicates the affected component/module
- **Subject**: Brief description in imperative mood

### Subject Rules
- Use imperative mood ("add" not "added" or "adds")
- No capitalization of first letter
- No period at the end
- Maximum 50 characters
- Be specific and descriptive

### Body (Optional)
```
feat(user-auth): add JWT token validation

Implement JWT token validation middleware for protected routes.
Validates token signature, expiration, and user permissions.
Uses jsonwebtoken library for token verification.
```

- Separated from subject by blank line
- Wrap at 72 characters
- Explain **what** and **why**, not **how**
- Can have multiple paragraphs

### Footer (Optional)
```
feat(user-auth): add JWT token validation

Implement JWT token validation middleware for protected routes.

BREAKING CHANGE: API now requires Authorization header with JWT token
Fixes #123
Refs #456, #789
```

- Reference issues and PRs
- Indicate breaking changes
- Co-authored-by for multiple contributors

## Breaking Changes

For breaking changes, use one of these formats:

### In footer:
```
feat(api)!: change response format to JSON:API spec

BREAKING CHANGE: All API responses now follow JSON:API specification.
Previous response format is no longer supported.
```

### With ! in type:
```
feat(api)!: change response format to JSON:API spec
```

## Workflow

### 1. Analyze Staged Changes

```bash
# Check what's staged
git diff --staged --name-only

# View detailed changes
git diff --staged
```

### 2. Categorize Changes

Group changes by type:
- All new features → `feat`
- All bug fixes → `fix`
- Mixed changes → Split into multiple commits

### 3. Determine Scope

Common scope patterns:
- By component: `feat(button): add loading state`
- By module: `fix(auth): prevent token refresh loop`
- By layer: `refactor(api): simplify error handling`
- No scope for global changes: `docs: update README`

### 4. Write Subject

Follow the imperative mood:
- ✅ `add user authentication`
- ❌ `added user authentication`
- ❌ `adds user authentication`

Be specific:
- ✅ `fix memory leak in event listeners`
- ❌ `fix bug`

### 5. Add Body (if needed)

Include body when:
- Change is not obvious from subject
- Need to explain rationale
- Multiple related changes
- Breaking changes

### 6. Add Footer (if applicable)

Include footer for:
- Issue references: `Fixes #123`
- Breaking changes: `BREAKING CHANGE: ...`
- Co-authors: `Co-authored-by: Name <email>`

## Examples

### Simple Feature
```
feat(user-profile): add avatar upload functionality
```

### Bug Fix with Issue Reference
```
fix(api): prevent race condition in token refresh

Add mutex lock to ensure only one token refresh request
is processed at a time, preventing duplicate refresh attempts.

Fixes #456
```

### Breaking Change
```
feat(api)!: migrate to GraphQL

Replace REST API with GraphQL endpoint for better query flexibility.

BREAKING CHANGE: All REST endpoints under /api/v1 are removed.
Use GraphQL endpoint at /graphql instead.

Migration guide: docs/migration-guide.md
```

### Refactoring
```
refactor(database): extract query builders to separate module

Move complex query building logic from models to dedicated
QueryBuilder classes for better maintainability and testability.
```

### Documentation
```
docs(README): add setup instructions for development environment
```

### Multiple Scopes
```
feat(auth,api): add OAuth2 integration

- Add OAuth2 authentication flow
- Update API middleware to support OAuth tokens
- Add tests for OAuth authentication

Refs #789
```

## Commit Message Template

For repository setup, create `.gitmessage`:

```
# <type>(<scope>): <subject>

# <body>

# <footer>

# Type: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
# Scope: Component/module affected (optional)
# Subject: Imperative mood, no cap, no period, max 50 chars
# Body: Explain what and why (optional, wrap at 72 chars)
# Footer: Issues, breaking changes, co-authors (optional)
```

Set as default:
```bash
git config commit.template .gitmessage
```

## CHANGELOG Generation

Generate CHANGELOG from conventional commits:

### Structure
```markdown
# Changelog

## [1.2.0] - 2024-01-15

### Features
- **auth**: Add JWT token validation (#123)
- **api**: Add rate limiting middleware (#124)

### Bug Fixes
- **database**: Fix connection pool leak (#125)
- **ui**: Prevent double form submission (#126)

### BREAKING CHANGES
- **api**: Change response format to JSON:API spec

## [1.1.0] - 2023-12-01
...
```

### Workflow
1. Parse commit history for conventional commits
2. Group by type (Features, Bug Fixes, etc.)
3. Note breaking changes
4. Link to issues/PRs

## Analysis Guide

For detailed analysis methodology, see [references/analysis-guide.md](references/analysis-guide.md).

Key aspects to analyze:
- **File changes**: What files were modified
- **Code context**: What the code does
- **Impact**: User-facing vs internal changes
- **Related changes**: Dependencies between changes

## Templates

For comprehensive commit message templates and examples, see [references/templates.md](references/templates.md).

## Best Practices

1. **One logical change per commit**: Don't mix features and fixes
2. **Commit often**: Small, focused commits are better
3. **Write for others**: Commit messages are documentation
4. **Be consistent**: Follow the convention in your project
5. **Use issue tracking**: Reference issues in footer
6. **Test before commit**: Ensure tests pass
7. **Review your diff**: Understand what you're committing

## Integration with Git Hooks

### Commit Message Validation

Create `.husky/commit-msg`:

```bash
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no -- commitlint --edit $1
```

### commitlint.config.js

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'build',
        'ci',
        'chore',
        'revert'
      ]
    ],
    'subject-case': [2, 'always', 'lower-case'],
    'subject-max-length': [2, 'always', 50]
  }
};
```

## Usage Tips

- **Ask user to stage specific changes** if too many unrelated changes
- **Suggest splitting into multiple commits** for mixed types
- **Provide commit message options** and let user choose
- **Explain the reasoning** behind type/scope selection
- **Include examples** from the codebase when suggesting messages
