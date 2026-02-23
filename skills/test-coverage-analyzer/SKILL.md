---
name: test-coverage-analyzer
description: 변경된 코드에 대한 테스트 커버리지를 분석하고, 누락된 테스트 케이스를 식별하여 테스트 코드 스켈레톤을 자동 생성하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) 테스트 커버리지 분석, (2) 테스트 누락 확인, (3) 테스트 생성해줘, (4) coverage 분석, (5) 어떤 테스트가 필요해?, (6) 변경된 코드에 대한 테스트 확인, (7) 테스트 품질 점검, (8) 누락된 테스트 케이스 식별, (9) 테스트 스켈레톤 생성.
---

# Test Coverage Analyzer

변경된 코드에 대한 테스트 커버리지를 분석하고, 누락된 테스트 케이스를 식별하여 테스트 코드 스켈레톤을 자동 생성합니다.

## Quick Start

사용자가 테스트 커버리지 분석을 요청하면 다음 워크플로우를 실행합니다:

1. **변경 파일 목록 추출**:
   ```bash
   # 스테이징된 변경 파일 확인
   git diff --cached --name-only --diff-filter=ACMR

   # 최근 커밋 대비 변경 파일 확인
   git diff --name-only --diff-filter=ACMR HEAD~1

   # 특정 브랜치 대비 변경 파일 확인
   git diff --name-only --diff-filter=ACMR main...HEAD
   ```

2. **소스 파일 필터링**: 테스트 파일과 설정 파일을 제외하고 소스 파일만 추출

