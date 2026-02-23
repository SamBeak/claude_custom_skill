---
name: tech-debt-tracker
description: 코드베이스에 축적된 기술 부채를 체계적으로 스캔, 분류, 추적하고 개선 로드맵을 생성하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) 기술 부채 분석, (2) TODO 정리, (3) 코드 건강도 점검, (4) 레거시 코드 분석, (5) tech debt 추적, (6) 코드 부채 현황, (7) deprecated 분석, (8) 타입 안전성 검사.
---

# Tech Debt Tracker

코드베이스에 축적된 기술 부채를 체계적으로 스캔하여 부채 마커, deprecated API 사용, 타입 안전성 갭, 코드 건강도를 종합 분석하고, Impact x Effort 매트릭스 기반의 개선 로드맵을 생성합니다.

## Quick Start

사용자가 기술 부채 분석을 요청하면 다음 워크플로우를 실행합니다:

1. **분석 범위 결정**:
   ```bash
   # 전체 코드베이스 스캔
   find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.tsx" -o -name "*.jsx" -o -name "*.py" -o -name "*.java" -o -name "*.go" -o -name "*.rb" \) \
       -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/dist/*" -not -path "*/build/*" -not -path "*/__pycache__/*" -not -path "*/.venv/*"

   # 또는 최근 변경 파일만 대상
   git diff --name-only HEAD~10
   ```

2. **부채 마커 스캔**: TODO, FIXME, HACK, XXX, TEMP, WORKAROUND, @deprecated, lint 억제 주석 수집

3. **Git blame 기반 부채 연령 분석**: 각 마커의 생성일, 작성자, 경과 일수 산출

4. **Deprecated API/라이브러리 사용 감지**: npm 패키지, Python stdlib, 프레임워크별 deprecated API 식별

5. **타입 안전성 갭 분석**: `any` 사용, 미타이핑 함수, lint 억제 카운트

6. **코드 건강도 메트릭 수집**: 테스트 부채, 문서 부채, 의존성 부채, 보안 부채

7. **Impact x Effort 매트릭스 생성 및 로드맵 출력**: [references/templates.md](references/templates.md) 형식 사용

## 1. 부채 마커 스캔

코드 내 기술 부채를 나타내는 주석 및 어노테이션을 수집합니다. 상세 분석 방법론은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

### 1.1 일반 부채 마커

코드 전반에서 공통적으로 사용되는 부채 표시 주석을 탐지합니다.

**탐지 패턴**:
```regex
# TODO 마커 (모든 언어 공통)
(?i)(//|#|/\*|\*|--|<!--|%)\s*(TODO|FIXME|HACK|XXX|TEMP|WORKAROUND|KLUDGE|OPTIMIZE|REFACTOR|REVIEW|NOTE|BUG)\b[:\s]?(.*)

# @deprecated 어노테이션 (Java, JSDoc, Python)
(@deprecated|@Deprecated|warnings\.warn\s*\(.*DeprecationWarning)

# Python deprecation 데코레이터
(deprecated|@deprecation\.deprecated)
```

**마커별 의미**:

| 마커 | 의미 | 기본 심각도 |
|------|------|-----------|
| TODO | 향후 구현 필요 | Medium |
| FIXME | 알려진 결함, 수정 필요 | High |
| HACK | 임시 우회 코드 | High |
| XXX | 심각한 문제, 즉시 대응 필요 | Critical |
| TEMP | 임시 코드, 제거 예정 | Medium |
| WORKAROUND | 우회 처리, 근본 해결 필요 | Medium |
| KLUDGE | 품질이 낮은 임시 해결책 | High |
| OPTIMIZE | 성능 개선 필요 지점 | Low |
| REFACTOR | 리팩토링 대상 | Medium |
| BUG | 알려진 버그 | High |

### 1.2 Lint 억제 주석

린터 또는 타입 검사기를 의도적으로 무시하는 주석을 수집합니다. 이는 기술 부채의 직접적 지표입니다.

