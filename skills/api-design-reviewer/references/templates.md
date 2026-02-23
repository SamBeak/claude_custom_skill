# API Design Reviewer 템플릿

이 문서는 API 설계 리뷰 보고서 출력 형식, 개선 가이드 템플릿, OpenAPI 보일러플레이트, 일관성 점수 카드 형식을 정의합니다.

---

## 1. API 설계 리뷰 보고서 출력 형식

### 전체 보고서 템플릿

```markdown
# API 설계 리뷰 보고서

> 리뷰 일시: {REVIEW_DATE}
> 리뷰 범위: {REVIEW_SCOPE}  (예: 전체 API, 변경된 엔드포인트)
> 분석 대상: {ENDPOINT_COUNT}개 엔드포인트 ({FRAMEWORK})
> 프로젝트: {PROJECT_NAME}

---

## 일관성 점수

| 영역 | 점수 | 등급 |
|------|------|------|
| 리소스 네이밍 | {NAMING_SCORE}/100 | {NAMING_GRADE} |
| HTTP 메서드 | {METHOD_SCORE}/100 | {METHOD_GRADE} |
| 필드 네이밍 | {FIELD_SCORE}/100 | {FIELD_GRADE} |
| 에러 응답 | {ERROR_SCORE}/100 | {ERROR_GRADE} |
| 페이지네이션 | {PAGINATION_SCORE}/100 | {PAGINATION_GRADE} |
| 날짜/시간 | {DATE_SCORE}/100 | {DATE_GRADE} |
| 버전 관리 | {VERSION_SCORE}/100 | {VERSION_GRADE} |
| 추가 항목 | {EXTRA_SCORE}/100 | {EXTRA_GRADE} |
| **종합** | **{TOTAL_SCORE}/100** | **{TOTAL_GRADE}** |

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

### 주요 발견 사항

{TOP_FINDINGS_SUMMARY}

---

## 상세 결과

### Critical 이슈

#### [{ISSUE_ID}] {ISSUE_TITLE}

- **심각도**: Critical
- **영역**: {REVIEW_AREA} (예: REST 규약, 응답 구조, 페이지네이션)
- **엔드포인트**: `{METHOD} {PATH}`
- **파일**: `{FILE_PATH}:{LINE_NUMBER}`

**현재 상태**:
```{LANGUAGE}
{CURRENT_CODE}
```

**문제 설명**:
{ISSUE_DESCRIPTION}

**개선 가이드**:
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

## 엔드포인트 목록

| # | 메서드 | 경로 | 설명 | 인증 | 페이지네이션 | 상태 |
|---|--------|------|------|------|-------------|------|
| 1 | {METHOD} | {PATH} | {DESCRIPTION} | {AUTH} | {PAGINATION} | {STATUS} |

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

### 간략 보고서 템플릿 (변경 사항 리뷰)

```markdown
## API 설계 리뷰 결과 (변경 사항)

> 리뷰 일시: {REVIEW_DATE}
> 변경 엔드포인트: {CHANGED_ENDPOINTS_COUNT}개

### 결과 요약
- Critical: {CRITICAL_COUNT}건
- High: {HIGH_COUNT}건
- Medium: {MEDIUM_COUNT}건
- Low: {LOW_COUNT}건

{IF_CRITICAL_OR_HIGH}
### 주요 이슈

다음 항목을 우선적으로 수정하는 것을 권장합니다:

{BLOCKING_ISSUES_LIST}

각 항목의 개선 방법은 아래를 참조하세요.
{END_IF}

{IF_CLEAN}
API 설계 이슈가 발견되지 않았습니다. 기존 설계 규칙을 잘 준수하고 있습니다.
{END_IF}
```

---

## 2. 개선 가이드 템플릿

### 리소스 네이밍 개선 가이드

```markdown
## 리소스 네이밍 개선 가이드

### 문제 설명
URL 경로에 동사가 사용되었거나, 리소스 이름이 단수형입니다.
REST API에서 리소스는 명사(복수형)로 표현하고, 동작은 HTTP 메서드로 구분합니다.

### 발견 위치
- 엔드포인트: `{METHOD} {CURRENT_PATH}`
- 파일: `{FILE_PATH}:{LINE_NUMBER}`