3. **소스-테스트 파일 매핑**: [파일 매핑 규칙](#file-mapping-rules)에 따라 각 소스 파일에 대응하는 테스트 파일 식별

4. **테스트 파일 존재 확인**: 매핑된 테스트 파일이 실제로 존재하는지 확인

5. **기존 테스트 커버리지 분석**: 존재하는 테스트 파일에서 함수/메서드별 테스트 유무 확인

6. **누락 테스트 케이스 식별**: Happy path, Edge case, Error scenario 등 누락된 테스트 식별

7. **테스트 스켈레톤 생성**: 누락된 테스트에 대한 코드 스켈레톤 자동 생성

8. **커버리지 보고서 출력**: 분석 결과를 구조화된 보고서로 출력

9. **사용자 확인 요청**: 생성할 테스트를 사용자에게 확인받고 진행

## File Mapping Rules

소스 파일에서 대응하는 테스트 파일을 자동으로 매핑합니다. 프로젝트의 기존 테스트 구조를 먼저 탐지하여 매핑 규칙을 결정합니다.

### 프로젝트 구조 자동 탐지

분석 시작 전 프로젝트의 테스트 디렉토리 구조를 탐지합니다:

```bash
# 테스트 디렉토리 탐지
ls -d test/ tests/ __tests__/ spec/ 2>/dev/null

# 기존 테스트 파일 패턴 탐지
find . -name "*.test.*" -o -name "*.spec.*" -o -name "test_*" | head -20

# package.json의 Jest 설정 확인 (JavaScript/TypeScript)
cat package.json | grep -A 10 '"jest"'

# pytest 설정 확인 (Python)
cat pyproject.toml pytest.ini setup.cfg 2>/dev/null | grep -A 5 'pytest\|testpaths'
```

### JavaScript / TypeScript

| 소스 파일 | 테스트 파일 (우선순위 순) |
|-----------|------------------------|
| `src/utils/format.ts` | `src/utils/__tests__/format.test.ts` > `test/utils/format.test.ts` > `tests/utils/format.test.ts` |
| `src/components/Button.tsx` | `src/components/__tests__/Button.test.tsx` > `test/components/Button.test.tsx` |
| `src/hooks/useAuth.ts` | `src/hooks/__tests__/useAuth.test.ts` > `test/hooks/useAuth.test.ts` |
| `src/api/user.ts` | `src/api/__tests__/user.test.ts` > `test/api/user.test.ts` |
| `lib/helper.js` | `lib/__tests__/helper.test.js` > `test/helper.test.js` |

**매핑 규칙**:
- `.test.{ts,tsx,js,jsx}` 또는 `.spec.{ts,tsx,js,jsx}` 확장자 사용
- `__tests__/` 하위 디렉토리 또는 `test/`, `tests/` 루트 디렉토리
- 기존 프로젝트 패턴을 우선 따름

### Python

| 소스 파일 | 테스트 파일 (우선순위 순) |
|-----------|------------------------|
| `src/utils/format.py` | `tests/utils/test_format.py` > `tests/test_format.py` > `test/test_format.py` |
| `src/services/auth.py` | `tests/services/test_auth.py` > `tests/test_auth.py` |
| `app/models/user.py` | `tests/models/test_user.py` > `tests/test_user.py` |
| `pkg/helper.py` | `tests/test_helper.py` > `test_helper.py` |

**매핑 규칙**:
- `test_` 접두사 + 원본 파일명
- `tests/` 디렉토리 하위에 소스 디렉토리 구조 미러링
- `conftest.py` 파일은 매핑에서 제외

### Java

| 소스 파일 | 테스트 파일 |
|-----------|-----------|
| `src/main/java/com/app/UserService.java` | `src/test/java/com/app/UserServiceTest.java` |
| `src/main/java/com/app/util/StringHelper.java` | `src/test/java/com/app/util/StringHelperTest.java` |
| `src/main/java/com/app/controller/UserController.java` | `src/test/java/com/app/controller/UserControllerTest.java` |

**매핑 규칙**:
- `src/main/` -> `src/test/` 경로 변환
- 클래스명 + `Test` 접미사
- 패키지 구조 유지

### Go

| 소스 파일 | 테스트 파일 |
|-----------|-----------|
| `pkg/handler/user.go` | `pkg/handler/user_test.go` |
| `internal/service/auth.go` | `internal/service/auth_test.go` |
| `cmd/server/main.go` | `cmd/server/main_test.go` |

**매핑 규칙**:
- 동일 디렉토리에 `_test.go` 접미사
- 동일 패키지 또는 `_test` 패키지

### 커스텀 매핑 지원

프로젝트 루트에 `.testmapping.json` 파일이 있으면 커스텀 매핑 규칙을 사용합니다:

```json
{
  "mappings": [
    {
      "source": "src/**/*.ts",
      "test": "test/**/*.test.ts",
      "transform": {
        "directory": { "replace": "src/", "with": "test/" },
        "filename": { "suffix": ".test" }
      }
    }
  ]
}
```

## Coverage Analysis

### 함수/메서드별 테스트 존재 여부 확인

소스 파일에서 함수와 메서드를 추출하고, 대응하는 테스트 파일에서 해당 함수/메서드에 대한 테스트가 존재하는지 확인합니다.

#### 소스 코드 분석

```
소스 파일 분석 단계:
1. 파일을 읽고 언어를 식별
2. 함수/메서드/클래스 선언을 추출
3. 각 함수의 시그니처(매개변수, 반환 타입)를 파싱
4. 분기문(if/else, switch, try/catch)을 식별
5. 외부 의존성 호출을 식별
```

**함수 추출 패턴 (언어별)**:

- **TypeScript/JavaScript**: `function`, `const fn = () =>`, `class method`, `export`
- **Python**: `def`, `class method`, `@staticmethod`, `@classmethod`
- **Java**: `public/private/protected method`, `static method`
- **Go**: `func`, `method receiver`

#### 테스트 파일 분석

```
테스트 파일 분석 단계:
1. 테스트 파일을 읽기
2. describe/it/test 블록 구조를 파싱
3. 각 테스트가 커버하는 함수/메서드를 식별
4. assertion 패턴을 분석
5. 모킹된 의존성을 식별
```

### 분기(Branch) 커버리지 분석

소스 코드의 분기문을 분석하여 테스트에서 각 분기가 커버되는지 확인합니다:

```
분기 유형         │ 확인 사항
─────────────────┼──────────────────────────────
if/else          │ true/false 양쪽 분기 테스트 존재 여부
if (복합 조건)    │ 각 조건 조합의 테스트 존재 여부
switch/case      │ 각 case + default 테스트 존재 여부
try/catch        │ 정상 경로 + 예외 경로 테스트 존재 여부
삼항 연산자       │ 양쪽 결과값 테스트 존재 여부
Optional chaining │ null/undefined 케이스 테스트 존재 여부
Guard clause     │ early return 조건 테스트 존재 여부
```

### 커버리지 도구 연동

기존 커버리지 도구의 출력을 파싱하여 정량적 커버리지 데이터를 활용합니다:

#### Jest (JavaScript/TypeScript)

```bash
# 커버리지 실행
npx jest --coverage --coverageReporters=json-summary --silent 2>/dev/null

# 커버리지 보고서 위치
cat coverage/coverage-summary.json
```

#### pytest (Python)

```bash
# 커버리지 실행
python -m pytest --cov=src --cov-report=json --quiet 2>/dev/null

# 커버리지 보고서 위치
cat coverage.json
```

#### JaCoCo (Java)

```bash
# Maven으로 커버리지 실행
mvn jacoco:report -q 2>/dev/null

# Gradle로 커버리지 실행
./gradlew jacocoTestReport 2>/dev/null

# 커버리지 보고서 위치
cat target/site/jacoco/jacoco.csv
# 또는
cat build/reports/jacoco/test/jacocoTestReport.csv
```

#### Go

```bash
# 커버리지 실행
go test -coverprofile=coverage.out ./... 2>/dev/null

# 함수별 커버리지 확인
go tool cover -func=coverage.out
```

### 커버리지 목표 대비 현황

프로젝트 설정에서 커버리지 목표를 읽거나, 기본값을 사용합니다:

```
커버리지 항목      │ 기본 목표  │ 권장 목표
─────────────────┼──────────┼──────────
라인 커버리지      │ 70%      │ 80%+
분기 커버리지      │ 60%      │ 70%+
함수 커버리지      │ 80%      │ 90%+
구문 커버리지      │ 70%      │ 80%+
```

## Missing Test Detection

### Happy Path 테스트

모든 공개(public/exported) 함수에 대해 정상 동작 테스트가 존재하는지 확인합니다:

- 함수의 기본 입력으로 기대 출력을 반환하는지
- 주요 사용 시나리오가 커버되는지
- 반환값과 부수 효과(side effects)가 모두 검증되는지

### Edge Case 제안

소스 코드의 매개변수 타입과 로직을 분석하여 Edge case를 자동으로 제안합니다:

```
매개변수 타입      │ 제안하는 Edge Case
─────────────────┼──────────────────────────────────
string           │ 빈 문자열 "", null, undefined, 매우 긴 문자열, 특수문자, 유니코드
number           │ 0, 음수, NaN, Infinity, Number.MAX_SAFE_INTEGER, 소수점
array            │ 빈 배열 [], 단일 요소, 매우 큰 배열, 중복 요소, null 요소
object           │ 빈 객체 {}, null, undefined, 순환 참조, 중첩된 객체
boolean          │ true, false, truthy/falsy 값
Date             │ 유효하지 않은 날짜, 경계값(윤년, 월말), 타임존
Promise          │ resolve, reject, 타임아웃, 동시 실행
File/Stream      │ 빈 파일, 큰 파일, 읽기 전용, 존재하지 않는 파일
```

### Error/Exception 시나리오

소스 코드의 예외 처리 패턴을 분석하여 테스트가 필요한 시나리오를 식별합니다:

- `throw` 문이 있는 모든 경로에 대한 테스트
- `try/catch` 블록의 catch 경로 테스트
- 외부 API 호출 실패 시나리오
- 데이터베이스 연결 실패 시나리오
- 파일 시스템 오류 시나리오
- 네트워크 타임아웃 시나리오
- 권한 부족(permission denied) 시나리오
- 유효성 검사 실패 시나리오

### 비동기 코드 테스트

비동기 패턴에 대한 테스트 존재 여부를 확인합니다:

- `async/await` 함수의 성공/실패 테스트
- `Promise.all`, `Promise.race`, `Promise.allSettled` 패턴 테스트
- 콜백 기반 비동기 코드 테스트
- 타이머/딜레이 관련 테스트 (`setTimeout`, `setInterval`)
- 이벤트 기반 비동기 처리 테스트
- 동시성(concurrency) 관련 테스트

### 외부 의존성 모킹 테스트

외부 의존성이 적절히 모킹되어 테스트되는지 확인합니다:

- HTTP 클라이언트 (axios, fetch, got 등)
- 데이터베이스 클라이언트 (Prisma, TypeORM, SQLAlchemy 등)
- 파일 시스템 (fs, os.path 등)
- 외부 서비스 SDK (AWS SDK, Firebase 등)
- 환경 변수 (`process.env`, `os.environ`)
- 시간 관련 함수 (`Date.now()`, `time.time()`)

## Skeleton Generation

### 테스트 스켈레톤 생성 원칙

1. **Given-When-Then 패턴** 적용: 모든 테스트를 Given(준비)-When(실행)-Then(검증) 구조로 작성
2. **명확한 테스트 이름**: 테스트 대상, 조건, 기대 결과를 포함하는 이름
3. **독립적인 테스트**: 각 테스트가 다른 테스트에 의존하지 않도록 설계
4. **최소 모킹**: 필요한 외부 의존성만 모킹
5. **구체적인 assertion**: `toBeTruthy()` 대신 `toBe(expectedValue)` 사용

### 언어별 테스트 프레임워크

#### Jest (JavaScript/TypeScript)

```typescript
import { describe, it, expect, jest, beforeEach, afterEach } from '@jest/globals';
import { TargetFunction } from '../src/target';

describe('TargetFunction', () => {
  // 공유 상태 설정
  let mockDependency: jest.Mocked<DependencyType>;

  beforeEach(() => {
    mockDependency = {
      method: jest.fn(),
    } as jest.Mocked<DependencyType>;
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  describe('정상 동작', () => {
    it('유효한 입력이 주어지면 기대 결과를 반환해야 한다', () => {
      // Given
      const input = { /* 유효한 입력 데이터 */ };

      // When
      const result = TargetFunction(input);

      // Then
      expect(result).toEqual(/* 기대 결과 */);
    });
  });

  describe('Edge Cases', () => {
    it('빈 입력이 주어지면 기본값을 반환해야 한다', () => {
      // Given
      const input = {};

      // When
      const result = TargetFunction(input);

      // Then
      expect(result).toEqual(/* 기본값 */);
    });

    it('null 입력이 주어지면 에러를 던져야 한다', () => {
      // Given & When & Then
      expect(() => TargetFunction(null)).toThrow(/* 에러 타입 */);
    });
  });

  describe('에러 시나리오', () => {
    it('의존성이 실패하면 적절한 에러를 반환해야 한다', async () => {
      // Given
      mockDependency.method.mockRejectedValue(new Error('연결 실패'));

      // When & Then
      await expect(TargetFunction(mockDependency)).rejects.toThrow('연결 실패');
    });
  });
});
```

#### Vitest (JavaScript/TypeScript)

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { TargetFunction } from '../src/target';

describe('TargetFunction', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('유효한 입력이 주어지면 기대 결과를 반환해야 한다', () => {
    // Given
    const input = { /* 유효한 입력 데이터 */ };

    // When
    const result = TargetFunction(input);

    // Then
    expect(result).toEqual(/* 기대 결과 */);
  });
});
```

#### pytest (Python)

```python
import pytest
from unittest.mock import Mock, patch, MagicMock
from src.target import TargetClass, target_function


