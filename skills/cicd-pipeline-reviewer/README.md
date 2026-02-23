# CI/CD Pipeline Reviewer

## 스킬 소개

**CI/CD Pipeline Reviewer**는 Claude Code 커스텀 스킬로, 프로젝트의 CI/CD 파이프라인 설정 파일을 자동으로 탐지하고 분석합니다. 보안 취약점, 성능 병목, 안정성 문제를 식별하고 각 발견사항에 대해 구체적인 Before/After YAML 예시와 함께 수정 가이드를 제공합니다.

GitHub Actions, GitLab CI, Jenkins, CircleCI 등 주요 CI/CD 플랫폼을 지원하며, 프로젝트에서 사용 중인 플랫폼을 자동으로 감지합니다.

### 주요 기능

- **플랫폼 자동 감지**: GitHub Actions, GitLab CI, Jenkins, CircleCI, Travis CI, Azure Pipelines 등 자동 식별
- **보안 분석**: 시크릿 하드코딩, 과도한 권한, 서드파티 액션 버전 미고정, 인젝션 취약점 탐지
- **성능 최적화**: 캐싱 전략, 병렬화, 조건부 실행, Docker 레이어 캐싱, 아티팩트 관리 점검
- **안정성 분석**: timeout, concurrency, retry, 환경 보호 규칙, 실패 알림 설정 확인
- **모범 사례 점검**: 재사용 가능한 워크플로우, 환경 분리, 배포 승인 게이트, 롤백 전략, 테스트 커버리지 게이트
- **심각도별 분류**: Critical / High / Medium / Low / Info 5단계 분류
- **Before/After 예시**: 각 발견사항에 대해 수정 전후 YAML 코드를 비교하여 제시

---

## 분석 카테고리

### 1. 보안 분석 (SEC)

CI/CD 파이프라인에서 가장 중요한 보안 위험을 탐지합니다:

| ID | 항목 | 심각도 |
|----|------|--------|
| SEC-01 | 시크릿 하드코딩 (설정 파일에 자격증명 직접 노출) | Critical |
| SEC-02 | GITHUB_TOKEN 과도한 권한 (permissions 미설정) | High |
| SEC-03 | 서드파티 액션 버전 미고정 (SHA 대신 태그만 사용) | High |
| SEC-04 | 신뢰할 수 없는 입력 사용 (PR 제목 인젝션 등) | Critical |
| SEC-05 | pull_request_target 오용 (포크에서 시크릿 노출) | Critical |
| SEC-06 | self-hosted 러너 보안 (공개 저장소에서 사용) | Medium~High |

### 2. 성능 최적화 (PERF)

파이프라인 실행 속도를 개선할 수 있는 항목을 분석합니다:

| ID | 항목 | 심각도 |
|----|------|--------|
| PERF-01 | 캐싱 전략 미적용 (npm, pip, gradle 등) | Medium |
| PERF-02 | 병렬화 미적용 (독립 job 순차 실행) | Low |
| PERF-03 | Matrix Strategy 미활용 | Info |
| PERF-04 | 조건부 실행 미적용 (paths 필터 없음) | Low |
| PERF-05 | Docker 레이어 캐싱 미적용 | Medium |
| PERF-06 | 아티팩트 관리 미비 (중복 빌드) | Low |

### 3. 안정성 분석 (REL)

파이프라인의 안정적 실행을 보장하기 위한 항목을 점검합니다:

| ID | 항목 | 심각도 |
|----|------|--------|
| REL-01 | Timeout 미설정 (무한 대기 가능) | Medium |
| REL-02 | Concurrency 미설정 (중복 실행) | Medium |
| REL-03 | Retry 미설정 (네트워크 오류에 취약) | Low |
| REL-04 | 환경 보호 규칙 미설정 (프로덕션 보호) | High |
| REL-05 | 실패 알림 미설정 | Low |
| REL-06 | Required Status Checks 미설정 | High |

