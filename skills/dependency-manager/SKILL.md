---
name: dependency-manager
description: 프로젝트 의존성을 분석하여 오래된 패키지를 탐지하고, 메이저 업그레이드 시 마이그레이션 가이드를 제공하며, 라이선스 호환성을 검증하고, 우선순위 기반 업그레이드 플랜을 생성하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) 의존성 업데이트/분석, (2) 패키지 업그레이드 확인, (3) 마이그레이션 가이드 요청, (4) 라이선스 확인/호환성 검사, (5) outdated 패키지 확인, (6) dependency update 또는 package upgrade, (7) 업그레이드 플랜 생성, (8) CVE 기반 패키지 업그레이드 경로 확인.
---

# Dependency Manager

프로젝트의 의존성을 종합적으로 분석하여 오래된 패키지를 탐지하고, patch/minor/major 단위로 분류합니다. 메이저 업그레이드 시에는 Breaking Change 영향 분석과 마이그레이션 가이드를 제공하며, 라이선스 호환성을 검증하고, 보안 취약점(CVE)과 연계하여 우선순위 기반 업그레이드 플랜을 생성합니다.

## Quick Start

사용자가 의존성 분석을 요청하면 다음 워크플로우를 실행합니다:

1. **패키지 매니저 자동 감지**:
   ```bash
   # 프로젝트 루트에서 매니페스트 파일 탐지
   if [ -f "package.json" ]; then
       echo "Node.js (npm/yarn/pnpm)"
   fi
   if [ -f "requirements.txt" ] || [ -f "Pipfile" ] || [ -f "pyproject.toml" ]; then
       echo "Python (pip/pipenv/poetry)"
   fi
   if [ -f "pom.xml" ]; then
       echo "Java (Maven)"
   fi
   if [ -f "build.gradle" ] || [ -f "build.gradle.kts" ]; then
       echo "Java/Kotlin (Gradle)"
   fi
   if [ -f "go.mod" ]; then
       echo "Go (go modules)"
   fi
   if [ -f "Cargo.toml" ]; then
       echo "Rust (Cargo)"
   fi
   if [ -f "composer.json" ]; then
       echo "PHP (Composer)"
   fi
   if [ -f "Gemfile" ]; then
       echo "Ruby (Bundler)"
   fi
   ```

2. **오래된 패키지 목록 수집** 후 patch/minor/major 분류

3. **보안 취약점(CVE) 연계 확인** (security-scanner 스킬 연동)

4. **라이선스 호환성 검증** (MIT/Apache/GPL 매트릭스)

5. **메이저 업그레이드 대상에 대한 Breaking Change 영향 분석**

6. **우선순위 기반 업그레이드 플랜 생성** ([references/templates.md](references/templates.md) 형식 사용)

## 패키지 매니저별 오래된 패키지 탐지

각 패키지 매니저에 맞는 명령어로 오래된 의존성 목록을 수집합니다. 상세 분석 방법론은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

### Node.js (npm / yarn / pnpm)

**탐지 명령어**:
```bash
# npm
npm outdated --json 2>/dev/null

# yarn (Classic)
yarn outdated --json 2>/dev/null

# yarn (Berry / v2+)
yarn upgrade-interactive 2>/dev/null

# pnpm
pnpm outdated --json 2>/dev/null
```

**package.json 의존성 추출**:
```bash
# dependencies + devDependencies 전체 목록
cat package.json | jq '{
  dependencies: .dependencies // {},
  devDependencies: .devDependencies // {}
}' 2>/dev/null

# lock 파일 기반 정확한 설치 버전 확인
cat package-lock.json | jq '.packages | to_entries[] | select(.key != "") | {name: .key, version: .value.version}' 2>/dev/null
```

**결과 분류 기준**:

| 필드 | 설명 |
|------|------|
| `current` | 현재 설치된 버전 |
| `wanted` | semver 범위 내 최신 버전 (minor/patch) |
| `latest` | 레지스트리의 최신 버전 |

