# API Design Reviewer

## 스킬 소개

**API Design Reviewer**는 Claude Code 커스텀 스킬로, API 설계의 일관성과 REST 규약 준수 여부를 종합적으로 분석합니다. 리소스 네이밍, HTTP 메서드 의미론, 응답 필드 명명 규칙, 페이지네이션 패턴, 에러 응답 표준화, 버전 관리 전략 등을 검토하고, 구체적인 개선 가이드를 제공합니다.

### 주요 기능

- **REST 규약 준수 검사**: 리소스 네이밍(복수형 명사), URL 구조, HTTP 메서드 의미론 검증
- **응답 필드 일관성 분석**: camelCase/snake_case 혼용 탐지, 날짜 형식(ISO 8601) 확인, 불리언 필드 접두사 검사
- **페이지네이션 패턴 분석**: 커서 기반 vs 오프셋 기반 탐지, 목록 엔드포인트 전체 적용 여부 확인
- **에러 응답 표준화 검증**: RFC 7807 Problem Details 형식 준수, HTTP 상태 코드 적절성, 에러 코드 체계 확인
- **버전 관리 전략 확인**: URL 경로/헤더/쿼리 파라미터 방식 일관성, 폐기(Deprecation) 헤더 설정 여부
- **추가 검사**: HATEOAS 링크, Content-Type 협상, 멱등성 키, Rate Limit 헤더, Batch/Bulk 패턴
- **심각도별 분류**: Critical / High / Medium / Low / Info 5단계 분류 및 개선 가이드 제공

---

## 분석 항목 요약

| 분석 영역 | 주요 검사 항목 | 심각도 범위 |
|----------|--------------|-----------|
| REST 규약 | 리소스 네이밍, HTTP 메서드 의미론, URL 구조 | Medium ~ Critical |
| 응답 구조 | 필드 네이밍 일관성, 날짜 형식, 응답 봉투 패턴 | Low ~ High |
| 페이지네이션 | 방식 일관성, 목록 엔드포인트 전체 적용 | Medium ~ Critical |
| 에러 처리 | RFC 7807 준수, 상태 코드, 에러 코드 체계 | Medium ~ High |
| 버전 관리 | 방식 일관성, 폐기 헤더 | Low ~ High |
| 추가 항목 | HATEOAS, Rate Limit, 멱등성, Batch | Info ~ Low |

---

## 사용 예시

### 예시 1: REST 규약 위반 탐지

**분석 대상 코드**:
```javascript
// URL에 동사 사용 (위반)
app.get('/getUsers', async (req, res) => { ... });
app.post('/createUser', async (req, res) => { ... });
app.post('/deleteUser/:id', async (req, res) => { ... });
```

**분석 결과**:
```
[MEDIUM] REST 규약 위반 - URL에 동사 사용
  파일: src/routes/users.js:1-3
  위반: /getUsers, /createUser, /deleteUser/:id
  권장: GET /users, POST /users, DELETE /users/:id
```

### 예시 2: 응답 필드 네이밍 혼용 탐지

**분석 대상 코드**:
```javascript
// 동일 프로젝트 내 camelCase와 snake_case 혼용
// users 엔드포인트
res.json({ userId: 1, user_name: "John", createdAt: "..." });

// posts 엔드포인트
res.json({ post_id: 1, postTitle: "...", created_at: "..." });
```

**분석 결과**:
```
[HIGH] 응답 필드 네이밍 규칙 혼용
  파일: src/routes/users.js, src/routes/posts.js
  camelCase 필드: userId, createdAt, postTitle
  snake_case 필드: user_name, post_id, created_at
  권장: 프로젝트 전체에서 단일 규칙 적용 (camelCase 또는 snake_case)
```

### 예시 3: 에러 응답 비일관 탐지

**분석 대상 코드**:
```javascript
// 엔드포인트마다 다른 에러 형식
res.status(400).json({ error: "Invalid input" });
res.status(404).json({ message: "Not found", code: 404 });
res.status(500).json({ err: "Server error", status: "fail" });
```

**분석 결과**:
```
[HIGH] 에러 응답 구조 비일관
  파일: src/routes/*.js
  발견된 형식: { error }, { message, code }, { err, status }
  권장: RFC 7807 Problem Details 형식으로 통일
```

### 예시 4: 페이지네이션 미적용 탐지

**분석 대상 코드**:
```javascript
app.get('/products', async (req, res) => {
    const products = await Product.find({});  // 전체 데이터 반환
    res.json(products);
});
```

**분석 결과**:
```
[CRITICAL] 페이지네이션 미적용 - 목록 엔드포인트에서 전체 데이터 반환
  파일: src/routes/products.js:1
  엔드포인트: GET /products
  권장: limit/offset 또는 cursor 기반 페이지네이션 적용
```

---

## 에러 처리

| 상황 | 대응 |
|------|------|
| API 라우트 파일을 찾을 수 없는 경우 | OpenAPI 정의 파일 탐색 또는 수동 경로 입력 요청 |
| 지원하지 않는 프레임워크 | 일반 정규식 기반 분석으로 전환 |
| 대규모 코드베이스 | 변경된 파일 또는 주요 라우트 파일만 우선 분석 |
| 동적 라우트 정의 | 컨텍스트 분석 보완 후 수동 확인 권장 |
| 마이크로서비스 환경 | 서비스별 개별 분석 후 교차 검증 안내 |
| GraphQL 프로젝트 | GraphQL 스키마 분석으로 전환하여 네이밍/에러 규칙만 검사 |

---

## 지원 프레임워크

| 프레임워크 | 언어 | 라우트 탐지 방식 |
|-----------|------|----------------|
| Express.js | JavaScript/TypeScript | `app.get()`, `router.get()` 패턴 |
| Fastify | JavaScript/TypeScript | `fastify.get()`, `fastify.route()` 패턴 |
| NestJS | TypeScript | `@Get()`, `@Post()` 데코레이터 |
| Django REST Framework | Python | `@api_view`, `ViewSet`, URLconf |
| Flask | Python | `@app.route()`, Blueprint |
| FastAPI | Python | `@app.get()`, APIRouter |
| Spring Boot | Java | `@GetMapping`, `@PostMapping` 어노테이션 |
| Gin | Go | `router.GET()`, `router.POST()` 패턴 |
| Ruby on Rails | Ruby | `resources`, `get/post` 라우트 |
| OpenAPI/Swagger | YAML/JSON | 스펙 파일 직접 분석 |

---

## 관련 문서

- [SKILL.md](SKILL.md) - 스킬 정의 및 상세 워크플로우
- [references/analysis-guide.md](references/analysis-guide.md) - 분석 방법론, 프레임워크별 라우트 추출, 일관성 점수 산정
- [references/templates.md](references/templates.md) - 보고서 템플릿, 개선 가이드, OpenAPI 보일러플레이트