### 개선 방법

| 현재 | 개선 | 이유 |
|------|------|------|
| `GET /getUsers` | `GET /users` | 동사 제거, HTTP 메서드가 조회를 의미 |
| `POST /createUser` | `POST /users` | 동사 제거, POST가 생성을 의미 |
| `PUT /updateUser/:id` | `PUT /users/:id` | 동사 제거, PUT이 수정을 의미 |
| `DELETE /deleteUser/:id` | `DELETE /users/:id` | 동사 제거, DELETE가 삭제를 의미 |
| `GET /user/:id` | `GET /users/:id` | 단수형을 복수형으로 변경 |
| `GET /user_list` | `GET /users` | 언더스코어 제거, 복수형 사용 |
| `GET /getUserPosts/:id` | `GET /users/:id/posts` | 계층 구조로 표현 |

### 참조
- [REST API 디자인 규칙](https://restfulapi.net/resource-naming/)
```

### 에러 응답 통일 가이드

```markdown
## 에러 응답 통일 가이드

### 문제 설명
프로젝트 내 엔드포인트마다 다른 에러 응답 형식을 사용하고 있습니다.
클라이언트가 일관적으로 에러를 처리하려면 통일된 형식이 필요합니다.

### 발견 위치
- 파일: {FILE_PATHS}

### 현재 사용 중인 에러 형식

```json
// 형식 1
{ "error": "메시지" }

// 형식 2
{ "message": "메시지", "code": 400 }

// 형식 3
{ "err": "메시지", "status": "fail" }
```

### 권장 에러 형식 (RFC 7807 기반)

```json
{
    "type": "https://api.example.com/errors/validation-error",
    "title": "Validation Error",
    "status": 400,
    "detail": "요청 본문의 'email' 필드가 유효한 이메일 형식이 아닙니다.",
    "instance": "/users",
    "errors": [
        {
            "field": "email",
            "message": "유효한 이메일 형식이 아닙니다",
            "code": "INVALID_FORMAT"
        }
    ]
}
```

### 구현 예시 (Express.js)

```javascript
// 공통 에러 응답 유틸리티
function createErrorResponse(type, title, status, detail, instance, errors) {
    return {
        type: `https://api.example.com/errors/${type}`,
        title,
        status,
        detail,
        instance,
        ...(errors && { errors })
    };
}

// 전역 에러 핸들러
app.use((err, req, res, next) => {
    if (err.name === 'ValidationError') {
        return res.status(400).json(createErrorResponse(
            'validation-error',
            'Validation Error',
            400,
            err.message,
            req.originalUrl,
            err.errors
        ));
    }

    if (err.name === 'NotFoundError') {
        return res.status(404).json(createErrorResponse(
            'not-found',
            'Resource Not Found',
            404,
            err.message,
            req.originalUrl
        ));
    }

    // 기본 서버 오류
    console.error(err.stack);
    return res.status(500).json(createErrorResponse(
        'internal-error',
        'Internal Server Error',
        500,
        '서버 내부 오류가 발생했습니다.',
        req.originalUrl
    ));
});
```

### 참조
- [RFC 7807 - Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc7807)
```

### 페이지네이션 구현 가이드

```markdown
## 페이지네이션 구현 가이드

### 문제 설명
목록 엔드포인트에 페이지네이션이 적용되지 않았거나,
프로젝트 내에서 여러 페이지네이션 방식이 혼용되고 있습니다.

### 발견 위치
- 엔드포인트: {ENDPOINTS}
- 파일: {FILE_PATHS}

### 오프셋 기반 페이지네이션 구현

```javascript
// 요청
// GET /users?page=2&limit=20

app.get('/users', async (req, res) => {
    const page = Math.max(1, parseInt(req.query.page) || 1);
    const limit = Math.min(100, Math.max(1, parseInt(req.query.limit) || 20));
    const offset = (page - 1) * limit;

    const [users, total] = await Promise.all([
        User.find().skip(offset).limit(limit),
        User.countDocuments()
    ]);

    res.json({
        data: users,
        meta: {
            total,
            page,
            limit,
            totalPages: Math.ceil(total / limit),
            hasNext: page * limit < total,
            hasPrev: page > 1
        }
    });
});
```

### 커서 기반 페이지네이션 구현

```javascript
// 요청
// GET /users?cursor=abc123&limit=20

