# Refactor Advisor

코드베이스를 정적 분석하여 리팩토링이 필요한 포인트를 식별하고, 구체적인 개선 방안과 우선순위를 제시하는 Claude Code 커스텀 스킬입니다.

## 스킬 소개

**Refactor Advisor**는 코드의 복잡도, 코드 스멜, 중복 코드, 네이밍 컨벤션을 종합적으로 분석하여 실행 가능한 리팩토링 가이드를 제공합니다. 단순한 경고 목록이 아니라, Before/After 코드 예시와 함께 우선순위가 매겨진 구체적인 개선 방안을 제시합니다.

### 이런 경우에 사용하세요

- 레거시 코드를 인수받아 어디서부터 개선해야 할지 모를 때
- PR 전 코드 품질을 점검하고 싶을 때
- "이 코드 좀 정리해줘"라고 막연하게 요청하고 싶을 때
- 기술 부채를 정량화하고 관리하고 싶을 때
- 팀의 코드 컨벤션 준수 여부를 확인하고 싶을 때

### 사용 예시

```
사용자: 리팩토링 분석해줘
사용자: src/services/ 디렉토리 코드 품질 분석해줘
사용자: 이 파일에서 코드 스멜 찾아줘
사용자: 복잡도가 높은 함수 찾아줘
사용자: 중복 코드 탐지해줘
사용자: 네이밍 컨벤션 검사해줘
```

## 분석 관점 요약

이 스킬은 다음 5가지 관점에서 코드를 분석합니다.

### 1. 코드 복잡도 분석

코드의 구조적 복잡성을 정량적으로 측정합니다.

| 메트릭 | 측정 대상 | 임계값 |
|--------|-----------|--------|
| Cyclomatic Complexity | 함수별 독립 실행 경로 수 | 10 이하 권장 |
| Cognitive Complexity | 함수별 인지적 이해 난이도 | 15 이하 권장 |
| 함수 길이 | 함수/메서드 코드 줄 수 | 30줄 이하 |
| 클래스/파일 크기 | 파일 전체 줄 수 | 300줄 이하 |
| 중첩 깊이 | 조건/반복문 중첩 단계 | 4단계 이하 |

### 2. 코드 스멜 감지

소프트웨어 설계 문제를 나타내는 8가지 코드 스멜 패턴을 탐지합니다.

| 코드 스멜 | 핵심 증상 | 주요 리팩토링 기법 |
|-----------|-----------|-------------------|
| God Class / God Method | 하나의 클래스/함수가 너무 많은 책임 | Extract Class / Extract Method |
| Feature Envy | 다른 클래스의 데이터를 과도하게 참조 | Move Method |
| Long Parameter List | 매개변수가 5개 이상 | Parameter Object / Builder |
| Primitive Obsession | 관련 원시 타입의 반복 사용 | Value Object / Enum |
| Data Clumps | 동일 데이터 그룹이 여러 곳에서 반복 | Extract Class |
| Switch Statements | 동일 조건 분기가 여러 곳에서 반복 | Strategy / Polymorphism |
| Dead Code | 사용되지 않는 함수/변수/import | 삭제 |
| Magic Numbers/Strings | 의미 불명의 리터럴 값 직접 사용 | Named Constant / Enum |

### 3. 중복 코드 탐지

프로젝트 내 중복/유사 코드 블록을 식별하고 공통 로직 추출을 제안합니다.

- 정확한 중복 (완전 동일 코드)
- 구조적 중복 (변수명만 다른 동일 로직)
- 기능적 중복 (같은 목적의 다른 구현)

### 4. 디자인 패턴 제안

코드 구조를 분석하여 적용 가능한 디자인 패턴을 제안합니다.

| 코드 패턴 | 적용 가능한 디자인 패턴 |
|-----------|----------------------|
| 반복적 조건 분기 | Strategy Pattern |
| 복잡한 객체 생성 | Factory / Builder Pattern |
| 상태 기반 복잡한 분기 | State Pattern |
| 인터페이스 비호환 | Adapter Pattern |

### 5. 네이밍 컨벤션 분석

코드의 가독성과 유지보수성을 높이기 위한 네이밍 분석을 수행합니다.

- 네이밍 패턴 일관성 (camelCase, snake_case, PascalCase 혼용 여부)
- 불필요한 약어 사용
- 의미 없는 이름 (temp, data, result 등)