class TestTargetFunction:
    """target_function에 대한 테스트"""

    def setup_method(self):
        """각 테스트 전 실행되는 설정"""
        self.mock_dependency = Mock()

    def test_유효한_입력이면_기대_결과를_반환한다(self):
        """Given 유효한 입력, When 함수 호출, Then 기대 결과 반환"""
        # Given
        input_data = {"key": "value"}

        # When
        result = target_function(input_data)

        # Then
        assert result == expected_value

    def test_빈_입력이면_기본값을_반환한다(self):
        """Given 빈 입력, When 함수 호출, Then 기본값 반환"""
        # Given
        input_data = {}

        # When
        result = target_function(input_data)

        # Then
        assert result == default_value

    def test_None_입력이면_ValueError를_발생시킨다(self):
        """Given None 입력, When 함수 호출, Then ValueError 발생"""
        # Given & When & Then
        with pytest.raises(ValueError, match="입력값이 필요합니다"):
            target_function(None)

    @patch("src.target.external_service")
    def test_외부_서비스_실패시_적절한_에러를_반환한다(self, mock_service):
        """Given 외부 서비스 실패, When 함수 호출, Then ConnectionError 발생"""
        # Given
        mock_service.call.side_effect = ConnectionError("연결 실패")

        # When & Then
        with pytest.raises(ConnectionError):
            target_function({"key": "value"})


