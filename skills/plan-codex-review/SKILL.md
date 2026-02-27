---
name: plan-codex-review
description: Automatically send Claude Code plan files to OpenAI Codex CLI for comprehensive review after plan mode exits. This skill MUST be triggered automatically whenever plan mode completes (ExitPlanMode) and the user approves the plan. Also activates when users request: (1) Reviewing a plan with Codex, (2) Getting external AI review of implementation plans, (3) Enhancing plans with security/performance/feasibility analysis, (4) Multi-perspective plan validation, (5) Cross-checking plans with OpenAI Codex CLI.
---

# Plan Codex Review

Automatically send Claude Code plan files to OpenAI Codex CLI for comprehensive multi-perspective review after plan mode completes, and merge the feedback back into the plan for enhancement.

## Auto-Trigger Behavior

**IMPORTANT: This skill runs automatically after plan mode exits.**

When plan mode ends (via `ExitPlanMode`) and the user approves the plan:

1. **Immediately proceed** with the Codex review workflow below — do NOT wait for user to request it
2. Inform the user: "플랜이 승인되었습니다. Codex CLI로 자동 리뷰를 시작합니다..."
3. If Codex CLI is not installed or auth fails, inform the user and **skip gracefully** — do not block plan implementation
4. After the review completes, ask the user if they want to proceed with implementation or address review findings first

### Skip Conditions

Do NOT auto-trigger if:
- The user explicitly says to skip the review (e.g., "리뷰 없이 진행해줘", "Codex 리뷰 건너뛰기")
- The plan is a trivial change (single file, < 10 lines of changes described)

## Quick Start

This workflow runs automatically after plan mode exits, or when manually requested:

1. **Detect plan file**:
   ```bash
   # Auto-detect most recent plan (written by ExitPlanMode)
   ls -t ~/.claude/plans/*.md 2>/dev/null | head -5
   ```

2. **Validate prerequisites**:
   ```bash
   codex --version
   ```

3. **Read plan file** and compose review prompt using [references/templates.md](references/templates.md) Master Review Prompt Template

4. **Write prompt to temp file**:
   ```bash
   # Write composed prompt (plan content embedded in template)
   # to /tmp/codex-plan-review-prompt-{timestamp}.md
   ```

5. **Execute Codex CLI**:
   ```bash
   codex exec -s read-only --full-auto - < /tmp/codex-plan-review-prompt-{timestamp}.md
   ```

6. **Parse output and merge** into the original plan file as `## Codex Review` section

7. **Ask user**: "리뷰 결과를 반영하여 플랜을 수정하시겠습니까, 아니면 현재 플랜대로 구현을 진행하시겠습니까?"

## Prerequisites

### Codex CLI Installation

```bash
npm install -g @openai/codex
```

### Authentication

One of the following:
```bash
# Option 1: Interactive login
codex login

# Option 2: Environment variable
export OPENAI_API_KEY="your-api-key"
```

### Prerequisite Validation

Before executing the review, verify:

```bash
# 1. Check Codex CLI is installed
codex --version
# If fails → show installation instructions and stop

# 2. Check authentication (attempt a minimal exec)
codex exec -s read-only --full-auto -m gpt-5.3-codex - <<< "echo hello"
# If auth error → show authentication instructions and stop
```

## Plan File Detection

### Auto-Detection

Search for plan files in the following locations (in order):

1. `~/.claude/plans/*.md` - Default Claude Code plans directory
2. `.claude/plans/*.md` - Project-local plans directory
3. User-specified path

### Selection Logic

- **Single file found**: Use it automatically
- **Multiple files found**: Use `AskUserQuestion` to let the user choose
- **No files found**: Ask user to provide a path or create a plan first

### Detection Commands

```bash
# List recent plan files with timestamps
ls -lt ~/.claude/plans/*.md 2>/dev/null | head -10

# Show file preview (first 5 lines)
head -5 ~/.claude/plans/{selected_file}
```

## Review Perspectives

The review analyzes the plan from 7 perspectives:

### 1. Technical Feasibility
- Implementation step completeness and ordering
- Technology choice appropriateness
- Edge case coverage
- Dependency sequence correctness
- Implicit assumptions

