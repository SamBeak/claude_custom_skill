---
name: changelog-release-manager
description: git 커밋 히스토리를 분석하여 시맨틱 버전 결정, CHANGELOG 자동 생성, 릴리스 노트 작성, Git 태그 및 GitHub Release를 자동화하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) 릴리스 준비, (2) CHANGELOG 생성, (3) 버전 업데이트, (4) 태그 생성, (5) 릴리스 노트 작성, (6) release, (7) version bump, (8) 새 버전 배포.
---

# Changelog Release Manager

git 커밋 히스토리를 분석하여 시맨틱 버전을 자동 결정하고, CHANGELOG.md를 생성하며, 릴리스 노트 초안을 작성하고, Git 태그 + GitHub Release까지 일괄 수행합니다.

## Quick Start

사용자가 릴리스 관련 작업을 요청하면 다음 워크플로우를 실행합니다:

1. **마지막 태그 이후 커밋 분석**:
   ```bash
   # 최신 태그 확인
   git describe --tags --abbrev=0

   # 최신 태그 이후 커밋 목록
   git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%H|%s|%an|%ai" --no-merges
   ```

2. **시맨틱 버전 결정**: 커밋 타입을 분석하여 major / minor / patch 중 적절한 bump 결정

3. **CHANGELOG 엔트리 생성**: 커밋을 타입별로 그룹화하여 Keep a Changelog 형식으로 작성

4. **릴리스 노트 초안 작성**: 사람이 읽기 좋은 릴리스 요약, 하이라이트, 기여자 목록 생성

5. **사용자 확인 후 릴리스 실행**: 태그 생성, 푸시, GitHub Release 생성

## 트리거 조건

다음 사용자 요청 시 이 스킬을 활성화합니다:

- "릴리스 준비해줘" / "릴리스 만들어줘"
- "CHANGELOG 생성해줘" / "CHANGELOG 업데이트"
- "버전 업데이트해줘" / "버전 올려줘"
- "태그 생성해줘" / "태그 만들어줘"
- "릴리스 노트 작성해줘" / "릴리스 노트 만들어줘"
- "release" / "new release" / "prepare release"
- "version bump" / "bump version"
- "새 버전 배포해줘"

## 커밋 분석

### 마지막 태그 이후 커밋 수집

```bash
# 태그 존재 여부 확인
git tag --sort=-v:refname | head -5

# 최신 태그 가져오기
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null)

# 태그가 있으면 태그 이후 커밋, 없으면 전체 커밋
if [ -n "$LAST_TAG" ]; then
  git log ${LAST_TAG}..HEAD --pretty=format:"%H|%s|%an|%ai" --no-merges
else
  git log --pretty=format:"%H|%s|%an|%ai" --no-merges
fi
```

### Conventional Commits 파싱

```regex
^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(([^)]+)\))?(!)?:\s+(.+)$
# 캡처 그룹:
# 1: type (feat, fix, etc.)
# 3: scope (선택)
# 4: breaking change marker (!)
# 5: description
```

### 비 Conventional Commits 키워드 분석

커밋 메시지가 Conventional Commits 형식이 아닌 경우, 키워드로 타입을 추론합니다:

| 키워드 패턴 | 추론 타입 | 설명 |
|------------|-----------|------|
| `add`, `added`, `new`, `create`, `implement` | feat | 새로운 기능 추가 |
| `fix`, `fixed`, `resolve`, `bug`, `patch`, `hotfix` | fix | 버그 수정 |
| `update`, `change`, `modify`, `improve`, `enhance` | refactor | 코드 변경/개선 |
| `remove`, `delete`, `drop`, `deprecate` | remove | 기능 제거 |
| `security`, `vulnerability`, `cve` | security | 보안 수정 |
| `performance`, `optimize`, `speed`, `cache` | perf | 성능 개선 |
| `breaking`, `migrate`, `incompatible` | breaking | 하위 호환성 깨짐 |

키워드로도 분류 불가능한 커밋은 "Other Changes" 섹션에 포함하고, 향후 `git-conventional-commits` 스킬 사용을 권장합니다.

## 시맨틱 버전 결정

### 버전 bump 규칙

