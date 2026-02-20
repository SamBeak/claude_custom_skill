# AI Agent Rules Generator

AI 기반 IDE와 코딩 에이전트를 위한 프로젝트별 맞춤형 코딩 규칙 및 가이드라인을 자동으로 생성하는 스킬입니다.

## 개요

코드베이스를 분석하여 Cursor, Windsurf, Cline, Continue, Zed, GitHub Copilot 등 다양한 AI 코딩 도구를 위한 프로젝트 특화 설정 파일을 자동 생성합니다.

## 주요 기능

- **자동 IDE 감지**: 프로젝트에 존재하는 설정 파일을 통해 사용 중인 IDE를 자동으로 감지
- **코드베이스 분석**: 기존 코드의 패턴, 컨벤션, 아키텍처를 분석하여 규칙 생성
- **다양한 플랫폼 지원**:
  - Cursor (`.cursorrules`)
  - Windsurf (`.windsurfrules`)
  - Cline/Claude Code (`.clinerules`)
  - Continue (`.continuerules`)
  - Zed (`.zed/settings.json`)
  - GitHub Copilot (`.github/copilot-instructions.md`)
  - 범용 형식 (`AI_CODING_GUIDE.md`)

## 사용 사례

다음과 같은 요청 시 이 스킬이 활성화됩니다:

1. `.cursorrules`, `.windsurfrules` 등 AI 에이전트 설정 파일 생성
2. 프로젝트별 코딩 가이드라인 문서화
3. 코드 스타일 및 아키텍처 패턴 정의
4. AI 페어 프로그래밍을 위한 컨텍스트 설정
5. AI 코드 생성을 위한 커스텀 인스트럭션 작성

## 생성되는 규칙 카테고리

### 1. 코드 스타일 및 포매팅
- 들여쓰기 방식 (탭/스페이스)
- 네이밍 컨벤션
- 따옴표 스타일, 세미콜론 사용
- 라인 길이, 포매팅 규칙

### 2. 아키텍처 패턴
- 레이어 구조 (MVC, Clean Architecture 등)
- 파일 조직 구조
- 의존성 주입 패턴
- 에러 핸들링 방식

### 3. 프레임워크별 규칙
- Spring Boot 컨벤션
- React/Vue 패턴
- 데이터베이스 쿼리 패턴
- API 설계 표준

### 4. 프로젝트 제약사항
- 기술 스택 제한사항
- 성능 요구사항
- 보안 가이드라인
- 테스트 요구사항

### 5. 도메인 지식
- 비즈니스 로직 패턴
- 데이터베이스 스키마 관계
- API 엔드포인트 컨벤션
- 공통 워크플로우

## 작업 흐름

### 코드베이스 분석을 통한 생성

1. 대표적인 파일 10-15개 샘플링
2. 패턴 추출 (Controller/Service 레이어, DTO/VO 사용 등)
3. 컨벤션 식별 (URL 패턴, 응답 포맷, 데이터베이스 네이밍 등)
4. 아키텍처 문서화 (패키지 구조, 레이어 책임 등)
5. 규칙 파일 생성

### 사용자 명세를 통한 생성

1. 프레임워크/스택, 특정 패턴, 피해야 할 실수 등 질문
2. 템플릿 선택
3. 요구사항에 맞게 커스터마이징
4. 규칙 파일 생성

## 참고 문서

- [SKILL.md](SKILL.md) - 스킬 전체 문서
- [references/templates.md](references/templates.md) - 플랫폼별 템플릿
- [references/analysis-guide.md](references/analysis-guide.md) - 코드베이스 분석 가이드

## 사용 예시

```
사용자: "Cursor를 위한 프로젝트 규칙 파일을 만들어줘"
→ 코드베이스를 분석하여 .cursorrules 파일 생성

사용자: "Spring Boot 프로젝트를 위한 AI 코딩 가이드를 만들어줘"
→ IDE를 자동 감지하거나 선택 후 해당 설정 파일 생성
```

## 베스트 프랙티스

- **구체적으로 작성**: 프로젝트의 실제 코드 예시 포함
- **이유 설명**: 컨텍스트는 AI가 더 나은 결정을 내리는 데 도움
- **안티 패턴 포함**: 하지 말아야 할 것도 명시
- **지속적 업데이트**: 프로젝트 진화에 따라 규칙도 업데이트
- **생성 코드 테스트**: AI가 규칙을 잘 따르는지 검증
