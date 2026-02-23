---
name: error-handling-analyzer
description: 코드베이스의 에러 처리 패턴을 분석하여 안티패턴, 관측성(observability) 누락, API 에러 응답 불일치, 복원력 부족 등을 탐지하고 개선 가이드를 제공하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) 에러 처리 분석, (2) 예외 처리 검사, (3) 로깅 분석, (4) 에러 바운더리, (5) 모니터링 설정, (6) error handling, (7) exception analysis, (8) 에러 응답 일관성 확인, (9) 복원력 패턴 점검.
---

# Error Handling Analyzer

코드베이스의 에러 처리 패턴을 정적 분석하여 안티패턴을 탐지하고, 구조화된 로깅 미적용, API 에러 응답 불일치, 에러 바운더리 누락, 외부 서비스 호출의 복원력 부족 등을 식별합니다. 각 발견 항목에 대해 심각도 분류와 Before/After 코드 예시를 포함한 구체적인 개선 가이드를 제공합니다.

## Quick Start

사용자가 에러 처리 분석을 요청하면 다음 워크플로우를 실행합니다:

1. **분석 범위 결정**:
   ```bash
   # 스테이징된 변경사항 분석
   git diff --staged --name-only --diff-filter=ACMR

   # 최근 커밋 대비 변경 파일 확인
   git diff --name-only --diff-filter=ACMR HEAD~5

   # 전체 코드베이스 분석 시
   find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.jsx" -o -name "*.tsx" -o -name "*.py" -o -name "*.java" -o -name "*.go" \) -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/dist/*" -not -path "*/build/*"
   ```

2. **에러 처리 안티패턴 탐지**: 빈 catch 블록, 에러 무시, 제네릭 에러 캐칭 등 패턴 매칭

3. **로깅 품질 분석**: console.log 사용 여부, 구조화된 로깅 미적용, 민감 정보 로깅 등 확인

4. **API 에러 응답 일관성 검사**: 상태 코드 사용, RFC 7807 준수 여부, 에러 응답 포맷 통일성

5. **에러 바운더리 점검**: React ErrorBoundary, Vue errorHandler, 전역 예외 핸들러 설정

6. **복원력 패턴 점검**: 외부 서비스 호출의 재시도/타임아웃/서킷 브레이커/폴백 전략

7. **결과를 심각도별 분류** (Critical / High / Medium / Low / Info)

8. **개선 가이드 및 보고서 생성** ([references/templates.md](references/templates.md) 형식 사용)

## 트리거 조건

이 스킬은 다음 상황에서 활성화됩니다:

- 사용자가 "에러 처리 분석", "예외 처리 검사" 요청 시
- "로깅 분석", "로깅 품질 확인" 요청 시
- "에러 바운더리" 설정 확인 또는 추가 요청 시
- "모니터링 설정", "관측성 확인" 요청 시
- "error handling", "exception analysis" 영문 요청 시
- "에러 응답 일관성 확인", "API 에러 포맷 점검" 요청 시
- "복원력 패턴 점검", "서킷 브레이커 확인" 요청 시
- "unhandled promise rejection 확인" 요청 시

## 1. 에러 처리 안티패턴 탐지

코드에서 에러를 부적절하게 처리하는 패턴을 식별합니다. 상세 분석 방법론은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

### 1.1 빈 catch 블록

에러를 캐치한 후 아무 처리도 하지 않는 패턴입니다. 에러가 조용히 무시되어 디버깅이 극도로 어려워집니다.

**탐지 정규식**:
```regex
# JavaScript/TypeScript - 빈 catch 블록
catch\s*\(\s*\w*\s*\)\s*\{\s*\}

# JavaScript/TypeScript - 주석만 있는 catch 블록
catch\s*\(\s*\w+\s*\)\s*\{\s*//.*\s*\}

# Python - 빈 except 블록
except\s*.*:\s*pass\s*$

# Java - 빈 catch 블록
catch\s*\(\s*\w+\s+\w+\s*\)\s*\{\s*\}
```

