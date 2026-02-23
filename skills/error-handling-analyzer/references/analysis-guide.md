# Error Handling Analyzer 분석 가이드

이 문서는 에러 처리 분석의 상세 방법론, 언어별 안티패턴 탐지 정규식, 프레임워크별 에러 바운더리 검증 방법, 복원력 패턴 분석 절차, false positive 판단 기준, 심각도 분류 매트릭스를 정의합니다.

---

## 1. 안티패턴 탐지 상세 정규식

### 1.1 빈 catch 블록 패턴

```regex
# JavaScript/TypeScript - 완전히 빈 catch
catch\s*\(\s*\w*\s*\)\s*\{\s*\}

# JavaScript/TypeScript - 주석만 있는 catch
catch\s*\(\s*\w+\s*\)\s*\{\s*(//[^\n]*\n?\s*)*\}

# JavaScript/TypeScript - TODO만 있는 catch
catch\s*\(\s*\w+\s*\)\s*\{\s*(//\s*TODO[^\n]*\n?\s*)*\}

# Python - pass만 있는 except
except\s*(\s+\w+(\s*,\s*\w+)*)?\s*:\s*\n\s+pass\s*$

# Python - ... (Ellipsis)만 있는 except
except\s*.*:\s*\n\s+\.\.\.\s*$

# Java - 빈 catch 블록
catch\s*\(\s*\w+(\s+\w+)+\s*\)\s*\{\s*\}

# Go - 에러 반환값 무시 (언더스코어)
\w+\s*,\s*_\s*:?=\s*\w+\s*\(
```

### 1.2 에러 삼킴(Swallowing) 패턴

```regex
# console.log만으로 에러 처리 완료
catch\s*\(\s*(\w+)\s*\)\s*\{\s*\n?\s*console\.(log|warn)\s*\([^)]*\)\s*;?\s*\n?\s*\}

# 에러 무시 후 기본값 반환
catch\s*\(\s*\w+\s*\)\s*\{\s*\n?\s*return\s+(null|undefined|false|0|''|""|``|\[\]|\{\})\s*;?\s*\n?\s*\}

# Python - 에러 무시 후 None 반환
except\s*.*:\s*\n\s+return\s+None\s*$

# Python - 에러 무시 후 빈 값 반환
except\s*.*:\s*\n\s+return\s+(\[\]|\{\}|''|""|0|False)\s*$

# Java - 에러 무시 후 null 반환
catch\s*\(.*\)\s*\{\s*\n?\s*return\s+null\s*;\s*\n?\s*\}
```

### 1.3 비구조화 로깅 패턴

```regex
# 에러 핸들링에서 console.log 사용
catch\s*\([^)]*\)\s*\{[\s\S]*?console\.(log|warn|info|error|debug)\s*\(

# Python - print로 에러 출력
except\s*.*:\s*\n\s+print\s*\(

# Java - System.out.println으로 에러 출력
catch\s*\(.*\)\s*\{[\s\S]*?System\.(out|err)\.print(ln)?\s*\(

# Java - e.printStackTrace() 단독 사용
catch\s*\(.*\s+(\w+)\s*\)\s*\{\s*\n?\s*\1\.printStackTrace\s*\(\s*\)\s*;?\s*\n?\s*\}

# 에러 로깅에 컨텍스트 정보 누락 (단순 에러 객체만 출력)
(logger|log)\.(error|warn)\s*\(\s*\w+\s*\)\s*;
```

### 1.4 미처리 Promise 패턴

```regex
# .then()만 있고 .catch() 누락
\.then\s*\([^)]*\)\s*;

# .then() 체인에서 마지막에 .catch() 누락
\.then\s*\([^)]*\)(\s*\.then\s*\([^)]*\))*\s*;

# async 함수 호출에서 에러 처리 누락 (await 없이 호출)
(?<!await\s)(?<!return\s)\b\w+\s*\(\s*[^)]*\)\s*;(?=\s*//.*async|\s*/\*.*async)

# Promise.all에 에러 처리 누락
Promise\.(all|allSettled|race|any)\s*\(\s*\[.*\]\s*\)(?!\s*\.(catch|then))
```

