---
name: refactor-advisor
description: 코드베이스를 분석하여 리팩토링이 필요한 포인트를 식별하고, 구체적인 개선 방안과 우선순위를 제시하는 스킬. 사용자가 "리팩토링 분석", "코드 품질 분석", "코드 스멜 찾아줘", "refactor", "복잡도 분석", "중복 코드 탐지", "네이밍 검사", "디자인 패턴 제안" 등을 요청할 때 자동으로 활성화됩니다.
---

# Refactor Advisor

코드베이스를 정적 분석하여 리팩토링이 필요한 포인트를 식별하고, Before/After 코드 예시와 함께 구체적인 개선 방안을 우선순위별로 제시합니다.

## Quick Start

사용자 요청에 따라 분석 범위를 결정하고 다음 워크플로우를 실행합니다.

1. **분석 범위 결정**:
   - 사용자가 특정 파일/디렉토리를 지정한 경우 해당 범위만 분석
   - "전체 분석" 요청 시 프로젝트 루트부터 전체 스캔
   - 지정 없이 "리팩토링 분석해줘" 요청 시 변경된 파일(git diff) 또는 현재 디렉토리 분석
   ```bash
   # 변경된 파일 목록 확인
   git diff --name-only HEAD~5
   # 또는 특정 디렉토리의 파일 목록
   find src/ -type f \( -name "*.ts" -o -name "*.js" -o -name "*.py" -o -name "*.java" -o -name "*.go" \)
   ```

2. **파일별 정적 분석 실행**: 각 파일을 읽고 아래 5가지 관점에서 분석

3. **복잡도 메트릭 수집**: Cyclomatic/Cognitive Complexity, 함수 길이, 클래스 크기, 중첩 깊이

4. **코드 스멜 패턴 매칭**: God Class, Feature Envy, Long Parameter List 등 8가지 스멜 탐지

5. **중복 코드 탐지**: 유사 코드 블록 식별 및 공통 로직 추출 제안

6. **개선안 생성**: Before/After 코드 예시 포함

7. **우선순위 매트릭스로 정렬**: 영향도(Impact) x 난이도(Effort) 기준

8. **리팩토링 리포트 출력**: [references/templates.md](references/templates.md) 형식에 따라 출력

## 1. 코드 복잡도 분석

코드의 구조적 복잡성을 정량적으로 측정하여 리팩토링 대상을 식별합니다.

### 1.1 Cyclomatic Complexity (순환 복잡도)

함수 내 독립 실행 경로의 수를 측정합니다. 분기문(if, else if, switch case, for, while, catch, &&, ||, ?)마다 +1을 부여하고, 기본값 1에서 시작합니다.

**임계값 기준**:

| 복잡도 | 위험도 | 조치 |
|--------|--------|------|
| 1-10 | 낮음 (양호) | 유지 |
| 11-20 | 보통 (주의) | 리팩토링 권장 |
| 21-50 | 높음 (위험) | 리팩토링 필수 |
| 50+ | 매우 높음 (위험) | 즉시 분할 필요 |

**측정 방법**:
```
CC = 1 + (if 수) + (else if 수) + (case 수) + (for 수) + (while 수)
     + (catch 수) + (&& 수) + (|| 수) + (삼항연산자 수)
```

각 함수/메서드에 대해 개별적으로 측정합니다. 파일 단위가 아닌 함수 단위로 보고합니다.

### 1.2 Cognitive Complexity (인지 복잡도)

개발자가 코드를 읽고 이해하는 데 필요한 인지적 노력을 측정합니다. Cyclomatic Complexity와 달리 중첩(nesting)에 가중치를 부여합니다.

**계산 규칙**:
- 기본 분기문(if, else if, else, switch, for, while, catch): +1
- 중첩 내부의 분기문: +1 (기본) + 중첩 깊이(nesting depth)
- 연속된 논리 연산자(같은 종류): 추가 비용 없음
- 종류가 바뀌는 논리 연산자: +1
- break, continue, goto: +1
- 재귀 호출: +1