**취약한 코드**:
```javascript
try {
    const data = await fetchUserData(userId);
    return data;
} catch (error) {
    // TODO: 나중에 처리
}
```

**개선된 코드**:
```javascript
try {
    const data = await fetchUserData(userId);
    return data;
} catch (error) {
    logger.error('사용자 데이터 조회 실패', {
        userId,
        error: error.message,
        stack: error.stack,
    });
    throw new ApplicationError('USER_DATA_FETCH_FAILED', error);
}
```

### 1.2 제네릭 에러 삼킴 (Error Swallowing)

에러를 캐치한 후 console.log만 출력하거나, 에러의 원인 정보를 소실시키는 패턴입니다.

**탐지 정규식**:
```regex
# console.log로만 에러 출력
catch\s*\(\s*(\w+)\s*\)\s*\{[\s\n]*console\.log\s*\(\s*\1\s*\)\s*;?\s*\}

# 에러 정보 무시하고 단순 메시지만 반환
catch\s*\(\s*\w+\s*\)\s*\{[\s\n]*return\s+(null|undefined|false|''|""|``)\s*;?\s*\}

# Python - 에러 무시 후 None 반환
except\s*.*:\s*\n\s*return\s+None
```

**취약한 코드**:
```javascript
async function getUser(id) {
    try {
        return await db.users.findById(id);
    } catch (e) {
        console.log(e);
        return null;
    }
}
```

**개선된 코드**:
```javascript
async function getUser(id) {
    try {
        return await db.users.findById(id);
    } catch (error) {
        logger.error('사용자 조회 실패', {
            userId: id,
            errorCode: error.code,
            errorMessage: error.message,
        });
        throw new DatabaseError('USER_QUERY_FAILED', { cause: error });
    }
}
```

### 1.3 비구조화 로깅 (console.log 의존)

프로덕션 환경에서 `console.log`를 사용하여 에러를 기록하는 것은 구조화된 검색, 필터링, 모니터링이 불가능합니다.

**탐지 정규식**:
```regex
# console.log를 에러 로깅에 사용
console\.(log|warn|info)\s*\(.*(?:error|err|exception|fail)

# console.error도 프로덕션에서는 구조화된 로거로 대체 권장
console\.error\s*\((?!.*structured|.*logger)

# Python - print를 에러 로깅에 사용
print\s*\(.*(?:error|err|exception|traceback)
```

**취약한 코드**:
```javascript
app.use((err, req, res, next) => {
    console.log('에러 발생:', err.message);
    console.error(err.stack);
    res.status(500).send('서버 오류');
});
```

**개선된 코드**:
```javascript
import { logger } from './utils/logger.js';

app.use((err, req, res, next) => {
    logger.error('요청 처리 중 에러 발생', {
        method: req.method,
        path: req.path,
        statusCode: err.statusCode || 500,
        errorMessage: err.message,
        stack: err.stack,
        requestId: req.id,
        userId: req.user?.id,
    });
    res.status(err.statusCode || 500).json({
        type: 'https://api.example.com/errors/internal-server-error',
        title: '서버 내부 오류',
        status: err.statusCode || 500,
        detail: process.env.NODE_ENV === 'production'
            ? '요청을 처리할 수 없습니다'
            : err.message,
        instance: req.path,
    });
});
```

### 1.4 미처리 Promise 거부 (Unhandled Promise Rejections)

비동기 함수에서 `.catch()`를 누락하거나, `async/await`에서 try/catch를 빠뜨린 패턴입니다. Node.js에서는 미처리 Promise 거부가 프로세스 종료로 이어질 수 있습니다.