### 1.5 제네릭 에러 캐칭 패턴

```regex
# Python - bare except
^(\s*)except\s*:\s*$

# Python - Exception 기본 클래스 캐칭 (최상위)
^(\s*)except\s+Exception\s*(as\s+\w+\s*)?:\s*$

# Python - BaseException 캐칭
^(\s*)except\s+BaseException\s*(as\s+\w+\s*)?:\s*$

# Java - 최상위 Exception 캐칭
catch\s*\(\s*Exception\s+\w+\s*\)

# Java - Throwable 캐칭
catch\s*\(\s*Throwable\s+\w+\s*\)

# Java - Error 캐칭 (java.lang.Error)
catch\s*\(\s*Error\s+\w+\s*\)
```

---

## 2. API 에러 응답 분석 방법

### 2.1 상태 코드 수집 절차

1. 모든 라우트/엔드포인트 파일을 식별합니다
2. `res.status()` 또는 `response.status_code` 호출을 추출합니다
3. catch 블록 내 상태 코드와 정상 경로 상태 코드를 분리합니다
4. 각 엔드포인트별로 반환 가능한 상태 코드 목록을 작성합니다

**수집 정규식**:

```regex
# Express.js - 상태 코드 추출
res\.status\s*\(\s*(\d{3})\s*\)

# Koa - 상태 코드 추출
ctx\.(status|response\.status)\s*=\s*(\d{3})

# Django - 상태 코드 추출
(HttpResponse|JsonResponse)\s*\(.*status\s*=\s*(\d{3})
return\s+Response\s*\(.*status\s*=\s*(status\.HTTP_\w+|\d{3})

# Spring - 상태 코드 추출
ResponseEntity\.(ok|created|badRequest|notFound|status)\s*\(
@ResponseStatus\s*\(\s*(value\s*=\s*)?(HttpStatus\.\w+)
HttpStatus\.(\w+)

# Flask - 상태 코드 추출
return\s+.*,\s*(\d{3})
abort\s*\(\s*(\d{3})\s*\)
```

### 2.2 에러 응답 포맷 일관성 검사

프로젝트 내 모든 에러 응답의 JSON 구조를 수집하고 비교합니다.

**탐지 대상 불일치 패턴**:

| 불일치 유형 | 예시 | 문제점 |
|------------|------|--------|
| 필드명 불일치 | `error` vs `message` vs `err` | 클라이언트에서 일관된 처리 불가 |
| 구조 불일치 | `{ error: "msg" }` vs `{ error: { message: "msg" } }` | 파싱 로직 분기 필요 |
| 상태 코드 불일치 | 에러인데 200 반환 | 클라이언트에서 에러 식별 불가 |
| 컨텐츠 타입 불일치 | JSON vs HTML vs 텍스트 | Content-Type 기반 처리 불가 |

### 2.3 엣지 케이스 에러 응답 누락 분석

각 엔드포인트에서 다음 시나리오에 대한 에러 응답이 구현되어 있는지 확인합니다:

```
엔드포인트별 필수 에러 응답 체크리스트:

GET /resource/:id
  - [ ] 404: 리소스를 찾을 수 없음
  - [ ] 401: 인증 필요
  - [ ] 403: 권한 없음

POST /resource
  - [ ] 400: 요청 데이터 유효성 검증 실패
  - [ ] 401: 인증 필요
  - [ ] 409: 리소스 충돌 (중복)
  - [ ] 422: 처리 불가능한 엔티티

PUT/PATCH /resource/:id
  - [ ] 400: 요청 데이터 유효성 검증 실패
  - [ ] 404: 리소스를 찾을 수 없음
  - [ ] 409: 동시성 충돌 (Optimistic Locking)

DELETE /resource/:id
  - [ ] 404: 리소스를 찾을 수 없음
  - [ ] 403: 삭제 권한 없음

공통
  - [ ] 429: Rate limiting 초과
  - [ ] 500: 예상치 못한 서버 오류
  - [ ] 502/504: 외부 서비스 장애/타임아웃
```

