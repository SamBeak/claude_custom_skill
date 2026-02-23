# CI/CD Pipeline Reviewer 템플릿

이 문서는 CI/CD 파이프라인 분석 보고서 출력 형식, 발견사항별 수정 가이드 템플릿, Before/After 예시 형식, 요약 보고서 형식을 정의합니다.

---

## 1. 전체 분석 보고서 템플릿

```markdown
# CI/CD 파이프라인 분석 보고서

> 분석 일시: {SCAN_DATE}
> 프로젝트: {PROJECT_NAME}
> 감지된 플랫폼: {PLATFORM_NAME}
> 분석 파일: {FILE_LIST}

---

## 요약

| 심각도 | 발견 건수 |
|--------|----------|
| Critical | {CRITICAL_COUNT} |
| High | {HIGH_COUNT} |
| Medium | {MEDIUM_COUNT} |
| Low | {LOW_COUNT} |
| Info | {INFO_COUNT} |
| **합계** | **{TOTAL_COUNT}** |

### 주요 발견 사항

{TOP_FINDINGS_SUMMARY}

---

## 보안 분석 결과

### [{FINDING_ID}] {FINDING_TITLE}

- **심각도**: {SEVERITY}
- **카테고리**: 보안 ({SEC_ID})
- **파일**: `{FILE_PATH}:{LINE_NUMBER}`
- **설명**: {DESCRIPTION}

**수정 전**:
```yaml
{BEFORE_CODE}
```

**수정 후**:
```yaml
{AFTER_CODE}
```

**참조**: {REFERENCE_URL}

---

## 성능 분석 결과

### [{FINDING_ID}] {FINDING_TITLE}

- **심각도**: {SEVERITY}
- **카테고리**: 성능 ({PERF_ID})
- **파일**: `{FILE_PATH}:{LINE_NUMBER}`
- **예상 효과**: {EXPECTED_IMPROVEMENT}

**수정 전**:
```yaml
{BEFORE_CODE}
```

**수정 후**:
```yaml
{AFTER_CODE}
```

---

## 안정성 분석 결과

(동일 형식 반복)

---

## 모범 사례 점검 결과

(동일 형식 반복)

---

## 수정 우선순위

### 즉시 대응 (Critical)
1. {ACTION_ITEM}

### 단기 대응 (High - 48시간 내)
1. {ACTION_ITEM}

### 중기 대응 (Medium - 1주일 내)
1. {ACTION_ITEM}

### 장기 대응 (Low/Info - 계획적)
1. {ACTION_ITEM}
```

---

## 2. 간략 보고서 템플릿

빠른 점검 시 사용하는 간략한 형식입니다.

```markdown
## CI/CD 파이프라인 점검 결과

> 플랫폼: {PLATFORM_NAME}
> 파일: {FILE_LIST}

### 결과 요약
- Critical: {CRITICAL_COUNT}건
- High: {HIGH_COUNT}건
- Medium: {MEDIUM_COUNT}건
- Low: {LOW_COUNT}건

{IF_CRITICAL_OR_HIGH}
### 즉시 수정 필요

| # | 심각도 | 항목 | 파일:줄 | 설명 |
|---|--------|------|---------|------|
| 1 | {SEVERITY} | {FINDING_ID} | {FILE}:{LINE} | {DESCRIPTION} |
{END_IF}

{IF_CLEAN}
보안 또는 안정성 관련 주요 이슈가 발견되지 않았습니다.
{END_IF}
```

---

## 3. 발견사항별 상세 템플릿

### SEC-01: 시크릿 하드코딩 수정 가이드

```markdown
## [SEC-01] 시크릿 하드코딩 발견

### 문제 설명
CI/CD 설정 파일에 자격증명, API 키, 토큰 등이 직접 작성되어 있습니다.
이 파일은 Git 저장소에 포함되므로, 저장소에 접근할 수 있는 모든 사람에게 자격증명이 노출됩니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`
- 유형: {SECRET_TYPE}

### 수정 방법

#### GitHub Actions
수정 전:
```yaml
env:
  DATABASE_URL: postgres://admin:secretpassword@db.example.com:5432/mydb
  API_KEY: sk_live_abcdef123456
```

수정 후:
```yaml
env:
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  API_KEY: ${{ secrets.API_KEY }}
```

시크릿 등록: Repository Settings > Secrets and variables > Actions > New repository secret

#### GitLab CI
수정 전:
```yaml
variables:
  DATABASE_URL: "postgres://admin:secretpassword@db.example.com:5432/mydb"
```

수정 후:
```yaml
variables:
  DATABASE_URL: $DATABASE_URL  # GitLab CI/CD Variables에서 설정
```

