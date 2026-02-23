# Changelog Release Manager

git 커밋 히스토리를 분석하여 시맨틱 버전 결정, CHANGELOG 자동 생성, 릴리스 노트 작성, Git 태그 및 GitHub Release를 자동화하는 Claude Code 스킬입니다.

## 스킬 소개

소프트웨어 릴리스는 버전 결정, CHANGELOG 작성, 릴리스 노트 정리, 태그 생성, GitHub Release 발행 등 여러 단계를 거치는 반복적인 작업입니다. Changelog Release Manager는 마지막 태그 이후의 커밋을 자동 분석하여 시맨틱 버전을 결정하고, Keep a Changelog 형식의 변경 이력을 생성하며, 릴리스 노트 초안까지 한번에 작성합니다.

기존 `git-conventional-commits` 스킬로 작성된 커밋 메시지가 있으면 가장 정확한 자동화가 가능하고, Conventional Commits 형식이 아닌 프로젝트에서도 키워드 기반 분류를 통해 합리적인 CHANGELOG를 생성합니다. `doc-generator` 스킬과 연동하면 릴리스 후 README의 버전 배지 및 변경 이력 링크를 자동으로 갱신할 수 있습니다.

## 주요 기능

### 1. 커밋 분석 및 시맨틱 버전 결정
- 마지막 Git 태그 이후의 모든 커밋을 자동 수집
- Conventional Commits 파싱으로 정확한 타입 분류
- BREAKING CHANGE → major, feat → minor, fix → patch 자동 판단
- 비표준 커밋도 키워드 분석으로 타입 추론
- package.json, pyproject.toml 등 버전 파일 일관성 검증

### 2. CHANGELOG 자동 생성
- Keep a Changelog 형식 준수
- 커밋을 Added, Fixed, Changed, Removed, Security, Performance로 그룹화
- 커밋 해시 링크 포함
- Breaking Changes 섹션 + 마이그레이션 노트
- 기존 CHANGELOG.md가 있으면 상단에 새 버전 엔트리 추가

### 3. 릴리스 노트 작성
- 사람이 읽기 좋은 변경사항 요약
- 주요 기능 하이라이트 섹션
- 기여자 목록 자동 수집
- Breaking Change 업그레이드 가이드

### 4. 릴리스 자동화
- Pre-release 검증 (미커밋 변경사항, 테스트 통과, 버전 일관성)
- Git 태그 생성 및 푸시
- GitHub Release 자동 생성 (gh CLI 사용)
- Alpha, Beta, RC 등 Pre-release 지원

### 5. 모노레포 지원
- Lerna, pnpm Workspace, Nx, Turborepo, Rush 자동 감지
- 패키지별 독립 CHANGELOG 생성
- 독립 버전 관리 및 통합 버전 관리 모두 지원

## 사용 예시

### 전체 릴리스 워크플로우

```
사용자: 릴리스 준비해줘

Claude: 커밋 히스토리를 분석합니다...

분석 결과:
- 마지막 태그: v1.2.3
- 새 커밋: 15개
- feat: 3개, fix: 5개, refactor: 2개, docs: 3개, chore: 2개
- Breaking Change: 없음

버전 결정: v1.2.3 → v1.3.0 (feat 커밋 존재로 minor bump)

CHANGELOG 엔트리와 릴리스 노트 초안을 생성했습니다.
[초안 내용 표시]

이 내용으로 릴리스를 진행할까요?
(A) 릴리스 실행 (B) 내용 수정 (C) 취소
```

### CHANGELOG만 생성

```
사용자: CHANGELOG 업데이트해줘

Claude: 최신 태그 이후 커밋을 분석합니다...

분석 결과:
- 마지막 태그: v2.0.0
- Unreleased 커밋: 8개
- Conventional Commits 준수율: 100%

CHANGELOG.md의 [Unreleased] 섹션을 업데이트했습니다.
[변경 내용 표시]
```

### 버전 bump만 수행

```
사용자: 버전 올려줘

Claude: 커밋을 분석하여 버전을 결정합니다...

현재 버전: v1.5.2
커밋 분석:
- fix 커밋 3개
- BREAKING CHANGE 없음, feat 없음

제안: v1.5.2 → v1.5.3 (patch bump)

package.json의 version 필드를 업데이트할까요?
```

### 모노레포 릴리스

```
사용자: @myorg/core 패키지 릴리스 준비해줘

Claude: 모노레포를 감지했습니다 (pnpm workspace).

packages/core/ 관련 커밋을 분석합니다...

분석 결과:
- 마지막 태그: @myorg/core@3.1.0
- 새 커밋: 6개 (packages/core/ 관련)
- feat: 2개, fix: 1개

버전 결정: @myorg/core@3.1.0 → @myorg/core@3.2.0

packages/core/CHANGELOG.md를 업데이트하겠습니다.
[초안 내용 표시]
```

## 에러 처리

| 에러 | 대응 |
|------|------|
| Git 태그 없음 | 전체 커밋 분석, 최초 릴리스 버전 제안 |
| 새 커밋 없음 | 릴리스 불필요 안내 |
| 커밋되지 않은 변경사항 | 릴리스 중단, 커밋 또는 stash 권유 |
| 테스트 실패 | 릴리스 중단, 실패 내용 표시 |
| Conventional Commits 미사용 | 키워드 기반 분류 + 스킬 추천 |
| 버전 파일 불일치 | 불일치 목록 표시, 통일 권유 |
| gh CLI 미설치 | 태그만 생성, GitHub Release는 수동 안내 |

## 관련 문서

- [SKILL.md](SKILL.md) - Claude Code용 상세 지침 (트리거 조건, 워크플로우, 에러 처리)
- [references/analysis-guide.md](references/analysis-guide.md) - 커밋 분석 방법론, 버전 결정 알고리즘
- [references/templates.md](references/templates.md) - CHANGELOG, 릴리스 노트, 버전 파일 업데이트 템플릿