**임계값**: 함수당 Cognitive Complexity 15 이하 권장

### 1.3 함수/메서드 길이 분석

**임계값**: 30줄 (공백/주석 제외)

함수가 30줄을 초과하면 Extract Method 리팩토링 후보로 표시합니다.

```
[경고] processUserData() - 87줄 (임계값: 30줄 초과)
  → 추출 가능한 책임:
    1. 입력 검증 로직 (12-25줄) → validateUserInput()
    2. 데이터 변환 로직 (26-55줄) → transformUserData()
    3. 저장 로직 (56-87줄) → persistUser()
```

### 1.4 클래스/파일 크기 분석

**임계값**: 300줄 (공백/주석 제외)

300줄을 초과하는 클래스/파일은 Single Responsibility Principle(SRP) 위반 가능성이 높습니다. 책임 분리를 통한 분할을 제안합니다.

### 1.5 중첩 깊이(Nesting Depth) 분석

**임계값**: 4단계

중첩이 4단계를 초과하면 Early Return, Guard Clause, Extract Method 등으로 중첩을 줄일 것을 제안합니다.

```
[경고] 중첩 깊이 6단계 감지 (line 45-72):
  if → for → if → try → if → for
  → 개선안: Guard Clause로 최외곽 if 제거 + 내부 로직 Extract Method
```

## 2. 중복 코드 탐지

### 2.1 유사 코드 블록 식별

다음 기준으로 중복/유사 코드를 탐지합니다:

- **정확한 중복**: 완전히 동일한 코드 블록 (3줄 이상)
- **구조적 중복**: 변수명만 다르고 로직이 동일한 코드
- **기능적 중복**: 동일한 목적을 다른 방식으로 구현한 코드

**탐지 방법**:
1. 파일 간 유사 구조의 함수를 비교
2. 동일한 패턴의 조건문/반복문 탐지
3. 동일한 API 호출 시퀀스 감지
4. 유사한 데이터 변환 로직 비교

### 2.2 복사-붙여넣기 패턴 감지

같은 프로젝트 내에서 복사-붙여넣기로 생성된 것으로 추정되는 코드를 감지합니다.

**감지 기준**:
- 5줄 이상의 연속된 동일/유사 코드가 2곳 이상 존재
- 변수명이나 문자열 리터럴만 다른 코드 블록
- 동일한 주석이 여러 곳에 반복

### 2.3 공통 로직 추출 제안

중복이 감지되면 다음을 제안합니다:
- 공통 유틸리티 함수 추출
- 공통 베이스 클래스 또는 믹스인 생성
- 템플릿 메서드 패턴 적용
- 고차 함수(Higher-Order Function) 활용

**리포트 형식**:
```
[중복] 유사 코드 블록 감지 (유사도: 92%)
  위치 1: src/services/userService.ts:45-62
  위치 2: src/services/orderService.ts:78-95
  → 제안: 공통 로직을 src/utils/validation.ts로 추출
  → 예상 제거 라인: 약 34줄
```

## 3. 코드 스멜(Code Smell) 감지

### 3.1 God Class / God Method

**판별 기준**:
- God Class: 메서드 10개 이상 + 필드 10개 이상, 또는 300줄 초과
- God Method: 함수 50줄 이상 + 매개변수 4개 이상, 또는 Cyclomatic Complexity 20 이상

**개선 방향**: 책임별로 클래스/메서드를 분리합니다.

### 3.2 Feature Envy

**판별 기준**: 한 클래스의 메서드가 자신의 데이터보다 다른 클래스의 데이터를 더 많이 참조하는 경우

**탐지 패턴**:
- `other.getX()`, `other.getY()`, `other.getZ()`를 같은 메서드에서 3회 이상 호출
- 자신의 필드 접근보다 외부 객체 필드 접근이 많은 경우

**개선 방향**: Move Method로 해당 메서드를 데이터가 있는 클래스로 이동합니다.

### 3.3 Long Parameter List

**판별 기준**: 매개변수 5개 이상

**개선 방향**:
- Parameter Object 도입 (관련 매개변수를 하나의 객체로 묶기)
- Builder Pattern 적용
- 메서드 분할

