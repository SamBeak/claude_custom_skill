# Dependency Manager 템플릿

이 문서는 의존성 분석 보고서, 업그레이드 플랜, 마이그레이션 가이드, 라이선스 호환성 보고서, 에러 메시지의 출력 형식 템플릿을 정의합니다.

---

## 1. 의존성 분석 보고서 출력 형식

### 전체 보고서 템플릿

```markdown
# 의존성 분석 보고서

> 분석 일시: {SCAN_DATE}
> 프로젝트: {PROJECT_NAME}
> 패키지 매니저: {PACKAGE_MANAGER} ({LANGUAGE})
> 총 의존성: {TOTAL_DEPS}개 (직접: {DIRECT_DEPS}개, 개발: {DEV_DEPS}개)
> Lock 파일: {LOCK_FILE_STATUS}

---

## 요약

| 분류 | 건수 | 비율 |
|------|------|------|
| Up-to-date | {UPTODATE_COUNT} | {UPTODATE_PERCENT}% |
| Patch 업데이트 | {PATCH_COUNT} | {PATCH_PERCENT}% |
| Minor 업데이트 | {MINOR_COUNT} | {MINOR_PERCENT}% |
| Major 업데이트 | {MAJOR_COUNT} | {MAJOR_PERCENT}% |
| **합계** | **{TOTAL_DEPS}** | **100%** |

### 핵심 지표

- 보안 취약점(CVE): {CVE_COUNT}건 (Critical: {CVE_CRITICAL}, High: {CVE_HIGH})
- 라이선스 충돌: {LICENSE_CONFLICT_COUNT}건
- 사용하지 않는 의존성: {UNUSED_COUNT}건

---

## 오래된 패키지 상세

### Major 업데이트 ({MAJOR_COUNT}건)

| 패키지 | 현재 버전 | 최신 버전 | 영향도 | CVE | 라이선스 |
|--------|----------|----------|--------|-----|---------|
| {PACKAGE_NAME} | {CURRENT_VERSION} | {LATEST_VERSION} | {IMPACT_LEVEL} | {CVE_INFO} | {LICENSE} |

### Minor 업데이트 ({MINOR_COUNT}건)

| 패키지 | 현재 버전 | 최신 버전 | CVE | 라이선스 |
|--------|----------|----------|-----|---------|
| {PACKAGE_NAME} | {CURRENT_VERSION} | {LATEST_VERSION} | {CVE_INFO} | {LICENSE} |

### Patch 업데이트 ({PATCH_COUNT}건)

| 패키지 | 현재 버전 | 최신 버전 | CVE | 라이선스 |
|--------|----------|----------|-----|---------|
| {PACKAGE_NAME} | {CURRENT_VERSION} | {LATEST_VERSION} | {CVE_INFO} | {LICENSE} |

---

## 보안 취약점 (CVE)

| 패키지 | CVE ID | 심각도 | CVSS | 설명 | 수정 버전 |
|--------|--------|--------|------|------|----------|
| {PACKAGE_NAME} | {CVE_ID} | {SEVERITY} | {CVSS_SCORE} | {CVE_DESCRIPTION} | {FIX_VERSION} |

---

## 라이선스 현황

| 라이선스 | 패키지 수 | 호환성 |
|---------|----------|--------|
| {LICENSE_TYPE} | {PACKAGE_COUNT} | {COMPATIBILITY_STATUS} |

---

## 사용하지 않는 의존성

| 패키지 | 유형 | 설치 위치 | 권장 조치 |
|--------|------|----------|----------|
| {PACKAGE_NAME} | {DEP_TYPE} | {LOCATION} | {RECOMMENDATION} |

---

## 업그레이드 플랜

(아래 '업그레이드 플랜 템플릿' 참조)
```

### 간략 보고서 템플릿 (빠른 확인용)

