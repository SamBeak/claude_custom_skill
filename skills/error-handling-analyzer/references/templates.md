# Error Handling Analyzer 템플릿

이 문서는 에러 처리 분석 보고서, 안티패턴별 개선 가이드, API 에러 응답 표준 포맷, 에러 바운더리 설정 가이드, 복원력 패턴 구현 가이드, 에러 메시지 표준 템플릿을 정의합니다.

---

## 1. 에러 처리 분석 보고서 출력 형식

### 전체 보고서 템플릿

```markdown
# 에러 처리 분석 보고서

> 분석 일시: {SCAN_DATE}
> 분석 범위: {SCAN_SCOPE} (예: git diff --staged, 전체 코드베이스)
> 분석 대상: {FILE_COUNT}개 파일 ({LANGUAGE_SUMMARY})
> 프로젝트: {PROJECT_NAME}
> 프레임워크: {FRAMEWORK_NAME}

---

## 요약

| 심각도 | 발견 건수 |
|--------|----------|
| Critical | {CRITICAL_COUNT} |
| High | {HIGH_COUNT} |
| Medium | {MEDIUM_COUNT} |
| Low | {LOW_COUNT} |
| Info | {INFO_COUNT} |
| **합계** | **{TOTAL_COUNT}** |

### 카테고리별 현황

| 카테고리 | 발견 건수 | 주요 이슈 |
|---------|----------|----------|
| 안티패턴 | {ANTIPATTERN_COUNT} | {TOP_ANTIPATTERN_ISSUE} |
| API 에러 응답 | {API_ERROR_COUNT} | {TOP_API_ERROR_ISSUE} |
| 에러 바운더리 | {BOUNDARY_COUNT} | {TOP_BOUNDARY_ISSUE} |
| 복원력 패턴 | {RESILIENCE_COUNT} | {TOP_RESILIENCE_ISSUE} |
| 로깅 품질 | {LOGGING_COUNT} | {TOP_LOGGING_ISSUE} |

---

## 상세 결과

### Critical 이슈

#### [{ISSUE_ID}] {ISSUE_TITLE}

- **심각도**: Critical
- **카테고리**: {CATEGORY} (예: 안티패턴 / API 에러 응답 / 에러 바운더리 / 복원력)
- **파일**: `{FILE_PATH}:{LINE_NUMBER}`
- **패턴**: {PATTERN_DESCRIPTION}

**현재 코드**:
```{LANGUAGE}
{CURRENT_CODE}
```

**문제점**:
{PROBLEM_DESCRIPTION}

**개선된 코드**:
```{LANGUAGE}
{IMPROVED_CODE}
```

**참조**:
- {REFERENCE_URL}

---

### High 이슈

(동일 형식 반복)

### Medium 이슈

(동일 형식 반복)

### Low 이슈

(동일 형식 반복)

### Info

(동일 형식 반복)

---

## 에러 바운더리 점검 결과

| 항목 | 상태 | 권장 사항 |
|------|------|----------|
| 프로세스 핸들러 (uncaughtException) | {STATUS} | {RECOMMENDATION} |
| 프로세스 핸들러 (unhandledRejection) | {STATUS} | {RECOMMENDATION} |
| Graceful Shutdown (SIGTERM) | {STATUS} | {RECOMMENDATION} |
| 전역 에러 미들웨어 | {STATUS} | {RECOMMENDATION} |
| React ErrorBoundary | {STATUS} | {RECOMMENDATION} |
| 에러 보고 서비스 연동 | {STATUS} | {RECOMMENDATION} |

---

## 복원력 패턴 점검 결과

| 외부 서비스 | 타임아웃 | 재시도 | 서킷 브레이커 | 폴백 |
|------------|---------|-------|-------------|------|
| {SERVICE_NAME} | {TIMEOUT_STATUS} | {RETRY_STATUS} | {CB_STATUS} | {FALLBACK_STATUS} |

---

## 권장 조치

### 즉시 대응 (Critical)
1. {ACTION_ITEM_1}
2. {ACTION_ITEM_2}

### 단기 대응 (High - 48시간 내)
1. {ACTION_ITEM_3}
2. {ACTION_ITEM_4}

### 중기 대응 (Medium - 1주일 내)
1. {ACTION_ITEM_5}

### 장기 대응 (Low - 계획적)
1. {ACTION_ITEM_6}
```

