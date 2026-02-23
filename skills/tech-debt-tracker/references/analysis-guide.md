# Tech Debt Tracker 분석 가이드

이 문서는 기술 부채 분석의 상세 방법론, 언어별 부채 패턴 정규식, deprecated API 매핑 테이블, false positive 판단 기준, 부채 연령 산출 알고리즘을 정의합니다.

---

## 1. 부채 마커 상세 탐지 패턴

### 언어별 주석 구문

부채 마커는 언어별 주석 문법에 따라 탐지 패턴이 달라집니다.

```regex
# JavaScript / TypeScript / Java / Go / C / C++ (한 줄 주석)
//\s*(TODO|FIXME|HACK|XXX|TEMP|WORKAROUND|KLUDGE|OPTIMIZE|REFACTOR|REVIEW|BUG)\b[:\s]?(.*)

# JavaScript / TypeScript / Java / CSS (블록 주석)
/\*[\s\S]*?\b(TODO|FIXME|HACK|XXX|TEMP|WORKAROUND)\b[:\s]?(.*?)(\*/|$)

# Python / Ruby / Shell / YAML (해시 주석)
#\s*(TODO|FIXME|HACK|XXX|TEMP|WORKAROUND|KLUDGE|OPTIMIZE|REFACTOR|BUG)\b[:\s]?(.*)

# HTML / XML (주석)
<!--\s*(TODO|FIXME|HACK|XXX|TEMP|WORKAROUND)\b[:\s]?(.*?)-->

# SQL (주석)
--\s*(TODO|FIXME|HACK|XXX|TEMP|WORKAROUND)\b[:\s]?(.*)
```

### 구조화된 TODO 형식 인식

프로젝트에 따라 구조화된 TODO 형식을 사용합니다. 이를 파싱하여 추가 메타데이터를 추출합니다.

```regex
# 이슈 번호 포함 형식
(TODO|FIXME)\s*\(#(\d+)\)\s*[:\s]?(.*)

# 작성자 포함 형식
(TODO|FIXME)\s*\((\w+)\)\s*[:\s]?(.*)

# 기한 포함 형식
(TODO|FIXME)\s*\((\d{4}-\d{2}(-\d{2})?)\)\s*[:\s]?(.*)

# 복합 형식
(TODO|FIXME)\s*\((\w+),\s*#(\d+),\s*(\d{4}-\d{2})\)\s*[:\s]?(.*)
```

**구조화 정보 추출 예시**:

| 원본 | 마커 | 작성자 | 이슈 | 기한 | 설명 |
|------|------|--------|------|------|------|
| `// TODO(kim, #123, 2025-06): 인증 분리` | TODO | kim | #123 | 2025-06 | 인증 분리 |
| `// FIXME(#456): 메모리 누수` | FIXME | - | #456 | - | 메모리 누수 |
| `# TODO(park): 캐시 전략 개선` | TODO | park | - | - | 캐시 전략 개선 |

---

## 2. Lint 억제 주석 상세 분석

### TypeScript 억제 패턴

```regex
# @ts-ignore - 다음 라인의 모든 타입 오류 무시
//\s*@ts-ignore\s*(.*)

# @ts-expect-error - 다음 라인에서 오류가 예상되는 경우 (더 안전)
//\s*@ts-expect-error\s*(.*)

# @ts-nocheck - 파일 전체 타입 검사 비활성화
//\s*@ts-nocheck
```

**@ts-ignore vs @ts-expect-error**:

| 속성 | @ts-ignore | @ts-expect-error |
|------|-----------|-----------------|
| 오류 없을 때 | 경고 없음 | 사용되지 않는 지시문 경고 발생 |
| 권장 여부 | 비권장 (부채) | 조건부 허용 (명확한 사유 필요) |
| 심각도 | High | Medium |

### ESLint 억제 패턴

```regex
# 파일 전체 규칙 비활성화
/\*\s*eslint-disable\s+([\w\-/,\s]+)\s*\*/

# 다음 라인 규칙 비활성화
//\s*eslint-disable-next-line\s+([\w\-/,\s]+)

# 현재 라인 규칙 비활성화
//\s*eslint-disable-line\s+([\w\-/,\s]+)

# 모든 규칙 비활성화 (가장 위험)
/\*\s*eslint-disable\s*\*/
```

**억제된 규칙별 심각도 분류**:

| 억제된 규칙 | 심각도 | 설명 |
|------------|--------|------|
| @typescript-eslint/no-explicit-any | High | 타입 안전성 우회 |
| @typescript-eslint/ban-ts-comment | High | TS 억제 허용 |
| no-console | Low | 디버그 로그 (프로덕션 시 Medium) |
| no-unused-vars | Low | 미사용 변수 |
| react-hooks/exhaustive-deps | Medium | Hook 의존성 누락 |
| no-eval | Critical | 보안 위험 코드 허용 |
| 규칙 미지정 (전체 비활성화) | Critical | 모든 규칙 무시 |

### Python 억제 패턴

```regex
# noqa - flake8 모든 규칙 무시
#\s*noqa\s*$

# noqa 특정 규칙 - flake8 특정 규칙 무시
#\s*noqa:\s*([\w\d,\s]+)

# type: ignore - mypy 타입 검사 무시
#\s*type:\s*ignore(\[[\w\-,\s]+\])?

# pylint: disable - pylint 규칙 무시
#\s*pylint:\s*disable\s*=\s*([\w\-,\s]+)

# noinspection - PyCharm/IntelliJ 검사 무시
#\s*noinspection\s+(\w+)
```

---

## 3. Deprecated API 상세 매핑

### JavaScript/TypeScript 생태계

#### Node.js 내장 API

| deprecated API | 대체 API | Node.js 버전 |
|---------------|---------|-------------|
| `url.parse()` | `new URL()` | 11.0+ |
| `url.resolve()` | `new URL(target, base)` | 11.0+ |
| `querystring` | `URLSearchParams` | 12.0+ |
| `path.exists()` | `fs.existsSync()` | 1.0+ |
| `fs.exists()` | `fs.existsSync()` / `fs.access()` | 1.0+ |
| `Buffer()` 생성자 | `Buffer.from()`, `Buffer.alloc()` | 6.0+ |
| `domain` 모듈 | `AsyncLocalStorage` | - |
| `punycode` 모듈 | `punycode/` userland 패키지 | 7.0+ |
| `crypto.createCipher()` | `crypto.createCipheriv()` | 10.0+ |
| `new SlowBuffer()` | `Buffer.allocUnsafeSlow()` | 6.0+ |
| `process.binding()` | 공식 API 사용 | - |
| `util.isArray()` | `Array.isArray()` | 4.0+ |
| `util.isDate()` | `instanceof Date` | 4.0+ |
| `util.pump()` | `stream.pipeline()` | 10.0+ |

**탐지 정규식**:
```regex
# Node.js deprecated API
(url\.parse\s*\(|url\.resolve\s*\(|require\s*\(\s*['"]querystring['"]\)|new\s+Buffer\s*\(|require\s*\(\s*['"]domain['"]\)|crypto\.createCipher\s*\(|new\s+SlowBuffer\s*\(|util\.isArray\s*\(|util\.isDate\s*\(|util\.pump\s*\()
```

#### React 생태계 (상세)

```regex
# React deprecated Lifecycle 메서드
(componentWillMount\s*\(|UNSAFE_componentWillMount\s*\()
(componentWillReceiveProps\s*\(|UNSAFE_componentWillReceiveProps\s*\()
(componentWillUpdate\s*\(|UNSAFE_componentWillUpdate\s*\()

# React 18 deprecated API
(ReactDOM\.render\s*\(|ReactDOM\.hydrate\s*\(|ReactDOM\.unmountComponentAtNode\s*\()

# React deprecated 유틸
(React\.createClass|React\.PropTypes|React\.createFactory)

# React 19 deprecated (defaultProps on function components)
(function\s+\w+|const\s+\w+\s*=)[\s\S]*?\.defaultProps\s*=
```

#### Next.js 생태계

| deprecated API | 대체 API | 버전 |
|---------------|---------|------|
| `getInitialProps` | `getServerSideProps` / `getStaticProps` | 9.3+ |
| `next/image` layout prop | width/height/fill props | 13.0+ |
| `next/router` (pages) | `next/navigation` (app router) | 13.0+ |
| `next/head` | Metadata API | 13.0+ |
| `next/link` passHref | 자동 적용 | 13.0+ |
| `next.config.js` target | 자동 감지 | 12.0+ |

#### Express.js 생태계

```regex
# Express deprecated API
(app\.del\s*\()           # app.delete() 사용
(res\.sendfile\s*\()      # res.sendFile() 사용 (대소문자)
(res\.json\s*\(\s*\d+)    # res.status(code).json() 사용
(req\.param\s*\()         # req.params/req.query/req.body 사용
(app\.configure\s*\()     # 환경별 분기 직접 구현
(res\.redirect\s*\(\s*30[12]) # 명시적 status 사용
```

