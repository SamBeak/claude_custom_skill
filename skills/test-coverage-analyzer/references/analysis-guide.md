# 테스트 커버리지 분석 가이드

테스트 커버리지를 분석하고 누락된 테스트 케이스를 도출하기 위한 상세 방법론 가이드입니다.

## 목차

1. [소스-테스트 파일 매핑 규칙](#소스-테스트-파일-매핑-규칙)
2. [커버리지 도구별 실행 및 파싱](#커버리지-도구별-실행-및-파싱)
3. [테스트 케이스 도출 기법](#테스트-케이스-도출-기법)
4. [모킹 전략 가이드](#모킹-전략-가이드)
5. [테스트 품질 체크리스트](#테스트-품질-체크리스트)

---

## 소스-테스트 파일 매핑 규칙

### 매핑 프로세스 개요

```
1. 프로젝트 구조 탐지
   ├─ 테스트 디렉토리 위치 확인
   ├─ 기존 테스트 파일 패턴 분석
   └─ 프레임워크 설정 파일 확인

2. 매핑 규칙 결정
   ├─ 기존 프로젝트 패턴 우선
   ├─ 언어별 기본 규칙 적용
   └─ 커스텀 매핑 파일 확인

3. 매핑 실행
   ├─ 소스 파일 경로 → 테스트 파일 경로 변환
   ├─ 다중 후보가 있으면 우선순위에 따라 선택
   └─ 매핑 결과 검증 (실제 존재 여부)
```

### JavaScript / TypeScript 상세 규칙

#### 디렉토리 구조 패턴

**패턴 1: Co-located (`__tests__` 하위)**
```
src/
├── services/
│   ├── userService.ts
│   └── __tests__/
│       └── userService.test.ts
├── utils/
│   ├── format.ts
│   └── __tests__/
│       └── format.test.ts
```

**패턴 2: 별도 test 디렉토리 (미러링)**
```
src/
├── services/
│   └── userService.ts
├── utils/
│   └── format.ts
test/
├── services/
│   └── userService.test.ts
├── utils/
│   └── format.test.ts
```

**패턴 3: 평면(flat) test 디렉토리**
```
src/
├── services/
│   └── userService.ts
tests/
├── userService.test.ts
├── format.test.ts
```

#### 탐지 명령어

```bash
# 테스트 디렉토리 구조 탐지
find . -type d -name "__tests__" | head -5
find . -type d -name "test" -maxdepth 2
find . -type d -name "tests" -maxdepth 2

# 테스트 파일 확장자 패턴 확인
find . -name "*.test.ts" -o -name "*.test.tsx" -o -name "*.spec.ts" | head -10

# Jest 설정에서 testMatch 확인
grep -A 5 "testMatch\|testPathPattern\|roots" package.json jest.config.* 2>/dev/null
```

#### 변환 규칙

| 소스 경로 | 변환 규칙 | 테스트 경로 |
|-----------|---------|-----------|
| `src/foo/bar.ts` | 디렉토리 내 `__tests__` | `src/foo/__tests__/bar.test.ts` |
| `src/foo/bar.ts` | `src/` → `test/` | `test/foo/bar.test.ts` |
| `src/foo/bar.tsx` | 디렉토리 내 `__tests__` | `src/foo/__tests__/bar.test.tsx` |
| `lib/helper.js` | `lib/` → `test/` | `test/helper.test.js` |
| `src/index.ts` | 디렉토리 내 `__tests__` | `src/__tests__/index.test.ts` |

### Python 상세 규칙

#### 디렉토리 구조 패턴

**패턴 1: 별도 tests 디렉토리 (미러링)**
```
src/
├── services/
│   └── auth.py
├── utils/
│   └── crypto.py
tests/
├── services/
│   └── test_auth.py
├── utils/
│   └── test_crypto.py
├── conftest.py
```

**패턴 2: 평면(flat) tests 디렉토리**
```
src/
├── services/
│   └── auth.py
tests/
├── test_auth.py
├── test_crypto.py
├── conftest.py
```

**패턴 3: 패키지 내부 테스트**
```
mypackage/
├── services/
│   ├── auth.py
│   └── tests/
│       └── test_auth.py
```

#### 탐지 명령어

```bash
# pytest 설정에서 testpaths 확인
grep -A 3 "testpaths\|python_files\|python_classes" pytest.ini pyproject.toml setup.cfg 2>/dev/null

# 테스트 파일 패턴 확인
find . -name "test_*.py" -o -name "*_test.py" | head -10

# conftest.py 위치 확인
find . -name "conftest.py" | head -5
```

#### 변환 규칙

| 소스 경로 | 변환 규칙 | 테스트 경로 |
|-----------|---------|-----------|
| `src/services/auth.py` | `src/` → `tests/`, `test_` 접두사 | `tests/services/test_auth.py` |
| `src/utils/crypto.py` | `src/` → `tests/`, `test_` 접두사 | `tests/utils/test_crypto.py` |
| `app/models/user.py` | `app/` → `tests/`, `test_` 접두사 | `tests/models/test_user.py` |
| `pkg/helper.py` | `pkg/` → `tests/`, `test_` 접두사 | `tests/test_helper.py` |

### Java 상세 규칙

#### 디렉토리 구조 (Maven/Gradle 표준)

```
src/
├── main/
│   └── java/
│       └── com/app/
│           ├── service/
│           │   └── UserService.java
│           └── controller/
│               └── UserController.java
├── test/
│   └── java/
│       └── com/app/
│           ├── service/
│           │   └── UserServiceTest.java
│           └── controller/
│               └── UserControllerTest.java
```

#### 변환 규칙

| 소스 경로 | 변환 규칙 | 테스트 경로 |
|-----------|---------|-----------|
| `src/main/java/com/app/Foo.java` | `main` → `test`, `Test` 접미사 | `src/test/java/com/app/FooTest.java` |
| `src/main/java/com/app/service/Bar.java` | `main` → `test`, `Test` 접미사 | `src/test/java/com/app/service/BarTest.java` |

### Go 상세 규칙

#### 디렉토리 구조 (Go 표준)

```
pkg/
├── handler/
│   ├── user.go
│   └── user_test.go
internal/
├── service/
│   ├── auth.go
│   └── auth_test.go
```

#### 변환 규칙

| 소스 경로 | 변환 규칙 | 테스트 경로 |
|-----------|---------|-----------|
| `pkg/handler/user.go` | 동일 디렉토리, `_test.go` 접미사 | `pkg/handler/user_test.go` |
| `internal/service/auth.go` | 동일 디렉토리, `_test.go` 접미사 | `internal/service/auth_test.go` |
| `cmd/server/main.go` | 동일 디렉토리, `_test.go` 접미사 | `cmd/server/main_test.go` |

---

## 커버리지 도구별 실행 및 파싱

### Jest (JavaScript/TypeScript)

#### 실행 명령어

```bash
# 기본 커버리지 실행 (JSON summary)
npx jest --coverage --coverageReporters=json-summary --silent

# 특정 파일만 커버리지 실행
npx jest --coverage --collectCoverageFrom='src/services/userService.ts' --silent

# 변경된 파일만 테스트 실행
npx jest --changedSince=main --coverage --silent
```

#### 출력 파싱 (coverage-summary.json)

```json
{
  "total": {
    "lines": { "total": 200, "covered": 160, "skipped": 0, "pct": 80 },
    "statements": { "total": 220, "covered": 176, "skipped": 0, "pct": 80 },
    "functions": { "total": 40, "covered": 35, "skipped": 0, "pct": 87.5 },
    "branches": { "total": 60, "covered": 42, "skipped": 0, "pct": 70 }
  },
  "src/services/userService.ts": {
    "lines": { "total": 50, "covered": 35, "skipped": 0, "pct": 70 },
    "statements": { "total": 55, "covered": 38, "skipped": 0, "pct": 69.09 },
    "functions": { "total": 8, "covered": 5, "skipped": 0, "pct": 62.5 },
    "branches": { "total": 12, "covered": 7, "skipped": 0, "pct": 58.33 }
  }
}
```

**파싱 포인트**:
- `pct` 값으로 각 항목의 커버리지 퍼센트 추출
- 파일별 섹션에서 개별 파일 커버리지 확인
- `functions.covered` / `functions.total`로 함수 커버리지 계산
- `branches.pct`로 분기 커버리지 확인

### pytest-cov (Python)

#### 실행 명령어

```bash
# 기본 커버리지 실행 (JSON)
python -m pytest --cov=src --cov-report=json --quiet

# 특정 모듈만 커버리지
python -m pytest --cov=src/services --cov-report=json --quiet

# 분기 커버리지 포함
python -m pytest --cov=src --cov-branch --cov-report=json --quiet

# 터미널 출력 + JSON
python -m pytest --cov=src --cov-report=term-missing --cov-report=json --quiet
```

#### 출력 파싱 (coverage.json)

```json
{
  "meta": {
    "version": "7.4.0",
    "timestamp": "2026-02-23T14:30:00"
  },
  "totals": {
    "covered_lines": 160,
    "num_statements": 200,
    "percent_covered": 80.0,
    "missing_lines": 40,
    "num_branches": 60,
    "num_partial_branches": 8,
    "covered_branches": 42,
    "percent_covered_branches": 70.0
  },
  "files": {
    "src/services/auth.py": {
      "summary": {
        "covered_lines": 35,
        "num_statements": 50,
        "percent_covered": 70.0,
        "missing_lines": 15
      },
      "missing_lines": [23, 24, 25, 45, 46, 67, 68, 69, 70, 89, 90, 91, 102, 103, 104],
      "executed_lines": [1, 2, 3, 5, 6, 7, 10, 11, 12]
    }
  }
}
```

**파싱 포인트**:
- `totals.percent_covered`로 전체 커버리지 확인
- `files` 섹션에서 파일별 커버리지 확인
- `missing_lines`로 테스트되지 않은 라인 식별 (핵심!)
- `percent_covered_branches`로 분기 커버리지 확인

### JaCoCo (Java)

#### 실행 명령어

```bash
# Maven
mvn clean test jacoco:report -q

# Gradle
./gradlew clean test jacocoTestReport

# 보고서 위치
# Maven: target/site/jacoco/jacoco.csv
# Gradle: build/reports/jacoco/test/jacocoTestReport.csv
```

#### 출력 파싱 (jacoco.csv)

```csv
GROUP,PACKAGE,CLASS,INSTRUCTION_MISSED,INSTRUCTION_COVERED,BRANCH_MISSED,BRANCH_COVERED,LINE_MISSED,LINE_COVERED,COMPLEXITY_MISSED,COMPLEXITY_COVERED,METHOD_MISSED,METHOD_COVERED
my-app,com.app.service,UserService,45,120,8,18,12,35,5,12,2,8
my-app,com.app.controller,UserController,30,90,5,15,8,28,3,10,1,7
```

**파싱 포인트**:
- `LINE_COVERED / (LINE_MISSED + LINE_COVERED)` = 라인 커버리지
- `BRANCH_COVERED / (BRANCH_MISSED + BRANCH_COVERED)` = 분기 커버리지
- `METHOD_COVERED / (METHOD_MISSED + METHOD_COVERED)` = 메서드 커버리지
- 클래스별 데이터 제공

### Go coverage

#### 실행 명령어

```bash
# 커버리지 프로파일 생성
go test -coverprofile=coverage.out ./...

# 함수별 커버리지 확인
go tool cover -func=coverage.out

# HTML 보고서 생성
go tool cover -html=coverage.out -o coverage.html

# 패키지별 커버리지 확인
go test -cover ./...
```

#### 출력 파싱 (go tool cover -func 출력)

```
github.com/app/pkg/handler/user.go:15:   CreateUser      80.0%
github.com/app/pkg/handler/user.go:45:   UpdateUser      60.0%
github.com/app/pkg/handler/user.go:78:   DeleteUser      0.0%
github.com/app/internal/service/auth.go:12:  Login       90.0%
github.com/app/internal/service/auth.go:50:  Logout      100.0%
total:                                       (statements) 72.5%
```

**파싱 포인트**:
- 마지막 줄의 `total`에서 전체 커버리지 추출
- 각 줄에서 `파일:라인:함수명 커버리지%` 패턴으로 함수별 커버리지 추출
- `0.0%`인 함수가 테스트 누락 함수

---

## 테스트 케이스 도출 기법

### 동치 분할 (Equivalence Partitioning)

입력 도메인을 동등한 동작을 하는 클래스로 나누어, 각 클래스에서 대표값을 선택하여 테스트합니다.

#### 원리

```
입력 도메인을 다음과 같이 분할:

유효한 클래스 (Valid Classes)
├── 일반적인 유효 입력
├── 최소 유효값 포함 클래스
└── 최대 유효값 포함 클래스

무효한 클래스 (Invalid Classes)
├── 범위 아래 값
├── 범위 위 값
├── 잘못된 타입
├── null/undefined/빈 값
└── 특수 문자/형식 오류
```

#### 적용 예시

```
함수: validateAge(age: number): boolean
- 유효 범위: 0 ~ 150

동치 클래스:
┌─────────────────┬──────────────┬─────────────┐
│ 클래스           │ 대표값        │ 기대 결과    │
├─────────────────┼──────────────┼─────────────┤
│ 유효: 일반       │ 25           │ true        │
│ 유효: 최소       │ 0            │ true        │
│ 유효: 최대       │ 150          │ true        │
│ 무효: 범위 아래  │ -1           │ false       │
│ 무효: 범위 위    │ 151          │ false       │
│ 무효: 비숫자     │ "abc"        │ false/throw │
│ 무효: null      │ null         │ false/throw │
│ 무효: 소수점     │ 25.5         │ 정책에 따라   │
└─────────────────┴──────────────┴─────────────┘
```

#### 테스트 코드 생성 규칙

각 동치 클래스마다 최소 1개의 테스트 케이스를 생성합니다:
- 유효 클래스: 대표값으로 정상 동작 확인
- 무효 클래스: 대표값으로 에러 처리 확인

### 경계값 분석 (Boundary Value Analysis)

동치 클래스의 경계에서 오류가 발생할 확률이 높으므로, 경계값을 집중적으로 테스트합니다.

#### 원리

```
범위 [min, max]에 대해 다음 값을 테스트:

     min-1    min    min+1    ...    max-1    max    max+1
      ↓        ↓       ↓              ↓       ↓       ↓
   [무효]   [유효]  [유효]          [유효]  [유효]  [무효]
```

#### 적용 예시

```
함수: getDiscount(quantity: number): number
- 1~9개: 0% 할인
- 10~99개: 10% 할인
- 100+개: 20% 할인

경계값 테스트:
┌───────────┬──────────┬──────────────┐
│ 입력값     │ 경계 유형  │ 기대 할인율   │
├───────────┼──────────┼──────────────┤
│ 0         │ 범위 밖   │ 에러/0%      │
│ 1         │ 최소 경계  │ 0%          │
│ 9         │ 상위 경계  │ 0%          │
│ 10        │ 전환점     │ 10%         │
│ 99        │ 상위 경계  │ 10%         │
│ 100       │ 전환점     │ 20%         │
│ -1        │ 범위 밖   │ 에러         │
│ 999999    │ 극단값     │ 20%         │
└───────────┴──────────┴──────────────┘
```

### 결정 테이블 (Decision Table Testing)

복합 조건이 있는 로직에서 모든 조건 조합을 체계적으로 테스트합니다.

#### 원리

조건과 동작의 조합을 테이블로 정리하여 누락 없이 테스트합니다.

#### 적용 예시

```
함수: canAccessResource(user, resource)
- 조건 1: 사용자가 인증되었는가?
- 조건 2: 사용자가 리소스 소유자인가?
- 조건 3: 리소스가 공개인가?

결정 테이블:
┌──────────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│          │ R1  │ R2  │ R3  │ R4  │ R5  │ R6  │ R7  │ R8  │
├──────────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│ 인증됨    │ Y   │ Y   │ Y   │ Y   │ N   │ N   │ N   │ N   │
│ 소유자    │ Y   │ Y   │ N   │ N   │ Y   │ Y   │ N   │ N   │
│ 공개      │ Y   │ N   │ Y   │ N   │ Y   │ N   │ Y   │ N   │
├──────────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│ 접근 허용 │ O   │ O   │ O   │ X   │ X   │ X   │ O   │ X   │
└──────────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

→ 8개의 테스트 케이스 생성
```

### 상태 전이 테스트 (State Transition Testing)

상태 기반 로직(상태 머신)의 모든 유효한 전이와 무효한 전이를 테스트합니다.

#### 적용 예시

```
주문 상태 전이:
  [생성됨] --결제--> [결제완료] --배송시작--> [배송중] --배송완료--> [완료]
     │                  │                                            │
     │--취소-->  [취소됨]  │--환불-->  [환불됨]                         │--반품--> [반품중]
                                                                           │
                                                                     [반품완료]

테스트해야 할 전이:
유효한 전이:
  - 생성됨 → 결제완료 (결제)
  - 생성됨 → 취소됨 (취소)
  - 결제완료 → 배송중 (배송시작)
  - 결제완료 → 환불됨 (환불)
  - 배송중 → 완료 (배송완료)
  - 완료 → 반품중 (반품)
  - 반품중 → 반품완료 (반품처리완료)

무효한 전이 (에러 발생 확인):
  - 생성됨 → 배송중 (결제 없이 배송 불가)
  - 취소됨 → 결제완료 (취소 후 결제 불가)
  - 완료 → 결제완료 (역방향 전이 불가)
  - 환불됨 → 배송중 (환불 후 배송 불가)
```

### 페어와이즈 테스트 (Pairwise Testing)

여러 매개변수의 모든 조합을 테스트하는 것이 비현실적일 때, 모든 매개변수 쌍의 조합을 최소한 한 번씩 포함하는 테스트 집합을 생성합니다.

#### 적용 예시

```
함수: formatDate(date, locale, timezone, format)
- locale: ["ko", "en", "ja"]
- timezone: ["UTC", "Asia/Seoul", "US/Eastern"]
- format: ["short", "long", "iso"]

전체 조합: 3 x 3 x 3 = 27개
페어와이즈: 9~12개로 축소 가능

페어와이즈 테스트 셋:
┌────┬──────┬─────────────┬────────┐
│ #  │locale│ timezone    │ format │
├────┼──────┼─────────────┼────────┤
│ 1  │ ko   │ UTC         │ short  │
│ 2  │ ko   │ Asia/Seoul  │ long   │
│ 3  │ ko   │ US/Eastern  │ iso    │
│ 4  │ en   │ UTC         │ long   │
│ 5  │ en   │ Asia/Seoul  │ iso    │
│ 6  │ en   │ US/Eastern  │ short  │
│ 7  │ ja   │ UTC         │ iso    │
│ 8  │ ja   │ Asia/Seoul  │ short  │
│ 9  │ ja   │ US/Eastern  │ long   │
└────┴──────┴─────────────┴────────┘
```

---

## 모킹 전략 가이드

### 모킹 원칙

1. **외부 의존성만 모킹**: 데이터베이스, HTTP 클라이언트, 파일 시스템 등
2. **내부 구현은 모킹하지 않음**: 테스트 대상의 내부 함수는 실제로 실행
3. **인터페이스 기반 모킹**: 구체적인 구현이 아닌 인터페이스/계약 기반으로 모킹
4. **최소 모킹**: 테스트에 필요한 최소한의 것만 모킹
5. **행동 검증 vs 상태 검증**: 가능하면 상태 검증을 우선, 필요한 경우에만 호출 검증

### JavaScript/TypeScript 모킹

#### Jest

```typescript
// 모듈 모킹
jest.mock('../services/userService');

// 함수 모킹
const mockFn = jest.fn();
mockFn.mockReturnValue('result');
mockFn.mockResolvedValue('async result');
mockFn.mockRejectedValue(new Error('failure'));

// 부분 모킹
jest.mock('../services/userService', () => ({
  ...jest.requireActual('../services/userService'),
  fetchUser: jest.fn(),
}));

// 타이머 모킹
jest.useFakeTimers();
jest.advanceTimersByTime(1000);
jest.useRealTimers();

// 날짜 모킹
jest.useFakeTimers({ now: new Date('2026-01-01T00:00:00Z') });

// 환경변수 모킹
const originalEnv = process.env;
beforeEach(() => {
  process.env = { ...originalEnv, API_KEY: 'test-key' };
});
afterEach(() => {
  process.env = originalEnv;
});

// fetch/axios 모킹
jest.mock('axios');
const mockedAxios = axios as jest.Mocked<typeof axios>;
mockedAxios.get.mockResolvedValue({ data: { id: 1, name: 'Test' } });
```

#### Vitest

```typescript
import { vi } from 'vitest';

// 모듈 모킹
vi.mock('../services/userService');

// 함수 모킹
const mockFn = vi.fn();
mockFn.mockReturnValue('result');

// 타이머 모킹
vi.useFakeTimers();
vi.advanceTimersByTime(1000);
vi.useRealTimers();

// 날짜 모킹
vi.setSystemTime(new Date('2026-01-01'));
```

### Python 모킹

#### unittest.mock

```python
from unittest.mock import Mock, MagicMock, patch, AsyncMock

# 기본 Mock
mock_service = Mock()
mock_service.fetch.return_value = {"id": 1, "name": "Test"}

# 예외 발생 모킹
mock_service.fetch.side_effect = ConnectionError("연결 실패")

# 비동기 Mock
mock_async = AsyncMock()
mock_async.return_value = {"id": 1}

# 데코레이터 패치
@patch("src.services.auth.external_api")
def test_login(mock_api):
    mock_api.verify.return_value = True
    result = login("user", "pass")
    assert result.success is True
    mock_api.verify.assert_called_once_with("user", "pass")

# 컨텍스트 매니저 패치
def test_fetch_data():
    with patch("src.services.data.requests.get") as mock_get:
        mock_get.return_value.json.return_value = {"data": [1, 2, 3]}
        mock_get.return_value.status_code = 200
        result = fetch_data("https://api.example.com")
        assert result == [1, 2, 3]

# 환경변수 패치
@patch.dict("os.environ", {"API_KEY": "test-key", "DEBUG": "true"})
def test_with_env():
    assert os.environ["API_KEY"] == "test-key"

# 파일 시스템 패치
@patch("builtins.open", mock_open(read_data="file content"))
def test_read_file():
    result = read_config("config.yaml")
    assert result == "file content"

# 날짜/시간 패치
@patch("src.utils.datetime")
def test_with_fixed_time(mock_datetime):
    mock_datetime.now.return_value = datetime(2026, 1, 1, 12, 0, 0)
    result = get_greeting()
    assert result == "좋은 오후입니다"
```

#### pytest-mock

```python
def test_with_mocker(mocker):
    mock_service = mocker.patch("src.services.auth.AuthService")
    mock_service.return_value.verify.return_value = True

    result = process_login("user", "pass")
    assert result.success is True
```

### Java 모킹 (Mockito)

```java
import static org.mockito.Mockito.*;
import static org.mockito.ArgumentMatchers.*;

// Mock 생성
@Mock
private UserRepository userRepository;

@InjectMocks
private UserService userService;

// 반환값 설정
when(userRepository.findById(1L)).thenReturn(Optional.of(user));
when(userRepository.findById(anyLong())).thenReturn(Optional.empty());

// 예외 발생 설정
when(userRepository.save(any())).thenThrow(new DataAccessException("DB 오류"));

// void 메서드 모킹
doNothing().when(userRepository).delete(any());
doThrow(new RuntimeException("삭제 실패")).when(userRepository).delete(null);

// 호출 검증
verify(userRepository).save(any(User.class));
verify(userRepository, times(2)).findById(anyLong());
verify(userRepository, never()).delete(any());

// 인자 캡쳐
ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
verify(userRepository).save(captor.capture());
User savedUser = captor.getValue();
assertEquals("test@example.com", savedUser.getEmail());

// 정적 메서드 모킹 (mockito-inline)
try (MockedStatic<LocalDateTime> mockedTime = mockStatic(LocalDateTime.class)) {
    mockedTime.when(LocalDateTime::now).thenReturn(fixedTime);
    // 테스트 실행
}
```

### Go 모킹

#### testify/mock

```go
type MockUserRepo struct {
    mock.Mock
}

func (m *MockUserRepo) FindByID(id string) (*User, error) {
    args := m.Called(id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*User), args.Error(1)
}

func TestGetUser(t *testing.T) {
    mockRepo := new(MockUserRepo)
    mockRepo.On("FindByID", "1").Return(&User{ID: "1", Name: "Test"}, nil)
    mockRepo.On("FindByID", "999").Return(nil, errors.New("not found"))

    service := NewUserService(mockRepo)

    user, err := service.GetUser("1")
    assert.NoError(t, err)
    assert.Equal(t, "Test", user.Name)

    _, err = service.GetUser("999")
    assert.Error(t, err)

    mockRepo.AssertExpectations(t)
}
```

#### gomock

```go
//go:generate mockgen -source=repository.go -destination=mock_repository.go -package=service

func TestGetUser(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockRepo := NewMockUserRepository(ctrl)
    mockRepo.EXPECT().FindByID("1").Return(&User{ID: "1"}, nil)

    service := NewUserService(mockRepo)
    user, err := service.GetUser("1")

    assert.NoError(t, err)
    assert.Equal(t, "1", user.ID)
}
```

### 모킹 대상별 전략 요약

| 의존성 유형 | 모킹 전략 | 주의사항 |
|-----------|---------|---------|
| HTTP 클라이언트 | 라이브러리 모킹 (axios, requests) 또는 MSW/httpretty | 실제 네트워크 호출 방지 |
| 데이터베이스 | Repository 인터페이스 모킹 또는 인메모리 DB | 트랜잭션 롤백 패턴도 고려 |
| 파일 시스템 | fs 모듈 모킹 또는 임시 디렉토리 사용 | 정리(cleanup) 보장 |
| 환경변수 | process.env / os.environ 패치 | 원본 복원 보장 |
| 시간/날짜 | 가짜 타이머 / 시간 모킹 | 테스트 후 복원 |
| 외부 SDK | SDK 클라이언트 모킹 | 응답 구조 정확히 재현 |
| 이벤트 | 이벤트 에미터 모킹 또는 스파이 | 비동기 이벤트 처리 주의 |
| 로거 | 로거 모킹 또는 스파이 | 로그 출력 검증 필요 시 |

---

## 테스트 품질 체크리스트

### 구조 품질

- [ ] **테스트 파일 구성**: 테스트가 논리적으로 그룹화되어 있는가 (describe/class별)
- [ ] **설정/해제**: beforeEach/afterEach (setUp/tearDown)이 적절히 사용되는가
- [ ] **테스트 이름**: 테스트 대상, 조건, 기대 결과가 명확하게 표현되는가
- [ ] **테스트 독립성**: 각 테스트가 다른 테스트에 의존하지 않는가
- [ ] **테스트 크기**: 각 테스트가 하나의 관심사만 다루는가

### 커버리지 품질

- [ ] **Happy path**: 모든 공개 함수에 정상 동작 테스트가 있는가
- [ ] **Edge case**: 경계값, null, 빈 값 등에 대한 테스트가 있는가
- [ ] **Error path**: 에러/예외 시나리오에 대한 테스트가 있는가
- [ ] **분기 커버리지**: if/else, switch, try/catch의 모든 분기가 테스트되는가
- [ ] **비동기 커버리지**: async/await, Promise의 성공/실패가 모두 테스트되는가

### Assertion 품질

- [ ] **구체적 값 비교**: `toBeTruthy()` 대신 `toBe(expectedValue)` 사용
- [ ] **에러 타입 검증**: `toThrow()` 대신 `toThrow(SpecificError)` 사용
- [ ] **다중 assertion**: 필요한 경우 여러 측면을 검증하되, 관련 없는 것은 별도 테스트로
- [ ] **부정 assertion**: 해서는 안 되는 것도 검증 (`not.toHaveBeenCalled()` 등)
- [ ] **비동기 assertion**: `await expect(promise).resolves/rejects` 적절 사용

### 모킹 품질

- [ ] **최소 모킹**: 필요한 외부 의존성만 모킹되는가
- [ ] **모킹 복원**: 각 테스트 후 모킹이 복원되는가
- [ ] **호출 검증**: 중요한 모킹 호출이 검증되는가
- [ ] **실제적 데이터**: 모킹 반환값이 실제 API 응답과 유사한가
- [ ] **에러 시나리오**: 의존성 실패 시나리오가 모킹되는가

### 유지보수 품질

- [ ] **DRY 원칙**: 공통 설정이 fixture/factory로 추출되어 있는가
- [ ] **가독성**: 테스트 코드를 읽으면 비즈니스 요구사항이 이해되는가
- [ ] **안정성**: 플레이키 테스트 패턴(타이밍, 난수, 날짜 의존)이 없는가
- [ ] **성능**: 불필요하게 느린 테스트(실제 네트워크, 긴 타이머)가 없는가
- [ ] **문서화**: 복잡한 테스트에 왜 이 테스트가 필요한지 주석이 있는가

### 점수 산출 기준

```
테스트 품질 점수 (100점 만점):

구조 품질 (25점)
├── 테스트 그룹화: 5점
├── 설정/해제 적절성: 5점
├── 테스트 이름 명확성: 5점
├── 테스트 독립성: 5점
└── 테스트 크기 적절성: 5점

커버리지 품질 (30점)
├── Happy path 커버리지: 10점
├── Edge case 커버리지: 8점
├── Error path 커버리지: 7점
└── 분기 커버리지: 5점

Assertion 품질 (20점)
├── 구체적 값 비교: 5점
├── 에러 타입 검증: 5점
├── assertion 완전성: 5점
└── 비동기 assertion: 5점

모킹 품질 (15점)
├── 최소 모킹 원칙: 5점
├── 모킹 복원: 5점
└── 실제적 데이터: 5점

유지보수 품질 (10점)
├── DRY 원칙: 3점
├── 가독성: 4점
└── 안정성: 3점
```