### 간략 보고서 템플릿 (커밋 전 검사)

```markdown
## 에러 처리 분석 결과 (커밋 전 검사)

> 분석 일시: {SCAN_DATE}
> 변경 파일: {CHANGED_FILES_COUNT}개

### 결과 요약
- Critical: {CRITICAL_COUNT}건
- High: {HIGH_COUNT}건
- Medium: {MEDIUM_COUNT}건
- Low: {LOW_COUNT}건

{IF_CRITICAL_OR_HIGH}
### 주요 발견 사항

다음 Critical/High 이슈가 발견되었습니다:

{BLOCKING_ISSUES_LIST}

각 항목의 개선 방법은 아래를 참조하세요.
{END_IF}

{IF_CLEAN}
에러 처리 관련 이슈가 발견되지 않았습니다.
{END_IF}
```

---

## 2. 안티패턴별 개선 가이드 템플릿

### 빈 catch 블록 개선 가이드

```markdown
## 빈 catch 블록 개선 가이드

### 문제 설명
에러를 캐치한 후 아무 처리도 하지 않아 에러가 조용히 무시됩니다.
이로 인해 버그 원인을 파악할 수 없고, 장애 대응이 지연됩니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`
- 코드:
  ```{LANGUAGE}
  {VULNERABLE_CODE}
  ```

### 개선 방법

#### JavaScript/TypeScript
**개선 전**:
```javascript
try {
    await saveUserData(user);
} catch (error) {
}
```

**개선 후**:
```javascript
try {
    await saveUserData(user);
} catch (error) {
    logger.error('사용자 데이터 저장 실패', {
        userId: user.id,
        error: error.message,
        stack: error.stack,
    });
    throw new DataPersistenceError('SAVE_USER_FAILED', { cause: error });
}
```

#### Python
**개선 전**:
```python
try:
    save_user_data(user)
except Exception:
    pass
```

**개선 후**:
```python
try:
    save_user_data(user)
except DatabaseError as e:
    logger.error("사용자 데이터 저장 실패", extra={"user_id": user.id, "error": str(e)})
    raise DataPersistenceError("SAVE_USER_FAILED") from e
```

#### Java
**개선 전**:
```java
try {
    userRepository.save(user);
} catch (Exception e) {
}
```

**개선 후**:
```java
try {
    userRepository.save(user);
} catch (DataAccessException e) {
    log.error("사용자 데이터 저장 실패: userId={}", user.getId(), e);
    throw new DataPersistenceException("SAVE_USER_FAILED", e);
}
```

### 예외: 의도적 무시가 허용되는 경우

에러를 무시하는 것이 정당한 경우에는 반드시 사유를 주석으로 명시하세요:

```javascript
try {
    fs.unlinkSync(tempFilePath);
} catch (error) {
    // 임시 파일이 이미 삭제된 경우 무시 (정상 시나리오)
}
```
```

### 미처리 Promise 거부 개선 가이드

```markdown
## 미처리 Promise 거부 개선 가이드

### 문제 설명
비동기 작업에서 .catch()를 누락하거나 try/catch로 감싸지 않아
Promise 거부가 처리되지 않습니다. Node.js에서는 이로 인해 프로세스가
비정상 종료될 수 있습니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`

### 개선 방법

#### .then()에 .catch() 추가
**개선 전**:
```javascript
fetchData(url).then(data => {
    processData(data);
});
```

**개선 후**:
```javascript
fetchData(url)
    .then(data => processData(data))
    .catch(error => {
        logger.error('데이터 처리 실패', { url, error: error.message });
    });
