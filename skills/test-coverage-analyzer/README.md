# Test Coverage Analyzer

변경된 코드에 대한 테스트 커버리지를 자동 분석하고, 누락된 테스트 케이스를 식별하여 테스트 코드 스켈레톤을 생성하는 Claude Code 커스텀 스킬입니다.

## 스킬 소개

코드를 작성하거나 수정한 후, 해당 코드에 대한 테스트가 충분한지 확인하는 것은 소프트웨어 품질의 핵심입니다. 이 스킬은 다음과 같은 방식으로 테스트 커버리지를 종합적으로 분석합니다:

- **변경 코드 자동 감지**: `git diff`를 통해 변경된 소스 파일을 자동으로 감지
- **소스-테스트 파일 매핑**: 소스 파일에 대응하는 테스트 파일을 언어별 규칙에 따라 자동 매핑
- **커버리지 갭 분석**: 함수/메서드별 테스트 존재 여부와 분기 커버리지를 분석
- **누락 테스트 식별**: Happy path, Edge case, Error scenario 등 누락된 테스트를 자동 식별
- **테스트 스켈레톤 생성**: Given-When-Then 패턴을 적용한 테스트 코드 스켈레톤 자동 생성
- **테스트 품질 평가**: 기존 테스트의 이름 명확성, assertion 적절성, 독립성 등을 평가

## 지원 언어 및 프레임워크

### 테스트 프레임워크

| 언어 | 테스트 프레임워크 | 모킹 라이브러리 |
|------|----------------|---------------|
| JavaScript | Jest, Vitest, Mocha + Chai | jest.fn(), vi.fn(), sinon |
| TypeScript | Jest, Vitest, Mocha + Chai | jest.fn(), vi.fn(), sinon |
| Python | pytest, unittest | unittest.mock, pytest-mock |
| Java | JUnit 5, TestNG | Mockito, EasyMock |
| Go | testing, testify | testify/mock, gomock |
| Kotlin | JUnit 5, Kotest | MockK |
| Rust | built-in #[test] | mockall |
| C# | xUnit, NUnit, MSTest | Moq, NSubstitute |

### 커버리지 도구

| 언어 | 커버리지 도구 | 실행 명령어 |
|------|-------------|-----------|
| JavaScript/TypeScript | Jest --coverage, c8, istanbul | `npx jest --coverage` |
| Python | pytest-cov, coverage.py | `pytest --cov=src` |
| Java | JaCoCo, Cobertura | `mvn jacoco:report` |
| Go | go test -cover | `go test -coverprofile=coverage.out ./...` |

## 사용 방법

다음과 같이 요청하면 스킬이 활성화됩니다:

- "테스트 커버리지 분석해줘"
- "테스트 누락 확인해줘"
- "이 코드에 대한 테스트 생성해줘"
- "coverage 분석"
- "어떤 테스트가 필요해?"
- "변경된 코드의 테스트 확인"

## 실제 사용 예시

### 예시 1: 변경 코드에 대한 테스트 분석

사용자 요청:
> "방금 수정한 코드에 대해 테스트 커버리지 분석해줘"

분석 과정:
```
1. git diff로 변경 파일 감지:
   - src/services/userService.ts (수정)
   - src/utils/validator.ts (신규)
   - src/controllers/userController.ts (수정)

2. 테스트 파일 매핑:
   - src/services/userService.ts → src/services/__tests__/userService.test.ts (존재)
   - src/utils/validator.ts → src/utils/__tests__/validator.test.ts (미존재!)
   - src/controllers/userController.ts → src/controllers/__tests__/userController.test.ts (존재)

3. 커버리지 분석 결과:
   - userService.ts: 3/5 함수 테스트됨 (60%)
   - validator.ts: 테스트 파일 없음 (0%)
   - userController.ts: 2/3 함수 테스트됨 (67%)
```

분석 결과 보고서:
```
============================================================
          테스트 커버리지 분석 보고서
============================================================

프로젝트: my-app
분석 범위: git diff (최근 변경)
분석 일시: 2026-02-23 14:30

------------------------------------------------------------
파일별 커버리지 현황
------------------------------------------------------------

파일                              │ 테스트 파일    │ 함수    │ 커버리지
─────────────────────────────────┼──────────────┼────────┼─────────
src/services/userService.ts      │ 존재          │ 3/5    │ 60%
src/utils/validator.ts           │ 미존재        │ 0/4    │ 0%
src/controllers/userController.ts│ 존재          │ 2/3    │ 67%

전체 커버리지: 5/12 함수 (42%)
목표 대비: 42% / 80% (미달)

------------------------------------------------------------
누락된 테스트 케이스
------------------------------------------------------------

[Critical] src/utils/validator.ts
  - 테스트 파일이 존재하지 않습니다
  - 제안: src/utils/__tests__/validator.test.ts 생성

[High] src/services/userService.ts
  - updateUser(): 테스트 없음
  - deleteUser(): 테스트 없음
  - createUser(): Edge case 누락 (빈 이메일, 중복 이메일)

[Medium] src/controllers/userController.ts
  - handleDelete(): 테스트 없음
  - handleCreate(): 에러 시나리오 테스트 누락

------------------------------------------------------------
테스트 품질 점수: 62/100
------------------------------------------------------------
```