**탐지 패턴**:
```regex
# TypeScript 억제
(@ts-ignore|@ts-expect-error|@ts-nocheck)

# ESLint 억제
(eslint-disable|eslint-disable-next-line|eslint-disable-line)(?:\s+(.+))?

# Prettier 억제
(prettier-ignore)

# Python 억제
(type:\s*ignore|# noqa|# noinspection|# pylint:\s*disable|# type:\s*ignore\[.+\])

# Java 억제
(@SuppressWarnings\s*\(\s*["'](.+)["']\))

# Go 억제
(//nolint|// nolint)

# Ruby 억제
(# rubocop:disable)
```

**수집 명령**:
```bash
# TypeScript/JavaScript 억제 주석 검색
grep -rn "@ts-ignore\|@ts-expect-error\|@ts-nocheck\|eslint-disable\|prettier-ignore" \
    --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" \
    --exclude-dir=node_modules --exclude-dir=dist .

# Python 억제 주석 검색
grep -rn "# type: ignore\|# noqa\|# noinspection\|# pylint: disable" \
    --include="*.py" --exclude-dir=__pycache__ --exclude-dir=.venv .

# Java 억제 어노테이션 검색
grep -rn "@SuppressWarnings" --include="*.java" .
```

### 1.3 마커 요약 집계

```bash
# 마커 유형별 카운트 집계
echo "=== 부채 마커 요약 ==="
for marker in TODO FIXME HACK XXX TEMP WORKAROUND BUG; do
    count=$(grep -rn "\b${marker}\b" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --include="*.py" --include="*.java" --exclude-dir=node_modules --exclude-dir=dist --exclude-dir=.git . 2>/dev/null | wc -l)
    echo "${marker}: ${count}건"
done

echo ""
echo "=== Lint 억제 주석 요약 ==="
ts_ignore=$(grep -rn "@ts-ignore\|@ts-expect-error" --include="*.ts" --include="*.tsx" --exclude-dir=node_modules . 2>/dev/null | wc -l)
eslint_disable=$(grep -rn "eslint-disable" --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx" --exclude-dir=node_modules . 2>/dev/null | wc -l)
noqa=$(grep -rn "# noqa" --include="*.py" --exclude-dir=__pycache__ . 2>/dev/null | wc -l)
echo "@ts-ignore/@ts-expect-error: ${ts_ignore}건"
echo "eslint-disable: ${eslint_disable}건"
echo "noqa: ${noqa}건"
```

## 2. Git Blame 기반 부채 연령 분석

각 부채 마커가 언제, 누구에 의해 생성되었는지 추적하여 부채의 연령과 소유자를 파악합니다.

### 2.1 부채 연령 산출

```bash
# 특정 파일의 TODO 라인에 대한 git blame 실행
git blame --line-porcelain <FILE_PATH> | grep -B10 "TODO\|FIXME\|HACK\|XXX" | grep "author-time\|author " | paste - -

# 부채 연령 계산 (현재 시간 - author-time)
# author-time은 Unix timestamp이므로 날짜로 변환
git blame --line-porcelain <FILE_PATH> | awk '/^author-time/ {t=$2} /^author / {a=$2} /TODO|FIXME|HACK|XXX/ {print a, t, $0}'
```

### 2.2 연령 기반 분류

| 연령 구간 | 분류 | 권장 조치 |
|----------|------|----------|
| 0~30일 | 신규 (Fresh) | 다음 스프린트에서 처리 |
| 31~90일 | 누적 중 (Aging) | 우선순위 검토 필요 |
| 91~180일 | 장기 방치 (Stale) | 즉시 처리 계획 수립 |
| 181~365일 | 만성 부채 (Chronic) | 팀 차원 해결 논의 |
| 365일 초과 | 화석 부채 (Fossil) | 여전히 유효한지 검증 필요 |

### 2.3 트렌드 분석

```bash
# 월별 TODO/FIXME 추가 추이 (최근 6개월)
for month in $(seq 0 5); do
    date_start=$(date -d "${month} months ago" +%Y-%m-01 2>/dev/null || date -v-${month}m +%Y-%m-01)
    date_end=$(date -d "$((month-1)) months ago" +%Y-%m-01 2>/dev/null || date -v-$((month-1))m +%Y-%m-01)
    added=$(git log --after="${date_start}" --before="${date_end}" --diff-filter=A -p | grep -c "^\+.*\(TODO\|FIXME\|HACK\)" || echo 0)
    removed=$(git log --after="${date_start}" --before="${date_end}" --diff-filter=D -p | grep -c "^\-.*\(TODO\|FIXME\|HACK\)" || echo 0)
    echo "${date_start}: +${added} / -${removed}"
done
```