**탐지 정규식**:
```regex
# .then()만 있고 .catch() 누락
\.then\s*\([^)]*\)\s*(?!\.catch)(?!\.finally);\s*$

# await 키워드가 try 블록 밖에서 사용
(?<!try\s*\{[\s\S]*?)await\s+\w+

# 프로세스 레벨 핸들러 누락 확인 (Node.js)
# 메인 진입점에서 unhandledRejection 핸들러가 없는 경우
process\.on\s*\(\s*['"]unhandledRejection['"]
```

**취약한 코드**:
```javascript
// .catch() 누락
fetchData(url).then(data => {
    processData(data);
});

// await를 try 블록 없이 사용
async function handleRequest(req, res) {
    const user = await getUser(req.params.id);
    const orders = await getOrders(user.id);
    res.json({ user, orders });
}
```

**개선된 코드**:
```javascript
// .catch() 추가
fetchData(url)
    .then(data => processData(data))
    .catch(error => {
        logger.error('데이터 처리 실패', { url, error: error.message });
    });

// try/catch로 감싸기
async function handleRequest(req, res, next) {
    try {
        const user = await getUser(req.params.id);
        const orders = await getOrders(user.id);
        res.json({ user, orders });
    } catch (error) {
        next(error);
    }
}
```

### 1.5 제네릭 에러 타입 캐칭

`Error` 기본 클래스만 캐칭하여 구체적인 에러 유형을 구분하지 못하는 패턴입니다.

**탐지 정규식**:
```regex
# JavaScript - 모든 에러를 동일하게 처리
catch\s*\(\s*\w+\s*\)\s*\{[^}]*\}(?![\s\S]*catch)

# Python - bare except 또는 너무 넓은 except
except\s*:\s*$
except\s+Exception\s*(?:as\s+\w+\s*)?:\s*$

# Java - 최상위 Exception 캐칭
catch\s*\(\s*Exception\s+\w+\s*\)
catch\s*\(\s*Throwable\s+\w+\s*\)
```

**취약한 코드**:
```python
try:
    result = process_payment(order)
except Exception as e:
    print(f"에러 발생: {e}")
    return None
```

**개선된 코드**:
```python
try:
    result = process_payment(order)
except PaymentGatewayError as e:
    logger.error("결제 게이트웨이 오류", extra={"order_id": order.id, "gateway_error": str(e)})
    raise PaymentProcessingError("GATEWAY_ERROR", cause=e)
except InsufficientFundsError as e:
    logger.warning("잔액 부족", extra={"order_id": order.id, "amount": order.amount})
    raise PaymentProcessingError("INSUFFICIENT_FUNDS", cause=e)
except ValidationError as e:
    logger.warning("결제 데이터 유효성 검증 실패", extra={"order_id": order.id, "errors": e.errors})
    raise PaymentProcessingError("VALIDATION_FAILED", cause=e)
except Exception as e:
    logger.critical("예상치 못한 결제 오류", extra={"order_id": order.id}, exc_info=True)
    raise PaymentProcessingError("UNEXPECTED_ERROR", cause=e)
```

## 2. API 에러 응답 일관성 검사

API 엔드포인트의 에러 응답이 일관된 포맷과 적절한 HTTP 상태 코드를 사용하는지 분석합니다.

### 2.1 HTTP 상태 코드 사용 분석

**탐지 정규식**:
```regex
# 모든 에러에 500만 사용하는 패턴
res\.status\s*\(\s*500\s*\)

# 에러 상황에서 200을 반환하는 패턴
catch\s*\(.*\)\s*\{[\s\S]*?res\.status\s*\(\s*200\s*\)

# 부적절한 상태 코드 사용
res\.status\s*\(\s*(?:200|201|204)\s*\).*(?:error|fail|invalid)

# 상태 코드 없이 에러 응답
res\.(json|send)\s*\(\s*\{.*error
```

**올바른 HTTP 상태 코드 매핑**:

| 상태 코드 | 의미 | 사용 상황 |
|-----------|------|----------|
| 400 | Bad Request | 요청 데이터 유효성 검증 실패 |
| 401 | Unauthorized | 인증 필요 또는 인증 실패 |
| 403 | Forbidden | 인가 실패 (권한 없음) |
| 404 | Not Found | 리소스를 찾을 수 없음 |
| 409 | Conflict | 리소스 상태 충돌 (중복 생성 등) |
| 422 | Unprocessable Entity | 요청은 이해했으나 처리 불가 |
| 429 | Too Many Requests | Rate limiting 초과 |
| 500 | Internal Server Error | 서버 내부 오류 (예상치 못한 에러) |
| 502 | Bad Gateway | 외부 서비스 호출 실패 |
| 503 | Service Unavailable | 서비스 일시 중단 |
| 504 | Gateway Timeout | 외부 서비스 타임아웃 |

### 2.2 RFC 7807 (Problem Details) 준수 여부

API 에러 응답이 RFC 7807 표준 포맷을 따르는지 확인합니다.

**표준 에러 응답 포맷**:
```json
{
    "type": "https://api.example.com/errors/validation-error",
    "title": "요청 데이터 유효성 검증 실패",
    "status": 400,
    "detail": "email 필드는 유효한 이메일 주소여야 합니다",
    "instance": "/api/users",
    "errors": [
        {
            "field": "email",
            "message": "유효한 이메일 주소를 입력하세요"
        }
    ]
}
```

**탐지 포인트**:
- 에러 응답에 `type`, `title`, `status`, `detail` 필드가 포함되는지 확인
- 에러 응답 포맷이 엔드포인트마다 다른지 확인
- 스택 트레이스가 프로덕션 응답에 노출되는지 확인

### 2.3 누락된 에러 상태 코드

엣지 케이스에 대한 에러 처리가 빠져 있는 패턴을 탐지합니다.

**검증 항목**:
- 인증이 필요한 엔드포인트에서 401 응답 누락
- 리소스 조회 시 404 응답 누락
- 입력 유효성 검증 후 400 응답 누락
- 외부 서비스 호출 실패 시 502/504 응답 누락
- Rate limiting 적용 시 429 응답 누락

## 3. 에러 바운더리 탐지

프레임워크별 전역 에러 처리 메커니즘이 적절히 설정되어 있는지 확인합니다.

### 3.1 React ErrorBoundary

**탐지 정규식**:
```regex
# ErrorBoundary 클래스 컴포넌트 존재 여부
componentDidCatch\s*\(|getDerivedStateFromError\s*\(

# react-error-boundary 라이브러리 사용 여부
from\s+['"]react-error-boundary['"]|require\s*\(\s*['"]react-error-boundary['"]\s*\)

# ErrorBoundary로 감싸는 패턴
<ErrorBoundary|<ErrorBoundaryProvider
```

**검증 항목**:
- 최상위 App 컴포넌트에 ErrorBoundary 래핑 여부
- 주요 라우트별 독립적인 ErrorBoundary 설정 여부
- ErrorBoundary의 fallback UI 정의 여부
- 에러 보고 서비스 연동 여부 (Sentry, Bugsnag 등)

**권장 구현**:
```jsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
    return (
        <div role="alert">
            <h2>문제가 발생했습니다</h2>
            <pre>{error.message}</pre>
            <button onClick={resetErrorBoundary}>다시 시도</button>
        </div>
    );
}

function App() {
    return (
        <ErrorBoundary
            FallbackComponent={ErrorFallback}
            onError={(error, info) => {
                logger.error('React 렌더링 에러', {
                    error: error.message,
                    componentStack: info.componentStack,
                });
            }}
            onReset={() => { window.location.reload(); }}
        >
            <Router />
        </ErrorBoundary>
    );
}
```

### 3.2 Vue errorHandler

**탐지 정규식**:
```regex
# Vue 전역 에러 핸들러
app\.config\.errorHandler\s*=
Vue\.config\.errorHandler\s*=

# Vue 경고 핸들러
app\.config\.warnHandler\s*=

# 컴포넌트 레벨 에러 캐칭
onErrorCaptured\s*\(|errorCaptured\s*\(
```