```

#### async/await에 try/catch 추가
**개선 전**:
```javascript
async function handleRequest(req, res) {
    const data = await fetchExternalAPI(req.params.id);
    res.json(data);
}
```

**개선 후**:
```javascript
async function handleRequest(req, res, next) {
    try {
        const data = await fetchExternalAPI(req.params.id);
        res.json(data);
    } catch (error) {
        next(error);
    }
}
```

#### Express asyncHandler 유틸리티
에러 처리 보일러플레이트를 줄이기 위해 asyncHandler를 사용합니다:

```javascript
function asyncHandler(fn) {
    return function (req, res, next) {
        return Promise.resolve(fn(req, res, next)).catch(next);
    };
}

// 사용
app.get('/api/users/:id', asyncHandler(async (req, res) => {
    const user = await getUser(req.params.id);
    res.json(user);
}));
```

#### 프로세스 레벨 안전장치
```javascript
process.on('unhandledRejection', (reason, promise) => {
    logger.error('미처리 Promise 거부', {
        reason: reason instanceof Error ? reason.message : String(reason),
        stack: reason instanceof Error ? reason.stack : undefined,
    });
});
```
```

### 비구조화 로깅 개선 가이드

```markdown
## 비구조화 로깅 개선 가이드

### 문제 설명
console.log/console.error를 사용한 에러 로깅은 구조화된 검색, 필터링,
모니터링이 불가능합니다. 프로덕션 환경에서는 구조화된 로깅 라이브러리를
사용해야 합니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`

### 개선 방법

#### winston 설정 (Node.js)
```javascript
import winston from 'winston';

const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.errors({ stack: true }),
        winston.format.json()
    ),
    defaultMeta: { service: '{SERVICE_NAME}' },
    transports: [
        new winston.transports.Console(),
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
    ],
});

export { logger };
```

#### pino 설정 (Node.js - 고성능)
```javascript
import pino from 'pino';

const logger = pino({
    level: process.env.LOG_LEVEL || 'info',
    transport: process.env.NODE_ENV !== 'production'
        ? { target: 'pino-pretty' }
        : undefined,
    serializers: {
        error: pino.stdSerializers.err,
        req: pino.stdSerializers.req,
        res: pino.stdSerializers.res,
    },
});

export { logger };
```

#### Python logging 설정
```python
import logging
import json

class JsonFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "message": record.getMessage(),
            "module": record.module,
            "function": record.funcName,
            "line": record.lineno,
        }
        if record.exc_info:
            log_data["exception"] = self.formatException(record.exc_info)
        if hasattr(record, "extra_data"):
            log_data.update(record.extra_data)
        return json.dumps(log_data, ensure_ascii=False)

logger = logging.getLogger("{SERVICE_NAME}")
handler = logging.StreamHandler()
handler.setFormatter(JsonFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)
```

#### 에러 로깅에 컨텍스트 포함
**개선 전**:
```javascript
console.error(error);
```

