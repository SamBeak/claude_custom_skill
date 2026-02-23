# Doc Generator 템플릿

문서 생성 시 사용하는 출력 형식 템플릿을 정의합니다.

## 목차

- [1. README.md 템플릿](#1-readmemd-템플릿)
- [2. API 문서 템플릿](#2-api-문서-템플릿)
- [3. CHANGELOG 템플릿](#3-changelog-템플릿)
- [4. JSDoc/Docstring 템플릿](#4-jsdocdocstring-템플릿)
- [5. 환경 변수 문서 템플릿](#5-환경-변수-문서-템플릿)
- [6. 에러 메시지 템플릿](#6-에러-메시지-템플릿)

---

## 1. README.md 템플릿

### 전체 README 템플릿

```markdown
# {PROJECT_NAME}

{PROJECT_DESCRIPTION}

## 주요 기능

- {FEATURE_1}
- {FEATURE_2}
- {FEATURE_3}

## 기술 스택

| 구분 | 기술 |
|------|------|
| Language | {LANGUAGE} |
| Framework | {FRAMEWORK} |
| Database | {DATABASE} |
| Testing | {TEST_FRAMEWORK} |
| CI/CD | {CICD} |

## 시작하기

### 필수 요건

- {RUNTIME} {VERSION} 이상
- {PACKAGE_MANAGER}
- {OTHER_PREREQUISITES}

### 설치

```bash
# 저장소 클론
git clone {REPO_URL}
cd {PROJECT_NAME}

# 의존성 설치
{INSTALL_COMMAND}
```

### 환경 설정

```bash
# 환경 변수 설정
cp .env.example .env
# .env 파일을 열어 필요한 값을 설정하세요
```

{ENV_VARS_TABLE}

### 실행

```bash
# 개발 서버
{DEV_COMMAND}

# 프로덕션 빌드
{BUILD_COMMAND}

# 프로덕션 실행
{START_COMMAND}
```

## 프로젝트 구조

```
{DIRECTORY_TREE}
```

{DIRECTORY_DESCRIPTIONS}

## 테스트

```bash
# 전체 테스트
{TEST_COMMAND}

# 특정 테스트
{TEST_SPECIFIC_COMMAND}
```

## API 문서

{API_SUMMARY_TABLE}

자세한 내용은 [API 문서]({API_DOC_LINK})를 참조하세요.

## 배포

{DEPLOYMENT_INSTRUCTIONS}

## 라이선스

이 프로젝트는 [{LICENSE_TYPE}]({LICENSE_LINK}) 라이선스 하에 배포됩니다.
```

### 간소화 README 템플릿 (소규모 프로젝트)

```markdown
# {PROJECT_NAME}

{PROJECT_DESCRIPTION}

## 설치 및 실행

```bash
{INSTALL_COMMAND}
{DEV_COMMAND}
```

## 라이선스

{LICENSE_TYPE}
```

---

## 2. API 문서 템플릿

### API 엔드포인트 요약 테이블

```markdown
## API 엔드포인트

| 메서드 | 경로 | 설명 | 인증 |
|--------|------|------|------|
| {METHOD} | `{PATH}` | {DESCRIPTION} | {AUTH_REQUIRED} |
```

### API 엔드포인트 상세 템플릿

```markdown
### {METHOD} {PATH}

{DESCRIPTION}

**인증**: {AUTH_REQUIRED}

**요청 파라미터**:

| 파라미터 | 위치 | 타입 | 필수 | 설명 |
|---------|------|------|------|------|
| {PARAM_NAME} | {path/query/body} | `{TYPE}` | {REQUIRED} | {DESCRIPTION} |

**요청 Body**:

```json
{REQUEST_BODY_EXAMPLE}
```

**응답** (`{STATUS_CODE}`):

```json
{RESPONSE_EXAMPLE}
```

**에러 응답**:

| 상태 코드 | 설명 |
|-----------|------|
| {ERROR_CODE} | {ERROR_DESCRIPTION} |
```

### API 그룹 템플릿

```markdown
## {GROUP_NAME} API

{GROUP_DESCRIPTION}

Base URL: `{BASE_PATH}`

{ENDPOINT_LIST}
```

---

## 3. CHANGELOG 템플릿

### 전체 CHANGELOG 템플릿

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/ko/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- {FEAT_DESCRIPTION} ({COMMIT_HASH_SHORT})

### Fixed
- {FIX_DESCRIPTION} ({COMMIT_HASH_SHORT})

### Changed
- {CHANGE_DESCRIPTION} ({COMMIT_HASH_SHORT})

## [{VERSION}] - {DATE}

### Added
- {FEAT_DESCRIPTION}

### Fixed
- {FIX_DESCRIPTION}

### Changed
- {CHANGE_DESCRIPTION}

### Removed
- {REMOVE_DESCRIPTION}

### Breaking Changes
- {BREAKING_DESCRIPTION}
```

### 릴리스 노트 템플릿

```markdown
# {PROJECT_NAME} v{VERSION} 릴리스 노트

릴리스 날짜: {DATE}

## 주요 변경사항

{HIGHLIGHTS}

## 상세 변경 내역

### 새로운 기능
- {FEAT_DESCRIPTION}

### 버그 수정
- {FIX_DESCRIPTION}

### 개선사항
- {IMPROVEMENT_DESCRIPTION}

### Breaking Changes

{BREAKING_CHANGES_DETAIL}

## 업그레이드 가이드

{UPGRADE_INSTRUCTIONS}

## 기여자

{CONTRIBUTORS}
```

---

## 4. JSDoc/Docstring 템플릿

### TypeScript/JavaScript JSDoc

```typescript
/**
 * {FUNCTION_DESCRIPTION}
 *
 * @param {PARAM_NAME} - {PARAM_DESCRIPTION}
 * @returns {RETURN_DESCRIPTION}
 * @throws {{ERROR_TYPE}} {ERROR_DESCRIPTION}
 *
 * @example
 * ```typescript
 * {USAGE_EXAMPLE}
 * ```
 */
```

### Python Docstring (Google 스타일)

```python
"""
{FUNCTION_DESCRIPTION}

Args:
    {PARAM_NAME}: {PARAM_DESCRIPTION}

Returns:
    {RETURN_TYPE}: {RETURN_DESCRIPTION}

Raises:
    {ERROR_TYPE}: {ERROR_DESCRIPTION}

Example:
    >>> {USAGE_EXAMPLE}
"""
```

### Java Javadoc

```java
/**
 * {FUNCTION_DESCRIPTION}
 *
 * @param {PARAM_NAME} {PARAM_DESCRIPTION}
 * @return {RETURN_DESCRIPTION}
 * @throws {ERROR_TYPE} {ERROR_DESCRIPTION}
 * @since {VERSION}
 */
```

### Go GoDoc

```go
// {FUNCTION_NAME} {FUNCTION_DESCRIPTION}
//
// {PARAM_NAME}은 {PARAM_DESCRIPTION}입니다.
// {ERROR_CONDITION}인 경우 {ERROR_TYPE}을 반환합니다.
```

### 클래스 JSDoc 템플릿

```typescript
/**
 * {CLASS_DESCRIPTION}
 *
 * {CLASS_RESPONSIBILITY}
 *
 * @example
 * ```typescript
 * const instance = new {CLASS_NAME}({CONSTRUCTOR_ARGS});
 * {USAGE_EXAMPLE}
 * ```
 */
```

### 인터페이스/타입 JSDoc 템플릿

```typescript
/**
 * {TYPE_DESCRIPTION}
 *
 * @property {PROPERTY_NAME} - {PROPERTY_DESCRIPTION}
 */
```

---

## 5. 환경 변수 문서 템플릿

```markdown
### 환경 변수

| 변수명 | 설명 | 필수 | 기본값 | 예시 |
|--------|------|------|--------|------|
| `{VAR_NAME}` | {DESCRIPTION} | {REQUIRED} | {DEFAULT} | `{EXAMPLE}` |
```

### 환경 변수 상세 템플릿

```markdown
### 환경 변수 상세

#### `{VAR_NAME}`

- **설명**: {DESCRIPTION}
- **필수**: {REQUIRED}
- **타입**: {TYPE} (string, number, boolean, url)
- **기본값**: `{DEFAULT}`
- **예시**: `{EXAMPLE}`
- **참고**: {NOTES}
```

---

## 6. 에러 메시지 템플릿

### 정보 메시지

```
ℹ️ {MESSAGE}
```

### 성공 메시지

```
✅ {DOCUMENT_TYPE} 문서가 성공적으로 생성되었습니다.
   경로: {FILE_PATH}
   크기: {LINE_COUNT}줄
```

### 경고 메시지

```
⚠️ {WARNING_MESSAGE}
   - {DETAIL_1}
   - {DETAIL_2}
   권장: {RECOMMENDATION}
```

### 에러 메시지

```
❌ {ERROR_MESSAGE}
   원인: {CAUSE}
   해결: {RESOLUTION}
```

### 문서 갱신 diff 표시

```
📝 README.md 갱신 사항:

  [기술 스택] 섹션 업데이트:
  - TypeScript 4.9 → TypeScript 5.3
  + Vitest 추가

  [디렉토리 구조] 섹션 업데이트:
  + src/middleware/ 추가

  기존 섹션 보존:
  - 프로젝트 소개 (수동 작성)
  - 기여 가이드 (수동 작성)

이 변경사항을 적용할까요?
```

### 문서화 현황 요약

```
📊 프로젝트 문서화 현황

문서 파일:
  ✅ README.md - 존재 (마지막 수정: {DATE})
  ❌ CHANGELOG.md - 없음
  ✅ API 문서 - docs/api.md 존재
  ⚠️ .env.example - 존재하나 3개 변수 미문서화

코드 문서화:
  전체 public 함수: {TOTAL}개
  문서화 완료: {DOCUMENTED}개 ({PERCENTAGE}%)
  미문서화: {UNDOCUMENTED}개

  미문서화 파일 TOP 5:
  1. src/services/userService.ts - 8개 함수 미문서화
  2. src/utils/helpers.ts - 6개 함수 미문서화
  3. ...

권장 작업:
  1. CHANGELOG.md 생성 (CHANGELOG 생성해줘)
  2. 미문서화 함수에 JSDoc 추가 (JSDoc 추가해줘)
  3. .env.example 변수 문서화
```
