---
name: cicd-pipeline-reviewer
description: CI/CD 파이프라인 설정 파일을 분석하여 보안 취약점, 성능 병목, 안정성 문제를 탐지하고 구체적인 개선안을 제시하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) CI/CD 분석, (2) 파이프라인 리뷰, (3) GitHub Actions 검사, (4) 빌드 최적화, (5) 워크플로우 분석, (6) pipeline review, (7) CI optimization, (8) 배포 파이프라인 점검, (9) GitLab CI 분석, (10) Jenkins 파이프라인 리뷰, (11) 워크플로우 보안 검사.
---

# CI/CD Pipeline Reviewer

CI/CD 파이프라인 설정 파일을 자동으로 탐지하고 분석하여 보안 취약점, 성능 최적화 기회, 안정성 개선점을 식별합니다. GitHub Actions, GitLab CI, Jenkins, CircleCI 등 주요 CI/CD 플랫폼을 지원하며, 각 발견사항에 대해 Before/After YAML 예시와 함께 구체적인 수정 가이드를 제공합니다.

## Quick Start

사용자가 CI/CD 파이프라인 분석을 요청하면 다음 워크플로우를 실행합니다:

1. **CI/CD 플랫폼 자동 감지**:
   ```bash
   # GitHub Actions
   ls .github/workflows/*.yml .github/workflows/*.yaml 2>/dev/null

   # GitLab CI
   ls .gitlab-ci.yml 2>/dev/null

   # Jenkins
   ls Jenkinsfile 2>/dev/null

   # CircleCI
   ls .circleci/config.yml 2>/dev/null

   # 기타 CI 설정
   ls .travis.yml azure-pipelines.yml bitbucket-pipelines.yml 2>/dev/null
   ```

2. **설정 파일 파싱**: 감지된 CI/CD 설정 파일의 전체 내용을 읽고 구조를 분석합니다

