# Git Conventional Commits

git diff 분석과 코드베이스 컨텍스트를 기반으로 Conventional Commits 규격에 맞는 시맨틱하고 표준화된 커밋 메시지를 자동 생성합니다.

## 기능

이 스킬은 다음과 같은 방식으로 일관되고 의미있는 커밋 메시지를 생성합니다:

- `git diff --staged` 분석으로 변경 사항 파악
- 영향받은 파일을 읽어서 컨텍스트 이해
- 변경 타입별로 분류 (feat, fix, refactor 등)
- 적절한 scope 자동 결정 (컴포넌트, 모듈, 파일)
- 잘 포맷된 커밋 메시지 생성
- git 히스토리에서 CHANGELOG 자동 생성

## 사용 시점

다음과 같은 경우에 이 스킬을 사용하세요:

- Conventional Commits 규격을 따르는 커밋 메시지 생성
- 어떤 타입의 변경인지 파악하고 싶을 때
- 커밋 메시지 scope 추천받기
- 여러 변경사항을 집중된 커밋들로 분리
- git 히스토리에서 CHANGELOG 파일 생성
- 프로젝트용 커밋 메시지 템플릿 설정
- git hooks로 커밋 규칙 강제

## 간단한 예시

### 요청
> "스테이징된 변경사항으로 커밋 메시지 만들어줘"

### 응답
```
feat(user-auth): add JWT token validation

보호된 라우트를 위한 JWT 토큰 검증 미들웨어 구현.
토큰 서명, 만료 시간, 사용자 권한을 검증합니다.

Fixes #123
```

---

### 요청
> "최근 커밋들로 CHANGELOG 만들어줘"

### 응답
```markdown
# Changelog

## [1.2.0] - 2024-01-15

### Features
- **auth**: JWT 토큰 검증 추가 (#123)
- **api**: 속도 제한 미들웨어 추가 (#124)

### Bug Fixes
- **database**: 커넥션 풀 누수 수정 (#125)
```

## Conventional Commits 포맷

기본 형식:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 타입 (Type)

- **feat**: 새로운 기능
- **fix**: 버그 수정
- **docs**: 문서 변경
- **style**: 코드 포맷팅 (기능 변경 없음)
- **refactor**: 코드 리팩토링 (기능 변경 없음)
- **perf**: 성능 개선
- **test**: 테스트 추가 또는 업데이트
- **build**: 빌드 시스템 변경
- **ci**: CI/CD 변경
- **chore**: 유지보수 작업

### 예시

**간단한 기능:**
```
feat(button): add loading state animation
```

**이슈 참조가 있는 버그 수정:**
```
fix(api): prevent token refresh race condition

여러 동시 API 호출이 여러 토큰 갱신 요청을 트리거해서
인증 실패가 발생했습니다.

Fixes #456
```

**Breaking change:**
```
feat(api)!: migrate to GraphQL

BREAKING CHANGE: 모든 REST 엔드포인트가 제거되었습니다.
대신 /graphql 엔드포인트를 사용하세요.
```

## 동작 방식

1. **스테이징된 변경사항 분석**: `git diff --staged` 실행
2. **영향받은 파일 읽기**: 코드 컨텍스트 이해
3. **변경사항 분류**: 타입과 scope별로 분류
4. **메시지 생성**: Conventional Commits 규격에 따라 생성
5. **확인 요청**: 커밋하기 전에 사용자 확인

## 장점

### 개발자에게
- **일관된 커밋**: 모든 커밋이 동일한 형식을 따름
- **더 나은 히스토리**: 무엇이 왜 변경되었는지 쉽게 이해
- **빠른 리뷰**: 리뷰어가 커밋 목적을 빠르게 파악
- **고민 불필요**: AI가 변경사항을 분석하고 분류

### 팀에게
- **자동 CHANGELOG**: 릴리스 노트 자동 생성
- **시맨틱 버저닝**: 버전 번호 자동 결정
- **더 나은 디버깅**: 버그가 언제 도입되었는지 쉽게 찾기
- **문서화**: 커밋 히스토리가 프로젝트 문서가 됨

