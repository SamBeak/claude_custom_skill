# Doc Generator 분석 가이드

프로젝트 코드베이스를 분석하여 문서를 자동 생성하기 위한 상세 방법론을 설명합니다.

## 목차

- [1. 프로젝트 구조 분석](#1-프로젝트-구조-분석)
- [2. 기술 스택 감지](#2-기술-스택-감지)
- [3. API 엔드포인트 추출](#3-api-엔드포인트-추출)
- [4. CHANGELOG 생성 로직](#4-changelog-생성-로직)
- [5. JSDoc/Docstring 분석](#5-jsdocdocstring-분석)
- [6. 분석 품질 체크리스트](#6-분석-품질-체크리스트)

---

## 1. 프로젝트 구조 분석

### 1.1 엔트리 포인트 감지

프로젝트의 진입점을 다음 순서로 감지합니다:

```
[엔트리 포인트 탐색 순서]
├─ package.json → "main", "module", "exports" 필드
├─ tsconfig.json → "include", "rootDir" 필드
├─ setup.py / pyproject.toml → 패키지 진입점
├─ pom.xml / build.gradle → 메인 클래스
├─ go.mod → 모듈 경로
├─ main.* 파일 (main.ts, main.py, Main.java, main.go)
├─ index.* 파일 (index.ts, index.js)
├─ app.* 파일 (app.ts, app.py, App.java)
└─ src/ 디렉토리 존재 여부
```

### 1.2 디렉토리 역할 분류

주요 디렉토리 패턴과 역할 매핑:

| 디렉토리 패턴 | 역할 | README 설명 |
|--------------|------|-------------|
| `src/`, `lib/`, `app/` | 소스 코드 | 애플리케이션 핵심 코드 |
| `src/components/`, `src/pages/` | UI 컴포넌트 | 프론트엔드 컴포넌트 및 페이지 |
| `src/api/`, `src/routes/`, `src/controllers/` | API 레이어 | API 엔드포인트 및 라우팅 |
| `src/services/`, `src/usecases/` | 비즈니스 로직 | 핵심 비즈니스 로직 |
| `src/models/`, `src/entities/` | 데이터 모델 | 데이터베이스 모델/엔티티 |
| `src/utils/`, `src/helpers/` | 유틸리티 | 공통 유틸리티 함수 |
| `src/config/` | 설정 | 애플리케이션 설정 |
| `src/middleware/` | 미들웨어 | Express/Koa 미들웨어 |
| `tests/`, `__tests__/`, `spec/` | 테스트 | 테스트 코드 |
| `docs/` | 문서 | 프로젝트 문서 |
| `scripts/` | 스크립트 | 빌드/배포/유틸리티 스크립트 |
| `public/`, `static/`, `assets/` | 정적 파일 | 정적 리소스 |
| `migrations/`, `prisma/` | DB 마이그레이션 | 데이터베이스 스키마 변경 |
| `.github/`, `.gitlab/` | CI/CD | CI/CD 워크플로우 설정 |
| `docker/`, `k8s/`, `terraform/` | 인프라 | 인프라 코드 |

### 1.3 디렉토리 트리 생성 규칙

- **최대 깊이**: 3단계 (너무 깊으면 가독성 저하)
- **파일 표시**: 주요 설정 파일만 트리에 포함 (package.json, tsconfig.json 등)
- **숨김 제외**: `.git/`, `.next/`, `__pycache__/` 등 제외
- **정렬**: 디렉토리 먼저, 파일은 알파벳순

```
my-project/
├── src/
│   ├── components/    # React 컴포넌트
│   ├── pages/         # 페이지 라우트
│   ├── services/      # 비즈니스 로직
│   ├── utils/         # 유틸리티 함수
│   └── index.ts       # 엔트리 포인트
├── tests/             # 테스트 코드
├── docs/              # 문서
├── package.json
├── tsconfig.json
└── README.md
```

---

## 2. 기술 스택 감지

### 2.1 패키지 매니저 감지

| 패키지 매니저 | 감지 파일 | 설치 명령 |
|--------------|-----------|-----------|
| npm | `package-lock.json` | `npm install` |
| yarn | `yarn.lock` | `yarn install` |
| pnpm | `pnpm-lock.yaml` | `pnpm install` |
| bun | `bun.lockb` | `bun install` |
| pip | `requirements.txt` | `pip install -r requirements.txt` |
| pipenv | `Pipfile` | `pipenv install` |
| poetry | `pyproject.toml` + `poetry.lock` | `poetry install` |
| Maven | `pom.xml` | `mvn install` |
| Gradle | `build.gradle` / `build.gradle.kts` | `./gradlew build` |
| Go Modules | `go.mod` | `go mod download` |
| Cargo | `Cargo.toml` | `cargo build` |
| Composer | `composer.json` | `composer install` |
| Bundler | `Gemfile` | `bundle install` |

### 2.2 프레임워크 감지

**감지 우선순위**: 의존성 목록 > 설정 파일 > 디렉토리 구조 > 파일 패턴

#### Node.js / TypeScript 프레임워크

```
package.json dependencies 에서 감지:
├─ "next" → Next.js
├─ "nuxt" → Nuxt.js
├─ "react" (without next/nuxt) → React (CRA or Vite)
├─ "vue" (without nuxt) → Vue.js
├─ "svelte" / "@sveltejs/kit" → Svelte / SvelteKit
├─ "@angular/core" → Angular
├─ "express" → Express.js
├─ "fastify" → Fastify
├─ "@nestjs/core" → NestJS
├─ "koa" → Koa
├─ "hono" → Hono
├─ "elysia" → Elysia (Bun)
└─ "electron" → Electron
```

#### Python 프레임워크

```
requirements.txt 또는 pyproject.toml 에서 감지:
├─ "django" → Django
├─ "flask" → Flask
├─ "fastapi" → FastAPI
├─ "tornado" → Tornado
├─ "starlette" → Starlette
└─ "aiohttp" → aiohttp
```

#### Java 프레임워크

```
pom.xml 또는 build.gradle 에서 감지:
├─ "spring-boot" → Spring Boot
├─ "quarkus" → Quarkus
├─ "micronaut" → Micronaut
└─ "jakarta.ee" → Jakarta EE
```

### 2.3 데이터베이스 감지

| 감지 소스 | 데이터베이스 |
|-----------|------------|
| `prisma/schema.prisma` → `provider` | PostgreSQL, MySQL, SQLite, MongoDB |
| `ormconfig.*`, `typeorm` 의존성 | TypeORM (DB 종류는 설정에서 추출) |
| `drizzle.config.*` | Drizzle ORM |
| `sequelize` 의존성 | Sequelize |
| `mongoose` 의존성 | MongoDB |
| `redis`, `ioredis` 의존성 | Redis |
| `docker-compose.yml` 내 서비스 | 컨테이너화된 DB |

### 2.4 실행 스크립트 추출

`package.json`의 `scripts` 필드에서 주요 명령 추출:

| 스크립트 키 | 문서 섹션 | 설명 |
|------------|-----------|------|
| `dev`, `start:dev`, `serve` | 개발 서버 실행 | 로컬 개발 환경 |
| `start`, `start:prod` | 프로덕션 실행 | 프로덕션 환경 |
| `build` | 빌드 | 프로덕션 빌드 |
| `test`, `test:unit`, `test:e2e` | 테스트 | 테스트 실행 |
| `lint`, `lint:fix` | 린팅 | 코드 스타일 검사 |
| `format` | 포매팅 | 코드 포매팅 |
| `db:migrate`, `prisma migrate` | DB 마이그레이션 | 스키마 변경 적용 |
| `seed`, `db:seed` | 시드 데이터 | 초기 데이터 삽입 |
| `docker:up` | Docker | 컨테이너 실행 |

---

## 3. API 엔드포인트 추출

### 3.1 Express / Fastify 라우트 추출

```regex
# Express 라우트
(app|router)\.(get|post|put|patch|delete|options|head|all)\s*\(\s*['"`]([^'"`]+)['"`]
# Fastify 라우트
(fastify|server)\.(get|post|put|patch|delete)\s*\(\s*['"`]([^'"`]+)['"`]
```

**추출 정보**:
- HTTP 메서드: 정규식의 캡처 그룹 2
- 경로: 캡처 그룹 3
- 미들웨어: 라우트 핸들러 앞의 함수 참조 (인증 등)
- 핸들러 함수: 마지막 인자

### 3.2 NestJS 컨트롤러 추출

```regex
# 컨트롤러 데코레이터
@Controller\s*\(\s*['"`]([^'"`]*)['"`]\s*\)
# HTTP 메서드 데코레이터
@(Get|Post|Put|Patch|Delete)\s*\(\s*['"`]?([^'"`\)]*?)['"`]?\s*\)
# 가드/인증 데코레이터
@(UseGuards|Auth)\s*\(
```

### 3.3 Django / DRF 뷰 추출

```regex
# URL 패턴
path\s*\(\s*['"`]([^'"`]+)['"`]
# ViewSet
class\s+(\w+)ViewSet\s*\(
# api_view 데코레이터
@api_view\s*\(\s*\[([^\]]+)\]
# action 데코레이터
@action\s*\(.*methods\s*=\s*\[([^\]]+)\]
```

### 3.4 Flask / FastAPI 라우트 추출

```regex
# Flask 라우트
@(app|blueprint)\.(route|get|post|put|patch|delete)\s*\(\s*['"`]([^'"`]+)['"`]
# FastAPI 라우트
@(app|router)\.(get|post|put|patch|delete)\s*\(\s*['"`]([^'"`]+)['"`]
```

### 3.5 Spring Boot 컨트롤러 추출

```regex
# 컨트롤러 매핑
@(Request|Get|Post|Put|Patch|Delete)Mapping\s*\(\s*(?:value\s*=\s*)?['"`]?([^'"`\)]+?)['"`]?\s*\)
# RestController
@RestController
@RequestMapping\s*\(\s*['"`]([^'"`]+)['"`]\s*\)
```

### 3.6 엔드포인트 정보 수집

각 엔드포인트에서 추출하는 정보:

| 정보 | 추출 소스 | 필수 여부 |
|------|-----------|-----------|
| HTTP 메서드 | 라우트 데코레이터/메서드 호출 | 필수 |
| 경로 | 라우트 문자열 | 필수 |
| 설명 | 함수 이름 + JSDoc/docstring | 선택 |
| 요청 바디 | 타입 정의, DTO 클래스, Pydantic 모델 | 선택 |
| 응답 타입 | 반환 타입, 직렬화 클래스 | 선택 |
| 인증 필요 | 인증 미들웨어/데코레이터 존재 | 선택 |
| 경로 파라미터 | URL의 `:param` 또는 `{param}` | 선택 |
| 쿼리 파라미터 | @Query 데코레이터, query_params 등 | 선택 |

---

## 4. CHANGELOG 생성 로직

### 4.1 커밋 파싱 플로우

```
1. git tag 목록 조회 (semver 정렬)
   └─ 태그 없으면 전체 커밋 → [Unreleased] 섹션

2. 태그 간 커밋 그룹화
   ├─ v1.2.0..v1.3.0 → [1.3.0] 섹션
   ├─ v1.1.0..v1.2.0 → [1.2.0] 섹션
   └─ 최신 태그..HEAD → [Unreleased] 섹션

3. 각 커밋 메시지 파싱
   ├─ Conventional Commits 형식?
   │   ├─ 예 → 타입별 자동 분류
   │   └─ 아니오 → 키워드 기반 분류
   └─ BREAKING CHANGE 마커 확인

4. 섹션별 정렬 및 출력
```

### 4.2 Conventional Commits 정규식

```regex
# Conventional Commit 파싱
^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(([^)]+)\))?(!)?:\s+(.+)$
# 캡처 그룹:
# 1: type (feat, fix, etc.)
# 3: scope (선택)
# 4: breaking change marker (!)
# 5: description
```

### 4.3 비표준 커밋 키워드 매핑

| 키워드 패턴 | CHANGELOG 섹션 |
|------------|----------------|
| `add`, `added`, `new`, `create`, `implement` | Added |
| `fix`, `fixed`, `resolve`, `bug`, `patch` | Fixed |
| `update`, `change`, `modify`, `improve`, `enhance` | Changed |
| `remove`, `delete`, `drop`, `deprecate` | Removed |
| `security`, `vulnerability`, `cve` | Security |
| `performance`, `optimize`, `speed` | Performance |
| 기타 | Other Changes |

### 4.4 Breaking Change 감지

다음 패턴을 Breaking Change로 감지합니다:

```regex
# 커밋 메시지 내 BREAKING CHANGE
BREAKING CHANGE:\s*(.+)
BREAKING-CHANGE:\s*(.+)
# 타입 뒤 ! 마커
^(feat|fix|refactor)(\([^)]+\))?!:
```

---

## 5. JSDoc/Docstring 분석

### 5.1 미문서화 함수 탐지

#### TypeScript/JavaScript

```regex
# export된 함수 선언
^export\s+(async\s+)?function\s+(\w+)
# export된 화살표 함수
^export\s+const\s+(\w+)\s*=\s*(async\s+)?\(
# export된 클래스
^export\s+(abstract\s+)?class\s+(\w+)
# 클래스 메서드 (public)
^\s+(public\s+)?(async\s+)?(\w+)\s*\(
```

문서화 여부: 해당 선언 바로 위에 `/** */` JSDoc 블록 존재 여부 확인

#### Python

```regex
# 함수 정의 (private 제외)
^def\s+([^_]\w+)\s*\(
# 클래스 정의
^class\s+(\w+)
# 메서드 (private 제외)
^\s+def\s+([^_]\w+)\s*\(self
```

문서화 여부: 함수/클래스 선언 다음 줄에 `"""docstring"""` 존재 여부 확인

### 5.2 타입 정보 추론

문서 생성 시 코드에서 타입 정보를 추론합니다:

| 소스 | 추론 대상 | 방법 |
|------|-----------|------|
| TypeScript 타입 어노테이션 | 매개변수 타입, 반환 타입 | 직접 추출 |
| Python 타입 힌트 | 매개변수 타입, 반환 타입 | `def f(x: int) -> str` 파싱 |
| Java 타입 선언 | 매개변수 타입, 반환 타입 | 메서드 시그니처 파싱 |
| JSDoc 타입 (기존) | 이미 문서화된 타입 | 보존 |
| 기본값 | 매개변수 기본값 | `param = "default"` 추출 |

### 5.3 설명 생성 휴리스틱

함수/클래스의 설명을 자동 생성할 때 사용하는 휴리스틱:

1. **함수명 분석**: camelCase/snake_case를 분리하여 자연어 설명 생성
   - `getUserById` → "ID로 사용자를 조회합니다"
   - `create_order` → "주문을 생성합니다"
2. **반환 타입 분석**: `Promise<User>` → "사용자 정보를 비동기로 반환합니다"
3. **매개변수 분석**: 매개변수명 + 타입으로 설명 추론
4. **함수 본문 분석**: throw/raise 패턴으로 @throws 추론, HTTP 호출 패턴으로 외부 의존성 표시

---

## 6. 분석 품질 체크리스트

문서 생성 후 다음 항목을 검증합니다:

### README.md

- [ ] 프로젝트 이름이 정확한가?
- [ ] 기술 스택 목록이 실제 의존성과 일치하는가?
- [ ] 설치 명령이 실행 가능한가? (패키지 매니저 일치)
- [ ] 실행 명령이 package.json scripts와 일치하는가?
- [ ] 환경 변수 목록이 .env.example과 일치하는가?
- [ ] 디렉토리 구조가 실제 구조와 일치하는가?
- [ ] 라이선스 정보가 LICENSE 파일과 일치하는가?

### API 문서

- [ ] 모든 라우트 파일의 엔드포인트가 포함되었는가?
- [ ] HTTP 메서드가 정확한가?
- [ ] 경로 파라미터가 누락 없이 문서화되었는가?
- [ ] 인증 필요 여부가 표시되었는가?

### CHANGELOG

- [ ] 모든 태그 간 커밋이 포함되었는가?
- [ ] Breaking Change가 누락 없이 표시되었는가?
- [ ] 날짜 형식이 ISO 8601 (YYYY-MM-DD)인가?
- [ ] Keep a Changelog 형식을 따르는가?

### JSDoc/Docstring

- [ ] 모든 public 함수/클래스가 문서화되었는가?
- [ ] 매개변수 타입이 정확한가?
- [ ] 반환 타입이 정확한가?
- [ ] 예외(throws) 정보가 정확한가?
- [ ] 기존 문서가 보존되었는가?