**개선 후**:
```javascript
logger.error('주문 처리 실패', {
    orderId: order.id,
    userId: user.id,
    amount: order.amount,
    error: error.message,
    stack: error.stack,
    requestId: req.id,
    timestamp: new Date().toISOString(),
});
```
```

---

## 3. API 에러 응답 표준 포맷 템플릿

### RFC 7807 Problem Details 기본 구조

```json
{
    "type": "https://api.{DOMAIN}/errors/{ERROR_TYPE}",
    "title": "{ERROR_TITLE}",
    "status": {HTTP_STATUS_CODE},
    "detail": "{ERROR_DETAIL}",
    "instance": "{REQUEST_PATH}",
    "timestamp": "{ISO_TIMESTAMP}",
    "traceId": "{TRACE_ID}"
}
```

### 유효성 검증 에러

```json
{
    "type": "https://api.{DOMAIN}/errors/validation-error",
    "title": "요청 데이터 유효성 검증 실패",
    "status": 400,
    "detail": "요청에 유효하지 않은 필드가 포함되어 있습니다",
    "instance": "/api/users",
    "timestamp": "2026-02-23T10:30:00Z",
    "traceId": "abc-123-def",
    "errors": [
        {
            "field": "email",
            "value": "invalid-email",
            "message": "유효한 이메일 주소를 입력하세요"
        },
        {
            "field": "password",
            "value": null,
            "message": "비밀번호는 8자 이상이어야 합니다"
        }
    ]
}
```

### 인증 에러

```json
{
    "type": "https://api.{DOMAIN}/errors/authentication-error",
    "title": "인증 실패",
    "status": 401,
    "detail": "유효하지 않은 인증 토큰입니다. 다시 로그인해 주세요.",
    "instance": "/api/orders",
    "timestamp": "2026-02-23T10:30:00Z",
    "traceId": "abc-123-def"
}
```

### 리소스 미발견 에러

```json
{
    "type": "https://api.{DOMAIN}/errors/resource-not-found",
    "title": "리소스를 찾을 수 없음",
    "status": 404,
    "detail": "요청하신 사용자(ID: {RESOURCE_ID})를 찾을 수 없습니다",
    "instance": "/api/users/{RESOURCE_ID}",
    "timestamp": "2026-02-23T10:30:00Z",
    "traceId": "abc-123-def"
}
```

### 내부 서버 에러

```json
{
    "type": "https://api.{DOMAIN}/errors/internal-server-error",
    "title": "서버 내부 오류",
    "status": 500,
    "detail": "요청을 처리하는 중 예상치 못한 오류가 발생했습니다. 잠시 후 다시 시도해 주세요.",
    "instance": "/api/orders",
    "timestamp": "2026-02-23T10:30:00Z",
    "traceId": "abc-123-def"
}
```

### 외부 서비스 장애 에러

```json
{
    "type": "https://api.{DOMAIN}/errors/service-unavailable",
    "title": "외부 서비스 일시 장애",
    "status": 502,
    "detail": "결제 서비스가 일시적으로 응답하지 않습니다. 잠시 후 다시 시도해 주세요.",
    "instance": "/api/payments",
    "timestamp": "2026-02-23T10:30:00Z",
    "traceId": "abc-123-def",
    "retryAfter": 30
}
```

### API 에러 응답 미들웨어 구현 (Express.js)

```javascript
class AppError extends Error {
    constructor(type, title, status, detail, extras = {}) {
        super(detail);
        this.type = type;
        this.title = title;
        this.status = status;
        this.detail = detail;
        this.extras = extras;
    }
}

function errorResponseMiddleware(err, req, res, next) {
    const status = err.status || 500;
    const traceId = req.id || crypto.randomUUID();

    const errorResponse = {
        type: err.type || `https://api.example.com/errors/${status === 500 ? 'internal-server-error' : 'unknown-error'}`,
        title: err.title || '서버 오류',
        status,
        detail: status === 500 && process.env.NODE_ENV === 'production'
            ? '요청을 처리하는 중 오류가 발생했습니다'
            : err.detail || err.message,
        instance: req.originalUrl,
        timestamp: new Date().toISOString(),
        traceId,
        ...err.extras,
    };

    logger.error('API 에러 응답', {
        ...errorResponse,
        stack: err.stack,
        method: req.method,
        userId: req.user?.id,
    });

    res.status(status).json(errorResponse);
}
```

---

## 4. 에러 바운더리 설정 가이드 템플릿

### React ErrorBoundary 설정

```jsx
// components/ErrorBoundary.jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
    constructor(props) {
        super(props);
        this.state = { hasError: false, error: null };
    }

    static getDerivedStateFromError(error) {
        return { hasError: true, error };
    }

    componentDidCatch(error, errorInfo) {
        // 에러 보고 서비스로 전송
        logger.error('React 렌더링 에러', {
            error: error.message,
            stack: error.stack,
            componentStack: errorInfo.componentStack,
        });
    }

    render() {
        if (this.state.hasError) {
            return this.props.fallback || (
                <div role="alert">
                    <h2>문제가 발생했습니다</h2>
                    <p>페이지를 새로고침하거나 잠시 후 다시 시도해 주세요.</p>
                    <button onClick={() => this.setState({ hasError: false })}>
                        다시 시도
                    </button>
                </div>
            );
        }
        return this.props.children;
    }
}

export default ErrorBoundary;
```

### Node.js 프로세스 핸들러 설정

```javascript
// bootstrap/errorHandlers.js
import { logger } from '../utils/logger.js';

