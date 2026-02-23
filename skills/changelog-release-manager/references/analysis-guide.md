# Changelog Release Manager 분석 가이드

릴리스 관리를 위한 커밋 분석 방법론, 시맨틱 버전 결정 알고리즘, CHANGELOG 생성 로직을 상세히 설명합니다.

## 목차

- [1. 커밋 수집 및 파싱](#1-커밋-수집-및-파싱)
- [2. 시맨틱 버전 결정 알고리즘](#2-시맨틱-버전-결정-알고리즘)
- [3. CHANGELOG 생성 로직](#3-changelog-생성-로직)
- [4. 릴리스 노트 구성 로직](#4-릴리스-노트-구성-로직)
- [5. 모노레포 분석](#5-모노레포-분석)
- [6. Pre-release 검증 체크리스트](#6-pre-release-검증-체크리스트)

---

## 1. 커밋 수집 및 파싱

### 1.1 태그 탐색

프로젝트의 태그를 semver 순으로 정렬하여 최신 태그를 찾습니다:

```bash
# semver 기준 정렬된 태그 목록
git tag --sort=-v:refname

# 최신 태그
git describe --tags --abbrev=0

# 태그 목록이 비어있는 경우 확인
git tag | wc -l
```

태그 형식 판별:

| 태그 형식 | 예시 | 설명 |
|-----------|------|------|
| `v{VERSION}` | v1.2.3 | 가장 일반적인 형식 |
| `{VERSION}` | 1.2.3 | v 접두사 없는 형식 |
| `{PACKAGE}@{VERSION}` | @myorg/core@1.2.3 | 모노레포 패키지별 태그 |
| `{PACKAGE}-v{VERSION}` | core-v1.2.3 | 모노레포 대안 형식 |

### 1.2 커밋 수집

```bash
# 태그가 있는 경우: 태그 이후 커밋
git log $(git describe --tags --abbrev=0)..HEAD \
  --pretty=format:"%H|%s|%an|%ai" \
  --no-merges

# 태그가 없는 경우: 전체 커밋 (최대 100개)
git log --pretty=format:"%H|%s|%an|%ai" \
  --no-merges \
  -100

# 커밋 body 포함 (BREAKING CHANGE footer 감지용)
git log $(git describe --tags --abbrev=0)..HEAD \
  --pretty=format:"%H|%s|%b|%an|%ai" \
  --no-merges
```

출력 필드:

| 필드 | format 코드 | 용도 |
|------|------------|------|
| 커밋 해시 (전체) | `%H` | CHANGELOG 링크 |
| 커밋 해시 (축약) | `%h` | 인라인 참조 |
| 커밋 제목 | `%s` | 타입 파싱, CHANGELOG 엔트리 |
| 커밋 body | `%b` | BREAKING CHANGE footer 감지 |
| 작성자 이름 | `%an` | 기여자 목록 |
| 작성 일시 | `%ai` | 시간순 정렬 |

### 1.3 Conventional Commits 파싱

#### 정규식

```regex
^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(([^)]+)\))?(!)?:\s+(.+)$
```

#### 파싱 결과 구조

각 커밋을 파싱한 결과:

```
{
  hash: "abc1234def5678",
  type: "feat",
  scope: "auth",          // 선택, 없을 수 있음
  breaking: false,         // ! 마커 또는 BREAKING CHANGE footer
  description: "add OAuth2 social login",
  body: "...",             // 선택
  author: "John Doe",
  date: "2026-02-20"
}
```

#### 파싱 실패 시 키워드 분석

Conventional Commits 형식이 아닌 커밋 메시지를 분석하는 순서:

```
1. 메시지 소문자 변환
2. 첫 단어 추출 (동사)
3. 키워드 매핑 테이블에서 타입 결정
4. 매핑 실패 → "other" 타입으로 분류
```

키워드 매핑 상세:

| 키워드 그룹 | 매핑 타입 | 우선순위 |
|------------|-----------|----------|
| `add`, `added`, `new`, `create`, `created`, `implement`, `implemented`, `introduce` | feat | 1 |
| `fix`, `fixed`, `resolve`, `resolved`, `bug`, `patch`, `hotfix`, `correct` | fix | 1 |
| `update`, `updated`, `change`, `changed`, `modify`, `modified`, `improve`, `improved`, `enhance` | refactor | 2 |
| `remove`, `removed`, `delete`, `deleted`, `drop`, `dropped`, `deprecate`, `deprecated` | remove | 2 |
| `security`, `secure`, `vulnerability`, `cve`, `xss`, `csrf`, `injection` | security | 1 |
| `performance`, `perf`, `optimize`, `optimized`, `speed`, `fast`, `cache`, `cached` | perf | 2 |
| `breaking`, `break`, `migrate`, `migration`, `incompatible` | breaking | 1 |
| `refactor`, `refactored`, `restructure`, `reorganize`, `cleanup`, `clean` | refactor | 3 |
| `doc`, `docs`, `document`, `documented`, `readme`, `comment` | docs | 3 |
| `test`, `tests`, `testing`, `spec`, `coverage` | test | 3 |

### 1.4 Breaking Change 감지

Breaking Change를 감지하는 모든 패턴:

```
1. 커밋 타입 뒤 ! 마커
   패턴: feat(api)!: ...
   정규식: ^(feat|fix|refactor|perf)(\([^)]+\))?!:

2. 커밋 body/footer의 BREAKING CHANGE
   패턴: BREAKING CHANGE: 설명
   정규식: BREAKING CHANGE:\s*(.+)

3. 커밋 body/footer의 BREAKING-CHANGE (하이픈)
   패턴: BREAKING-CHANGE: 설명
   정규식: BREAKING-CHANGE:\s*(.+)

4. 키워드 기반 (비표준 커밋)
   패턴: "breaking change", "incompatible", "migration required"
   정규식: (breaking\s+change|incompatible|migration\s+required)
```

---

## 2. 시맨틱 버전 결정 알고리즘

### 2.1 현재 버전 확인

여러 소스에서 현재 버전을 확인하고 우선순위에 따라 결정합니다:

```
[버전 소스 탐색 순서]
├─ 1. Git 태그 (최우선)
│     git describe --tags --abbrev=0
│     → v1.2.3 → 1.2.3
├─ 2. package.json
│     grep '"version"' package.json
│     → "version": "1.2.3" → 1.2.3
├─ 3. pyproject.toml
│     grep 'version' pyproject.toml
│     → version = "1.2.3" → 1.2.3
├─ 4. Cargo.toml
│     grep '^version' Cargo.toml
│     → version = "1.2.3" → 1.2.3
├─ 5. VERSION 파일
│     cat VERSION
│     → 1.2.3
└─ 6. 없음 → 0.0.0 (최초 릴리스)
```

### 2.2 Bump 타입 결정

```
[커밋 목록 분석]
│
├─ Breaking Change 존재?
│   └─ 예 → MAJOR bump
│
├─ feat 타입 커밋 존재?
│   └─ 예 → MINOR bump
│
├─ fix / perf / refactor / security 커밋 존재?
│   └─ 예 → PATCH bump
│
├─ docs / style / test / ci / chore / build만 존재?
│   └─ 예 → PATCH bump (선택적: 릴리스 건너뛰기 제안)
│
└─ 커밋 없음?
    └─ 릴리스 불필요
```

### 2.3 새 버전 산출

| 현재 버전 | Bump 타입 | 새 버전 | 규칙 |
|-----------|-----------|---------|------|
| 1.2.3 | MAJOR | 2.0.0 | major + 1, minor = 0, patch = 0 |
| 1.2.3 | MINOR | 1.3.0 | minor + 1, patch = 0 |
| 1.2.3 | PATCH | 1.2.4 | patch + 1 |
| 0.0.0 | 최초 릴리스 | 0.1.0 또는 1.0.0 | 사용자에게 확인 |
| 1.3.0-beta.2 | RELEASE | 1.3.0 | pre-release 제거 |

### 2.4 버전 일관성 검증

프로젝트 내 모든 버전 소스가 일치하는지 확인합니다:

```
[일관성 검증]
├─ Git 태그: v1.2.3
├─ package.json: 1.2.3     ← 일치
├─ pyproject.toml: 1.2.2   ← 불일치!
└─ VERSION: 1.2.3          ← 일치

결과: 불일치 감지 → 사용자에게 경고
```

---

## 3. CHANGELOG 생성 로직

### 3.1 커밋 그룹화

파싱된 커밋들을 CHANGELOG 섹션별로 그룹화합니다:

```
[커밋 그룹화 플로우]
│
├─ Breaking Changes 그룹
│   └─ breaking = true인 모든 커밋
│
├─ Added 그룹
│   └─ type = "feat"
│
├─ Fixed 그룹
│   └─ type = "fix" (security 관련 제외)
│
├─ Changed 그룹
│   └─ type = "refactor" 또는 type = "revert"
│
├─ Removed 그룹
│   └─ type = "remove" 또는 description에 "remove", "delete" 포함
│
├─ Performance 그룹
│   └─ type = "perf"
│
├─ Security 그룹
│   └─ type = "security" 또는 (type = "fix" 이면서 security 키워드)
│
└─ Other Changes 그룹 (선택)
    └─ 위 그룹에 속하지 않는 커밋 (docs, style, test, ci, build, chore)
```

### 3.2 엔트리 정렬

각 그룹 내에서 커밋을 정렬하는 규칙:

1. scope가 있는 커밋 → scope 알파벳순
2. scope가 없는 커밋 → description 알파벳순
3. 동일 scope 내 → 시간순 (최신이 먼저)

### 3.3 커밋 해시 링크 생성

GitHub 리포지토리 URL을 감지하여 커밋 해시에 링크를 추가합니다:

```bash
# 원격 저장소 URL 추출
git remote get-url origin
# https://github.com/owner/repo.git → https://github.com/owner/repo
# git@github.com:owner/repo.git → https://github.com/owner/repo
```

링크 형식: `([abc1234](https://github.com/owner/repo/commit/abc1234def5678))`

원격 URL을 감지하지 못하면 해시만 표시: `(abc1234)`

### 3.4 기존 CHANGELOG 갱신

```
[기존 CHANGELOG.md 갱신 플로우]
│
├─ 1. 파일 파싱
│   ├─ 헤더 영역 (# Changelog, 서문) 추출
│   ├─ [Unreleased] 섹션 추출 (있으면)
│   └─ 기존 버전 섹션들 추출
│
├─ 2. 새 버전 섹션 구성
│   ├─ [Unreleased] 내용을 새 버전으로 이동
│   ├─ 새 커밋 분석 결과 병합
│   └─ 날짜 추가
│
├─ 3. 파일 재구성
│   ├─ 헤더 보존
│   ├─ 비어있는 [Unreleased] 섹션 추가
│   ├─ 새 버전 섹션 삽입
│   └─ 기존 버전 섹션들 보존
│
└─ 4. 비교 링크 갱신 (하단)
    ├─ [Unreleased]: .../v{NEW}...HEAD
    ├─ [{NEW}]: .../v{OLD}...v{NEW}
    └─ 기존 링크 보존
```

---

## 4. 릴리스 노트 구성 로직

### 4.1 하이라이트 선별

릴리스 노트의 하이라이트 섹션에 포함할 항목 선별 기준:

| 우선순위 | 기준 | 설명 |
|---------|------|------|
| 1 | Breaking Changes | 항상 하이라이트에 포함 |
| 2 | feat 커밋 중 scope가 있는 것 | 주요 기능 추가 |
| 3 | security 관련 커밋 | 보안 패치 강조 |
| 4 | perf 커밋 중 수치가 있는 것 | 정량적 성능 개선 |

### 4.2 기여자 목록 수집

```bash
# 기여자 이름 + 이메일
git log $(git describe --tags --abbrev=0)..HEAD \
  --pretty=format:"%an <%ae>" \
  --no-merges | sort | uniq

# GitHub 사용자명이 필요한 경우
git log $(git describe --tags --abbrev=0)..HEAD \
  --pretty=format:"%an" \
  --no-merges | sort | uniq
```

### 4.3 업그레이드 가이드 생성

Breaking Change가 있는 경우 업그레이드 가이드를 자동 구성합니다:

```
[업그레이드 가이드 구성]
├─ 각 Breaking Change 별:
│   ├─ 변경 내용 설명
│   ├─ Before / After 코드 예시 (가능한 경우)
│   └─ 마이그레이션 단계
├─ 의존성 업데이트 명령
│   └─ npm install / yarn install / pip install 등
└─ 환경 변수 변경 사항 (있는 경우)
```

---

## 5. 모노레포 분석

### 5.1 모노레포 감지

```
[모노레포 설정 파일 탐색]
├─ lerna.json → Lerna
│   └─ "packages" 필드에서 패키지 경로 추출
├─ pnpm-workspace.yaml → pnpm Workspace
│   └─ "packages" 필드에서 glob 패턴 추출
├─ nx.json → Nx
│   └─ projects 목록 추출
├─ rush.json → Rush
│   └─ "projects" 필드에서 패키지 경로 추출
├─ turbo.json → Turborepo
│   └─ package.json의 "workspaces" 참조
└─ package.json → Yarn/npm Workspaces
    └─ "workspaces" 필드에서 glob 패턴 추출
```

### 5.2 패키지 목록 구성

```bash
# pnpm workspace 예시
cat pnpm-workspace.yaml
# packages:
#   - 'packages/*'
#   - 'apps/*'

# 실제 패키지 디렉토리 나열
ls packages/
ls apps/

# 각 패키지의 이름과 버전
for pkg in packages/*/; do
  grep -E '"(name|version)"' "${pkg}package.json" 2>/dev/null
done
```

### 5.3 패키지별 커밋 필터링

```bash
# 특정 패키지 경로에 영향을 준 커밋만 필터링
git log $(git describe --tags --match "@myorg/core@*" --abbrev=0 2>/dev/null || echo HEAD~50)..HEAD \
  --pretty=format:"%H|%s|%an|%ai" \
  --no-merges \
  -- packages/core/
```

### 5.4 버전 관리 전략 판별

```
[버전 관리 전략 판별]
├─ lerna.json의 "version" 필드
│   ├─ "independent" → 독립 버전 관리
│   └─ "x.y.z" (구체 버전) → 통합 버전 관리
├─ nx.json 또는 turbo.json
│   └─ 별도 설정 없으면 → 독립 버전 관리 기본
└─ 기타
    └─ 사용자에게 전략 확인
```

---

## 6. Pre-release 검증 체크리스트

릴리스 실행 전 검증 항목:

### 작업 디렉토리

- [ ] `git status --porcelain` 출력이 비어있는가?
- [ ] 현재 브랜치가 main/master 또는 release 브랜치인가?
- [ ] 원격 브랜치와 동기화되어 있는가? (`git fetch && git status`)

### 테스트

- [ ] 테스트 스위트가 통과하는가?
- [ ] 테스트 커버리지가 프로젝트 기준 이상인가? (선택)

### 버전

- [ ] 새 버전 번호가 기존 태그와 중복되지 않는가?
- [ ] 모든 버전 파일(package.json 등)이 일관적인가?
- [ ] Pre-release인 경우 올바른 형식인가? (x.y.z-alpha.N)

### CHANGELOG

- [ ] CHANGELOG.md가 Keep a Changelog 형식을 따르는가?
- [ ] `# Changelog` 헤더가 존재하는가?
- [ ] 버전 섹션이 `## [x.y.z] - YYYY-MM-DD` 형식인가?
- [ ] 날짜가 ISO 8601 형식인가?
- [ ] Breaking Change가 누락 없이 문서화되었는가?

### GitHub Release

- [ ] `gh` CLI가 설치되어 있는가?
- [ ] GitHub에 인증되어 있는가? (`gh auth status`)
- [ ] 릴리스 노트 파일이 준비되었는가?