```markdown
## 의존성 현황 요약

> {PROJECT_NAME} | {PACKAGE_MANAGER} | {SCAN_DATE}

| 항목 | 수치 |
|------|------|
| 전체 의존성 | {TOTAL_DEPS}개 |
| 오래된 패키지 | {OUTDATED_COUNT}개 ({OUTDATED_PERCENT}%) |
| Major 업데이트 | {MAJOR_COUNT}건 |
| 보안 취약점 | {CVE_COUNT}건 |
| 라이선스 충돌 | {LICENSE_CONFLICT_COUNT}건 |

{IF_CRITICAL}
### 즉시 조치 필요

{CRITICAL_ITEMS_LIST}
{END_IF}

{IF_CLEAN}
모든 의존성이 양호한 상태입니다.
{END_IF}
```

---

## 2. 업그레이드 플랜 템플릿

### 우선순위 기반 업그레이드 플랜

```markdown
# 업그레이드 플랜

> 생성 일시: {PLAN_DATE}
> 프로젝트: {PROJECT_NAME}
> 대상 패키지: {TARGET_COUNT}개

---

## P0: 긴급 (즉시 대응 - 24시간 내)

{IF_P0_EXISTS}
### {PACKAGE_NAME} {CURRENT_VERSION} -> {TARGET_VERSION}

- **사유**: {UPGRADE_REASON}
- **CVE**: {CVE_ID} (CVSS: {CVSS_SCORE})
- **업그레이드 유형**: {UPDATE_TYPE}
- **영향도**: {IMPACT_LEVEL}
- **영향 파일**: {AFFECTED_FILES_COUNT}개

**실행 명령어**:
```bash
{UPGRADE_COMMAND}
```

**검증 단계**:
1. [ ] 업그레이드 실행
2. [ ] 빌드 확인
3. [ ] 테스트 실행
4. [ ] 배포 전 확인
{END_IF}

{IF_P0_EMPTY}
긴급 업그레이드 대상이 없습니다.
{END_IF}

---

## P1: 높은 우선순위 (1주일 내)

| 패키지 | 현재 | 목표 | 유형 | 사유 |
|--------|------|------|------|------|
| {PACKAGE_NAME} | {CURRENT} | {TARGET} | {TYPE} | {REASON} |

---

## P2: 보통 우선순위 (1개월 내)

| 패키지 | 현재 | 목표 | 유형 | 사유 |
|--------|------|------|------|------|
| {PACKAGE_NAME} | {CURRENT} | {TARGET} | {TYPE} | {REASON} |

---

## P3: 낮은 우선순위 (다음 스프린트)

| 패키지 | 현재 | 목표 | 유형 |
|--------|------|------|------|
| {PACKAGE_NAME} | {CURRENT} | {TARGET} | {TYPE} |

---

## 일괄 업그레이드 명령어

### Patch 일괄 업데이트
```bash
{BATCH_PATCH_COMMAND}
```

### Minor 일괄 업데이트
```bash
{BATCH_MINOR_COMMAND}
```

### Major 개별 업데이트
```bash
# 각 Major 업데이트는 개별 실행 및 테스트 권장
{MAJOR_UPGRADE_COMMANDS}
```

---

## 테스트 전략

### 업그레이드 전 확인
- [ ] 현재 전체 테스트 통과 확인
- [ ] 테스트 커버리지 기준선 기록

### 업그레이드 후 검증
- [ ] 단위 테스트 통과
- [ ] 통합 테스트 통과
- [ ] E2E 테스트 통과
- [ ] 빌드 정상 완료
- [ ] 런타임 에러 없음

### 롤백 계획
```bash
{ROLLBACK_COMMAND}
```
```

---

## 3. 마이그레이션 가이드 템플릿

### 메이저 업그레이드 마이그레이션 가이드