- `current == wanted == latest`: 최신 상태
- `current < wanted`: patch 또는 minor 업데이트 가능
- `wanted < latest`: major 업데이트 가능

### Python (pip / pipenv / poetry)

**탐지 명령어**:
```bash
# pip
pip list --outdated --format=json 2>/dev/null

# pipenv
pipenv update --dry-run 2>/dev/null

# poetry
poetry show --outdated 2>/dev/null
```

**requirements.txt 파싱**:
```regex
# 버전 고정 패턴
^([a-zA-Z0-9_-]+)==(\d+\.\d+\.\d+)
# 범위 지정 패턴
^([a-zA-Z0-9_-]+)([><=!~]+.*)
# 버전 미지정 (항상 최신)
^([a-zA-Z0-9_-]+)\s*$
```

**검증 항목**:
- `requirements.txt`에서 버전 고정 여부 확인
- `Pipfile.lock` 또는 `poetry.lock` 존재 여부 확인
- 가상 환경 내 실제 설치 버전과 명세 비교

### Java (Maven)

**탐지 명령어**:
```bash
# Maven Versions Plugin
mvn versions:display-dependency-updates 2>/dev/null

# 의존성 트리 확인
mvn dependency:tree 2>/dev/null

# 사용하지 않는 의존성 탐지
mvn dependency:analyze 2>/dev/null
```

**pom.xml 의존성 추출**:
```bash
# groupId, artifactId, version 추출
grep -A3 '<dependency>' pom.xml 2>/dev/null | grep -E '<(groupId|artifactId|version)>'
```

### Java/Kotlin (Gradle)

**탐지 명령어**:
```bash
# Gradle Versions Plugin (ben-manes)
./gradlew dependencyUpdates -Drevision=release 2>/dev/null

# 의존성 목록 확인
./gradlew dependencies --configuration runtimeClasspath 2>/dev/null
```

### Go (go modules)

**탐지 명령어**:
```bash
# 직접 의존성 업데이트 확인
go list -m -u all 2>/dev/null

# go.mod 파일에서 의존성 추출
grep -E '^(require|replace)\s' go.mod 2>/dev/null

# 간접 의존성 포함 전체 목록
go mod graph 2>/dev/null
```

### Rust (Cargo)

**탐지 명령어**:
```bash
# 오래된 크레이트 확인
cargo outdated 2>/dev/null

# 의존성 트리 확인
cargo tree 2>/dev/null

# 보안 감사
cargo audit 2>/dev/null
```

### PHP (Composer)

**탐지 명령어**:
```bash
# 오래된 패키지 확인
composer outdated --direct --format=json 2>/dev/null

# 의존성 목록
composer show --format=json 2>/dev/null
```

### Ruby (Bundler)

**탐지 명령어**:
```bash
# 오래된 gem 확인
bundle outdated --strict 2>/dev/null

# 보안 감사
bundle audit check --update 2>/dev/null
```

## 버전 분류 체계

오래된 패키지를 SemVer(Semantic Versioning) 기준으로 분류합니다.

### SemVer 분류 규칙

```
MAJOR.MINOR.PATCH (예: 2.4.1)
```

| 분류 | 조건 | 위험도 | 설명 |
|------|------|--------|------|
| Patch | `MAJOR` 동일, `MINOR` 동일, `PATCH` 차이 | 낮음 | 버그 수정만 포함, 하위 호환 보장 |
| Minor | `MAJOR` 동일, `MINOR` 차이 | 보통 | 기능 추가, 하위 호환 보장 |
| Major | `MAJOR` 차이 | 높음 | Breaking Change 포함 가능 |

### 버전 비교 정규식

