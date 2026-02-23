---
name: api-design-reviewer
description: API 설계의 일관성과 REST 규약 준수 여부를 분석하고, 응답 형식, 페이지네이션, 에러 처리, 버전 관리 등 전반적인 API 디자인 품질을 검토하여 개선 가이드를 제공하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) API 설계 리뷰, (2) REST API 분석, (3) 엔드포인트 분석, (4) API 일관성 검사, (5) API 컨벤션 확인, (6) api design review, (7) endpoint review, (8) API 응답 형식 검토.
---

# API Design Reviewer

API 설계의 일관성과 REST 규약 준수 여부를 종합적으로 분석합니다. 리소스 네이밍, HTTP 메서드 의미론, 응답 필드 명명 규칙, 페이지네이션 패턴, 에러 응답 표준화, 버전 관리 전략 등을 검토하고, 구체적인 개선 가이드를 제공합니다.

## Quick Start

사용자가 API 설계 리뷰를 요청하면 다음 워크플로우를 실행합니다:

1. **분석 대상 수집**:
   ```bash
   # API 라우트 파일 탐색
   find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.java" -o -name "*.go" -o -name "*.rb" \) \
       -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/dist/*" \
       | xargs grep -l -E "(app\.(get|post|put|patch|delete)|@(Get|Post|Put|Patch|Delete)Mapping|@api_view|router\.(get|post|put|patch|delete)|@app\.route)" 2>/dev/null

   # OpenAPI/Swagger 정의 파일 탐색
   find . -type f \( -name "openapi.yaml" -o -name "openapi.json" -o -name "swagger.yaml" -o -name "swagger.json" -o -name "*.openapi.yaml" -o -name "*.openapi.json" \) \
       -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null
   ```

2. **엔드포인트 목록 추출** 후 REST 규약 준수 여부 패턴 매칭

3. **응답 구조 분석**: 필드 네이밍 일관성, 날짜 형식, null 처리 확인

4. **페이지네이션 패턴 분석**: 커서 기반 vs 오프셋 기반 일관성 확인

5. **에러 응답 표준화 검증**: RFC 7807 준수 여부, 상태 코드 적절성

6. **버전 관리 전략 확인**: URL 경로, 헤더, 쿼리 파라미터 방식 일관성

7. **결과를 심각도별 분류** (Critical / High / Medium / Low / Info)

8. **개선 가이드 및 보고서 생성** ([references/templates.md](references/templates.md) 형식 사용)

## REST 규약 준수 검사

각 REST 설계 원칙별로 엔드포인트를 분석합니다. 상세 분석 방법론은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

### 리소스 네이밍 규칙

**올바른 패턴**: 복수형 명사, 동사 미사용, 계층 구조 표현

```
# 올바른 리소스 네이밍
GET    /users                    # 사용자 목록 조회
GET    /users/:id                # 특정 사용자 조회
POST   /users                    # 사용자 생성
PUT    /users/:id                # 사용자 전체 수정
PATCH  /users/:id                # 사용자 부분 수정
DELETE /users/:id                # 사용자 삭제
GET    /users/:id/posts          # 특정 사용자의 게시물 목록
GET    /users/:id/posts/:postId  # 특정 사용자의 특정 게시물
```

**탐지 패턴 (위반 사항)**:

```regex
# URL에 동사 사용 (위반)
(get|fetch|retrieve|create|add|update|modify|edit|delete|remove)[-_]?\w+\s*['"]
['"]/(get|create|update|delete|fetch|add|remove|modify|edit|find|search|list)[-/]

# 단수형 리소스 이름 (위반)
['"]/(?!auth|login|logout|health|status|me|search|batch|bulk)(user|post|comment|order|product|item|category|tag|role|permission|notification|message|file|image|document|report|log|event|task|project|team|member|setting|config|profile|account|payment|invoice|subscription|session|token)[/'"]

# camelCase 또는 PascalCase URL 경로 (위반)
['"]\/[a-z]+[A-Z][a-zA-Z]*[/'"]
['"]\/[A-Z][a-zA-Z]+[/'"]

# 언더스코어 URL 경로 (위반 - 하이픈 권장)
['"]\/[a-z]+_[a-z]+[/'"]
```