### 2. Security Considerations
- OWASP Top 10 vulnerability check
- Authentication and authorization review
- Input validation and sanitization
- Injection risk assessment (SQL, XSS, command)
- Secrets and credential handling

### 3. Performance Optimization
- N+1 query detection
- Caching strategy evaluation
- Async/parallel processing opportunities
- Database index recommendations
- Pagination for large datasets
- Resource management

### 4. Test Strategy
- Unit/integration/E2E test coverage
- Critical path test completeness
- Edge case and error scenario coverage
- Mocking strategy assessment
- Coverage target evaluation

### 5. Alternative Approaches
- Simpler solution identification
- Design pattern recommendations
- Library/framework alternatives
- Architecture simplification opportunities
- Industry best practice alignment

### 6. Dependencies
- Version compatibility check
- Known vulnerability assessment
- License compatibility verification
- Bundle size impact evaluation
- Maintenance status review

### 7. Error Handling
- Failure mode identification
- Recovery strategy evaluation
- Logging and monitoring adequacy
- Timeout and retry strategy
- Graceful degradation assessment

For detailed checklists and examples, see [references/analysis-guide.md](references/analysis-guide.md).

## Codex CLI Invocation Strategy

### Prompt Composition

1. Read the Master Review Prompt Template from [references/templates.md](references/templates.md)
2. Read the plan file content
3. Replace `{PLAN_CONTENT}` placeholder with actual plan content
4. Write the composed prompt to a temporary file

```bash
# Generate timestamp for unique filename
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
PROMPT_FILE="/tmp/codex-plan-review-prompt-${TIMESTAMP}.md"

# Write composed prompt to temp file (using Write tool, not echo)
# The prompt file contains the Master Review Prompt Template
# with {PLAN_CONTENT} replaced by the actual plan content
```

### Execution

```bash
# Always use the best latest model (gpt-5.3-codex)
codex exec -s read-only --full-auto -m gpt-5.3-codex - < /tmp/codex-plan-review-prompt-{timestamp}.md
```

**Key flags:**
- `-s read-only` - Sandbox mode prevents Codex from modifying any files
- `--full-auto` - No interactive prompts, runs to completion
- `- < {file}` - Reads prompt from stdin via file redirect

### Timeout

Use a 5-minute timeout (300 seconds). If exceeded, inform the user and suggest retrying with a lighter model.

```bash
timeout 300 codex exec -s read-only --full-auto -m gpt-5.3-codex - < /tmp/codex-plan-review-prompt-{timestamp}.md
```

## Output Parsing and Merging

### Parsing Strategy

1. Capture full stdout from Codex execution
2. Look for expected section headers:
   - `### Review Summary`
   - `### Technical Feasibility`
   - `### Security Considerations`
   - `### Performance Optimization`
   - `### Test Strategy`
   - `### Alternative Approaches`
   - `### Dependencies`
   - `### Error Handling`
   - `### Action Items`
3. If section headers are found → structured merge
4. If section headers are NOT found → raw append with format warning

### Merge Process

1. Read the original plan file
2. Append horizontal rule (`---`)
3. Append `## Codex Review (Generated: YYYY-MM-DD HH:MM)` header
4. Append attribution blockquote
5. Append parsed review sections
6. Write back to the original plan file using the Edit or Write tool

### Merged Output Format

```markdown
[Original plan content - unchanged]

---

## Codex Review (Generated: YYYY-MM-DD HH:MM)

> Reviewed by OpenAI Codex CLI ({MODEL}) in read-only sandbox mode.

### Review Summary
- Critical Issues: N
- High Issues: N
- Medium Issues: N
- Recommendations: N

### Technical Feasibility
[findings]

### Security Considerations
[findings]

### Performance Optimization
[findings]

### Test Strategy
[findings]

### Alternative Approaches
[findings]

### Dependencies
[findings]

### Error Handling
[findings]

### Action Items
- [ ] [Priority: Critical/High/Medium] Description of action item
```

## Error Handling

### Error Scenarios and Responses

