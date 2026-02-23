# API Design Reviewer 분석 가이드

이 문서는 API 설계 리뷰의 상세 분석 방법론, 프레임워크별 라우트 추출 방법, 응답 구조 분석 기법, 일관성 점수 산정 기준, false positive 판단 기준을 정의합니다.

---

## 1. 프레임워크별 라우트 추출 방법

### Express.js (JavaScript/TypeScript)

**라우트 정의 패턴**:

```regex
# 기본 라우트
(app|router)\.(get|post|put|patch|delete|all|use)\s*\(\s*['"]([^'"]+)['"]

# 라우터 마운트
app\.use\s*\(\s*['"]([^'"]+)['"]\s*,\s*(\w+Router|\w+router|\w+Routes|\w+routes)

# 컨트롤러 기반 (class-validator 등)
@(Get|Post|Put|Patch|Delete|Head|Options|All)\s*\(\s*['"]?([^'")]*)?['"]?\s*\)
```

**추출 명령**:

```bash
# Express 라우트 추출
grep -rnE "(app|router)\.(get|post|put|patch|delete)\s*\(\s*['\"]" --include="*.js" --include="*.ts" .

# 라우터 마운트 포인트 추출
grep -rnE "app\.use\s*\(\s*['\"]\/[^'\"]+['\"]" --include="*.js" --include="*.ts" .
```

### Django REST Framework (Python)

**라우트 정의 패턴**:

```regex
# URLconf 패턴
path\s*\(\s*['"]([^'"]+)['"]
re_path\s*\(\s*['"]([^'"]+)['"]

# ViewSet 등록
router\.(register|register_viewset)\s*\(\s*['"]([^'"]+)['"]

# @api_view 데코레이터
@api_view\s*\(\s*\[([^\]]+)\]\s*\)
```

**추출 명령**:

```bash
# Django URL 패턴 추출
grep -rnE "path\s*\(\s*['\"]" --include="*.py" .
grep -rnE "router\.register\s*\(\s*['\"]" --include="*.py" .
```

### Spring Boot (Java)

**라우트 정의 패턴**:

```regex
# 컨트롤러 매핑
@(Get|Post|Put|Patch|Delete|Request)Mapping\s*\(\s*(value\s*=\s*)?['"]([^'"]+)['"]

# 클래스 레벨 매핑
@RequestMapping\s*\(\s*(value\s*=\s*)?['"]([^'"]+)['"]
```

**추출 명령**:

```bash
# Spring Boot 매핑 추출
grep -rnE "@(Get|Post|Put|Patch|Delete|Request)Mapping" --include="*.java" .
```

### FastAPI (Python)

**라우트 정의 패턴**:

```regex
# FastAPI 라우트
@(app|router)\.(get|post|put|patch|delete)\s*\(\s*['"]([^'"]+)['"]
```

**추출 명령**:

```bash
# FastAPI 라우트 추출
grep -rnE "@(app|router)\.(get|post|put|patch|delete)\s*\(\s*['\"]" --include="*.py" .
```

### Gin (Go)

**라우트 정의 패턴**:

```regex
# Gin 라우트
(router|r|group|g)\.(GET|POST|PUT|PATCH|DELETE|HEAD|OPTIONS)\s*\(\s*['"]([^'"]+)['"]
```

**추출 명령**:

```bash
# Gin 라우트 추출
grep -rnE "\.(GET|POST|PUT|PATCH|DELETE)\s*\(\s*['\"]" --include="*.go" .
```

### OpenAPI/Swagger 정의

**분석 명령**:

```bash
# OpenAPI 파일에서 경로 추출
grep -E "^\s+\/[a-zA-Z]" openapi.yaml 2>/dev/null
grep -E "\"\/[a-zA-Z]" openapi.json 2>/dev/null

# 메서드별 엔드포인트 추출
grep -B1 -E "^\s+(get|post|put|patch|delete):" openapi.yaml 2>/dev/null
```

---

## 2. 응답 구조 분석 기법

### 네이밍 컨벤션 분석

**분석 절차**:

1. 모든 API 응답에서 JSON 필드명을 추출합니다
2. 각 필드명의 네이밍 패턴(camelCase, snake_case, PascalCase, kebab-case)을 판별합니다
3. 프로젝트 전체에서 가장 많이 사용되는 패턴을 기준으로 설정합니다
4. 기준과 다른 패턴을 사용하는 필드를 위반으로 보고합니다