### Python 생태계 (상세)

#### Python 내장 함수

| deprecated 함수 | 대체 함수 | Python 버전 |
|----------------|----------|-----------|
| `print` 문 (Python 2) | `print()` 함수 | 3.0+ |
| `execfile()` | `exec(open().read())` | 3.0+ |
| `raw_input()` | `input()` | 3.0+ |
| `has_key()` | `in` 연산자 | 3.0+ |
| `dict.iteritems()` | `dict.items()` | 3.0+ |
| `reduce()` 내장 | `functools.reduce()` | 3.0+ |
| `reload()` 내장 | `importlib.reload()` | 3.4+ |
| `collections.MutableMapping` | `collections.abc.MutableMapping` | 3.3+ |

#### Django deprecated API

```regex
# Django deprecated 패턴
(from\s+django\.conf\.urls\s+import\s+url\b)          # path() 사용
(django\.utils\.encoding\.force_text)                   # force_str() 사용
(django\.utils\.encoding\.smart_text)                   # smart_str() 사용
(django\.utils\.translation\.ugettext)                  # gettext() 사용
(from\s+django\.shortcuts\s+import\s+render_to_response) # render() 사용
(django\.conf\.urls\.defaults)                          # django.conf.urls 사용
```

#### Flask deprecated API

```regex
# Flask deprecated 패턴
(flask\.ext\.)                    # flask_extension 직접 import
(from\s+flask\s+import.*Markup)   # markupsafe.Markup 사용
(app\.before_first_request)       # app 초기화 패턴 사용
```

### Java 생태계 (상세)

#### Java 내장 API

| deprecated API | 대체 API | Java 버전 |
|---------------|---------|---------|
| `Thread.stop()` | `Thread.interrupt()` 패턴 | 1.2+ |
| `Thread.suspend()` / `resume()` | `wait()` / `notify()` 패턴 | 1.2+ |
| `Runtime.exec(String)` | `ProcessBuilder` | 5+ |
| `Date` 대부분의 메서드 | `java.time` 패키지 | 8+ |
| `Calendar` | `java.time` 패키지 | 8+ |
| `StringTokenizer` | `String.split()` 또는 `Pattern` | 1.4+ |
| `Vector` | `ArrayList` + `Collections.synchronizedList()` | 1.2+ |
| `Hashtable` | `HashMap` + `ConcurrentHashMap` | 1.2+ |
| `Stack` | `Deque` (ArrayDeque) | 1.6+ |
| `Enumeration` | `Iterator` | 1.2+ |

#### Spring Framework deprecated API

```regex
# Spring deprecated 패턴
(WebSecurityConfigurerAdapter)           # SecurityFilterChain 빈 사용
(antMatchers\s*\(|mvcMatchers\s*\()      # requestMatchers() 사용
(authorizeRequests\s*\()                 # authorizeHttpRequests() 사용
(and\s*\(\s*\)\s*\.csrf)                 # SecurityFilterChain 체인 패턴
(@EnableGlobalMethodSecurity)            # @EnableMethodSecurity 사용
```

---

## 4. False Positive 판단 기준

### 부채 마커 오탐 판정

#### 제외 조건

| 조건 | 설명 | 예시 |
|------|------|------|
| 라이브러리 코드 | node_modules, vendor 등 외부 코드 | `node_modules/lodash/...` |
| 생성 파일 | dist, build, .next, __pycache__ | `dist/bundle.js` |
| Lock 파일 | package-lock.json, yarn.lock | lock 파일 내 TODO |
| 문서 파일 | .md, .txt, .rst (분석 가이드 등) | README의 TODO 섹션 설명 |
| 주석 내 URL | URL에 포함된 키워드 | `https://github.com/fixme/repo` |
| 변수/함수명 | 이름에 포함된 키워드 | `const todoList = []`, `function fixFormatter()` |
| 테스트 데이터 | 테스트 fixture 내 문자열 | `const input = "TODO: test"` |

**제외 경로 정규식**:
```regex
# FP 가능성 높은 경로 패턴
(node_modules|vendor|dist|build|\.next|__pycache__|\.venv|\.git|coverage|\.nyc_output|\.cache)
```

**변수/함수명 내 키워드 판별**:
```regex
# 변수명에 TODO/FIXME가 포함된 경우 (FP)
(const|let|var|function|class|def|type|interface)\s+\w*(todo|fixme|hack)\w*
(TODO|FIXME)_?(LIST|ITEMS|COUNT|FLAG|STATUS|FILTER|SORT)
```