**검증 항목**:
- `app.config.errorHandler` 설정 여부
- `onErrorCaptured` 훅 사용 여부
- 에러 보고 서비스 연동 여부

### 3.3 전역 예외 핸들러

**Node.js**:
```regex
# 프로세스 레벨 핸들러
process\.on\s*\(\s*['"]uncaughtException['"]
process\.on\s*\(\s*['"]unhandledRejection['"]

# Express 전역 에러 미들웨어 (4개 인자)
app\.use\s*\(\s*(async\s+)?\(\s*(err|error)\s*,\s*(req|request)\s*,\s*(res|response)\s*,\s*next\s*\)
```

**Python**:
```regex
# sys.excepthook 설정
sys\.excepthook\s*=

# Django 미들웨어 에러 처리
process_exception\s*\(\s*self\s*,\s*request\s*,\s*exception

# Flask 에러 핸들러
@app\.errorhandler\s*\(
app\.register_error_handler\s*\(
```

**Java (Spring)**:
```regex
# @ControllerAdvice 전역 에러 핸들러
@ControllerAdvice|@RestControllerAdvice

# @ExceptionHandler 메서드
@ExceptionHandler\s*\(

# 전역 예외 처리 설정
extends\s+ResponseEntityExceptionHandler
```

### 3.4 프로세스 레벨 에러 핸들러 (Node.js)

**권장 설정**:
```javascript
// 미처리 예외 핸들러
process.on('uncaughtException', (error) => {
    logger.fatal('미처리 예외 발생', {
        error: error.message,
        stack: error.stack,
    });
    // 정리 작업 수행 후 프로세스 종료
    gracefulShutdown().then(() => {
        process.exit(1);
    });
});

// 미처리 Promise 거부 핸들러
process.on('unhandledRejection', (reason, promise) => {
    logger.error('미처리 Promise 거부', {
        reason: reason instanceof Error ? reason.message : String(reason),
        stack: reason instanceof Error ? reason.stack : undefined,
    });
});

// SIGTERM 핸들러 (Graceful Shutdown)
process.on('SIGTERM', () => {
    logger.info('SIGTERM 수신, 정상 종료 시작');
    gracefulShutdown().then(() => {
        process.exit(0);
    });
});
```

## 4. 복원력 패턴 점검

외부 서비스 호출 시 장애에 대비한 복원력 패턴이 적용되어 있는지 분석합니다.

### 4.1 재시도(Retry) 패턴

**탐지 정규식**:
```regex
# HTTP 클라이언트 호출에서 재시도 설정 여부
(fetch|axios|got|node-fetch|superagent|request)\s*\((?!.*retry)

# Python requests에서 재시도 설정 여부
requests\.(get|post|put|delete|patch)\s*\((?!.*retry)(?!.*Retry)

# Java에서 재시도 설정 여부
(RestTemplate|WebClient|HttpClient)\s*\.(?!.*retry|.*Retry)
```

**검증 항목**:
- 외부 API 호출에 재시도 로직 적용 여부
- 지수 백오프(Exponential Backoff) 전략 사용 여부
- 최대 재시도 횟수 설정 여부
- 재시도 대상 에러 코드 지정 여부 (5xx만 재시도 등)

### 4.2 타임아웃(Timeout) 패턴

**탐지 정규식**:
```regex
# fetch에 타임아웃 미설정
fetch\s*\(\s*[^)]+\)(?!.*timeout|.*signal|.*AbortController)

# axios에 타임아웃 미설정
axios\.(get|post|put|delete|patch)\s*\((?!.*timeout)

# Python requests에 타임아웃 미설정
requests\.(get|post|put|delete|patch)\s*\([^)]*\)(?!.*timeout)

# 데이터베이스 쿼리에 타임아웃 미설정
(\.query|\.execute|\.find|\.findOne|\.aggregate)\s*\((?!.*timeout|.*maxTimeMS)
```

