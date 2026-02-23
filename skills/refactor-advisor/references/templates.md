# 리팩토링 리포트 템플릿

리팩토링 분석 결과를 출력하기 위한 표준 템플릿 모음입니다. 리포트 출력 형식, 코드 스멜별 Before/After 템플릿, 우선순위 매트릭스 시각화, 복잡도 대시보드, 개선 제안 카드 형식을 정의합니다.

## 목차

1. [리팩토링 리포트 출력 형식](#리팩토링-리포트-출력-형식)
2. [코드 스멜별 Before/After 템플릿](#코드-스멜별-beforeafter-템플릿)
3. [우선순위 매트릭스 시각화](#우선순위-매트릭스-시각화)
4. [복잡도 대시보드 형식](#복잡도-대시보드-형식)
5. [개선 제안 카드 형식](#개선-제안-카드-형식)
6. [에러 메시지 템플릿](#에러-메시지-템플릿)

---

## 리팩토링 리포트 출력 형식

분석 완료 후 사용자에게 제시하는 전체 리포트의 표준 형식입니다.

### 전체 리포트 템플릿

```markdown
# 리팩토링 분석 리포트

> 분석 일시: {YYYY-MM-DD HH:MM}
> 분석 범위: {분석 대상 경로}
> 분석 파일 수: {N}개
> 총 코드 줄 수: {N}줄 (공백/주석 제외)

---

## 요약 대시보드

| 분석 항목 | 발견 수 | 심각도 분포 |
|-----------|---------|------------|
| 복잡도 이슈 | {N}건 | Critical: {N} / High: {N} / Medium: {N} |
| 코드 스멜 | {N}건 | Critical: {N} / High: {N} / Medium: {N} |
| 중복 코드 | {N}건 | 총 {N}줄 중복 (중복률 {N}%) |
| 네이밍 이슈 | {N}건 | 일관성: {N}건 / 약어: {N}건 / 모호함: {N}건 |

### 전체 건강도 점수

```
코드 건강도: {점수}/100

[████████████████████░░░░░░░░░░] {N}%

복잡도:   [████████░░] {N}/100
코드스멜: [██████░░░░] {N}/100
중복:     [█████████░] {N}/100
네이밍:   [███████░░░] {N}/100
```

---

## 1. 복잡도 분석 결과

### 위험 함수 목록 (Cyclomatic Complexity 기준)

| 순위 | 파일 | 함수명 | CC | 인지복잡도 | 줄 수 | 중첩 | 판정 |
|------|------|--------|-----|-----------|-------|------|------|
| 1 | {파일경로} | {함수명} | {N} | {N} | {N} | {N} | 위험 |
| 2 | {파일경로} | {함수명} | {N} | {N} | {N} | {N} | 주의 |
| ... | | | | | | | |

### 파일별 크기 경고

| 파일 | 줄 수 | 함수 수 | 상태 |
|------|-------|---------|------|
| {파일경로} | {N}줄 | {N}개 | 임계값 초과 |
| ... | | | |

---

## 2. 코드 스멜 감지 결과

### 발견된 코드 스멜

{코드 스멜별 상세 내용 - 아래 개선 제안 카드 형식 사용}

---

## 3. 중복 코드 탐지 결과

### 중복 블록 목록

| # | 유형 | 유사도 | 위치 1 | 위치 2 | 줄 수 | 제안 |
|---|------|--------|--------|--------|-------|------|
| 1 | 구조적 | {N}% | {파일:줄} | {파일:줄} | {N}줄 | {추출 대상} |
| ... | | | | | | |

---

## 4. 디자인 패턴 제안

| # | 현재 코드 패턴 | 제안 패턴 | 적용 위치 | 기대 효과 |
|---|---------------|-----------|-----------|-----------|
| 1 | {현재 패턴} | {제안 패턴} | {파일:줄} | {효과} |
| ... | | | | |

---

## 5. 네이밍 컨벤션 분석 결과

### 일관성 위반

| 파일 | 현재 이름 | 사용된 패턴 | 기대 패턴 | 개선안 |
|------|-----------|-----------|-----------|--------|
| {파일경로} | {이름} | {snake_case} | {camelCase} | {개선이름} |

### 약어/모호한 이름

| 파일 | 현재 이름 | 유형 | 개선안 |
|------|-----------|------|--------|
| {파일경로} | {이름} | 약어/모호 | {개선이름} |

---

## 6. 우선순위별 액션 아이템

### P1 - 즉시 실행 (High Impact / Low Effort)
- [ ] {액션 아이템 설명} — `{파일경로}`

### P2 - 계획적 실행 (High Impact / High Effort)
- [ ] {액션 아이템 설명} — `{파일경로}`

### P3 - 빈 시간에 처리 (Low Impact / Low Effort)
- [ ] {액션 아이템 설명} — `{파일경로}`

### P4 - 보류/후순위 (Low Impact / High Effort)
- [ ] {액션 아이템 설명} — `{파일경로}`
```

### 간략 리포트 템플릿 (파일 단위 분석 시)

단일 파일 또는 소규모 분석 시 사용하는 간략한 형식입니다.

```markdown
## 리팩토링 분석: {파일명}

> {파일경로} | {N}줄 | 함수 {N}개

### 발견 사항 요약
- 복잡도 이슈: {N}건
- 코드 스멜: {N}건
- 네이밍 이슈: {N}건

### 상세 발견 사항

{개선 제안 카드 형식으로 나열}

### 액션 아이템
- [ ] [P{N}] {설명}
```

---

## 코드 스멜별 Before/After 템플릿

각 코드 스멜에 대한 표준 설명과 Before/After 코드 예시 템플릿입니다.

### God Class 템플릿

```markdown
#### God Class 감지

**위치**: `{파일경로}`
**지표**: {N}줄, 메서드 {N}개, 필드 {N}개
**심각도**: High

**문제**: 하나의 클래스가 너무 많은 책임을 가지고 있어 유지보수와 테스트가 어렵습니다.

**식별된 책임 영역**:
1. {책임 영역 1} — 관련 메서드: {메서드 목록}
2. {책임 영역 2} — 관련 메서드: {메서드 목록}
3. {책임 영역 3} — 관련 메서드: {메서드 목록}

**Before**:
```{language}
class UserManager {
  // 인증 관련 (책임 1)
  login(email, password) { ... }
  logout() { ... }
  resetPassword(email) { ... }

  // 프로필 관련 (책임 2)
  updateProfile(data) { ... }
  uploadAvatar(file) { ... }
  getPreferences() { ... }

  // 알림 관련 (책임 3)
  sendEmail(to, subject, body) { ... }
  sendPush(userId, message) { ... }
  getNotificationHistory() { ... }
}
```

**After**:
```{language}
class AuthService {
  login(email, password) { ... }
  logout() { ... }
  resetPassword(email) { ... }
}

class ProfileService {
  updateProfile(data) { ... }
  uploadAvatar(file) { ... }
  getPreferences() { ... }
}

class NotificationService {
  sendEmail(to, subject, body) { ... }
  sendPush(userId, message) { ... }
  getNotificationHistory() { ... }
}
```

**리팩토링 단계**:
1. 책임 영역별로 새 클래스 생성
2. 관련 메서드와 필드를 새 클래스로 이동
3. 기존 UserManager에서 새 클래스를 주입(DI)하여 위임
4. 외부 참조를 점진적으로 새 클래스로 이전
5. UserManager의 위임 메서드 제거
```

### God Method 템플릿

```markdown
#### God Method 감지

**위치**: `{파일경로}:{줄번호}` — `{함수명}()`
**지표**: {N}줄, CC={N}, 인지복잡도={N}, 매개변수={N}개
**심각도**: High

**문제**: 하나의 함수가 여러 작업을 수행하여 이해하기 어렵고 테스트가 복잡합니다.

**식별된 하위 작업**:
1. {작업 1} — {시작줄}-{끝줄}
2. {작업 2} — {시작줄}-{끝줄}
3. {작업 3} — {시작줄}-{끝줄}

**Before**:
```{language}
function {함수명}({매개변수}) {
  // 작업 1: {설명}
  {코드...}

  // 작업 2: {설명}
  {코드...}

  // 작업 3: {설명}
  {코드...}
}
```

**After**:
```{language}
function {함수명}({매개변수}) {
  const result1 = {하위함수1}({매개변수});
  const result2 = {하위함수2}(result1);
  return {하위함수3}(result2);
}

function {하위함수1}({매개변수}) {
  // 작업 1
}

function {하위함수2}({매개변수}) {
  // 작업 2
}

function {하위함수3}({매개변수}) {
  // 작업 3
}
```
```

### Feature Envy 템플릿

```markdown
#### Feature Envy 감지

**위치**: `{파일경로}:{줄번호}` — `{클래스명}.{메서드명}()`
**지표**: 외부 참조 {N}회 vs 자체 참조 {N}회 (외부 참조 비율 {N}%)
**대상 클래스**: `{참조 대상 클래스명}`
**심각도**: Medium

**문제**: 이 메서드는 자신의 클래스보다 `{참조 대상 클래스명}`의 데이터를 더 많이 사용합니다.

**Before**:
```{language}
class OrderPrinter {
  printOrder(order: Order) {
    console.log(`주문번호: ${order.getId()}`);
    console.log(`고객: ${order.getCustomerName()}`);
    console.log(`상품: ${order.getItems().map(i => i.getName()).join(", ")}`);
    console.log(`총액: ${order.getTotalPrice()}`);
    console.log(`배송지: ${order.getShippingAddress()}`);
  }
}
```

**After**:
```{language}
class Order {
  // ... 기존 코드 ...

  formatForPrint(): string {
    return [
      `주문번호: ${this.id}`,
      `고객: ${this.customerName}`,
      `상품: ${this.items.map(i => i.name).join(", ")}`,
      `총액: ${this.totalPrice}`,
      `배송지: ${this.shippingAddress}`,
    ].join("\n");
  }
}

class OrderPrinter {
  printOrder(order: Order) {
    console.log(order.formatForPrint());
  }
}
```
```

### Long Parameter List 템플릿

```markdown
#### Long Parameter List 감지

**위치**: `{파일경로}:{줄번호}` — `{함수명}()`
**지표**: 매개변수 {N}개 (임계값: 5개)
**심각도**: Medium

**문제**: 매개변수가 많아 함수 호출이 복잡하고 실수하기 쉽습니다.

**Before**:
```{language}
function createUser(
  name: string,
  email: string,
  age: number,
  address: string,
  phone: string,
  role: string,
  department: string
) { ... }
```

**After**:
```{language}
interface CreateUserParams {
  name: string;
  email: string;
  age: number;
  address: string;
  phone: string;
  role: string;
  department: string;
}

function createUser(params: CreateUserParams) { ... }

// 호출
createUser({
  name: "홍길동",
  email: "hong@example.com",
  age: 30,
  address: "서울시 강남구",
  phone: "010-1234-5678",
  role: "developer",
  department: "engineering",
});
```
```

### Primitive Obsession 템플릿

```markdown
#### Primitive Obsession 감지

**위치**: `{파일경로}`
**지표**: `{접두사}` 관련 원시 타입 변수 {N}개가 그룹으로 반복 사용
**심각도**: Medium

**문제**: 관련된 데이터가 별도의 원시 타입 변수로 흩어져 있어 응집도가 낮습니다.

**Before**:
```{language}
function processAddress(
  street: string,
  city: string,
  state: string,
  zipCode: string,
  country: string
) {
  const fullAddress = `${street}, ${city}, ${state} ${zipCode}, ${country}`;
  // ...
}
```

**After**:
```{language}
class Address {
  constructor(
    readonly street: string,
    readonly city: string,
    readonly state: string,
    readonly zipCode: string,
    readonly country: string,
  ) {}

  format(): string {
    return `${this.street}, ${this.city}, ${this.state} ${this.zipCode}, ${this.country}`;
  }

  validate(): boolean {
    return !!(this.street && this.city && this.zipCode && this.country);
  }
}

function processAddress(address: Address) {
  const fullAddress = address.format();
  // ...
}
```
```

### Data Clumps 템플릿

```markdown
#### Data Clumps 감지

**위치**: {N}곳에서 동일 매개변수 그룹 반복
**매개변수 그룹**: `({매개변수1}, {매개변수2}, {매개변수3})`
**반복 위치**:
  - `{파일1}:{줄번호}` — `{함수명1}()`
  - `{파일2}:{줄번호}` — `{함수명2}()`
  - `{파일3}:{줄번호}` — `{함수명3}()`
**심각도**: Medium

**문제**: 동일한 매개변수 조합이 여러 곳에서 반복되어 변경 시 모든 위치를 수정해야 합니다.

**Before**:
```{language}
function calculateDistance(lat1: number, lng1: number, lat2: number, lng2: number) { ... }
function formatLocation(lat: number, lng: number) { ... }
function isWithinRange(lat: number, lng: number, rangKm: number) { ... }
```

**After**:
```{language}
interface GeoPoint {
  latitude: number;
  longitude: number;
}

function calculateDistance(from: GeoPoint, to: GeoPoint) { ... }
function formatLocation(point: GeoPoint) { ... }
function isWithinRange(point: GeoPoint, rangeKm: number) { ... }
```
```

### Switch Statements 템플릿

```markdown
#### 반복적 Switch/If-Else 감지

**위치**: {N}곳에서 `{조건변수}`에 대한 동일 분기 반복
**반복 위치**:
  - `{파일1}:{줄번호}` — `{함수명1}()`
  - `{파일2}:{줄번호}` — `{함수명2}()`
**분기 값**: `{값1}`, `{값2}`, `{값3}`, ...
**심각도**: Medium

**문제**: 동일한 조건에 대한 분기가 여러 곳에 산재하여, 새로운 케이스 추가 시 모든 위치를 수정해야 합니다.

**Before**:
```{language}
// 위치 1
function getLabel(type: string): string {
  switch (type) {
    case "A": return "타입 A";
    case "B": return "타입 B";
    case "C": return "타입 C";
  }
}

// 위치 2
function getIcon(type: string): string {
  switch (type) {
    case "A": return "icon-a.svg";
    case "B": return "icon-b.svg";
    case "C": return "icon-c.svg";
  }
}
```

**After**:
```{language}
interface TypeConfig {
  label: string;
  icon: string;
}

const TYPE_MAP: Record<string, TypeConfig> = {
  A: { label: "타입 A", icon: "icon-a.svg" },
  B: { label: "타입 B", icon: "icon-b.svg" },
  C: { label: "타입 C", icon: "icon-c.svg" },
};

function getLabel(type: string): string {
  return TYPE_MAP[type]?.label ?? "알 수 없음";
}

function getIcon(type: string): string {
  return TYPE_MAP[type]?.icon ?? "default.svg";
}
```
```

### Dead Code 템플릿

```markdown
#### Dead Code 감지

**심각도**: Low
**유형**: {미사용 함수 / 미사용 변수 / 미사용 import / 주석 처리된 코드 / 도달 불가능 코드}

**발견 목록**:

| # | 유형 | 파일 | 줄 | 이름/내용 | 확인 방법 |
|---|------|------|-----|----------|-----------|
| 1 | 미사용 함수 | {파일} | {줄} | `{함수명}()` | 프로젝트 전체 Grep 결과 0건 |
| 2 | 미사용 import | {파일} | {줄} | `{모듈명}` | 파일 내 참조 0건 |
| 3 | 주석 코드 | {파일} | {줄-줄} | {N}줄 블록 | 주석 내 코드 구조 감지 |

**조치**: 위 항목을 삭제합니다. 버전 관리 시스템(git)에 이력이 남으므로 안전하게 제거할 수 있습니다.
```

### Magic Numbers/Strings 템플릿

```markdown
#### Magic Number/String 감지

**심각도**: Low

**발견 목록**:

| # | 파일 | 줄 | 현재 코드 | 제안 상수명 | 용도 |
|---|------|-----|----------|------------|------|
| 1 | {파일} | {줄} | `{리터럴값}` | `{상수명}` | {용도 설명} |
| 2 | {파일} | {줄} | `"{문자열}"` | `{상수명}` | {용도 설명} |

**Before**:
```{language}
if (retryCount > 3) { ... }
if (response.status === 429) { ... }
const timeout = 30000;
```

**After**:
```{language}
const MAX_RETRY_COUNT = 3;
const HTTP_TOO_MANY_REQUESTS = 429;
const REQUEST_TIMEOUT_MS = 30000;

if (retryCount > MAX_RETRY_COUNT) { ... }
if (response.status === HTTP_TOO_MANY_REQUESTS) { ... }
const timeout = REQUEST_TIMEOUT_MS;
```
```

---

## 우선순위 매트릭스 시각화

### 매트릭스 표시 형식

분석 결과의 모든 이슈를 4분면으로 분류하여 시각적으로 표시합니다.

```markdown
## 우선순위 매트릭스

```
 영향도
  ^
  │
높음│  [P2] 계획적 실행              [P1] 즉시 실행
  │  ┌─────────────────────┐     ┌─────────────────────┐
  │  │ {이슈 목록}          │     │ {이슈 목록}          │
  │  │                     │     │                     │
  │  └─────────────────────┘     └─────────────────────┘
  │
낮음│  [P4] 보류/후순위              [P3] 빈 시간에 처리
  │  ┌─────────────────────┐     ┌─────────────────────┐
  │  │ {이슈 목록}          │     │ {이슈 목록}          │
  │  │                     │     │                     │
  │  └─────────────────────┘     └─────────────────────┘
  │
  └──────────────────────────────────────────────────────> 용이성
                높은 난이도                   낮은 난이도
```

### 예시

```
 영향도
  ^
  │
높음│  [P2] 계획적 실행              [P1] 즉시 실행
  │  ┌─────────────────────┐     ┌─────────────────────┐
  │  │ - UserService 분할   │     │ - Dead Code 3건 제거 │
  │  │ - Strategy 패턴 적용 │     │ - Magic Number 5건  │
  │  │ - 중복 모듈 통합     │     │ - 미사용 import 8건  │
  │  └─────────────────────┘     └─────────────────────┘
  │
낮음│  [P4] 보류/후순위              [P3] 빈 시간에 처리
  │  ┌─────────────────────┐     ┌─────────────────────┐
  │  │ - 전체 네이밍 통일   │     │ - 약어 풀기 12건    │
  │  │ - 아키텍처 전환      │     │ - 주석 정리 5건     │
  │  │                     │     │ - 변수명 개선 8건    │
  │  └─────────────────────┘     └─────────────────────┘
  │
  └──────────────────────────────────────────────────────>
```
```

### 우선순위 판정 기준 테이블

```markdown
### 우선순위 판정 기준

| 코드 스멜 유형 | 기본 영향도 | 기본 난이도 | 기본 우선순위 |
|---------------|-----------|-----------|-------------|
| Dead Code | 높음 (버그 위험 감소) | 낮음 (삭제만) | P1 |
| Magic Numbers | 높음 (가독성) | 낮음 (상수 추출) | P1 |
| 미사용 import | 높음 (빌드 최적화) | 낮음 (삭제만) | P1 |
| Long Parameter List | 보통 | 낮음 | P1~P3 |
| God Method | 높음 | 보통 | P2 |
| God Class | 높음 | 높음 | P2 |
| Switch Statements | 보통 | 보통 | P2~P3 |
| Feature Envy | 보통 | 보통 | P2~P3 |
| Primitive Obsession | 보통 | 보통 | P2~P3 |
| Data Clumps | 보통 | 낮음 | P3 |
| 약어 사용 | 낮음 | 낮음 | P3 |
| 의미 없는 이름 | 낮음 | 낮음 | P3 |
| 네이밍 일관성 | 낮음 | 높음 (전체 수정) | P4 |
```

---

## 복잡도 대시보드 형식

파일/함수별 복잡도 메트릭을 대시보드 형태로 시각화합니다.

### 프로젝트 전체 복잡도 개요

```markdown
## 복잡도 대시보드

### 전체 통계

| 메트릭 | 값 | 상태 |
|--------|-----|------|
| 분석 파일 수 | {N}개 | |
| 분석 함수 수 | {N}개 | |
| 평균 CC (함수당) | {N} | {양호/주의/위험} |
| 최대 CC | {N} — `{함수명}()` | {양호/주의/위험} |
| CC > 10 함수 수 | {N}개 ({N}%) | |
| 평균 인지복잡도 | {N} | {양호/주의/위험} |
| 평균 함수 길이 | {N}줄 | {양호/주의/위험} |
| 30줄 초과 함수 수 | {N}개 ({N}%) | |
| 300줄 초과 파일 수 | {N}개 ({N}%) | |
| 중첩 4단계 초과 | {N}곳 | |

### 함수 복잡도 분포

```
Cyclomatic Complexity 분포:

 1-5   ████████████████████████████  {N}개 ({N}%)  양호
 6-10  ████████████████             {N}개 ({N}%)  양호
 11-15 ████████                     {N}개 ({N}%)  주의
 16-20 ████                         {N}개 ({N}%)  위험
 21+   ██                           {N}개 ({N}%)  매우 위험
```

### 상위 위험 함수 (Top 10)

```
순위  CC   인지  줄수  중첩  함수명
─── ──── ──── ──── ──── ────────────────────────
 1   {N}  {N}  {N}  {N}  {파일}:{함수명}()
 2   {N}  {N}  {N}  {N}  {파일}:{함수명}()
 3   {N}  {N}  {N}  {N}  {파일}:{함수명}()
 ...
```
```

### 파일별 복잡도 히트맵

```markdown
### 파일별 복잡도 히트맵

| 파일 | 줄수 | 함수수 | 평균CC | 최대CC | 스멜수 | 건강도 |
|------|------|--------|--------|--------|--------|--------|
| {파일1} | {N} | {N} | {N} | {N} | {N} | {바} |
| {파일2} | {N} | {N} | {N} | {N} | {N} | {바} |

건강도 범례:
  양호 = 모든 메트릭 임계값 이하
  주의 = 1-2개 메트릭 임계값 초과
  위험 = 3개 이상 메트릭 임계값 초과
```

---

## 개선 제안 카드 형식

각 발견 사항을 일관된 카드 형식으로 표시합니다.

### 개선 제안 카드 템플릿

```markdown
---

### [{P 우선순위}][{심각도}] {코드 스멜 유형}: {간결한 설명}

**위치**: `{파일경로}:{줄번호}` — `{함수/클래스명}`
**지표**: {관련 수치 (CC={N}, 줄수={N}, 매개변수={N} 등)}
**영향도**: {높음/보통/낮음} | **난이도**: {높음/보통/낮음}
**예상 소요**: {시간 추정}

**문제 설명**:
{이 코드가 왜 문제인지 간결하게 설명}

**Before**:
```{language}
{문제가 있는 현재 코드}
```

**After**:
```{language}
{개선된 코드}
```

**리팩토링 단계**:
1. {단계 1}
2. {단계 2}
3. {단계 3}

**영향 범위**: {리팩토링 시 수정이 필요한 다른 파일/함수 목록}

**확인 사항**:
- [ ] 기존 테스트 통과 확인
- [ ] {추가 확인 사항}

---
```

### 개선 제안 카드 예시 (실제)

```markdown
---

### [P1][High] God Method: processUserRegistration() 분할 필요

**위치**: `src/services/authService.ts:45` — `AuthService.processUserRegistration()`
**지표**: CC=24, 인지복잡도=31, 92줄, 중첩 최대 5단계
**영향도**: 높음 | **난이도**: 보통
**예상 소요**: 약 1시간

**문제 설명**:
회원가입 처리 함수가 입력 검증, 중복 확인, 비밀번호 해싱, DB 저장, 이메일 발송, 로깅까지 모두 담당하고 있어 테스트와 수정이 어렵습니다.

**Before**:
```typescript
async processUserRegistration(name: string, email: string, password: string, ...) {
  // 입력 검증 (15줄)
  if (!name || name.length < 2) throw new Error("...");
  if (!email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) throw new Error("...");
  // ... 15줄 더 ...

  // 중복 확인 (8줄)
  const existing = await this.userRepo.findByEmail(email);
  if (existing) throw new Error("이미 존재하는 이메일");
  // ...

  // 비밀번호 해싱 + DB 저장 + 이메일 발송 + 로깅 (69줄)
  // ...
}
```

**After**:
```typescript
async processUserRegistration(params: RegistrationParams) {
  this.validateRegistrationInput(params);
  await this.ensureUniqueEmail(params.email);
  const hashedPassword = await this.hashPassword(params.password);
  const user = await this.createUser({ ...params, password: hashedPassword });
  await this.sendWelcomeEmail(user);
  this.logger.info("회원가입 완료", { userId: user.id });
  return user;
}
```

**리팩토링 단계**:
1. `RegistrationParams` 인터페이스 생성 (Long Parameter List 동시 해결)
2. `validateRegistrationInput()` 추출 (입력 검증 로직)
3. `ensureUniqueEmail()` 추출 (중복 확인 로직)
4. `hashPassword()` 추출 (비밀번호 처리 로직)
5. `createUser()` 추출 (DB 저장 로직)
6. `sendWelcomeEmail()` 추출 (이메일 발송 로직)
7. 원래 함수를 오케스트레이션 함수로 변환

**영향 범위**:
- `src/controllers/authController.ts` — `processUserRegistration` 호출 부분 매개변수 변경
- `src/tests/authService.test.ts` — 테스트 케이스 업데이트

**확인 사항**:
- [ ] 기존 회원가입 테스트 통과
- [ ] 입력 검증 실패 케이스 테스트
- [ ] 중복 이메일 검사 테스트
- [ ] 이메일 발송 실패 시 롤백 동작 확인

---
```

---

## 에러 메시지 템플릿

분석 중 발생할 수 있는 에러 상황에 대한 사용자 메시지 템플릿입니다.

### 분석 대상 파일 없음

```
분석 대상 파일을 찾을 수 없습니다.

지정된 경로: {경로}

지원하는 파일 확장자:
  - JavaScript/TypeScript: .js, .jsx, .ts, .tsx
  - Python: .py
  - Java: .java
  - Go: .go
  - Ruby: .rb
  - C#: .cs

다음을 확인해 주세요:
1. 경로가 올바른지 확인
2. 지원되는 언어의 소스 파일이 포함되어 있는지 확인
3. 특정 파일을 분석하려면 파일 경로를 직접 지정
```

### 파일 읽기 실패

```
일부 파일을 읽을 수 없어 건너뛰었습니다.

건너뛴 파일:
  - {파일1}: {에러 사유}
  - {파일2}: {에러 사유}

나머지 {N}개 파일에 대한 분석은 정상적으로 완료되었습니다.
```

### 분석 범위 과대

```
분석 대상 파일이 {N}개로, 분석 시간이 오래 걸릴 수 있습니다.

범위를 좁혀서 분석하시겠습니까?

제안:
1. 특정 디렉토리만 분석: "src/services/ 디렉토리만 분석해줘"
2. 변경된 파일만 분석: "변경된 파일만 리팩토링 분석해줘"
3. 특정 파일만 분석: "{파일명} 파일만 분석해줘"
4. 전체 분석 계속: "그래도 전체 분석해줘"
```

### 지원하지 않는 언어

```
{확장자} 파일에 대한 언어별 상세 분석은 지원되지 않습니다.

범용 분석(줄 수, 중복 코드, 네이밍)은 수행되었습니다.
언어별 상세 분석(복잡도 메트릭, 코드 스멜)은 제한됩니다.

지원하는 언어:
  JavaScript, TypeScript, Python, Java, Go, Ruby, C#
```

### Git 저장소 아님

```
현재 디렉토리가 git 저장소가 아닙니다.

git diff 기반 변경 파일 분석 대신 전체 디렉토리 스캔으로 전환합니다.

분석 대상: {경로}
발견된 소스 파일: {N}개
```