### Deprecated API 오탐 판정

| 조건 | 설명 | 예시 |
|------|------|------|
| 호환성 레이어 | deprecated API를 래핑하여 마이그레이션 중 | `function legacyRender() { ReactDOM.render(...) }` |
| 폴리필 코드 | 구버전 호환을 위한 의도적 사용 | `if (!Buffer.from) { Buffer = ...}` |
| 주석 설명 | deprecated 사용 사유가 명시된 경우 | `// Using ReactDOM.render for React 17 compat` |
| 테스트 코드 | 테스트에서 deprecated API를 테스트하는 경우 | `it('should support legacy render', ...)` |

### any 타입 오탐 판정

| 조건 | 설명 | 예시 |
|------|------|------|
| 외부 타입 정의 | .d.ts 파일 내 any | `declare module 'legacy' { export function parse(input: any): void }` |
| 제네릭 제약 | extends any 또는 제네릭 폴백 | `function wrap<T = any>(value: T)` |
| catch 블록 | catch (error: any) | TypeScript 4.x 호환 |
| JSON.parse 결과 | 파싱 결과의 타입 단언 전 | `const data: any = JSON.parse(text)` |

---

## 5. 부채 연령 산출 알고리즘

### Git Blame 기반 연령 계산

```bash
# 1. 부채 마커가 포함된 라인 번호 추출
grep -n "TODO\|FIXME\|HACK\|XXX\|TEMP\|WORKAROUND" <FILE_PATH> | cut -d: -f1

# 2. 각 라인에 대한 git blame 실행 (porcelain 형식)
git blame --line-porcelain -L <LINE>,<LINE> <FILE_PATH>

# 3. author-time 추출 (Unix timestamp)
# 출력에서 "author-time <timestamp>" 라인 파싱

# 4. 연령 계산
# 현재_timestamp - author_time = 경과_초
# 경과_초 / 86400 = 경과_일
```

### 벌크 분석 스크립트

```bash
#!/bin/bash
# 프로젝트 내 모든 부채 마커의 연령을 일괄 분석

echo "파일|라인|마커|작성자|경과일|내용"
echo "---|---|---|---|---|---"

find . -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.py" -o -name "*.java" \) \
    -not -path "*/node_modules/*" -not -path "*/dist/*" -not -path "*/.git/*" \
    -print0 | while IFS= read -r -d '' file; do

    grep -n "TODO\|FIXME\|HACK\|XXX" "$file" 2>/dev/null | while IFS=: read -r lineno content; do
        blame_info=$(git blame --line-porcelain -L "${lineno},${lineno}" "$file" 2>/dev/null)
        author=$(echo "$blame_info" | grep "^author " | sed 's/^author //')
        author_time=$(echo "$blame_info" | grep "^author-time " | sed 's/^author-time //')

        if [ -n "$author_time" ]; then
            now=$(date +%s)
            age_days=$(( (now - author_time) / 86400 ))
            marker=$(echo "$content" | grep -oE "(TODO|FIXME|HACK|XXX)" | head -1)
            desc=$(echo "$content" | sed 's/.*\(TODO\|FIXME\|HACK\|XXX\)[: ]*//')
            echo "${file}|${lineno}|${marker}|${author}|${age_days}|${desc}"
        fi
    done
done
```

### 트렌드 분석 알고리즘

부채 증감 추이를 월별로 추적합니다.

```bash
# 월별 부채 마커 추가/제거 추이
for i in $(seq 0 11); do
    month_start=$(date -d "${i} months ago" +%Y-%m-01 2>/dev/null || gdate -d "${i} months ago" +%Y-%m-01)
    month_end=$(date -d "$((i-1)) months ago" +%Y-%m-01 2>/dev/null || gdate -d "$((i-1)) months ago" +%Y-%m-01)

    # 해당 월에 추가된 부채 마커
    added=$(git log --after="${month_start}" --before="${month_end}" -p --all 2>/dev/null \
        | grep -c "^\+.*\(TODO\|FIXME\|HACK\|XXX\|TEMP\|WORKAROUND\)" || echo 0)

    # 해당 월에 제거된 부채 마커
    removed=$(git log --after="${month_start}" --before="${month_end}" -p --all 2>/dev/null \
        | grep -c "^\-.*\(TODO\|FIXME\|HACK\|XXX\|TEMP\|WORKAROUND\)" || echo 0)

    net=$((added - removed))
    trend="="
    if [ "$net" -gt 0 ]; then trend="+${net} (증가)"; fi
    if [ "$net" -lt 0 ]; then trend="${net} (감소)"; fi
    if [ "$net" -eq 0 ]; then trend="0 (유지)"; fi

    echo "${month_start}: 추가 ${added}, 제거 ${removed}, 순변동 ${trend}"
done
```