시크릿 등록: Settings > CI/CD > Variables > Add variable (Protected + Masked)

#### Jenkins
수정 전:
```groovy
environment {
    DB_PASSWORD = 'secretpassword'
}
```

수정 후:
```groovy
environment {
    DB_PASSWORD = credentials('db-password-credential-id')
}
```

시크릿 등록: Jenkins > Manage Jenkins > Credentials

### 추가 조치
- 노출된 자격증명을 즉시 교체(rotate)하세요
- Git 히스토리에서 노출된 값을 제거하세요 (BFG Repo-Cleaner 사용)
```

### SEC-03: 서드파티 액션 SHA 고정 가이드

```markdown
## [SEC-03] 서드파티 액션 버전 미고정

### 문제 설명
서드파티 GitHub Actions를 태그(@v1)로만 참조하고 있습니다.
태그는 변경 가능하므로, 악의적인 코드가 삽입된 버전으로 교체될 수 있습니다 (공급망 공격).

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`
- 액션: `{ACTION_NAME}@{TAG}`

### SHA 확인 방법

```bash
# 특정 태그의 commit SHA 확인
gh api repos/{OWNER}/{REPO}/git/ref/tags/{TAG} --jq '.object.sha'

# 또는 git ls-remote
git ls-remote https://github.com/{OWNER}/{REPO}.git refs/tags/{TAG}
```

### 수정 예시

수정 전:
```yaml
- uses: actions/checkout@v4
- uses: some-org/deploy-action@v2
```

수정 후:
```yaml
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
- uses: some-org/deploy-action@a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0 # v2.3.0
```

주석에 원래 태그/버전을 기록하면 업데이트 시 편리합니다.

### 자동화 도구
- Dependabot: `.github/dependabot.yml`에 `github-actions` 생태계 추가
- Renovate: GitHub Actions 업데이트 자동화

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```
```

### PERF-01: 캐싱 전략 적용 가이드

```markdown
## [PERF-01] 캐싱 전략 미적용

### 문제 설명
의존성 설치 시 캐싱이 설정되지 않아 매 실행마다 패키지를 네트워크에서 다운로드합니다.
캐싱을 적용하면 빌드 시간을 30초~2분 이상 단축할 수 있습니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`
- 패키지 매니저: {PACKAGE_MANAGER}

### 플랫폼별 수정 방법

#### GitHub Actions - Node.js
```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'  # 이 한 줄 추가로 캐싱 활성화
```

#### GitHub Actions - Python
```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: 'pip'
```

#### GitHub Actions - Go
```yaml
- uses: actions/setup-go@v5
  with:
    go-version: '1.22'
    cache: true
```

#### GitLab CI
```yaml
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/
  policy: pull-push
```

#### CircleCI
```yaml
steps:
  - restore_cache:
      keys:
        - npm-deps-{{ checksum "package-lock.json" }}
        - npm-deps-
  - run: npm ci
  - save_cache:
      key: npm-deps-{{ checksum "package-lock.json" }}
      paths:
        - node_modules
```
```

### REL-02: Concurrency 설정 가이드

```markdown
## [REL-02] Concurrency 미설정

### 문제 설명
동일 브랜치/PR에 대해 여러 워크플로우가 동시에 실행될 수 있습니다.
이로 인해 리소스 낭비, 배포 충돌, 빌드 큐 지연이 발생할 수 있습니다.

### 수정 방법

#### GitHub Actions
```yaml
# 워크플로우 레벨에 추가
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}
```

#### GitLab CI
```yaml
# interruptible 키워드로 이전 파이프라인 취소
stages:
  - test
  - deploy

test:
  interruptible: true
  script:
    - npm test

# 또는 auto_cancel 기능 (GitLab 16.x+)
workflow:
  auto_cancel:
    on_new_commit: interruptible
```

#### Jenkins
```groovy
pipeline {
    options {
        disableConcurrentBuilds()
    }
    // ...
}
```
```

---

## 4. Before/After 비교 형식

모든 발견사항에서 사용하는 Before/After 비교의 표준 형식입니다.

```markdown
### [{FINDING_ID}] {FINDING_TITLE}

**파일**: `{FILE_PATH}:{LINE_NUMBER}`

수정 전:
```yaml
# 문제가 되는 현재 코드 (하이라이트할 부분에 주석 추가)
{ORIGINAL_CODE_WITH_COMMENT}
```

수정 후:
```yaml
# 수정된 코드 (변경된 부분에 주석 추가)
{FIXED_CODE_WITH_COMMENT}
```

**변경 사유**: {REASON}
**예상 효과**: {EXPECTED_EFFECT}
```