@pytest.fixture
def sample_data():
    """테스트용 샘플 데이터 fixture"""
    return {
        "id": 1,
        "name": "테스트",
        "items": [1, 2, 3],
    }
```

#### JUnit 5 (Java)

```java
import org.junit.jupiter.api.*;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.ValueSource;
import org.junit.jupiter.params.provider.NullAndEmptySource;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

@DisplayName("TargetService 테스트")
class TargetServiceTest {

    @Mock
    private DependencyService dependencyService;

    private TargetService targetService;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
        targetService = new TargetService(dependencyService);
    }

    @Nested
    @DisplayName("process 메서드")
    class ProcessMethod {

        @Test
        @DisplayName("유효한 입력이 주어지면 기대 결과를 반환해야 한다")
        void shouldReturnExpectedResult_WhenValidInput() {
            // Given
            String input = "valid-input";
            when(dependencyService.fetch(input)).thenReturn(expectedData);

            // When
            Result result = targetService.process(input);

            // Then
            assertNotNull(result);
            assertEquals(expectedValue, result.getValue());
            verify(dependencyService).fetch(input);
        }

        @ParameterizedTest
        @NullAndEmptySource
        @DisplayName("null 또는 빈 입력이 주어지면 IllegalArgumentException을 던져야 한다")
        void shouldThrowException_WhenNullOrEmptyInput(String input) {
            // Given & When & Then
            assertThrows(IllegalArgumentException.class,
                () -> targetService.process(input));
        }

        @Test
        @DisplayName("의존성이 실패하면 ServiceException을 던져야 한다")
        void shouldThrowServiceException_WhenDependencyFails() {
            // Given
            when(dependencyService.fetch(anyString()))
                .thenThrow(new RuntimeException("연결 실패"));

            // When & Then
            assertThrows(ServiceException.class,
                () -> targetService.process("input"));
        }
    }
}
```

#### Go testing

```go
package handler