3. **보안 분석**: [보안 분석 규칙](#보안-분석)에 따라 취약점을 탐지합니다

4. **성능 분석**: [성능 최적화 규칙](#성능-최적화)에 따라 병목과 개선 기회를 식별합니다

5. **안정성 분석**: [안정성 분석 규칙](#안정성-분석)에 따라 장애 가능성을 점검합니다

6. **모범 사례 점검**: [모범 사례](#모범-사례)에 따라 개선 권장사항을 제시합니다

7. **보고서 생성**: [references/templates.md](references/templates.md)의 형식으로 Before/After 예시를 포함한 보고서를 출력합니다

## 트리거 조건

이 스킬은 다음 상황에서 활성화됩니다:

- "CI/CD 분석해줘" / "CI/CD 리뷰해줘"
- "파이프라인 리뷰해줘" / "파이프라인 검사해줘"
- "GitHub Actions 검사해줘" / "워크플로우 분석해줘"
- "빌드 최적화해줘" / "빌드 속도 개선해줘"
- "GitLab CI 분석" / "Jenkins 파이프라인 리뷰"
- "pipeline review" / "CI optimization"
- "배포 파이프라인 점검해줘"
- "워크플로우 보안 검사해줘"

## 플랫폼 자동 감지

### 감지 규칙

| 플랫폼 | 설정 파일 경로 | 식별 기준 |
|--------|---------------|----------|
| GitHub Actions | `.github/workflows/*.yml`, `.github/workflows/*.yaml` | 디렉토리 + `on:` 트리거 |
| GitLab CI | `.gitlab-ci.yml` | 파일명 + `stages:` 키워드 |
| Jenkins | `Jenkinsfile`, `Jenkinsfile.*` | 파일명 + `pipeline {}` 블록 |
| CircleCI | `.circleci/config.yml` | 디렉토리 + `version:` 키워드 |
| Travis CI | `.travis.yml` | 파일명 + `language:` 키워드 |
| Azure Pipelines | `azure-pipelines.yml` | 파일명 + `trigger:` 키워드 |
| Bitbucket Pipelines | `bitbucket-pipelines.yml` | 파일명 + `pipelines:` 키워드 |

### 복수 플랫폼 지원

프로젝트에서 여러 CI/CD 플랫폼을 동시에 사용하는 경우, 모든 설정 파일을 분석하고 플랫폼별로 구분하여 보고합니다.

## 보안 분석

CI/CD 파이프라인에서 발생할 수 있는 보안 취약점을 탐지합니다. 상세 방법론은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

### SEC-01: 시크릿 하드코딩

파이프라인 설정 파일에 자격증명, API 키, 토큰 등이 직접 작성된 경우를 탐지합니다.

**탐지 패턴**:
```regex
# 환경 변수에 값 직접 할당
(env|environment)\s*:[\s\S]*?\w+\s*:\s*['"]?(AKIA|sk_live|ghp_|xox[baprs]-|SG\.|AIza|eyJ)[^\s'"]+

# run 스텝 내 하드코딩된 토큰
(curl|wget|http).*(-H|--header)\s*['"]?(Authorization|Bearer|Token)\s*[:=]?\s*[A-Za-z0-9_\-\.]{20,}

# 비밀번호 직접 노출
(password|passwd|pwd|secret|token|key)\s*[:=]\s*['"][^'"$\{]{4,}['"]
```

**심각도**: Critical

**수정 가이드 (GitHub Actions)**:

Before:
```yaml
env:
  AWS_ACCESS_KEY_ID: AKIAIOSFODNN7EXAMPLE
  AWS_SECRET_ACCESS_KEY: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

After:
```yaml
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### SEC-02: GITHUB_TOKEN 과도한 권한

`permissions` 블록이 없거나 `write-all`로 설정된 경우 최소 권한 원칙 위반을 탐지합니다.

**탐지 패턴**:
```regex
# permissions 블록 자체가 없음 (워크플로우 레벨)
^(?!.*permissions\s*:).*on\s*:

# 과도한 권한 설정
permissions\s*:\s*(write-all|read-all)

# contents: write가 불필요한 경우
permissions\s*:[\s\S]*?contents\s*:\s*write
```

**심각도**: High

Before:
```yaml
name: CI
on: [push]
# permissions 미설정 - 기본적으로 모든 권한 부여
jobs:
  build:
    runs-on: ubuntu-latest
```

After:
```yaml
name: CI
on: [push]
permissions:
  contents: read
  checks: write
jobs:
  build:
    runs-on: ubuntu-latest
```

### SEC-03: 서드파티 액션 버전 미고정

서드파티 GitHub Actions를 태그(`@v1`)로만 참조하여 공급망 공격에 취약한 경우를 탐지합니다.

**탐지 패턴**:
```regex
# 태그만 사용 (SHA 미사용)
uses\s*:\s*[^/]+/[^@]+@v\d+(?!\.)
uses\s*:\s*[^/]+/[^@]+@[a-z]+(?![a-f0-9]{40})

# actions/ 공식 액션 제외한 서드파티
uses\s*:\s*(?!actions/)[^/]+/[^@]+@(?!main$|master$)[^\s]+(?<![a-f0-9]{40})
```

**심각도**: High

Before:
```yaml
steps:
  - uses: actions/checkout@v4
  - uses: some-org/custom-action@v1
  - uses: another-org/deploy-action@main
```

After:
```yaml
steps:
  - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
  - uses: some-org/custom-action@a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0 # v1.2.0
  - uses: another-org/deploy-action@f1e2d3c4b5a6f7e8d9c0b1a2f3e4d5c6b7a8f9e0 # main 2024-01-15
```

### SEC-04: 신뢰할 수 없는 입력 사용

PR 제목, 브랜치명, 커밋 메시지 등 외부 입력이 `run` 스텝에서 직접 사용되는 인젝션 취약점을 탐지합니다.

**탐지 패턴**:
```regex
# GitHub 컨텍스트에서 신뢰할 수 없는 입력을 run에서 직접 사용
run\s*:\s*.*\$\{\{\s*(github\.event\.(pull_request\.title|pull_request\.body|issue\.title|issue\.body|comment\.body|review\.body|head_commit\.message|commits\[\*\]\.message)|github\.head_ref)

# curl이나 명령어에 직접 삽입
run\s*:.*\$\{\{\s*github\.event\..*\}\}
```

**심각도**: Critical

Before:
```yaml
- name: Print PR title
  run: echo "${{ github.event.pull_request.title }}"
```

After:
```yaml
- name: Print PR title
  env:
    PR_TITLE: ${{ github.event.pull_request.title }}
  run: echo "$PR_TITLE"
```

### SEC-05: pull_request_target 오용

`pull_request_target` 트리거에서 PR의 코드를 체크아웃하고 실행하면 시크릿이 포크에 노출될 수 있습니다.

**탐지 패턴**:
```regex
# pull_request_target + PR HEAD 체크아웃 조합
on\s*:[\s\S]*?pull_request_target[\s\S]*?ref\s*:\s*\$\{\{\s*github\.event\.pull_request\.head\.(sha|ref)

# pull_request_target에서 시크릿 사용
on\s*:[\s\S]*?pull_request_target[\s\S]*?\$\{\{\s*secrets\.
```

**심각도**: Critical

### SEC-06: self-hosted 러너 보안

`self-hosted` 러너를 공개 저장소에서 사용하면 악의적인 코드가 내부 인프라에서 실행될 수 있습니다.

**탐지 패턴**:
```regex
runs-on\s*:\s*self-hosted
runs-on\s*:\s*\[.*self-hosted.*\]
```

**심각도**: Medium (공개 저장소인 경우 High)

## 성능 최적화

파이프라인 실행 속도를 개선할 수 있는 항목을 분석합니다.

### PERF-01: 캐싱 전략 미적용

의존성 캐싱이 설정되지 않아 매 실행마다 패키지를 다운로드하는 경우를 탐지합니다.

**탐지 규칙**:
- `npm install` 또는 `yarn install`이 있지만 `actions/cache` 또는 `setup-node`의 `cache` 옵션이 없는 경우
- `pip install`이 있지만 `actions/cache` 또는 `setup-python`의 `cache` 옵션이 없는 경우
- `gradle` 명령이 있지만 `gradle/gradle-build-action`의 캐싱이 없는 경우

**심각도**: Medium

Before:
```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
  - run: npm ci
```

After:
```yaml
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'
  - run: npm ci
```

### PERF-02: 병렬화 미적용

순차 실행되는 독립적인 작업들이 병렬화되지 않은 경우를 탐지합니다.

**탐지 규칙**:
- `needs` 의존성이 없는 job들이 순차적으로 정의되어 있지만 실제로는 독립적인 경우
- 테스트, 린트, 빌드가 하나의 job에서 순차 실행되는 경우
- matrix strategy를 활용할 수 있지만 사용하지 않는 경우

**심각도**: Low

Before:
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint
  test:
    needs: lint  # 실제로는 lint와 독립적
    runs-on: ubuntu-latest
    steps:
      - run: npm test
```

After:
```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - run: npm run lint
  test:
    runs-on: ubuntu-latest  # needs 제거 - lint와 병렬 실행
    steps:
      - run: npm test
```

### PERF-03: Matrix Strategy 미활용

여러 버전/환경에서 테스트해야 하는 경우 matrix를 사용하면 병렬 실행이 가능합니다.

**심각도**: Info

Before:
```yaml
jobs:
  test-node-18:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm test
  test-node-20:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm test
```

After:
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
      fail-fast: false
    steps:
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

### PERF-04: 조건부 실행 미적용

모든 push/PR에서 불필요한 작업이 실행되는 경우를 탐지합니다.

**탐지 규칙**:
- `paths` 또는 `paths-ignore` 필터가 없는 워크플로우
- 문서 변경에도 전체 테스트가 실행되는 경우
- `[skip ci]` 패턴 미지원

**심각도**: Low

Before:
```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

After:
```yaml
on:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - '.gitignore'
      - 'LICENSE'
  pull_request:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - '.gitignore'
      - 'LICENSE'
```

### PERF-05: Docker 레이어 캐싱 미적용

Docker 이미지 빌드 시 레이어 캐싱을 활용하지 않는 경우를 탐지합니다.

**심각도**: Medium

Before:
```yaml
- name: Build Docker image
  run: docker build -t myapp .
```

After:
```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Build Docker image
  uses: docker/build-push-action@v5
  with:
    context: .
    push: false
    tags: myapp:latest
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

### PERF-06: 아티팩트 관리 미비

빌드 아티팩트를 job 간에 공유해야 하지만 매번 빌드하는 경우를 탐지합니다.

**심각도**: Low

Before:
```yaml
jobs:
  test:
    steps:
      - run: npm ci && npm run build
      - run: npm test
  deploy:
    needs: test
    steps:
      - run: npm ci && npm run build  # 중복 빌드
      - run: npm run deploy
```

After:
```yaml
jobs:
  test:
    steps:
      - run: npm ci && npm run build
      - run: npm test
      - uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
  deploy:
    needs: test
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
      - run: npm run deploy
```

## 안정성 분석

파이프라인의 안정적인 실행을 보장하기 위한 항목을 점검합니다.

### REL-01: Timeout 미설정

job 또는 step에 timeout이 설정되지 않아 무한 대기가 발생할 수 있는 경우를 탐지합니다.

**탐지 패턴**:
```regex
# job 레벨 timeout 미설정
jobs\s*:[\s\S]*?runs-on\s*:(?![\s\S]*?timeout-minutes)

# step 레벨에서 긴 작업에 timeout 미설정
run\s*:\s*(npm\s+test|pytest|mvn|gradle|docker\s+build)(?![\s\S]*?timeout-minutes)
```

**심각도**: Medium

Before:
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: npm test
```

After:
```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - run: npm test
        timeout-minutes: 15
```

### REL-02: Concurrency 미설정

동일 브랜치에서 여러 워크플로우가 동시에 실행되어 리소스 낭비 또는 충돌이 발생할 수 있는 경우를 탐지합니다.

**탐지 패턴**:
```regex
# concurrency 블록 미설정
^(?!.*concurrency\s*:).*on\s*:[\s\S]*?(push|pull_request)
```

**심각도**: Medium

Before:
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

After:
```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}
```

### REL-03: Retry 미설정

네트워크 요청이나 외부 서비스 호출이 있지만 retry 로직이 없는 경우를 탐지합니다.

**탐지 패턴**:
```regex
# 네트워크 의존 명령에 retry 없음
run\s*:\s*(curl|wget|npm\s+(install|ci)|pip\s+install|docker\s+pull|apt-get|apk\s+add)(?![\s\S]*?(retry|retries|attempts))
```

**심각도**: Low

Before:
```yaml
- name: Install dependencies
  run: npm ci
```

After:
```yaml
- name: Install dependencies
  run: npm ci
  env:
    npm_config_retry: 3
# 또는 별도 retry 로직
- name: Install dependencies
  uses: nick-fields/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: npm ci
```

### REL-04: 환경 보호 규칙 미설정

프로덕션 배포에 승인 게이트나 환경 보호 규칙이 없는 경우를 탐지합니다.

**탐지 규칙**:
- `deploy` 또는 `production` 관련 job에 `environment` 설정이 없는 경우
- 프로덕션 배포가 자동으로 실행되는 경우 (수동 승인 없음)

**심각도**: High

Before:
```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh production
```

After:
```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.example.com
    steps:
      - run: ./deploy.sh production
```

### REL-05: 실패 알림 미설정

파이프라인 실패 시 팀에 알림을 보내는 설정이 없는 경우를 탐지합니다.

**탐지 규칙**:
- `if: failure()` 조건의 알림 스텝이 없는 경우
- Slack, Teams, 이메일 등 알림 채널 설정이 없는 경우

**심각도**: Low

Before:
```yaml
jobs:
  deploy:
    steps:
      - run: ./deploy.sh
```

After:
```yaml
jobs:
  deploy:
    steps:
      - run: ./deploy.sh

      - name: Notify on failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "배포 실패: ${{ github.workflow }} - ${{ github.ref }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### REL-06: Required Status Checks 미설정

PR 머지 시 필수 상태 검사가 설정되지 않아 실패한 빌드가 머지될 수 있는 경우를 안내합니다.

**심각도**: High (보고서에 권장사항으로 제시)

## 모범 사례

### BP-01: 재사용 가능한 워크플로우

중복된 워크플로우 코드를 재사용 가능한 워크플로우 또는 composite action으로 분리할 수 있는 경우를 탐지합니다.

**탐지 규칙**:
- 여러 워크플로우 파일에서 동일한 step 블록이 반복되는 경우
- 유사한 패턴의 job이 여러 워크플로우에 존재하는 경우

**심각도**: Info

Before (`.github/workflows/ci.yml`과 `.github/workflows/release.yml`에서 중복):
```yaml
# 두 파일 모두에 동일한 setup 코드가 있음
steps:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
    with:
      node-version: '20'
      cache: 'npm'
  - run: npm ci
  - run: npm run build
```

After (`.github/workflows/reusable-build.yml`):
```yaml
name: Reusable Build
on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run build
```

호출측:
```yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
```

### BP-02: 환경 분리

개발, 스테이징, 프로덕션 환경이 분리되어 있는지 확인합니다.

**탐지 규칙**:
- 배포 워크플로우에 환경 구분 없이 단일 배포 대상만 있는 경우
- `environment` 키워드 없이 배포가 실행되는 경우

**심각도**: Medium

### BP-03: 배포 승인 게이트

프로덕션 배포에 수동 승인이 필요한지 확인합니다. GitHub Environments의 `required_reviewers` 또는 `workflow_dispatch` 트리거를 권장합니다.

**심각도**: High (프로덕션 배포 시)

### BP-04: 롤백 전략

배포 실패 시 자동 또는 수동 롤백 절차가 정의되어 있는지 확인합니다.

**탐지 규칙**:
- 배포 워크플로우에 rollback step이 없는 경우
- `if: failure()` 조건의 롤백 명령이 없는 경우

**심각도**: Medium

### BP-05: 테스트 커버리지 게이트

PR 머지 전 테스트 커버리지 기준을 충족하는지 확인하는 단계가 있는지 점검합니다.

**심각도**: Low

Before:
```yaml
- name: Run tests
  run: npm test
```

After:
```yaml
- name: Run tests with coverage
  run: npm test -- --coverage --coverageReporters=text --coverageReporters=lcov

- name: Check coverage threshold
  run: |
    COVERAGE=$(npx coverage-summary --json | jq '.total.lines.pct')
    if (( $(echo "$COVERAGE < 80" | bc -l) )); then
      echo "커버리지가 80% 미만입니다: ${COVERAGE}%"
      exit 1
    fi
```

## 플랫폼별 분석 항목

### GitHub Actions 전용

| 분석 항목 | 설명 | 심각도 |
|-----------|------|--------|
| `permissions` 블록 검증 | 워크플로우/job 레벨 최소 권한 | High |
| `pull_request_target` 안전성 | 포크 PR에서 시크릿 노출 방지 | Critical |
| `CODEOWNERS` 파일 존재 | 워크플로우 변경 리뷰 강제 | Medium |
| `actions/cache` 활용 | 의존성 캐싱 최적화 | Medium |
| `concurrency` 설정 | 중복 실행 방지 | Medium |
| 커스텀 액션 버전 고정 | SHA 기반 버전 참조 | High |
| `environment` 보호 규칙 | 배포 승인 게이트 | High |
| `workflow_dispatch` 입력 검증 | 수동 실행 입력 안전성 | Medium |

### GitLab CI 전용

| 분석 항목 | 설명 | 심각도 |
|-----------|------|--------|
| `rules` vs `only/except` | 최신 구문 사용 권장 | Info |
| Protected variables 사용 | 시크릿 변수 보호 | High |
| `artifacts:expire_in` 설정 | 아티팩트 보관 기간 관리 | Low |
| `cache:key` 적절성 | 캐시 키 전략 최적화 | Medium |
| `needs` 키워드 활용 | DAG 기반 병렬화 | Low |
| `interruptible` 설정 | 불필요한 실행 취소 | Low |
| `environment:auto_stop_in` | 리뷰 환경 자동 정리 | Low |

### Jenkins 전용

| 분석 항목 | 설명 | 심각도 |
|-----------|------|--------|
| Scripted vs Declarative | Declarative Pipeline 권장 | Info |
| `credentials()` 사용 | 자격증명 관리 방식 | High |
| `agent` 라벨 적절성 | 빌드 에이전트 선택 | Medium |
| `post` 블록 활용 | 성공/실패 후속 처리 | Medium |
| `options { timeout() }` | 타임아웃 설정 | Medium |
| `disableConcurrentBuilds()` | 동시 빌드 방지 | Medium |
| 보안 플러그인 활용 | OWASP, Snyk 등 연동 | Medium |

### CircleCI 전용

| 분석 항목 | 설명 | 심각도 |
|-----------|------|--------|
| `orbs` 버전 고정 | Orb 버전 명시적 지정 | High |
| `resource_class` 적절성 | 리소스 크기 최적화 | Low |
| `persist_to_workspace` 활용 | 워크스페이스를 통한 데이터 공유 | Low |
| `contexts` 사용 | 시크릿 컨텍스트 관리 | High |
| `no_output_timeout` 설정 | 무응답 타임아웃 | Medium |
| `parallelism` 활용 | 테스트 병렬 실행 | Low |

## 심각도 분류

### Critical (즉시 대응)

- 시크릿 하드코딩 (설정 파일에 자격증명 직접 노출)
- `pull_request_target` + PR HEAD 체크아웃 (시크릿 노출)
- 신뢰할 수 없는 입력의 직접 실행 (커맨드 인젝션)
- 인증 없는 self-hosted 러너에서 포크 PR 실행

### High (48시간 내 대응)

- `permissions` 미설정 또는 과도한 권한
- 서드파티 액션 SHA 미고정 (공급망 공격 위험)
- 프로덕션 배포에 승인 게이트 없음
- Required status checks 미설정

### Medium (1주일 내 대응)

- 의존성 캐싱 미적용
- timeout 미설정
- concurrency 미설정
- Docker 레이어 캐싱 미활용
- 환경 분리 미비

### Low (계획적 대응)

- 불필요한 조건 실행 (path 필터 미적용)
- 테스트 병렬화 미적용
- 아티팩트 관리 미비
- retry 미설정
- 테스트 커버리지 게이트 없음
- 실패 알림 미설정

### Info (참고)

- 재사용 가능한 워크플로우 분리 권장
- matrix strategy 활용 제안
- 최신 구문 사용 권장 (GitLab `rules` 등)
- 롤백 전략 문서화 권장

## 워크플로우 상세

```
1. 플랫폼 자동 감지
   ├─ .github/workflows/*.yml → GitHub Actions
   ├─ .gitlab-ci.yml → GitLab CI
   ├─ Jenkinsfile → Jenkins
   ├─ .circleci/config.yml → CircleCI
   └─ 기타 CI/CD 설정 파일

2. 설정 파일 파싱
   ├─ YAML 구조 분석 (트리거, jobs, steps)
   ├─ 환경 변수 및 시크릿 참조 추출
   ├─ 서드파티 의존성 (actions, orbs, plugins) 목록화
   └─ 배포 대상 환경 식별

3. 보안 분석 (SEC-01 ~ SEC-06)
   ├─ 시크릿 하드코딩 탐지
   ├─ 권한 설정 검증
   ├─ 액션/의존성 버전 고정 확인
   ├─ 인젝션 취약점 탐지
   ├─ pull_request_target 안전성
   └─ self-hosted 러너 보안

4. 성능 분석 (PERF-01 ~ PERF-06)
   ├─ 캐싱 전략 검증
   ├─ 병렬화 기회 식별
   ├─ 조건부 실행 검증
   ├─ Docker 캐싱 확인
   └─ 아티팩트 관리 검증

5. 안정성 분석 (REL-01 ~ REL-06)
   ├─ timeout 설정 확인
   ├─ concurrency 설정 확인
   ├─ retry 로직 확인
   ├─ 환경 보호 규칙 확인
   ├─ 실패 알림 확인
   └─ required status checks 안내

6. 모범 사례 점검 (BP-01 ~ BP-05)
   ├─ 재사용 가능한 워크플로우 분리
   ├─ 환경 분리 (dev/staging/prod)
   ├─ 배포 승인 게이트
   ├─ 롤백 전략
   └─ 테스트 커버리지 게이트

7. 보고서 생성
   ├─ 심각도별 분류 (Critical/High/Medium/Low/Info)
   ├─ Before/After YAML 예시
   ├─ 수정 우선순위 안내
   └─ 보고서 출력 (references/templates.md 형식)
```

## 에러 처리

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | CI/CD 설정 파일 없음 | 모든 플랫폼 감지 실패 | "CI/CD 설정 파일을 찾을 수 없습니다" 안내 후 파일 경로 직접 입력 요청 |
| 2 | YAML 구문 오류 | 파싱 실패 | 구문 오류 위치를 안내하고 수정 가이드 제공 |
| 3 | 지원하지 않는 플랫폼 | 설정 파일 형식 미인식 | 일반적인 CI/CD 보안 점검만 수행 |
| 4 | 대규모 파이프라인 | 설정 파일 > 1000줄 | 주요 보안/성능 항목 우선 분석 |
| 5 | 암호화된 시크릿 참조 | 실제 값 확인 불가 | 시크릿 참조 패턴만 검증, 값 자체는 분석 대상 제외 |
| 6 | 복합 파이프라인 | include, extends 등 참조 | 참조 파일도 함께 읽어서 전체 구조 분석 |
| 7 | Git 저장소 아님 | `git status` 실패 | 현재 디렉토리의 설정 파일만 분석 |
| 8 | 커스텀 액션 소스 없음 | `uses: ./` 참조 파일 없음 | 참조 누락 경고 후 나머지 분석 계속 진행 |

## Best Practices

1. **보안 우선**: 시크릿 하드코딩, 인젝션 취약점은 반드시 즉시 수정
2. **최소 권한 원칙**: `permissions` 블록에 필요한 최소 권한만 명시
3. **버전 고정**: 모든 서드파티 액션/orb를 SHA로 고정하여 공급망 공격 방지
4. **캐싱 활용**: 의존성 캐싱으로 빌드 시간 단축
5. **병렬화**: 독립적인 작업은 병렬 실행하여 전체 파이프라인 시간 단축
6. **환경 분리**: dev, staging, production 환경을 명확히 분리
7. **승인 게이트**: 프로덕션 배포에 반드시 수동 승인 또는 자동 검증 단계 포함
8. **모니터링**: 파이프라인 실패 시 즉각적인 알림 설정
9. **재사용**: 중복 코드는 재사용 가능한 워크플로우/composite action으로 분리
10. **문서화**: 파이프라인 구조와 배포 절차를 문서로 관리

## 참조 문서

- [references/analysis-guide.md](references/analysis-guide.md) - 상세 분석 방법론, 플랫폼별 탐지 규칙, 심각도 판단 기준, false positive 처리
- [references/templates.md](references/templates.md) - 보고서 출력 형식, Before/After 예시 템플릿, 수정 가이드 템플릿

## 관련 스킬

- **security-scanner**: 코드 레벨 보안 취약점 스캔 (CI/CD 설정이 아닌 애플리케이션 코드)
- **pr-review-checklist**: PR 생성 시 인프라/CI 변경에 대한 체크리스트 생성
