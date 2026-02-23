# Dependency Manager 분석 가이드

이 문서는 의존성 분석의 상세 방법론, 버전 비교 알고리즘, 패키지 매니저별 심화 가이드, 라이선스 호환성 판단 기준, 우선순위 산정 로직, 마이그레이션 영향 분석 절차를 정의합니다.

---

## 1. 패키지 매니저 자동 감지 방법

### 감지 우선순위

프로젝트 루트에서 다음 순서로 매니페스트 파일을 탐색합니다. 복수의 매니페스트가 존재할 경우 모두 분석합니다.

```bash
# 매니페스트 파일 탐색 (우선순위 순)
MANIFEST_FILES=(
    "package.json"           # Node.js (npm/yarn/pnpm)
    "requirements.txt"       # Python (pip)
    "Pipfile"                # Python (pipenv)
    "pyproject.toml"         # Python (poetry / PEP 621)
    "pom.xml"                # Java (Maven)
    "build.gradle"           # Java/Kotlin (Gradle - Groovy)
    "build.gradle.kts"       # Java/Kotlin (Gradle - Kotlin DSL)
    "go.mod"                 # Go (go modules)
    "Cargo.toml"             # Rust (Cargo)
    "composer.json"          # PHP (Composer)
    "Gemfile"                # Ruby (Bundler)
)

for file in "${MANIFEST_FILES[@]}"; do
    if [ -f "$file" ]; then
        echo "감지됨: $file"
    fi
done
```

### Node.js 패키지 매니저 구분

```bash
# npm vs yarn vs pnpm 구분
if [ -f "pnpm-lock.yaml" ]; then
    echo "pnpm"
elif [ -f "yarn.lock" ]; then
    # Yarn Classic vs Berry 구분
    if [ -f ".yarnrc.yml" ] || [ -d ".yarn" ]; then
        echo "yarn (Berry / v2+)"
    else
        echo "yarn (Classic / v1)"
    fi
elif [ -f "package-lock.json" ]; then
    echo "npm"
else
    echo "npm (lock 파일 없음 - 생성 권장)"
fi
```

### Python 패키지 매니저 구분

```bash
# pip vs pipenv vs poetry 구분
if [ -f "pyproject.toml" ]; then
    if grep -q '\[tool\.poetry\]' pyproject.toml 2>/dev/null; then
        echo "poetry"
    elif grep -q '\[project\]' pyproject.toml 2>/dev/null; then
        echo "PEP 621 (pip)"
    fi
elif [ -f "Pipfile" ]; then
    echo "pipenv"
elif [ -f "requirements.txt" ]; then
    echo "pip"
fi
```

### Monorepo 감지

```bash
# Monorepo 구조 감지
if [ -f "lerna.json" ] || [ -f "nx.json" ] || [ -f "turbo.json" ]; then
    echo "JavaScript/TypeScript Monorepo 감지"
fi

# package.json의 workspaces 필드 확인
if [ -f "package.json" ]; then
    WORKSPACES=$(cat package.json | jq '.workspaces' 2>/dev/null)
    if [ "$WORKSPACES" != "null" ] && [ -n "$WORKSPACES" ]; then
        echo "npm/yarn Workspaces 감지"
    fi
fi

# pnpm workspace 확인
if [ -f "pnpm-workspace.yaml" ]; then
    echo "pnpm Workspace 감지"
fi
```

---

## 2. 버전 비교 알고리즘

### SemVer 파싱

```regex
# 전체 SemVer 정규식 (RFC 기반)
^v?(?P<major>0|[1-9]\d*)\.(?P<minor>0|[1-9]\d*)\.(?P<patch>0|[1-9]\d*)(?:-(?P<prerelease>(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+(?P<buildmetadata>[0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$
```

### 버전 비교 의사 코드

```
함수 compare_versions(current, latest):
    c = parse_semver(current)
    l = parse_semver(latest)

    if c.major != l.major:
        return "Major", l.major - c.major, "Breaking Change 가능"
    elif c.minor != l.minor:
        return "Minor", l.minor - c.minor, "하위 호환 기능 추가"
    elif c.patch != l.patch:
        return "Patch", l.patch - c.patch, "버그 수정"
    else:
        if c.prerelease and not l.prerelease:
            return "Patch", 0, "안정 버전으로 전환"
        return "Up-to-date", 0, "최신 상태"
```

### Pre-release 버전 처리