**트렌드 판정 기준**:
- 추가 > 제거 (3개월 연속): 부채 증가 추세 - 즉시 개선 필요
- 추가 = 제거: 부채 유지 - 감소 전략 필요
- 추가 < 제거 (3개월 연속): 부채 감소 추세 - 유지

## 3. Deprecated API/라이브러리 사용 감지

### 3.1 Deprecated npm 패키지 탐지

```bash
# npm outdated로 오래된 패키지 확인
npm outdated --json 2>/dev/null

# npm deprecated 패키지 확인
npm ls --json 2>/dev/null | jq '[.dependencies | to_entries[] | select(.value.deprecated)] | length'

# 특정 deprecated 패키지 목록
npm ls --json 2>/dev/null | jq '.dependencies | to_entries[] | select(.value.deprecated) | {name: .key, deprecated: .value.deprecated}'
```

**자주 발견되는 deprecated npm 패키지**:

| 패키지 | 대체 패키지 | 비고 |
|--------|-----------|------|
| request | axios, node-fetch, got | 2020년 deprecated |
| moment | dayjs, date-fns, luxon | 유지보수 모드 |
| uuid/v1, uuid/v4 (이전 API) | uuid (v9+) | API 변경 |
| querystring | URLSearchParams | Node.js 내장 |
| chalk (v4 이하 CJS) | chalk v5+ (ESM) | ESM 전환 |
| @types/node (매우 오래된 버전) | 최신 @types/node | 주기적 업데이트 |

### 3.2 Python deprecated stdlib 탐지

```regex
# Python deprecated stdlib 사용
(import\s+distutils|from\s+distutils\s+import)
(import\s+imp\b|from\s+imp\s+import)
(import\s+optparse|from\s+optparse\s+import)
(import\s+formatter|from\s+formatter\s+import)
(import\s+parser|from\s+parser\s+import)
(import\s+symbol|from\s+symbol\s+import)
(import\s+asynchat|from\s+asynchat\s+import)
(import\s+asyncore|from\s+asyncore\s+import)
(import\s+smtpd|from\s+smtpd\s+import)
(import\s+cgi\b|from\s+cgi\s+import)
(import\s+cgitb|from\s+cgitb\s+import)
```

**Python deprecated stdlib 매핑**:

| deprecated 모듈 | 대체 모듈 | Python 버전 |
|----------------|----------|-----------|
| distutils | setuptools, build | 3.12에서 제거 |
| imp | importlib | 3.12에서 제거 |
| optparse | argparse | 3.2부터 deprecated |
| asynchat/asyncore | asyncio | 3.12에서 제거 |
| cgi/cgitb | 직접 구현 또는 웹 프레임워크 | 3.13에서 제거 |
| smtpd | aiosmtpd | 3.12에서 제거 |
| formatter | 직접 구현 | 3.10에서 제거 |

### 3.3 프레임워크별 Deprecated API 탐지

#### React

```regex
# React Class Component (함수 컴포넌트로 전환 권장)
(class\s+\w+\s+extends\s+(React\.)?Component)
(class\s+\w+\s+extends\s+(React\.)?PureComponent)

# Deprecated React API
(componentWillMount|componentWillReceiveProps|componentWillUpdate|UNSAFE_componentWillMount|UNSAFE_componentWillReceiveProps|UNSAFE_componentWillUpdate)
(ReactDOM\.render\s*\()
(React\.createClass)
(React\.PropTypes)
(defaultProps\s*=)
(findDOMNode\s*\()
```

| deprecated API | 대체 API | 비고 |
|---------------|---------|------|
| componentWillMount | useEffect / constructor | React 17+ |
| componentWillReceiveProps | getDerivedStateFromProps | React 17+ |
| ReactDOM.render() | createRoot().render() | React 18+ |
| React.createClass | class extends Component / 함수 컴포넌트 | React 16+ |
| defaultProps (함수 컴포넌트) | ES6 기본 매개변수 | React 19+ |
| findDOMNode | useRef | React 18+ |

#### Next.js

```regex
# Next.js deprecated API
(getInitialProps)
(next\/image.*layout\s*=)
(next\/router.*useRouter(?!.*from\s+['"]next\/navigation))
(next\/head)
```