import (
    "errors"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "github.com/stretchr/testify/require"
)

// MockDependency는 DependencyInterface의 모킹 구현
type MockDependency struct {
    mock.Mock
}

func (m *MockDependency) Fetch(id string) (*Data, error) {
    args := m.Called(id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*Data), args.Error(1)
}

func TestTargetFunction(t *testing.T) {
    t.Run("유효한 입력이 주어지면 기대 결과를 반환해야 한다", func(t *testing.T) {
        // Given
        mockDep := new(MockDependency)
        mockDep.On("Fetch", "valid-id").Return(&Data{Value: "expected"}, nil)

        // When
        result, err := TargetFunction(mockDep, "valid-id")

        // Then
        require.NoError(t, err)
        assert.Equal(t, "expected", result.Value)
        mockDep.AssertExpectations(t)
    })

    t.Run("빈 입력이 주어지면 에러를 반환해야 한다", func(t *testing.T) {
        // Given
        mockDep := new(MockDependency)

        // When
        result, err := TargetFunction(mockDep, "")

        // Then
        assert.Nil(t, result)
        assert.EqualError(t, err, "입력값이 비어있습니다")
    })

    t.Run("의존성이 실패하면 적절한 에러를 반환해야 한다", func(t *testing.T) {
        // Given
        mockDep := new(MockDependency)
        mockDep.On("Fetch", "id").Return(nil, errors.New("연결 실패"))

        // When
        result, err := TargetFunction(mockDep, "id")

        // Then
        assert.Nil(t, result)
        assert.Contains(t, err.Error(), "연결 실패")
    })
}