### 4. 모범 사례 (BP)

| ID | 항목 | 심각도 |
|----|------|--------|
| BP-01 | 재사용 가능한 워크플로우 분리 | Info |
| BP-02 | 환경 분리 (dev/staging/prod) | Medium |
| BP-03 | 배포 승인 게이트 | High |
| BP-04 | 롤백 전략 | Medium |
| BP-05 | 테스트 커버리지 게이트 | Low |

---

## 사용 예시

### 기본 분석 요청

```
사용자: CI/CD 파이프라인 분석해줘

Claude: CI/CD 설정 파일을 탐지합니다...

[감지된 플랫폼: GitHub Actions]
  - .github/workflows/ci.yml
  - .github/workflows/deploy.yml

분석을 시작합니다...

## CI/CD 파이프라인 분석 보고서

| 심각도 | 발견 건수 |
|--------|----------|
| Critical | 1 |
| High | 3 |
| Medium | 2 |
| Low | 1 |

### [SEC-04] Critical: 신뢰할 수 없는 입력 사용
  파일: .github/workflows/ci.yml:25
  run 스텝에서 ${{ github.event.pull_request.title }}을 직접 사용하고 있습니다.
  PR 제목에 악성 명령어를 삽입하여 코드 실행이 가능합니다.

  수정 전:
    run: echo "${{ github.event.pull_request.title }}"
  수정 후:
    env:
      PR_TITLE: ${{ github.event.pull_request.title }}
    run: echo "$PR_TITLE"
...
```

### 특정 플랫폼 분석

```
사용자: GitHub Actions 워크플로우 보안 검사해줘

Claude: .github/workflows/ 디렉토리의 워크플로우 파일을 분석합니다...
```

### 빌드 최적화 요청

```
사용자: 빌드 최적화해줘

Claude: CI/CD 설정 파일을 분석하여 성능 최적화 기회를 찾겠습니다...

[PERF-01] 캐싱 전략 미적용
  npm ci를 실행하지만 캐싱이 설정되지 않았습니다.
  예상 절감 시간: 약 30-60초

  수정 전:
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
    - run: npm ci

  수정 후:
    - uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
    - run: npm ci
...
```

---

## 지원 플랫폼

| 플랫폼 | 설정 파일 | 지원 수준 |
|--------|----------|----------|
| GitHub Actions | `.github/workflows/*.yml` | 전체 분석 (보안, 성능, 안정성, 모범사례) |
| GitLab CI | `.gitlab-ci.yml` | 전체 분석 |
| Jenkins | `Jenkinsfile` | 전체 분석 |
| CircleCI | `.circleci/config.yml` | 전체 분석 |
| Travis CI | `.travis.yml` | 기본 분석 (보안, 캐싱) |
| Azure Pipelines | `azure-pipelines.yml` | 기본 분석 |
| Bitbucket Pipelines | `bitbucket-pipelines.yml` | 기본 분석 |

---

## 에러 처리

| 에러 | 대응 |
|------|------|
| CI/CD 설정 파일 없음 | 파일 경로 직접 입력 요청 |
| YAML 구문 오류 | 오류 위치 안내 및 수정 가이드 제공 |
| 지원하지 않는 플랫폼 | 일반 CI/CD 보안 점검만 수행 |
| 대규모 파이프라인 (>1000줄) | 주요 보안/성능 항목 우선 분석 |
| 암호화된 시크릿 참조 | 시크릿 참조 패턴만 검증 |
| 복합 파이프라인 (include, extends) | 참조 파일도 함께 분석 |

---

## 관련 문서

- [SKILL.md](SKILL.md) - 스킬 정의 및 상세 워크플로우, 탐지 패턴, Before/After 예시
- [references/analysis-guide.md](references/analysis-guide.md) - 분석 방법론, 플랫폼별 상세 규칙, 심각도 판단 기준
- [references/templates.md](references/templates.md) - 보고서 출력 형식, 수정 가이드 템플릿
