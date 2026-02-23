# CI/CD Pipeline Reviewer 분석 가이드

이 문서는 CI/CD 파이프라인 분석의 상세 방법론, 플랫폼별 탐지 규칙, 심각도 판단 기준, false positive 처리 방법을 정의합니다.

---

## 1. 분석 실행 순서

CI/CD 파이프라인 분석 시 다음 순서로 실행합니다.

### 1단계: 플랫폼 감지 및 파일 수집

```bash
# 모든 CI/CD 설정 파일 탐색
echo "=== GitHub Actions ==="
ls .github/workflows/*.yml .github/workflows/*.yaml 2>/dev/null

echo "=== GitLab CI ==="
ls .gitlab-ci.yml 2>/dev/null
# include로 참조되는 파일도 탐색
grep -r "include:" .gitlab-ci.yml 2>/dev/null | grep "local:" | awk -F"'" '{print $2}'

echo "=== Jenkins ==="
ls Jenkinsfile Jenkinsfile.* 2>/dev/null
# shared library 참조 확인
grep -r "@Library" Jenkinsfile 2>/dev/null

echo "=== CircleCI ==="
ls .circleci/config.yml 2>/dev/null

echo "=== 기타 ==="
ls .travis.yml azure-pipelines.yml bitbucket-pipelines.yml 2>/dev/null
```

### 2단계: 구조 분석

각 설정 파일에서 다음 요소를 추출합니다:

- **트리거 (Trigger)**: 어떤 이벤트에 의해 파이프라인이 실행되는지
- **작업 (Jobs/Stages)**: 파이프라인을 구성하는 작업 단위
- **스텝 (Steps)**: 각 작업 내의 실행 단계
- **환경 변수**: 정의된 환경 변수와 시크릿 참조
- **의존성**: 서드파티 액션, orb, 플러그인
- **배포 대상**: 배포 환경 및 대상 서비스
- **조건부 실행**: 조건부 실행 규칙 (paths, branches 등)

### 3단계: 보안 분석 (SEC-01 ~ SEC-06)

가장 위험한 보안 취약점부터 분석합니다.

### 4단계: 성능 분석 (PERF-01 ~ PERF-06)

캐싱, 병렬화, 조건부 실행 등 성능 관련 항목을 분석합니다.

### 5단계: 안정성 분석 (REL-01 ~ REL-06)

timeout, concurrency, retry 등 안정성 관련 항목을 분석합니다.

### 6단계: 모범 사례 점검 (BP-01 ~ BP-05)

재사용성, 환경 분리, 배포 전략 등을 점검합니다.

### 7단계: False Positive 필터링

아래 FP 판단 기준을 적용하여 오탐을 제거합니다.

### 8단계: 보고서 생성

[references/templates.md](templates.md)의 형식으로 보고서를 생성합니다.

---

## 2. 보안 분석 상세 규칙

### SEC-01: 시크릿 하드코딩 탐지

**GitHub Actions**:
```regex
# env 블록에서 실제 값 할당 (시크릿 컨텍스트 미사용)
env\s*:[\s\S]*?\b\w*(KEY|SECRET|TOKEN|PASSWORD|CREDENTIAL|AUTH)\w*\s*:\s*['"]?(?!\$\{\{)[A-Za-z0-9+/=_\-\.]{8,}['"]?

# run 스텝에서 값 직접 사용
run\s*:.*--password\s+[^\$\s]+
run\s*:.*--token\s+[^\$\s]+
run\s*:.*-p\s+['"][^'"$]+['"]

# AWS 자격증명 직접 노출
AKIA[0-9A-Z]{16}
(aws_secret_access_key|AWS_SECRET_ACCESS_KEY)\s*[:=]\s*['"]?[A-Za-z0-9/+=]{40}
```

**GitLab CI**:
```regex
# variables 블록에서 실제 값 (CI/CD 변수 미사용)
variables\s*:[\s\S]*?\b\w*(KEY|SECRET|TOKEN|PASSWORD)\w*\s*:\s*['"]?(?!\$)[A-Za-z0-9+/=_\-\.]{8,}

# script에서 직접 사용
script\s*:[\s\S]*?(password|token|secret|key)\s*[:=]\s*['"][^'"$]+['"]
```

**Jenkins**:
```regex
# 파이프라인에서 자격증명 하드코딩
(password|secret|token|apiKey)\s*[:=]\s*['"][^'"]+['"]

# environment 블록에서 직접 할당 (credentials() 미사용)
environment\s*\{[\s\S]*?\w+\s*=\s*['"][^'"]+['"](?!.*credentials\()
```