---

## 5. 플랫폼별 보고서 접두어

각 플랫폼에 맞는 보고서 접두어를 사용합니다.

### GitHub Actions

```markdown
## GitHub Actions 파이프라인 분석

> 워크플로우 파일: {WORKFLOW_FILES}
> 총 워크플로우 수: {WORKFLOW_COUNT}
> 총 Job 수: {JOB_COUNT}
> 사용 중인 서드파티 액션: {ACTION_COUNT}개
```

### GitLab CI

```markdown
## GitLab CI 파이프라인 분석

> 설정 파일: .gitlab-ci.yml
> 포함 파일: {INCLUDE_FILES}
> 총 Stage 수: {STAGE_COUNT}
> 총 Job 수: {JOB_COUNT}
> 사용 중인 변수: {VARIABLE_COUNT}개
```

### Jenkins

```markdown
## Jenkins 파이프라인 분석

> 파이프라인 유형: {Declarative|Scripted}
> 파일: {JENKINSFILE_PATH}
> 총 Stage 수: {STAGE_COUNT}
> 사용 중인 플러그인: {PLUGIN_COUNT}개
```

### CircleCI

```markdown
## CircleCI 파이프라인 분석

> 설정 파일: .circleci/config.yml
> Config 버전: {CONFIG_VERSION}
> 총 Job 수: {JOB_COUNT}
> 사용 중인 Orb: {ORB_LIST}
```

---

## 6. 종합 권장사항 템플릿

보고서 마지막에 포함되는 종합 권장사항입니다.

```markdown
---

## 종합 권장사항

### 보안 강화
- [ ] 모든 시크릿을 플랫폼 시크릿 관리 기능으로 이전
- [ ] 서드파티 액션/orb를 SHA 또는 정확한 버전으로 고정
- [ ] permissions 블록에 최소 필요 권한만 명시
- [ ] 신뢰할 수 없는 입력(PR 제목 등)을 환경 변수로 우회
- [ ] CODEOWNERS에 CI/CD 설정 파일 경로 등록

### 성능 최적화
- [ ] 의존성 캐싱 설정 (npm, pip, gradle 등)
- [ ] 독립적인 작업을 병렬 실행으로 전환
- [ ] 문서/비코드 변경에 대한 paths 필터 적용
- [ ] Docker 빌드 시 레이어 캐싱 활성화
- [ ] 빌드 아티팩트를 job 간 공유하여 중복 빌드 제거

### 안정성 개선
- [ ] 모든 job에 적절한 timeout 설정
- [ ] concurrency 설정으로 중복 실행 방지
- [ ] 네트워크 의존 작업에 retry 로직 추가
- [ ] 프로덕션 배포에 environment 보호 규칙 설정
- [ ] 파이프라인 실패 시 알림 채널 설정

### 장기 개선
- [ ] 중복 코드를 재사용 가능한 워크플로우로 분리
- [ ] 환경별(dev/staging/prod) 배포 워크플로우 분리
- [ ] 배포 실패 시 자동 롤백 절차 추가
- [ ] 테스트 커버리지 게이트 설정
- [ ] Dependabot/Renovate로 액션 버전 자동 업데이트
```

---

## 7. 에러 메시지 템플릿

### CI/CD 설정 파일 미발견

```markdown
## CI/CD 설정 파일을 찾을 수 없습니다

현재 프로젝트에서 CI/CD 설정 파일을 탐지하지 못했습니다.

확인된 경로:
- .github/workflows/*.yml - 없음
- .gitlab-ci.yml - 없음
- Jenkinsfile - 없음
- .circleci/config.yml - 없음

다음 중 하나를 시도하세요:
1. CI/CD 설정 파일의 경로를 직접 알려주세요
2. 프로젝트 루트 디렉토리에서 다시 실행해주세요
```

### YAML 구문 오류

```markdown
## YAML 구문 오류 발견

파일 `{FILE_PATH}`에서 YAML 구문 오류가 발견되었습니다.

오류 위치: {LINE}:{COLUMN}
오류 내용: {ERROR_MESSAGE}

파이프라인 분석을 계속하기 전에 구문 오류를 먼저 수정하세요.
YAML 문법 검증: https://www.yamllint.com/
```

### 지원하지 않는 플랫폼

```markdown
## 지원하지 않는 CI/CD 플랫폼

감지된 설정 파일 `{FILE_PATH}`의 CI/CD 플랫폼을 특정할 수 없습니다.

일반적인 CI/CD 보안 점검만 수행합니다:
- 시크릿 하드코딩 탐지
- 기본적인 보안 설정 확인
```