---

## 3. 에러 바운더리 검증 상세

### 3.1 React 프로젝트 분석 절차

1. `package.json`에서 React 의존성 확인
2. 최상위 App 컴포넌트 파일 식별 (App.jsx, App.tsx, _app.tsx 등)
3. ErrorBoundary 컴포넌트 또는 `react-error-boundary` 사용 여부 확인
4. ErrorBoundary의 fallback UI 정의 여부 확인
5. 라우트별 독립적인 ErrorBoundary 설정 여부 확인
6. 에러 보고 서비스 연동 여부 확인

**검증 정규식**:

```regex
# ErrorBoundary 클래스 컴포넌트
class\s+\w+\s+extends\s+.*Component[\s\S]*?componentDidCatch
class\s+\w+\s+extends\s+.*Component[\s\S]*?getDerivedStateFromError

# react-error-boundary 라이브러리
import\s+.*ErrorBoundary.*from\s+['"]react-error-boundary['"]
import\s+.*useErrorHandler.*from\s+['"]react-error-boundary['"]

# ErrorBoundary JSX 사용
<ErrorBoundary[\s>]
<ErrorBoundary\s+.*FallbackComponent
<ErrorBoundary\s+.*fallback\s*=
<ErrorBoundary\s+.*onError\s*=

# Sentry React 연동
import\s+.*Sentry.*from\s+['"]@sentry/react['"]
Sentry\.ErrorBoundary
Sentry\.withErrorBoundary
```

### 3.2 Vue 프로젝트 분석 절차

1. `package.json`에서 Vue 의존성 및 버전 확인 (Vue 2 vs Vue 3)
2. 메인 진입점 파일 식별 (main.js, main.ts)
3. `app.config.errorHandler` 설정 여부 확인
4. 컴포넌트에서 `onErrorCaptured` 훅 사용 여부 확인

**검증 정규식**:

```regex
# Vue 3 전역 에러 핸들러
app\.config\.errorHandler\s*=\s*(function|\()

# Vue 2 전역 에러 핸들러
Vue\.config\.errorHandler\s*=\s*(function|\()

# Vue 3 경고 핸들러
app\.config\.warnHandler\s*=

# Composition API 에러 캐칭
onErrorCaptured\s*\(\s*(function|\()

# Options API 에러 캐칭
errorCaptured\s*\(\s*\)|\s*errorCaptured\s*:\s*(function|\()
```

### 3.3 Node.js/Express 프로젝트 분석 절차

1. 메인 진입점 파일 식별 (index.js, app.js, server.js, main.ts)
2. 프로세스 레벨 핸들러 확인 (uncaughtException, unhandledRejection)
3. Express 전역 에러 미들웨어 확인 (4개 인자 미들웨어)
4. 라우터별 에러 처리 확인

**검증 정규식**:

```regex
# uncaughtException 핸들러
process\.on\s*\(\s*['"]uncaughtException['"]\s*,

# unhandledRejection 핸들러
process\.on\s*\(\s*['"]unhandledRejection['"]\s*,

# SIGTERM 핸들러 (Graceful Shutdown)
process\.on\s*\(\s*['"]SIGTERM['"]\s*,
process\.on\s*\(\s*['"]SIGINT['"]\s*,

# Express 전역 에러 미들웨어 (err, req, res, next 4개 인자)
app\.use\s*\(\s*(async\s+)?\(\s*(err|error)\s*,\s*(req|request)\s*,\s*(res|response)\s*,\s*next\s*\)

# 404 핸들러
app\.use\s*\(\s*(async\s+)?\(\s*(req|request)\s*,\s*(res|response)[\s\S]*?404
```

### 3.4 Spring Boot 프로젝트 분석 절차

1. `pom.xml` 또는 `build.gradle`에서 Spring Boot 의존성 확인
2. `@ControllerAdvice` 또는 `@RestControllerAdvice` 클래스 존재 여부 확인
3. `@ExceptionHandler` 메서드별 처리 대상 예외 확인
4. `ResponseEntityExceptionHandler` 상속 여부 확인

