# Plan Review Prompt Templates

Comprehensive collection of prompt templates for reviewing Claude Code plan files using OpenAI Codex CLI.

## Table of Contents

1. [Master Review Prompt Template](#master-review-prompt-template)
2. [Single-Perspective Prompt Templates](#single-perspective-prompt-templates)
3. [Codex CLI Command Templates](#codex-cli-command-templates)
4. [Merged Plan Output Template](#merged-plan-output-template)
5. [Error Message Templates](#error-message-templates)

## Master Review Prompt Template

The primary prompt sent to Codex CLI for comprehensive plan review. Replace `{PLAN_CONTENT}` with the actual plan file content.

```markdown
# Plan Review Request

You are an expert software architect and code reviewer. Analyze the following implementation plan and provide a comprehensive review from multiple perspectives.

## Plan Content

{PLAN_CONTENT}

## Review Instructions

Analyze this plan from the following 7 perspectives. For each perspective, identify issues and provide actionable recommendations. Classify each finding by severity: Critical, High, Medium, Low, or Info.

### 1. Technical Feasibility
- Are all implementation steps complete and logically ordered?
- Are there missing steps or implicit assumptions?
- Are the proposed technologies and approaches appropriate?
- Are there edge cases not addressed?
- Is the dependency order correct?

### 2. Security Considerations
- Are there potential security vulnerabilities (OWASP Top 10)?
- Is authentication and authorization properly handled?
- Is user input validated and sanitized?
- Are there injection risks (SQL, XSS, command injection)?
- Is sensitive data properly protected?
- Are secrets/credentials handled securely?

### 3. Performance Optimization
- Are there potential N+1 query problems?
- Is caching strategy considered where appropriate?
- Are there opportunities for async/parallel processing?
- Are database indexes considered?
- Is pagination implemented for large datasets?
- Are there potential memory leaks or resource issues?

### 4. Test Strategy
- Is the test strategy comprehensive (unit, integration, E2E)?
- Is test coverage adequate for critical paths?
- Are edge cases and error scenarios tested?
- Is the mocking strategy appropriate?
- Are there missing test scenarios?

### 5. Alternative Approaches
- Are there simpler solutions that achieve the same goal?
- Are there better design patterns for this use case?
- Are there existing libraries that could reduce complexity?
- Could the architecture be simplified?
- Are there industry best practices not being followed?

### 6. Dependencies
- Are all dependencies identified and version-compatible?
- Are there known vulnerabilities in proposed dependencies?
- Are licenses compatible with the project?
- Are there lighter alternatives to heavy dependencies?
- Is the dependency count reasonable?

### 7. Error Handling
- Are all failure modes identified and handled?
- Is there a recovery strategy for each failure mode?
- Is logging sufficient for debugging?
- Are error messages user-friendly?
- Is monitoring and alerting considered?
- Are there timeout and retry strategies?

## Output Format

Structure your response EXACTLY as follows:

### Review Summary
- Critical Issues: [count]
- High Issues: [count]
- Medium Issues: [count]
- Recommendations: [count]

### Technical Feasibility
[Your findings here with severity tags like [Critical], [High], [Medium], [Low], [Info]]

### Security Considerations
[Your findings here]

### Performance Optimization
[Your findings here]

### Test Strategy
[Your findings here]

### Alternative Approaches
[Your findings here]

### Dependencies
[Your findings here]

### Error Handling
[Your findings here]

### Action Items
- [ ] [Priority: Critical/High/Medium] Description of action item
```

## Single-Perspective Prompt Templates

Use these when reviewing from a single perspective only.

### Security-Only Review

```markdown
# Security Review Request

You are a security expert. Analyze the following implementation plan for security vulnerabilities and risks.

## Plan Content

{PLAN_CONTENT}

## Security Review Checklist

Analyze this plan for:
1. **OWASP Top 10** vulnerabilities
2. **Authentication & Authorization** gaps
3. **Data Validation** issues (input sanitization, output encoding)
4. **Injection Risks** (SQL, XSS, command, LDAP, etc.)
5. **Sensitive Data Exposure** (PII, credentials, tokens)
6. **Security Misconfiguration** risks
7. **Cryptography** weaknesses
8. **Access Control** flaws

## Output Format

### Security Review Summary
- Critical: [count]
- High: [count]
- Medium: [count]

### Findings
[List each finding with severity, description, and recommendation]

### Action Items
- [ ] [Priority] Description
```

### Performance-Only Review

```markdown
# Performance Review Request

You are a performance engineering expert. Analyze the following implementation plan for performance bottlenecks and optimization opportunities.

## Plan Content

{PLAN_CONTENT}

## Performance Review Checklist

Analyze this plan for:
1. **Database** - N+1 queries, missing indexes, unoptimized queries
2. **Caching** - Missing cache layers, cache invalidation strategy
3. **Async Processing** - Blocking operations, parallelization opportunities
4. **Memory** - Potential leaks, excessive allocation, unbounded growth
5. **Network** - Excessive API calls, payload size, connection pooling
6. **Scalability** - Horizontal/vertical scaling considerations
7. **Resource Management** - Connection pools, file handles, threads

## Output Format

### Performance Review Summary
- Critical: [count]
- High: [count]
- Medium: [count]

### Findings
[List each finding with severity, description, and recommendation]

### Action Items
- [ ] [Priority] Description
```

### Test-Strategy-Only Review

```markdown
# Test Strategy Review Request

You are a QA engineering expert. Analyze the following implementation plan for test strategy completeness.

## Plan Content

{PLAN_CONTENT}

## Test Strategy Review Checklist

Analyze this plan for:
1. **Unit Tests** - Coverage of individual functions/methods
2. **Integration Tests** - Component interaction testing
3. **E2E Tests** - User journey coverage
4. **Edge Cases** - Boundary conditions, null/empty inputs
5. **Error Scenarios** - Failure mode testing
6. **Mocking Strategy** - External dependency isolation
7. **Test Data** - Fixture management, data factories
8. **CI Integration** - Automated test execution

## Output Format

### Test Strategy Review Summary
- Missing Test Areas: [count]
- Improvement Suggestions: [count]

### Findings
[List each finding with description and recommendation]

### Action Items
- [ ] [Priority] Description
```

### Feasibility-Only Review

```markdown
# Technical Feasibility Review Request

You are a senior software architect. Analyze the following implementation plan for technical feasibility and completeness.

## Plan Content

{PLAN_CONTENT}

## Feasibility Review Checklist

Analyze this plan for:
1. **Completeness** - Missing implementation steps
2. **Logical Order** - Correct dependency sequence
3. **Technology Choices** - Appropriate tools and frameworks
4. **Edge Cases** - Unaddressed scenarios
5. **Assumptions** - Implicit or unstated assumptions
6. **Complexity** - Unnecessary over-engineering
7. **Maintainability** - Long-term code health

## Output Format

### Feasibility Review Summary
- Blocking Issues: [count]
- Improvement Suggestions: [count]

### Findings
[List each finding with description and recommendation]

### Action Items
- [ ] [Priority] Description
```

## Codex CLI Command Templates

### Default Execution (Read-Only Sandbox)

```bash
codex exec -s read-only --full-auto - < /tmp/codex-plan-review-prompt-{timestamp}.md
```

### Specific Model

```bash
# Use o4-mini for faster, cheaper reviews
codex exec -s read-only --full-auto -m o4-mini - < /tmp/codex-plan-review-prompt-{timestamp}.md

# Use o3 for deeper analysis
codex exec -s read-only --full-auto -m o3 - < /tmp/codex-plan-review-prompt-{timestamp}.md
```

### With Timeout

```bash
timeout 300 codex exec -s read-only --full-auto - < /tmp/codex-plan-review-prompt-{timestamp}.md
```

### Capture Output

```bash
codex exec -s read-only --full-auto - < /tmp/codex-plan-review-prompt-{timestamp}.md 2>&1
```

## Merged Plan Output Template

The format appended to the original plan file after review:

```markdown
---

## Codex Review (Generated: YYYY-MM-DD HH:MM)

> Reviewed by OpenAI Codex CLI ({MODEL}) in read-only sandbox mode.

### Review Summary
- Critical Issues: N
- High Issues: N
- Medium Issues: N
- Recommendations: N

### Technical Feasibility
[findings from Codex output]

### Security Considerations
[findings from Codex output]

### Performance Optimization
[findings from Codex output]

### Test Strategy
[findings from Codex output]

### Alternative Approaches
[findings from Codex output]

### Dependencies
[findings from Codex output]

### Error Handling
[findings from Codex output]

### Action Items
- [ ] [Priority: Critical/High/Medium] Description of action item
```

## Error Message Templates

### Codex CLI Not Installed

```
⚠️ OpenAI Codex CLI가 설치되어 있지 않습니다.

설치 방법:
  npm install -g @openai/codex

설치 후 인증을 설정하세요:
  codex login
  또는 OPENAI_API_KEY 환경변수를 설정하세요.
```

### Authentication Failed

```
⚠️ Codex CLI 인증에 실패했습니다.

다음 방법 중 하나로 인증을 설정하세요:
1. codex login 명령어 실행
2. OPENAI_API_KEY 환경변수 설정:
   export OPENAI_API_KEY="your-api-key"
```

### Plan File Not Found

```
⚠️ 플랜 파일을 찾을 수 없습니다.

~/.claude/plans/ 디렉토리에 .md 파일이 없습니다.

다음 중 하나를 시도하세요:
1. Claude Code에서 plan mode로 플랜을 먼저 생성하세요.
2. 플랜 파일 경로를 직접 지정하세요.
```

### Empty Plan File

```
⚠️ 플랜 파일이 비어 있습니다.

파일: {FILE_PATH}

플랜 내용이 포함된 파일을 사용하세요.
```

### Codex Execution Timeout

```
⚠️ Codex CLI 실행이 시간 초과되었습니다 (5분).

다음을 시도하세요:
1. 더 가벼운 모델로 재시도: codex exec -m o4-mini ...
2. 단일 관점 리뷰로 범위를 줄이세요.
3. 네트워크 연결을 확인하세요.
```

### Empty Codex Output

```
⚠️ Codex CLI에서 출력이 없습니다.

다음을 시도하세요:
1. 다른 모델로 재시도하세요 (예: -m o4-mini).
2. 프롬프트를 단순화하세요.
3. Codex CLI 상태를 확인하세요: codex --version
```

### Unexpected Output Format

```
⚠️ Codex 출력 형식이 예상과 다릅니다.

예상된 섹션 헤더가 발견되지 않았습니다.
원본 출력을 그대로 플랜 파일에 추가합니다.

수동으로 리뷰 결과를 확인하고 정리하세요.
```

### File Write Failure

```
⚠️ 플랜 파일에 리뷰 결과를 쓸 수 없습니다.

파일: {FILE_PATH}

아래에 리뷰 결과를 출력합니다. 수동으로 복사하세요:

---
{REVIEW_CONTENT}
---
```