### 3.4 Primitive Obsession

**판별 기준**: 관련된 원시 타입 변수가 그룹으로 반복 사용되는 경우

**탐지 패턴**:
- 같은 접두사를 가진 변수가 3개 이상 (예: `userEmail`, `userName`, `userAge`)
- 문자열로 표현된 상태값 (예: `"active"`, `"inactive"`)
- 원시 타입 배열로 표현된 구조화된 데이터

**개선 방향**: Value Object, Enum, 전용 클래스 도입

### 3.5 Data Clumps

**판별 기준**: 동일한 매개변수 그룹이 3곳 이상에서 반복 사용되는 경우

**탐지 패턴**:
- 동일한 3개 이상의 매개변수 조합이 여러 함수에서 반복
- 같은 필드 그룹이 여러 클래스에서 반복 선언

**개선 방향**: 반복되는 데이터 그룹을 별도 클래스/인터페이스로 추출

### 3.6 Switch Statements (반복적 switch/if-else)

**판별 기준**: 동일한 조건으로 분기하는 switch/if-else 체인이 2곳 이상에서 반복

**탐지 패턴**:
- 같은 타입/enum에 대한 switch 문이 여러 메서드에 존재
- 같은 조건의 if-else 체인이 반복
- 타입 코드에 따른 분기가 산재

**개선 방향**: Strategy Pattern, Polymorphism, Map/Dictionary 기반 디스패치

### 3.7 Dead Code (사용되지 않는 코드)

**판별 기준**: 프로젝트 내에서 참조되지 않는 코드

**탐지 대상**:
- 호출되지 않는 함수/메서드
- 사용되지 않는 변수/상수
- 사용되지 않는 import/require 문
- 주석 처리된 코드 블록 (5줄 이상)
- 도달 불가능한 코드 (unreachable code)

**개선 방향**: 제거 (버전 관리 시스템에 이력이 남으므로 안전하게 삭제 가능)

### 3.8 Magic Numbers/Strings

**판별 기준**: 의미가 명시되지 않은 리터럴 값이 코드에 직접 사용되는 경우

**탐지 패턴**:
- 숫자 리터럴이 조건문/계산식에 직접 사용 (0, 1, -1, 2 제외)
- 문자열 리터럴이 비교/분기에 직접 사용
- 같은 매직 넘버/문자열이 2곳 이상에서 반복

**개선 방향**: 명명된 상수(Named Constant) 또는 Enum으로 추출

## 4. 디자인 패턴 제안

코드 구조를 분석하여 적용 가능한 디자인 패턴을 제안합니다.

### 4.1 Strategy Pattern

**적용 조건**:
- 동일한 인터페이스로 여러 알고리즘/동작이 교체 가능한 경우
- switch/if-else로 동작을 선택하는 패턴이 반복되는 경우
- 런타임에 동작을 변경해야 하는 경우

**Before**:
```typescript
function calculateDiscount(type: string, amount: number): number {
  if (type === "vip") return amount * 0.2;
  else if (type === "member") return amount * 0.1;
  else if (type === "coupon") return amount * 0.05;
  else return 0;
}
```

**After**:
```typescript
interface DiscountStrategy {
  calculate(amount: number): number;
}

class VipDiscount implements DiscountStrategy {
  calculate(amount: number) { return amount * 0.2; }
}

class MemberDiscount implements DiscountStrategy {
  calculate(amount: number) { return amount * 0.1; }
}

const strategies: Record<string, DiscountStrategy> = {
  vip: new VipDiscount(),
  member: new MemberDiscount(),
};

function calculateDiscount(type: string, amount: number): number {
  return strategies[type]?.calculate(amount) ?? 0;
}
```

### 4.2 Factory / Builder Pattern

**적용 조건**:
- 객체 생성 로직이 복잡하고 여러 곳에서 반복되는 경우 (Factory)
- 생성자에 매개변수가 많고 선택적 매개변수가 있는 경우 (Builder)
- 조건에 따라 다른 서브클래스를 생성하는 경우 (Factory Method)