function setupProcessHandlers(server) {
    // 미처리 예외
    process.on('uncaughtException', (error) => {
        logger.fatal('미처리 예외 발생', {
            error: error.message,
            stack: error.stack,
        });
        gracefulShutdown(server, 'uncaughtException');
    });

    // 미처리 Promise 거부
    process.on('unhandledRejection', (reason, promise) => {
        logger.error('미처리 Promise 거부', {
            reason: reason instanceof Error ? reason.message : String(reason),
            stack: reason instanceof Error ? reason.stack : undefined,
        });
    });

    // Graceful Shutdown
    process.on('SIGTERM', () => {
        logger.info('SIGTERM 수신');
        gracefulShutdown(server, 'SIGTERM');
    });

    process.on('SIGINT', () => {
        logger.info('SIGINT 수신');
        gracefulShutdown(server, 'SIGINT');
    });
}

async function gracefulShutdown(server, signal) {
    logger.info(`정상 종료 시작 (${signal})`);

    // 새로운 요청 수신 중단
    server.close(() => {
        logger.info('HTTP 서버 종료 완료');
    });

    // 진행 중인 작업 완료 대기 (최대 30초)
    const timeout = setTimeout(() => {
        logger.warn('정상 종료 타임아웃, 강제 종료');
        process.exit(1);
    }, 30000);

    try {
        // 데이터베이스 연결 종료
        await db.disconnect();
        // 메시지 큐 연결 종료
        await messageQueue.disconnect();

        clearTimeout(timeout);
        logger.info('정상 종료 완료');
        process.exit(0);
    } catch (error) {
        logger.error('정상 종료 중 에러', { error: error.message });
        clearTimeout(timeout);
        process.exit(1);
    }
}

export { setupProcessHandlers };
```

### Spring @ControllerAdvice 설정

```java
// exception/GlobalExceptionHandler.java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    private static final Logger log = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ProblemDetail> handleResourceNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setType(URI.create("https://api.example.com/errors/resource-not-found"));
        problem.setTitle("리소스를 찾을 수 없음");
        problem.setInstance(URI.create(request.getRequestURI()));

        log.warn("리소스 미발견: {}", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(problem);
    }

    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ProblemDetail> handleValidation(
            ValidationException ex, HttpServletRequest request) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.BAD_REQUEST, ex.getMessage());
        problem.setType(URI.create("https://api.example.com/errors/validation-error"));
        problem.setTitle("유효성 검증 실패");
        problem.setProperty("errors", ex.getErrors());

        log.warn("유효성 검증 실패: {}", ex.getErrors());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(problem);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ProblemDetail> handleGenericException(
            Exception ex, HttpServletRequest request) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.INTERNAL_SERVER_ERROR,
            "요청을 처리하는 중 오류가 발생했습니다");
        problem.setType(URI.create("https://api.example.com/errors/internal-server-error"));
        problem.setTitle("서버 내부 오류");

        log.error("예상치 못한 오류: ", ex);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(problem);
    }
}
```

---

## 5. 복원력 패턴 구현 가이드 템플릿

### 재시도 패턴 (Exponential Backoff)

```javascript
// utils/retry.js
async function withRetry(fn, options = {}) {
    const {
        maxRetries = 3,
        baseDelay = 1000,
        maxDelay = 30000,
        retryableErrors = [502, 503, 504],
        onRetry = () => {},
    } = options;

    for (let attempt = 0; attempt <= maxRetries; attempt++) {
        try {
            return await fn();
        } catch (error) {
            const isLastAttempt = attempt === maxRetries;
            const isRetryable = retryableErrors.includes(error.status)
                || error.code === 'ECONNRESET'
                || error.code === 'ETIMEDOUT';

            if (isLastAttempt || !isRetryable) {
                throw error;
            }

            const delay = Math.min(
                baseDelay * Math.pow(2, attempt) + Math.random() * 1000,
                maxDelay
            );

            onRetry({ attempt: attempt + 1, delay, error });
            logger.warn('재시도 예정', {
                attempt: attempt + 1,
                maxRetries,
                delay,
                error: error.message,
            });

            await new Promise(resolve => setTimeout(resolve, delay));
        }
    }
}