app.get('/users', async (req, res) => {
    const limit = Math.min(100, Math.max(1, parseInt(req.query.limit) || 20));
    const cursor = req.query.cursor;

    const query = cursor
        ? { _id: { $gt: decodeCursor(cursor) } }
        : {};

    const users = await User.find(query).limit(limit + 1);

    const hasNext = users.length > limit;
    if (hasNext) users.pop();

    res.json({
        data: users,
        meta: {
            limit,
            hasNext,
            nextCursor: hasNext ? encodeCursor(users[users.length - 1]._id) : null
        }
    });
});

function encodeCursor(id) {
    return Buffer.from(id.toString()).toString('base64');
}

function decodeCursor(cursor) {
    return Buffer.from(cursor, 'base64').toString('utf8');
}
```

### 참조
- [REST API 페이지네이션 설계](https://www.moesif.com/blog/technical/api-design/REST-API-Design-Filtering-Sorting-and-Pagination/)
```

---

## 3. OpenAPI 보일러플레이트

### 기본 OpenAPI 3.0 템플릿

```yaml
openapi: 3.0.3
info:
  title: {PROJECT_NAME} API
  description: {PROJECT_DESCRIPTION}
  version: 1.0.0
  contact:
    name: API Support
    email: support@example.com

servers:
  - url: https://api.example.com/v1
    description: 프로덕션 서버
  - url: https://staging-api.example.com/v1
    description: 스테이징 서버

tags:
  - name: users
    description: 사용자 관리

paths:
  /users:
    get:
      tags: [users]
      summary: 사용자 목록 조회
      operationId: listUsers
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/LimitParam'
        - $ref: '#/components/parameters/SortParam'
        - name: status
          in: query
          schema:
            type: string
            enum: [active, inactive, suspended]
      responses:
        '200':
          description: 성공
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'
        '401':
          $ref: '#/components/responses/Unauthorized'
        '429':
          $ref: '#/components/responses/TooManyRequests'

    post:
      tags: [users]
      summary: 사용자 생성
      operationId: createUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: 생성 성공
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          $ref: '#/components/responses/Conflict'

  /users/{userId}:
    get:
      tags: [users]
      summary: 사용자 상세 조회
      operationId: getUser
      parameters:
        - $ref: '#/components/parameters/UserIdParam'
      responses:
        '200':
          description: 성공
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
        '404':
          $ref: '#/components/responses/NotFound'

components:
  parameters:
    PageParam:
      name: page
      in: query
      schema:
        type: integer
        minimum: 1
        default: 1

    LimitParam:
      name: limit
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20

    SortParam:
      name: sort
      in: query
      schema:
        type: string
      description: "정렬 기준 (예: -created_at, name)"

    UserIdParam:
      name: userId
      in: path
      required: true
      schema:
        type: string
        format: uuid

  schemas:
    UserResponse:
      type: object
      properties:
        data:
          $ref: '#/components/schemas/User'
        links:
          type: object
          properties:
            self:
              type: string

    UserListResponse:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        meta:
          $ref: '#/components/schemas/PaginationMeta'

    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
        email:
          type: string
          format: email
        isActive:
          type: boolean
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time

    CreateUserRequest:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email

    PaginationMeta:
      type: object
      properties:
        total:
          type: integer
        page:
          type: integer
        limit:
          type: integer
        totalPages:
          type: integer
        hasNext:
          type: boolean
        hasPrev:
          type: boolean

    ProblemDetail:
      type: object
      properties:
        type:
          type: string
          format: uri
        title:
          type: string
        status:
          type: integer
        detail:
          type: string
        instance:
          type: string
        errors:
          type: array
          items:
            type: object
            properties:
              field:
                type: string
              message:
                type: string
              code:
                type: string

  responses:
    BadRequest:
      description: 잘못된 요청
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetail'
          example:
            type: https://api.example.com/errors/validation-error
            title: Validation Error
            status: 400
            detail: "요청 본문의 유효성 검증에 실패했습니다."

    Unauthorized:
      description: 인증 필요
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetail'
          example:
            type: https://api.example.com/errors/unauthorized
            title: Unauthorized
            status: 401
            detail: "유효한 인증 토큰이 필요합니다."

    NotFound:
      description: 리소스 없음
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetail'
          example:
            type: https://api.example.com/errors/not-found
            title: Not Found
            status: 404
            detail: "요청한 리소스를 찾을 수 없습니다."

    Conflict:
      description: 충돌
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetail'
          example:
            type: https://api.example.com/errors/conflict
            title: Conflict
            status: 409
            detail: "이미 존재하는 리소스입니다."

    TooManyRequests:
      description: 요청 초과
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetail'
          example:
            type: https://api.example.com/errors/rate-limit-exceeded
            title: Too Many Requests
            status: 429
            detail: "요청 횟수가 제한을 초과했습니다."
      headers:
        Retry-After:
          schema:
            type: integer
          description: 재시도까지 대기 시간(초)
        X-RateLimit-Limit:
          schema:
            type: integer
          description: 제한 시간 내 허용 요청 수
        X-RateLimit-Remaining:
          schema:
            type: integer
          description: 남은 요청 수

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - BearerAuth: []
```