**Builder 적용 전**:
```typescript
const config = new ServerConfig(
  "localhost", 8080, true, false, 30000,
  "utf-8", null, true, 100, "production"
);
```

**Builder 적용 후**:
```typescript
const config = ServerConfig.builder()
  .host("localhost")
  .port(8080)
  .ssl(true)
  .timeout(30000)
  .maxConnections(100)
  .environment("production")
  .build();
```

### 4.3 State Pattern

**적용 조건**:
- 객체가 여러 상태를 가지며 상태에 따라 동작이 달라지는 경우
- 상태 전이 로직이 복잡한 if-else/switch로 구현된 경우
- 새로운 상태 추가 시 기존 코드를 많이 수정해야 하는 경우

### 4.4 Adapter Pattern

**적용 조건**:
- 기존 인터페이스와 새로운 인터페이스가 호환되지 않는 경우
- 서드파티 라이브러리를 래핑하여 일관된 인터페이스를 제공해야 하는 경우
- 레거시 코드와 새 코드를 통합해야 하는 경우

## 5. 네이밍 컨벤션 분석

### 5.1 일관성 검사

프로젝트 내 네이밍 패턴의 일관성을 검사합니다.

**검사 항목**:
- 함수명: camelCase vs snake_case 혼용 여부
- 클래스명: PascalCase 준수 여부
- 상수명: UPPER_SNAKE_CASE 준수 여부
- 파일명: kebab-case / camelCase / PascalCase 일관성
- Boolean 변수: is/has/can/should 접두사 사용 여부

### 5.2 약어 사용 감지

**탐지 대상**: 의미가 불명확한 약어

**예시**:
- `usr` → `user`
- `mgr` → `manager`
- `cnt` → `count`
- `btn` → `button`
- `msg` → `message`
- `val` → `value`
- `tmp` → 보다 구체적인 이름으로 변경

**예외**: 도메인에서 널리 통용되는 약어 (예: `id`, `url`, `api`, `db`, `io`)

### 5.3 의미 없는 이름 감지

**탐지 대상**: 코드의 의도를 전달하지 못하는 이름

| 문제 이름 | 유형 | 개선 제안 |
|-----------|------|-----------|
| `temp`, `tmp` | 임시 변수 | 실제 용도를 반영한 이름 |
| `data`, `info` | 범용 명사 | 구체적 데이터 유형 명시 |
| `result`, `res` | 결과 변수 | 어떤 결과인지 명시 |
| `obj`, `item` | 범용 참조 | 구체적 도메인 객체명 |
| `flag`, `status` | 상태 변수 | 무엇의 상태인지 명시 |
| `handler`, `processor` | 범용 동사 | 구체적 동작 명시 |
| `x`, `y`, `a`, `b` | 단일 문자 | 루프 변수 외 사용 지양 |
| `doSomething` | 모호한 동사 | 구체적 동작 명시 |

## 워크플로우 상세

분석 전체 흐름을 단계별로 설명합니다.

```
1. [범위 결정]
   ├─ 사용자 지정 파일/디렉토리 → 해당 범위
   ├─ "전체 분석" → 프로젝트 루트
   └─ 미지정 → git diff 변경 파일 또는 현재 디렉토리
       └─ git 저장소가 아닌 경우 → 현재 디렉토리 전체

2. [파일 목록 수집]
   ├─ 언어별 파일 필터링 (.ts, .js, .py, .java, .go, .rb 등)
   ├─ node_modules, .git, dist, build 등 제외
   └─ 파일 크기 제한: 5000줄 이상 파일은 경고 후 분석

3. [파일별 정적 분석]
   ├─ 각 파일을 Read 도구로 읽기
   ├─ 함수/클래스 단위 파싱
   ├─ 복잡도 메트릭 수집
   ├─ 코드 스멜 패턴 매칭
   └─ 네이밍 컨벤션 검사

4. [중복 코드 탐지]
   ├─ 파일 간 유사 코드 블록 비교
   └─ 복사-붙여넣기 패턴 감지

5. [분석 결과 종합]
   ├─ 발견 사항 심각도 분류 (Critical / High / Medium / Low / Info)
   ├─ 우선순위 매트릭스 적용 (영향도 x 난이도)
   └─ 디자인 패턴 제안 매칭

6. [리포트 생성]
   ├─ 요약 대시보드 (복잡도 + 스멜 + 중복 + 네이밍)
   ├─ 상세 발견 사항 (Before/After 코드 예시 포함)
   ├─ 우선순위별 액션 아이템
   └─ references/templates.md 형식에 따라 출력
```

