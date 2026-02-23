---
name: doc-generator
description: 프로젝트 코드베이스를 분석하여 README.md, API 문서, CHANGELOG, JSDoc/docstring 등 문서를 자동 생성하고 갱신하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) 문서 생성해줘, (2) README 만들어줘, (3) API 문서 생성, (4) CHANGELOG 생성, (5) JSDoc 추가해줘, (6) docstring 생성, (7) 문서 업데이트, (8) 프로젝트 문서화.
---

# Doc Generator

프로젝트 코드베이스를 정적 분석하여 README.md, API 문서(OpenAPI/Swagger), CHANGELOG.md, JSDoc/docstring 등을 자동 생성하고 기존 문서를 갱신합니다.

## Quick Start

사용자가 문서 생성을 요청하면 다음 워크플로우를 실행합니다:

1. **프로젝트 분석**:
   ```bash
   # 프로젝트 구조 파악
   ls -la
   # 기술 스택 감지
   ls package.json requirements.txt pom.xml go.mod Cargo.toml composer.json Gemfile 2>/dev/null
   # 기존 문서 확인
   ls README.md CHANGELOG.md docs/ 2>/dev/null
   ```

2. **문서 유형 결정**: 사용자 요청 + 프로젝트 상태를 기반으로 생성할 문서 유형 결정

3. **코드베이스 분석**: 프로젝트 구조, 기술 스택, 의존성, 엔트리 포인트, 공개 API 분석