```regex
# SemVer 파싱
^v?(\d+)\.(\d+)\.(\d+)(?:-([a-zA-Z0-9.]+))?(?:\+([a-zA-Z0-9.]+))?$

# pre-release 버전 감지
^v?\d+\.\d+\.\d+-(alpha|beta|rc|dev|canary|next|preview)
```

### 분류 알고리즘

```
입력: current_version, latest_version
1. SemVer 파싱 → (major, minor, patch)
2. major 비교:
   - major 다름 → "Major" 분류
3. minor 비교:
   - minor 다름 → "Minor" 분류
4. patch 비교:
   - patch 다름 → "Patch" 분류
5. 모두 같음 → "Up-to-date"
```

## 메이저 업그레이드 영향 분석

Major 버전 변경이 감지된 패키지에 대해 Breaking Change 영향 분석을 수행합니다.

### 분석 절차

1. **CHANGELOG / Release Notes 확인**: 패키지의 공식 변경 이력을 확인합니다
2. **Breaking Change 목록 추출**: 삭제된 API, 변경된 인터페이스, 동작 변경 사항을 식별합니다
3. **프로젝트 내 사용 패턴 분석**: 실제 코드에서 해당 패키지를 어떻게 사용하는지 확인합니다
4. **영향 범위 산정**: 영향받는 파일 수와 코드 라인 수를 산정합니다
5. **마이그레이션 가이드 생성**: 단계별 업그레이드 절차와 코드 변환 예시를 제공합니다

### 프로젝트 내 사용 패턴 분석

```bash
# Node.js - 특정 패키지의 import/require 사용 위치 확인
grep -rn "require\s*(\s*['\"]PACKAGE_NAME['\"])" --include="*.js" --include="*.ts" --include="*.mjs" .
grep -rn "from\s*['\"]PACKAGE_NAME" --include="*.js" --include="*.ts" --include="*.mjs" .

# Python - 특정 패키지의 import 사용 위치 확인
grep -rn "^import\s\+PACKAGE_NAME\|^from\s\+PACKAGE_NAME" --include="*.py" .

# Java - 특정 패키지의 import 사용 위치 확인
grep -rn "^import\s\+GROUP_ID\.ARTIFACT_ID" --include="*.java" .

# Go - 특정 모듈의 import 사용 위치 확인
grep -rn "\"MODULE_PATH\"" --include="*.go" .
```

### Breaking Change 영향도 평가

| 영향도 | 기준 | 조치 |
|--------|------|------|
| Critical | 10개 이상 파일에서 삭제된 API 사용 | 전용 마이그레이션 브랜치 생성 권장 |
| High | 5-9개 파일에서 변경된 API 사용 | 충분한 테스트 후 업그레이드 |
| Medium | 1-4개 파일에서 변경된 API 사용 | 일반 업그레이드 진행 |
| Low | 간접 의존성만 영향 또는 API 변경 미사용 | 바로 업그레이드 가능 |

### 주요 패키지 마이그레이션 패턴

#### React 주요 버전 마이그레이션

```regex
# React 사용 패턴 탐지
(React\.createClass|componentWillMount|componentWillReceiveProps|componentWillUpdate)
# React 18 → 19 주의 패턴
(ReactDOM\.render\s*\(|ReactDOM\.hydrate\s*\()
# Deprecated lifecycle 메서드
(UNSAFE_componentWillMount|UNSAFE_componentWillReceiveProps|UNSAFE_componentWillUpdate)
```

#### Express.js 주요 버전 마이그레이션

```regex
# Express 4 → 5 주의 패턴
(app\.del\s*\(|req\.param\s*\(|res\.json\s*\(\s*status|res\.send\s*\(\s*status|app\.param\s*\(\s*fn\))
```

#### Django 주요 버전 마이그레이션

```regex
# Django 3.x → 4.x 주의 패턴
(django\.conf\.urls\.url|default_app_config|USE_L10N|PASSWORD_RESET_TIMEOUT_DAYS)
# Django 4.x → 5.x 주의 패턴
(logout\(\s*\)|index_together|django\.utils\.timezone\.utc)
```