// 테이블 기반 테스트 (Table-Driven Tests)
func TestTargetFunction_TableDriven(t *testing.T) {
    tests := []struct {
        name        string
        input       string
        expected    string
        expectError bool
    }{
        {"유효한 입력", "valid", "expected-result", false},
        {"빈 입력", "", "", true},
        {"특수문자 입력", "!@#$%", "", true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result, err := TargetFunction(nil, tt.input)
            if tt.expectError {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
                assert.Equal(t, tt.expected, result)
            }
        })
    }
}
```

## Language Support

### 지원 언어 및 프레임워크 매트릭스

| 언어 | 테스트 프레임워크 | 모킹 라이브러리 | 커버리지 도구 |
|------|----------------|---------------|-------------|
| JavaScript | Jest, Vitest, Mocha + Chai | jest.fn(), vi.fn(), sinon | Jest --coverage, c8, istanbul |
| TypeScript | Jest, Vitest, Mocha + Chai | jest.fn(), vi.fn(), sinon | Jest --coverage, c8, istanbul |
| Python | pytest, unittest | unittest.mock, pytest-mock | pytest-cov, coverage.py |
| Java | JUnit 5, TestNG | Mockito, EasyMock | JaCoCo, Cobertura |
| Go | testing, testify | testify/mock, gomock | go test -cover |
| Kotlin | JUnit 5, Kotest | MockK, Mockito-Kotlin | JaCoCo |
| Rust | built-in #[test] | mockall | cargo-tarpaulin |
| C# | xUnit, NUnit, MSTest | Moq, NSubstitute | coverlet |

### 프레임워크 자동 감지

프로젝트 설정 파일을 분석하여 사용 중인 테스트 프레임워크를 자동으로 감지합니다:

```
감지 파일                    │ 프레임워크
───────────────────────────┼──────────────────
package.json (jest)        │ Jest
vite.config.ts (vitest)    │ Vitest
.mocharc.yml               │ Mocha
pytest.ini / pyproject.toml│ pytest
build.gradle (JUnit)       │ JUnit 5
pom.xml (JUnit)            │ JUnit 5
go.mod                     │ Go testing
Cargo.toml                 │ Rust #[test]
*.csproj (xUnit)           │ xUnit
```

## Test Quality Analysis

### 테스트 이름 명확성 검사

좋은 테스트 이름은 다음 구조를 따릅니다:
- `[테스트 대상]_[조건]_[기대 결과]`
- 또는 자연어: "~이면 ~해야 한다"

```
검사 항목                    │ 나쁜 예시              │ 좋은 예시
───────────────────────────┼──────────────────────┼─────────────────────────────
테스트 대상 명시             │ test1()              │ testCalculateTotal()
조건 명시                   │ testFail()           │ testCalculateTotal_EmptyCart()
기대 결과 명시              │ testAdd()            │ testAdd_TwoPositiveNumbers_ReturnsSum()
한국어 설명 (describe/it)   │ it('works')          │ it('유효한 입력이면 합계를 반환한다')
```

### 단언(Assertion) 적절성 확인

```
검사 항목                    │ 개선 필요                     │ 적절함
───────────────────────────┼────────────────────────────┼──────────────────────────
구체적인 값 비교             │ expect(result).toBeTruthy() │ expect(result).toBe(42)
에러 타입 검증              │ expect(fn).toThrow()        │ expect(fn).toThrow(TypeError)
배열 검증                   │ expect(arr.length > 0)      │ expect(arr).toHaveLength(3)
객체 부분 검증              │ expect(obj.name).toBe(..)   │ expect(obj).toMatchObject({..})
호출 검증                   │ (검증 없음)                   │ expect(mock).toHaveBeenCalledWith(..)
```

### 테스트 독립성 검증

다음 패턴을 감지하여 경고합니다:

- **공유 변수 변경**: 테스트 간 공유되는 변수를 변경하는 코드
- **테스트 순서 의존**: 특정 순서로 실행해야만 통과하는 테스트
- **전역 상태 변경**: `global`, `process.env` 등을 변경하고 복원하지 않는 코드
- **파일 시스템 의존**: 특정 파일이 존재해야만 통과하는 테스트
- **외부 서비스 의존**: 실제 외부 서비스에 의존하는 테스트

### 플레이키(Flaky) 테스트 패턴 감지

```
플레이키 패턴                │ 설명                           │ 해결 방법
───────────────────────────┼──────────────────────────────┼──────────────────────
타이밍 의존                 │ setTimeout, sleep 사용          │ 가짜 타이머(fake timer) 사용
네트워크 의존               │ 실제 API 호출                   │ 모킹 또는 MSW 사용
난수 의존                   │ Math.random() 사용              │ 시드(seed) 고정 또는 모킹
날짜/시간 의존              │ Date.now() 직접 사용             │ 시간 모킹
경쟁 조건                   │ 비동기 작업 순서 의존            │ await/Promise 체인 사용
환경 의존                   │ 특정 OS, Node 버전 필요          │ 환경 조건 명시 또는 모킹
```

## Workflow

전체 워크플로우를 단계별로 설명합니다:

```
1. [분석 시작] 사용자 요청 수신
   ├─ 변경 범위 확인 (git diff, 특정 파일, 전체 프로젝트)
   └─ 프로젝트 언어/프레임워크 감지