**네이밍 패턴 판별 정규식**:

```regex
# camelCase 판별
^[a-z][a-zA-Z0-9]*$

# snake_case 판별
^[a-z][a-z0-9]*(_[a-z0-9]+)*$

# PascalCase 판별
^[A-Z][a-zA-Z0-9]*$

# kebab-case 판별 (JSON 필드에서는 비권장)
^[a-z][a-z0-9]*(-[a-z0-9]+)*$

# UPPER_SNAKE_CASE 판별 (열거형 값)
^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$
```

**필드 추출 정규식**:

```regex
# JSON 응답 필드 추출
res\.(json|send)\s*\(\s*\{[\s\S]*?['"]([\w]+)['"]\s*:
['"]([\w]+)['"]\s*:\s*[^:,}]+[,}]

# TypeScript 인터페이스/타입에서 필드 추출
(interface|type)\s+\w+(Response|Dto|Schema|Model)\s*\{[\s\S]*?([\w]+)\s*[?:]

# Python Pydantic/dataclass에서 필드 추출
(class\s+\w+(Response|Schema|Model)[\s\S]*?)([\w]+)\s*:\s*(str|int|float|bool|list|dict|Optional)
```

### 날짜 형식 분석

**ISO 8601 준수 확인 정규식**:

```regex
# 올바른 ISO 8601 형식
\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}(\.\d{1,6})?(Z|[+-]\d{2}:\d{2})

# 비표준 날짜 형식 (위반)
\d{4}\/\d{2}\/\d{2}                    # 슬래시 구분자
\d{2}-\d{2}-\d{4}                      # DD-MM-YYYY
\d{2}\/\d{2}\/\d{4}                    # MM/DD/YYYY
\d{10,13}                               # Unix timestamp (숫자만)
[A-Z][a-z]{2}\s+\d{1,2},?\s+\d{4}     # 영문 날짜 (Jan 1, 2024)
```

### null 필드 처리 분석

**일관성 확인 방법**:

1. 응답에서 `null` 값을 가진 필드가 포함되는 경우를 수집합니다
2. 동일 유형의 응답에서 해당 필드가 아예 생략되는 경우를 수집합니다
3. 두 방식이 혼용되면 비일관으로 보고합니다

```regex
# null 반환 패턴
['"]\w+['"]\s*:\s*null

# 조건부 필드 포함 패턴 (스프레드 연산자)
\.\.\.\(\w+\s*&&\s*\{
\.\.\.\(\w+\s*\?\s*\{
(if|when)\s*\(\s*\w+\s*\)\s*\{[\s\S]*?['"]\w+['"]\s*:
```

---

## 3. 일관성 점수 산정 기준

### 점수 산정 공식

```
일관성 점수 = (일관 항목 수 / 전체 검사 항목 수) * 100

등급:
  90-100: A (우수) - 일관성이 잘 유지됨
  80-89:  B (양호) - 소수 항목에서 비일관 발견
  70-79:  C (보통) - 여러 항목에서 비일관 발견
  60-69:  D (미흡) - 상당수 항목에서 비일관
  0-59:   F (불량) - 전반적인 설계 검토 필요
```

### 검사 항목별 가중치

| 검사 항목 | 가중치 | 설명 |
|----------|--------|------|
| 리소스 네이밍 규칙 | 15% | 복수형 명사, 동사 미사용, kebab-case |
| HTTP 메서드 의미론 | 15% | 올바른 메서드 사용, 상태 코드 매칭 |
| 필드 네이밍 일관성 | 20% | 단일 컨벤션 사용 (camelCase 또는 snake_case) |
| 에러 응답 구조 | 15% | 일관적 에러 형식, 상태 코드 적절성 |
| 페이지네이션 패턴 | 10% | 단일 방식 사용, 목록 전체 적용 |
| 날짜/시간 형식 | 10% | ISO 8601 준수 |
| 버전 관리 | 5% | 단일 방식 일관 적용 |
| 추가 항목 | 10% | HATEOAS, Rate Limit, 멱등성 등 |

### 개별 항목 점수 계산