```regex
# Pre-release 식별자 우선순위
# alpha < beta < rc < (release)
# 예: 2.0.0-alpha.1 < 2.0.0-beta.1 < 2.0.0-rc.1 < 2.0.0

^v?\d+\.\d+\.\d+-(alpha|beta|rc|dev|canary|next|preview|snapshot|nightly)
```

**처리 규칙**:
- Pre-release 버전은 `latest` 비교 대상에서 기본적으로 제외
- 사용자가 명시적으로 요청한 경우에만 pre-release 포함
- 현재 버전이 pre-release인 경우 안정 버전으로의 업그레이드를 우선 권장

### 비표준 버전 처리

| 패턴 | 예시 | 처리 방법 |
|------|------|----------|
| CalVer (Calendar Versioning) | `2024.01.15` | 날짜 기반 비교 |
| 단순 숫자 | `23`, `24` | 숫자 크기 비교 |
| 해시 기반 | `abc1234` | 비교 불가 - 수동 확인 안내 |
| 날짜 + 빌드 | `20240115.1` | CalVer로 처리 |
| 로컬 수정 | `1.0.0-custom` | pre-release로 처리 |

---

## 3. 패키지 매니저별 심화 분석 가이드

### Node.js (npm) 심화 분석

#### npm outdated 결과 파싱

```bash
# JSON 형식으로 오래된 패키지 목록 추출
npm outdated --json 2>/dev/null | jq 'to_entries[] | {
  name: .key,
  current: .value.current,
  wanted: .value.wanted,
  latest: .value.latest,
  type: .value.type,
  location: .value.location,
  dependent: .value.dependent
}'
```

#### 의존성 트리 분석

```bash
# 직접 의존성만 확인
npm ls --depth=0 --json 2>/dev/null

# 특정 패키지의 의존 관계 확인
npm ls PACKAGE_NAME --all --json 2>/dev/null

# 중복 설치된 패키지 확인
npm dedupe --dry-run 2>/dev/null

# 사용하지 않는 의존성 탐지
npx depcheck 2>/dev/null
```

#### package.json 버전 범위 해석

| 기호 | 의미 | 예시 | 허용 범위 |
|------|------|------|----------|
| `^` | Minor까지 허용 | `^1.2.3` | `>=1.2.3 <2.0.0` |
| `~` | Patch까지 허용 | `~1.2.3` | `>=1.2.3 <1.3.0` |
| `>=` | 이상 | `>=1.2.3` | `>=1.2.3` |
| `*` | 모든 버전 | `*` | 모든 버전 |
| 없음 | 정확한 버전 | `1.2.3` | `1.2.3` |
| `latest` | 최신 | `latest` | 최신 배포 버전 |

```regex
# package.json 버전 범위 파싱
"(?P<package>[^"]+)"\s*:\s*"(?P<range>[\^~>=<*]?\s*\d+(?:\.\d+)*(?:-[a-zA-Z0-9.]+)?)"
```

### Python (pip) 심화 분석

#### requirements.txt 파싱

```regex
# 고정 버전
^(?P<package>[a-zA-Z0-9_-]+)==(?P<version>\d+(?:\.\d+)*)

# 호환 릴리스
^(?P<package>[a-zA-Z0-9_-]+)~=(?P<version>\d+(?:\.\d+)*)

# 범위 지정
^(?P<package>[a-zA-Z0-9_-]+)(?P<constraint>[><=!]+\d+(?:\.\d+)*(?:\s*,\s*[><=!]+\d+(?:\.\d+)*)*)

# 추가 옵션 (extras)
^(?P<package>[a-zA-Z0-9_-]+)\[(?P<extras>[a-zA-Z0-9_,-]+)\](?P<constraint>.*)

# 주석 및 빈 줄 제외
^(?!#)(?!\s*$)
```

#### pyproject.toml 의존성 추출

```bash
# poetry 의존성 확인
grep -A100 '\[tool\.poetry\.dependencies\]' pyproject.toml 2>/dev/null | grep -B0 '^\[' | head -n -1

# PEP 621 의존성 확인
grep -A100 '^\[project\]' pyproject.toml 2>/dev/null | grep -A50 'dependencies\s*=' | head -50
```

### Go (go modules) 심화 분석

```bash
# 직접 의존성만 추출
grep -v '// indirect' go.mod | grep -E '^\t' 2>/dev/null

# 간접 의존성 확인
grep '// indirect' go.mod 2>/dev/null

# 업데이트 가능한 모듈 확인 (major 업데이트 포함)
go list -m -u -json all 2>/dev/null | jq 'select(.Update != null) | {
  Path: .Path,
  Version: .Version,
  Update: .Update.Version,
  Indirect: .Indirect
}'

# 모듈 그래프에서 특정 모듈의 의존 경로 확인
go mod why MODULE_PATH 2>/dev/null
```