## 실제 리팩토링 Before/After 예시

### 예시 1: God Method 분할

**Before** - 하나의 함수에 여러 책임이 혼재 (82줄):
```typescript
async function processOrder(order: Order) {
  // 입력 검증 (15줄)
  if (!order.userId) throw new Error("userId required");
  if (!order.items || order.items.length === 0) throw new Error("items required");
  const user = await db.users.findById(order.userId);
  if (!user) throw new Error("user not found");
  if (user.status !== "active") throw new Error("user inactive");

  // 가격 계산 (20줄)
  let totalPrice = 0;
  for (const item of order.items) {
    const product = await db.products.findById(item.productId);
    if (!product) throw new Error(`product ${item.productId} not found`);
    if (product.stock < item.quantity) throw new Error("insufficient stock");
    totalPrice += product.price * item.quantity;
  }

  // 할인 적용 (15줄)
  if (user.membership === "vip") totalPrice *= 0.8;
  else if (user.membership === "gold") totalPrice *= 0.9;
  if (order.couponCode) {
    const coupon = await db.coupons.findByCode(order.couponCode);
    if (coupon && coupon.expiresAt > new Date()) {
      totalPrice -= coupon.discountAmount;
    }
  }

  // 결제 처리 + 재고 차감 + 알림 발송 (32줄)
  // ... 생략 ...
}
```

**After** - 책임별로 함수를 분리:
```typescript
async function processOrder(order: Order) {
  const user = await validateOrderInput(order);
  const totalPrice = await calculateTotalPrice(order.items);
  const finalPrice = await applyDiscounts(totalPrice, user, order.couponCode);
  await executePayment(user, finalPrice);
  await updateInventory(order.items);
  await sendOrderNotification(user, order);
}

async function validateOrderInput(order: Order): Promise<User> {
  if (!order.userId) throw new Error("userId required");
  if (!order.items?.length) throw new Error("items required");
  const user = await db.users.findById(order.userId);
  if (!user) throw new Error("user not found");
  if (user.status !== "active") throw new Error("user inactive");
  return user;
}

async function calculateTotalPrice(items: OrderItem[]): Promise<number> {
  let total = 0;
  for (const item of items) {
    const product = await db.products.findById(item.productId);
    if (!product) throw new Error(`product ${item.productId} not found`);
    if (product.stock < item.quantity) throw new Error("insufficient stock");
    total += product.price * item.quantity;
  }
  return total;
}

async function applyDiscounts(
  price: number, user: User, couponCode?: string
): Promise<number> {
  let finalPrice = applyMembershipDiscount(price, user.membership);
  if (couponCode) {
    finalPrice = await applyCouponDiscount(finalPrice, couponCode);
  }
  return finalPrice;
}
```

### 예시 2: Magic Numbers 상수화

**Before**:
```python
def calculate_shipping(weight, distance):
    if weight < 5:
        base = 3000
    elif weight < 20:
        base = 5000
    else:
        base = 8000

    if distance > 100:
        base *= 1.5
    if distance > 500:
        base *= 2.0

    return base
```

**After**:
```python
LIGHT_WEIGHT_THRESHOLD_KG = 5
MEDIUM_WEIGHT_THRESHOLD_KG = 20
LIGHT_SHIPPING_FEE = 3000
MEDIUM_SHIPPING_FEE = 5000
HEAVY_SHIPPING_FEE = 8000
REGIONAL_DISTANCE_KM = 100
REGIONAL_SURCHARGE_RATE = 1.5
LONG_DISTANCE_KM = 500
LONG_DISTANCE_SURCHARGE_RATE = 2.0

def calculate_shipping(weight: float, distance: float) -> int:
    base = _get_base_fee(weight)
    return _apply_distance_surcharge(base, distance)

def _get_base_fee(weight: float) -> int:
    if weight < LIGHT_WEIGHT_THRESHOLD_KG:
        return LIGHT_SHIPPING_FEE
    elif weight < MEDIUM_WEIGHT_THRESHOLD_KG:
        return MEDIUM_SHIPPING_FEE
    return HEAVY_SHIPPING_FEE

def _apply_distance_surcharge(base: int, distance: float) -> int:
    if distance > LONG_DISTANCE_KM:
        return int(base * LONG_DISTANCE_SURCHARGE_RATE)
    if distance > REGIONAL_DISTANCE_KM:
        return int(base * REGIONAL_SURCHARGE_RATE)
    return base
```