## 트리거 조건

다음 키워드가 사용자 요청에 포함되면 이 스킬을 활성화합니다:

- 한국어: "리팩토링 분석", "코드 품질 분석", "코드 스멜 찾아줘", "복잡도 분석", "중복 코드 탐지", "네이밍 검사", "코드 정리", "코드 개선", "리팩토링 포인트", "코드 리뷰"
- 영어: "refactor", "refactoring", "code smell", "complexity analysis", "dead code", "code quality"

## 복잡도 메트릭 요약

각 메트릭의 임계값과 권장 조치를 정리합니다.

| 메트릭 | 측정 단위 | 양호 | 주의 | 위험 | 조치 |
|--------|-----------|------|------|------|------|
| Cyclomatic Complexity | 함수당 | 1-10 | 11-20 | 21+ | Extract Method |
| Cognitive Complexity | 함수당 | 1-10 | 11-15 | 16+ | 중첩 해소/분할 |
| 함수 길이 | 줄 수 | 1-30 | 31-60 | 61+ | Extract Method |
| 클래스/파일 크기 | 줄 수 | 1-300 | 301-500 | 501+ | 책임 분리 |
| 중첩 깊이 | 단계 수 | 1-3 | 4 | 5+ | Guard Clause/Early Return |
| 매개변수 수 | 개수 | 1-3 | 4 | 5+ | Parameter Object |

## 코드 스멜 카탈로그 요약

| 코드 스멜 | 감지 기준 | 주요 리팩토링 기법 |
|-----------|-----------|-------------------|
| God Class | 메서드 10+ 또는 300줄+ | Extract Class |
| God Method | 50줄+ 또는 CC 20+ | Extract Method |
| Feature Envy | 외부 객체 참조 > 자체 참조 | Move Method |
| Long Parameter List | 매개변수 5개+ | Parameter Object / Builder |
| Primitive Obsession | 관련 원시 타입 그룹 반복 | Value Object / Enum |
| Data Clumps | 동일 매개변수 그룹 3곳+ 반복 | Extract Class |
| Switch Statements | 동일 조건 switch 2곳+ 반복 | Strategy / Polymorphism |
| Dead Code | 참조 없는 함수/변수/import | 삭제 |
| Magic Numbers | 의미 없는 리터럴 직접 사용 | Named Constant / Enum |

## 우선순위 매트릭스

발견된 이슈를 **영향도(Impact)**와 **난이도(Effort)**로 분류하여 우선순위를 결정합니다.

```
              높은 영향도(High Impact)    낮은 영향도(Low Impact)
           ┌─────────────────────────┬─────────────────────────┐
높은 난이도 │  [P2] 계획적 실행        │  [P4] 보류/후순위       │
(High      │  충분한 시간 확보 후      │  ROI 낮음,              │
 Effort)   │  단계적으로 진행          │  필요시 재검토           │
           ├─────────────────────────┼─────────────────────────┤
낮은 난이도 │  [P1] 즉시 실행          │  [P3] 빈 시간에 처리    │
(Low       │  높은 ROI,               │  빠르게 처리 가능,      │
 Effort)   │  바로 착수               │  코드 품질 향상          │
           └─────────────────────────┴─────────────────────────┘
```