### SEC-02: 권한 설정 검증

**GitHub Actions 검증 규칙**:

1. 워크플로우 레벨에 `permissions` 블록이 있는지 확인
2. `permissions`가 `write-all` 또는 `read-all`이 아닌지 확인
3. 각 권한이 실제 필요한 최소 범위인지 확인

```regex
# 워크플로우에 permissions 미설정 (최상위 레벨)
^name\s*:[\s\S]*?^on\s*:[\s\S]*?^jobs\s*:(?![\s\S]*?^permissions\s*:)

# 과도한 권한
permissions\s*:\s*(write-all|read-all)

# 개별 권한 검증 - 불필요한 write 권한
# checkout만 하는데 contents: write인 경우
# 테스트만 하는데 packages: write인 경우
```

**필요 권한 매핑**:

| 작업 | 필요 권한 |
|------|----------|
| 코드 체크아웃 | `contents: read` |
| PR 코멘트 작성 | `pull-requests: write` |
| 체크 결과 등록 | `checks: write` |
| 패키지 게시 | `packages: write` |
| 배포 상태 업데이트 | `deployments: write` |
| 이슈 생성/수정 | `issues: write` |
| 릴리스 생성 | `contents: write` |
| GitHub Pages 배포 | `pages: write`, `id-token: write` |

### SEC-03: 서드파티 액션 버전 고정 검증

**검증 규칙**:

1. `uses:` 키워드에서 참조하는 액션의 버전 형식 확인
2. `actions/*` 공식 액션도 SHA 고정 권장 (Critical은 아님)
3. 비공식 서드파티 액션은 반드시 SHA 고정 필요

```regex
# SHA가 아닌 태그만 사용하는 패턴
uses\s*:\s*(?!\./)([^/]+/[^@\s]+)@(?!([a-f0-9]{40}|[a-f0-9]{64}))[^\s]+

# 유효한 SHA 고정 패턴 (40자 또는 64자 hex)
uses\s*:\s*[^/]+/[^@]+@[a-f0-9]{40,64}
```

**예외**: 로컬 액션 (`uses: ./`) 및 재사용 워크플로우 (`uses: ./.github/workflows/`)는 SHA 고정 불필요

### SEC-04: 신뢰할 수 없는 입력 상세

**위험한 GitHub 컨텍스트 목록**:

| 컨텍스트 | 위험 수준 | 설명 |
|---------|----------|------|
| `github.event.pull_request.title` | Critical | PR 작성자가 제어 가능 |
| `github.event.pull_request.body` | Critical | PR 작성자가 제어 가능 |
| `github.event.issue.title` | Critical | 이슈 작성자가 제어 가능 |
| `github.event.issue.body` | Critical | 이슈 작성자가 제어 가능 |
| `github.event.comment.body` | Critical | 코멘트 작성자가 제어 가능 |
| `github.event.review.body` | High | 리뷰어가 제어 가능 |
| `github.event.head_commit.message` | High | 커밋 작성자가 제어 가능 |
| `github.head_ref` | High | 브랜치명에 인젝션 가능 |
| `github.event.commits[*].message` | High | 커밋 작성자가 제어 가능 |
| `github.event.commits[*].author.name` | Medium | 커밋 작성자가 제어 가능 |

**안전한 사용 패턴**:
```yaml
# 1. 환경 변수로 우회
- name: Safe usage
  env:
    TITLE: ${{ github.event.pull_request.title }}
  run: echo "$TITLE"

# 2. 스크립트에서 사용 (actions/github-script)
- uses: actions/github-script@v7
  with:
    script: |
      const title = context.payload.pull_request.title;
      // JavaScript에서 안전하게 처리
```

### SEC-05: pull_request_target 분석

**위험 패턴 탐지**:

```regex
# 위험: pull_request_target + PR HEAD 체크아웃
on\s*:[\s\S]*?pull_request_target

# 위의 트리거와 함께 아래 패턴이 있으면 Critical
ref\s*:\s*\$\{\{\s*github\.event\.pull_request\.head\.(sha|ref)\s*\}\}
ref\s*:\s*\$\{\{\s*github\.head_ref\s*\}\}
```