### 예시 2: 테스트 스켈레톤 자동 생성

사용자 요청:
> "validator.ts에 대한 테스트 생성해줘"

생성된 테스트 코드:
```typescript
// src/utils/__tests__/validator.test.ts
import { describe, it, expect } from '@jest/globals';
import {
  validateEmail,
  validatePassword,
  validateUsername,
  sanitizeInput,
} from '../validator';

describe('validateEmail', () => {
  describe('정상 동작', () => {
    it('유효한 이메일 형식이면 true를 반환해야 한다', () => {
      // Given
      const email = 'user@example.com';

      // When
      const result = validateEmail(email);

      // Then
      expect(result).toBe(true);
    });
  });

  describe('Edge Cases', () => {
    it('빈 문자열이면 false를 반환해야 한다', () => {
      expect(validateEmail('')).toBe(false);
    });

    it('@ 기호가 없으면 false를 반환해야 한다', () => {
      expect(validateEmail('userexample.com')).toBe(false);
    });

    it('도메인이 없으면 false를 반환해야 한다', () => {
      expect(validateEmail('user@')).toBe(false);
    });

    it('null이 주어지면 false를 반환해야 한다', () => {
      expect(validateEmail(null as any)).toBe(false);
    });
  });
});

describe('validatePassword', () => {
  describe('정상 동작', () => {
    it('유효한 비밀번호면 true를 반환해야 한다', () => {
      // Given
      const password = 'SecureP@ss123';

      // When
      const result = validatePassword(password);

      // Then
      expect(result).toBe(true);
    });
  });

  describe('Edge Cases', () => {
    it('8자 미만이면 false를 반환해야 한다', () => {
      expect(validatePassword('Ab1!')).toBe(false);
    });

    it('빈 문자열이면 false를 반환해야 한다', () => {
      expect(validatePassword('')).toBe(false);
    });

    it('특수문자가 없으면 false를 반환해야 한다', () => {
      expect(validatePassword('SecurePass123')).toBe(false);
    });
  });
});

// ... 나머지 함수에 대한 테스트 계속
```

### 예시 3: Python 프로젝트 분석

사용자 요청:
> "테스트 누락 확인해줘"

```
============================================================
          테스트 커버리지 분석 보고서
============================================================

프로젝트: my-python-app
분석 범위: git diff main...HEAD
분석 일시: 2026-02-23 14:30

------------------------------------------------------------
파일별 커버리지 현황
------------------------------------------------------------

파일                              │ 테스트 파일    │ 함수    │ 커버리지
─────────────────────────────────┼──────────────┼────────┼─────────
src/services/auth.py             │ 존재          │ 4/6    │ 67%
src/utils/crypto.py              │ 존재          │ 2/3    │ 67%
src/models/user.py               │ 미존재        │ 0/5    │ 0%

pytest-cov 결과 (기존 테스트 실행):
  라인 커버리지: 54%
  분기 커버리지: 38%

누락 테스트:
  [Critical] src/models/user.py - 테스트 파일 미존재
  [High] src/services/auth.py - verify_token(): 만료 토큰 테스트 누락
  [High] src/services/auth.py - refresh_token(): 테스트 없음
  [Medium] src/utils/crypto.py - decrypt(): 잘못된 키 에러 테스트 누락

생성 가능한 테스트:
  1. tests/models/test_user.py (신규 파일)
  2. tests/services/test_auth.py에 2개 테스트 추가
  3. tests/utils/test_crypto.py에 1개 테스트 추가

테스트를 생성하시겠습니까? (전체/선택/취소)
```

## 커버리지 보고서 형식

보고서는 다음 섹션으로 구성됩니다:

1. **헤더**: 프로젝트명, 분석 범위, 분석 일시
2. **파일별 커버리지 현황**: 각 소스 파일의 테스트 존재 여부와 함수별 커버리지
3. **누락 테스트 목록**: 심각도별 분류 (Critical / High / Medium)
4. **커버리지 도구 데이터**: 가능한 경우 정량적 커버리지 수치
5. **테스트 품질 점수**: 기존 테스트의 품질 평가 (100점 만점)
6. **생성 제안**: 생성 가능한 테스트 스켈레톤 목록

## 동작 방식

```
사용자 요청
    │
    v
변경 파일 감지 (git diff)
    │
    v
소스-테스트 파일 매핑
    │
    ├─ 테스트 파일 존재 → 커버리지 분석
    │                        │
    │                        v
    │                   함수별 테스트 확인
    │                        │
    │                        v
    │                   누락 테스트 식별
    │
    └─ 테스트 파일 미존재 → 경고 + 전체 스켈레톤 생성 제안
                              │
                              v
                       테스트 스켈레톤 생성
                              │
                              v
                       커버리지 보고서 출력
                              │
                              v
                       사용자 확인 요청
                              │
                              v
                       선택된 테스트 생성
```

## 관련 문서

- [SKILL.md](SKILL.md) - Claude Code용 상세 사용 지침
- [references/analysis-guide.md](references/analysis-guide.md) - 분석 방법론 및 도구 가이드
- [references/templates.md](references/templates.md) - 보고서 및 테스트 스켈레톤 템플릿