```
항목 점수 계산:
  전체 준수: 100점
  대부분 준수 (90% 이상): 80점
  부분 준수 (70-89%): 60점
  미흡 (50-69%): 40점
  대부분 미준수 (50% 미만): 20점
  완전 미준수: 0점
```

---

## 4. False Positive 판단 기준

### 제외 조건 (False Positive로 판정)

#### REST 규약

| 조건 | 설명 | 예시 |
|------|------|------|
| Actions 엔드포인트 | 리소스에 대한 동작을 표현하는 엔드포인트 | `POST /users/:id/activate` |
| 인증 관련 경로 | 로그인/로그아웃 등 인증 엔드포인트 | `/auth/login`, `/auth/logout` |
| 헬스체크 경로 | 서버 상태 확인 엔드포인트 | `/health`, `/status`, `/ping` |
| 검색 엔드포인트 | 검색 전용 엔드포인트 | `GET /search?q=keyword` |
| 단일 리소스 경로 | 현재 사용자 등 단수형이 적절한 경우 | `/me`, `/profile`, `/settings` |
| Webhook 엔드포인트 | 외부 서비스 콜백 수신 | `POST /webhooks/stripe` |
| 파일 업로드 | 파일 처리 전용 엔드포인트 | `POST /upload`, `POST /files/upload` |
| RPC 스타일 명시적 설계 | 의도적 RPC 패턴 | `POST /rpc/calculateTax` |

#### 응답 구조

| 조건 | 설명 | 예시 |
|------|------|------|
| 외부 API 프록시 | 외부 서비스의 응답을 그대로 전달 | 결제 게이트웨이 응답 |
| 레거시 호환성 | 기존 클라이언트와의 호환성 유지 | 구버전 API 응답 형식 |
| 표준 프로토콜 | OAuth, OpenID Connect 등 표준 응답 | `access_token`, `token_type` |
| 서드파티 라이브러리 | 프레임워크가 자동 생성하는 응답 | Django REST Framework 기본 에러 |

### 컨텍스트 분석 규칙

1. **의도적 설계 확인**: 주석이나 문서에 특정 설계 결정의 이유가 명시된 경우 심각도 하향
2. **파일 경로 기반 판단**:
   ```
   # FP 가능성 높은 경로
   */test/*
   */tests/*
   */__tests__/*
   */mock/*
   */fixture/*
   */example/*
   */docs/*
   */migrations/*
   */scripts/*
   ```
3. **프레임워크 기본 동작**: 프레임워크가 자동으로 생성하는 응답 구조는 위반으로 판정하지 않음
4. **버전별 분리**: 구버전 API(`/v1/`)와 신버전 API(`/v2/`)의 설계 차이는 별도로 평가

### 심각도 조정 규칙

| 조건 | 조정 |
|------|------|
| 내부 전용 API (관리자 도구) | 심각도 1단계 하향 |
| 프로토타입/MVP 단계 | 심각도 1단계 하향 |
| 공개 API (외부 개발자 사용) | 심각도 유지 또는 1단계 상향 |
| 결제/인증 관련 API | 심각도 1단계 상향 |
| 레거시 호환성 유지 | 심각도 1단계 하향 (단, 신규 API에는 미적용) |

---

## 5. 엔드포인트 분석 절차

### 1단계: 전체 엔드포인트 목록 작성

```bash
# 모든 라우트 파일에서 엔드포인트 추출
find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.java" -o -name "*.go" \) \
    -not -path "*/node_modules/*" -not -path "*/.git/*" -not -path "*/vendor/*" -not -path "*/dist/*" \
    -not -path "*/__pycache__/*" \
    | head -5000
```

### 2단계: 엔드포인트 분류

| 분류 | 설명 | 검사 중점 |
|------|------|----------|
| 목록 조회 | `GET /resources` | 페이지네이션, 필터링, 정렬 |
| 단건 조회 | `GET /resources/:id` | 응답 구조, 404 처리 |
| 생성 | `POST /resources` | 201 상태 코드, 유효성 검증, 멱등성 |
| 전체 수정 | `PUT /resources/:id` | 멱등성, 전체 필드 교체 |
| 부분 수정 | `PATCH /resources/:id` | 부분 필드 수정 |
| 삭제 | `DELETE /resources/:id` | 204 상태 코드, 멱등성 |
| 관계 리소스 | `GET /resources/:id/children` | 계층 구조, 페이지네이션 |
| 액션 | `POST /resources/:id/action` | 동사 허용, 상태 코드 |