#### Express.js

```regex
# Express deprecated API
(app\.del\s*\()
(res\.sendfile\s*\()
(res\.json\s*\(\s*status)
(req\.param\s*\()
```

### 3.4 EOL 런타임 버전 감지

```bash
# Node.js 버전 확인
node_version=$(node -v 2>/dev/null | sed 's/v//')
echo "Node.js 버전: ${node_version}"
# .nvmrc, .node-version, package.json engines 필드도 확인
cat .nvmrc 2>/dev/null || cat .node-version 2>/dev/null
cat package.json 2>/dev/null | jq '.engines.node' 2>/dev/null

# Python 버전 확인
python_version=$(python3 --version 2>/dev/null | awk '{print $2}')
echo "Python 버전: ${python_version}"
# pyproject.toml, setup.cfg의 python_requires 확인
grep -i "python_requires\|requires-python" pyproject.toml setup.cfg 2>/dev/null

# Java 버전 확인
java_version=$(java -version 2>&1 | head -1)
echo "Java 버전: ${java_version}"
```

**EOL 런타임 기준 (2025년 기준)**:

| 런타임 | EOL 버전 | 현행 LTS | 비고 |
|--------|---------|---------|------|
| Node.js | 16 이하 | 20, 22 | 짝수 버전만 LTS |
| Python | 3.8 이하 | 3.11, 3.12 | 5년 주기 |
| Java | 8 (상용), 11 | 17, 21 | LTS 주기 확인 |

## 4. 타입 안전성 갭 분석

### 4.1 TypeScript `any` 사용 분석

```regex
# 명시적 any 타입 사용
(:\s*any\b|<any>|as\s+any\b|:\s*any\[\]|:\s*Array<any>)

# 암시적 any 가능성 (noImplicitAny 미적용 시)
(function\s+\w+\s*\([^)]*\)\s*\{)  # 반환 타입 미지정 함수
```

```bash
# any 사용 위치 및 개수 집계
grep -rn ": any\b\|<any>\|as any\b" --include="*.ts" --include="*.tsx" --exclude-dir=node_modules --exclude-dir=dist . | wc -l

# 파일별 any 사용 빈도
grep -rl ": any\b\|<any>\|as any\b" --include="*.ts" --include="*.tsx" --exclude-dir=node_modules --exclude-dir=dist . | while read f; do
    count=$(grep -c ": any\b\|<any>\|as any\b" "$f")
    echo "${count} ${f}"
done | sort -rn | head -20
```

**any 사용 심각도 분류**:

| 사용 패턴 | 심각도 | 설명 |
|----------|--------|------|
| 함수 매개변수에 `any` | High | 타입 가드 부재, 런타임 오류 위험 |
| 함수 반환 타입에 `any` | High | 호출자가 타입 추론 불가 |
| 변수 선언에 `any` | Medium | 해당 스코프 내 타입 안전성 상실 |
| `as any` 타입 단언 | High | 타입 검사 강제 우회 |
| 제네릭 `<any>` | Medium | 타입 파라미터의 의미 상실 |
| 외부 라이브러리 타입 정의 내 `any` | Low | 타입 정의 개선 필요 |

### 4.2 @ts-ignore 억제 분석

```bash
# @ts-ignore 사용 위치와 사유 수집
grep -rn "@ts-ignore\|@ts-expect-error" --include="*.ts" --include="*.tsx" --exclude-dir=node_modules . | while read line; do
    file=$(echo "$line" | cut -d: -f1)
    lineno=$(echo "$line" | cut -d: -f2)
    # 해당 라인과 다음 라인(억제 대상) 함께 출력
    echo "--- ${file}:${lineno} ---"
    sed -n "${lineno},$((lineno+1))p" "$file"
done
```

### 4.3 Python 타입 힌트 누락 분석

```regex
# public 함수에 타입 힌트 없는 매개변수
(def\s+[a-z]\w+\s*\([^)]*[a-zA-Z_]\w*\s*[,)])  # : type 없는 매개변수

# 반환 타입 미지정 public 함수
(def\s+[a-z]\w+\s*\([^)]*\)\s*:)  # -> ReturnType 없음
```