### 프로젝트에게
- **전문적인 외관**: 성숙도와 세심함을 보여줌
- **쉬운 유지보수**: 미래의 개발자가 히스토리를 이해하기 쉬움
- **자동 릴리스**: CI/CD 도구가 커밋을 파싱 가능
- **읽기 쉬운 git log**: `git log --oneline`이 가독성 있어짐

## 프로젝트 설정

### 1. 커밋 메시지 템플릿

프로젝트 루트에 `.gitmessage` 파일 생성:

```
# <type>(<scope>): <subject>

# <body>

# <footer>

# Type: feat, fix, docs, style, refactor, perf, test, build, ci, chore
# Scope: 영향받은 컴포넌트/모듈 (선택사항)
# Subject: 명령형 어조, 소문자, 마침표 없음, 최대 50자
```

기본값으로 설정:
```bash
git config commit.template .gitmessage
```

### 2. 커밋 린팅 (선택사항)

commitlint 설치:
```bash
npm install --save-dev @commitlint/cli @commitlint/config-conventional
```

`commitlint.config.js` 생성:
```javascript
module.exports = {
  extends: ['@commitlint/config-conventional']
};
```

### 3. Husky로 Git Hooks 설정 (선택사항)

```bash
npm install --save-dev husky
npx husky install
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit $1'
```

## 주요 시나리오

### 혼합된 변경사항

**문제**: 여러 관련없는 것들을 수정함

**해결**: 스킬이 여러 커밋으로 나누기를 제안
```bash
# auth 관련 변경사항만 스테이징
git add src/auth/*

# auth 변경사항 커밋
git commit -m "feat(auth): add OAuth2 support"

# API 변경사항 스테이징
git add src/api/*

# API 변경사항 커밋
git commit -m "refactor(api): simplify error handling"
```

### Breaking Changes

**문제**: 변경사항이 하위 호환성을 깨뜨림

**해결**: `!` 또는 `BREAKING CHANGE` footer 사용
```
feat(api)!: change response format

BREAKING CHANGE: 모든 API 응답이 이제 JSON:API 규격을 따릅니다.
이전 응답 형식은 더 이상 지원되지 않습니다.

마이그레이션 가이드: docs/api-migration.md
```

### 버그 수정

**문제**: 버그를 수정하고 이슈 참조가 필요함

**해결**: footer에 이슈 참조 추가
```
fix(payment): prevent duplicate charge on retry

결제 요청에 멱등성 키를 추가하여 사용자가 실패한
결제를 재시도할 때 중복 청구를 방지합니다.

Fixes #789
```

## 통합 예시

### GitHub Actions와 함께

릴리스 시 자동으로 CHANGELOG 생성:

```yaml
name: Release
on:
  push:
    tags:
      - 'v*'
jobs:
  changelog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Generate CHANGELOG
        run: |
          npx conventional-changelog -p angular -i CHANGELOG.md -s
```

### semantic-release와 함께

자동으로 버전 결정 및 배포:

```javascript
// release.config.js
module.exports = {
  branches: ['main'],
  plugins: [
    '@semantic-release/commit-analyzer',
    '@semantic-release/release-notes-generator',
    '@semantic-release/changelog',
    '@semantic-release/npm',
    '@semantic-release/github'
  ]
};
```

## 팁

- **자주 커밋**: 작고 집중된 커밋이 큰 커밋보다 좋음
- **구체적으로**: "fix login bug" → "fix password validation regex"
- **명령형 어조 사용**: "add feature" not "added feature"
- **이유 설명**: body는 무엇이 아니라 왜를 설명해야 함
- **이슈 참조**: 항상 이슈 트래커에 링크
- **먼저 테스트**: 커밋 전에 테스트가 통과하는지 확인
- **diff 리뷰**: 커밋 전에 항상 `git diff --staged` 확인

## 참고 자료

- [Conventional Commits 명세](https://www.conventionalcommits.org/)
- [시맨틱 버저닝](https://semver.org/)
- [commitlint](https://commitlint.js.org/)
- [conventional-changelog](https://github.com/conventional-changelog/conventional-changelog)

## 관련 문서

- [SKILL.md](SKILL.md) - Claude Code용 상세 사용 지침
- [references/analysis-guide.md](references/analysis-guide.md) - 변경사항 분석 방법
- [references/templates.md](references/templates.md) - 메시지 템플릿과 예시