## 라이선스 호환성 검증

프로젝트의 의존성에 포함된 라이선스의 호환성을 검증합니다.

### 라이선스 분류

| 카테고리 | 라이선스 | 제약 수준 |
|---------|---------|----------|
| Permissive (허용적) | MIT, BSD-2-Clause, BSD-3-Clause, ISC, Apache-2.0, Unlicense, 0BSD | 낮음 |
| Weak Copyleft (약한 카피레프트) | LGPL-2.1, LGPL-3.0, MPL-2.0, EPL-2.0, CDDL-1.0 | 보통 |
| Strong Copyleft (강한 카피레프트) | GPL-2.0, GPL-3.0, AGPL-3.0 | 높음 |
| Non-commercial / Proprietary | CC-BY-NC, SSPL, BSL | 매우 높음 |

### 호환성 매트릭스

| 프로젝트 라이선스 \ 의존성 라이선스 | MIT | Apache-2.0 | LGPL-3.0 | GPL-3.0 | AGPL-3.0 |
|-----------------------------------|-----|-----------|----------|---------|----------|
| MIT | OK | OK | 조건부 | 불가 | 불가 |
| Apache-2.0 | OK | OK | 조건부 | 불가 | 불가 |
| LGPL-3.0 | OK | OK | OK | 불가 | 불가 |
| GPL-3.0 | OK | OK | OK | OK | 불가 |
| AGPL-3.0 | OK | OK | OK | OK | OK |

**"조건부"** 의미: 동적 링크 시 허용, 정적 링크 시 제한

### 라이선스 탐지 방법

```bash
# Node.js - license-checker 사용
npx license-checker --json --production 2>/dev/null

# Node.js - 수동 확인
cat node_modules/*/package.json | jq '{name: .name, version: .version, license: .license}' 2>/dev/null

# Python - pip-licenses 사용
pip-licenses --format=json --with-urls 2>/dev/null

# Go - go-licenses 사용
go-licenses csv ./... 2>/dev/null

# Rust - cargo-license 사용
cargo license --json 2>/dev/null

# PHP - composer licenses
composer licenses --format=json 2>/dev/null

# Ruby - license_finder 사용
license_finder report --format=json 2>/dev/null
```

### 라이선스 위반 탐지 정규식

```regex
# 라이선스 파일에서 GPL 계열 탐지
(GNU\s+General\s+Public\s+License|GPL-[23]\.0|AGPL-3\.0)

# SSPL (Server Side Public License) 탐지
(Server\s+Side\s+Public\s+License|SSPL)

# 라이선스 미명시 패키지 탐지 (package.json)
"license"\s*:\s*("UNLICENSED"|"SEE LICENSE"|"NONE"|"")
```

### 라이선스 충돌 알림 기준

| 상황 | 심각도 | 설명 |
|------|--------|------|
| GPL 의존성 + MIT 프로젝트 | Critical | 프로젝트 전체가 GPL로 전환 필요 |
| AGPL 의존성 + 상용 서비스 | Critical | 서비스 소스 코드 공개 의무 발생 |
| 라이선스 미명시 패키지 | High | 법적 리스크 불확실 |
| LGPL + 정적 링크 | Medium | 동적 링크로 전환 권장 |
| Deprecated 라이선스 | Low | 최신 라이선스 버전으로 갱신 권장 |

## 우선순위 기반 업그레이드 플랜 생성

분석 결과를 종합하여 우선순위가 부여된 업그레이드 플랜을 생성합니다.

### 우선순위 결정 공식