**안전한 대안**:
- PR에서 코드를 실행하지 않고 레이블만 붙이는 경우: `pull_request_target` 사용 가능
- PR 코드를 빌드/테스트해야 하는 경우: `pull_request` 트리거 사용
- 시크릿이 필요한 경우: 2단계 워크플로우 (pull_request로 빌드 -> workflow_run으로 배포)

---

## 3. 성능 분석 상세 규칙

### PERF-01: 캐싱 전략 분석

**언어/도구별 캐싱 설정 확인**:

| 도구 | 캐시 대상 | 권장 설정 |
|------|----------|----------|
| npm/yarn | `node_modules`, `~/.npm` | `setup-node`의 `cache` 옵션 |
| pip | `~/.cache/pip` | `setup-python`의 `cache` 옵션 |
| Gradle | `~/.gradle/caches` | `gradle/gradle-build-action` |
| Maven | `~/.m2/repository` | `actions/cache` + path 지정 |
| Go | `~/go/pkg/mod` | `setup-go`의 `cache` 옵션 |
| Rust | `~/.cargo/registry`, `target/` | `Swatinem/rust-cache` |
| Docker | Docker layer cache | `docker/build-push-action` + `cache-from/to` |

**탐지 정규식**:
```regex
# npm/yarn 사용하지만 캐싱 미설정
(npm\s+(ci|install)|yarn\s+install)(?![\s\S]*?(cache\s*:\s*['"]?npm|cache\s*:\s*['"]?yarn|actions/cache))

# pip 사용하지만 캐싱 미설정
pip\s+install(?![\s\S]*?(cache\s*:\s*['"]?pip|actions/cache.*pip))

# gradle 사용하지만 캐싱 미설정
(gradle|gradlew)(?![\s\S]*?(gradle-build-action|actions/cache.*gradle))
```

### PERF-02: 병렬화 분석

**독립 작업 식별 기준**:

1. `needs` 의존성이 설정되어 있지만 실제로는 다른 job의 출력이 필요 없는 경우
2. lint, test, security-scan 등이 순차 실행되지만 각각 독립적인 경우
3. 하나의 job에서 무관한 작업들이 순차 실행되는 경우

```regex
# 불필요한 needs 의존성 후보
needs\s*:\s*\[?\s*[\w-]+\s*\]?[\s\S]*?steps\s*:(?![\s\S]*?(download-artifact|needs\.\w+\.outputs))
```

### PERF-04: 조건부 실행 분석

**경로 필터 권장 규칙**:

| 워크플로우 유형 | 권장 paths 필터 |
|---------------|----------------|
| 프론트엔드 빌드 | `src/frontend/**`, `*.tsx`, `*.jsx`, `*.css`, `package.json` |
| 백엔드 빌드 | `src/backend/**`, `*.py`, `*.java`, `*.go`, `requirements.txt` |
| 문서 빌드 | `docs/**`, `*.md`, `mkdocs.yml` |
| 인프라 | `terraform/**`, `k8s/**`, `Dockerfile` |
| 전체 테스트 | `src/**`, `tests/**`, `package.json` |

---

## 4. 안정성 분석 상세 규칙

### REL-01: Timeout 권장값

| 작업 유형 | 권장 timeout | 이유 |
|-----------|-------------|------|
| 린트/포맷 검사 | 5-10분 | 빠르게 완료되어야 함 |
| 단위 테스트 | 15-30분 | 테스트 수에 따라 조정 |
| 통합 테스트 | 30-60분 | 외부 서비스 포함 |
| E2E 테스트 | 30-60분 | 브라우저 실행 포함 |
| Docker 빌드 | 15-30분 | 이미지 크기에 따라 |
| 배포 | 15-30분 | 서비스 규모에 따라 |
| 전체 파이프라인 | 60-120분 | 모든 작업 합산 |

### REL-02: Concurrency 설정 가이드

**권장 concurrency 그룹 패턴**:

| 시나리오 | group 키 | cancel-in-progress |
|----------|---------|-------------------|
| PR 빌드 | `ci-${{ github.ref }}` | `true` |
| main 브랜치 빌드 | `ci-main` | `false` |
| 배포 | `deploy-${{ github.ref }}` | `false` |
| 혼합 | `${{ github.workflow }}-${{ github.ref }}` | `github.ref != 'refs/heads/main'` |

### REL-04: 환경 보호 규칙 분석