// 사용
const data = await withRetry(
    () => fetch(`${API_URL}/data`).then(res => {
        if (!res.ok) {
            const error = new Error(`HTTP ${res.status}`);
            error.status = res.status;
            throw error;
        }
        return res.json();
    }),
    { maxRetries: 3, baseDelay: 1000 }
);
```

### 타임아웃 패턴

```javascript
// utils/timeout.js
async function withTimeout(fn, timeoutMs, errorMessage) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeoutMs);

    try {
        const result = await fn(controller.signal);
        return result;
    } catch (error) {
        if (error.name === 'AbortError') {
            throw new TimeoutError(
                errorMessage || `작업이 ${timeoutMs}ms 내에 완료되지 않았습니다`
            );
        }
        throw error;
    } finally {
        clearTimeout(timeoutId);
    }
}

// 사용
const data = await withTimeout(
    (signal) => fetch(`${API_URL}/data`, { signal }).then(r => r.json()),
    5000,
    '외부 API 응답 타임아웃'
);
```

### 서킷 브레이커 패턴 (opossum)

```javascript
// services/circuitBreaker.js
import CircuitBreaker from 'opossum';
import { logger } from '../utils/logger.js';

function createCircuitBreaker(fn, options = {}) {
    const breaker = new CircuitBreaker(fn, {
        timeout: options.timeout || 5000,
        errorThresholdPercentage: options.errorThreshold || 50,
        resetTimeout: options.resetTimeout || 30000,
        volumeThreshold: options.volumeThreshold || 10,
        ...options,
    });

    breaker.on('open', () => {
        logger.warn('서킷 브레이커 오픈', { name: options.name });
    });

    breaker.on('halfOpen', () => {
        logger.info('서킷 브레이커 Half-Open', { name: options.name });
    });

    breaker.on('close', () => {
        logger.info('서킷 브레이커 닫힘', { name: options.name });
    });

    breaker.on('fallback', () => {
        logger.warn('서킷 브레이커 폴백 실행', { name: options.name });
    });

    if (options.fallback) {
        breaker.fallback(options.fallback);
    }

    return breaker;
}

// 사용
const paymentBreaker = createCircuitBreaker(
    (orderId) => paymentAPI.processPayment(orderId),
    {
        name: 'payment-service',
        timeout: 10000,
        errorThreshold: 30,
        fallback: (orderId) => {
            logger.warn('결제 서비스 폴백: 대기열에 추가', { orderId });
            return paymentQueue.enqueue(orderId);
        },
    }
);
```

---

## 6. 에러 메시지 표준 템플릿

### 에러 클래스 계층 구조 (JavaScript/TypeScript)

```javascript
// errors/AppError.js
class AppError extends Error {
    constructor(code, message, statusCode = 500, extras = {}) {
        super(message);
        this.name = this.constructor.name;
        this.code = code;
        this.statusCode = statusCode;
        this.extras = extras;
        this.timestamp = new Date().toISOString();
    }

    toJSON() {
        return {
            type: `https://api.example.com/errors/${this.code.toLowerCase().replace(/_/g, '-')}`,
            title: this.message,
            status: this.statusCode,
            code: this.code,
            ...this.extras,
        };
    }
}

class ValidationError extends AppError {
    constructor(errors) {
        super('VALIDATION_FAILED', '요청 데이터 유효성 검증 실패', 400, { errors });
    }
}

class AuthenticationError extends AppError {
    constructor(detail = '인증이 필요합니다') {
        super('AUTHENTICATION_REQUIRED', detail, 401);
    }
}

class AuthorizationError extends AppError {
    constructor(detail = '접근 권한이 없습니다') {
        super('FORBIDDEN', detail, 403);
    }
}

class NotFoundError extends AppError {
    constructor(resource, id) {
        super('RESOURCE_NOT_FOUND', `${resource}(ID: ${id})를 찾을 수 없습니다`, 404);
    }
}

