# Error Handling Analyzer

## 스킬 소개

**Error Handling Analyzer**는 Claude Code 커스텀 스킬로, 코드베이스의 에러 처리 패턴을 정적 분석하여 안티패턴과 관측성(observability) 누락을 탐지합니다. 빈 catch 블록, 비구조화 로깅, API 에러 응답 불일치, 에러 바운더리 미설정, 외부 서비스 호출의 복원력 부족 등을 식별하고, 심각도별 분류와 함께 구체적인 Before/After 개선 가이드를 제공합니다.

### 주요 기능

1. **에러 처리 안티패턴 탐지**: 빈 catch 블록, 에러 삼킴(swallowing), console.log 기반 로깅, 미처리 Promise 거부, 제네릭 에러 타입 캐칭 등 5가지 핵심 안티패턴을 식별합니다.

2. **API 에러 응답 일관성 검사**: HTTP 상태 코드 사용 적절성, RFC 7807 (Problem Details) 준수 여부, 에러 응답 포맷 통일성, 엣지 케이스에 대한 누락된 에러 상태 코드를 분석합니다.

3. **에러 바운더리 탐지**: React ErrorBoundary, Vue errorHandler, Express 전역 에러 미들웨어, Spring @ControllerAdvice, Node.js 프로세스 레벨 핸들러(uncaughtException, unhandledRejection) 등 프레임워크별 에러 바운더리 설정을 점검합니다.

4. **복원력 패턴 점검**: 외부 서비스 호출에 대한 재시도(Retry), 타임아웃(Timeout), 서킷 브레이커(Circuit Breaker), 폴백(Fallback) 전략 적용 여부를 확인하고 개선안을 제시합니다.

5. **Before/After 코드 예시 제공**: 각 발견 항목에 대해 취약한 코드와 개선된 코드를 병렬로 제시하여 즉시 적용 가능한 가이드를 제공합니다.

6. **연관 스킬 시너지**: refactor-advisor(에러 관련 리팩토링), security-scanner(A09 로깅 보안), test-coverage-analyzer(에러 시나리오 테스트)와 연계하여 종합적인 코드 품질 개선을 지원합니다.

---

## 분석 카테고리 요약

| 카테고리 | 분석 대상 | 주요 탐지 항목 |
|---------|----------|---------------|
| 안티패턴 탐지 | catch/except 블록 | 빈 catch, 에러 삼킴, console.log 의존, 미처리 Promise, 제네릭 캐칭 |
| API 에러 응답 | HTTP 응답 코드 | 상태 코드 오용, RFC 7807 미준수, 에러 포맷 불일치, 누락된 상태 코드 |
| 에러 바운더리 | 전역 에러 핸들러 | React ErrorBoundary, Vue errorHandler, Express 미들웨어, 프로세스 핸들러 |
| 복원력 패턴 | 외부 서비스 호출 | 재시도 미적용, 타임아웃 미설정, 서킷 브레이커 미적용, 폴백 전략 부재 |
| 로깅 품질 | 에러 로깅 방식 | 비구조화 로깅, 컨텍스트 누락, 민감 정보 로깅, 로그 레벨 오용 |

---

## 사용 예시

### 예시 1: 에러 처리 분석 요청

```
에러 처리 분석해줘
```

분석 결과:
```
[HIGH] 빈 catch 블록 발견
  파일: src/services/userService.js:45
  패턴: catch 블록에서 에러를 무시하고 있습니다

[MEDIUM] console.log 기반 에러 로깅
  파일: src/controllers/orderController.js:78
  패턴: 구조화된 로거 대신 console.error를 사용하고 있습니다

[HIGH] 미처리 Promise 거부
  파일: src/utils/dataFetcher.js:23
  패턴: .then()에 .catch()가 누락되어 있습니다
```

### 예시 2: API 에러 응답 점검

```
API 에러 응답 일관성 확인해줘
```

분석 결과:
```
[MEDIUM] 에러 응답 포맷 불일치
  - /api/users: { error: "message" }
  - /api/orders: { message: "error", code: 123 }
  - /api/products: { err: { msg: "error" } }
  권장: RFC 7807 Problem Details 포맷으로 통일

[HIGH] 부적절한 상태 코드 사용
  파일: src/routes/auth.js:34
  패턴: 인증 실패 시 500 대신 401을 반환해야 합니다
```

### 예시 3: 복원력 패턴 점검

```
외부 서비스 호출의 복원력 패턴 점검해줘
```

분석 결과:
```
[HIGH] 타임아웃 미설정
  파일: src/services/paymentService.js:56
  패턴: 결제 API 호출에 타임아웃이 설정되지 않았습니다

[MEDIUM] 재시도 로직 미적용
  파일: src/services/notificationService.js:23
  패턴: 알림 API 호출 실패 시 재시도하지 않습니다

[LOW] 서킷 브레이커 미적용
  파일: src/services/recommendationService.js:12
  패턴: 추천 서비스 장애 시 폴백 전략이 없습니다
```

### 예시 4: 에러 바운더리 점검

```
에러 바운더리 설정 확인해줘
```

분석 결과:
```
[CRITICAL] 프로세스 레벨 에러 핸들러 미설정
  파일: src/index.js
  패턴: uncaughtException, unhandledRejection 핸들러가 없습니다

[HIGH] React ErrorBoundary 미설정
  파일: src/App.jsx
  패턴: 최상위 컴포넌트에 ErrorBoundary가 없습니다
```

---

## 심각도 분류

| 심각도 | 설명 | 대응 시간 |
|--------|------|----------|
| Critical | 서비스 전체 중단 또는 민감 정보 노출 가능 | 즉시 |
| High | 핵심 기능의 에러 처리 누락 또는 부적절 | 48시간 내 |
| Medium | 로깅 품질 저하 또는 에러 응답 불일치 | 1주일 내 |
| Low | 복원력 패턴 미적용 또는 표준 미준수 | 계획적 |
| Info | 모범 사례 권장 사항 | 참고 |

---

## 지원 언어 및 프레임워크

| 언어/프레임워크 | 파일 확장자 | 주요 분석 항목 |
|---------------|-----------|---------------|
| JavaScript | `.js`, `.jsx`, `.mjs`, `.cjs` | try/catch, Promise, console.log, fetch/axios |
| TypeScript | `.ts`, `.tsx` | JavaScript + 타입 기반 에러 핸들링 |
| Python | `.py` | try/except, logging, requests |
| Java | `.java` | try/catch, @ControllerAdvice, @ExceptionHandler |
| Go | `.go` | error 반환값 검사, panic/recover |
| React | `.jsx`, `.tsx` | ErrorBoundary, componentDidCatch |
| Vue | `.vue` | errorHandler, onErrorCaptured |
| Express.js | `.js`, `.ts` | 에러 미들웨어, uncaughtException |
| Spring Boot | `.java` | @ControllerAdvice, ResponseEntityExceptionHandler |
| Django/Flask | `.py` | errorhandler, process_exception |

---

## 관련 문서

- [SKILL.md](SKILL.md) - 스킬 정의 및 상세 워크플로우
- [references/analysis-guide.md](references/analysis-guide.md) - 분석 방법론, 탐지 패턴 상세, 체크리스트
- [references/templates.md](references/templates.md) - 보고서 템플릿 및 개선 가이드 템플릿