### 3단계: 교차 검증

1. **리소스 간 일관성**: 모든 리소스가 동일한 CRUD 패턴을 따르는지 확인
2. **응답 구조 통일**: 모든 엔드포인트의 응답 봉투가 동일한지 확인
3. **에러 처리 통일**: 모든 엔드포인트가 동일한 에러 형식을 사용하는지 확인
4. **인증/인가 일관성**: 동일 수준의 리소스에 동일한 인증 정책이 적용되는지 확인

### 4단계: 결과 종합 및 보고서 생성

1. 각 검사 항목별 점수를 산정합니다
2. 가중치를 적용하여 일관성 점수를 계산합니다
3. 심각도별로 분류하여 보고서를 생성합니다 ([references/templates.md](templates.md) 형식 사용)

---

## 6. 수동 분석 방법 안내

코드에서 응답 구조를 자동 추론할 수 없는 경우, 다음 방법으로 수동 분석을 진행합니다.

### OpenAPI 정의 기반 분석

```bash
# OpenAPI 파일이 존재하는 경우
find . -type f \( -name "openapi.*" -o -name "swagger.*" -o -name "api-spec.*" \) \
    -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null
```

OpenAPI 파일에서 다음 항목을 추출하여 분석합니다:

1. `paths` 섹션: 모든 엔드포인트 경로와 메서드
2. `components/schemas` 섹션: 응답 스키마 정의
3. `components/responses` 섹션: 공통 응답 정의
4. `info.version`: API 버전 정보

### 실행 기반 분석

개발 서버를 실행하여 실제 응답을 수집합니다:

```bash
# curl을 이용한 응답 수집
curl -s http://localhost:3000/api/users | jq '.' 2>/dev/null
curl -s http://localhost:3000/api/users/1 | jq '.' 2>/dev/null
curl -s http://localhost:3000/api/users?invalid=true 2>/dev/null

# 에러 응답 수집
curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/api/nonexistent 2>/dev/null
```

### Postman/Insomnia 컬렉션 분석

```bash
# Postman 컬렉션 파일 탐색
find . -type f \( -name "*.postman_collection.json" -o -name "*.insomnia.json" -o -name ".insomnia" \) \
    -not -path "*/node_modules/*" -not -path "*/.git/*" 2>/dev/null
```

---

## 7. 프레임워크별 주요 검사 포인트

### Express.js 특화 검사

| 검사 항목 | 패턴 | 심각도 |
|----------|------|--------|
| 에러 미들웨어 존재 | `app.use((err, req, res, next)` | High |
| 404 핸들러 존재 | `app.use((req, res)` (마지막 미들웨어) | Medium |
| helmet 사용 | `app.use(helmet` | Medium |
| cors 설정 | `app.use(cors` | Medium |
| express-validator 사용 | `body().isEmail()`, `param().isInt()` | Medium |
| express-rate-limit 사용 | `rateLimit({` | Low |

### Django REST Framework 특화 검사

| 검사 항목 | 패턴 | 심각도 |
|----------|------|--------|
| Serializer 정의 | `class *Serializer(serializers.` | Medium |
| Permission 설정 | `permission_classes` | High |
| Pagination 설정 | `DEFAULT_PAGINATION_CLASS` | Medium |
| Throttle 설정 | `DEFAULT_THROTTLE_RATES` | Low |
| Filter 설정 | `filterset_fields`, `filter_backends` | Low |
| Exception handler | `EXCEPTION_HANDLER` | Medium |

### Spring Boot 특화 검사

| 검사 항목 | 패턴 | 심각도 |
|----------|------|--------|
| `@Valid` 사용 | `@Valid @RequestBody` | Medium |
| `@ControllerAdvice` | `@ControllerAdvice` 전역 예외 처리 | High |
| ResponseEntity 사용 | `ResponseEntity<` | Medium |
| Pageable 사용 | `Pageable pageable` | Medium |
| `@PreAuthorize` | 메서드 레벨 인가 | High |
| CORS 설정 | `@CrossOrigin`, `WebMvcConfigurer` | Medium |
