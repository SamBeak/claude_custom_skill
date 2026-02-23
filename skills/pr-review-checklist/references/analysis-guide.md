# PR Review Checklist 분석 가이드

변경사항 분석, 파일 분류, 체크리스트 항목 결정, 위험도 판단, PR 크기 분류에 대한 상세 방법론을 다룹니다.

## 목차

1. [분석 워크플로우 개요](#분석-워크플로우-개요)
2. [Phase 1: 변경사항 수집](#phase-1-변경사항-수집)
3. [Phase 2: 파일 분류](#phase-2-파일-분류)
4. [Phase 3: 체크리스트 항목 결정](#phase-3-체크리스트-항목-결정)
5. [Phase 4: 위험도 판단](#phase-4-위험도-판단)
6. [Phase 5: PR 크기 분류](#phase-5-pr-크기-분류)
7. [Phase 6: 누락 항목 점검](#phase-6-누락-항목-점검)
8. [분석 품질 체크리스트](#분석-품질-체크리스트)

## 분석 워크플로우 개요

PR 리뷰 체크리스트 생성은 6단계의 분석 과정을 거칩니다:

```
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  Phase 1      │   │  Phase 2      │   │  Phase 3      │
│  변경사항 수집  │──▶│  파일 분류     │──▶│  체크리스트    │
│  (git diff)   │   │  (카테고리화)   │   │  항목 결정     │
└───────────────┘   └───────────────┘   └───────────────┘
        │                                       │
        ▼                                       ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  Phase 6      │   │  Phase 5      │   │  Phase 4      │
│  누락 항목     │◀──│  PR 크기      │◀──│  위험도 판단   │
│  점검          │   │  분류          │   │               │
└───────────────┘   └───────────────┘   └───────────────┘
```

## Phase 1: 변경사항 수집

### 1.1 Git 명령어 실행

```bash
# 1. 현재 브랜치 확인
git branch --show-current

# 2. base branch 감지
git rev-parse --verify main 2>/dev/null && echo "main" || \
  (git rev-parse --verify master 2>/dev/null && echo "master" || echo "unknown")

# 3. 변경 파일 목록 (상태 포함)
git diff ${BASE}...HEAD --name-status

# 4. 변경 통계
git diff ${BASE}...HEAD --stat

# 5. 변경 줄 수 (숫자만)
git diff ${BASE}...HEAD --shortstat

# 6. 전체 diff 내용
git diff ${BASE}...HEAD

# 7. 커밋 이력
git log --oneline ${BASE}...HEAD
```

### 1.2 변경사항 파싱

`git diff --name-status` 출력을 파싱하여 각 파일의 상태를 추출합니다:

```
상태코드  파일경로
────────  ──────────────────────────────
A         src/services/newService.ts        → 신규 파일
M         src/controllers/userController.ts → 수정된 파일
D         src/utils/deprecated.ts           → 삭제된 파일
R100      old/path.ts  new/path.ts          → 이름 변경 (100% 유사도)
R085      old/file.ts  new/file.ts          → 이름 변경 + 수정 (85% 유사도)
```

### 1.3 변경 통계 수집

`git diff --shortstat` 출력에서 다음 정보를 추출합니다:

```
N files changed, N insertions(+), N deletions(-)
```

파일별 변경 줄 수는 `git diff --stat`에서 추출합니다.

## Phase 2: 파일 분류

### 2.1 분류 규칙 상세

각 파일을 경로 패턴과 확장자를 기준으로 카테고리에 매칭합니다. 매칭은 위에서 아래로 우선순위 순서대로 시도하며, 첫 번째로 매칭되는 카테고리에 분류합니다.

#### 카테고리 1: 테스트 변경 (최우선)

| 매칭 패턴 | 예시 |
|-----------|------|
| `**/tests/**` | `tests/unit/userService.test.ts` |
| `**/test/**` | `test/integration/api.test.js` |
| `**/__tests__/**` | `src/__tests__/utils.test.ts` |
| `*.test.*` | `userService.test.ts` |
| `*.spec.*` | `auth.spec.js` |
| `**/fixtures/**` | `tests/fixtures/userData.json` |
| `**/mocks/**` | `tests/mocks/apiMock.ts` |
| `jest.config.*` | `jest.config.ts` |
| `vitest.config.*` | `vitest.config.ts` |
| `cypress/**` | `cypress/e2e/login.cy.ts` |
| `playwright/**` | `playwright/tests/checkout.spec.ts` |
| `**/cypress/**` | `e2e/cypress/integration/flow.ts` |
| `**/*.stories.*` | `src/components/Button.stories.tsx` |
| `**/storybook/**` | `storybook/config.ts` |
| `pytest.ini` | `pytest.ini` |
| `conftest.py` | `tests/conftest.py` |
| `**/test_*.py` | `tests/test_views.py` |
| `**/*_test.go` | `handlers/user_test.go` |

#### 카테고리 2: DB 변경

| 매칭 패턴 | 예시 |
|-----------|------|
| `**/migrations/**` | `db/migrations/20240115_create_users.sql` |
| `**/migrate/**` | `src/migrate/001_init.ts` |
| `*.sql` | `scripts/seed.sql` |
| `schema.*` | `prisma/schema.prisma` |
| `**/prisma/**` | `prisma/schema.prisma` |
| `**/drizzle/**` | `drizzle/0001_init.ts` |
| `**/typeorm/**` | `typeorm/migrations/CreateUser.ts` |
| `**/sequelize/**` | `sequelize/migrations/20240115-create-user.js` |
| `**/knex/**` | `knex/migrations/20240115_init.js` |
| `**/alembic/**` | `alembic/versions/abc123_add_column.py` |
| `**/flyway/**` | `flyway/V1__init.sql` |
| `**/liquibase/**` | `liquibase/changelog/init.xml` |
| `**/*.entity.*` | `src/entities/user.entity.ts` (DB 카테고리 또는 비즈니스 로직, diff 내용에 따라 판단) |

#### 카테고리 3: 설정 변경

| 매칭 패턴 | 예시 |
|-----------|------|
| `*.config.*` | `webpack.config.js`, `next.config.ts` |
| `*.env*` | `.env.local`, `.env.production` |
| `.env*` | `.env`, `.env.example` |
| `docker-compose.*` | `docker-compose.yml` |
| `Dockerfile*` | `Dockerfile`, `Dockerfile.prod` |
| `nginx.*` | `nginx.conf` |
| `webpack.*` | `webpack.dev.js` |
| `vite.*` | `vite.config.ts` |
| `tsconfig.*` | `tsconfig.json`, `tsconfig.build.json` |
| `package.json` | `package.json` |
| `*.lock` | `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` |
| `requirements*.txt` | `requirements.txt`, `requirements-dev.txt` |
| `Pipfile*` | `Pipfile`, `Pipfile.lock` |
| `pyproject.toml` | `pyproject.toml` |
| `Gemfile*` | `Gemfile`, `Gemfile.lock` |
| `go.mod` / `go.sum` | `go.mod` |
| `Cargo.toml` / `Cargo.lock` | `Cargo.toml` |
| `.eslintrc*` | `.eslintrc.json` |
| `.prettierrc*` | `.prettierrc` |
| `babel.config.*` | `babel.config.js` |
| `.editorconfig` | `.editorconfig` |

#### 카테고리 4: API 변경

| 매칭 패턴 | 예시 |
|-----------|------|
| `**/controllers/**` | `src/controllers/userController.ts` |
| `**/routes/**` | `src/routes/apiRoutes.ts` |
| `**/handlers/**` | `src/handlers/webhookHandler.ts` |
| `**/endpoints/**` | `src/endpoints/payment.ts` |
| `**/api/**` | `src/api/v1/users.ts` |
| `**/resolvers/**` | `src/resolvers/userResolver.ts` |
| `*.controller.*` | `user.controller.ts` |
| `*.route.*` | `auth.route.ts` |
| `*.handler.*` | `webhook.handler.ts` |
| `*.resolver.*` | `user.resolver.ts` |
| `**/middleware/**` | `src/middleware/auth.ts` |
| `**/interceptors/**` | `src/interceptors/logging.ts` |
| `**/guards/**` | `src/guards/roleGuard.ts` |
| `**/views.py` (Django) | `apps/users/views.py` |
| `**/serializers.py` | `apps/users/serializers.py` |
| `**/urls.py` | `apps/users/urls.py` |

#### 카테고리 5: UI 변경

| 매칭 패턴 | 예시 |
|-----------|------|
| `**/components/**` | `src/components/Button.tsx` |
| `**/pages/**` | `src/pages/Dashboard.tsx` |
| `**/views/**` (프론트엔드 프레임워크 맥락) | `src/views/HomeView.vue` |
| `**/layouts/**` | `src/layouts/MainLayout.tsx` |
| `*.tsx` | `App.tsx` (컴포넌트 관련) |
| `*.jsx` | `Header.jsx` |
| `*.vue` | `UserList.vue` |
| `*.svelte` | `Counter.svelte` |
| `*.css` | `styles/main.css` |
| `*.scss` | `styles/variables.scss` |
| `*.less` | `theme.less` |
| `*.styled.*` | `Button.styled.ts` |
| `*.module.css` | `Dashboard.module.css` |
| `**/styles/**` | `src/styles/global.css` |
| `**/theme/**` | `src/theme/colors.ts` |
| `**/assets/**` | `src/assets/logo.svg` |
| `**/public/**` | `public/favicon.ico` |
| `**/templates/**` (HTML 템플릿) | `templates/index.html` |

**주의**: `*.tsx`/`*.jsx` 파일이라도 `controllers/`, `api/`, `routes/` 경로에 있으면 API 변경으로 분류합니다. 즉, 경로 패턴이 확장자 패턴보다 우선합니다.

#### 카테고리 6: 비즈니스 로직

| 매칭 패턴 | 예시 |
|-----------|------|
| `**/services/**` | `src/services/authService.ts` |
| `**/models/**` | `src/models/User.ts` |
| `**/entities/**` | `src/entities/Order.ts` |
| `**/domain/**` | `src/domain/payment/PaymentProcessor.ts` |
| `**/usecases/**` | `src/usecases/CreateOrder.ts` |
| `**/lib/**` | `src/lib/encryption.ts` |
| `**/utils/**` | `src/utils/dateFormatter.ts` |
| `**/helpers/**` | `src/helpers/validation.ts` |
| `**/repositories/**` | `src/repositories/userRepository.ts` |
| `**/managers/**` | `src/managers/cacheManager.ts` |
| `**/providers/**` | `src/providers/emailProvider.ts` |
| `**/adapters/**` | `src/adapters/paymentAdapter.ts` |

#### 카테고리 7: 문서 변경

| 매칭 패턴 | 예시 |
|-----------|------|
| `*.md` | `README.md` |
| `*.mdx` | `docs/guide.mdx` |
| `*.rst` | `docs/api.rst` |
| `**/docs/**` | `docs/setup.md` |
| `**/documentation/**` | `documentation/api.md` |
| `CHANGELOG*` | `CHANGELOG.md` |
| `LICENSE*` | `LICENSE` |
| `CONTRIBUTING*` | `CONTRIBUTING.md` |
| `*.txt` (문서성) | `NOTICE.txt` |

#### 카테고리 8: 인프라/CI

| 매칭 패턴 | 예시 |
|-----------|------|
| `.github/**` | `.github/workflows/ci.yml` |
| `.gitlab-ci.*` | `.gitlab-ci.yml` |
| `Jenkinsfile*` | `Jenkinsfile` |
| `**/terraform/**` | `terraform/main.tf` |
| `**/k8s/**` | `k8s/deployment.yaml` |
| `**/kubernetes/**` | `kubernetes/service.yaml` |
| `**/helm/**` | `helm/values.yaml` |
| `.circleci/**` | `.circleci/config.yml` |
| `**/ansible/**` | `ansible/playbook.yml` |
| `**/cloudformation/**` | `cloudformation/stack.yaml` |
| `Makefile` | `Makefile` |
| `Procfile` | `Procfile` |
| `*.tf` | `main.tf` |
| `*.hcl` | `terragrunt.hcl` |

### 2.2 분류 우선순위 결정

한 파일이 여러 카테고리에 매칭될 수 있는 경우, 다음 우선순위로 단일 카테고리를 결정합니다:

```
1. 테스트 변경     (최우선: *.test.*, *.spec.*, tests/** 등)
2. DB 변경         (migrations/**, *.sql, schema.*)
3. 설정 변경       (*.config.*, *.env*, package.json 등)
4. API 변경        (controllers/**, routes/**, api/**)
5. UI 변경         (components/**, pages/**, *.tsx, *.css 등)
6. 비즈니스 로직    (services/**, models/**, utils/**)
7. 문서 변경       (*.md, docs/**)
8. 인프라/CI       (.github/**, terraform/**)
```

**예시 (우선순위 적용):**

| 파일 | 매칭되는 카테고리 | 최종 분류 | 이유 |
|------|------------------|----------|------|
| `tests/api/controllers/auth.test.ts` | 테스트, API | **테스트** | 테스트가 최우선 |
| `prisma/migrations/001.sql` | DB, 설정 | **DB** | DB가 설정보다 우선 |
| `src/api/config.ts` | API, 설정 | **설정** | 설정이 API보다 우선 |
| `src/components/Button.test.tsx` | 테스트, UI | **테스트** | 테스트가 최우선 |
| `src/services/auth.ts` | 비즈니스 로직 | **비즈니스 로직** | 단일 매칭 |

### 2.3 분류 결과 출력 형식

```
[분류 결과]

[DB 변경] (2개 파일)
  A  prisma/migrations/20240115_add_role.sql
  M  prisma/schema.prisma

[API 변경] (3개 파일)
  M  src/controllers/userController.ts
  A  src/routes/adminRoutes.ts
  M  src/middleware/auth.ts

[UI 변경] (2개 파일)
  M  src/components/UserProfile.tsx
  A  src/pages/AdminDashboard.tsx

[테스트 변경] (1개 파일)
  A  tests/controllers/userController.test.ts

[설정 변경] (1개 파일)
  M  .env.example
```

## Phase 3: 체크리스트 항목 결정

### 3.1 기본 원칙

- **해당 카테고리에 변경이 있을 때만** 해당 체크리스트를 포함합니다
- 체크리스트 항목은 **변경 내용의 맥락을 반영**하여 조정할 수 있습니다
- 기본 체크리스트에 없는 항목도 diff 분석을 통해 **추가할 수 있습니다**

### 3.2 체크리스트 항목 조정 로직

#### diff 내용 기반 추가 항목

diff 내용을 분석하여 다음과 같은 추가 체크리스트 항목을 동적으로 생성합니다:

| diff에서 감지되는 패턴 | 추가 체크리스트 항목 |
|----------------------|-------------------|
| `password`, `secret`, `token`, `api_key` | 시크릿이 하드코딩되지 않았는지 확인 |
| `eval(`, `exec(`, `dangerouslySetInnerHTML` | 코드 인젝션 위험 확인 |
| `console.log`, `print(`, `debugger` | 디버그 코드가 제거되었는지 확인 |
| `TODO`, `FIXME`, `HACK`, `XXX` | 임시 코드가 정리되었는지 확인 |
| `sleep(`, `setTimeout(` (비즈니스 로직 내) | 하드코딩된 대기 시간이 적절한지 확인 |
| `catch {}`, `catch (e) {}` (빈 catch) | 에러가 적절히 처리되는지 확인 |
| `any` (TypeScript) | 타입 안전성이 유지되는지 확인 |
| `!important` (CSS) | CSS 우선순위 남용이 없는지 확인 |
| `SELECT *` | 필요한 컬럼만 조회하는지 확인 |
| `.innerHTML =` | XSS 위험이 없는지 확인 |

#### 파일 규모 기반 항목 조정

| 파일 변경 규모 | 조정 |
|---------------|------|
| 단일 파일, 10줄 미만 변경 | 핵심 체크리스트만 포함 (간소화) |
| 50줄 이상 변경 | 전체 체크리스트 포함 |
| 200줄 이상 변경 | 전체 체크리스트 + 성능/복잡도 관련 항목 추가 |
| 500줄 이상 변경 | 전체 체크리스트 + PR 분할 권고 추가 |

### 3.3 복합 카테고리 처리

여러 카테고리의 변경이 동시에 존재할 때의 추가 체크리스트:

| 복합 카테고리 | 추가 확인 항목 |
|-------------|--------------|
| DB 변경 + API 변경 | API 응답이 새로운 스키마를 올바르게 반영하는가? |
| API 변경 + UI 변경 | 프론트엔드가 새로운 API 응답 형식을 처리하는가? |
| DB 변경 + 비즈니스 로직 | 마이그레이션과 코드 변경의 배포 순서가 올바른가? |
| 설정 변경 + 인프라/CI | CI 파이프라인이 새로운 설정을 올바르게 사용하는가? |
| API 변경 + 테스트 변경 | 새로운/변경된 API에 대한 테스트가 충분한가? |

## Phase 4: 위험도 판단

### 4.1 위험도 기준

변경 내용의 위험도를 3단계로 분류합니다.

#### 고위험 (High Risk)

다음 중 하나라도 해당하면 고위험으로 판단합니다:

```
[고위험 패턴]

1. DB 스키마 변경
   - 컬럼 삭제 (DROP COLUMN)
   - 컬럼 타입 변경 (ALTER COLUMN ... TYPE)
   - 제약조건 변경 (DROP CONSTRAINT, ALTER CONSTRAINT)
   - 테이블 삭제 (DROP TABLE)
   - 인덱스 삭제 (DROP INDEX)

2. 인증/인가 코드 변경
   - 로그인/로그아웃 로직
   - 세션 관리
   - 토큰 생성/검증
   - 비밀번호 해싱
   - OAuth/SSO 연동
   - 권한 검사 미들웨어

3. 결제/금융 로직 변경
   - 결제 처리 (charge, payment, billing)
   - 정산 로직 (settlement, payout)
   - 환불 처리 (refund)
   - 금액 계산 (amount, price, fee)

4. 보안 설정 변경
   - CORS 설정
   - CSP (Content Security Policy)
   - 암호화/복호화 로직
   - 시크릿/API 키 관련 코드
   - 접근 제어 목록 (ACL)

5. 삭제 작업
   - 5개 이상 파일 삭제
   - public API 제거
   - 기능 폐기 (deprecation)
```

#### 중위험 (Medium Risk)

```
[중위험 패턴]

1. API 응답 형식 변경
   - 필드 추가/제거/이름변경
   - 상태 코드 변경
   - 에러 응답 형식 변경

2. 공통 유틸리티/라이브러리 변경
   - 여러 모듈에서 import되는 코드 변경
   - 공통 타입 정의 변경
   - 공유 컴포넌트 변경

3. 의존성 업데이트
   - major 버전 업데이트
   - 새로운 의존성 추가
   - 의존성 제거

4. 환경 변수 변경
   - 새 환경 변수 추가
   - 기존 환경 변수 이름 변경
   - 기본값 변경

5. DB 인덱스 변경
   - 새 인덱스 추가 (성능 영향)
   - 인덱스 변경 (쿼리 플랜 영향)
```

#### 저위험 (Low Risk)

```
[저위험 패턴]

1. 문서만 변경 (소스코드 변경 없음)
2. 테스트만 변경 (소스코드 변경 없음)
3. 코드 스타일/포맷팅만 변경
4. 주석만 추가/수정
5. 변수/함수 이름 변경 (로직 변경 없음)
6. 타입 정의만 변경 (런타임 영향 없음)
7. 로그 메시지만 변경
```

### 4.2 위험도 표시 형식

PR description에 위험도를 표시합니다:

```markdown
## Risk Level

위험도: 고위험 (High)

고위험 사유:
- DB 스키마 변경: users 테이블에 컬럼 타입 변경 포함
- 인증 로직 변경: JWT 토큰 검증 로직 수정

특별 리뷰 요청:
- DB 마이그레이션 롤백 시나리오를 반드시 확인해 주세요
- 인증 관련 변경은 보안 팀 리뷰를 권장합니다
```

### 4.3 위험도별 추천 행동

| 위험도 | 추천 행동 |
|--------|----------|
| **고위험** | 2인 이상 리뷰 필수, staging 환경 테스트, 롤백 계획 수립 |
| **중위험** | 1인 이상 리뷰, 관련 기능 수동 테스트 |
| **저위험** | 셀프 리뷰 후 머지 가능, CI 통과 확인 |

## Phase 5: PR 크기 분류

### 5.1 크기 기준

`git diff --shortstat`에서 추출한 총 변경 줄 수(insertions + deletions)를 기준으로 분류합니다:

| 크기 | 변경 줄 수 | 예상 리뷰 시간 | 설명 |
|------|-----------|--------------|------|
| **S** (Small) | 1-50줄 | 10분 이내 | 단순 수정, 버그 픽스, 타이포 수정 |
| **M** (Medium) | 51-200줄 | 30분 이내 | 일반적인 기능 추가, 리팩토링 |
| **L** (Large) | 201-500줄 | 1시간 이내 | 큰 기능, 대규모 리팩토링 |
| **XL** (Extra Large) | 501줄 이상 | 1시간 이상 | 대규모 변경, 분할 필요 |

### 5.2 크기 보정

다음 항목은 변경 줄 수에서 제외하거나 가중치를 조정합니다:

| 항목 | 보정 방법 |
|------|----------|
| 자동 생성 파일 (`*.lock`, `*.generated.*`) | 줄 수에서 제외 |
| 테스트 코드 | 0.5배 가중치 (테스트는 양이 많아도 리뷰 부담 적음) |
| 마이그레이션 SQL | 자동 생성 여부에 따라 제외 또는 포함 |
| 단순 이동/이름 변경 (`R100`) | 줄 수에서 제외 |
| 스냅샷 파일 (`*.snap`) | 줄 수에서 제외 |
| 미디어/바이너리 파일 | 줄 수에서 제외 (파일 수만 계산) |

### 5.3 XL PR 경고 및 분할 제안

XL PR이 감지되면 다음 경고를 표시합니다:

```
경고: PR 크기가 매우 큽니다 (XL: {N}줄 변경, {M}개 파일).

대규모 PR은 다음과 같은 문제가 있습니다:
- 리뷰어가 집중력을 유지하기 어렵습니다
- 버그 발견 확률이 낮아집니다
- 머지 충돌 가능성이 높아집니다
- 롤백이 어렵습니다

분할 제안:
1. DB 마이그레이션을 별도 PR로 분리
2. API 변경과 UI 변경을 분리
3. 리팩토링과 기능 추가를 분리
4. 테스트 추가를 별도 PR로 분리
```

## Phase 6: 누락 항목 점검

### 6.1 테스트 파일 누락 점검

소스 코드가 변경된 파일 목록에서 테스트 카테고리를 제외한 파일들에 대해, 대응하는 테스트 파일이 변경 목록에 있는지 확인합니다.

#### 테스트 파일 매칭 규칙

소스 파일 경로를 테스트 파일 경로로 변환하여 매칭합니다:

| 소스 파일 패턴 | 예상 테스트 파일 위치 |
|--------------|-------------------|
| `src/services/user.ts` | `tests/services/user.test.ts` 또는 `src/services/user.test.ts` 또는 `src/services/__tests__/user.ts` |
| `src/controllers/auth.ts` | `tests/controllers/auth.test.ts` 또는 `src/controllers/auth.test.ts` |
| `app/models/order.py` | `tests/models/test_order.py` 또는 `tests/test_models.py` |
| `handlers/user.go` | `handlers/user_test.go` |
| `lib/utils.rb` | `spec/lib/utils_spec.rb` 또는 `test/lib/utils_test.rb` |

#### 테스트 점검 제외 대상

다음 파일은 테스트 누락 점검에서 제외합니다:

- 설정 파일 (`*.config.*`, `*.env*`, `package.json` 등)
- 문서 파일 (`*.md`, `*.rst` 등)
- 인프라/CI 파일 (`.github/**`, `Dockerfile` 등)
- 타입 정의 파일 (`*.d.ts`, `*.pyi` 등)
- 자동 생성 파일 (`*.generated.*`, `*.lock` 등)
- 스타일 파일 (`*.css`, `*.scss` 등)
- 에셋 파일 (이미지, 폰트 등)

### 6.2 문서 갱신 누락 점검

다음 조건에서 문서 갱신 누락을 경고합니다:

| 변경 유형 | 문서 갱신 필요 조건 |
|----------|-----------------|
| API 엔드포인트 추가/변경 | `*.md`, `**/docs/**`, swagger/openapi 파일 변경 없음 |
| 환경 변수 추가 | `.env.example` 또는 배포 가이드 변경 없음 |
| 의존성 추가 | `README.md`의 설치 가이드 변경 없음 |
| public API 변경 | API 문서 변경 없음 |
| 설정 구조 변경 | 설정 가이드 변경 없음 |

### 6.3 마이그레이션 누락 점검

모델/엔티티 파일이 변경되었지만 마이그레이션 파일이 변경 목록에 없는 경우 경고합니다.

| 모델 파일 패턴 | 대응 마이그레이션 위치 |
|--------------|-------------------|
| `**/models/**` | `**/migrations/**` |
| `**/entities/**` | `**/migrations/**` |
| `prisma/schema.prisma` | `prisma/migrations/**` |
| `**/models.py` (Django) | `**/migrations/**` |

**주의**: 모든 모델 변경이 마이그레이션을 필요로 하지는 않습니다 (예: 메서드만 추가). 경고는 "확인해 주세요" 수준으로 제공합니다.

## 분석 품질 체크리스트

분석 결과를 PR description에 반영하기 전에 확인합니다:

- [ ] 모든 변경 파일이 하나의 카테고리에 분류되었는가
- [ ] 분류 우선순위가 올바르게 적용되었는가
- [ ] 해당 카테고리에 대한 체크리스트만 포함되었는가 (불필요한 체크리스트 없음)
- [ ] diff 내용 분석을 통한 추가 항목이 반영되었는가
- [ ] 위험도 판단이 변경 내용에 부합하는가
- [ ] PR 크기 분류가 정확한가 (보정 규칙 적용)
- [ ] 테스트 누락 점검이 올바르게 수행되었는가 (제외 대상 확인)
- [ ] 문서 갱신 누락 점검이 적절한가
- [ ] 마이그레이션 누락 점검이 적절한가 (거짓 양성 최소화)
- [ ] 경고 메시지가 한국어로 작성되었는가
- [ ] 체크리스트 항목이 실질적으로 유용한가 (형식적 항목 아님)