**검증 항목**:
- 모든 리소스 이름이 복수형 명사인지 확인
- URL 경로에 동사가 포함되지 않는지 확인 (actions 엔드포인트 제외)
- 리소스 계층 구조가 논리적인지 확인
- URL 경로가 kebab-case를 사용하는지 확인
- 경로 깊이가 3단계 이내인지 확인 (`/a/:id/b/:id/c`까지 권장)

### HTTP 메서드 의미론

각 HTTP 메서드가 올바른 의미로 사용되는지 검증합니다.

| 메서드 | 의미 | 멱등성 | 안전성 | 요청 본문 | 응답 상태 코드 |
|--------|------|--------|--------|----------|---------------|
| GET | 리소스 조회 | O | O | 없음 | 200 OK |
| POST | 리소스 생성 | X | X | 있음 | 201 Created |
| PUT | 리소스 전체 교체 | O | X | 있음 | 200 OK |
| PATCH | 리소스 부분 수정 | X | X | 있음 | 200 OK |
| DELETE | 리소스 삭제 | O | X | 없음 | 204 No Content |
| HEAD | 헤더만 조회 | O | O | 없음 | 200 OK |
| OPTIONS | 허용 메서드 조회 | O | O | 없음 | 204 No Content |

**탐지 패턴 (위반 사항)**:

```regex
# GET 요청에서 데이터 변경 (위반)
(app|router)\.(get)\s*\([^)]*\)[\s\S]*?(\.save\(|\.create\(|\.update\(|\.delete\(|\.remove\(|\.destroy\(|INSERT|UPDATE|DELETE)

# POST 요청에서 조회만 수행 (설계 검토 필요)
(app|router)\.(post)\s*\([^)]*search[^)]*\)

# DELETE 요청에 요청 본문 사용 (비권장)
(app|router)\.(delete)\s*\([^)]*\)[\s\S]*?req\.body

# POST에서 201 대신 200 반환 (검토 필요)
(app|router)\.(post)\s*\([^)]*\)[\s\S]*?res\.(status\(200\)|json\()(?!.*status\(201\))
```

**검증 항목**:
- GET 요청이 부수 효과를 발생시키지 않는지 확인
- POST 요청이 리소스 생성 후 201 상태 코드를 반환하는지 확인
- PUT 요청이 전체 리소스를 교체하는지 확인 (부분 수정이면 PATCH 사용)
- DELETE 요청이 204 또는 200을 반환하는지 확인
- 멱등성이 보장되어야 하는 메서드(GET, PUT, DELETE)가 실제로 멱등한지 확인

### URL 구조 및 쿼리 파라미터

**필터링 쿼리 파라미터 규칙**:

```
# 올바른 필터링 패턴
GET /users?status=active                    # 단일 필터
GET /users?status=active&role=admin         # 복수 필터
GET /users?created_after=2024-01-01         # 날짜 범위 필터
GET /users?name=john                        # 검색 필터
GET /users?fields=id,name,email             # 필드 선택 (sparse fieldsets)
```

**정렬 쿼리 파라미터 규칙**:

```
# 올바른 정렬 패턴
GET /users?sort=created_at                  # 오름차순 정렬
GET /users?sort=-created_at                 # 내림차순 정렬 (- 접두사)
GET /users?sort=name,-created_at            # 다중 정렬
GET /users?order_by=name&order=asc          # 명시적 방향 지정
```

**페이지네이션 쿼리 파라미터 규칙**:

```
# 오프셋 기반
GET /users?page=2&limit=20                  # 페이지 + 제한
GET /users?offset=20&limit=20               # 오프셋 + 제한

# 커서 기반
GET /users?cursor=abc123&limit=20           # 커서 + 제한
GET /users?after=abc123&first=20            # GraphQL 스타일
```

**탐지 패턴 (위반 사항)**:

```regex
# 비표준 쿼리 파라미터 이름
(pageNum|pageNo|pageSize|per_page|perPage|count|num|rows|size)(?!=.*(?:표준|standard|convention))

# 배열 파라미터 비일관적 표현
(\?|\&)(ids=[^&]*,[^&]*|ids\[\]=[^&]*|ids\.0=)
```

## 응답 필드 네이밍 일관성

### 네이밍 컨벤션 탐지

프로젝트 전체에서 응답 필드의 네이밍 컨벤션이 일관적인지 확인합니다.

**탐지 대상**:

| 검사 항목 | camelCase 예시 | snake_case 예시 | 심각도 |
|----------|---------------|----------------|--------|
| 필드 네이밍 혼용 | `userId`, `userName` | `user_id`, `user_name` | High |
| 불리언 필드 접두사 | `isActive`, `hasPermission` | `is_active`, `has_permission` | Medium |
| 날짜 필드 형식 | `createdAt: "2024-01-01T00:00:00Z"` | `created_at: "2024-01-01"` | Medium |
| ID 필드 일관성 | `userId` vs `user_id` | - | High |
| 상태 필드 표현 | `status: "ACTIVE"` | `status: "active"` | Low |

**탐지 정규식**:

```regex
# 동일 응답에서 camelCase와 snake_case 혼용 탐지
# camelCase 필드
['"]([\w]*[a-z][A-Z]\w*)['"]\s*:

# snake_case 필드
['"]([\w]*_[\w]+)['"]\s*:

# 불리언 필드에 is_/has_ 접두사 미사용
['"](active|enabled|disabled|visible|hidden|deleted|verified|confirmed|approved|blocked|locked|public|private|read|archived|starred|pinned|muted|featured)['"]\s*:\s*(true|false)

# 비표준 날짜 형식 (ISO 8601 미준수)
['"]\w*(date|time|_at|At|Date|Time|Timestamp|timestamp)['"]\s*:\s*['"](?!\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2})

# null vs 부재 필드 비일관적 처리
['"]\w+['"]\s*:\s*null
```

**검증 항목**:
- 전체 API에서 단일 네이밍 컨벤션(camelCase 또는 snake_case)을 사용하는지 확인
- 불리언 필드에 `is_`/`has_`/`can_`/`should_` 접두사를 사용하는지 확인
- 날짜/시간 필드가 ISO 8601 형식(`YYYY-MM-DDTHH:mm:ssZ`)을 사용하는지 확인
- null 필드와 부재 필드의 처리 방식이 일관적인지 확인
- 열거형 값의 표기 방식(UPPER_CASE, lowercase, PascalCase)이 일관적인지 확인

### 응답 봉투(Envelope) 패턴

```json
// 권장: 일관적인 응답 구조
{
    "data": { ... },
    "meta": {
        "requestId": "req-abc123",
        "timestamp": "2024-01-01T00:00:00Z"
    }
}

// 목록 응답
{
    "data": [ ... ],
    "meta": {
        "total": 150,
        "page": 1,
        "limit": 20,
        "hasNext": true
    }
}
```

**탐지 정규식**:

```regex
# 응답 봉투 미사용 (배열 직접 반환)
res\.(json|send)\s*\(\s*\[
res\.(json|send)\s*\(\s*await\s+\w+\.find

# 비일관적 봉투 구조
['"](result|results|items|records|rows|entries|content|payload|response|body)['"]\s*:
```

## 페이지네이션 패턴 분석

### 커서 기반 vs 오프셋 기반 탐지

프로젝트 내 목록 엔드포인트에서 사용되는 페이지네이션 방식을 식별합니다.

**오프셋 기반 탐지**:

```regex
# 오프셋 기반 페이지네이션 패턴
(page|offset)\s*[:=]\s*(req\.(query|params)|request\.(query|GET|args)|parseInt|Number)
\.skip\s*\(\s*(page|offset)
\.offset\s*\(\s*(page|offset)
OFFSET\s+\?|OFFSET\s+\$|OFFSET\s+:
LIMIT\s+\?\s*OFFSET|LIMIT\s+\$\d+\s+OFFSET
```