```
최종 우선순위 = 보안 점수 + 호환성 점수 + 영향도 점수 + 최신성 점수

보안 점수:
  Critical CVE 존재: +100
  High CVE 존재: +70
  Medium CVE 존재: +40
  CVE 없음: +0

호환성 점수:
  라이선스 충돌(Critical): +80
  라이선스 충돌(High): +50
  호환 문제 없음: +0

영향도 점수 (Breaking Change):
  Patch 업데이트: +10
  Minor 업데이트: +20
  Major 업데이트 (Low 영향): +30
  Major 업데이트 (Medium 영향): +40
  Major 업데이트 (High 영향): +50
  Major 업데이트 (Critical 영향): +60

최신성 점수:
  3년 이상 미업데이트: +30
  1-3년 미업데이트: +20
  6개월-1년 미업데이트: +10
  6개월 이내: +0
```

### 우선순위 등급

| 등급 | 점수 범위 | 권장 대응 시간 | 설명 |
|------|----------|-------------|------|
| P0 (긴급) | 150 이상 | 즉시 (24시간 내) | Critical CVE 또는 심각한 라이선스 충돌 |
| P1 (높음) | 100-149 | 1주일 내 | High CVE 또는 주요 호환성 문제 |
| P2 (보통) | 50-99 | 1개월 내 | Minor 업데이트 또는 Medium CVE |
| P3 (낮음) | 0-49 | 다음 스프린트 | Patch 업데이트, 기능 개선 |

### 업그레이드 실행 순서

```
1. P0 (긴급) - CVE Critical 패키지
   ├─ Patch 업데이트로 해결 가능 → 즉시 적용
   └─ Major 업데이트 필요 → 마이그레이션 브랜치 생성

2. P1 (높음) - CVE High 또는 라이선스 충돌
   ├─ 대안 패키지 존재 → 패키지 교체 검토
   └─ 대안 없음 → 영향 분석 후 업그레이드

3. P2 (보통) - Minor 업데이트 일괄 적용
   ├─ 테스트 실행 후 적용
   └─ 실패 시 개별 업그레이드로 전환

4. P3 (낮음) - Patch 업데이트 일괄 적용
   └─ 자동 업데이트 가능 (Dependabot / Renovate)
```

## Security Scanner 연동

security-scanner 스킬이 탐지한 CVE 정보를 활용하여 실행 가능한 업그레이드 경로를 제안합니다.

### CVE → 업그레이드 경로 매핑

```
1. security-scanner 결과에서 의존성 관련 CVE 추출
   ├─ npm audit --json → advisories 추출
   ├─ pip audit --format=json → vulnerabilities 추출
   └─ cargo audit → advisories 추출

2. 각 CVE에 대해:
   ├─ 수정된 버전 확인
   ├─ 현재 버전에서 수정 버전까지의 업그레이드 경로 산출
   ├─ 경로상 Breaking Change 존재 여부 확인
   └─ 최소 영향 업그레이드 경로 선택

3. 결과:
   ├─ Patch로 해결 가능 → "npm install PACKAGE@VERSION"
   ├─ Minor로 해결 가능 → "npm install PACKAGE@VERSION" + 테스트
   └─ Major 필요 → 마이그레이션 가이드 생성
```

### 의존성 감사 결과 통합

```bash
# Node.js - npm audit 결과에서 업그레이드 경로 추출
npm audit --json 2>/dev/null | jq '.vulnerabilities | to_entries[] | {
  package: .key,
  severity: .value.severity,
  via: [.value.via[] | select(type == "object") | .title],
  fixAvailable: .value.fixAvailable,
  range: .value.range
}'

# Python - pip audit 결과에서 업그레이드 경로 추출
pip audit --format=json 2>/dev/null | jq '.dependencies[] | select(.vulns | length > 0) | {
  name: .name,
  version: .version,
  vulns: [.vulns[] | {id: .id, fix_versions: .fix_versions}]
}'
```

## Test Coverage Analyzer 연동

업그레이드 대상 패키지에 대한 테스트 커버리지를 확인하여 안전한 업그레이드를 지원합니다.

### 연동 워크플로우