---

## 4. 일관성 점수 카드 형식

### 점수 카드 템플릿

```markdown
## API 일관성 점수 카드

### 종합 점수: {TOTAL_SCORE}/100 ({TOTAL_GRADE})

#### 세부 항목

| 항목 | 점수 | 가중치 | 기여 점수 | 상태 |
|------|------|--------|----------|------|
| 리소스 네이밍 | {SCORE}/100 | 15% | {WEIGHTED} | {EMOJI_STATUS} |
| HTTP 메서드 의미론 | {SCORE}/100 | 15% | {WEIGHTED} | {EMOJI_STATUS} |
| 필드 네이밍 일관성 | {SCORE}/100 | 20% | {WEIGHTED} | {EMOJI_STATUS} |
| 에러 응답 구조 | {SCORE}/100 | 15% | {WEIGHTED} | {EMOJI_STATUS} |
| 페이지네이션 | {SCORE}/100 | 10% | {WEIGHTED} | {EMOJI_STATUS} |
| 날짜/시간 형식 | {SCORE}/100 | 10% | {WEIGHTED} | {EMOJI_STATUS} |
| 버전 관리 | {SCORE}/100 | 5% | {WEIGHTED} | {EMOJI_STATUS} |
| 추가 항목 | {SCORE}/100 | 10% | {WEIGHTED} | {EMOJI_STATUS} |

#### 등급 기준
- A (90-100): 우수 - API 설계 일관성이 잘 유지됨
- B (80-89): 양호 - 소수 항목에서 개선 필요
- C (70-79): 보통 - 여러 항목에서 개선 필요
- D (60-69): 미흡 - 상당수 항목에서 개선 필요
- F (0-59): 불량 - 전반적인 API 설계 검토 필요

#### 개선 우선순위
1. {PRIORITY_1}
2. {PRIORITY_2}
3. {PRIORITY_3}
```

---

## 5. Rate Limit 헤더 설정 템플릿

### Express.js (express-rate-limit)

```javascript
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15분
    max: 100,                   // 최대 100회
    standardHeaders: true,      // RateLimit-* 헤더 사용
    legacyHeaders: false,       // X-RateLimit-* 헤더 비활성화
    message: {
        type: 'https://api.example.com/errors/rate-limit-exceeded',
        title: 'Too Many Requests',
        status: 429,
        detail: '요청 횟수가 제한을 초과했습니다. 잠시 후 다시 시도하세요.'
    }
});

app.use('/api/', apiLimiter);
```

### Nginx

```nginx
# Rate limiting 설정
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

server {
    location /api/ {
        limit_req zone=api burst=20 nodelay;
        limit_req_status 429;

        # Rate Limit 헤더 추가
        add_header X-RateLimit-Limit 100 always;
        add_header X-RateLimit-Remaining $limit_remaining always;
        add_header X-RateLimit-Reset $limit_reset always;
    }
}
```
