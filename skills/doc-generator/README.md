# Doc Generator

프로젝트 코드베이스를 분석하여 README.md, API 문서, CHANGELOG, JSDoc/docstring 등 다양한 문서를 자동 생성하고 갱신하는 Claude Code 스킬입니다.

## 스킬 소개

소프트웨어 개발에서 문서화는 가장 중요하면서도 가장 많이 미루는 작업입니다. Doc Generator는 코드베이스의 구조, 기술 스택, API 엔드포인트, git 히스토리를 자동으로 분석하여 사람이 읽기 좋은 문서를 생성합니다.

기존 `git-conventional-commits` 스킬과 연동하면 Conventional Commits 기반의 정확한 CHANGELOG를 자동 생성할 수 있으며, `pr-review-checklist` 스킬의 "문서 미갱신" 경고를 사전에 방지할 수 있습니다.

## 주요 기능

### 1. README.md 자동 생성/갱신
- 프로젝트 이름, 설명, 기술 스택 자동 감지
- 설치 방법, 실행 방법, 테스트 방법 자동 추출
- 디렉토리 구조 트리 생성
- 환경 변수 문서화 (.env.example 기반)
- 기존 README의 수동 작성 섹션 보존

### 2. API 문서 생성
- Express, NestJS, Django, Flask, Spring Boot, FastAPI, Go 등 주요 프레임워크 지원
- HTTP 메서드, 경로, 파라미터, 응답 형식 자동 추출
- 인증 미들웨어 적용 여부 표시
- Markdown 테이블 형식 출력

### 3. CHANGELOG.md 생성
- git 커밋 히스토리 + 태그 기반 버전별 변경 이력
- Conventional Commits 자동 파싱 (feat → Added, fix → Fixed 등)
- Keep a Changelog 형식 준수
- BREAKING CHANGE 자동 강조

### 4. JSDoc / Docstring 일괄 생성
- 미문서화된 public 함수/클래스/메서드 자동 탐지
- TypeScript/JavaScript (JSDoc), Python (docstring), Java (Javadoc), Go (GoDoc) 지원
- 매개변수 타입, 반환값, 예외 정보 자동 추론
- 기존 문서는 보존하고 누락된 태그만 보완

## 사용 예시

### README 생성

```
사용자: README 만들어줘

Claude: 프로젝트를 분석합니다...

📋 프로젝트 분석 결과:
- 이름: my-api-server
- 기술 스택: Node.js + Express + TypeScript + PostgreSQL
- 패키지 매니저: npm
- 테스트: Jest
- CI/CD: GitHub Actions

README.md 초안을 생성했습니다. [초안 내용 표시]

이대로 작성할까요? 수정할 부분이 있으면 말씀해주세요.
```

### CHANGELOG 생성

```
사용자: CHANGELOG 생성해줘

Claude: git 히스토리를 분석합니다...

📋 분석 결과:
- 태그: v1.0.0, v1.1.0, v1.2.0
- v1.2.0 이후 미릴리스 커밋: 12개
- Conventional Commits 준수율: 85%

CHANGELOG.md를 생성했습니다. [CHANGELOG 내용 표시]
```

### JSDoc 일괄 생성

```
사용자: src/services 폴더에 JSDoc 추가해줘

Claude: src/services 폴더를 분석합니다...

📋 분석 결과:
- 전체 export 함수: 24개
- 미문서화: 18개 (75%)
- 부분 문서화 (태그 누락): 3개

18개 함수에 JSDoc을 추가하겠습니다. [변경 내용 미리보기]
```

## 에러 처리

| 에러 | 대응 |
|------|------|
| 프로젝트 파일 없음 | 기본 README 템플릿 제공 |
| 기술 스택 감지 불가 | 사용자에게 직접 확인 |
| git 이력 없음 | CHANGELOG 생성 불가 안내 |
| API 엔드포인트 없음 | API 문서 생성 건너뛰기 |
| Conventional Commits 미사용 | 키워드 기반 분류 + 스킬 추천 |

## 관련 문서

- [SKILL.md](SKILL.md) - Claude Code용 상세 지침
- [references/analysis-guide.md](references/analysis-guide.md) - 프로젝트 분석 방법론, 기술 스택 감지 로직
- [references/templates.md](references/templates.md) - README, API 문서, CHANGELOG, JSDoc 출력 템플릿