**검증 항목**:
- HTTP 요청에 연결 타임아웃 및 응답 타임아웃 설정 여부
- 데이터베이스 쿼리에 타임아웃 설정 여부
- 적절한 타임아웃 값 범위 (너무 길거나 짧지 않은지)

### 4.3 서킷 브레이커(Circuit Breaker) 패턴

**탐지 정규식**:
```regex
# 서킷 브레이커 라이브러리 사용 여부
(opossum|cockatiel|brakes|circuit-?breaker|resilience4j|Polly|Hystrix|pybreaker)

# 서킷 브레이커 패턴 구현 여부
circuitBreaker|CircuitBreaker|circuit_breaker|CIRCUIT_BREAKER
```

**검증 항목**:
- 핵심 외부 서비스 호출에 서킷 브레이커 적용 여부
- Open/Half-Open/Closed 상태 전환 임계값 설정
- 서킷 오픈 시 폴백 전략 정의 여부
- 서킷 상태 모니터링 및 알림 설정

### 4.4 폴백(Fallback) 전략

**탐지 정규식**:
```regex
# 외부 서비스 호출 실패 시 폴백 처리 여부
catch\s*\(.*\)\s*\{[\s\S]*?(?:fallback|default|cache|backup|alternative)

# 캐시 기반 폴백
(getFromCache|cacheGet|redis\.get|cache\.get)[\s\S]*?catch
```

**검증 항목**:
- 외부 서비스 장애 시 캐시된 데이터 반환 전략
- 기본값(Default Value) 반환 전략
- 대체 서비스 호출 전략
- Graceful Degradation 구현 여부

**취약한 코드**:
```javascript
async function getProductRecommendations(userId) {
    const response = await fetch(`${RECOMMENDATION_API}/users/${userId}`);
    return response.json();
}
```

**개선된 코드**:
```javascript
import CircuitBreaker from 'opossum';

const recommendationBreaker = new CircuitBreaker(fetchRecommendations, {
    timeout: 3000,
    errorThresholdPercentage: 50,
    resetTimeout: 30000,
});

recommendationBreaker.fallback((userId) => {
    logger.warn('추천 서비스 폴백 활성화', { userId });
    return cache.get(`recommendations:${userId}`) || DEFAULT_RECOMMENDATIONS;
});

async function getProductRecommendations(userId) {
    return recommendationBreaker.fire(userId);
}

async function fetchRecommendations(userId) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), 3000);

    try {
        const response = await fetch(
            `${RECOMMENDATION_API}/users/${userId}`,
            { signal: controller.signal }
        );

        if (!response.ok) {
            throw new ExternalServiceError('RECOMMENDATION_API_ERROR', response.status);
        }

        const data = await response.json();
        await cache.set(`recommendations:${userId}`, data, { ttl: 300 });
        return data;
    } finally {
        clearTimeout(timeoutId);
    }
}
```

## 5. 심각도 분류 기준

분석 결과를 다음 기준으로 분류합니다:

### Critical (즉시 대응)

- 프로세스 레벨 에러 핸들러(`uncaughtException`, `unhandledRejection`) 미설정
- 에러 발생 시 민감 정보(스택 트레이스, DB 쿼리, 환경 변수)가 클라이언트에 노출
- 에러 처리 누락으로 인한 서비스 전체 중단 가능성

### High (48시간 내 대응)

- 최상위 에러 바운더리(ErrorBoundary, ControllerAdvice 등) 미설정
- 핵심 비즈니스 로직의 빈 catch 블록
- 외부 서비스 호출에 타임아웃 미설정
- API 에러 응답에서 내부 구현 세부사항 노출

### Medium (1주일 내 대응)

- console.log 기반 에러 로깅 (구조화된 로거 미사용)
- API 에러 응답 포맷 불일치
- 제네릭 에러 타입만 캐칭
- 외부 서비스 호출에 재시도 로직 미적용