**커서 기반 탐지**:

```regex
# 커서 기반 페이지네이션 패턴
(cursor|after|before|next_cursor|nextCursor|next_token|nextToken)\s*[:=]\s*(req\.(query|params)|request\.(query|GET|args))
where\s*\(\s*['"]id['"]\s*[><=]\s*\?
\.where\s*\(\s*['"]id['"]\s*,\s*['"][><=]['"]\s*,
(next_cursor|nextCursor|cursor|endCursor|next_page_token)\s*[:=]
```

**검증 항목**:
- 모든 목록 엔드포인트에 페이지네이션이 적용되었는지 확인
- 페이지네이션 방식이 프로젝트 전체에서 일관적인지 확인
- 기본 페이지 크기(`limit`)가 설정되어 있는지 확인
- 최대 페이지 크기 제한이 있는지 확인
- 응답에 총 항목 수(`total`) 또는 다음 페이지 존재 여부(`hasNext`)가 포함되는지 확인
- 커서 기반의 경우 커서 값이 불투명(opaque)한지 확인 (단순 ID 노출 방지)

**페이지네이션 미적용 목록 엔드포인트 탐지**:

```regex
# 목록 조회인데 페이지네이션 파라미터 미사용
(app|router)\.(get)\s*\(\s*['"]\/\w+['"]\s*,[\s\S]*?(\.find\(|\.findAll\(|\.all\(|SELECT\s+\*|\.list\()[\s\S]*?(?!.*(?:limit|page|offset|cursor|skip|take|pagination|paginate))
```

### 페이지네이션 응답 형식

| 필수 필드 | 오프셋 기반 | 커서 기반 | 설명 |
|----------|-----------|----------|------|
| `data` | O | O | 결과 배열 |
| `total` | O | 선택 | 전체 항목 수 |
| `page` | O | X | 현재 페이지 번호 |
| `limit` | O | O | 페이지 크기 |
| `totalPages` | 선택 | X | 전체 페이지 수 |
| `hasNext` | 선택 | O | 다음 페이지 존재 여부 |
| `hasPrev` | 선택 | O | 이전 페이지 존재 여부 |
| `nextCursor` | X | O | 다음 페이지 커서 |
| `prevCursor` | X | 선택 | 이전 페이지 커서 |

## 에러 응답 표준화

### RFC 7807 Problem Details 형식

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

**탐지 패턴 (위반 사항)**:

```regex
# 비표준 에러 응답 구조
res\.(status\(\d+\)\.)?(json|send)\s*\(\s*\{\s*['"]?(error|err|msg|message|errorMessage)['"]?\s*:

# 에러 응답에 상태 코드 누락
res\.json\s*\(\s*\{\s*['"]error['"]

# 비일관적 에러 필드명
['"](error_code|errorCode|err_code|errCode|error_message|errorMessage|err_msg|errMsg)['"]\s*:

# 부적절한 HTTP 상태 코드 사용
res\.status\(200\)[\s\S]*?(error|err|fail|invalid|unauthorized|forbidden|not.?found)
res\.status\(500\)[\s\S]*?(validation|invalid|missing|required)
```

### HTTP 상태 코드 적절성

| 상태 코드 | 의미 | 사용 장면 |
|----------|------|----------|
| 200 OK | 성공 | 조회/수정 성공 |
| 201 Created | 생성 성공 | POST로 리소스 생성 시 |
| 204 No Content | 본문 없음 | DELETE 성공 시 |
| 400 Bad Request | 잘못된 요청 | 유효성 검증 실패 |
| 401 Unauthorized | 인증 필요 | 인증 토큰 미제공/만료 |
| 403 Forbidden | 접근 거부 | 권한 부족 |
| 404 Not Found | 리소스 없음 | 존재하지 않는 리소스 |
| 409 Conflict | 충돌 | 중복 리소스 생성 시도 |
| 422 Unprocessable Entity | 처리 불가 | 의미적 유효성 검증 실패 |
| 429 Too Many Requests | 요청 초과 | Rate limiting 적용 |
| 500 Internal Server Error | 서버 오류 | 예기치 않은 서버 오류 |