**배포 job 식별 패턴**:
```regex
# job 이름에서 배포 키워드
(deploy|release|publish|push-to-prod|production|staging)[\w-]*\s*:

# step에서 배포 명령
(kubectl\s+apply|helm\s+(install|upgrade)|aws\s+ecs|gcloud\s+(app\s+deploy|run\s+deploy)|terraform\s+apply|ssh.*deploy|rsync.*deploy|scp.*deploy)

# Docker push
docker\s+push
```

---

## 5. 플랫폼별 특수 규칙

### GitHub Actions 특수 규칙

**CODEOWNERS 검증**:
```bash
# .github/workflows 변경에 대한 CODEOWNERS 설정 확인
grep -q '.github/workflows' .github/CODEOWNERS 2>/dev/null
```

CODEOWNERS에 `.github/workflows/` 경로가 없으면 누구나 워크플로우를 수정할 수 있어 보안 위험이 됩니다.

**workflow_dispatch 입력 검증**:
```regex
# 입력값이 run에서 직접 사용되는지 확인
workflow_dispatch\s*:[\s\S]*?inputs\s*:[\s\S]*?(\w+)\s*:[\s\S]*?run\s*:.*\$\{\{\s*github\.event\.inputs\.\1\s*\}\}
```

### GitLab CI 특수 규칙

**rules vs only/except**:
```regex
# 레거시 only/except 사용 감지
(only|except)\s*:
```

GitLab 12.3 이후 `rules` 키워드 사용을 권장합니다.

**include 파일 분석**:
```regex
# include된 파일 목록 추출
include\s*:[\s\S]*?-?\s*(local|project|remote|template)\s*:\s*['"]?([^'"]+)
```

### Jenkins 특수 규칙

**Scripted vs Declarative**:
```regex
# Scripted Pipeline 감지
^node\s*\{|^node\s*\(

# Declarative Pipeline 감지
^pipeline\s*\{
```

Declarative Pipeline 사용을 권장합니다.

**credentials() 사용 확인**:
```regex
# 안전: credentials() 함수 사용
credentials\s*\(\s*['"][^'"]+['"]\s*\)

# 위험: 직접 할당
environment\s*\{[\s\S]*?\w+\s*=\s*['"][^'"\$]+['"]
```

### CircleCI 특수 규칙

**Orb 버전 고정 확인**:
```regex
# 버전 범위 또는 latest 사용 (위험)
orbs\s*:[\s\S]*?\w+\s*:\s*[\w-]+/[\w-]+@(latest|volatile|\d+\.\d+)

# 정확한 버전 사용 (안전)
orbs\s*:[\s\S]*?\w+\s*:\s*[\w-]+/[\w-]+@\d+\.\d+\.\d+
```

---

## 6. False Positive 판단 기준

### 제외 조건

| 조건 | 설명 | 예시 |
|------|------|------|
| 주석 내 패턴 | YAML 주석에 포함된 패턴 | `# password: example-password` |
| 예시/문서 파일 | 실제 실행되지 않는 파일 | `docs/ci-guide.yml`, `examples/` |
| 비활성 워크플로우 | `disabled` 상태 또는 주석 처리 | 파일명에 `disabled` 포함 |
| 테스트 워크플로우 | CI 테스트용 워크플로우 | `.github/workflows/test-ci.yml` |
| 플레이스홀더 값 | 명백한 예시 값 | `YOUR_API_KEY`, `changeme`, `xxx` |
| 환경 변수 참조 | 시크릿 컨텍스트 사용 | `${{ secrets.API_KEY }}`, `$CI_VARIABLE` |
| 공개 키 | 비밀이 아닌 공개 설정 | `NEXT_PUBLIC_*`, publishable key |

### 시크릿 FP 필터링 정규식

```regex
# 플레이스홀더 패턴
(?i)(your[-_]?(api[-_]?)?key|example|placeholder|dummy|fake|test|sample|todo|changeme|replace[-_]?me|insert[-_]?here|xxx+|\.\.\.|\*{3,})

# 환경 변수 참조 (값이 아님)
\$\{\{.*secrets\..*\}\}
\$\{[A-Z_]+\}
\$[A-Z_]+
process\.env\.\w+
os\.environ\.\w+
```

### 심각도 조정 규칙