### Low (계획적 대응)

- 서킷 브레이커 미적용 (트래픽이 높은 서비스의 경우 Medium으로 상향)
- 에러 응답의 RFC 7807 미준수
- 에러 모니터링 대시보드 미설정
- 폴백 전략 미구현

### Info (참고 사항)

- 구조화된 로깅 모범 사례 권장
- 에러 코드 체계 정의 권장
- 에러 처리 문서화 권장
- 헬스체크 엔드포인트 구현 권장

## 에러 처리

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | Git 저장소가 아닌 경우 | `git status` 실패 | 전체 디렉토리 스캔으로 전환 |
| 2 | 지원하지 않는 언어 | 파일 확장자 미매칭 | 일반 패턴(빈 catch 등)만 탐지 |
| 3 | 대규모 코드베이스 | 파일 수 > 10,000 | 변경된 파일만 우선 분석 |
| 4 | 바이너리 파일 포함 | 파일 유형 검사 | 바이너리 파일 제외 |
| 5 | 프레임워크 미식별 | 설정 파일 미발견 | 범용 에러 처리 패턴만 분석 |
| 6 | 패턴 오탐 (False Positive) | 컨텍스트 분석 | [references/analysis-guide.md](references/analysis-guide.md)의 FP 판단 기준 적용 |
| 7 | 모노레포 구조 | 다중 패키지 감지 | 패키지별 독립 분석 수행 |

## 모범 사례 (Best Practices)

1. **커스텀 에러 클래스 정의**: 비즈니스 도메인에 맞는 에러 계층 구조를 설계하세요
2. **구조화된 로깅 도입**: winston, pino, bunyan 등 구조화된 로깅 라이브러리를 사용하세요
3. **에러 코드 체계 수립**: 에러를 고유 코드로 식별할 수 있는 체계를 만드세요 (예: `AUTH_001`, `PAY_002`)
4. **에러 바운더리 계층화**: 애플리케이션 전체, 라우트별, 컴포넌트별 에러 바운더리를 설정하세요
5. **복원력 패턴 적용**: 외부 서비스 호출에 재시도, 타임아웃, 서킷 브레이커를 적용하세요
6. **에러 모니터링 연동**: Sentry, Datadog, New Relic 등과 연동하여 실시간 에러 모니터링을 구축하세요
7. **API 에러 응답 표준화**: RFC 7807 Problem Details를 기반으로 에러 응답을 통일하세요
8. **프로세스 핸들러 설정**: `uncaughtException`, `unhandledRejection`, `SIGTERM` 핸들러를 설정하세요
9. **에러 로깅에 컨텍스트 포함**: 에러 발생 시 요청 ID, 사용자 ID, 파라미터 등 디버깅에 필요한 컨텍스트를 함께 기록하세요
10. **주기적 점검**: 분기별로 에러 처리 코드를 리뷰하고 개선하세요

## 연관 스킬 (Cross-references)

이 스킬은 다음 스킬과 시너지 효과를 제공합니다:

- **[refactor-advisor](../refactor-advisor/SKILL.md)**: 에러 처리 관련 코드 스멜(빈 catch, 과도한 try/catch 중첩)에 대한 리팩토링 제안과 연계
- **[security-scanner](../security-scanner/SKILL.md)**: OWASP A09 (Security Logging and Monitoring Failures) 분석과 연계하여 보안 이벤트 로깅 누락 탐지
- **[test-coverage-analyzer](../test-coverage-analyzer/SKILL.md)**: 에러 시나리오에 대한 테스트 케이스 누락 식별 및 에러 경로 테스트 스켈레톤 생성

## 참조 문서

- [references/analysis-guide.md](references/analysis-guide.md) - 상세 분석 방법론, 언어별 패턴 정규식, false positive 판단 기준, 체크리스트
- [references/templates.md](references/templates.md) - 보고서 출력 형식, 개선 가이드 템플릿, 에러 메시지 표준 템플릿