class ConflictError extends AppError {
    constructor(detail) {
        super('RESOURCE_CONFLICT', detail, 409);
    }
}

class ExternalServiceError extends AppError {
    constructor(service, detail) {
        super('EXTERNAL_SERVICE_ERROR', `${service} 서비스 호출 실패: ${detail}`, 502);
    }
}

class TimeoutError extends AppError {
    constructor(detail = '요청 처리 시간이 초과되었습니다') {
        super('TIMEOUT', detail, 504);
    }
}

export {
    AppError,
    ValidationError,
    AuthenticationError,
    AuthorizationError,
    NotFoundError,
    ConflictError,
    ExternalServiceError,
    TimeoutError,
};
```

### 에러 코드 체계 표준

| 접두사 | 도메인 | 예시 코드 | 설명 |
|--------|-------|----------|------|
| `AUTH` | 인증/인가 | `AUTH_001` | 인증 토큰 만료 |
| `AUTH` | 인증/인가 | `AUTH_002` | 권한 부족 |
| `VAL` | 유효성 검증 | `VAL_001` | 필수 필드 누락 |
| `VAL` | 유효성 검증 | `VAL_002` | 잘못된 형식 |
| `USR` | 사용자 | `USR_001` | 사용자 미발견 |
| `USR` | 사용자 | `USR_002` | 중복 이메일 |
| `PAY` | 결제 | `PAY_001` | 결제 게이트웨이 오류 |
| `PAY` | 결제 | `PAY_002` | 잔액 부족 |
| `ORD` | 주문 | `ORD_001` | 주문 미발견 |
| `ORD` | 주문 | `ORD_002` | 주문 상태 변경 불가 |
| `EXT` | 외부 서비스 | `EXT_001` | 외부 API 타임아웃 |
| `EXT` | 외부 서비스 | `EXT_002` | 외부 API 응답 오류 |
| `SYS` | 시스템 | `SYS_001` | 데이터베이스 연결 실패 |
| `SYS` | 시스템 | `SYS_002` | 캐시 서버 연결 실패 |

### 에러 메시지 작성 가이드

에러 메시지는 다음 원칙을 따릅니다:

1. **사용자 향 메시지**: 기술 용어를 피하고 명확한 안내를 제공합니다
2. **개발자 향 메시지**: 디버깅에 필요한 구체적인 정보를 포함합니다
3. **로그 메시지**: 검색과 필터링이 가능한 구조화된 형태로 작성합니다

**사용자 향 메시지 예시**:

| 에러 유형 | 메시지 |
|----------|--------|
| 유효성 검증 | "입력하신 이메일 주소의 형식이 올바르지 않습니다" |
| 인증 필요 | "로그인이 필요한 서비스입니다. 로그인 후 다시 시도해 주세요" |
| 권한 부족 | "이 작업을 수행할 권한이 없습니다" |
| 리소스 미발견 | "요청하신 정보를 찾을 수 없습니다" |
| 서버 오류 | "일시적인 오류가 발생했습니다. 잠시 후 다시 시도해 주세요" |
| 외부 서비스 장애 | "서비스가 일시적으로 불안정합니다. 잠시 후 다시 시도해 주세요" |
| Rate Limiting | "요청이 너무 많습니다. {RETRY_AFTER}초 후에 다시 시도해 주세요" |

### 로그 메시지 표준 형식

```
[{LEVEL}] {TIMESTAMP} [{SERVICE}] [{TRACE_ID}] {MESSAGE} | {CONTEXT_JSON}
```

**예시**:
```
[ERROR] 2026-02-23T10:30:00Z [order-service] [abc-123] 주문 처리 실패 | {"orderId":"ORD-456","userId":"USR-789","error":"INSUFFICIENT_FUNDS","amount":50000}
[WARN]  2026-02-23T10:30:01Z [payment-service] [abc-123] 결제 재시도 | {"attempt":2,"maxRetries":3,"delay":2000,"gatewayError":"TIMEOUT"}
[INFO]  2026-02-23T10:30:03Z [payment-service] [abc-123] 결제 재시도 성공 | {"attempt":3,"orderId":"ORD-456"}
```