```markdown
# 마이그레이션 가이드: {PACKAGE_NAME} {CURRENT_VERSION} -> {TARGET_VERSION}

> 생성 일시: {GUIDE_DATE}
> 영향도: {IMPACT_LEVEL}
> 예상 작업 시간: {ESTIMATED_TIME}

---

## 요약

| 항목 | 수치 |
|------|------|
| Breaking Changes | {BREAKING_COUNT}건 |
| Deprecated API 사용 | {DEPRECATED_COUNT}건 |
| 삭제된 API 사용 | {REMOVED_COUNT}건 |
| 영향받는 파일 | {AFFECTED_FILES}개 |

---

## Breaking Changes

### {CHANGE_NUMBER}. {CHANGE_TITLE}

**변경 내용**: {CHANGE_DESCRIPTION}

**영향 파일**:
- `{FILE_PATH}:{LINE_NUMBER}`

**수정 전**:
```{LANGUAGE}
{BEFORE_CODE}
```

**수정 후**:
```{LANGUAGE}
{AFTER_CODE}
```

**참고**: {REFERENCE_URL}

---

## 마이그레이션 단계

### Phase 1: 사전 준비

- [ ] 현재 전체 테스트 통과 확인
- [ ] 마이그레이션 전용 브랜치 생성
  ```bash
  git checkout -b migrate/{PACKAGE_NAME}-{TARGET_VERSION}
  ```
- [ ] Deprecated API 사전 교체 ({DEPRECATED_COUNT}건)

### Phase 2: 업그레이드 실행

- [ ] 패키지 업그레이드 실행
  ```bash
  {UPGRADE_COMMAND}
  ```
- [ ] 피어 의존성(peer dependencies) 호환성 확인
- [ ] 빌드 에러 수정

### Phase 3: Breaking Change 적용

{BREAKING_CHANGE_STEPS}

### Phase 4: 검증

- [ ] 전체 테스트 실행
  ```bash
  {TEST_COMMAND}
  ```
- [ ] 빌드 확인
  ```bash
  {BUILD_COMMAND}
  ```
- [ ] 스테이징 환경 배포 및 확인
- [ ] E2E 테스트 실행

### Phase 5: 완료

- [ ] PR 생성 및 코드 리뷰
- [ ] 프로덕션 배포
- [ ] 모니터링 (배포 후 24시간)

---

## 관련 리소스

- 공식 마이그레이션 가이드: {OFFICIAL_GUIDE_URL}
- CHANGELOG: {CHANGELOG_URL}
- Release Notes: {RELEASE_NOTES_URL}
```

---

## 4. 라이선스 호환성 보고서 템플릿

```markdown
# 라이선스 호환성 보고서

> 분석 일시: {SCAN_DATE}
> 프로젝트: {PROJECT_NAME}
> 프로젝트 라이선스: {PROJECT_LICENSE}
> 분석 대상: {TOTAL_PACKAGES}개 패키지

---

## 라이선스 분포

| 라이선스 | 카테고리 | 패키지 수 | 비율 |
|---------|---------|----------|------|
| MIT | Permissive | {MIT_COUNT} | {MIT_PERCENT}% |
| Apache-2.0 | Permissive | {APACHE_COUNT} | {APACHE_PERCENT}% |
| BSD-3-Clause | Permissive | {BSD3_COUNT} | {BSD3_PERCENT}% |
| ISC | Permissive | {ISC_COUNT} | {ISC_PERCENT}% |
| LGPL-3.0 | Weak Copyleft | {LGPL_COUNT} | {LGPL_PERCENT}% |
| GPL-3.0 | Strong Copyleft | {GPL_COUNT} | {GPL_PERCENT}% |
| 기타 | - | {OTHER_COUNT} | {OTHER_PERCENT}% |
| 미명시 | - | {UNKNOWN_COUNT} | {UNKNOWN_PERCENT}% |

---

## 호환성 검증 결과

### 호환 ({COMPATIBLE_COUNT}건)

프로젝트 라이선스 `{PROJECT_LICENSE}`와 호환되는 의존성입니다.

| 패키지 | 버전 | 라이선스 |
|--------|------|---------|
| {PACKAGE_NAME} | {VERSION} | {LICENSE} |

### 충돌 ({CONFLICT_COUNT}건)

{IF_CONFLICTS}
| # | 패키지 | 라이선스 | 심각도 | 설명 | 권장 조치 |
|---|--------|---------|--------|------|----------|
| {NUM} | {PACKAGE_NAME}@{VERSION} | {LICENSE} | {SEVERITY} | {DESCRIPTION} | {RECOMMENDATION} |
{END_IF}

{IF_NO_CONFLICTS}
라이선스 충돌이 발견되지 않았습니다.
{END_IF}

### 미명시 ({UNKNOWN_COUNT}건)

{IF_UNKNOWN}
| 패키지 | 버전 | license 필드 | 권장 조치 |
|--------|------|-------------|----------|
| {PACKAGE_NAME} | {VERSION} | {LICENSE_FIELD} | {RECOMMENDATION} |
{END_IF}

---

## 권장 조치

### 즉시 대응 (Critical)
{CRITICAL_ACTIONS}

### 검토 필요 (High)
{HIGH_ACTIONS}

### 참고 사항
{INFO_NOTES}
```

