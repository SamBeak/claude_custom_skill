---
name: pr-review-checklist
description: PR 생성 시 git diff를 분석하여 변경사항 종류별 맞춤 리뷰 체크리스트를 자동 생성하고, PR description을 작성하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) PR 만들어줘, (2) 풀 리퀘스트 생성, (3) 리뷰 체크리스트 만들어줘, (4) PR description 작성, (5) 변경사항 분석해줘, (6) 코드 리뷰 준비해줘, (7) PR 올려줘.
---

# PR Review Checklist Generator

`git diff`를 분석하여 변경 파일을 자동 분류하고, 변경 종류별 맞춤 리뷰 체크리스트를 생성한 후, PR description을 작성하여 GitHub PR을 생성하는 스킬입니다.

## Quick Start

사용자가 PR 생성 또는 리뷰 체크리스트를 요청하면 다음 순서로 진행합니다:

1. **Base branch 확인**:
   ```bash
   # 현재 브랜치와 base branch 확인
   git branch --show-current
   git log --oneline main...HEAD
   ```

2. **변경사항 분석**:
   ```bash
   # base branch 기준 diff 분석
   git diff main...HEAD --stat
   git diff main...HEAD --name-status
   git diff main...HEAD
   ```

3. **변경 파일 분류**: [파일 분류 규칙](#파일-분류-규칙)에 따라 카테고리 분류

4. **맞춤 체크리스트 생성**: [체크리스트 생성 로직](#체크리스트-생성-로직)에 따라 카테고리별 체크리스트 생성

5. **누락 항목 경고**: 테스트 파일 누락, 문서 미갱신 등 경고

6. **PR description 초안 작성**: [PR Description 형식](#pr-description-형식)에 따라 작성

7. **사용자 확인 후 PR 생성**:
   ```bash
   gh pr create --title "{제목}" --body "{PR description}" --base main
   ```

## 트리거 조건

다음과 같은 사용자 요청 시 이 스킬을 활성화합니다:

- "PR 만들어줘" / "PR 생성해줘"
- "풀 리퀘스트 만들어줘"
- "리뷰 체크리스트 만들어줘" / "체크리스트 생성해줘"
- "PR description 작성해줘"
- "변경사항 분석해줘"
- "코드 리뷰 준비해줘"
- "PR 올려줘"
- "create PR" / "make a pull request"

## 파일 분류 규칙

변경된 파일을 경로 패턴과 확장자를 기반으로 다음 카테고리로 분류합니다.

### 카테고리 정의

| 카테고리 | 경로/파일 패턴 | 설명 |
|----------|---------------|------|
| **DB 변경** | `**/migrations/**`, `**/migrate/**`, `*.sql`, `schema.*`, `**/prisma/**`, `**/drizzle/**`, `**/typeorm/**`, `**/sequelize/**` | 데이터베이스 스키마, 마이그레이션 |
| **API 변경** | `**/controllers/**`, `**/routes/**`, `**/handlers/**`, `**/endpoints/**`, `**/api/**`, `**/resolvers/**`, `*.controller.*`, `*.route.*`, `*.handler.*` | API 엔드포인트, 컨트롤러 |
| **UI 변경** | `**/components/**`, `**/pages/**`, `**/views/**`, `**/layouts/**`, `*.tsx`, `*.jsx`, `*.vue`, `*.svelte`, `*.css`, `*.scss`, `*.less`, `*.styled.*` | 프론트엔드 컴포넌트, 페이지, 스타일 |
| **설정 변경** | `*.config.*`, `*.env*`, `.env*`, `docker-compose.*`, `Dockerfile*`, `*.yaml`, `*.yml`, `*.toml`, `nginx.*`, `webpack.*`, `vite.*`, `tsconfig.*`, `package.json` | 설정 파일, 환경 변수, 인프라 |
| **테스트 변경** | `**/tests/**`, `**/test/**`, `**/__tests__/**`, `*.test.*`, `*.spec.*`, `**/fixtures/**`, `**/mocks/**`, `jest.*`, `vitest.*`, `cypress/**`, `playwright/**` | 테스트 코드, 테스트 설정 |
| **문서 변경** | `*.md`, `*.mdx`, `*.rst`, `**/docs/**`, `**/documentation/**`, `CHANGELOG*`, `LICENSE*` | 문서, 가이드 |
| **비즈니스 로직** | `**/services/**`, `**/models/**`, `**/entities/**`, `**/domain/**`, `**/usecases/**`, `**/lib/**`, `**/utils/**`, `**/helpers/**` | 핵심 비즈니스 로직 |
| **인프라/CI** | `.github/**`, `.gitlab-ci.*`, `Jenkinsfile`, `**/terraform/**`, `**/k8s/**`, `**/helm/**`, `.circleci/**` | CI/CD, 인프라 코드 |

### 변경 상태 분류

`git diff --name-status` 출력 기준:

| 상태 코드 | 의미 | 표기 |
|-----------|------|------|
| `A` | 신규 파일 추가 | Added |
| `M` | 기존 파일 수정 | Modified |
| `D` | 파일 삭제 | Deleted |
| `R` | 파일 이름 변경 | Renamed |
| `C` | 파일 복사 | Copied |

### 파일 분류 우선순위

하나의 파일이 여러 카테고리에 해당할 수 있는 경우, 다음 우선순위를 따릅니다:

1. **테스트 변경** (테스트 파일은 항상 테스트로 분류)
2. **DB 변경** (마이그레이션 파일은 항상 DB로 분류)
3. **설정 변경** (config/env 파일은 항상 설정으로 분류)
4. **API 변경**
5. **UI 변경**
6. **비즈니스 로직**
7. **문서 변경**
8. **인프라/CI**

## 체크리스트 생성 로직

각 카테고리에 해당하는 변경이 감지되면 해당 카테고리의 체크리스트 항목을 생성합니다.

### DB 변경 체크리스트

DB 변경(마이그레이션 파일, 스키마 변경 등)이 감지된 경우:

```markdown
#### DB / 마이그레이션 체크리스트
- [ ] 롤백(down) 마이그레이션이 작성되어 있는가?
- [ ] 롤백 실행 시 데이터 손실이 없는가?
- [ ] 기존 데이터와의 호환성이 보장되는가? (데이터 무결성)
- [ ] 필요한 인덱스가 추가되었는가?
- [ ] 대용량 테이블 변경 시 lock 영향을 고려했는가?
- [ ] NOT NULL 컬럼 추가 시 기본값이 설정되어 있는가?
- [ ] 외래키 제약조건이 올바르게 설정되었는가?
- [ ] 마이그레이션 실행 순서가 올바른가?
- [ ] staging/production 환경에서 테스트 되었는가?
```

### API 변경 체크리스트

API 엔드포인트, 컨트롤러 등이 변경된 경우:

```markdown
#### API 변경 체크리스트
- [ ] 하위 호환성(backward compatibility)이 유지되는가?
- [ ] API 버전 관리가 적절한가? (v1, v2 등)
- [ ] 인증(authentication) 처리가 적용되어 있는가?
- [ ] 인가(authorization) / 권한 검사가 구현되어 있는가?
- [ ] 요청(request) 입력값 검증(validation)이 있는가?
- [ ] 응답(response) 형식이 일관적인가?
- [ ] 에러 응답 형식이 표준을 따르는가?
- [ ] Rate limiting이 적용되어 있는가?
- [ ] API 문서(Swagger/OpenAPI 등)가 갱신되었는가?
- [ ] CORS 설정이 올바른가?
```

### UI 변경 체크리스트

프론트엔드 컴포넌트, 페이지 등이 변경된 경우:

```markdown
#### UI 변경 체크리스트
- [ ] 웹 접근성(a11y)이 준수되는가? (aria 속성, 키보드 내비게이션, 스크린리더 등)
- [ ] 반응형 디자인이 적용되어 있는가? (모바일, 태블릿, 데스크탑)
- [ ] 크로스 브라우저 호환성이 확인되었는가? (Chrome, Firefox, Safari, Edge)
- [ ] 로딩 상태(loading state)가 처리되어 있는가?
- [ ] 에러 상태(error state)가 처리되어 있는가?
- [ ] 빈 상태(empty state)가 처리되어 있는가?
- [ ] 국제화(i18n) / 다국어 처리가 적용되어 있는가?
- [ ] XSS 방지를 위한 입력값 이스케이프가 되어 있는가?
- [ ] 이미지 최적화(lazy loading, 적절한 포맷)가 되어 있는가?
- [ ] 성능 (불필요한 리렌더링, 메모이제이션 등)이 고려되었는가?
```

### 설정 변경 체크리스트

설정 파일, 환경 변수 등이 변경된 경우:

```markdown
#### 설정 변경 체크리스트
- [ ] 환경별(dev/staging/production) 분리가 되어 있는가?
- [ ] 시크릿/민감 정보가 코드에 하드코딩되지 않았는가?
- [ ] .env 파일이 .gitignore에 포함되어 있는가?
- [ ] 환경 변수 변경 시 배포 문서가 갱신되었는가?
- [ ] 기본값(fallback)이 적절하게 설정되어 있는가?
- [ ] Docker/인프라 설정 변경 시 영향 범위가 확인되었는가?
- [ ] 설정값의 타입과 범위가 검증되는가?
- [ ] 팀원에게 새로운 환경 변수 추가가 공유되었는가?
```

### 테스트 변경 체크리스트

테스트 코드가 변경된 경우:

```markdown
#### 테스트 변경 체크리스트
- [ ] 테스트 커버리지가 유지 또는 향상되었는가?
- [ ] 엣지 케이스(경계값, null, 빈값 등)가 테스트되었는가?
- [ ] 모킹(mocking) 전략이 적절한가? (과도한 모킹 아닌지)
- [ ] 테스트가 독립적으로 실행 가능한가? (테스트 간 의존성 없음)
- [ ] 테스트 데이터가 적절하게 관리되는가? (fixture, factory 등)
- [ ] 비동기 테스트가 올바르게 처리되는가? (타임아웃, await 등)
- [ ] CI 환경에서 안정적으로 통과하는가? (flaky test 아닌지)
- [ ] 에러/실패 시나리오가 테스트되었는가?
```

### 비즈니스 로직 변경 체크리스트

핵심 비즈니스 로직이 변경된 경우:

```markdown
#### 비즈니스 로직 변경 체크리스트
- [ ] 기존 기능에 대한 회귀(regression)가 없는가?
- [ ] 입력값 검증(validation)이 적절한가?
- [ ] 에러 처리가 충분한가? (try-catch, 에러 전파 등)
- [ ] 로깅이 적절하게 추가되었는가?
- [ ] 동시성(concurrency) 이슈가 고려되었는가?
- [ ] 트랜잭션 처리가 올바른가? (필요한 경우)
- [ ] 메모리 누수 가능성이 없는가?
- [ ] 외부 의존성 장애 시 graceful degradation이 되는가?
```

### 인프라/CI 변경 체크리스트

CI/CD 및 인프라 코드가 변경된 경우:

```markdown
#### 인프라/CI 변경 체크리스트
- [ ] CI 파이프라인이 정상적으로 동작하는가?
- [ ] 배포 스크립트 변경 시 롤백 절차가 확인되었는가?
- [ ] 인프라 변경 시 비용 영향이 검토되었는가?
- [ ] 보안 그룹/네트워크 규칙이 올바른가?
- [ ] 환경 변수/시크릿 관리가 적절한가?
- [ ] 모니터링/알림 설정이 포함되어 있는가?
```

## 누락 항목 경고

변경사항을 분석한 후 다음 항목이 누락되었는지 확인하고 경고합니다.

### 테스트 파일 누락 경고

소스 코드가 변경되었지만 대응하는 테스트 파일이 없는 경우:

```
경고: 다음 파일에 대응하는 테스트 파일이 변경 또는 추가되지 않았습니다.
  - src/services/userService.ts → tests/services/userService.test.ts 없음
  - src/controllers/authController.ts → tests/controllers/authController.test.ts 없음

테스트 추가를 고려해 주세요.
```

**테스트 파일 매칭 규칙:**
- `src/**/*.ts` → `tests/**/*.test.ts` 또는 `src/**/*.test.ts` 또는 `src/**/__tests__/*.ts`
- `src/**/*.js` → `tests/**/*.test.js` 또는 `src/**/*.test.js`
- `app/**/*.py` → `tests/**/*test*.py` 또는 `tests/test_*.py`

### 문서 갱신 누락 경고

API 변경이나 설정 변경이 있지만 문서 파일 변경이 없는 경우:

```
경고: API 또는 설정이 변경되었지만 관련 문서가 갱신되지 않았습니다.
  - API 엔드포인트가 변경되었습니다. API 문서 갱신이 필요할 수 있습니다.
  - 환경 변수가 추가되었습니다. 배포 가이드 갱신이 필요할 수 있습니다.
```

### 마이그레이션 누락 경고

모델/엔티티 파일이 변경되었지만 마이그레이션 파일이 없는 경우:

```
경고: 모델/엔티티 파일이 변경되었지만 마이그레이션 파일이 포함되지 않았습니다.
  - src/models/User.ts 가 변경되었습니다.
  마이그레이션이 필요한 변경인지 확인해 주세요.
```

## PR Description 형식

PR description은 다음 형식으로 작성합니다. 상세 템플릿은 [references/templates.md](references/templates.md)를 참조합니다.

### 기본 구조

```markdown
## Summary
{변경 목적과 배경을 1-3문장으로 요약}

## Changes
### 변경 파일 통계
- 신규: N개 파일
- 수정: N개 파일
- 삭제: N개 파일
- 총 변경: +N줄 / -N줄

### 주요 변경사항
- {변경 1에 대한 설명}
- {변경 2에 대한 설명}

## Review Checklist
{카테고리별 맞춤 체크리스트 — 위 체크리스트 생성 로직에 따라}

## Warnings
{누락 항목 경고 — 해당사항이 있는 경우에만}

## Test Plan
- [ ] {테스트 항목 1}
- [ ] {테스트 항목 2}

## Related Issues
- Fixes #{이슈 번호} (해당하는 경우)
- Refs #{이슈 번호} (해당하는 경우)
```

### PR 크기 분류

변경량에 따라 PR 크기를 분류하고 레이블을 추천합니다:

| 크기 | 변경 줄 수 | 레이블 | 설명 |
|------|-----------|--------|------|
| **S** (Small) | 1-50줄 | `size/S` | 간단한 수정, 버그 수정 |
| **M** (Medium) | 51-200줄 | `size/M` | 일반적인 기능 추가 |
| **L** (Large) | 201-500줄 | `size/L` | 큰 기능, 리팩토링 |
| **XL** (Extra Large) | 501줄 이상 | `size/XL` | 대규모 변경, 분할 검토 필요 |

XL 이상의 PR은 다음 경고를 표시합니다:

```
경고: PR 크기가 매우 큽니다 (XL: {N}줄 변경).
대규모 PR은 리뷰 품질이 저하될 수 있습니다.
가능하다면 여러 개의 작은 PR로 분할하는 것을 권장합니다.
```

## GitHub Labels 추천

변경 카테고리에 따라 GitHub Label을 추천합니다:

| 카테고리 | 추천 Labels |
|----------|------------|
| DB 변경 | `database`, `migration` |
| API 변경 | `api`, `backend` |
| UI 변경 | `frontend`, `ui` |
| 설정 변경 | `config`, `infrastructure` |
| 테스트 변경 | `test`, `quality` |
| 문서 변경 | `documentation` |
| 비즈니스 로직 | `feature` 또는 `fix` (변경 성격에 따라) |
| 인프라/CI | `ci/cd`, `devops` |
| Breaking Change | `breaking-change` |

## Workflow 상세

### Step 1: Base Branch 확인

```bash
# 현재 브랜치 확인
CURRENT_BRANCH=$(git branch --show-current)

# base branch 감지 (기본값: main)
# main 또는 master 중 존재하는 브랜치 사용
git rev-parse --verify main 2>/dev/null && BASE="main" || BASE="master"

# 브랜치 분기점 이후 커밋 확인
git log --oneline ${BASE}...HEAD
```

base branch는 다음 순서로 감지합니다:
1. 사용자가 명시적으로 지정한 경우 → 해당 브랜치
2. `main` 브랜치가 존재하면 → `main`
3. `master` 브랜치가 존재하면 → `master`
4. 어느 것도 없으면 → 사용자에게 확인

### Step 2: 변경사항 분석

```bash
# 변경 파일 목록과 상태 (Added, Modified, Deleted 등)
git diff ${BASE}...HEAD --name-status

# 변경 통계 (파일별 추가/삭제 줄 수)
git diff ${BASE}...HEAD --stat

# 전체 diff (체크리스트 항목 결정에 사용)
git diff ${BASE}...HEAD

# 커밋 이력 (PR description의 Summary 작성에 사용)
git log --oneline ${BASE}...HEAD
```

### Step 3: 파일 분류

`git diff --name-status` 출력을 분석하여 [파일 분류 규칙](#파일-분류-규칙)에 따라 각 파일을 카테고리로 분류합니다.

분류 결과 예시:
```
[DB 변경]
  A  prisma/migrations/20240115_add_user_role.sql
  M  prisma/schema.prisma

[API 변경]
  M  src/controllers/userController.ts
  A  src/routes/adminRoutes.ts

[UI 변경]
  M  src/components/UserProfile.tsx
  A  src/pages/AdminDashboard.tsx

[테스트 변경]
  A  tests/controllers/userController.test.ts

[설정 변경]
  M  .env.example
```

### Step 4: 체크리스트 생성

분류된 카테고리 각각에 대해 [체크리스트 생성 로직](#체크리스트-생성-로직)의 해당 체크리스트를 포함합니다.

**중요**: 해당 카테고리에 변경이 없으면 해당 체크리스트는 생략합니다. 즉, 실제 변경된 카테고리에 대한 체크리스트만 포함합니다.

### Step 5: 누락 항목 경고

[누락 항목 경고](#누락-항목-경고) 규칙에 따라 누락된 항목을 점검하고 경고 섹션에 포함합니다.

### Step 6: PR Description 작성

[PR Description 형식](#pr-description-형식)에 따라 초안을 작성합니다.

- **Summary**: `git log` 커밋 메시지와 diff 내용을 분석하여 변경 목적 요약
- **Changes**: 파일 통계와 주요 변경사항을 나열
- **Review Checklist**: 카테고리별 맞춤 체크리스트
- **Warnings**: 누락 항목 경고 (해당사항 있을 때만)
- **Test Plan**: 테스트 방법 기술
- **Related Issues**: 커밋 메시지나 브랜치 이름에서 이슈 번호 추출

### Step 7: 사용자 확인 및 PR 생성

1. PR description 초안을 사용자에게 보여줍니다.
2. 수정 요청이 있으면 반영합니다.
3. 사용자가 승인하면 `gh pr create`로 PR을 생성합니다.

```bash
gh pr create \
  --title "{PR 제목}" \
  --body "{PR description}" \
  --base ${BASE} \
  --label "{추천 레이블}"
```

## Error Handling

### 에러 시나리오 및 대응

| # | 에러 | 감지 방법 | 대응 |
|---|------|----------|------|
| 1 | git 저장소가 아님 | `git rev-parse --is-inside-work-tree` 실패 | "git 저장소가 아닙니다" 안내 후 중단 |
| 2 | base branch 없음 | `git rev-parse --verify main` 및 `master` 모두 실패 | 사용자에게 base branch 확인 요청 |
| 3 | 변경사항 없음 | `git diff` 출력이 비어있음 | "변경사항이 없습니다" 안내 |
| 4 | 커밋되지 않은 변경 | `git status`에 unstaged/uncommitted 변경 존재 | 커밋 먼저 하도록 안내 |
| 5 | gh CLI 미설치 | `gh --version` 실패 | 체크리스트만 생성, PR 생성은 안내만 |
| 6 | gh 인증 안됨 | `gh auth status` 실패 | `gh auth login` 안내 |
| 7 | 원격 브랜치 push 안됨 | `git ls-remote` 확인 | push 먼저 하도록 안내 |
| 8 | 충돌 존재 | `gh pr create` 실패 시 conflict 메시지 | 충돌 해결 안내 |

### 에러 메시지 템플릿

상세 에러 메시지는 [references/templates.md](references/templates.md)의 에러 메시지 섹션을 참조합니다.

## 위험도 판단 기준

변경 내용에 따라 리뷰 시 특별히 주의해야 할 위험 수준을 판단합니다.

### 고위험 변경

다음 패턴이 감지되면 고위험으로 표시합니다:

- **DB 스키마 변경**: 컬럼 삭제, 타입 변경, 제약조건 변경
- **인증/인가 코드 변경**: 로그인, 권한 검사, 토큰 처리 관련 코드
- **결제/금융 로직 변경**: 결제 처리, 정산, 환불 관련 코드
- **보안 설정 변경**: CORS, CSP, 암호화, 시크릿 관련 설정
- **삭제 작업**: 파일 삭제, 기능 제거, API 폐기

### 중위험 변경

- **API 응답 형식 변경**: 클라이언트에 영향을 줄 수 있는 변경
- **공통 유틸리티 변경**: 여러 곳에서 사용되는 코드 변경
- **의존성 업데이트**: package.json, requirements.txt 등의 변경
- **환경 변수 추가/변경**: 배포 시 설정 필요

### 저위험 변경

- **문서 변경**: README, 주석 등
- **테스트만 변경**: 소스 코드 변경 없이 테스트만 추가/수정
- **코드 스타일 변경**: 포맷팅, 린트 수정
- **리네이밍**: 변수/함수명 변경 (로직 변경 없음)

## Best Practices

1. **작은 PR 유지**: 가능하면 200줄 이내의 PR을 만들어 리뷰 품질을 높이세요
2. **하나의 목적**: 하나의 PR에는 하나의 목적만 담으세요 (기능 추가와 리팩토링 분리)
3. **테스트 포함**: 소스 코드 변경 시 반드시 관련 테스트를 함께 포함하세요
4. **문서 동기화**: API나 설정 변경 시 관련 문서를 함께 갱신하세요
5. **의미 있는 커밋**: PR 내 커밋은 논리적 단위로 분리하세요
6. **자동 생성 체크리스트 활용**: 생성된 체크리스트를 리뷰어가 실제로 확인하도록 하세요
7. **경고 무시하지 않기**: 누락 경고가 있으면 반드시 확인 후 의도적 생략인지 판단하세요

## Analysis Guide

파일 분류 기준, 체크리스트 결정 로직, 위험도 판단에 대한 상세 방법론은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

## Templates

PR description 템플릿, 카테고리별 체크리스트 템플릿, 경고 메시지 등은 [references/templates.md](references/templates.md)를 참조하세요.