**검증 정규식**:

```regex
# 전역 예외 핸들러 클래스
@(Controller|RestController)Advice
@ControllerAdvice\s*(\([^)]*\))?

# 예외 핸들러 메서드
@ExceptionHandler\s*\(\s*(\w+\.class|\{[^}]+\})\s*\)

# ResponseEntityExceptionHandler 상속
extends\s+ResponseEntityExceptionHandler

# Spring Security 에러 핸들러
AuthenticationEntryPoint|AccessDeniedHandler
```

### 3.5 Django/Flask 프로젝트 분석 절차

**Django 검증 정규식**:

```regex
# 미들웨어 에러 처리
def\s+process_exception\s*\(\s*self\s*,\s*request\s*,\s*exception

# 커스텀 에러 핸들러
handler400\s*=|handler403\s*=|handler404\s*=|handler500\s*=

# DRF 예외 핸들러
EXCEPTION_HANDLER.*=
custom_exception_handler|def\s+\w+_exception_handler
```

**Flask 검증 정규식**:

```regex
# 에러 핸들러 데코레이터
@app\.errorhandler\s*\(\s*(\d{3}|\w+Error)\s*\)
@\w+\.errorhandler\s*\(\s*(\d{3}|\w+Error)\s*\)

# register_error_handler 호출
app\.register_error_handler\s*\(\s*(\d{3})\s*,

# Flask-RESTful 에러 처리
Api\s*\(.*errors\s*=
```

---

## 4. 복원력 패턴 분석 상세

### 4.1 재시도 패턴 분석

**분석 절차**:

1. 외부 HTTP 호출 지점을 모두 식별합니다
2. 각 호출 지점에서 재시도 설정이 있는지 확인합니다
3. 재시도 전략의 적절성을 평가합니다

**적절한 재시도 전략 기준표**:

| 항목 | 권장 설정 | 위험 설정 |
|------|----------|----------|
| 최대 재시도 횟수 | 3~5회 | 무제한 또는 0 |
| 백오프 전략 | 지수 백오프 (Exponential Backoff) | 고정 간격 또는 즉시 재시도 |
| 재시도 대상 | 5xx, 네트워크 에러, 타임아웃 | 모든 에러 (4xx 포함) |
| 지터(Jitter) | 랜덤 지터 추가 | 지터 없음 (Thundering Herd) |
| 멱등성 확인 | 멱등 요청만 재시도 | POST 등 비멱등 요청 무조건 재시도 |

**재시도 라이브러리 탐지**:

```regex
# JavaScript/TypeScript
(axios-retry|retry-axios|p-retry|async-retry|got\.retry|fetch-retry|exponential-backoff)

# Python
(tenacity|retrying|backoff|urllib3\.util\.retry|requests\.adapters\.HTTPAdapter)

# Java
(resilience4j.*retry|spring-retry|@Retryable|RetryTemplate|Retryer)

# Go
(go-retryablehttp|backoff|retry-go)
```

### 4.2 타임아웃 패턴 분석

**분석 절차**:

1. 외부 호출 지점(HTTP, DB, 메시지 큐 등)을 식별합니다
2. 각 호출에 타임아웃이 설정되어 있는지 확인합니다
3. 타임아웃 값의 적절성을 평가합니다

**권장 타임아웃 범위**:

| 호출 유형 | 연결 타임아웃 | 응답 타임아웃 | 전체 타임아웃 |
|----------|-------------|-------------|-------------|
| 내부 API 호출 | 1~3초 | 3~5초 | 5~10초 |
| 외부 API 호출 | 3~5초 | 5~15초 | 10~30초 |
| 데이터베이스 쿼리 | 1~3초 | 5~30초 | 10~60초 |
| 파일 업로드 | 5~10초 | 30~120초 | 60~300초 |
| 결제 API | 3~5초 | 10~30초 | 15~60초 |

**타임아웃 미설정 탐지 정규식**:

```regex
# fetch (기본 타임아웃 없음)
\bfetch\s*\(\s*[^)]+\)(?![\s\S]*?(signal|AbortController|timeout))

# axios (기본 타임아웃 0 = 무한)
axios\.(get|post|put|delete|patch|request)\s*\((?![\s\S]*?timeout)

# Python requests (기본 타임아웃 None = 무한)
requests\.(get|post|put|delete|patch|head)\s*\([^)]*\)(?![\s\S]*?timeout)

# Node.js http/https
https?\.(?:get|request)\s*\((?![\s\S]*?timeout)

# 데이터베이스 클라이언트
(pool|client|connection)\.(query|execute)\s*\((?![\s\S]*?(timeout|statement_timeout|lock_timeout))
```

### 4.3 서킷 브레이커 분석

**분석 절차**:

1. 핵심 외부 서비스 의존성을 식별합니다
2. 각 의존성에 서킷 브레이커가 적용되어 있는지 확인합니다
3. 서킷 브레이커 설정의 적절성을 평가합니다

**서킷 브레이커 라이브러리 탐지**:

```regex
# JavaScript/TypeScript
(opossum|cockatiel|brakes|mollitia|@nestjs\/terminus)

# Python
(pybreaker|circuitbreaker|aiobreaker)

# Java
(resilience4j.*circuitbreaker|Hystrix|@CircuitBreaker|CircuitBreakerRegistry)

# Go
(gobreaker|go-circuitbreaker|hystrix-go)
```

**권장 서킷 브레이커 설정**:

| 설정 항목 | 권장값 | 설명 |
|----------|-------|------|
| 실패 임계값 | 50~60% | 이 비율 이상 실패 시 서킷 오픈 |
| 슬라이딩 윈도우 크기 | 10~20 요청 | 실패율 계산에 사용하는 샘플 크기 |
| 오픈 지속 시간 | 30~60초 | 서킷 오픈 후 Half-Open 전환까지 대기 시간 |
| Half-Open 허용 요청 | 3~5 요청 | 서비스 복구 확인을 위한 시험 요청 수 |
| 폴백 전략 | 필수 | 캐시, 기본값, 대체 서비스 중 하나 |

### 4.4 폴백 전략 분석

**탐지 절차**:

1. 서킷 브레이커의 폴백 함수 정의 여부 확인
2. catch 블록에서 대체 데이터 소스 접근 여부 확인
3. 캐시 기반 폴백 구현 여부 확인
4. Graceful Degradation 전략 확인

**폴백 패턴 탐지 정규식**:

```regex
# 서킷 브레이커 폴백
\.fallback\s*\(|fallbackMethod\s*=

# 캐시 기반 폴백
catch[\s\S]*?(cache\.get|getFromCache|redis\.get|memcached\.get)

# 기본값 폴백
catch[\s\S]*?(DEFAULT_|default|fallback|FALLBACK)

# 대체 서비스 폴백
catch[\s\S]*?(backup|secondary|alternative|failover)
```

---

## 5. False Positive 판단 기준

오탐(false positive)을 최소화하기 위한 판단 기준입니다.

### 5.1 의도적인 에러 무시 (정당한 사유)

| 패턴 | 정당한 사유 | 판정 |
|------|-----------|------|
| `catch (e) { /* intentionally empty */ }` | 명시적 주석으로 의도 표시 | FP |
| `catch (e) { /* optional feature */ }` | 선택적 기능의 실패 | FP (Info로 기록) |
| `try { fs.unlinkSync(path); } catch {}` | 파일 삭제 실패 무시 (이미 없는 경우) | FP (Info로 기록) |
| `catch (e) { // 이 에러는 무시해도 안전합니다: ... }` | 사유가 명시된 주석 | FP |
| `.catch(() => {})` (cleanup에서) | 정리 작업 실패 무시 | FP (Info로 기록) |

**의도적 무시 탐지 정규식**:
```regex
catch\s*\([^)]*\)\s*\{\s*(//|/\*)\s*(intentional|ignore|suppress|optional|expected|safe to ignore|무시|의도적)
```

### 5.2 테스트 코드의 에러 패턴