---

## 5. 에러 메시지 템플릿

### 매니페스트 파일 미존재

```markdown
## 매니페스트 파일을 찾을 수 없습니다

현재 디렉토리에서 지원되는 패키지 매니페스트 파일을 찾지 못했습니다.

### 지원 매니페스트 파일

| 언어 | 파일명 |
|------|--------|
| JavaScript/TypeScript | `package.json` |
| Python | `requirements.txt`, `Pipfile`, `pyproject.toml` |
| Java (Maven) | `pom.xml` |
| Java/Kotlin (Gradle) | `build.gradle`, `build.gradle.kts` |
| Go | `go.mod` |
| Rust | `Cargo.toml` |
| PHP | `composer.json` |
| Ruby | `Gemfile` |

### 조치 방법

1. 프로젝트 루트 디렉토리에서 실행하고 있는지 확인하세요
2. 위 파일 중 하나 이상이 프로젝트에 존재하는지 확인하세요
3. 매니페스트 파일이 다른 위치에 있다면 해당 경로를 지정해 주세요
```

### 패키지 매니저 미설치

```markdown
## {PACKAGE_MANAGER} 도구를 찾을 수 없습니다

`{COMMAND_NAME}` 명령어를 실행할 수 없습니다. {PACKAGE_MANAGER}가 설치되어 있지 않거나 PATH에 등록되지 않았습니다.

### 설치 방법

```bash
{INSTALL_COMMAND}
```

### 대안

매니페스트 파일(`{MANIFEST_FILE}`)을 직접 분석하여 기본적인 의존성 정보를 제공합니다.
일부 기능(정확한 최신 버전 확인, CVE 조회 등)이 제한될 수 있습니다.
```

### Lock 파일 미존재

```markdown
## Lock 파일을 찾을 수 없습니다

`{LOCK_FILE_NAME}` 파일이 존재하지 않습니다. 매니페스트 파일 기반으로 분석을 진행하지만, 실제 설치된 버전과 차이가 있을 수 있습니다.

### Lock 파일 생성 권장

```bash
{LOCK_GENERATE_COMMAND}
```

### Lock 파일의 중요성

- 정확한 설치 버전을 기록하여 재현 가능한 빌드 보장
- 팀원 간 동일한 의존성 버전 사용
- 보안 감사 시 정확한 버전 기반 CVE 확인 가능
- 반드시 버전 관리(Git)에 포함시키세요
```

### 네트워크 오류

```markdown
## 패키지 레지스트리에 접근할 수 없습니다

{REGISTRY_URL}에 연결할 수 없습니다.

### 가능한 원인
- 네트워크 연결 문제
- 프록시 설정 미적용
- VPN 연결 필요
- 레지스트리 서비스 일시 중단

### 대응
로컬에 캐시된 정보를 기반으로 분석을 진행합니다.
최신 버전 정보 및 CVE 데이터가 정확하지 않을 수 있습니다.

### 프록시 설정 확인

```bash
{PROXY_CONFIG_COMMANDS}
```
```

### Private 레지스트리 인증 실패

```markdown
## Private 레지스트리 인증에 실패했습니다

{REGISTRY_URL}에 대한 인증이 실패했습니다 (HTTP {STATUS_CODE}).

### 인증 설정 방법

```bash
{AUTH_CONFIG_COMMANDS}
```

### 대응
공개 패키지에 대해서만 분석을 진행합니다.
Private 패키지는 분석 결과에서 제외됩니다.
```

### SemVer 미준수 패키지

```markdown
## SemVer를 따르지 않는 패키지가 발견되었습니다

다음 패키지는 Semantic Versioning 규칙을 따르지 않아 자동 분류가 불가능합니다.

| 패키지 | 현재 버전 | 최신 버전 | 버전 형식 |
|--------|----------|----------|----------|
| {PACKAGE_NAME} | {CURRENT_VERSION} | {LATEST_VERSION} | {VERSION_FORMAT} |

### 대응
해당 패키지는 수동으로 변경 내역을 확인해 주세요:
- CHANGELOG: {CHANGELOG_URL}
- Release Notes: {RELEASE_URL}
```

