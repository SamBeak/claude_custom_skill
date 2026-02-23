# PR Review Checklist 템플릿 모음

PR description 템플릿, 파일 타입별 체크리스트 템플릿, 경고 메시지 템플릿, GitHub Labels 추천 규칙을 포함합니다.

## 목차

1. [PR Description 템플릿](#pr-description-템플릿)
2. [파일 타입별 체크리스트 템플릿](#파일-타입별-체크리스트-템플릿)
3. [경고 메시지 템플릿](#경고-메시지-템플릿)
4. [GitHub Labels 추천 규칙](#github-labels-추천-규칙)
5. [PR 제목 템플릿](#pr-제목-템플릿)
6. [에러 메시지 템플릿](#에러-메시지-템플릿)

## PR Description 템플릿

### 기본 PR Description 템플릿

`{변수}`는 분석 결과로 치환됩니다.

```markdown
## Summary
{변경 목적과 배경을 1-3문장으로 요약}

## Changes

### 변경 파일 통계
| 구분 | 파일 수 |
|------|---------|
| 신규 (Added) | {added_count}개 |
| 수정 (Modified) | {modified_count}개 |
| 삭제 (Deleted) | {deleted_count}개 |
| **총 변경** | **+{insertions}줄 / -{deletions}줄** |

### PR 크기: {size_label} ({total_lines}줄)

### 주요 변경사항
{changes_list}

## Review Checklist

{category_checklists}

## Warnings

{warnings_section}

## Test Plan
{test_plan}

## Related Issues
{related_issues}

---
🤖 Generated with [Claude Code](https://claude.ai/code) - PR Review Checklist Skill
```

### 간략 PR Description 템플릿 (Size S)

작은 변경(50줄 이하)에 사용하는 간략한 템플릿:

```markdown
## Summary
{변경 목적 1문장 요약}

## Changes
{주요 변경사항 불릿 리스트}

총 변경: +{insertions}줄 / -{deletions}줄 (Size: S)

## Checklist
{핵심 체크리스트만 포함}

---
🤖 Generated with [Claude Code](https://claude.ai/code)
```

### 대규모 PR Description 템플릿 (Size XL)

500줄 이상의 대규모 변경에 사용하는 상세 템플릿:

```markdown
## Summary
{변경 목적과 배경 상세 설명}

> ⚠️ **대규모 PR 경고**: 이 PR은 {total_lines}줄의 변경을 포함하고 있습니다 (Size: XL).
> 리뷰 품질을 위해 가능하다면 PR 분할을 고려해 주세요.

## Changes

### 변경 파일 통계
| 구분 | 파일 수 |
|------|---------|
| 신규 (Added) | {added_count}개 |
| 수정 (Modified) | {modified_count}개 |
| 삭제 (Deleted) | {deleted_count}개 |
| **총 변경** | **+{insertions}줄 / -{deletions}줄** |

### PR 크기: XL ({total_lines}줄) - 분할 검토 권장

### 카테고리별 변경 요약

#### {category_1_name} ({category_1_count}개 파일)
{category_1_files_and_descriptions}

#### {category_2_name} ({category_2_count}개 파일)
{category_2_files_and_descriptions}

{additional_categories}

## Risk Level

위험도: {risk_level}

{risk_reasons}

## Review Checklist

{category_checklists}

## Warnings

{warnings_section}

## Test Plan
{test_plan}

## Migration / Deployment Notes
{deployment_notes}

## Related Issues
{related_issues}

## PR 분할 제안
{split_suggestions}

---
🤖 Generated with [Claude Code](https://claude.ai/code) - PR Review Checklist Skill
```

## 파일 타입별 체크리스트 템플릿

### DB / 마이그레이션 체크리스트 템플릿

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

### API 변경 체크리스트 템플릿

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

### UI 변경 체크리스트 템플릿

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

### 설정 변경 체크리스트 템플릿

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

### 테스트 변경 체크리스트 템플릿

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

### 비즈니스 로직 변경 체크리스트 템플릿

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

### 인프라/CI 변경 체크리스트 템플릿

```markdown
#### 인프라/CI 변경 체크리스트
- [ ] CI 파이프라인이 정상적으로 동작하는가?
- [ ] 배포 스크립트 변경 시 롤백 절차가 확인되었는가?
- [ ] 인프라 변경 시 비용 영향이 검토되었는가?
- [ ] 보안 그룹/네트워크 규칙이 올바른가?
- [ ] 환경 변수/시크릿 관리가 적절한가?
- [ ] 모니터링/알림 설정이 포함되어 있는가?
```

### 문서 변경 체크리스트 템플릿

```markdown
#### 문서 변경 체크리스트
- [ ] 문서 내용이 실제 코드/기능과 일치하는가?
- [ ] 코드 예제가 정상적으로 동작하는가?
- [ ] 링크가 유효한가? (깨진 링크 없음)
- [ ] 오타나 문법 오류가 없는가?
- [ ] 목차(TOC)가 갱신되었는가? (해당하는 경우)
```

## 경고 메시지 템플릿

### 테스트 파일 누락 경고

```
⚠️ 테스트 파일 누락 경고

다음 파일에 대응하는 테스트 파일이 변경 또는 추가되지 않았습니다:
{missing_test_files_list}

테스트 추가를 고려해 주세요. 의도적 생략이라면 PR description에 사유를 기재해 주세요.
```

변수 치환 예시:
```
⚠️ 테스트 파일 누락 경고

다음 파일에 대응하는 테스트 파일이 변경 또는 추가되지 않았습니다:
  - src/services/userService.ts → tests/services/userService.test.ts 없음
  - src/controllers/authController.ts → tests/controllers/authController.test.ts 없음

테스트 추가를 고려해 주세요. 의도적 생략이라면 PR description에 사유를 기재해 주세요.
```

### 문서 갱신 누락 경고

```
⚠️ 문서 갱신 누락 경고

{change_type}이(가) 변경되었지만 관련 문서가 갱신되지 않았습니다:
{missing_doc_details}

관련 문서의 갱신이 필요한지 확인해 주세요.
```

변수 치환 예시:
```
⚠️ 문서 갱신 누락 경고

API 엔드포인트가 변경되었지만 관련 문서가 갱신되지 않았습니다:
  - POST /api/users 엔드포인트가 추가되었습니다. API 문서 갱신이 필요할 수 있습니다.
  - 새 환경 변수 DATABASE_URL이 추가되었습니다. .env.example 및 배포 가이드 갱신이 필요할 수 있습니다.

관련 문서의 갱신이 필요한지 확인해 주세요.
```

### 마이그레이션 누락 경고

```
⚠️ 마이그레이션 누락 경고

모델/엔티티 파일이 변경되었지만 마이그레이션 파일이 포함되지 않았습니다:
{model_files_list}

마이그레이션이 필요한 변경인지 확인해 주세요.
(메서드만 추가된 경우 등 마이그레이션이 불필요한 변경일 수 있습니다.)
```

### PR 크기 경고 (XL)

```
⚠️ 대규모 PR 경고

PR 크기가 매우 큽니다 (XL: {total_lines}줄 변경, {file_count}개 파일).

대규모 PR은 리뷰 품질이 저하될 수 있습니다:
- 리뷰어의 집중력 저하
- 버그 발견 확률 감소
- 머지 충돌 가능성 증가
- 롤백 어려움

가능하다면 여러 개의 작은 PR로 분할하는 것을 권장합니다.

분할 제안:
{split_suggestions}
```

### 고위험 변경 경고

```
⚠️ 고위험 변경 경고

이 PR에는 특별한 주의가 필요한 고위험 변경이 포함되어 있습니다:
{high_risk_items}

권장 사항:
- 2인 이상의 리뷰를 받아주세요
- staging 환경에서 테스트를 진행해 주세요
- 롤백 계획을 확인해 주세요
```

### 시크릿 노출 경고

```
🚨 시크릿 노출 경고

다음 파일에서 시크릿 또는 민감 정보가 감지되었습니다:
{secret_files_list}

감지된 패턴:
{detected_patterns}

절대 시크릿을 코드에 하드코딩하지 마세요.
환경 변수나 시크릿 관리 도구를 사용해 주세요.
```

### 디버그 코드 경고

```
⚠️ 디버그 코드 경고

다음 파일에서 디버그 코드가 감지되었습니다:
{debug_code_locations}

배포 전에 디버그 코드를 제거해 주세요.
```

## GitHub Labels 추천 규칙

### 카테고리 기반 레이블

변경 카테고리에 따라 자동으로 추천하는 레이블입니다:

| 카테고리 | 추천 레이블 | 레이블 색상 (참고) |
|----------|-----------|----------------|
| DB 변경 | `database`, `migration` | `#0075ca` |
| API 변경 | `api`, `backend` | `#d73a4a` |
| UI 변경 | `frontend`, `ui` | `#7057ff` |
| 설정 변경 | `config`, `infrastructure` | `#e4e669` |
| 테스트 변경 | `test`, `quality` | `#0e8a16` |
| 문서 변경 | `documentation` | `#0075ca` |
| 비즈니스 로직 | 변경 성격에 따라 `feature` 또는 `fix` | `#a2eeef` |
| 인프라/CI | `ci/cd`, `devops` | `#f9d0c4` |

### PR 크기 기반 레이블

| PR 크기 | 레이블 |
|---------|--------|
| S (1-50줄) | `size/S` |
| M (51-200줄) | `size/M` |
| L (201-500줄) | `size/L` |
| XL (501줄+) | `size/XL` |

### 위험도 기반 레이블

| 위험도 | 레이블 |
|--------|--------|
| 고위험 | `high-risk` |
| 중위험 | `medium-risk` |
| 저위험 | (레이블 없음) |

### 특수 상황 레이블

| 조건 | 레이블 |
|------|--------|
| Breaking Change 포함 | `breaking-change` |
| 긴급 수정 | `hotfix` |
| WIP (Work in Progress) | `wip` |
| 리뷰 요청 | `needs-review` |
| 테스트 누락 | `needs-tests` |
| 문서 누락 | `needs-docs` |

### 레이블 조합 예시

```bash
# 예시 1: API 변경 + 중간 크기
gh pr create --label "api,backend,size/M"

# 예시 2: DB 마이그레이션 + 고위험 + 대규모
gh pr create --label "database,migration,high-risk,size/XL"

# 예시 3: UI 기능 추가 + 테스트 포함
gh pr create --label "frontend,ui,feature,size/M"

# 예시 4: 설정 변경 + 시크릿 주의
gh pr create --label "config,infrastructure,high-risk"
```

## PR 제목 템플릿

### 기본 형식

```
{type}({scope}): {description}
```

Conventional Commits 형식을 따릅니다.

### Type 결정 규칙

| 변경 성격 | Type | 예시 |
|----------|------|------|
| 새 기능 추가 | `feat` | `feat(user): 프로필 이미지 업로드 기능 추가` |
| 버그 수정 | `fix` | `fix(auth): 토큰 만료 시 무한 리프레시 방지` |
| 리팩토링 | `refactor` | `refactor(api): 에러 핸들링 공통화` |
| 문서 변경 | `docs` | `docs(readme): 설치 가이드 추가` |
| 테스트 추가 | `test` | `test(payment): 결제 플로우 통합 테스트 추가` |
| 설정 변경 | `chore` | `chore(docker): production 빌드 최적화` |
| CI/CD 변경 | `ci` | `ci(github): 자동 배포 워크플로우 추가` |
| 성능 개선 | `perf` | `perf(query): 사용자 목록 조회 쿼리 최적화` |
| 스타일 변경 | `style` | `style(lint): ESLint 규칙 적용` |
| 빌드 변경 | `build` | `build(deps): TypeScript 5.0 업그레이드` |

### Scope 결정 규칙

변경 파일의 공통 상위 디렉토리 또는 주요 도메인을 scope로 사용합니다:

| 변경 위치 | Scope 예시 |
|----------|-----------|
| `src/controllers/user*` | `user` |
| `src/services/auth*` | `auth` |
| `src/components/Dashboard*` | `dashboard` |
| `prisma/migrations/*` | `db` |
| `.github/workflows/*` | `ci` |
| 여러 도메인에 걸친 변경 | scope 생략 또는 가장 주요한 도메인 |

### PR 제목 예시

```
feat(user): 사용자 프로필 편집 기능 추가
fix(payment): 결제 금액 소수점 반올림 오류 수정
refactor(auth): JWT 토큰 검증 로직 분리
docs(api): OpenAPI 스펙 v2.1 갱신
test(order): 주문 취소 엣지케이스 테스트 추가
chore(deps): lodash 4.17.21 보안 패치 적용
ci(deploy): staging 자동 배포 파이프라인 구성
perf(search): 검색 쿼리 인덱스 최적화
```

## 에러 메시지 템플릿

### git 저장소가 아닌 경우

```
❌ git 저장소가 아닙니다.

현재 디렉토리가 git 저장소인지 확인해 주세요.
git 저장소를 초기화하려면:
  git init
```

### Base branch를 찾을 수 없는 경우

```
❌ base branch를 찾을 수 없습니다.

main 또는 master 브랜치가 존재하지 않습니다.
base branch를 지정해 주세요:
  예: "main 브랜치 기준으로 PR 만들어줘"
  예: "develop 브랜치에 PR 올려줘"
```

### 변경사항이 없는 경우

```
ℹ️ base branch 대비 변경사항이 없습니다.

현재 브랜치: {current_branch}
Base branch: {base_branch}

변경사항을 커밋한 후 다시 시도해 주세요.
```

### 커밋되지 않은 변경이 있는 경우

```
⚠️ 커밋되지 않은 변경사항이 있습니다.

다음 파일이 아직 커밋되지 않았습니다:
{unstaged_files}

모든 변경사항을 커밋한 후 PR을 생성해 주세요.
커밋을 도와드릴까요?
```

### gh CLI가 설치되지 않은 경우

```
⚠️ GitHub CLI(gh)가 설치되어 있지 않습니다.

PR description과 체크리스트는 아래에 출력합니다.
수동으로 GitHub에서 PR을 생성할 때 사용해 주세요.

GitHub CLI 설치:
  - macOS: brew install gh
  - Windows: winget install --id GitHub.cli
  - Linux: https://github.com/cli/cli/blob/trunk/docs/install_linux.md

설치 후 인증:
  gh auth login
```

### gh 인증이 안 된 경우

```
⚠️ GitHub CLI 인증이 필요합니다.

다음 명령어로 인증을 설정해 주세요:
  gh auth login

인증 후 다시 PR 생성을 요청해 주세요.
```

### 원격 브랜치에 push되지 않은 경우

```
⚠️ 현재 브랜치가 원격 저장소에 push되지 않았습니다.

현재 브랜치: {current_branch}

먼저 push를 진행합니다:
  git push -u origin {current_branch}

push 후 PR을 생성하시겠습니까?
```

### PR 생성 실패 (충돌) 경우

```
❌ PR 생성에 실패했습니다. 충돌이 존재합니다.

{base_branch} 브랜치와 충돌이 있습니다.
다음 단계를 진행해 주세요:

1. base branch를 merge 또는 rebase:
   git fetch origin
   git rebase origin/{base_branch}

2. 충돌 해결 후 force push:
   git push --force-with-lease

3. 다시 PR 생성을 요청해 주세요.
```

### PR 생성 성공 메시지

```
✅ PR이 성공적으로 생성되었습니다!

PR URL: {pr_url}
제목: {pr_title}
Base: {base_branch} ← {current_branch}
레이블: {labels}
크기: {size_label} ({total_lines}줄)

리뷰어에게 리뷰를 요청해 주세요.
```
