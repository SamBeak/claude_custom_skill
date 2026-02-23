# 리팩토링 분석 방법론 가이드

코드 복잡도 계산, 코드 스멜 판별, 중복 코드 탐지, 리팩토링 기법 적용에 대한 상세 방법론을 제공합니다.

## 목차

1. [복잡도 계산 방법](#복잡도-계산-방법)
2. [코드 스멜 판별 기준](#코드-스멜-판별-기준)
3. [중복 코드 탐지 알고리즘](#중복-코드-탐지-알고리즘)
4. [리팩토링 기법별 적용 가이드](#리팩토링-기법별-적용-가이드)
5. [언어별 분석 도구 연동](#언어별-분석-도구-연동)

---

## 복잡도 계산 방법

### Cyclomatic Complexity (순환 복잡도) 상세 계산법

Thomas McCabe가 1976년에 제안한 소프트웨어 복잡도 측정 지표입니다. 함수 내 독립 실행 경로의 수를 측정합니다.

#### 기본 공식

```
CC(함수) = 1 + (분기점 수)
```

#### 언어별 분기점 목록

**JavaScript / TypeScript**:
| 구문 | 가중치 | 설명 |
|------|--------|------|
| `if` | +1 | 조건 분기 |
| `else if` | +1 | 추가 조건 분기 |
| `case` (switch) | +1 | 각 case마다 |
| `for` | +1 | 반복문 |
| `for...of` / `for...in` | +1 | 반복문 |
| `while` | +1 | 반복문 |
| `do...while` | +1 | 반복문 |
| `catch` | +1 | 예외 처리 분기 |
| `&&` | +1 | 논리 AND (단축 평가) |
| `\|\|` | +1 | 논리 OR (단축 평가) |
| `??` | +1 | Nullish coalescing |
| `? :` (삼항) | +1 | 삼항 연산자 |
| `?.` (optional chaining) | +0 | 분기로 계산하지 않음 |

**Python**:
| 구문 | 가중치 | 설명 |
|------|--------|------|
| `if` | +1 | 조건 분기 |
| `elif` | +1 | 추가 조건 분기 |
| `for` | +1 | 반복문 |
| `while` | +1 | 반복문 |
| `except` | +1 | 각 except 절마다 |
| `and` | +1 | 논리 AND |
| `or` | +1 | 논리 OR |
| `if...else` (표현식) | +1 | 인라인 조건 |
| list/dict comprehension 내 `if` | +1 | 필터 조건 |
| `match` / `case` (3.10+) | +1 per case | 각 case마다 |

**Java / Go**:
| 구문 | 가중치 | 설명 |
|------|--------|------|
| `if` | +1 | 조건 분기 |
| `else if` | +1 | 추가 조건 분기 |
| `case` (switch) | +1 | 각 case마다 |
| `for` | +1 | 반복문 |
| `while` | +1 | 반복문 |
| `do...while` (Java) | +1 | 반복문 |
| `catch` (Java) / 다중 반환 에러 체크 (Go) | +1 | 예외/에러 처리 |
| `&&` | +1 | 논리 AND |
| `\|\|` | +1 | 논리 OR |
| `? :` (Java 삼항) | +1 | 삼항 연산자 |

#### 계산 예제

```typescript
function processPayment(order: Order): Result {  // 시작: 1
  if (!order.isValid()) {                         // +1 (if)
    return Result.error("invalid order");
  }

  for (const item of order.items) {               // +1 (for)
    if (item.quantity <= 0 || item.price < 0) {   // +1 (if) +1 (||)
      return Result.error("invalid item");
    }
  }

  try {
    if (order.paymentMethod === "card") {          // +1 (if)
      return processCardPayment(order);
    } else if (order.paymentMethod === "bank") {   // +1 (else if)
      return processBankTransfer(order);
    } else {
      return Result.error("unsupported method");
    }
  } catch (e) {                                    // +1 (catch)
    return Result.error(e.message);
  }
}
// CC = 1 + 1 + 1 + 1 + 1 + 1 + 1 + 1 = 8 (양호)
```

### Cognitive Complexity (인지 복잡도) 상세 계산법

SonarSource가 제안한 지표로, 코드를 읽고 이해하는 데 필요한 인지적 노력을 측정합니다. Cyclomatic Complexity와의 핵심 차이점은 **중첩에 추가 비용을 부과**한다는 것입니다.

#### 계산 규칙 상세

**규칙 1: 기본 증가 (Increment)**
다음 구문에 +1을 부여합니다:
- `if`, `else if`, `else`
- `switch`
- `for`, `foreach`, `while`, `do...while`
- `catch`
- `break`/`continue` (레이블 포함 시)
- 재귀 호출
- `goto` (해당 언어에 존재 시)

**규칙 2: 중첩 증가 (Nesting Increment)**
중첩 내부의 구문은 **기본 +1에 추가로 현재 중첩 깊이만큼** 추가됩니다:
```
비용 = 1(기본) + nesting_depth
```

**규칙 3: 논리 연산자 처리**
- 같은 종류의 논리 연산자가 연속되면 추가 비용 없음
- 종류가 바뀌면 +1

```typescript
// 예시: 논리 연산자
a && b && c          // +1 (하나의 && 시퀀스)
a || b || c          // +1 (하나의 || 시퀀스)
a && b || c          // +1 (&&) + 1 (||로 전환) = +2
a || b && c || d     // +1 (||) + 1 (&& 전환) + 1 (|| 전환) = +3
```

#### 계산 예제

```typescript
function findActiveUsers(users: User[]): User[] {  // nesting = 0
  const result: User[] = [];

  for (const user of users) {                       // +1 (for), nesting = 1
    if (user.isActive) {                            // +2 (if +1, nesting +1), nesting = 2
      if (user.age >= 18                            // +3 (if +1, nesting +2), nesting = 3
          && user.verified) {                       // +1 (&&)
        result.push(user);
      } else {                                      // +1 (else)
        if (user.parentConsent) {                   // +4 (if +1, nesting +3)
          result.push(user);
        }
      }
    }
  }
  return result;
}
// Cognitive Complexity = 1 + 2 + 3 + 1 + 1 + 4 = 12 (주의 수준)
```

#### 개선된 버전 (Cognitive Complexity 감소)

```typescript
function findActiveUsers(users: User[]): User[] {
  return users.filter(isEligibleUser);
}

function isEligibleUser(user: User): boolean {
  if (!user.isActive) return false;           // +1
  if (user.age >= 18 && user.verified) {      // +1 (if) + 1 (&&)
    return true;
  }
  return user.parentConsent;                  // +0
}
// Cognitive Complexity = 3 (양호)
```

---

## 코드 스멜 판별 기준

각 코드 스멜에 대한 구체적인 판별 기준값과 탐지 방법을 정의합니다.

### God Class 판별 기준

| 지표 | 임계값 | 가중치 |
|------|--------|--------|
| 코드 줄 수 (LOC) | 300줄 초과 | 높음 |
| 메서드 수 | 10개 초과 | 높음 |
| 필드/속성 수 | 10개 초과 | 보통 |
| 다른 클래스 의존 수 | 8개 초과 | 보통 |
| 책임 영역 수 | 3개 초과 | 높음 |

**종합 판정**: 위 지표 중 2개 이상이 임계값을 초과하면 God Class로 판정합니다.

**책임 영역 식별 방법**:
1. 메서드 이름의 동사 그룹 분류 (validate, calculate, send, format 등)
2. 필드 접근 패턴 분석 (어떤 메서드가 어떤 필드를 사용하는지)
3. import/의존성 그룹 분석

### God Method 판별 기준

| 지표 | 임계값 |
|------|--------|
| 코드 줄 수 (LOC) | 50줄 초과 |
| Cyclomatic Complexity | 20 초과 |
| 매개변수 수 | 4개 초과 |
| 지역 변수 수 | 10개 초과 |
| 중첩 깊이 최대값 | 4단계 초과 |

**종합 판정**: LOC 50줄 초과이거나 CC 20 초과이면 God Method로 판정합니다.

### Feature Envy 판별 기준

```
외부 참조 비율 = (다른 클래스 멤버 접근 수) / (전체 멤버 접근 수)
```

| 지표 | 임계값 |
|------|--------|
| 외부 참조 비율 | 50% 초과 |
| 특정 외부 클래스 참조 수 | 3회 이상 |
| 자체 필드 접근 수 | 1회 이하 |

### Long Parameter List 판별 기준

| 언어 | 임계값 | 비고 |
|------|--------|------|
| 일반 | 5개 이상 | |
| 생성자 | 4개 이상 | Builder Pattern 권장 |
| 콜백/이벤트 핸들러 | 3개 이상 | Event Object 권장 |

### Primitive Obsession 판별 기준

**탐지 패턴**:
1. **접두사 그룹**: 같은 접두사를 가진 원시 타입 변수 3개 이상
   ```typescript
   // 감지 대상
   const userName = "John";
   const userEmail = "john@email.com";
   const userAge = 30;
   const userAddress = "Seoul";
   // → User 클래스/인터페이스로 추출 제안
   ```

2. **문자열 상태값**: 문자열로 표현된 제한된 값의 집합
   ```typescript
   // 감지 대상
   if (status === "active" || status === "inactive" || status === "suspended")
   // → enum 또는 union type으로 추출 제안
   ```

3. **원시 타입 유효성 검증 반복**: 같은 타입에 대한 유효성 검증이 여러 곳에서 반복
   ```typescript
   // 감지 대상
   if (email.includes("@") && email.includes(".")) { ... }  // 여러 곳에서 반복
   // → Email Value Object로 추출 제안
   ```

### Data Clumps 판별 기준

동일한 매개변수 조합(3개 이상)이 3곳 이상의 함수/메서드에서 반복되면 Data Clump로 판정합니다.

```typescript
// 감지 대상: (latitude, longitude, altitude)가 3곳 이상에서 반복
function calculateDistance(lat1: number, lng1: number, alt1: number,
                          lat2: number, lng2: number, alt2: number) { }
function formatLocation(lat: number, lng: number, alt: number) { }
function storeLocation(lat: number, lng: number, alt: number) { }

// → GeoCoordinate { latitude, longitude, altitude } 클래스로 추출
```

### Switch Statements 판별 기준

동일한 조건 변수에 대한 switch/if-else 체인이 **2곳 이상의 함수/메서드에서 반복**되면 감지합니다.

**탐지 방법**:
1. switch/if-else 체인에서 분기 조건의 변수/타입 추출
2. 같은 변수/타입으로 분기하는 switch/if-else가 다른 위치에 존재하는지 확인
3. case/분기의 값 집합이 동일한지 비교

### Dead Code 판별 기준

| 대상 | 탐지 방법 |
|------|-----------|
| 미사용 함수 | 프로젝트 전체에서 해당 함수명으로 Grep 검색, export 여부 확인 |
| 미사용 변수 | 선언 후 읽기 참조가 없는 변수 |
| 미사용 import | import 후 사용되지 않는 모듈/심볼 |
| 주석 처리된 코드 | 5줄 이상 연속된 주석이 코드 구조를 가진 경우 |
| 도달 불가능한 코드 | return/throw/break 이후의 코드 |

**주의사항**:
- `export`된 함수는 외부 프로젝트에서 사용될 수 있으므로 라이브러리 프로젝트에서는 제외
- 리플렉션이나 동적 호출로 사용되는 코드는 false positive 가능
- 테스트 코드에서만 사용되는 코드는 Dead Code로 분류하지 않음

### Magic Numbers/Strings 판별 기준

**탐지 대상**:
- 조건문/계산식에 직접 사용된 숫자 리터럴 (예외: 0, 1, -1, 2, 100)
- 비교/분기에 직접 사용된 문자열 리터럴
- 같은 리터럴 값이 2곳 이상에서 반복 사용

**예외 (탐지 제외)**:
- 배열 인덱스: `arr[0]`, `arr[1]`
- 단순 증감: `i + 1`, `count - 1`
- 표준 값: `100`(퍼센트), `1000`(밀리초→초 변환)
- 테스트 코드의 기대값
- 상수 정의문 자체

---

## 중복 코드 탐지 알고리즘

### 탐지 전략

코드 중복을 3단계로 탐지합니다.

#### 1단계: 정확한 중복 (Type 1 Clone)

공백/주석을 제외하고 완전히 동일한 코드 블록을 탐지합니다.

**알고리즘**:
1. 각 파일에서 공백/주석을 제거한 정규화된 코드를 생성
2. 연속된 N줄(기본 5줄) 단위로 슬라이딩 윈도우 적용
3. 각 윈도우의 해시값을 계산
4. 동일한 해시값을 가진 블록 쌍을 중복으로 보고

#### 2단계: 구조적 중복 (Type 2 Clone)

변수명/리터럴만 다르고 구조가 동일한 코드를 탐지합니다.

**알고리즘**:
1. 코드에서 식별자(변수명, 함수명)와 리터럴을 플레이스홀더로 치환
2. 치환된 코드에 대해 1단계와 동일한 해시 비교 수행
3. 플레이스홀더 매핑이 일관적인 경우에만 중복으로 판정

**예시**:
```typescript
// 위치 1
const userAge = parseInt(userInput, 10);
if (userAge < 0 || userAge > 150) {
  throw new Error("Invalid age");
}

// 위치 2 (구조적 중복)
const productPrice = parseInt(priceInput, 10);
if (productPrice < 0 || productPrice > 999999) {
  throw new Error("Invalid price");
}
```

#### 3단계: 기능적 중복 (Type 3/4 Clone)

동일한 목적을 다른 방식으로 구현한 코드를 탐지합니다. 이 단계는 Claude의 코드 이해 능력을 활용하여 의미적으로 유사한 코드를 식별합니다.

**탐지 기준**:
- 동일한 입력을 받아 동일한 출력을 생성하는 함수
- 같은 API를 호출하는 유사한 시퀀스
- 같은 데이터 구조를 변환하는 유사한 로직

### 중복 보고 형식

```
╔════════════════════════════════════════════════════════╗
║  중복 코드 감지 보고                                    ║
╠════════════════════════════════════════════════════════╣
║  유형: Type 2 (구조적 중복)                             ║
║  유사도: 94%                                           ║
║  중복 줄 수: 18줄 x 2곳 = 36줄                          ║
╠────────────────────────────────────────────────────────╣
║  위치 1: src/services/userService.ts:45-62             ║
║  위치 2: src/services/orderService.ts:78-95            ║
╠────────────────────────────────────────────────────────╣
║  제안: 공통 유효성 검증 로직을 추출                      ║
║  대상: src/utils/validation.ts                         ║
║  예상 제거 라인: 약 18줄                                ║
╚════════════════════════════════════════════════════════╝
```

### 최소 중복 크기 기준

| 유형 | 최소 줄 수 | 비고 |
|------|-----------|------|
| 정확한 중복 | 3줄 | 매우 짧은 중복도 보고 |
| 구조적 중복 | 5줄 | 변수명 치환 후 비교 |
| 기능적 중복 | 10줄 | 의미적 분석 필요 |

---

## 리팩토링 기법별 적용 가이드

### Extract Method (메서드 추출)

**적용 대상**: 긴 함수, 중복 코드 블록, 주석으로 구분된 코드 섹션

**적용 절차**:
1. 추출할 코드 블록을 식별 (주석이 있다면 주석이 함수명 후보)
2. 해당 블록에서 사용되는 변수 분석
   - 읽기만 하는 변수 → 매개변수로 전달
   - 수정되는 변수 → 반환값으로 처리
   - 여러 변수가 수정되면 → 객체로 묶어 반환 또는 분할 재고
3. 새 함수를 생성하고 코드 블록을 이동
4. 원래 위치에서 새 함수를 호출
5. 테스트 실행으로 동작 보존 확인

**안전성 체크리스트**:
- [ ] 추출된 코드 블록이 하나의 명확한 책임만 수행하는가
- [ ] 반환값이 단일하거나 의미 있는 객체인가
- [ ] 매개변수가 3개 이하인가
- [ ] 부수 효과(side effect)가 명확히 문서화되어 있는가
- [ ] 기존 테스트가 여전히 통과하는가

### Move Method (메서드 이동)

**적용 대상**: Feature Envy 스멜이 감지된 메서드

**적용 절차**:
1. 메서드가 가장 많이 참조하는 클래스를 식별
2. 해당 클래스로 메서드를 복사
3. 원래 클래스의 참조를 self 참조로 변경
4. 원래 클래스에서 메서드를 제거하고 위임(delegate)으로 교체 (점진적)
5. 모든 호출 지점을 새 위치로 업데이트
6. 테스트 실행

**주의사항**:
- 순환 의존성이 발생하지 않는지 확인
- 접근 제한자(private/protected) 조정 필요 여부 확인
- 상속 관계에 영향을 미치는지 확인

### Replace Conditional with Polymorphism (조건문을 다형성으로 교체)

**적용 대상**: 반복적 switch/if-else 체인, 타입 코드에 따른 분기

**적용 절차**:
1. 공통 인터페이스/추상 클래스 정의
2. 각 분기 조건에 대응하는 구체 클래스 생성
3. 각 구체 클래스에 해당 분기의 로직 구현
4. 팩토리 또는 맵으로 구체 클래스 선택 로직 구현
5. 기존 switch/if-else를 다형성 호출로 교체
6. 테스트 실행

**예시 (TypeScript)**:

Before:
```typescript
function calculateArea(shape: string, params: number[]): number {
  switch (shape) {
    case "circle":
      return Math.PI * params[0] ** 2;
    case "rectangle":
      return params[0] * params[1];
    case "triangle":
      return (params[0] * params[1]) / 2;
    default:
      throw new Error(`Unknown shape: ${shape}`);
  }
}
```

After:
```typescript
interface Shape {
  calculateArea(): number;
}

class Circle implements Shape {
  constructor(private radius: number) {}
  calculateArea() { return Math.PI * this.radius ** 2; }
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  calculateArea() { return this.width * this.height; }
}

class Triangle implements Shape {
  constructor(private base: number, private height: number) {}
  calculateArea() { return (this.base * this.height) / 2; }
}
```

### Introduce Parameter Object (매개변수 객체 도입)

**적용 대상**: Long Parameter List, Data Clumps

**적용 절차**:
1. 함께 전달되는 매개변수 그룹을 식별
2. 해당 그룹을 담는 클래스/인터페이스 정의
3. 함수 시그니처를 새 매개변수 객체로 변경
4. 모든 호출 지점을 업데이트
5. (선택) 매개변수 객체에 관련 동작을 이동하여 풍부한 도메인 객체로 발전

### Replace Magic Number with Named Constant (매직 넘버를 명명 상수로 교체)

**적용 대상**: Magic Numbers/Strings 스멜

**적용 절차**:
1. 코드에서 매직 넘버/문자열을 식별
2. 의미를 반영한 상수명 결정
3. 상수를 적절한 위치에 선언 (파일 상단, 설정 파일, enum 등)
4. 모든 사용처에서 리터럴을 상수로 교체
5. 테스트 실행

**네이밍 가이드**:
```typescript
// Bad
const x = 86400;

// Good
const SECONDS_PER_DAY = 86400;

// Bad
if (status === "P") { ... }

// Good
const STATUS_PENDING = "P";
if (status === STATUS_PENDING) { ... }

// Better (TypeScript)
enum OrderStatus {
  PENDING = "P",
  COMPLETED = "C",
  CANCELLED = "X",
}
```

### Extract Class (클래스 추출)

**적용 대상**: God Class, Data Clumps

**적용 절차**:
1. 클래스 내 책임 영역을 식별 (메서드/필드의 관련성 분석)
2. 분리할 책임 영역의 메서드/필드 결정
3. 새 클래스를 생성하고 관련 메서드/필드를 이동
4. 원래 클래스에서 새 클래스를 참조하도록 구성
5. 필요 시 위임(delegation) 메서드 유지 (점진적 마이그레이션)
6. 외부 참조 업데이트
7. 테스트 실행

### Introduce Guard Clause (가드 절 도입)

**적용 대상**: 깊은 중첩 (nesting depth 4단계 이상)

**적용 절차**:
1. 함수 시작 부분의 조건 검사를 식별
2. 조건 불만족 시 early return으로 변환
3. 중첩 레벨 감소 확인
4. 테스트 실행

**예시**:

Before:
```typescript
function processOrder(order: Order): Result {
  if (order) {
    if (order.items.length > 0) {
      if (order.isPaid) {
        if (order.shippingAddress) {
          // 실제 로직 (중첩 4단계)
          return shipOrder(order);
        } else {
          return Result.error("주소 필요");
        }
      } else {
        return Result.error("미결제");
      }
    } else {
      return Result.error("상품 없음");
    }
  } else {
    return Result.error("주문 없음");
  }
}
```

After:
```typescript
function processOrder(order: Order): Result {
  if (!order) return Result.error("주문 없음");
  if (order.items.length === 0) return Result.error("상품 없음");
  if (!order.isPaid) return Result.error("미결제");
  if (!order.shippingAddress) return Result.error("주소 필요");

  // 실제 로직 (중첩 0단계)
  return shipOrder(order);
}
```

---

## 언어별 분석 도구 연동

Claude Code에서 각 언어별 정적 분석 도구의 결과를 활용하여 분석 정확도를 높일 수 있습니다.

### JavaScript / TypeScript - ESLint

**복잡도 관련 규칙**:
```json
{
  "rules": {
    "complexity": ["warn", 10],
    "max-depth": ["warn", 4],
    "max-lines-per-function": ["warn", { "max": 30, "skipBlankLines": true, "skipComments": true }],
    "max-params": ["warn", 5],
    "max-lines": ["warn", { "max": 300, "skipBlankLines": true, "skipComments": true }],
    "no-unused-vars": "warn",
    "no-unreachable": "error",
    "no-magic-numbers": ["warn", { "ignore": [0, 1, -1, 2] }]
  }
}
```

**ESLint 결과 활용 명령**:
```bash
# ESLint 실행 (JSON 포맷으로 결과 수집)
npx eslint --format json src/ 2>/dev/null

# 복잡도만 확인
npx eslint --rule '{"complexity": ["error", 10]}' --format json src/ 2>/dev/null

# 미사용 변수만 확인
npx eslint --rule '{"no-unused-vars": "error"}' --format json src/ 2>/dev/null
```

### Python - Pylint / Radon

**Pylint 관련 검사**:
```bash
# 전체 분석
pylint --output-format=json src/ 2>/dev/null

# 복잡도 관련
pylint --disable=all --enable=R0901,R0902,R0903,R0904,R0911,R0912,R0913,R0914,R0915 src/
```

**Pylint 메시지 코드와 스멜 매핑**:
| 코드 | 이름 | 대응 코드 스멜 |
|------|------|---------------|
| R0901 | too-many-ancestors | 깊은 상속 트리 |
| R0902 | too-many-instance-attributes | God Class (필드 과다) |
| R0903 | too-few-public-methods | 데이터 클래스 후보 |
| R0904 | too-many-public-methods | God Class (메서드 과다) |
| R0911 | too-many-return-statements | 복잡한 분기 |
| R0912 | too-many-branches | 높은 Cyclomatic Complexity |
| R0913 | too-many-arguments | Long Parameter List |
| R0914 | too-many-locals | God Method (지역 변수 과다) |
| R0915 | too-many-statements | God Method (줄 수 과다) |

**Radon (복잡도 전용 도구)**:
```bash
# Cyclomatic Complexity 측정
radon cc src/ -s -a -j

# Maintainability Index 측정
radon mi src/ -s -j

# 줄 수 통계
radon raw src/ -s -j
```

### Java - PMD / SpotBugs

**PMD 관련 규칙**:
```bash
# 복잡도 분석
pmd check -d src/ -R category/java/design.xml -f json

# 코드 스멜
pmd check -d src/ -R category/java/bestpractices.xml -f json

# 중복 코드
pmd cpd --dir src/ --minimum-tokens 100 --format json
```

**PMD 규칙과 스멜 매핑**:
| PMD 규칙 | 대응 코드 스멜 |
|----------|---------------|
| CyclomaticComplexity | 높은 순환 복잡도 |
| GodClass | God Class |
| ExcessiveMethodLength | God Method |
| ExcessiveParameterList | Long Parameter List |
| AvoidDuplicateLiterals | Magic Numbers/Strings |
| UnusedLocalVariable | Dead Code |
| UnusedPrivateMethod | Dead Code |
| SwitchDensity | Switch Statements |

### SonarQube 메트릭 활용

SonarQube가 프로젝트에 설정되어 있는 경우, 다음 메트릭을 활용할 수 있습니다.

**주요 메트릭**:
| SonarQube 메트릭 | 이 스킬에서의 활용 |
|-----------------|-------------------|
| `cognitive_complexity` | Cognitive Complexity 기준값 |
| `complexity` | Cyclomatic Complexity 기준값 |
| `duplicated_lines_density` | 중복 코드 비율 |
| `code_smells` | 전체 코드 스멜 수 |
| `sqale_index` | 기술 부채 (시간 단위) |
| `reliability_rating` | 신뢰성 등급 (A-E) |
| `maintainability_rating` | 유지보수성 등급 (A-E) |

**SonarQube API 활용** (프로젝트에 설정된 경우):
```bash
# 컴포넌트별 메트릭 조회
curl -s "http://localhost:9000/api/measures/component?component=project-key&metricKeys=complexity,cognitive_complexity,duplicated_lines_density,code_smells"

# 이슈 목록 조회
curl -s "http://localhost:9000/api/issues/search?componentKeys=project-key&types=CODE_SMELL&ps=100"
```

### Go - golangci-lint

**복잡도 관련 린터**:
```bash
# 전체 분석
golangci-lint run --out-format json ./...

# 복잡도 중심
golangci-lint run --enable gocyclo,gocognit,funlen,nestif --out-format json ./...
```

**린터와 스멜 매핑**:
| 린터 | 대응 코드 스멜 |
|------|---------------|
| gocyclo | 높은 Cyclomatic Complexity |
| gocognit | 높은 Cognitive Complexity |
| funlen | God Method (함수 길이) |
| nestif | 깊은 중첩 |
| dupl | 중복 코드 |
| unused | Dead Code |
| unconvert | 불필요한 타입 변환 |

### 도구 결과 통합 전략

1. **도구가 설치되어 있는 경우**: 도구 실행 결과를 먼저 수집하고, Claude의 분석과 교차 검증
2. **도구가 설치되어 있지 않은 경우**: Claude의 코드 읽기/분석 능력으로 직접 분석
3. **결과 통합**: 도구 결과와 Claude 분석 결과를 합산하되, 중복 제거

```bash
# 도구 존재 여부 확인
command -v eslint >/dev/null 2>&1 && echo "ESLint 사용 가능"
command -v pylint >/dev/null 2>&1 && echo "Pylint 사용 가능"
command -v pmd >/dev/null 2>&1 && echo "PMD 사용 가능"
command -v golangci-lint >/dev/null 2>&1 && echo "golangci-lint 사용 가능"
```