2. [파일 매핑] 소스 → 테스트 파일 매핑
   ├─ 프로젝트 테스트 구조 탐지
   ├─ 소스 파일별 테스트 파일 경로 계산
   └─ 커스텀 매핑 규칙 확인 (.testmapping.json)

3. [존재 확인] 테스트 파일 존재 여부 확인
   ├─ 존재하는 테스트 파일 → 커버리지 분석으로 진행
   ├─ 존재하지 않는 테스트 파일 → 경고 출력 + 생성 제안
   └─ 결과: 존재/미존재 파일 목록

4. [커버리지 분석] 기존 테스트의 커버리지 분석
   ├─ 함수/메서드별 테스트 존재 확인
   ├─ 분기 커버리지 분석
   ├─ 커버리지 도구 연동 (가능한 경우)
   └─ 결과: 함수별 커버리지 현황

5. [누락 식별] 누락된 테스트 케이스 식별
   ├─ Happy path 테스트 확인
   ├─ Edge case 제안
   ├─ Error/exception 시나리오 확인
   ├─ 비동기 코드 테스트 확인
   ├─ 외부 의존성 모킹 확인
   └─ 결과: 누락된 테스트 케이스 목록

6. [스켈레톤 생성] 테스트 코드 스켈레톤 생성
   ├─ 언어/프레임워크별 템플릿 적용
   ├─ describe/it 구조 생성
   ├─ Given-When-Then 패턴 적용
   ├─ 모킹 코드 생성
   └─ 결과: 생성 가능한 테스트 스켈레톤

7. [보고서 출력] 커버리지 보고서 출력
   ├─ 파일별 커버리지 현황
   ├─ 함수별 테스트 현황
   ├─ 누락 테스트 목록
   ├─ 테스트 품질 점수
   └─ 형식: references/templates.md의 보고서 템플릿 사용

8. [사용자 확인] 생성할 테스트 확인 요청
   ├─ 생성할 테스트 파일 목록 제시
   ├─ 사용자 선택 대기
   ├─ 선택된 테스트만 생성
   └─ 생성 후 실행 가이드 제공