### Rust (Cargo) 심화 분석

```bash
# 오래된 크레이트 확인 (cargo-outdated 필요)
cargo outdated --depth 1 2>/dev/null

# Cargo.toml 의존성 파싱
grep -A100 '\[dependencies\]' Cargo.toml 2>/dev/null | grep -B0 '^\[' | head -n -1

# 보안 감사 (cargo-audit 필요)
cargo audit --json 2>/dev/null

# 의존성 트리 확인
cargo tree --depth 1 2>/dev/null
```

---

## 4. 라이선스 호환성 상세 판단 기준

### 라이선스 식별 방법

```bash
# 방법 1: 매니페스트 파일에서 추출
# Node.js
cat package.json | jq '.license' 2>/dev/null

# Python (pyproject.toml)
grep -i 'license' pyproject.toml 2>/dev/null

# Rust
grep -A5 '\[package\]' Cargo.toml | grep 'license' 2>/dev/null

# 방법 2: LICENSE 파일 내용 분석
# MIT 라이선스 식별
grep -il "MIT License\|Permission is hereby granted, free of charge" LICENSE* 2>/dev/null

# Apache 2.0 식별
grep -il "Apache License.*Version 2\.0\|Licensed under the Apache License" LICENSE* 2>/dev/null

# GPL 계열 식별
grep -il "GNU General Public License\|GNU GENERAL PUBLIC LICENSE" LICENSE* 2>/dev/null

# 방법 3: SPDX 식별자 매칭
```

### 라이선스 식별 정규식

```regex
# MIT
(?i)(MIT\s+License|Permission\s+is\s+hereby\s+granted.*free\s+of\s+charge)

# Apache-2.0
(?i)(Apache\s+License.*Version\s+2\.0|Licensed\s+under\s+the\s+Apache\s+License)

# BSD-2-Clause
(?i)(BSD\s+2-Clause|Simplified\s+BSD\s+License|Redistribution\s+and\s+use.*2\s+conditions)

# BSD-3-Clause
(?i)(BSD\s+3-Clause|New\s+BSD\s+License|Redistribution\s+and\s+use.*3\s+conditions)

# GPL-2.0
(?i)(GNU\s+General\s+Public\s+License.*version\s+2|GPL-2\.0)

# GPL-3.0
(?i)(GNU\s+General\s+Public\s+License.*version\s+3|GPL-3\.0)

# LGPL-2.1
(?i)(GNU\s+Lesser\s+General\s+Public\s+License.*version\s+2\.1|LGPL-2\.1)

# LGPL-3.0
(?i)(GNU\s+Lesser\s+General\s+Public\s+License.*version\s+3|LGPL-3\.0)

# AGPL-3.0
(?i)(GNU\s+Affero\s+General\s+Public\s+License|AGPL-3\.0)

# ISC
(?i)(ISC\s+License|Permission\s+to\s+use.*ISC)

# MPL-2.0
(?i)(Mozilla\s+Public\s+License.*Version\s+2\.0|MPL-2\.0)

# Unlicense
(?i)(This\s+is\s+free\s+and\s+unencumbered\s+software|Unlicense)

# SSPL
(?i)(Server\s+Side\s+Public\s+License|SSPL)
```

### 상세 호환성 판단 규칙

**규칙 1: Permissive + Permissive = 항상 호환**
- MIT + Apache-2.0 = OK
- BSD + ISC = OK

**규칙 2: Permissive + Copyleft = 프로젝트 라이선스에 따라 다름**
- MIT 프로젝트 + GPL 의존성 = 프로젝트가 GPL로 전환되어야 함 (비호환)
- GPL 프로젝트 + MIT 의존성 = OK

**규칙 3: Weak Copyleft는 사용 방식에 따라 다름**
- LGPL: 동적 링크 시 허용, 정적 링크 시 소스 공개 의무
- MPL-2.0: 파일 단위 카피레프트, 다른 파일은 자유

**규칙 4: AGPL은 네트워크 사용도 소스 공개 트리거**
- SaaS / 웹 서비스에서 AGPL 의존성 사용 시 전체 소스 공개 의무

**규칙 5: 이중 라이선스(Dual License)**
- 복수 라이선스 중 선택 가능한 경우 호환 가능한 라이선스 선택
- "AND" 관계(동시 적용)와 "OR" 관계(선택 적용) 구분 필요

### 라이선스 위험도 등급