```bash
# Python 타입 힌트 커버리지 추정
total_funcs=$(grep -rn "def \w\+(" --include="*.py" --exclude-dir=__pycache__ --exclude-dir=.venv . 2>/dev/null | wc -l)
typed_funcs=$(grep -rn "def \w\+(.*:" --include="*.py" --exclude-dir=__pycache__ --exclude-dir=.venv . 2>/dev/null | grep "\->" | wc -l)
echo "전체 함수: ${total_funcs}, 타입 힌트 적용: ${typed_funcs}"
if [ "$total_funcs" -gt 0 ]; then
    coverage=$((typed_funcs * 100 / total_funcs))
    echo "타입 힌트 커버리지: ${coverage}%"
fi
```

### 4.4 Java Raw Types 사용 분석

```regex
# Raw type 사용 (제네릭 미적용)
(List\s+\w+\s*=|Map\s+\w+\s*=|Set\s+\w+\s*=|Collection\s+\w+\s*=)(?!.*<)
(new\s+ArrayList\s*\(\)|new\s+HashMap\s*\(\)|new\s+HashSet\s*\(\))(?!.*<)

# @SuppressWarnings("unchecked") 사용
(@SuppressWarnings\s*\(\s*["']unchecked["']\))
(@SuppressWarnings\s*\(\s*\{[^}]*["']unchecked["'][^}]*\}\))
```

## 5. 코드 건강도 메트릭

### 5.1 테스트 부채

테스트가 부족하거나 누락된 영역을 식별합니다.

```bash
# 소스 파일 대비 테스트 파일 비율
src_count=$(find . -type f \( -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.java" \) \
    -not -path "*/node_modules/*" -not -path "*/test*" -not -path "*/__test*" -not -path "*/spec/*" \
    -not -path "*/dist/*" -not -name "*.test.*" -not -name "*.spec.*" -not -name "test_*" | wc -l)
test_count=$(find . -type f \( -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" -o -path "*/test/*" -o -path "*/__tests__/*" \) \
    -not -path "*/node_modules/*" | wc -l)
echo "소스 파일: ${src_count}, 테스트 파일: ${test_count}"
if [ "$src_count" -gt 0 ]; then
    ratio=$((test_count * 100 / src_count))
    echo "테스트 파일 비율: ${ratio}%"
fi
```

**테스트 부채 평가 기준**:

| 테스트 파일 비율 | 등급 | 판정 |
|----------------|------|------|
| 80% 이상 | A | 양호 |
| 60~79% | B | 보통 |
| 40~59% | C | 부채 누적 |
| 20~39% | D | 심각한 부채 |
| 20% 미만 | F | 즉시 개선 필요 |

### 5.2 문서 부채

공개 API에 대한 문서화 현황을 분석합니다.

```regex
# JSDoc 미적용 export 함수 (TypeScript/JavaScript)
(export\s+(default\s+)?function\s+\w+)(?<!\/\*\*[\s\S]*?\*\/\s*export)

# Python docstring 미적용 public 함수
(def\s+[a-z]\w+\s*\([^)]*\)\s*(?:->.*)?:\s*\n\s*(?!"""|'''))

# Java Javadoc 미적용 public 메서드
(public\s+\w+\s+\w+\s*\([^)]*\))(?<!\/\*\*[\s\S]*?\*\/\s*public)
```

### 5.3 의존성 부채

```bash
# Node.js: 오래된 패키지 확인
npm outdated --json 2>/dev/null | jq 'to_entries | length'

# Python: 오래된 패키지 확인
pip list --outdated --format=json 2>/dev/null | jq 'length'

# 메이저 버전 차이가 있는 패키지 (높은 부채)
npm outdated --json 2>/dev/null | jq '[to_entries[] | select(.value.current != .value.latest) | select((.value.current | split(".")[0]) != (.value.latest | split(".")[0]))] | length'
```

**의존성 부채 분류**:

| 차이 | 심각도 | 설명 |
|------|--------|------|
| 메이저 버전 차이 | High | Breaking change 가능, 마이그레이션 필요 |
| 마이너 버전 차이 | Medium | 기능 추가/개선, 호환성 유지 |
| 패치 버전 차이 | Low | 버그 수정, 안정성 개선 |
| deprecated 패키지 | High | 대체 패키지로 마이그레이션 필요 |
| CVE 보유 패키지 | Critical | 보안 취약점, 즉시 업데이트 |

