# Claude Custom Skills

Claude Code용 커스텀 스킬 모음

## Skills

### 코드 품질

#### [code-formatter](./skills/code-formatter)
코드베이스 분석을 통해 언어별 코드 포매팅 설정 파일(ESLint, Prettier, EditorConfig 등)을 자동 생성합니다.

#### [refactor-advisor](./skills/refactor-advisor)
코드베이스를 분석하여 복잡도, 코드 스멜, 중복 코드를 탐지하고 우선순위별 리팩토링 방안을 Before/After 코드 예시와 함께 제시합니다.

### 보안

#### [security-scanner](./skills/security-scanner)
코드 변경사항에서 OWASP Top 10 기반 보안 취약점을 사전 탐지하고, 시크릿/API 키 노출을 감지하여 수정 가이드를 제공합니다.

### 테스트

#### [test-coverage-analyzer](./skills/test-coverage-analyzer)
변경된 코드에 대한 테스트 커버리지를 분석하고, 누락된 테스트 케이스를 식별하여 테스트 코드 스켈레톤을 자동 생성합니다.

### Git / 버전 관리

#### [git-conventional-commits](./skills/git-conventional-commits)
git diff 분석과 코드베이스 컨텍스트를 기반으로 Conventional Commits 규격에 맞는 커밋 메시지를 자동 생성합니다.

#### [changelog-release-manager](./skills/changelog-release-manager)
git 커밋 히스토리를 분석하여 CHANGELOG를 생성하고, 시맨틱 버전 결정 및 GitHub Release 자동화를 수행합니다.

### 코드 리뷰

#### [pr-review-checklist](./skills/pr-review-checklist)
PR 생성 시 git diff를 분석하여 변경사항 종류별 맞춤 리뷰 체크리스트를 자동 생성하고, PR description을 작성합니다.

### 문서화

#### [doc-generator](./skills/doc-generator)
프로젝트 코드베이스를 분석하여 README.md, API 문서, CHANGELOG, JSDoc/docstring 등 다양한 문서를 자동 생성하고 갱신합니다.

### API 설계

#### [api-design-reviewer](./skills/api-design-reviewer)
REST API 엔드포인트의 설계 품질을 분석하여 네이밍 컨벤션, 응답 일관성, 페이지네이션, 에러 표준화, 버전 관리 등을 리뷰합니다.

### 에러 처리

#### [error-handling-analyzer](./skills/error-handling-analyzer)
코드의 에러 처리 안티패턴을 탐지하고, API 에러 응답 일관성, 에러 바운더리, 복원력 패턴(재시도/타임아웃/서킷 브레이커)을 분석합니다.

### 의존성 관리

#### [dependency-manager](./skills/dependency-manager)
프로젝트 의존성을 분석하여 outdated 패키지 분류, 메이저 업그레이드 마이그레이션 가이드, 라이선스 호환성 검사를 수행합니다.

### CI/CD

#### [cicd-pipeline-reviewer](./skills/cicd-pipeline-reviewer)
GitHub Actions, GitLab CI, Jenkins 등 CI/CD 파이프라인 설정을 분석하여 보안, 성능, 안정성 관점에서 개선안을 제시합니다.

### 기술 부채

#### [tech-debt-tracker](./skills/tech-debt-tracker)
TODO/FIXME 마커, deprecated API 사용, 타입 안전성 갭 등을 종합 분석하여 기술 부채 현황과 우선순위 기반 개선 로드맵을 생성합니다.

### 플랜 리뷰

#### [plan-codex-review](./skills/plan-codex-review)
Claude Code 플랜 파일을 OpenAI Codex CLI로 전송하여 기술적 타당성, 보안, 성능, 테스트 전략 등 다각도로 리뷰하고 개선합니다.

### AI 설정

#### [ai-agent-rules](./skills/ai-agent-rules)
AI 기반 IDE와 코딩 에이전트를 위한 프로젝트별 커스텀 코딩 규칙 및 가이드라인을 생성합니다.

## Usage

각 스킬의 디렉토리에서 `SKILL.md` 파일을 참조하여 사용 방법을 확인하세요.

## License

이 프로젝트는 MIT 라이선스 하에 배포됩니다.