### 의존성 과다

```markdown
## 의존성이 매우 많습니다 ({TOTAL_COUNT}개)

분석 시간이 오래 걸릴 수 있으므로 직접 의존성({DIRECT_COUNT}개)만 우선 분석합니다.

### 전체 분석 실행

전체 의존성(간접 포함)을 분석하려면 다시 요청해 주세요:
> "전체 의존성 분석해줘 (간접 포함)"

### 의존성 최소화 권장

패키지 수가 많으면 보안 취약점 노출 면적이 넓어집니다.
사용하지 않는 의존성을 정리하는 것을 권장합니다:

```bash
{UNUSED_CHECK_COMMAND}
```
```

### Monorepo 감지

```markdown
## Monorepo 프로젝트가 감지되었습니다

프로젝트에 다수의 매니페스트 파일이 존재합니다. 워크스페이스 단위로 분석을 분리합니다.

### 감지된 워크스페이스

| # | 경로 | 패키지 매니저 | 의존성 수 |
|---|------|-------------|----------|
| {NUM} | {WORKSPACE_PATH} | {PACKAGE_MANAGER} | {DEP_COUNT} |

### 분석 방식 선택

다음 중 선택해 주세요:
1. 전체 워크스페이스 일괄 분석
2. 특정 워크스페이스만 분석 (경로 지정)
3. 루트 매니페스트만 분석
```

---

## 6. 업그레이드 명령어 템플릿

### Node.js (npm)

```bash
# Patch 일괄 업데이트
npm update

# 특정 패키지 Minor 업데이트
npm install {PACKAGE}@{VERSION}

# 특정 패키지 Major 업데이트
npm install {PACKAGE}@latest

# 자동 보안 패치
npm audit fix

# Breaking Change 포함 자동 수정 (주의)
npm audit fix --force
```

### Python (pip)

```bash
# 특정 패키지 업데이트
pip install {PACKAGE}=={VERSION}

# requirements.txt 전체 업데이트
pip install -r requirements.txt --upgrade

# 보안 취약 패키지만 업데이트
pip install {PACKAGE}>={FIX_VERSION}
```

### Go (go modules)

```bash
# 특정 모듈 업데이트
go get {MODULE}@{VERSION}

# 모든 직접 의존성 Minor/Patch 업데이트
go get -u ./...

# 간접 의존성 정리
go mod tidy
```

### Rust (Cargo)

```bash
# 호환 범위 내 업데이트
cargo update

# 특정 크레이트 업데이트
cargo update -p {CRATE_NAME}

# Major 업데이트 (Cargo.toml 수정 필요)
# Cargo.toml에서 버전을 변경한 후:
cargo update
```

---

## 7. 자동화 도구 설정 템플릿

### Dependabot 설정

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "{ECOSYSTEM}"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Asia/Seoul"
    open-pull-requests-limit: 10
    reviewers:
      - "{REVIEWER}"
    labels:
      - "dependencies"
      - "automated"
    commit-message:
      prefix: "deps"
      include: "scope"
    # 보안 업데이트는 즉시
    # 일반 업데이트는 주간
    ignore:
      - dependency-name: "*"
        update-types: ["version-update:semver-major"]
    groups:
      patch-updates:
        update-types:
          - "patch"
      minor-updates:
        update-types:
          - "minor"
```

### Renovate Bot 설정

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    "schedule:weekends",
    ":semanticCommits",
    "group:allNonMajor"
  ],
  "labels": ["dependencies", "automated"],
  "timezone": "Asia/Seoul",
  "prHourlyLimit": 5,
  "prConcurrentLimit": 10,
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true,
      "automergeType": "branch"
    },
    {
      "matchUpdateTypes": ["major"],
      "labels": ["dependencies", "breaking-change"],
      "automerge": false
    },
    {
      "matchDepTypes": ["devDependencies"],
      "automerge": true
    }
  ],
  "vulnerabilityAlerts": {
    "labels": ["security"],
    "automerge": true
  }
}
```