4. **문서 생성/갱신**: [문서 유형별 생성 로직](#문서-유형별-생성-로직)에 따라 생성

5. **사용자 확인**: 초안을 보여주고 수정 요청을 반영한 후 파일 작성

## 트리거 조건

다음 사용자 요청 시 이 스킬을 활성화합니다:

- "문서 생성해줘" / "문서화해줘" / "문서 만들어줘"
- "README 만들어줘" / "README 생성" / "README 업데이트"
- "API 문서 생성" / "Swagger 문서" / "OpenAPI 스펙"
- "CHANGELOG 생성" / "변경 이력 만들어줘"
- "JSDoc 추가해줘" / "docstring 생성해줘" / "주석 추가"
- "프로젝트 문서화" / "documentation"
- "릴리스 노트 작성" / "release notes"

## 문서 유형별 생성 로직

### 1. README.md 생성/갱신

프로젝트 루트의 README.md를 생성하거나 기존 내용을 갱신합니다.

#### 프로젝트 정보 자동 감지

| 감지 항목 | 감지 소스 | 설명 |
|-----------|-----------|------|
| 프로젝트 이름 | `package.json`, 디렉토리 이름, `setup.py`, `pom.xml` | 프로젝트 식별자 |
| 설명 | `package.json#description`, `setup.py`, `pyproject.toml` | 프로젝트 한 줄 요약 |
| 기술 스택 | 파일 확장자, 프레임워크 설정 파일, 의존성 목록 | 사용 언어/프레임워크 |
| 설치 방법 | 패키지 매니저 파일, Dockerfile, Makefile | 의존성 설치 명령 |
| 실행 방법 | `package.json#scripts`, Makefile, Procfile, `main.*` 파일 | 앱 실행 명령 |
| 테스트 방법 | 테스트 설정 파일 (jest.config, pytest.ini 등) | 테스트 실행 명령 |
| 라이선스 | LICENSE 파일, package.json#license | 라이선스 유형 |
| 디렉토리 구조 | 프로젝트 루트의 디렉토리 트리 | 주요 폴더 역할 설명 |

#### 기술 스택 감지 규칙

```bash
# 프레임워크 감지
grep -l "next" package.json 2>/dev/null       # Next.js
grep -l "react" package.json 2>/dev/null      # React
grep -l "vue" package.json 2>/dev/null        # Vue.js
grep -l "express" package.json 2>/dev/null    # Express
grep -l "fastify" package.json 2>/dev/null    # Fastify
grep -l "nestjs" package.json 2>/dev/null     # NestJS
ls manage.py 2>/dev/null                       # Django
ls app.py 2>/dev/null                          # Flask
grep -l "spring" pom.xml 2>/dev/null          # Spring Boot
ls go.mod 2>/dev/null                          # Go
ls Cargo.toml 2>/dev/null                      # Rust
```

#### README 섹션 구성

README.md는 다음 섹션으로 구성합니다 (해당하는 경우에만 포함):

1. **프로젝트 제목 + 배지** (빌드 상태, 버전, 라이선스)
2. **소개**: 프로젝트 목적과 주요 기능 (1-3문장)
3. **주요 기능**: 핵심 기능 불릿 목록
4. **기술 스택**: 사용 언어, 프레임워크, 주요 라이브러리
5. **시작하기** (Getting Started):
   - 필수 요건 (Prerequisites)
   - 설치 (Installation)
   - 환경 설정 (Configuration) — `.env.example` 기반
   - 실행 (Running)
6. **프로젝트 구조**: 주요 디렉토리와 역할 설명
7. **API 문서**: API 엔드포인트 요약 (API 프로젝트인 경우)
8. **테스트**: 테스트 실행 방법
9. **배포**: 배포 방법 (CI/CD 설정이 있는 경우)
10. **기여 가이드**: 기여 방법 (오픈소스 프로젝트인 경우)
11. **라이선스**: 라이선스 정보

#### 기존 README 갱신 규칙

기존 README.md가 있는 경우:
- 기존 섹션 구조를 유지하며 내용만 갱신
- 수동 작성된 것으로 보이는 섹션(프로젝트 소개, 기여 가이드 등)은 보존
- 자동 감지 가능한 섹션(기술 스택, 디렉토리 구조)만 업데이트
- 변경 전후를 diff 형태로 보여주고 사용자 확인 후 적용

### 2. API 문서 생성

프로젝트의 API 엔드포인트를 분석하여 문서를 생성합니다.

#### 프레임워크별 API 감지

| 프레임워크 | 감지 패턴 | 파일 위치 |
|-----------|-----------|-----------|
| Express/Fastify | `app.get()`, `router.post()` 등 | `**/routes/**`, `**/controllers/**` |
| NestJS | `@Get()`, `@Post()` 데코레이터 | `**/*.controller.ts` |
| Django REST | `@api_view`, `class ViewSet` | `**/views.py`, `**/viewsets.py` |
| Flask | `@app.route()`, `@blueprint.route()` | `**/*.py` |
| Spring Boot | `@GetMapping`, `@PostMapping` 등 | `**/*Controller.java` |
| FastAPI | `@app.get()`, `@router.post()` | `**/*.py` |
| Go (net/http) | `http.HandleFunc()`, gorilla mux | `**/*.go` |

#### API 문서 출력 형식

각 엔드포인트에 대해 다음 정보를 추출합니다:

- **HTTP 메서드 + 경로**: `GET /api/users/:id`
- **설명**: 함수/메서드명, 주석, 데코레이터 정보에서 추출
- **요청 파라미터**: path params, query params, body schema
- **응답 형식**: 응답 타입, 상태 코드
- **인증 여부**: 인증 미들웨어 적용 여부

출력 형식은 Markdown 테이블 형식을 기본으로 합니다. 상세 템플릿은 [references/templates.md](references/templates.md)를 참조하세요.

### 3. CHANGELOG.md 생성

git 커밋 히스토리를 분석하여 CHANGELOG.md를 생성합니다.

#### 커밋 분석

```bash
# 태그 목록 확인
git tag --sort=-v:refname

# 최신 태그 이후 커밋
git log $(git describe --tags --abbrev=0 2>/dev/null || echo "HEAD~50")..HEAD --pretty=format:"%H|%s|%an|%ai" --no-merges

# 태그가 없는 경우 전체 커밋
git log --pretty=format:"%H|%s|%an|%ai" --no-merges
```

#### Conventional Commits 파싱 규칙

커밋 메시지가 Conventional Commits 형식을 따르는 경우:

| 타입 | CHANGELOG 섹션 | 설명 |
|------|----------------|------|
| `feat` | Added | 새로운 기능 |
| `fix` | Fixed | 버그 수정 |
| `docs` | Documentation | 문서 변경 |
| `style` | (제외) | 코드 스타일 변경 |
| `refactor` | Changed | 리팩토링 |
| `perf` | Performance | 성능 개선 |
| `test` | (제외) | 테스트 추가/수정 |
| `build` | Build | 빌드 시스템 변경 |
| `ci` | (제외) | CI 설정 변경 |
| `chore` | (제외) | 기타 |
| `BREAKING CHANGE` | Breaking Changes | 하위 호환성 깨는 변경 |

#### 비 Conventional Commits 처리

커밋 메시지가 Conventional Commits 형식이 아닌 경우:
- 커밋 메시지 키워드 분석: "add" → Added, "fix" → Fixed, "update" → Changed, "remove" → Removed
- 분류 불가능한 커밋은 "Other Changes" 섹션에 포함
- 사용자에게 `git-conventional-commits` 스킬 사용을 권장

#### CHANGELOG 형식

[Keep a Changelog](https://keepachangelog.com/ko/1.1.0/) 형식을 따릅니다:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Added
- {feat 커밋 내용}

### Fixed
- {fix 커밋 내용}

### Changed
- {refactor 커밋 내용}

## [1.2.0] - 2024-01-15

### Added
- ...
```

### 4. JSDoc / Docstring 일괄 생성

미문서화된 공개(public) 함수, 클래스, 메서드에 JSDoc/docstring을 추가합니다.

#### 대상 탐지

미문서화 대상을 다음 기준으로 식별합니다:

| 언어 | 대상 | 문서화 여부 판단 |
|------|------|-----------------|
| TypeScript/JavaScript | `export function`, `export class`, `export const` | 직전에 `/** */` 없음 |
| Python | `def`, `class` (언더스코어 미시작) | `"""docstring"""` 없음 |
| Java | `public` 메서드/클래스 | 직전에 `/** */` Javadoc 없음 |
| Go | 대문자 시작 함수/타입 | 직전에 `//` 주석 없음 |

#### 생성 규칙

- **함수/메서드**: 요약 설명, @param (각 매개변수 타입 + 설명), @returns (반환 타입 + 설명), @throws (예외 설명)
- **클래스**: 요약 설명, 주요 책임, @example 사용 예시
- **인터페이스/타입**: 요약 설명, 각 필드 설명
- 이미 존재하는 문서는 보존하고, 빠진 태그만 보완

#### 언어별 형식

**TypeScript/JavaScript (JSDoc)**:
```typescript
/**
 * 사용자 정보를 조회합니다.
 *
 * @param userId - 조회할 사용자의 고유 ID
 * @param options - 조회 옵션
 * @returns 사용자 정보 객체
 * @throws {NotFoundError} 사용자가 존재하지 않는 경우
 */
export async function getUser(userId: string, options?: GetUserOptions): Promise<User> {
```

**Python (docstring)**:
```python
def get_user(user_id: str, options: dict = None) -> User:
    """사용자 정보를 조회합니다.

    Args:
        user_id: 조회할 사용자의 고유 ID
        options: 조회 옵션 딕셔너리

    Returns:
        User: 사용자 정보 객체

    Raises:
        NotFoundError: 사용자가 존재하지 않는 경우
    """
```

**Java (Javadoc)**:
```java
/**
 * 사용자 정보를 조회합니다.
 *
 * @param userId 조회할 사용자의 고유 ID
 * @param options 조회 옵션
 * @return 사용자 정보 객체
 * @throws NotFoundException 사용자가 존재하지 않는 경우
 */
public User getUser(String userId, GetUserOptions options) {
```

**Go (GoDoc)**:
```go
// GetUser 사용자 정보를 조회합니다.
// userId는 조회할 사용자의 고유 ID입니다.
// 사용자가 존재하지 않으면 ErrNotFound를 반환합니다.
func GetUser(userId string, options *GetUserOptions) (*User, error) {
```

## 프로젝트 구조 분석

### 디렉토리 트리 생성

```bash
# 주요 디렉토리만 표시 (depth 제한)
find . -maxdepth 3 -type d \
  -not -path '*/node_modules/*' \
  -not -path '*/.git/*' \
  -not -path '*/dist/*' \
  -not -path '*/build/*' \
  -not -path '*/__pycache__/*' \
  -not -path '*/.next/*' \
  -not -path '*/target/*' \
  -not -path '*/vendor/*' | sort
```

### 제외 대상

디렉토리 트리 및 분석에서 제외하는 경로:

- `node_modules/`, `vendor/`, `.git/`
- `dist/`, `build/`, `.next/`, `target/`, `out/`
- `__pycache__/`, `.mypy_cache/`, `.pytest_cache/`
- `coverage/`, `.nyc_output/`
- `.idea/`, `.vscode/` (IDE 설정)

## 환경 설정 문서화

### .env.example 기반 문서화

`.env.example` 또는 `.env.sample`이 존재하면 환경 변수 목록을 문서화합니다:

```bash
# .env 파일에서 변수 추출
grep -E '^[A-Z_]+=.*' .env.example 2>/dev/null | sed 's/=.*//'
```

각 환경 변수에 대해:
- **변수명**: 환경 변수 이름
- **필수 여부**: 값이 비어있으면 필수, 기본값이 있으면 선택
- **설명**: 변수명에서 유추 또는 주석에서 추출
- **예시 값**: `.env.example`의 값 (시크릿은 마스킹)

## git-conventional-commits 연동

이 스킬은 `git-conventional-commits` 스킬과 연동하여 CHANGELOG 품질을 높입니다:

1. 커밋이 Conventional Commits 형식이면 → 자동으로 정확한 카테고리 분류
2. 커밋이 비표준 형식이면 → 키워드 기반 분류 + 향후 `git-conventional-commits` 스킬 사용 권장
3. CHANGELOG 생성 후 → PR 리뷰 시 `pr-review-checklist`의 "문서 미갱신" 경고 해소

## 워크플로우 상세

```
1. [요청 분석]
   ├─ "README" → README.md 생성/갱신
   ├─ "API 문서" → API 엔드포인트 문서화
   ├─ "CHANGELOG" → git log 기반 CHANGELOG 생성
   ├─ "JSDoc" / "docstring" → 코드 주석 일괄 생성
   └─ "문서화" (일반) → 프로젝트 상태 분석 후 필요한 문서 추천

2. [프로젝트 분석]
   ├─ 기술 스택 감지 (패키지 매니저, 프레임워크, 언어)
   ├─ 디렉토리 구조 파악
   ├─ 기존 문서 현황 확인
   └─ 공개 API / 엔드포인트 식별

3. [문서 생성]
   ├─ 해당 문서 유형의 템플릿 로드
   ├─ 프로젝트 분석 결과로 템플릿 채움
   ├─ 기존 문서가 있으면 diff 계산
   └─ 초안 생성

4. [사용자 확인]
   ├─ 초안 또는 diff 표시
   ├─ 수정 요청 반영
   └─ 최종 확인 후 파일 작성/갱신
```

## 에러 처리

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | 프로젝트 파일 없음 | 루트 디렉토리가 비어있음 | 기본 README 템플릿 제공 |
| 2 | 기술 스택 감지 불가 | 패키지 매니저 파일 없음 | 사용자에게 기술 스택 직접 확인 |
| 3 | API 엔드포인트 없음 | 라우터/컨트롤러 파일 미발견 | API 문서 생성 건너뛰기 안내 |
| 4 | git 이력 없음 | `git log` 빈 출력 | CHANGELOG 생성 불가 안내, 수동 작성 가이드 제공 |
| 5 | Conventional Commits 미사용 | 커밋 메시지 형식 불일치 | 키워드 기반 분류 + 스킬 추천 |
| 6 | 기존 README 충돌 | 수동 작성 내용 감지 | 수동 섹션 보존, 자동 섹션만 갱신 |
| 7 | 대규모 프로젝트 | 파일 수 > 1000 | 핵심 엔트리 포인트 위주 분석 |
| 8 | 지원하지 않는 언어 | 확장자 미매칭 | 범용 README만 생성, JSDoc 생성 건너뛰기 |

## Best Practices

1. **점진적 문서화**: 한 번에 모든 문서를 생성하려 하지 말고, 가장 필요한 것부터
2. **기존 문서 존중**: 수동 작성된 내용은 자동 갱신하지 않음
3. **코드와 문서 동기화**: 코드 변경 시 관련 문서도 함께 갱신
4. **Keep a Changelog 준수**: CHANGELOG는 사람이 읽기 편한 형식 유지
5. **과도한 문서 지양**: 자명한 코드에 불필요한 주석을 추가하지 않음
6. **환경 변수 문서화 필수**: `.env.example`과 README의 환경 설정 섹션을 일치시킴
7. **예시 코드 포함**: API 문서에는 요청/응답 예시를 반드시 포함

## Analysis Guide

프로젝트 분석 방법론, 기술 스택 감지 로직, API 엔드포인트 추출 방법에 대한 상세 가이드는 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

## Templates

README 템플릿, API 문서 템플릿, CHANGELOG 템플릿, JSDoc/docstring 형식 등은 [references/templates.md](references/templates.md)를 참조하세요.