[Semantic Versioning 2.0.0](https://semver.org/) 명세를 따릅니다.

| 조건 | Bump | 예시 |
|------|------|------|
| `BREAKING CHANGE` footer 또는 타입 뒤 `!` 마커 존재 | **Major** | 1.2.3 → 2.0.0 |
| `feat` 타입 커밋 존재 (breaking 없음) | **Minor** | 1.2.3 → 1.3.0 |
| `fix`, `perf`, `refactor` 등만 존재 | **Patch** | 1.2.3 → 1.2.4 |
| `docs`, `style`, `test`, `ci`, `chore`만 존재 | **Patch** | 1.2.3 → 1.2.4 |
| 태그 없음 (최초 릴리스) | **0.1.0** 또는 **1.0.0** | 사용자에게 확인 |

### Breaking Change 감지

다음 패턴을 Breaking Change로 감지합니다:

```regex
# 커밋 메시지 body/footer 내 BREAKING CHANGE
BREAKING CHANGE:\s*(.+)
BREAKING-CHANGE:\s*(.+)

# 타입 뒤 ! 마커
^(feat|fix|refactor|perf)(\([^)]+\))?!:
```

### 버전 결정 플로우

```
1. 모든 커밋 수집
   └─ Conventional Commits 파싱

2. Breaking Change 존재?
   ├─ 예 → Major bump
   └─ 아니오
      ├─ feat 커밋 존재?
      │   ├─ 예 → Minor bump
      │   └─ 아니오 → Patch bump
      └─ 커밋 없음 → 릴리스 불필요 안내

3. 현재 버전 + bump → 새 버전 산출

4. 사용자에게 버전 제안 및 확인
   ├─ 승인 → 해당 버전으로 진행
   └─ 수정 요청 → 사용자 지정 버전 사용
```

### 버전 소스 감지

프로젝트에서 현재 버전을 감지하는 소스:

| 소스 | 감지 방법 | 우선순위 |
|------|-----------|----------|
| Git 태그 | `git describe --tags --abbrev=0` | 1 (최우선) |
| `package.json` | `version` 필드 | 2 |
| `pyproject.toml` | `[project] version` 또는 `[tool.poetry] version` | 2 |
| `Cargo.toml` | `[package] version` | 2 |
| `pom.xml` | `<version>` 태그 | 2 |
| `build.gradle` | `version` 속성 | 2 |
| `VERSION` 파일 | 파일 전체 내용 | 3 |

## CHANGELOG 생성

### Keep a Changelog 형식

[Keep a Changelog](https://keepachangelog.com/ko/1.1.0/) 형식을 따릅니다.

### 커밋 타입별 CHANGELOG 섹션 매핑

| 커밋 타입 | CHANGELOG 섹션 | 포함 여부 |
|-----------|----------------|-----------|
| `feat` | Added | 항상 포함 |
| `fix` | Fixed | 항상 포함 |
| `refactor` | Changed | 항상 포함 |
| `perf` | Performance | 항상 포함 |
| `security` 또는 보안 관련 fix | Security | 항상 포함 |
| `revert` | Removed 또는 Changed | 항상 포함 |
| `docs` | Documentation | 선택 (사용자 설정) |
| `style` | (제외) | 기본 제외 |
| `test` | (제외) | 기본 제외 |
| `ci` | (제외) | 기본 제외 |
| `chore` | (제외) | 기본 제외 |
| `build` | (제외) | 기본 제외 |
| `BREAKING CHANGE` | Breaking Changes | 항상 포함 (최상단) |

### CHANGELOG 엔트리 형식

각 엔트리는 다음 형식을 따릅니다:

```markdown
## [1.3.0] - 2026-02-23

### Breaking Changes
- **api**: 응답 형식이 JSON:API 규격으로 변경됨 ([abc1234](https://github.com/owner/repo/commit/abc1234))
  - 마이그레이션 가이드: `docs/migration-v1.3.md` 참조

### Added
- **auth**: OAuth2 소셜 로그인 지원 추가 ([def5678](https://github.com/owner/repo/commit/def5678))
- **dashboard**: 실시간 알림 기능 추가 ([ghi9012](https://github.com/owner/repo/commit/ghi9012))

### Fixed
- **payment**: 결제 재시도 시 중복 청구 방지 ([jkl3456](https://github.com/owner/repo/commit/jkl3456))
- **ui**: 모바일 환경에서 버튼 정렬 오류 수정 ([mno7890](https://github.com/owner/repo/commit/mno7890))

### Changed
- **database**: 쿼리 빌더를 별도 모듈로 분리 ([pqr1234](https://github.com/owner/repo/commit/pqr1234))

### Performance
- **images**: 이미지 지연 로딩 구현으로 초기 로드 40% 개선 ([stu5678](https://github.com/owner/repo/commit/stu5678))

### Security
- **api**: SQL 인젝션 취약점 수정 ([vwx9012](https://github.com/owner/repo/commit/vwx9012))
```

### 기존 CHANGELOG 갱신 규칙

기존 CHANGELOG.md가 있는 경우:

1. 파일 상단의 헤더(`# Changelog`, 서문)를 보존
2. `## [Unreleased]` 섹션이 있으면 해당 내용을 새 버전 섹션으로 이동
3. 새 버전 섹션을 기존 버전 목록 맨 위에 삽입
4. 기존 버전 섹션은 수정하지 않음
5. 하단의 비교 링크(`[Unreleased]: ...`)를 갱신

## 릴리스 노트 생성

### 릴리스 노트 구성

릴리스 노트는 CHANGELOG보다 사람이 읽기 편한 형태로 작성합니다:

```markdown
# v1.3.0 릴리스 노트

릴리스 날짜: 2026-02-23

## 하이라이트

이번 릴리스에서는 OAuth2 소셜 로그인과 실시간 알림 기능이 추가되었습니다.
이미지 지연 로딩을 통해 초기 페이지 로드 속도가 40% 개선되었습니다.

## 주요 변경사항

### 새로운 기능
- OAuth2 소셜 로그인 (Google, GitHub, Kakao) 지원
- 대시보드 실시간 알림 (WebSocket 기반)

### 버그 수정
- 결제 재시도 시 중복 청구 문제 해결
- 모바일 UI 정렬 오류 수정

### 성능 개선
- 이미지 지연 로딩으로 초기 로드 시간 8초 → 4.8초

### 보안
- SQL 인젝션 취약점 패치

## Breaking Changes

### API 응답 형식 변경

API 응답이 JSON:API 규격으로 변경되었습니다.

**Before:**
```json
{ "user": { "id": 1, "name": "홍길동" } }
```

**After:**
```json
{ "data": { "type": "user", "id": "1", "attributes": { "name": "홍길동" } } }
```

마이그레이션 가이드: `docs/migration-v1.3.md`

## 업그레이드 가이드

1. API 클라이언트 코드에서 응답 파싱 로직 변경
2. `npm install` 또는 `yarn install`로 의존성 업데이트
3. 환경 변수 `OAUTH_CLIENT_ID`, `OAUTH_CLIENT_SECRET` 설정

## 기여자

이번 릴리스에 기여해주신 분들께 감사합니다:

- @contributor1
- @contributor2
- @contributor3
```

### 기여자 목록 수집

```bash
# 태그 이후 기여자 목록
git log $(git describe --tags --abbrev=0)..HEAD --pretty=format:"%an" --no-merges | sort | uniq
```

## 릴리스 자동화

### Pre-release 검증

릴리스를 실행하기 전에 다음 항목을 검증합니다:

#### 1. 작업 디렉토리 상태

```bash
# 커밋되지 않은 변경사항 확인
git status --porcelain
```

커밋되지 않은 변경사항이 있으면 릴리스를 중단하고 사용자에게 안내합니다.

#### 2. 테스트 통과 확인

```bash
# Node.js 프로젝트
npm test 2>/dev/null || yarn test 2>/dev/null

# Python 프로젝트
pytest 2>/dev/null || python -m unittest discover 2>/dev/null

# Go 프로젝트
go test ./... 2>/dev/null
```

테스트가 실패하면 릴리스를 중단하고 실패 내용을 안내합니다.

#### 3. CHANGELOG 형식 검증

```bash
# CHANGELOG.md 존재 확인
ls CHANGELOG.md 2>/dev/null

# Keep a Changelog 형식 기본 검증
# - "# Changelog" 헤더 존재
# - "## [x.y.z]" 형식의 버전 섹션 존재
# - 날짜 형식 YYYY-MM-DD
```

#### 4. 버전 일관성 확인

프로젝트 내 버전이 명시된 모든 파일에서 버전이 일치하는지 확인합니다:

```bash
# package.json 버전
grep '"version"' package.json 2>/dev/null

# pyproject.toml 버전
grep 'version' pyproject.toml 2>/dev/null

# Cargo.toml 버전
grep '^version' Cargo.toml 2>/dev/null
```

### Git 태그 + GitHub Release

사용자 확인 후 릴리스를 실행합니다:

```bash
# 1. 버전 파일 업데이트 (해당되는 경우)
# package.json, pyproject.toml 등의 version 필드 갱신

# 2. CHANGELOG.md 갱신
# 새 버전 엔트리를 CHANGELOG.md에 추가

# 3. 변경사항 커밋
git add CHANGELOG.md package.json  # 해당 파일들
git commit -m "chore: release v{VERSION}"

# 4. 태그 생성
git tag -a v{VERSION} -m "Release v{VERSION}"

# 5. 푸시
git push origin main  # 또는 현재 브랜치
git push origin v{VERSION}

# 6. GitHub Release 생성 (gh CLI 사용)
gh release create v{VERSION} \
  --title "v{VERSION}" \
  --notes-file release-notes.md
```

### Pre-release (RC, Beta, Alpha)

정식 릴리스 전 사전 릴리스를 지원합니다:

| 유형 | 버전 형식 | 태그 형식 | 설명 |
|------|-----------|-----------|------|
| Alpha | 1.3.0-alpha.1 | v1.3.0-alpha.1 | 초기 개발 버전 |
| Beta | 1.3.0-beta.1 | v1.3.0-beta.1 | 기능 완료, 테스트 중 |
| RC | 1.3.0-rc.1 | v1.3.0-rc.1 | 릴리스 후보 |

```bash
# Pre-release 태그 생성
git tag -a v{VERSION}-{TYPE}.{NUM} -m "Pre-release v{VERSION}-{TYPE}.{NUM}"
git push origin v{VERSION}-{TYPE}.{NUM}

# GitHub Pre-release 생성
gh release create v{VERSION}-{TYPE}.{NUM} \
  --title "v{VERSION}-{TYPE}.{NUM}" \
  --notes-file release-notes.md \
  --prerelease
```

## 모노레포 지원

### 모노레포 감지

```bash
# 모노레포 설정 파일 확인
ls lerna.json 2>/dev/null        # Lerna
ls pnpm-workspace.yaml 2>/dev/null  # pnpm Workspace
ls nx.json 2>/dev/null           # Nx
ls rush.json 2>/dev/null         # Rush
ls turbo.json 2>/dev/null        # Turborepo

# 워크스페이스 패키지 목록
cat pnpm-workspace.yaml 2>/dev/null
cat lerna.json 2>/dev/null | grep -A 10 '"packages"'
cat package.json 2>/dev/null | grep -A 10 '"workspaces"'
```

### 모노레포 릴리스 전략

#### 독립 버전 관리 (Independent Versioning)

각 패키지가 독립적인 버전을 가집니다:

```
packages/
├── @myorg/core         v2.1.0
├── @myorg/ui           v1.5.3
└── @myorg/utils        v3.0.0
```

- 해당 패키지 디렉토리 내의 커밋만 분석
- 패키지별 CHANGELOG.md 생성
- 패키지별 독립 태그: `@myorg/core@2.1.0`

#### 통합 버전 관리 (Fixed/Locked Versioning)

모든 패키지가 동일한 버전을 가집니다:

```
packages/
├── @myorg/core         v2.1.0
├── @myorg/ui           v2.1.0
└── @myorg/utils        v2.1.0
```

- 전체 커밋을 분석하여 단일 버전 결정
- 루트 CHANGELOG.md 생성
- 단일 태그: `v2.1.0`

### 패키지별 커밋 필터링

```bash
# 특정 패키지에 영향을 준 커밋만 필터링
git log $(git describe --tags --abbrev=0)..HEAD \
  --pretty=format:"%H|%s|%an|%ai" \
  --no-merges \
  -- packages/core/
```

### 패키지별 CHANGELOG

모노레포에서 각 패키지의 CHANGELOG를 생성합니다:

```
packages/core/CHANGELOG.md    ← @myorg/core 관련 변경사항만
packages/ui/CHANGELOG.md      ← @myorg/ui 관련 변경사항만
packages/utils/CHANGELOG.md   ← @myorg/utils 관련 변경사항만
CHANGELOG.md                  ← 전체 프로젝트 요약 (선택)
```

## 워크플로우 상세

```
1. [요청 분석]
   ├─ "릴리스 준비" → 전체 릴리스 워크플로우 실행
   ├─ "CHANGELOG 생성" → CHANGELOG 생성만 실행
   ├─ "버전 업데이트" → 버전 결정 + 파일 업데이트
   ├─ "태그 생성" → 태그 + 푸시
   ├─ "릴리스 노트" → 릴리스 노트 초안만 생성
   └─ "새 버전 배포" → 전체 릴리스 워크플로우 실행

2. [커밋 분석]
   ├─ 최신 태그 탐색
   ├─ 태그 이후 커밋 수집
   ├─ Conventional Commits 파싱
   ├─ 비표준 커밋 키워드 분석
   └─ Breaking Change 감지

3. [버전 결정]
   ├─ bump 타입 결정 (major / minor / patch)
   ├─ 새 버전 번호 산출
   ├─ 버전 파일 일관성 확인
   └─ 사용자 확인

4. [문서 생성]
   ├─ CHANGELOG 엔트리 작성
   ├─ 릴리스 노트 초안 작성
   └─ 사용자 확인 및 수정

5. [릴리스 실행] (사용자 승인 후)
   ├─ 버전 파일 업데이트
   ├─ CHANGELOG.md 갱신
   ├─ 릴리스 커밋 생성
   ├─ Git 태그 생성 + 푸시
   └─ GitHub Release 생성
```

## 에러 처리

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | Git 태그 없음 | `git describe --tags` 실패 | 전체 커밋 분석, 최초 릴리스(v0.1.0 또는 v1.0.0) 제안 |
| 2 | 새 커밋 없음 | `git log` 빈 출력 | 릴리스 불필요 안내, 마지막 릴리스 정보 표시 |
| 3 | 커밋되지 않은 변경사항 | `git status --porcelain` 비어있지 않음 | 릴리스 중단, 커밋 또는 stash 권유 |
| 4 | 테스트 실패 | 테스트 명령 exit code != 0 | 릴리스 중단, 실패 테스트 목록 표시 |
| 5 | Conventional Commits 미사용 | 커밋 메시지 파싱 실패율 > 50% | 키워드 기반 분류 진행, `git-conventional-commits` 스킬 추천 |
| 6 | 버전 파일 불일치 | 여러 소스의 버전이 다름 | 불일치 목록 표시, 통일 후 진행 권유 |
| 7 | CHANGELOG 형식 오류 | Keep a Changelog 형식 미준수 | 형식 수정 제안 또는 새로 생성 |
| 8 | gh CLI 미설치 | `gh --version` 실패 | 태그만 생성, GitHub Release는 수동 안내 |
| 9 | 원격 푸시 권한 없음 | `git push` 실패 | 로컬 태그만 생성, 푸시는 사용자에게 위임 |
| 10 | 모노레포 패키지 감지 실패 | 워크스페이스 설정 파일 파싱 오류 | 루트 레벨 단일 프로젝트로 처리 |

## Best Practices

1. **Conventional Commits 사용**: 정확한 버전 결정과 CHANGELOG 자동화의 핵심. `git-conventional-commits` 스킬과 병행 사용 권장
2. **자주 릴리스**: 작은 변경사항을 자주 릴리스하는 것이 큰 변경사항을 한꺼번에 릴리스하는 것보다 안전
3. **릴리스 전 테스트 필수**: 태그를 생성하기 전에 반드시 테스트 스위트 통과 확인
4. **Breaking Change 문서화**: Breaking Change에는 반드시 마이그레이션 가이드 포함
5. **Pre-release 활용**: 큰 변경사항은 alpha/beta/RC를 거쳐 점진적 릴리스
6. **CHANGELOG 사람이 읽기 좋게**: 기술 용어보다는 사용자 관점의 변경 설명 작성
7. **버전 파일 동기화**: package.json, pyproject.toml 등 모든 버전 파일을 함께 업데이트
8. **릴리스 노트에 업그레이드 가이드 포함**: 사용자가 업그레이드 시 무엇을 해야 하는지 명시

## git-conventional-commits 연동

이 스킬은 `git-conventional-commits` 스킬과 긴밀하게 연동됩니다:

1. `git-conventional-commits`로 작성된 커밋 메시지 → 이 스킬에서 자동으로 정확한 버전 결정 및 CHANGELOG 분류
2. Conventional Commits 형식이 아닌 커밋 → 키워드 기반 분류 시도 + `git-conventional-commits` 스킬 사용 권장
3. 릴리스 후 → `doc-generator` 스킬로 README의 버전 배지 및 변경 이력 링크 갱신

## Analysis Guide

커밋 분석 방법론, 버전 결정 알고리즘, CHANGELOG 생성 로직에 대한 상세 가이드는 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

## Templates

CHANGELOG 템플릿, 릴리스 노트 템플릿, 버전 파일 업데이트 패턴 등은 [references/templates.md](references/templates.md)를 참조하세요.