| 조건 | 조정 |
|------|------|
| 비공개(private) 저장소 | self-hosted 러너 심각도 1단계 하향 |
| 내부 전용 서비스 | 외부 접근 불가 시 심각도 1단계 하향 |
| 공식 actions/* 액션 | SHA 미고정 심각도를 Medium으로 하향 |
| `pull_request` (target 아님) | 시크릿 노출 위험 없으므로 FP |
| 주석 내 발견 | Info로 하향 |
| 예시/문서 파일 | Info로 하향 |

---

## 7. 심각도 분류 매트릭스

### 분류 기준표

| 심각도 | 공격 가능성 | 영향 범위 | 대응 시간 |
|--------|-----------|----------|----------|
| Critical | 즉시 악용 가능 | 시크릿 노출, 코드 실행 | 즉시 |
| High | 악용 가능성 높음 | 권한 상승, 공급망 공격 | 48시간 |
| Medium | 조건부 악용 | 성능 저하, 리소스 낭비 | 1주일 |
| Low | 악용 어려움 | 효율성 저하 | 1개월 |
| Info | 악용 불가 | 없음 (개선 권장) | 계획적 |

### 복합 조건 심각도 결정

| 발견사항 | 조건 | 최종 심각도 |
|---------|------|-----------|
| 시크릿 하드코딩 | 프로덕션 키 | Critical |
| 시크릿 하드코딩 | 테스트/개발 키 | High |
| 시크릿 하드코딩 | 플레이스홀더 | FP (제외) |
| 액션 SHA 미고정 | 비공식 서드파티 | High |
| 액션 SHA 미고정 | 공식 actions/* | Medium |
| 액션 SHA 미고정 | 로컬 액션 (`./ `) | FP (제외) |
| self-hosted 러너 | 공개 저장소 | High |
| self-hosted 러너 | 비공개 저장소 | Medium |
| permissions 미설정 | 시크릿 접근 있음 | High |
| permissions 미설정 | 읽기 전용 작업 | Medium |
| timeout 미설정 | 배포 작업 | Medium |
| timeout 미설정 | 린트/포맷 작업 | Low |

---

## 8. 분석 우선순위

보안 분석 > 안정성 분석 > 성능 분석 > 모범 사례 순으로 분석합니다.

### 1순위: 즉시 대응 필요 (보안)

1. SEC-01: 시크릿 하드코딩
2. SEC-04: 신뢰할 수 없는 입력 사용
3. SEC-05: pull_request_target 오용

### 2순위: 조기 대응 필요 (보안 + 안정성)

4. SEC-02: 과도한 권한
5. SEC-03: 서드파티 액션 SHA 미고정
6. REL-04: 환경 보호 규칙 미설정
7. REL-06: Required Status Checks

### 3순위: 개선 권장 (성능 + 안정성)

8. PERF-01: 캐싱 미적용
9. REL-01: Timeout 미설정
10. REL-02: Concurrency 미설정
11. PERF-05: Docker 캐싱 미적용

### 4순위: 장기 개선 (모범 사례)

12. BP-01 ~ BP-05: 재사용 워크플로우, 환경 분리, 승인 게이트, 롤백, 커버리지

---

## 9. 크로스 플랫폼 공통 규칙

플랫폼에 관계없이 적용되는 공통 분석 규칙입니다.

### 시크릿 관리

모든 플랫폼에서 자격증명은 해당 플랫폼의 시크릿 관리 기능을 통해 관리해야 합니다:

| 플랫폼 | 시크릿 관리 방법 |
|--------|----------------|
| GitHub Actions | Repository Secrets, Environment Secrets, Organization Secrets |
| GitLab CI | CI/CD Variables (Protected, Masked) |
| Jenkins | Credentials Store, HashiCorp Vault 연동 |
| CircleCI | Project Settings > Environment Variables, Contexts |

### 파이프라인 코드 리뷰

모든 CI/CD 설정 파일 변경에 대해 코드 리뷰를 강제해야 합니다:

- GitHub: `CODEOWNERS` 파일에 `.github/workflows/` 경로 등록
- GitLab: `CODEOWNERS` 또는 Merge Request Approvals
- Jenkins: Jenkinsfile 변경에 대한 PR 리뷰 강제
- CircleCI: `.circleci/config.yml` 변경에 대한 PR 리뷰 강제

### 로그 보안

파이프라인 로그에 시크릿이 노출되지 않도록 확인합니다:

```regex
# 로그에 시크릿 출력 가능 패턴
(echo|print|log|console\.log|cat)\s+.*\$(SECRET|TOKEN|PASSWORD|KEY|CREDENTIAL)
(echo|print)\s+.*\$\{\{\s*secrets\.
```