**우선순위 부여 규칙**:
- **P1 (즉시 실행)**: Dead Code 제거, Magic Number 상수화, 미사용 import 정리
- **P2 (계획적 실행)**: God Class 분할, 복잡한 함수 분리, 디자인 패턴 적용
- **P3 (빈 시간에 처리)**: 네이밍 개선, 약어 풀기, 단순 중복 제거
- **P4 (보류/후순위)**: 대규모 아키텍처 변경, 프레임워크 마이그레이션

## 에러 처리

### 분석 중 발생 가능한 에러와 대응

| # | 에러 상황 | 대응 |
|---|-----------|------|
| 1 | 분석 대상 파일 없음 | 지원 언어 확장자 안내 후 범위 재지정 요청 |
| 2 | 파일 읽기 실패 | 해당 파일 건너뛰고 나머지 분석 계속, 실패 파일 목록 리포트 |
| 3 | 바이너리 파일 감지 | 자동 제외 후 알림 |
| 4 | 파일이 너무 큰 경우 (5000줄+) | 경고 출력 후 분석 진행, 주요 함수 중심 분석 |
| 5 | 지원하지 않는 언어 | 범용 분석(줄 수, 중복, 네이밍)만 수행 후 한계 안내 |
| 6 | git 저장소가 아닌 경우 | git diff 대신 전체 디렉토리 스캔으로 전환 |
| 7 | 빈 파일 | 자동 제외 |
| 8 | 분석 범위가 너무 넓은 경우 | 파일 수 표시 후 범위 축소 제안 (100개 파일 초과 시) |

### 에러 메시지 형식

에러 발생 시 한국어로 사용자 친화적 메시지를 출력합니다. 상세 내용은 [references/templates.md](references/templates.md)를 참조합니다.

## Best Practices

### 분석 시

1. **범위를 좁게 시작**: 전체 코드베이스보다 변경된 파일이나 특정 모듈부터 분석
2. **언어별 관례 존중**: Python의 snake_case, Java의 camelCase 등 언어 표준을 기준으로 판단
3. **컨텍스트 고려**: 단순 메트릭 초과만으로 판단하지 말고, 코드의 역할과 맥락을 함께 고려
4. **False Positive 주의**: 자동 생성 코드, 설정 파일, 테스트 코드는 별도 기준 적용
5. **점진적 개선**: 한 번에 모든 것을 고치려 하지 말고, 우선순위에 따라 단계적으로 개선

### 리포트 작성 시

1. **Before/After 코드 포함**: 모든 개선 제안에 구체적인 코드 예시 제공
2. **영향 범위 명시**: 리팩토링 시 영향받는 다른 파일/함수 목록 제시
3. **테스트 가이드**: 리팩토링 후 확인해야 할 테스트 목록 포함
4. **단계별 가이드**: 복잡한 리팩토링은 단계별 실행 순서 제공
5. **위험도 명시**: 각 리팩토링의 위험도(동작 변경 가능성)를 명시

### 리팩토링 실행 시

1. **테스트 먼저 확인**: 기존 테스트가 통과하는지 먼저 확인
2. **작은 단위로 커밋**: 하나의 리팩토링 단위로 커밋하여 롤백 용이하게
3. **동작 보존 검증**: 리팩토링 전후 동일한 동작을 보장하는지 확인
4. **CI 파이프라인 확인**: 리팩토링 후 CI가 통과하는지 검증

## 분석 가이드

복잡도 계산 방법, 코드 스멜 판별 기준, 리팩토링 기법별 상세 가이드는 [references/analysis-guide.md](references/analysis-guide.md)를 참조합니다.

주요 내용:
- **Cyclomatic/Cognitive Complexity 상세 계산법**: 언어별 분기문 가중치
- **코드 스멜 판별 기준값**: 각 스멜별 구체적 수치 기준
- **리팩토링 기법 가이드**: Extract Method, Move Method, Replace Conditional with Polymorphism 등
- **언어별 분석 도구 연동**: ESLint, Pylint, SonarQube 메트릭 활용

## 템플릿

리팩토링 리포트 출력 형식, 코드 스멜별 Before/After 템플릿, 우선순위 매트릭스 시각화, 복잡도 대시보드 형식은 [references/templates.md](references/templates.md)를 참조합니다.