| 등급 | 라이선스 | 상용 프로젝트 영향 | 오픈소스 프로젝트 영향 |
|------|---------|------------------|---------------------|
| 안전 | MIT, BSD-2, BSD-3, ISC, Apache-2.0, Unlicense, 0BSD | 제한 없음 | 제한 없음 |
| 주의 | LGPL-2.1, LGPL-3.0, MPL-2.0, EPL-2.0 | 사용 방식 확인 필요 | 일반적으로 호환 |
| 위험 | GPL-2.0, GPL-3.0 | 소스 공개 의무 발생 | 라이선스 전환 필요 가능 |
| 매우 위험 | AGPL-3.0, SSPL | SaaS에서 소스 공개 의무 | 라이선스 전환 필요 가능 |
| 불확실 | UNLICENSED, NONE, SEE LICENSE | 법적 리스크 불확실 | 법적 리스크 불확실 |

---

## 5. 메이저 업그레이드 영향 분석 절차

### 단계 1: 프로젝트 내 사용 패턴 수집

```bash
# Node.js - 패키지의 모든 import/require 위치 탐색
grep -rn "require\s*(\s*['\"]PACKAGE['\"])" --include="*.js" --include="*.ts" --include="*.jsx" --include="*.tsx" --include="*.mjs" --include="*.cjs" . 2>/dev/null
grep -rn "from\s*['\"]PACKAGE" --include="*.js" --include="*.ts" --include="*.jsx" --include="*.tsx" --include="*.mjs" --include="*.cjs" . 2>/dev/null

# Python - 패키지의 모든 import 위치 탐색
grep -rn "^import\s\+PACKAGE\|^from\s\+PACKAGE" --include="*.py" . 2>/dev/null

# Go - 모듈의 모든 import 위치 탐색
grep -rn "\"MODULE_PATH" --include="*.go" . 2>/dev/null

# 사용 API/함수 추출 (Node.js 예시)
grep -rn "PACKAGE\.\w\+" --include="*.js" --include="*.ts" . 2>/dev/null | sed 's/.*PACKAGE\.\([a-zA-Z_]*\).*/\1/' | sort -u
```

### 단계 2: Breaking Change 매칭

주요 패키지의 알려진 Breaking Change 패턴을 코드와 매칭합니다.

#### React 18 → 19 Breaking Change 패턴

```regex
# 삭제/변경된 API
ReactDOM\.render\s*\(
ReactDOM\.hydrate\s*\(
ReactDOM\.unmountComponentAtNode\s*\(
React\.createFactory\s*\(
react-dom/test-utils
act\s*\(\s*\).*from\s*['"]react-dom/test-utils['"]

# Deprecated props
defaultProps\s*=\s*\{  # 함수 컴포넌트에서
propTypes\s*=\s*\{     # propTypes 지원 축소
```

#### Express 4 → 5 Breaking Change 패턴

```regex
# 삭제된 API
app\.del\s*\(
req\.param\s*\(
res\.json\s*\(\s*\d+
res\.send\s*\(\s*\d+\s*,
res\.sendfile\s*\(    # 소문자 f
app\.param\s*\(\s*function

# 변경된 동작
res\.redirect\s*\(\s*['"]back['"]
req\.host\b
```

#### Django 4.x → 5.x Breaking Change 패턴

```regex
# 삭제된 기능
index_together\s*=
django\.utils\.timezone\.utc\b
logout\s*\(\s*request\s*\)  # 인자 변경
USE_L10N\s*=
```

#### TypeScript 4 → 5 Breaking Change 패턴

```regex
# 변경된 컴파일러 옵션
"target"\s*:\s*"es3"
"moduleResolution"\s*:\s*"node"  # node → node10 명칭 변경
"importsNotUsedAsValues"
# 삭제된 API
(ts\.SymbolFlags\.EnumMember|ts\.createLanguageService)
```

### 단계 3: 영향 범위 산정

```
영향 범위 = {
  직접_영향_파일수: grep 결과에서 패키지 사용 파일 수,
  삭제_API_사용건수: Breaking Change 패턴 매칭 건수,
  간접_의존_패키지수: 해당 패키지를 의존하는 다른 패키지 수,
  테스트_파일_영향: 테스트 코드 중 영향받는 파일 수
}

영향도 등급:
  Critical: 직접_영향 >= 10 AND 삭제_API >= 5
  High:     직접_영향 >= 5 OR 삭제_API >= 3
  Medium:   직접_영향 >= 1 AND 삭제_API >= 1
  Low:      간접_의존만 영향 OR 삭제_API == 0
```

