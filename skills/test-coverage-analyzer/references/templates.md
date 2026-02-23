# 테스트 커버리지 분석 템플릿

커버리지 분석 보고서, 테스트 스켈레톤, 경고 메시지 등을 위한 종합 템플릿 모음입니다.

## 목차

1. [커버리지 분석 보고서 형식](#커버리지-분석-보고서-형식)
2. [언어별 테스트 스켈레톤 템플릿](#언어별-테스트-스켈레톤-템플릿)
3. [누락 테스트 경고 메시지](#누락-테스트-경고-메시지)
4. [Given-When-Then 패턴 템플릿](#given-when-then-패턴-템플릿)
5. [테스트 품질 점수 카드](#테스트-품질-점수-카드)

---

## 커버리지 분석 보고서 형식

### 전체 보고서 템플릿

```
============================================================
          테스트 커버리지 분석 보고서
============================================================

프로젝트: {PROJECT_NAME}
분석 범위: {ANALYSIS_SCOPE}
분석 일시: {TIMESTAMP}
분석 도구: {COVERAGE_TOOL} (사용 가능한 경우)

============================================================
1. 파일별 커버리지 현황
============================================================

파일                              │ 테스트 파일    │ 함수    │ 커버리지
─────────────────────────────────┼──────────────┼────────┼─────────
{SOURCE_FILE_1}                  │ {STATUS}     │ {M}/{N}│ {PCT}%
{SOURCE_FILE_2}                  │ {STATUS}     │ {M}/{N}│ {PCT}%
{SOURCE_FILE_3}                  │ {STATUS}     │ {M}/{N}│ {PCT}%

전체 커버리지: {COVERED}/{TOTAL} 함수 ({TOTAL_PCT}%)
목표 대비: {TOTAL_PCT}% / {TARGET_PCT}% ({STATUS_LABEL})

============================================================
2. 함수별 테스트 현황
============================================================

{SOURCE_FILE_1}:
  함수/메서드                      │ 테스트 상태   │ 세부 사항
  ────────────────────────────────┼────────────┼──────────────
  {FUNCTION_1}()                  │ 테스트됨     │ Happy path + Edge case
  {FUNCTION_2}()                  │ 부분 테스트   │ Happy path만 존재
  {FUNCTION_3}()                  │ 미테스트      │ 테스트 없음
  {FUNCTION_4}()                  │ 테스트됨     │ Happy path + Error case

============================================================
3. 분기 커버리지 현황
============================================================

{SOURCE_FILE_1}:
  라인 {LINE}  │ if/else     │ true 분기만 테스트됨 (false 분기 누락)
  라인 {LINE}  │ try/catch   │ try만 테스트됨 (catch 경로 누락)
  라인 {LINE}  │ switch      │ 3/5 case 테스트됨 (case 'error', default 누락)

============================================================
4. 누락된 테스트 케이스
============================================================

[Critical] {DESCRIPTION}
  파일: {FILE_PATH}
  이유: {REASON}
  제안: {SUGGESTION}

[High] {DESCRIPTION}
  파일: {FILE_PATH}
  이유: {REASON}
  제안: {SUGGESTION}

[Medium] {DESCRIPTION}
  파일: {FILE_PATH}
  이유: {REASON}
  제안: {SUGGESTION}

============================================================
5. 커버리지 도구 데이터 (해당 시)
============================================================

도구: {TOOL_NAME}
라인 커버리지: {LINE_PCT}%
분기 커버리지: {BRANCH_PCT}%
함수 커버리지: {FUNC_PCT}%
구문 커버리지: {STMT_PCT}%

============================================================
6. 테스트 품질 점수
============================================================

총점: {SCORE}/100

  구조 품질:      {STRUCTURE_SCORE}/25
  커버리지 품질:   {COVERAGE_SCORE}/30
  Assertion 품질: {ASSERTION_SCORE}/20
  모킹 품질:      {MOCK_SCORE}/15
  유지보수 품질:   {MAINTAIN_SCORE}/10

============================================================
7. 생성 제안
============================================================

생성 가능한 테스트:
  1. {FILE_1} (신규 파일) - {DESCRIPTION}
  2. {FILE_2}에 {N}개 테스트 추가
  3. {FILE_3}에 {N}개 테스트 추가

테스트를 생성하시겠습니까? (전체/선택/취소)
============================================================
```

### 간략 보고서 템플릿 (요약용)

```
테스트 커버리지 요약 ({PROJECT_NAME})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

변경 파일: {TOTAL_FILES}개
테스트 파일 존재: {TESTED_FILES}개
테스트 파일 미존재: {UNTESTED_FILES}개

함수 커버리지: {COVERED_FUNCS}/{TOTAL_FUNCS} ({FUNC_PCT}%)
누락 테스트: Critical {CRITICAL}개 | High {HIGH}개 | Medium {MEDIUM}개

최우선 조치: {TOP_ACTION}
```

### 파일별 상세 보고서 템플릿

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
파일: {SOURCE_FILE}
테스트: {TEST_FILE} ({EXISTS_STATUS})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

함수 목록:
  [O] {FUNCTION_1}() - 테스트 완료
      - Happy path: O
      - Edge case: O (빈 입력, null)
      - Error case: O (타임아웃, 연결 오류)

  [△] {FUNCTION_2}() - 부분 테스트
      - Happy path: O
      - Edge case: X (빈 입력 테스트 누락)
      - Error case: X (예외 시나리오 누락)

  [X] {FUNCTION_3}() - 미테스트
      - 테스트가 존재하지 않습니다
      - 제안: Happy path + null 입력 + 에러 시나리오

분기 커버리지:
  라인 {LINE}: if ({CONDITION}) - true만 테스트됨
  라인 {LINE}: try/catch - catch 경로 미테스트

외부 의존성:
  - {DEPENDENCY_1}: 모킹됨 (O)
  - {DEPENDENCY_2}: 모킹 안됨 (X) - 모킹 필요

품질 점수: {SCORE}/100
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 언어별 테스트 스켈레톤 템플릿

### Jest (JavaScript/TypeScript) 스켈레톤

```typescript
// {TEST_FILE_PATH}
// 자동 생성된 테스트 스켈레톤 - 커버리지 분석 기반
// 생성 일시: {TIMESTAMP}

import { describe, it, expect, jest, beforeEach, afterEach } from '@jest/globals';
import {
  {EXPORTED_FUNCTIONS}
} from '{RELATIVE_SOURCE_PATH}';

// 외부 의존성 모킹
{MOCK_IMPORTS}

describe('{MODULE_NAME}', () => {
  // 공유 설정
  {SHARED_SETUP}

  beforeEach(() => {
    jest.clearAllMocks();
    {BEFORE_EACH_SETUP}
  });

  afterEach(() => {
    jest.restoreAllMocks();
    {AFTER_EACH_CLEANUP}
  });

  // ========================================
  // {FUNCTION_NAME_1}
  // ========================================
  describe('{FUNCTION_NAME_1}', () => {
    describe('정상 동작 (Happy Path)', () => {
      it('{FUNCTION_NAME_1}에 유효한 입력이 주어지면 기대 결과를 반환해야 한다', () => {
        // Given
        const input = {INPUT_EXAMPLE};

        // When
        const result = {FUNCTION_NAME_1}(input);

        // Then
        expect(result).toEqual({EXPECTED_RESULT});
      });

      // TODO: 추가 정상 시나리오
    });

    describe('Edge Cases', () => {
      it('빈 입력이 주어지면 {EXPECTED_BEHAVIOR}', () => {
        // Given
        const input = {EMPTY_INPUT};

        // When
        const result = {FUNCTION_NAME_1}(input);

        // Then
        expect(result).toEqual({EXPECTED_EMPTY_RESULT});
      });

      it('null이 주어지면 {EXPECTED_BEHAVIOR}', () => {
        // Given & When & Then
        expect(() => {FUNCTION_NAME_1}(null)).toThrow({EXPECTED_ERROR});
      });

      // TODO: 경계값 테스트
      // TODO: 특수 문자 입력 테스트
    });

    describe('에러 시나리오', () => {
      it('외부 의존성 실패 시 적절한 에러를 처리해야 한다', async () => {
        // Given
        {MOCK_SETUP_ERROR}

        // When & Then
        await expect({FUNCTION_NAME_1}({ERROR_INPUT}))
          .rejects.toThrow({EXPECTED_ERROR_TYPE});
      });

      // TODO: 타임아웃 시나리오
      // TODO: 네트워크 오류 시나리오
    });
  });

  // ========================================
  // {FUNCTION_NAME_2}
  // ========================================
  // TODO: 위와 동일한 구조로 나머지 함수 테스트 작성
});
```

### pytest (Python) 스켈레톤

```python
# {TEST_FILE_PATH}
# 자동 생성된 테스트 스켈레톤 - 커버리지 분석 기반
# 생성 일시: {TIMESTAMP}

import pytest
from unittest.mock import Mock, patch, MagicMock, AsyncMock
from {SOURCE_MODULE} import {IMPORTED_NAMES}


# ========================================
# Fixtures
# ========================================

@pytest.fixture
def mock_{DEPENDENCY_1}():
    """외부 의존성 {DEPENDENCY_1} fixture"""
    mock = Mock()
    mock.{METHOD}.return_value = {MOCK_RETURN}
    return mock


@pytest.fixture
def sample_data():
    """테스트용 샘플 데이터"""
    return {
        {SAMPLE_DATA}
    }


# ========================================
# {CLASS_OR_FUNCTION_1} 테스트
# ========================================

class Test{CLASS_OR_FUNCTION_1}:
    """{CLASS_OR_FUNCTION_1}에 대한 테스트"""

    # --- 정상 동작 (Happy Path) ---

    def test_유효한_입력이면_기대_결과를_반환한다(self, sample_data):
        """Given 유효한 입력, When 함수 호출, Then 기대 결과 반환"""
        # Given
        input_data = sample_data

        # When
        result = {FUNCTION_CALL}(input_data)

        # Then
        assert result == {EXPECTED_RESULT}

    # --- Edge Cases ---

    def test_빈_입력이면_기본값을_반환한다(self):
        """Given 빈 입력, When 함수 호출, Then 기본값 반환"""
        # Given
        input_data = {EMPTY_INPUT}

        # When
        result = {FUNCTION_CALL}(input_data)

        # Then
        assert result == {DEFAULT_VALUE}

    def test_None_입력이면_예외를_발생시킨다(self):
        """Given None 입력, When 함수 호출, Then ValueError 발생"""
        # Given & When & Then
        with pytest.raises(ValueError, match="{ERROR_MESSAGE}"):
            {FUNCTION_CALL}(None)

    @pytest.mark.parametrize("input_val,expected", [
        ({PARAM_1}, {EXPECTED_1}),
        ({PARAM_2}, {EXPECTED_2}),
        ({PARAM_3}, {EXPECTED_3}),
    ])
    def test_다양한_입력에_대해_올바른_결과를_반환한다(self, input_val, expected):
        """Given 다양한 입력, When 함수 호출, Then 각각 올바른 결과"""
        assert {FUNCTION_CALL}(input_val) == expected

    # --- 에러 시나리오 ---

    @patch("{SOURCE_MODULE}.{DEPENDENCY}")
    def test_외부_서비스_실패시_적절한_에러를_반환한다(self, mock_dep):
        """Given 외부 서비스 실패, When 함수 호출, Then 적절한 에러 발생"""
        # Given
        mock_dep.{METHOD}.side_effect = ConnectionError("연결 실패")

        # When & Then
        with pytest.raises(ConnectionError):
            {FUNCTION_CALL}({ERROR_INPUT})

    # TODO: 비동기 시나리오 테스트
    # TODO: 타임아웃 시나리오 테스트


# ========================================
# {CLASS_OR_FUNCTION_2} 테스트
# ========================================

# TODO: 위와 동일한 구조로 나머지 함수/클래스 테스트 작성
```

### JUnit 5 (Java) 스켈레톤

```java
// {TEST_FILE_PATH}
// 자동 생성된 테스트 스켈레톤 - 커버리지 분석 기반
// 생성 일시: {TIMESTAMP}

package {PACKAGE};

import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;
import static org.mockito.ArgumentMatchers.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("{CLASS_NAME} 테스트")
class {CLASS_NAME}Test {

    @Mock
    private {DEPENDENCY_TYPE} {DEPENDENCY_NAME};

    @InjectMocks
    private {CLASS_NAME} {INSTANCE_NAME};

    // ========================================
    // 공통 설정
    // ========================================

    @BeforeEach
    void setUp() {
        // 각 테스트 전 초기화
        {SETUP_CODE}
    }

    @AfterEach
    void tearDown() {
        // 각 테스트 후 정리
        {CLEANUP_CODE}
    }

    // ========================================
    // {METHOD_NAME_1} 테스트
    // ========================================

    @Nested
    @DisplayName("{METHOD_NAME_1} 메서드")
    class {METHOD_NAME_1}Test {

        @Test
        @DisplayName("유효한 입력이 주어지면 기대 결과를 반환해야 한다")
        void shouldReturnExpectedResult_WhenValidInput() {
            // Given
            {INPUT_TYPE} input = {INPUT_VALUE};
            when({DEPENDENCY_NAME}.{DEP_METHOD}(any())).thenReturn({MOCK_RETURN});

            // When
            {RETURN_TYPE} result = {INSTANCE_NAME}.{METHOD_NAME_1}(input);

            // Then
            assertNotNull(result);
            assertEquals({EXPECTED_VALUE}, result.{GETTER}());
            verify({DEPENDENCY_NAME}).{DEP_METHOD}(any());
        }

        @ParameterizedTest
        @NullAndEmptySource
        @DisplayName("null 또는 빈 입력이 주어지면 IllegalArgumentException을 던져야 한다")
        void shouldThrowException_WhenNullOrEmptyInput(String input) {
            // Given & When & Then
            assertThrows(IllegalArgumentException.class,
                () -> {INSTANCE_NAME}.{METHOD_NAME_1}(input));
        }

        @ParameterizedTest
        @ValueSource(strings = {{EDGE_VALUES}})
        @DisplayName("경계값 입력에 대해 올바르게 동작해야 한다")
        void shouldHandleBoundaryValues(String input) {
            // Given
            when({DEPENDENCY_NAME}.{DEP_METHOD}(any())).thenReturn({MOCK_RETURN});

            // When
            {RETURN_TYPE} result = {INSTANCE_NAME}.{METHOD_NAME_1}(input);

            // Then
            assertNotNull(result);
        }

        @Test
        @DisplayName("의존성이 실패하면 ServiceException을 던져야 한다")
        void shouldThrowServiceException_WhenDependencyFails() {
            // Given
            when({DEPENDENCY_NAME}.{DEP_METHOD}(any()))
                .thenThrow(new RuntimeException("연결 실패"));

            // When & Then
            ServiceException exception = assertThrows(ServiceException.class,
                () -> {INSTANCE_NAME}.{METHOD_NAME_1}({INPUT_VALUE}));

            assertTrue(exception.getMessage().contains("연결 실패"));
        }
    }

    // ========================================
    // {METHOD_NAME_2} 테스트
    // ========================================

    // TODO: 위와 동일한 구조로 나머지 메서드 테스트 작성
}
```

### Go testing 스켈레톤

```go
// {TEST_FILE_PATH}
// 자동 생성된 테스트 스켈레톤 - 커버리지 분석 기반
// 생성 일시: {TIMESTAMP}

package {PACKAGE}

import (
    "errors"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"
    "github.com/stretchr/testify/require"
)

// ========================================
// Mock 정의
// ========================================

type Mock{DEPENDENCY} struct {
    mock.Mock
}

func (m *Mock{DEPENDENCY}) {METHOD}({PARAMS}) ({RETURNS}) {
    args := m.Called({PARAM_NAMES})
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).({RETURN_TYPE}), args.Error(1)
}

// ========================================
// {FUNCTION_NAME_1} 테스트
// ========================================

func Test{FUNCTION_NAME_1}(t *testing.T) {
    t.Run("유효한 입력이 주어지면 기대 결과를 반환해야 한다", func(t *testing.T) {
        // Given
        mockDep := new(Mock{DEPENDENCY})
        mockDep.On("{METHOD}", {MOCK_ARGS}).Return({MOCK_RETURN}, nil)

        // When
        result, err := {FUNCTION_NAME_1}(mockDep, {INPUT})

        // Then
        require.NoError(t, err)
        assert.Equal(t, {EXPECTED}, result.{FIELD})
        mockDep.AssertExpectations(t)
    })

    t.Run("빈 입력이 주어지면 에러를 반환해야 한다", func(t *testing.T) {
        // Given
        mockDep := new(Mock{DEPENDENCY})

        // When
        result, err := {FUNCTION_NAME_1}(mockDep, "")

        // Then
        assert.Nil(t, result)
        assert.EqualError(t, err, "입력값이 비어있습니다")
    })

    t.Run("의존성이 실패하면 적절한 에러를 반환해야 한다", func(t *testing.T) {
        // Given
        mockDep := new(Mock{DEPENDENCY})
        mockDep.On("{METHOD}", {MOCK_ARGS}).Return(nil, errors.New("연결 실패"))

        // When
        result, err := {FUNCTION_NAME_1}(mockDep, {INPUT})

        // Then
        assert.Nil(t, result)
        assert.Contains(t, err.Error(), "연결 실패")
    })
}

// 테이블 기반 테스트
func Test{FUNCTION_NAME_1}_TableDriven(t *testing.T) {
    tests := []struct {
        name        string
        input       {INPUT_TYPE}
        mockReturn  {RETURN_TYPE}
        mockErr     error
        expected    {EXPECTED_TYPE}
        expectError bool
        errorMsg    string
    }{
        {
            name:       "유효한 입력",
            input:      {VALID_INPUT},
            mockReturn: {VALID_RETURN},
            mockErr:    nil,
            expected:   {EXPECTED_RESULT},
            expectError: false,
        },
        {
            name:        "빈 입력",
            input:       {EMPTY_INPUT},
            expectError: true,
            errorMsg:    "입력값이 비어있습니다",
        },
        {
            name:        "의존성 실패",
            input:       {VALID_INPUT},
            mockErr:     errors.New("연결 실패"),
            expectError: true,
            errorMsg:    "연결 실패",
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Given
            mockDep := new(Mock{DEPENDENCY})
            if tt.mockReturn != nil || tt.mockErr != nil {
                mockDep.On("{METHOD}", mock.Anything).Return(tt.mockReturn, tt.mockErr)
            }

            // When
            result, err := {FUNCTION_NAME_1}(mockDep, tt.input)

            // Then
            if tt.expectError {
                assert.Error(t, err)
                if tt.errorMsg != "" {
                    assert.Contains(t, err.Error(), tt.errorMsg)
                }
            } else {
                require.NoError(t, err)
                assert.Equal(t, tt.expected, result)
            }
        })
    }
}

// ========================================
// {FUNCTION_NAME_2} 테스트
// ========================================

// TODO: 위와 동일한 구조로 나머지 함수 테스트 작성
```

---

## 누락 테스트 경고 메시지

### 테스트 파일 미존재

```
[Critical] 테스트 파일이 존재하지 않습니다

  소스 파일: {SOURCE_FILE}
  예상 테스트 파일: {EXPECTED_TEST_FILE}

  이 파일에는 {FUNCTION_COUNT}개의 공개 함수/메서드가 있으며,
  어떤 테스트도 존재하지 않습니다.

  제안: {EXPECTED_TEST_FILE} 파일을 생성하여 다음 함수를 테스트하세요:
  {FUNCTION_LIST}

  테스트 스켈레톤을 자동 생성하시겠습니까?
```

### 함수 테스트 미존재

```
[High] 함수에 대한 테스트가 없습니다

  소스 파일: {SOURCE_FILE}
  함수: {FUNCTION_NAME}()
  테스트 파일: {TEST_FILE}

  이 함수는 {PARAM_COUNT}개의 매개변수를 받고 {RETURN_TYPE}을 반환합니다.
  {BRANCH_COUNT}개의 분기가 있지만 테스트가 존재하지 않습니다.

  필요한 테스트:
  - Happy path: 유효한 입력으로 정상 동작 확인
  - Edge case: {EDGE_CASE_SUGGESTIONS}
  - Error case: {ERROR_CASE_SUGGESTIONS}
```

### Happy Path 테스트 누락

```
[High] Happy path 테스트가 누락되었습니다

  소스 파일: {SOURCE_FILE}
  함수: {FUNCTION_NAME}()

  이 함수의 기본 정상 동작을 검증하는 테스트가 없습니다.
  가장 일반적인 사용 시나리오에 대한 테스트를 추가하세요.

  제안 테스트:
  - 유효한 입력 ({EXAMPLE_INPUT})으로 호출 시 {EXPECTED_OUTPUT} 반환 확인
```

### Edge Case 누락

```
[Medium] Edge case 테스트가 누락되었습니다

  소스 파일: {SOURCE_FILE}
  함수: {FUNCTION_NAME}()
  매개변수: {PARAM_NAME} ({PARAM_TYPE})

  다음 Edge case에 대한 테스트가 없습니다:
  {MISSING_EDGE_CASES}

  각 Edge case에 대해 함수가 올바르게 동작하는지 확인하세요.
```

### 분기 커버리지 누락

```
[Medium] 분기 커버리지가 부족합니다

  소스 파일: {SOURCE_FILE}
  라인 {LINE_NUMBER}: {BRANCH_TYPE}

  {TESTED_BRANCHES}개 분기 중 {UNTESTED_COUNT}개가 테스트되지 않았습니다:
  {UNTESTED_BRANCH_LIST}

  누락된 분기에 대한 테스트를 추가하세요.
```

### 에러 시나리오 누락

```
[High] 에러 시나리오 테스트가 누락되었습니다

  소스 파일: {SOURCE_FILE}
  함수: {FUNCTION_NAME}()

  이 함수에는 다음과 같은 에러 처리 경로가 있지만 테스트되지 않았습니다:
  - 라인 {LINE}: {ERROR_DESCRIPTION}

  에러 시나리오 테스트를 추가하여 예외 처리가 올바르게
  동작하는지 확인하세요.
```

### 비동기 코드 테스트 누락

```
[High] 비동기 코드 테스트가 누락되었습니다

  소스 파일: {SOURCE_FILE}
  함수: {FUNCTION_NAME}() (async)

  이 함수는 비동기로 동작하지만 다음 시나리오에 대한 테스트가 없습니다:
  - Promise 정상 resolve 시나리오
  - Promise reject 시나리오
  - 타임아웃 시나리오

  async/await을 사용한 테스트를 추가하세요.
```

### 모킹 누락

```
[Medium] 외부 의존성 모킹이 누락되었습니다

  소스 파일: {SOURCE_FILE}
  함수: {FUNCTION_NAME}()
  의존성: {DEPENDENCY_NAME}

  이 함수는 {DEPENDENCY_NAME}에 의존하지만, 테스트에서
  해당 의존성이 모킹되지 않았습니다.

  실제 외부 서비스를 호출하면 테스트가 불안정(flaky)해질 수 있습니다.
  {DEPENDENCY_NAME}을 모킹하여 테스트를 격리하세요.
```

### 테스트 품질 경고

```
[Low] 테스트 이름이 명확하지 않습니다

  테스트 파일: {TEST_FILE}
  테스트: {TEST_NAME}

  테스트 이름에서 테스트 대상, 조건, 기대 결과를 알 수 없습니다.
  다음 형식을 권장합니다:
  - "[함수명]_[조건]_[기대결과]"
  - 또는 "~이면 ~해야 한다"

  예시: "{SUGGESTED_NAME}"
```

```
[Low] assertion이 구체적이지 않습니다

  테스트 파일: {TEST_FILE}
  테스트: {TEST_NAME}
  라인 {LINE}: {CURRENT_ASSERTION}

  더 구체적인 assertion을 사용하세요:
  변경 전: {CURRENT_ASSERTION}
  변경 후: {SUGGESTED_ASSERTION}
```

---

## Given-When-Then 패턴 템플릿

### 기본 패턴

```
Given [사전 조건/준비]
When  [테스트 대상 동작 실행]
Then  [기대하는 결과 검증]
```

### 패턴별 템플릿

#### 단순 함수 테스트

```
Given 유효한 입력 데이터 {INPUT}이 준비되었을 때
When  {FUNCTION_NAME}({INPUT})을 호출하면
Then  {EXPECTED_RESULT}를 반환해야 한다
```

#### Edge Case 테스트

```
Given {EDGE_CONDITION}인 입력 {INPUT}이 준비되었을 때
When  {FUNCTION_NAME}({INPUT})을 호출하면
Then  {EXPECTED_BEHAVIOR}해야 한다 (예: 기본값 반환 / 에러 발생)
```

#### 에러 시나리오 테스트

```
Given {ERROR_CONDITION}인 상태에서
When  {FUNCTION_NAME}({INPUT})을 호출하면
Then  {ERROR_TYPE} 에러가 발생해야 하고
And   에러 메시지에 "{ERROR_MESSAGE_PATTERN}"이 포함되어야 한다
```

#### 모킹 의존성 테스트

```
Given {DEPENDENCY}가 {MOCK_BEHAVIOR}으로 모킹되었을 때
And   {ADDITIONAL_SETUP}
When  {FUNCTION_NAME}({INPUT})을 호출하면
Then  {EXPECTED_RESULT}를 반환해야 하고
And   {DEPENDENCY}.{METHOD}가 {EXPECTED_ARGS}로 호출되어야 한다
```

#### 비동기 함수 테스트

```
Given {ASYNC_SETUP}이 준비되었을 때
When  await {FUNCTION_NAME}({INPUT})을 호출하면
Then  Promise가 {EXPECTED_VALUE}로 resolve되어야 한다
```

```
Given {FAILURE_SETUP}인 상태에서
When  await {FUNCTION_NAME}({INPUT})을 호출하면
Then  Promise가 {ERROR_TYPE}으로 reject되어야 한다
```

#### 상태 변경 테스트

```
Given {INITIAL_STATE}인 상태에서
When  {ACTION}을 수행하면
Then  상태가 {EXPECTED_STATE}로 변경되어야 하고
And   {SIDE_EFFECT}가 발생해야 한다
```

### 언어별 Given-When-Then 코드 패턴

#### TypeScript (Jest)

```typescript
it('{GIVEN} {WHEN} {THEN}', () => {
  // Given
  const input = /* 사전 조건 설정 */;
  const mockDep = jest.fn().mockReturnValue(/* 모킹 */);

  // When
  const result = targetFunction(input);

  // Then
  expect(result).toEqual(/* 기대 결과 */);
  expect(mockDep).toHaveBeenCalledWith(/* 기대 인자 */);
});
```

#### Python (pytest)

```python
def test_{given}_{when}_{then}(self):
    """{GIVEN}, {WHEN}, {THEN}"""
    # Given
    input_data = ...  # 사전 조건 설정

    # When
    result = target_function(input_data)

    # Then
    assert result == expected_result
```

#### Java (JUnit 5)

```java
@Test
@DisplayName("{GIVEN}이 주어지면 {WHEN}할 때 {THEN}해야 한다")
void should{Then}_When{When}() {
    // Given
    var input = /* 사전 조건 설정 */;
    when(dependency.method(any())).thenReturn(/* 모킹 */);

    // When
    var result = target.method(input);

    // Then
    assertNotNull(result);
    assertEquals(expected, result);
    verify(dependency).method(any());
}
```

#### Go (testing)

```go
t.Run("{GIVEN}이 주어지면 {THEN}해야 한다", func(t *testing.T) {
    // Given
    input := /* 사전 조건 설정 */
    mockDep := new(MockDependency)
    mockDep.On("Method", mock.Anything).Return(/* 모킹 */)

    // When
    result, err := TargetFunction(mockDep, input)

    // Then
    require.NoError(t, err)
    assert.Equal(t, expected, result)
    mockDep.AssertExpectations(t)
})
```

---

## 테스트 품질 점수 카드

### 점수 카드 전체 템플릿

```
============================================================
          테스트 품질 점수 카드
============================================================

프로젝트: {PROJECT_NAME}
분석 대상: {TARGET_FILES}
분석 일시: {TIMESTAMP}

============================================================
총점: {TOTAL_SCORE} / 100  {GRADE}
============================================================

┌─────────────────────────────────────────────┐
│ 항목              │ 점수    │ 상태          │
├───────────────────┼─────────┼──────────────┤
│ 구조 품질          │ {S}/25  │ {STATUS}     │
│ 커버리지 품질      │ {C}/30  │ {STATUS}     │
│ Assertion 품질    │ {A}/20  │ {STATUS}     │
│ 모킹 품질          │ {M}/15  │ {STATUS}     │
│ 유지보수 품질      │ {R}/10  │ {STATUS}     │
└───────────────────┴─────────┴──────────────┘

============================================================
세부 평가
============================================================

구조 품질 ({STRUCTURE_SCORE}/25):
  [O] 테스트 그룹화 (describe/class별): {SCORE}/5
  [O] 설정/해제 적절성: {SCORE}/5
  [{STATUS}] 테스트 이름 명확성: {SCORE}/5
  [{STATUS}] 테스트 독립성: {SCORE}/5
  [{STATUS}] 테스트 크기 적절성: {SCORE}/5

커버리지 품질 ({COVERAGE_SCORE}/30):
  [{STATUS}] Happy path 커버리지: {SCORE}/10
  [{STATUS}] Edge case 커버리지: {SCORE}/8
  [{STATUS}] Error path 커버리지: {SCORE}/7
  [{STATUS}] 분기 커버리지: {SCORE}/5

Assertion 품질 ({ASSERTION_SCORE}/20):
  [{STATUS}] 구체적 값 비교: {SCORE}/5
  [{STATUS}] 에러 타입 검증: {SCORE}/5
  [{STATUS}] assertion 완전성: {SCORE}/5
  [{STATUS}] 비동기 assertion: {SCORE}/5

모킹 품질 ({MOCK_SCORE}/15):
  [{STATUS}] 최소 모킹 원칙: {SCORE}/5
  [{STATUS}] 모킹 복원: {SCORE}/5
  [{STATUS}] 실제적 데이터: {SCORE}/5

유지보수 품질 ({MAINTAIN_SCORE}/10):
  [{STATUS}] DRY 원칙: {SCORE}/3
  [{STATUS}] 가독성: {SCORE}/4
  [{STATUS}] 안정성 (플레이키 없음): {SCORE}/3

============================================================
개선 권장사항
============================================================

{RECOMMENDATIONS}

============================================================
등급 기준
============================================================

  90-100: A (우수) - 테스트 품질이 매우 높음
  80-89:  B (양호) - 대부분의 영역이 잘 되어 있음
  70-79:  C (보통) - 기본적인 테스트는 있으나 개선 필요
  60-69:  D (미흡) - 상당한 개선이 필요함
  0-59:   F (부족) - 테스트가 심각하게 부족함
============================================================
```

### 간략 점수 카드

```
테스트 품질: {TOTAL_SCORE}/100 ({GRADE})
━━━━━━━━━━━━━━━━━━━━━━━━━━
구조:      {BAR_1} {S}/25
커버리지:   {BAR_2} {C}/30
Assertion: {BAR_3} {A}/20
모킹:      {BAR_4} {M}/15
유지보수:   {BAR_5} {R}/10

최우선 개선: {TOP_IMPROVEMENT}
```

### 점수 시각화 바

점수에 따라 다음과 같이 시각화합니다:

```
80-100%: [##########] 우수
60-79%:  [######----] 양호
40-59%:  [####------] 보통
20-39%:  [##--------] 미흡
0-19%:   [----------] 부족
```

### 파일별 점수 카드

```
파일별 테스트 품질:

{SOURCE_FILE_1}:
  점수: {SCORE}/100 ({GRADE})
  테스트 파일: {TEST_FILE}
  함수 커버리지: {M}/{N} ({PCT}%)
  주요 이슈: {TOP_ISSUE}

{SOURCE_FILE_2}:
  점수: {SCORE}/100 ({GRADE})
  테스트 파일: {TEST_FILE}
  함수 커버리지: {M}/{N} ({PCT}%)
  주요 이슈: {TOP_ISSUE}
```

### 개선 권장사항 템플릿

```
개선 권장사항 (우선순위 순):

1. [Critical] {RECOMMENDATION_1}
   현재: {CURRENT_STATE}
   목표: {TARGET_STATE}
   영향: 점수 +{POINTS}점

2. [High] {RECOMMENDATION_2}
   현재: {CURRENT_STATE}
   목표: {TARGET_STATE}
   영향: 점수 +{POINTS}점

3. [Medium] {RECOMMENDATION_3}
   현재: {CURRENT_STATE}
   목표: {TARGET_STATE}
   영향: 점수 +{POINTS}점
```