| 패턴 | 사유 | 판정 |
|------|------|------|
| `expect(() => fn()).toThrow()` | 에러 발생 테스트 | FP |
| `assert.throws(() => fn())` | 에러 발생 테스트 | FP |
| `try { fn(); fail() } catch (e) { expect(e)... }` | 에러 검증 테스트 | FP |
| `with pytest.raises(...)` | Python 에러 테스트 | FP |

### 5.3 파일 경로 기반 제외

```
# FP 가능성이 높은 경로
*/test/*
*/tests/*
*/__tests__/*
*/spec/*
*/__mocks__/*
*/fixtures/*
*/examples/*
*/demo/*
*/node_modules/*
*/vendor/*
*/.git/*
*/dist/*
*/build/*
*.test.js
*.test.ts
*.spec.js
*.spec.ts
*_test.py
*_test.go
```

### 5.4 프레임워크별 예외 사항

| 프레임워크 | 패턴 | 사유 | 판정 |
|-----------|------|------|------|
| Express | `app.use((err, req, res, next) => { ... })` | 전역 에러 미들웨어 | 해당 패턴은 에러 바운더리로 인정 |
| React Query | `useQuery({ retry: false })` | 라이브러리 자체 재시도 관리 | FP |
| Axios Interceptor | `axios.interceptors.response.use(null, handler)` | 인터셉터 기반 에러 처리 | 에러 처리 존재로 인정 |
| Django REST | `raise serializers.ValidationError(...)` | DRF의 에러 처리 메커니즘 | FP |
| Spring | `@ResponseStatus(HttpStatus.NOT_FOUND)` | 선언적 에러 응답 | 에러 응답으로 인정 |

### 5.5 심각도 조정 규칙

| 조건 | 조정 |
|------|------|
| 테스트 파일에서 발견 | 보고 대상에서 제외 |
| 주석 내 발견 | 보고 대상에서 제외 |
| 의도적 무시 주석이 있는 경우 | Info로 하향 |
| 개발 전용 코드 (devDependencies 관련) | 심각도 1단계 하향 |
| 비핵심 기능 (로깅, 분석 등) | 심각도 1단계 하향 |
| 핵심 비즈니스 로직 (결제, 인증 등) | 심각도 1단계 상향 |
| 프로덕션 배포 코드 | 심각도 유지 |

---

## 6. 심각도 분류 매트릭스

### 6.1 분류 기준표

| 심각도 | 영향 범위 | 디버깅 난이도 | 사용자 영향 | 대응 시간 |
|--------|----------|-------------|-----------|----------|
| Critical | 시스템 전체 | 매우 어려움 | 서비스 중단 | 즉시 |
| High | 주요 기능 | 어려움 | 기능 장애 | 48시간 |
| Medium | 일부 기능 | 보통 | 경험 저하 | 1주일 |
| Low | 제한적 | 쉬움 | 미미함 | 1개월 |
| Info | 없음 | N/A | 없음 | 계획적 |

### 6.2 패턴별 기본 심각도

| 패턴 | 기본 심각도 | 상향 조건 | 하향 조건 |
|------|-----------|----------|----------|
| 프로세스 핸들러 미설정 | Critical | - | 개발 환경 전용 시 High |
| 민감 정보 클라이언트 노출 | Critical | - | 개발 환경 전용 시 High |
| 에러 바운더리 미설정 | High | 프로덕션 SPA 시 Critical | 서버 렌더링 시 Medium |
| 빈 catch (비즈니스 로직) | High | 결제/인증 관련 시 Critical | 유틸리티 함수 시 Medium |
| 타임아웃 미설정 | High | 결제 API 시 Critical | 내부 서비스 시 Medium |
| console.log 에러 로깅 | Medium | 프로덕션 배포 시 High | 개발 환경 시 Low |
| 에러 응답 포맷 불일치 | Medium | 공개 API 시 High | 내부 API 시 Low |
| 제네릭 에러 캐칭 | Medium | 핵심 로직 시 High | 유틸리티 시 Low |
| 재시도 미적용 | Medium | 핵심 서비스 시 High | 부가 기능 시 Low |
| 서킷 브레이커 미적용 | Low | 고트래픽 시 Medium | 저트래픽 시 Info |
| RFC 7807 미준수 | Low | 공개 API 시 Medium | 내부 전용 시 Info |
| 에러 코드 체계 미정의 | Info | - | - |