**트렌드 판정 기준**:

| 패턴 | 판정 | 권장 조치 |
|------|------|----------|
| 3개월 연속 순증가 | 부채 가속 | 즉시 부채 해소 스프린트 편성 |
| 1~2개월 순증가 | 부채 증가 | 다음 스프린트에 부채 해소 작업 포함 |
| 증감 반복 | 부채 유지 | 부채 예산(10~20%) 준수 확인 |
| 3개월 연속 순감소 | 부채 감소 | 현재 전략 유지 |

---

## 6. 건강도 점수 산출

### 종합 건강도 계산 공식

5개 영역의 점수를 가중 평균하여 종합 건강도를 산출합니다.

```
종합 건강도 = (부채_마커_점수 x 0.20) + (타입_안전성_점수 x 0.25) +
             (deprecated_점수 x 0.15) + (테스트_부채_점수 x 0.25) +
             (의존성_부채_점수 x 0.15)
```

**영역별 점수 산출**:

#### 부채 마커 점수 (100점 만점)
```
점수 = max(0, 100 - (TODO_count x 0.5) - (FIXME_count x 1.0) - (HACK_count x 1.5) - (XXX_count x 2.0) - (stale_90d_count x 1.0))
```

#### 타입 안전성 점수 (100점 만점)
```
점수 = max(0, 100 - (any_count x 0.3) - (ts_ignore_count x 1.0) - (ts_nocheck_count x 5.0))
```

#### Deprecated 점수 (100점 만점)
```
점수 = max(0, 100 - (deprecated_api_count x 2.0) - (eol_runtime x 20.0) - (deprecated_pkg_count x 3.0))
```

#### 테스트 부채 점수 (100점 만점)
```
점수 = (test_file_ratio / target_ratio) x 100   # target_ratio = 0.8 (80%)
```

#### 의존성 부채 점수 (100점 만점)
```
점수 = max(0, 100 - (critical_cve x 20) - (high_cve x 10) - (major_outdated x 3) - (deprecated_pkg x 5))
```

### 종합 등급 기준

| 점수 구간 | 등급 | 판정 | 권장 조치 |
|----------|------|------|----------|
| 90~100 | A | 건강함 | 현재 수준 유지 |
| 75~89 | B | 양호 | 점진적 개선 |
| 60~74 | C | 주의 | 부채 해소 계획 수립 |
| 40~59 | D | 경고 | 즉시 부채 해소 스프린트 편성 |
| 0~39 | F | 위험 | 전사 차원 대응 필요 |

---

## 7. 분석 실행 순서

기술 부채 분석 시 다음 순서로 실행합니다:

### 1단계: 프로젝트 유형 감지

```bash
# 프로젝트 유형 및 언어 감지
[ -f "package.json" ] && echo "Node.js"
[ -f "tsconfig.json" ] && echo "TypeScript"
[ -f "requirements.txt" ] || [ -f "pyproject.toml" ] && echo "Python"
[ -f "pom.xml" ] || [ -f "build.gradle" ] && echo "Java"
[ -f "go.mod" ] && echo "Go"
[ -f "Gemfile" ] && echo "Ruby"
```

### 2단계: 부채 마커 수집 (최우선)

모든 부채 마커를 수집하고 유형별로 분류합니다.

### 3단계: Git blame 연령 분석

수집된 마커에 대해 생성일과 작성자를 추적합니다.

### 4단계: Lint 억제 주석 수집

@ts-ignore, eslint-disable, noqa 등을 수집합니다.

### 5단계: Deprecated API 스캔

언어/프레임워크별 deprecated API 사용을 감지합니다.

### 6단계: 타입 안전성 분석

any 사용, 미타이핑 함수, raw types를 분석합니다.

### 7단계: 코드 건강도 메트릭 수집

테스트 비율, 문서화 비율, 의존성 현황, 보안 취약점을 수집합니다.

### 8단계: Impact x Effort 분류

각 부채 항목을 매트릭스에 배치하고 P1~P4 우선순위를 부여합니다.

### 9단계: 건강도 점수 산출

종합 건강도 점수와 등급을 계산합니다.

### 10단계: 보고서 및 로드맵 생성

[references/templates.md](../templates.md)의 형식으로 보고서를 생성하고, 스프린트 단위 로드맵을 제안합니다.