```
1. 업그레이드 대상 패키지 목록 확정

2. 각 패키지별 사용 파일 식별:
   ├─ import/require 문 검색
   └─ 영향받는 소스 파일 목록 생성

3. test-coverage-analyzer로 해당 파일의 테스트 커버리지 확인:
   ├─ 테스트 파일 존재 여부
   ├─ 함수별 커버리지 비율
   └─ Edge case 테스트 유무

4. 커버리지 부족 시:
   ├─ 업그레이드 전 테스트 보강 권장
   └─ 테스트 스켈레톤 생성 제안

5. 충분한 커버리지 확인 후 업그레이드 진행 권장
```

## 트리거 조건

이 스킬은 다음 상황에서 활성화됩니다:

- 사용자가 "의존성 업데이트", "의존성 분석", "의존성 확인" 요청 시
- "패키지 업그레이드", "패키지 업데이트", "패키지 확인" 요청 시
- "마이그레이션 가이드", "업그레이드 가이드" 요청 시
- "라이선스 확인", "라이선스 호환성 검사" 요청 시
- "outdated", "dependency update", "package upgrade" 영문 요청 시
- "업그레이드 플랜 생성", "업그레이드 계획" 요청 시
- "CVE 기반 업그레이드", "보안 패치" 요청 시

## 에러 처리

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | 매니페스트 파일 미존재 | 파일 탐지 실패 | 지원되는 매니페스트 파일 목록 안내 |
| 2 | 패키지 매니저 미설치 | 명령어 실행 실패 | 설치 방법 안내 후 수동 분석 제공 |
| 3 | Lock 파일 미존재 | 파일 탐지 실패 | 매니페스트 기반 분석으로 전환, lock 파일 생성 권장 |
| 4 | 네트워크 오류 (레지스트리 접근 불가) | API 호출 타임아웃 | 캐시된 정보 또는 로컬 분석만 수행 |
| 5 | Private 레지스트리 인증 실패 | 401/403 응답 | 인증 설정 안내, 공개 패키지만 분석 |
| 6 | 지원하지 않는 패키지 매니저 | 매니페스트 미매칭 | [references/analysis-guide.md](references/analysis-guide.md)의 수동 분석 가이드 안내 |
| 7 | 의존성 수가 매우 많음 (>1000) | 패키지 수 카운트 | 직접 의존성(direct)만 우선 분석 |
| 8 | SemVer 미준수 패키지 | 버전 파싱 실패 | 해당 패키지는 수동 확인 안내 |
| 9 | Monorepo 프로젝트 | 다중 매니페스트 감지 | 워크스페이스 단위로 분석 분리 |

## 모범 사례

1. **Lock 파일 유지**: `package-lock.json`, `poetry.lock`, `go.sum` 등을 항상 버전 관리에 포함
2. **정기적 업데이트**: 주 1회 이상 `outdated` 명령으로 오래된 패키지 확인
3. **자동화 도구 활용**: Dependabot, Renovate Bot으로 자동 PR 생성
4. **단계별 업그레이드**: Major 업데이트는 한 번에 하나씩, 충분한 테스트 후 적용
5. **CVE 즉시 대응**: Critical/High CVE 발견 시 24~48시간 내 패치 적용
6. **라이선스 사전 검증**: 새 패키지 추가 시 반드시 라이선스 호환성 확인
7. **의존성 최소화**: 불필요한 의존성은 제거하여 공격 표면(attack surface) 축소
8. **테스트 선행**: 업그레이드 전 관련 코드의 테스트 커버리지 확인 및 보강

## 참조 문서

- [references/analysis-guide.md](references/analysis-guide.md) - 상세 분석 방법론, 버전 비교 알고리즘, 패키지 매니저별 심화 가이드
- [references/templates.md](references/templates.md) - 업그레이드 플랜 보고서, 마이그레이션 가이드, 라이선스 보고서 템플릿