---

## 7. 분석 실행 순서

에러 처리 분석 시 다음 순서로 분석을 실행합니다:

### 1단계: 프로젝트 구조 식별

```bash
# 프로젝트 유형 및 프레임워크 감지
ls package.json requirements.txt pom.xml build.gradle go.mod 2>/dev/null

# 프레임워크 확인
cat package.json 2>/dev/null | grep -E '"(react|vue|express|koa|next|nuxt|nest)"'
```

### 2단계: 파일 수집 및 분류

```bash
# 분석 대상 파일 수집 (테스트 파일 제외)
find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.jsx" -o -name "*.tsx" -o -name "*.py" -o -name "*.java" -o -name "*.go" \) \
    -not -path "*/node_modules/*" \
    -not -path "*/.git/*" \
    -not -path "*/dist/*" \
    -not -path "*/build/*" \
    -not -path "*/__pycache__/*" \
    -not -name "*.test.*" \
    -not -name "*.spec.*" \
    -not -path "*/__tests__/*" \
    | head -5000
```

### 3단계: 안티패턴 탐지 (최우선)

빈 catch 블록, 에러 삼킴, 비구조화 로깅, 미처리 Promise, 제네릭 캐칭 순서로 탐지합니다.

### 4단계: 에러 바운더리 점검

프레임워크별 전역 에러 핸들러 및 프로세스 레벨 핸들러를 확인합니다.

### 5단계: API 에러 응답 분석

상태 코드 사용, 응답 포맷 일관성, 누락된 에러 상태 코드를 확인합니다.

### 6단계: 복원력 패턴 점검

외부 서비스 호출의 재시도, 타임아웃, 서킷 브레이커, 폴백 전략을 확인합니다.

### 7단계: False Positive 필터링

위의 FP 판단 기준을 적용하여 오탐을 제거합니다.

### 8단계: 심각도 분류 및 보고서 생성

매트릭스를 적용하여 최종 심각도를 결정하고, [references/templates.md](templates.md)의 형식으로 보고서를 생성합니다.

---

## 8. 분석 체크리스트

전체 분석 완료 시 다음 체크리스트를 통해 누락 여부를 확인합니다:

### 안티패턴

- [ ] 빈 catch 블록 탐지 완료
- [ ] 에러 삼킴 패턴 탐지 완료
- [ ] 비구조화 로깅 패턴 탐지 완료
- [ ] 미처리 Promise 거부 탐지 완료
- [ ] 제네릭 에러 캐칭 탐지 완료

### 에러 바운더리

- [ ] 프로세스 레벨 핸들러 확인 (uncaughtException, unhandledRejection)
- [ ] 프레임워크 전역 에러 핸들러 확인
- [ ] 라우트/컴포넌트별 에러 바운더리 확인
- [ ] Graceful Shutdown 핸들러 확인 (SIGTERM, SIGINT)

### API 에러 응답

- [ ] HTTP 상태 코드 적절성 확인
- [ ] 에러 응답 포맷 일관성 확인
- [ ] 민감 정보 노출 여부 확인
- [ ] 누락된 에러 상태 코드 확인

### 복원력

- [ ] 외부 HTTP 호출 타임아웃 설정 확인
- [ ] 데이터베이스 쿼리 타임아웃 설정 확인
- [ ] 재시도 로직 적용 여부 확인
- [ ] 서킷 브레이커 적용 여부 확인
- [ ] 폴백 전략 구현 여부 확인

### 로깅 품질

- [ ] 구조화된 로거 사용 여부 확인
- [ ] 에러 로깅에 컨텍스트 정보 포함 여부 확인
- [ ] 민감 정보 로깅 여부 확인
- [ ] 로그 레벨 적절성 확인 (error vs warn vs info)