### 예시 3: 반복적 Switch를 Strategy Pattern으로 전환

**Before**:
```typescript
function getNotificationMessage(type: string, userName: string): string {
  switch (type) {
    case "welcome": return `${userName}님, 환영합니다!`;
    case "order":   return `${userName}님, 주문이 완료되었습니다.`;
    case "payment": return `${userName}님, 결제가 처리되었습니다.`;
    case "shipped": return `${userName}님, 상품이 발송되었습니다.`;
    default:        return `${userName}님, 알림이 있습니다.`;
  }
}

function getNotificationChannel(type: string): string {
  switch (type) {
    case "welcome": return "email";
    case "order":   return "push";
    case "payment": return "sms";
    case "shipped": return "push";
    default:        return "email";
  }
}
// 동일한 type 분기가 5개 이상의 함수에서 반복...
```

**After**:
```typescript
interface NotificationStrategy {
  getMessage(userName: string): string;
  getChannel(): string;
}

class WelcomeNotification implements NotificationStrategy {
  getMessage(userName: string) { return `${userName}님, 환영합니다!`; }
  getChannel() { return "email"; }
}

class OrderNotification implements NotificationStrategy {
  getMessage(userName: string) { return `${userName}님, 주문이 완료되었습니다.`; }
  getChannel() { return "push"; }
}

const strategies: Record<string, NotificationStrategy> = {
  welcome: new WelcomeNotification(),
  order: new OrderNotification(),
  // ...
};

function getNotificationStrategy(type: string): NotificationStrategy {
  return strategies[type] ?? new DefaultNotification();
}
```

## 우선순위 매트릭스 설명

발견된 모든 리팩토링 포인트는 **영향도(Impact)**와 **난이도(Effort)** 두 축으로 분류됩니다.

```
              높은 영향도                낮은 영향도
           ┌─────────────────────────┬─────────────────────────┐
높은 난이도 │  [P2] 계획적 실행        │  [P4] 보류/후순위       │
           │  - God Class 분할       │  - 대규모 구조 변경     │
           │  - 복잡 함수 재설계      │  - 프레임워크 전환      │
           │  - 디자인 패턴 적용      │  - 스타일 취향 변경     │
           ├─────────────────────────┼─────────────────────────┤
낮은 난이도 │  [P1] 즉시 실행          │  [P3] 빈 시간에 처리    │
           │  - Dead Code 제거       │  - 네이밍 개선          │
           │  - Magic Number 상수화  │  - 약어 풀기            │
           │  - 미사용 import 정리   │  - 주석 정리            │
           └─────────────────────────┴─────────────────────────┘
```

### 영향도 판단 기준

- **높은 영향도**: 버그 위험, 유지보수 비용 증가, 기능 확장 저해, 팀 생산성 저하
- **낮은 영향도**: 코드 미관, 마이너 컨벤션 위반, 성능에 미미한 영향

### 난이도 판단 기준

- **높은 난이도**: 여러 파일 수정 필요, 인터페이스 변경, 테스트 대폭 수정, 동작 변경 위험
- **낮은 난이도**: 단일 파일 수정, 동작 변경 없음, 기존 테스트 유지 가능

### 리포트에서의 표현

리포트에서 각 이슈는 다음과 같이 우선순위가 태그됩니다:

```
[P1][High Impact / Low Effort] Dead Code: 미사용 함수 getUserLegacy() 삭제
  → 파일: src/services/userService.ts:145-162
  → 조치: 함수 삭제 (다른 파일에서 참조 없음 확인 완료)

[P2][High Impact / High Effort] God Class: UserService 클래스 분할 필요
  → 파일: src/services/userService.ts (487줄, 메서드 18개)
  → 조치: 인증 로직 → AuthService, 프로필 로직 → ProfileService로 분리
  → 예상 소요: 2-3시간, 관련 테스트 수정 필요

[P3][Low Impact / Low Effort] Naming: 약어 사용 개선
  → usr → user, msg → message, btn → button
  → 파일: src/components/UserCard.tsx
```

## 관련 문서

- [SKILL.md](SKILL.md) - Claude Code용 상세 동작 지침
- [references/analysis-guide.md](references/analysis-guide.md) - 분석 방법론 상세 가이드
- [references/templates.md](references/templates.md) - 리포트 출력 템플릿