| # | Error | Detection | Response |
|---|-------|-----------|----------|
| 1 | Codex CLI not installed | `codex --version` returns error | Show installation command, stop |
| 2 | Authentication not configured | Codex exec returns auth error | Show authentication options, stop |
| 3 | No plan file found | No `.md` files in plans directories | Ask user for path or suggest creating a plan |
| 4 | Plan file is empty | File content is blank/whitespace | Warn and stop |
| 5 | Execution timeout | Command exceeds 5 minutes | Suggest narrower scope or retry |
| 6 | Empty Codex output | stdout is blank | Retry with gpt-5.3-codex |
| 7 | Unexpected output format | Section headers not found | Append raw output with format warning |
| 8 | File write failure | Write/Edit tool error | Print review to console as fallback |
| 9 | Multiple plan files | More than one `.md` found | Use `AskUserQuestion` for user selection |
| 10 | Network error | Connection refused/timeout | Check internet, retry |

### Error Message Templates

See [references/templates.md](references/templates.md) for pre-formatted error messages in Korean.

## Workflow Summary

Complete step-by-step workflow (runs automatically after plan mode exits):

```
0. [AUTO-TRIGGER] Plan mode exits → user approves plan
   ├─ Inform user: "Codex CLI로 자동 리뷰를 시작합니다..."
   └─ Check skip conditions (user opt-out, trivial plan)

1. Detect plan file
   ├─ Auto-detect from ~/.claude/plans/ (most recent)
   ├─ If multiple → use the one just created by plan mode
   └─ If none → ask for path

2. Validate prerequisites
   ├─ codex --version
   ├─ Auth check
   └─ If fails → warn and skip gracefully (do not block implementation)

3. Read plan file content

4. Compose prompt
   ├─ Load Master Review Prompt Template
   ├─ Replace {PLAN_CONTENT} with plan content
   └─ Write to /tmp/codex-plan-review-prompt-{timestamp}.md

5. Execute Codex CLI
   └─ codex exec -s read-only --full-auto -m gpt-5.3-codex - < {prompt_file}

6. Parse Codex output
   ├─ Look for section headers
   └─ Extract structured findings

7. Merge into plan file
   ├─ Append --- separator
   ├─ Append ## Codex Review header
   └─ Append parsed sections

8. Present summary to user
   ├─ Issue counts by severity
   ├─ Key findings highlights
   └─ Action items overview

9. Ask user for next step
   ├─ Option A: 리뷰 반영 후 플랜 수정
   ├─ Option B: 현재 플랜대로 구현 진행
   └─ Option C: 특정 관점만 재리뷰

10. Clean up
    └─ Remove temp prompt file
```

## Model Selection

**항상 최고 성능의 최신 모델을 사용합니다.**

| Model | Speed | Depth | Cost | Use Case |
|-------|-------|-------|------|----------|
| **gpt-5.3-codex** | Medium | Very Deep | High | **기본값 - 모든 리뷰에 사용** |
| gpt-5.3-codex-spark | Very Fast | Deep | Medium | 실시간 빠른 리뷰 (사용자 요청 시) |

모든 자동/수동 리뷰에서 `-m gpt-5.3-codex`를 명시적으로 지정합니다. 사용자가 다른 모델을 명시적으로 요청하는 경우에만 변경합니다.

## Best Practices

1. **Automatic review** - Let the auto-trigger handle reviews after every plan mode exit
2. **Iterate** - Re-run after addressing critical findings
3. **Always use the best model** - 항상 `gpt-5.3-codex`를 사용하여 최고 품질의 리뷰를 보장
4. **Preserve context** - Keep the review appended to the plan for reference
5. **Prioritize findings** - Address Critical and High issues first
6. **Single perspective mode** - For focused reviews, use single-perspective templates from [references/templates.md](references/templates.md)
7. **Verify Codex output** - AI reviews may have false positives; validate findings

## Analysis Guide

For detailed analysis methodology, see [references/analysis-guide.md](references/analysis-guide.md).

Key aspects analyzed:
- **Plan structure** - Completeness and ordering of implementation steps
- **Security posture** - Vulnerability and risk assessment
- **Performance profile** - Bottleneck and optimization analysis
- **Test coverage** - Strategy completeness and gap analysis

## Templates

For comprehensive prompt templates and output formats, see [references/templates.md](references/templates.md).