---

## 6. 우선순위 산정 로직 상세

### 점수 계산 상세

```
보안 점수 계산:
  for each CVE in package.cves:
    if CVE.cvss >= 9.0: score += 100  # Critical
    elif CVE.cvss >= 7.0: score += 70 # High
    elif CVE.cvss >= 4.0: score += 40 # Medium
    else: score += 10                 # Low
  score = min(score, 100)  # 최대 100점

호환성 점수 계산:
  if license_conflict == "Critical": score = 80
  elif license_conflict == "High": score = 50
  elif license_conflict == "Medium": score = 30
  else: score = 0

영향도 점수 계산:
  if update_type == "patch": score = 10
  elif update_type == "minor": score = 20
  elif update_type == "major":
    if impact == "Low": score = 30
    elif impact == "Medium": score = 40
    elif impact == "High": score = 50
    elif impact == "Critical": score = 60

최신성 점수 계산:
  days_since_update = today - last_update_date
  if days_since_update > 1095: score = 30  # 3년 이상
  elif days_since_update > 365: score = 20 # 1-3년
  elif days_since_update > 180: score = 10 # 6개월-1년
  else: score = 0
```

### 우선순위 그룹화

```
P0 그룹 (150+): 즉시 대응
  ├─ Critical CVE + 어떤 업데이트 유형이든
  ├─ High CVE + 라이선스 충돌
  └─ 라이선스 Critical 충돌 + Major 영향

P1 그룹 (100-149): 1주일 내
  ├─ High CVE 단독
  ├─ Medium CVE + Major 업데이트
  └─ 라이선스 High 충돌

P2 그룹 (50-99): 1개월 내
  ├─ Minor 업데이트 + Medium CVE
  ├─ Major 업데이트 (Low 영향)
  └─ 장기 미업데이트 (3년+) + Minor

P3 그룹 (0-49): 다음 스프린트
  ├─ Patch 업데이트 (CVE 없음)
  ├─ Minor 업데이트 (CVE 없음)
  └─ 최신 상태에 가까운 패키지
```

---

## 7. 사용하지 않는 의존성 탐지

### 탐지 방법

```bash
# Node.js - depcheck
npx depcheck --json 2>/dev/null

# Python - pip-autoremove (설치 필요)
pip-autoremove --list 2>/dev/null

# Go - 사용하지 않는 모듈 정리
go mod tidy -v 2>/dev/null

# Rust - 사용하지 않는 크레이트 탐지 (cargo-udeps 필요)
cargo +nightly udeps 2>/dev/null
```

### 의존성 정리 권장 사항

| 상황 | 권장 조치 |
|------|----------|
| `dependencies`에 있지만 코드에서 미사용 | `devDependencies`로 이동 또는 제거 |
| `devDependencies`에 있지만 스크립트에서 미사용 | 제거 |
| 간접 의존성이 직접 의존성에 중복 | `npm dedupe` 또는 수동 정리 |
| 기능이 중복되는 패키지 | 하나로 통합 (예: moment → dayjs) |
| 번들 크기가 큰 패키지 | 경량 대안 검토 (예: lodash → lodash-es 또는 개별 import) |

---

## 8. 분석 품질 체크리스트

### 분석 완료 전 검증 항목

- [ ] 프로젝트의 모든 매니페스트 파일을 식별했는가?
- [ ] Lock 파일이 존재하면 lock 파일 기반으로 정확한 버전을 확인했는가?
- [ ] 오래된 패키지를 patch/minor/major로 정확히 분류했는가?
- [ ] SemVer를 따르지 않는 패키지를 별도로 표시했는가?
- [ ] Pre-release 버전을 적절히 처리했는가?
- [ ] CVE 정보를 최신 데이터로 확인했는가?
- [ ] 라이선스 호환성을 프로젝트 라이선스 기준으로 검증했는가?
- [ ] 메이저 업그레이드 대상의 Breaking Change를 코드와 매칭했는가?
- [ ] 영향받는 파일 수와 코드 라인 수를 정확히 산정했는가?
- [ ] 우선순위 점수를 공식에 따라 일관되게 계산했는가?
- [ ] Monorepo인 경우 워크스페이스 단위로 분석을 분리했는가?
- [ ] 직접 의존성과 간접 의존성을 구분했는가?
- [ ] 사용하지 않는 의존성을 식별했는가?
- [ ] 보고서 형식이 [references/templates.md](templates.md)를 따르는가?
- [ ] security-scanner 및 test-coverage-analyzer 연동 데이터를 반영했는가?