```

## Error Handling

### 에러 시나리오 및 대응

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | git 저장소가 아님 | `git rev-parse` 실패 | 수동 파일 지정 모드로 전환 |
| 2 | 변경 파일 없음 | `git diff` 출력 비어있음 | 전체 프로젝트 분석 또는 특정 파일 지정 요청 |
| 3 | 지원하지 않는 언어 | 확장자 매핑 실패 | 경고 출력 후 해당 파일 건너뜀 |
| 4 | 테스트 파일 미존재 | 파일 시스템 확인 실패 | 새 테스트 파일 생성 제안 |
| 5 | 소스 파일 파싱 실패 | 구문 분석 오류 | 경고 출력 후 기본 스켈레톤 제공 |
| 6 | 커버리지 도구 미설치 | 도구 실행 실패 | 정적 분석만으로 진행 |
| 7 | 대규모 변경 (50+ 파일) | 파일 수 체크 | 범위 축소 요청 또는 배치 처리 |
| 8 | 모노레포 구조 | 복수 패키지 감지 | 패키지별 개별 분석 |
| 9 | 커버리지 보고서 파싱 실패 | JSON 파싱 오류 | 정적 분석 결과만 제공 |
| 10 | 테스트 프레임워크 감지 실패 | 설정 파일 없음 | 사용자에게 프레임워크 선택 요청 |

### 에러 메시지 템플릿

에러 메시지는 [references/templates.md](references/templates.md)에서 사전 정의된 한국어 메시지를 사용합니다.

## Best Practices

### 분석 시

1. **프로젝트 구조 우선 파악**: 무작정 분석하지 말고 프로젝트의 테스트 관례를 먼저 파악합니다
2. **기존 테스트 스타일 존중**: 새로 생성하는 테스트는 기존 테스트의 스타일과 패턴을 따릅니다
3. **점진적 접근**: 한 번에 모든 테스트를 생성하지 말고 우선순위가 높은 것부터 제안합니다
4. **정적 분석 + 동적 분석**: 코드 읽기만으로 부족할 때 커버리지 도구 결과를 함께 활용합니다

### 테스트 생성 시

1. **테스트 가독성**: 테스트 코드는 문서의 역할도 하므로 명확하게 작성합니다
2. **하나의 테스트, 하나의 관심사**: 각 테스트는 하나의 동작만 검증합니다
3. **Arrange-Act-Assert (AAA) / Given-When-Then**: 일관된 구조를 사용합니다
4. **구체적인 assertion**: `toBeTruthy()` 대신 정확한 값을 비교합니다
5. **의미 있는 테스트 데이터**: `"test"`, `123` 대신 도메인에 맞는 데이터를 사용합니다
6. **모킹 최소화**: 필요한 외부 의존성만 모킹하고, 내부 구현은 모킹하지 않습니다

### 보고서 작성 시

1. **실행 가능한 제안**: 추상적인 조언 대신 구체적인 코드를 제시합니다
2. **우선순위 제시**: Critical > High > Medium 순으로 정렬하여 가장 중요한 누락을 먼저 표시합니다
3. **정량적 데이터**: 가능한 경우 커버리지 퍼센트와 같은 숫자를 포함합니다
4. **컨텍스트 제공**: 왜 이 테스트가 필요한지 설명합니다

## Analysis Guide

상세한 분석 방법론은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

주요 분석 방법:
- **동치 분할 (Equivalence Partitioning)**: 입력을 동치 클래스로 나누어 테스트 케이스 도출
- **경계값 분석 (Boundary Value Analysis)**: 경계값에서의 동작을 집중 테스트
- **결정 테이블 (Decision Table)**: 복합 조건의 모든 조합을 체계적으로 테스트
- **상태 전이 테스트 (State Transition Testing)**: 상태 기반 로직의 전이를 테스트

## Templates

보고서 및 테스트 스켈레톤 템플릿은 [references/templates.md](references/templates.md)를 참조하세요.

- 커버리지 분석 보고서 형식
- 언어별 테스트 스켈레톤 템플릿
- 누락 테스트 경고 메시지
- Given-When-Then 패턴 템플릿
- 테스트 품질 점수 카드