### 5.4 보안 부채

패치되지 않은 보안 취약점을 집계합니다.

```bash
# npm audit 결과에서 심각도별 카운트
npm audit --json 2>/dev/null | jq '{
    critical: .metadata.vulnerabilities.critical,
    high: .metadata.vulnerabilities.high,
    moderate: .metadata.vulnerabilities.moderate,
    low: .metadata.vulnerabilities.low
}'

# pip audit 결과
pip audit --format=json 2>/dev/null | jq '.vulnerabilities | length'
```

## 6. Impact x Effort 매트릭스 및 개선 로드맵

### 6.1 분류 기준

각 부채 항목을 **영향도(Impact)**와 **해결 노력(Effort)** 두 축으로 분류합니다.

**영향도 (Impact) 기준**:

| 등급 | 기준 | 예시 |
|------|------|------|
| High | 사용자 경험에 직접 영향, 장애 가능성, 보안 위험 | 미패치 CVE, 타입 안전성 부재로 인한 런타임 오류 |
| Low | 개발 생산성에 영향, 유지보수 비용 증가 | 오래된 TODO, 문서 미비, 코드 스타일 불일치 |

**해결 노력 (Effort) 기준**:

| 등급 | 기준 | 예시 |
|------|------|------|
| High | 1주일 이상, 다수 파일 변경, 호환성 검증 필요 | 프레임워크 마이그레이션, 대규모 리팩토링 |
| Low | 1~2일 이내, 제한된 파일 변경, 단순 대체 | TODO 해결, deprecated API 단순 대체, 타입 추가 |

### 6.2 우선순위 매트릭스

```
                  High Impact
                      |
         P1           |          P2
    (즉시 처리)       |      (계획 수립)
    High Impact,      |    High Impact,
    Low Effort        |    High Effort
                      |
  -------- Low Effort-+--------- High Effort ------
                      |
         P3           |          P4
    (틈새 시간 활용)  |    (장기 검토)
    Low Impact,       |    Low Impact,
    Low Effort        |    High Effort
                      |
                  Low Impact
```

| 우선순위 | 조합 | 권장 시기 | 예시 |
|---------|------|----------|------|
| P1 | High Impact + Low Effort | 현재 스프린트 | deprecated API 단순 대체, CVE 패치 |
| P2 | High Impact + High Effort | 다음 스프린트 계획 | 프레임워크 마이그레이션, 대규모 타입 추가 |
| P3 | Low Impact + Low Effort | 일상 작업 중 처리 | 오래된 TODO 정리, 주석 개선 |
| P4 | Low Impact + High Effort | 분기 계획 시 검토 | 레거시 코드 전면 재작성 |

### 6.3 로드맵 생성

분석 결과를 바탕으로 스프린트 단위 개선 로드맵을 생성합니다. 상세 템플릿은 [references/templates.md](references/templates.md)를 참조하세요.

```markdown
## 기술 부채 개선 로드맵

### 현재 스프린트 (P1 - 즉시 처리)
- [ ] [Critical] CVE-XXXX-XXXX 패치: {패키지}를 {버전}으로 업데이트
- [ ] [High] @ts-ignore 제거: {파일}:{라인} - 적절한 타입 정의로 대체
- [ ] [High] deprecated API 대체: ReactDOM.render -> createRoot

### 다음 스프린트 (P2 - 계획 수립)
- [ ] [High] TypeScript strict 모드 단계적 적용 (any 200건 -> 100건)
- [ ] [High] 테스트 커버리지 60% -> 80% 달성

### 지속 개선 (P3 - 틈새 시간)
- [ ] [Medium] TODO 정리: 90일 이상 방치된 TODO 30건 처리
- [ ] [Low] 문서화: 공개 API 중 미문서화 함수 20건 추가

### 분기 계획 (P4 - 장기 검토)
- [ ] [Medium] moment.js -> dayjs 마이그레이션
- [ ] [Low] 레거시 Class Component -> 함수 컴포넌트 전환 (50개 파일)
```

## 7. 스킬 연계 (Synergy)