**검증 항목**:
- 모든 에러 응답이 일관적인 구조를 사용하는지 확인
- HTTP 상태 코드가 에러 유형에 적절한지 확인
- 에러 코드가 열거형으로 정의되어 있는지 확인
- 클라이언트가 에러를 프로그래밍적으로 처리할 수 있는지 확인
- 내부 서버 오류 시 민감 정보가 노출되지 않는지 확인
- 유효성 검증 에러 시 개별 필드 에러가 제공되는지 확인

### 에러 코드 열거형 체계

```
# 권장 에러 코드 체계
AUTH_001: 인증 토큰 미제공
AUTH_002: 인증 토큰 만료
AUTH_003: 유효하지 않은 인증 토큰
AUTH_004: 권한 부족
VALIDATION_001: 필수 필드 누락
VALIDATION_002: 유효하지 않은 형식
VALIDATION_003: 범위 초과
RESOURCE_001: 리소스를 찾을 수 없음
RESOURCE_002: 리소스 중복
RESOURCE_003: 리소스 상태 충돌
RATE_001: 요청 횟수 초과
SERVER_001: 내부 서버 오류
SERVER_002: 외부 서비스 오류
```

## 버전 관리 전략

### 버전 관리 방식 탐지

**URL 경로 방식**:

```regex
# URL 경로 기반 버전
['"]\/v\d+\/
['"]\/api\/v\d+\/
(app|router)\.(use|get|post|put|patch|delete)\s*\(\s*['"]\/v\d+
```

**헤더 방식**:

```regex
# 헤더 기반 버전
(Accept-Version|API-Version|X-API-Version)\s*[:=]
req\.headers?\[['"]accept-version['"]\]
req\.headers?\[['"]api-version['"]\]
req\.header\s*\(\s*['"]Accept-Version['"]\)
```

**쿼리 파라미터 방식**:

```regex
# 쿼리 파라미터 기반 버전
[\?&]version=\d+
[\?&]v=\d+
req\.(query|params)\.version
req\.(query|params)\.v\b
```

**검증 항목**:
- 단일 버전 관리 방식을 일관적으로 사용하는지 확인
- 버전 번호가 순차적이고 의미적인지 확인
- 구버전 API에 폐기(deprecation) 헤더가 설정되어 있는지 확인
- 버전 간 호환성이 문서화되어 있는지 확인

### 폐기(Deprecation) 헤더

```regex
# Deprecation 헤더 사용 여부
(Deprecation|Sunset|X-Deprecated)\s*[:=]
res\.(set|header|setHeader)\s*\(\s*['"]Deprecation['"]
res\.(set|header|setHeader)\s*\(\s*['"]Sunset['"]
```

**권장 폐기 헤더**:

```
Deprecation: true
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Link: <https://api.example.com/v2/users>; rel="successor-version"
```

## 추가 검사 항목

### HATEOAS / 링크 기반 탐색

API 응답에 관련 리소스 링크가 포함되는지 확인합니다.

```regex
# HATEOAS 링크 존재 여부
['"](links|_links|href|self|next|prev|first|last|related)['"]\s*:
```

**권장 링크 형식**:

```json
{
    "data": { "id": 1, "name": "John" },
    "links": {
        "self": "/users/1",
        "posts": "/users/1/posts",
        "profile": "/users/1/profile"
    }
}
```

### Content-Type 협상

```regex
# Content-Type 헤더 설정 여부
res\.(type|contentType|set)\s*\(\s*['"]application\/json['"]
Content-Type\s*:\s*application\/json
Accept\s*:\s*application\/json

# 커스텀 미디어 타입
application\/vnd\.\w+
```

### 멱등성 보장 (POST/PUT)

```regex
# 멱등성 키(Idempotency-Key) 사용 여부
(Idempotency-Key|Idempotent-Key|X-Idempotency-Key|X-Request-Id)\s*[:=]
req\.headers?\[['"]idempotency-key['"]\]
req\.header\s*\(\s*['"]Idempotency-Key['"]\)
```

**검증 항목**:
- POST 요청에 멱등성 키 지원 여부 확인
- PUT 요청의 멱등성 보장 여부 확인
- 중복 요청 처리 메커니즘 존재 여부 확인

### Rate Limit 헤더

```regex
# Rate Limit 관련 헤더 설정 여부
(X-RateLimit-Limit|X-RateLimit-Remaining|X-RateLimit-Reset|RateLimit-Limit|RateLimit-Remaining|RateLimit-Reset|Retry-After)\s*[:=]
res\.(set|header|setHeader)\s*\(\s*['"](X-RateLimit|RateLimit|Retry-After)
```

**권장 Rate Limit 헤더**:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
Retry-After: 60
```

### Batch/Bulk 작업 패턴

```regex
# Batch/Bulk 엔드포인트 탐지
['"]\/(batch|bulk)['"]
['"]\/\w+\/(batch|bulk)['"]
(app|router)\.(post|put|patch|delete)\s*\(\s*['"][^'"]*\/(batch|bulk)

# 배열 기반 일괄 처리 패턴
req\.body\s*\.\s*(items|operations|requests|batch|data)\s*\.\s*(forEach|map|every|some|filter)
Array\.isArray\s*\(\s*req\.body
```

**검증 항목**:
- 대량 작업을 위한 전용 엔드포인트가 존재하는지 확인
- Batch 요청의 최대 크기 제한이 있는지 확인
- 부분 성공/실패 처리가 구현되어 있는지 확인
- Batch 응답에 각 항목별 결과가 포함되는지 확인

## 심각도 분류 기준

스캔 결과를 다음 기준으로 분류합니다:

### Critical (즉시 대응)

- GET 요청에서 데이터 변경 발생 (안전성 위반)
- 에러 응답에 내부 구현 정보 노출 (스택 트레이스 등)
- 인증 없이 민감 데이터 접근 가능
- 페이지네이션 미적용으로 전체 데이터 반환 (대규모 데이터셋)

### High (48시간 내 대응)

- 응답 필드 네이밍 규칙 혼용 (camelCase와 snake_case 혼재)
- HTTP 상태 코드 부적절 사용 (200으로 에러 반환)
- 에러 응답 구조 비일관
- 페이지네이션 방식 비일관
- 버전 관리 전략 비일관

### Medium (1주일 내 대응)

- URL에 동사 사용
- 리소스 이름 단수형 사용
- 날짜 형식 비일관 (ISO 8601 미준수)
- Rate Limit 헤더 미설정
- 불리언 필드 접두사 미사용
- 에러 코드 열거형 미정의

### Low (계획적 대응)

- HATEOAS 링크 미제공
- Content-Type 협상 미지원
- 멱등성 키 미지원
- 폐기(Deprecation) 헤더 미설정
- Batch/Bulk 엔드포인트 미제공

### Info (참고 사항)

- API 설계 모범 사례 권장
- 문서화 수준 개선 제안
- 클라이언트 SDK 자동 생성 가능 여부

## 워크플로우 상세

```
1. 분석 대상 수집
   ├─ API 라우트 파일 탐색
   ├─ OpenAPI/Swagger 정의 파일 탐색
   └─ 컨트롤러/핸들러 파일 식별

2. 엔드포인트 목록 추출
   ├─ HTTP 메서드 + URL 경로 매핑
   ├─ 요청/응답 스키마 식별
   └─ 미들웨어/데코레이터 분석

3. REST 규약 검사
   ├─ 리소스 네이밍 규칙 확인
   ├─ HTTP 메서드 의미론 검증
   ├─ URL 구조 및 쿼리 파라미터 확인
   └─ 경로 계층 구조 분석

4. 응답 구조 분석
   ├─ 필드 네이밍 일관성 확인
   ├─ 날짜/시간 형식 확인
   ├─ null 처리 방식 확인
   └─ 응답 봉투 패턴 확인

5. 페이지네이션 분석
   ├─ 방식 탐지 (커서/오프셋)
   ├─ 목록 엔드포인트 전체 적용 확인
   └─ 응답 형식 일관성 확인

6. 에러 응답 검증
   ├─ RFC 7807 준수 여부
   ├─ 상태 코드 적절성
   ├─ 에러 코드 체계 확인
   └─ 민감 정보 노출 방지 확인

7. 버전 관리 및 추가 검사
   ├─ 버전 관리 방식 일관성
   ├─ HATEOAS/링크 제공 여부
   ├─ Rate Limit 헤더 설정
   ├─ 멱등성 보장 여부
   └─ Batch 작업 지원 여부

8. 결과 분류 및 보고서 생성
   ├─ 심각도별 분류 (Critical/High/Medium/Low/Info)
   ├─ 개선 가이드 첨부
   └─ 보고서 출력 (references/templates.md 형식)
```

## 트리거 조건

이 스킬은 다음 상황에서 활성화됩니다:

- 사용자가 "API 설계 리뷰", "API 디자인 리뷰" 요청 시
- "REST API 분석", "REST API 검토" 요청 시
- "엔드포인트 분석", "엔드포인트 검토" 요청 시
- "API 일관성 검사", "API 일관성 확인" 요청 시
- "API 컨벤션", "API 컨벤션 검사" 요청 시
- "api design review", "endpoint review" 영문 요청 시
- "API 응답 형식 검토", "API 에러 형식 확인" 요청 시
- "페이지네이션 패턴 분석", "API 버전 관리 확인" 요청 시

## 에러 처리

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | API 라우트 파일을 찾을 수 없는 경우 | 파일 탐색 결과 0건 | OpenAPI 정의 파일 또는 수동 경로 입력 요청 |
| 2 | 지원하지 않는 프레임워크 | 라우트 패턴 미매칭 | 일반 정규식 기반 분석으로 전환 |
| 3 | OpenAPI 스펙 파싱 실패 | YAML/JSON 파싱 오류 | 코드 기반 분석으로 전환 |
| 4 | 대규모 코드베이스 | 파일 수 > 5,000 | 변경된 파일 또는 주요 라우트 파일만 우선 분석 |
| 5 | 동적 라우트 정의 | 정규식으로 추출 불가 | 컨텍스트 분석으로 보완하고 수동 확인 권장 |
| 6 | 마이크로서비스 환경 | 단일 저장소에 전체 API 미포함 | 서비스별 개별 분석 후 교차 검증 안내 |
| 7 | GraphQL 프로젝트 | REST 패턴 미매칭 | GraphQL 스키마 분석으로 전환하여 네이밍/에러 규칙만 검사 |
| 8 | 응답 구조 추론 불가 | 코드에서 응답 스키마 미식별 | OpenAPI 정의 기반 분석 권장, [references/analysis-guide.md](references/analysis-guide.md)의 수동 분석 방법 안내 |

## 참조 문서

- [references/analysis-guide.md](references/analysis-guide.md) - 상세 분석 방법론, 프레임워크별 라우트 추출 방법, 일관성 점수 산정 기준
- [references/templates.md](references/templates.md) - 보고서 출력 형식, 개선 가이드 템플릿, OpenAPI 보일러플레이트

## 모범 사례

1. **설계 우선 접근**: OpenAPI 스펙을 먼저 작성하고, 코드를 구현하는 API-First 방식 권장
2. **일관성 유지**: 프로젝트 전체에서 단일 네이밍 규칙, 에러 형식, 페이지네이션 방식 사용
3. **문서화 동기화**: 코드 변경 시 OpenAPI 스펙도 함께 업데이트
4. **버전 관리 계획**: API 출시 전 버전 관리 전략을 수립하고, 폐기 정책을 문서화
5. **에러 코드 체계화**: 도메인별 에러 코드를 열거형으로 정의하고, 클라이언트에 문서 제공
6. **Rate Limiting 적용**: 모든 공개 API에 Rate Limit 헤더를 설정하고, 429 응답 처리 구현
7. **정기적 리뷰**: 분기별 API 설계 리뷰를 수행하여 일관성 유지 및 개선 사항 반영