Tech Debt Tracker는 다른 분석 스킬과 가장 높은 연계성을 가집니다. 각 스킬이 생성하는 결과를 기술 부채 관점에서 통합 분석합니다.

### 연계 스킬 매트릭스

| 연계 스킬 | 부채 영역 | 연계 방법 |
|----------|----------|----------|
| refactor-advisor | 코드 품질 부채 | 복잡도, 코드 스멜 결과를 부채로 분류 |
| test-coverage-analyzer | 테스트 부채 | 커버리지 갭을 테스트 부채로 집계 |
| security-scanner | 보안 부채 | 미패치 CVE, 취약 패턴을 보안 부채로 분류 |
| pr-review-checklist | 부채 유입 방지 | PR 리뷰 시 신규 부채 마커 감지 |

### 연계 워크플로우

1. **종합 코드 건강도 분석** 요청 시:
   - Tech Debt Tracker: 부채 마커 + deprecated + 타입 갭 분석
   - refactor-advisor: 복잡도 + 코드 스멜 분석
   - test-coverage-analyzer: 테스트 누락 분석
   - security-scanner: 보안 취약점 분석
   - 결과를 통합하여 단일 건강도 리포트 생성

2. **부채 유입 방지** (PR 리뷰 시):
   - 신규 TODO/FIXME 추가 시 경고
   - 신규 @ts-ignore 추가 시 경고
   - any 사용 증가 시 경고

## 트리거 조건

이 스킬은 다음 상황에서 활성화됩니다:

- "기술 부채 분석", "기술 부채 현황 파악" 요청 시
- "TODO 정리", "TODO/FIXME 현황" 요청 시
- "코드 건강도", "코드 건강도 점검" 요청 시
- "레거시 코드 분석", "레거시 마이그레이션" 요청 시
- "tech debt", "tech debt tracking" 영문 요청 시
- "코드 부채", "코드 부채 현황" 요청 시
- "deprecated 분석", "deprecated API 확인" 요청 시
- "타입 안전성", "타입 안전성 검사" 요청 시

## 에러 처리

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | Git 저장소가 아닌 경우 | `git status` 실패 | blame 분석 건너뛰고 마커 스캔만 수행 |
| 2 | 지원하지 않는 언어 | 파일 확장자 미매칭 | 일반 부채 마커(TODO/FIXME)만 탐지 |
| 3 | npm/pip 미설치 | 패키지 매니저 명령 실패 | 의존성 분석 건너뛰고 수동 확인 안내 |
| 4 | 대규모 코드베이스 (파일 10,000+) | 파일 수 확인 | 최근 변경 파일 또는 주요 디렉토리만 스캔 |
| 5 | git blame 타임아웃 | 실행 시간 초과 | 샘플링 방식으로 전환 (상위 100개 파일만) |
| 6 | package.json 미존재 | 파일 존재 여부 확인 | Node.js 의존성 분석 건너뛰기 |
| 7 | tsconfig.json strict 모드 미확인 | 파일 파싱 실패 | 기본 패턴 매칭으로 대체 |
| 8 | 바이너리/생성 파일 포함 | 파일 유형 검사 | dist, build, node_modules 등 자동 제외 |

## 모범 사례

1. **정기 스캔**: 주 1회 이상 전체 코드베이스 부채 스캔으로 추이 모니터링
2. **부채 예산 설정**: 스프린트 시간의 10~20%를 부채 해소에 할당
3. **신규 부채 관리**: PR 리뷰 시 신규 TODO/FIXME에 이슈 번호 또는 기한 명시 (예: `// TODO(#123): 2025-06 인증 로직 분리`)
4. **Zero Tolerance 정책**: Critical/High 심각도 부채는 2주 이내 해소 원칙
5. **부채 시각화**: 대시보드에 부채 추이 차트를 추가하여 팀 전체 인식 공유
6. **자동화**: pre-commit hook으로 기한 없는 TODO 추가를 경고 또는 차단
7. **레트로스펙티브 연계**: 스프린트 회고 시 부채 증감 추이를 함께 리뷰

## 참조 문서

- [references/analysis-guide.md](references/analysis-guide.md) - 상세 분석 방법론, 언어별 패턴 정의, false positive 판단 기준
- [references/templates.md](references/templates.md) - 부채 리포트 출력 형식, 로드맵 템플릿, pre-commit hook 설정
