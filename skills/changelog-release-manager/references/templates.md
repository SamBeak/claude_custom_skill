# Changelog Release Manager 템플릿

CHANGELOG, 릴리스 노트, 버전 파일 업데이트, 에러 메시지 등 릴리스 관리에 사용하는 출력 형식 템플릿을 정의합니다.

## 목차

- [1. CHANGELOG 템플릿](#1-changelog-템플릿)
- [2. 릴리스 노트 템플릿](#2-릴리스-노트-템플릿)
- [3. 버전 파일 업데이트 템플릿](#3-버전-파일-업데이트-템플릿)
- [4. Git 태그 및 Release 템플릿](#4-git-태그-및-release-템플릿)
- [5. 모노레포 템플릿](#5-모노레포-템플릿)
- [6. 메시지 템플릿](#6-메시지-템플릿)

---

## 1. CHANGELOG 템플릿

### 전체 CHANGELOG 템플릿

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/ko/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [{VERSION}] - {YYYY-MM-DD}

### Breaking Changes
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))
  - 마이그레이션: {MIGRATION_NOTE}

### Added
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))

### Fixed
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))

### Changed
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))

### Removed
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))

### Performance
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))

### Security
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))

[Unreleased]: {REPO_URL}/compare/v{VERSION}...HEAD
[{VERSION}]: {REPO_URL}/compare/v{PREV_VERSION}...v{VERSION}
```

### 최초 CHANGELOG 템플릿 (태그 없음)

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/ko/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [{VERSION}] - {YYYY-MM-DD}

### Added
- 프로젝트 초기 릴리스
- {FEATURE_1}
- {FEATURE_2}
- {FEATURE_3}

[{VERSION}]: {REPO_URL}/releases/tag/v{VERSION}
```

### CHANGELOG 엔트리 형식

scope가 있는 경우:
```markdown
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))
```

scope가 없는 경우:
```markdown
- {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))
```

Breaking Change 엔트리:
```markdown
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}))
  - 마이그레이션: {MIGRATION_NOTE}
```

이슈 참조가 있는 경우:
```markdown
- **{SCOPE}**: {DESCRIPTION} ([{HASH_SHORT}]({COMMIT_URL}), [#{ISSUE}]({ISSUE_URL}))
```

---

## 2. 릴리스 노트 템플릿

### 전체 릴리스 노트 템플릿

```markdown
# v{VERSION} 릴리스 노트

릴리스 날짜: {YYYY-MM-DD}

## 하이라이트

{HIGHLIGHT_SUMMARY}

{HIGHLIGHT_ITEMS}

## 주요 변경사항

### 새로운 기능
- {FEAT_DESCRIPTION}

### 버그 수정
- {FIX_DESCRIPTION}

### 성능 개선
- {PERF_DESCRIPTION}

### 보안
- {SECURITY_DESCRIPTION}

## Breaking Changes

### {BREAKING_TITLE}

{BREAKING_DESCRIPTION}

**Before:**
```{LANG}
{BEFORE_CODE}
```

**After:**
```{LANG}
{AFTER_CODE}
```

마이그레이션 가이드: {MIGRATION_LINK}

## 업그레이드 가이드

1. {UPGRADE_STEP_1}
2. {UPGRADE_STEP_2}
3. {UPGRADE_STEP_3}

## 기여자

이번 릴리스에 기여해주신 분들께 감사합니다:

{CONTRIBUTOR_LIST}

## 전체 변경 이력

전체 변경 내역은 [CHANGELOG.md](CHANGELOG.md)를 참조하세요.
```

### 간소화 릴리스 노트 (Breaking Change 없음)

```markdown
# v{VERSION} 릴리스 노트

릴리스 날짜: {YYYY-MM-DD}

## 변경사항

### 새로운 기능
- {FEAT_DESCRIPTION}

### 버그 수정
- {FIX_DESCRIPTION}

### 개선사항
- {IMPROVEMENT_DESCRIPTION}

## 기여자

{CONTRIBUTOR_LIST}
```

### Pre-release 릴리스 노트 템플릿

```markdown
# v{VERSION}-{TYPE}.{NUM} (Pre-release)

릴리스 날짜: {YYYY-MM-DD}

> 이 릴리스는 {TYPE} 버전입니다. 프로덕션 환경에서 사용하지 마세요.

## 변경사항 ({PREV_PRERELEASE} 이후)

- {CHANGE_1}
- {CHANGE_2}

## 알려진 이슈

- {KNOWN_ISSUE_1}
- {KNOWN_ISSUE_2}

## 피드백

이슈 또는 피드백은 [GitHub Issues]({ISSUES_URL})에 남겨주세요.
```

### 하이라이트 항목 형식

기능 하이라이트:
```markdown
- **{FEATURE_NAME}**: {DESCRIPTION}
```

성능 하이라이트 (수치 포함):
```markdown
- **{OPTIMIZATION_NAME}**: {DESCRIPTION} ({BEFORE} → {AFTER})
```

보안 하이라이트:
```markdown
- **보안 패치**: {DESCRIPTION}
```

---

## 3. 버전 파일 업데이트 템플릿

### package.json

변경 전:
```json
{
  "name": "{PACKAGE_NAME}",
  "version": "{OLD_VERSION}",
```

변경 후:
```json
{
  "name": "{PACKAGE_NAME}",
  "version": "{NEW_VERSION}",
```

### pyproject.toml (Poetry)

변경 전:
```toml
[tool.poetry]
name = "{PACKAGE_NAME}"
version = "{OLD_VERSION}"
```

변경 후:
```toml
[tool.poetry]
name = "{PACKAGE_NAME}"
version = "{NEW_VERSION}"
```

### pyproject.toml (PEP 621)

변경 전:
```toml
[project]
name = "{PACKAGE_NAME}"
version = "{OLD_VERSION}"
```

변경 후:
```toml
[project]
name = "{PACKAGE_NAME}"
version = "{NEW_VERSION}"
```

### Cargo.toml

변경 전:
```toml
[package]
name = "{PACKAGE_NAME}"
version = "{OLD_VERSION}"
```

변경 후:
```toml
[package]
name = "{PACKAGE_NAME}"
version = "{NEW_VERSION}"
```

### VERSION 파일

```
{NEW_VERSION}
```

---

## 4. Git 태그 및 Release 템플릿

### 태그 메시지

```
Release v{VERSION}
```

### 태그 생성 명령

```bash
git tag -a v{VERSION} -m "Release v{VERSION}"
```

### 릴리스 커밋 메시지

```
chore: release v{VERSION}
```

또는 body 포함:

```
chore: release v{VERSION}

Release version {VERSION} with the following changes:
- {SUMMARY_1}
- {SUMMARY_2}

See CHANGELOG.md for full details.
```

### GitHub Release 생성 명령

정식 릴리스:
```bash
gh release create v{VERSION} \
  --title "v{VERSION}" \
  --notes-file release-notes.md
```

Pre-release:
```bash
gh release create v{VERSION}-{TYPE}.{NUM} \
  --title "v{VERSION}-{TYPE}.{NUM}" \
  --notes-file release-notes.md \
  --prerelease
```

Draft 릴리스:
```bash
gh release create v{VERSION} \
  --title "v{VERSION}" \
  --notes-file release-notes.md \
  --draft
```

---

## 5. 모노레포 템플릿

### 패키지별 CHANGELOG 헤더

```markdown
# Changelog - {PACKAGE_NAME}

All notable changes to `{PACKAGE_NAME}` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/ko/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
```

### 모노레포 태그 형식

독립 버전:
```bash
git tag -a "{PACKAGE_NAME}@{VERSION}" -m "Release {PACKAGE_NAME}@{VERSION}"
```

통합 버전:
```bash
git tag -a "v{VERSION}" -m "Release v{VERSION}"
```

### 모노레포 릴리스 커밋 메시지

독립 버전:
```
chore({PACKAGE_SCOPE}): release {PACKAGE_NAME}@{VERSION}
```

통합 버전:
```
chore: release v{VERSION}
```

### 모노레포 릴리스 노트 (통합)

```markdown
# v{VERSION} 릴리스 노트

릴리스 날짜: {YYYY-MM-DD}

## 패키지별 변경사항

### {PACKAGE_1} (v{PKG1_VERSION})
- {CHANGE_1}
- {CHANGE_2}

### {PACKAGE_2} (v{PKG2_VERSION})
- {CHANGE_1}
- {CHANGE_2}

## 기여자

{CONTRIBUTOR_LIST}
```

---

## 6. 메시지 템플릿

### 분석 결과 요약

```
릴리스 분석 결과:
- 마지막 태그: {LAST_TAG}
- 새 커밋: {COMMIT_COUNT}개
  - feat: {FEAT_COUNT}개
  - fix: {FIX_COUNT}개
  - refactor: {REFACTOR_COUNT}개
  - 기타: {OTHER_COUNT}개
- Breaking Change: {BREAKING_COUNT}개
- Conventional Commits 준수율: {COMPLIANCE}%

버전 결정: {OLD_VERSION} → {NEW_VERSION} ({BUMP_TYPE} bump)
```

### 릴리스 완료 메시지

```
릴리스가 완료되었습니다.

- 버전: v{VERSION}
- 태그: v{VERSION}
- CHANGELOG: CHANGELOG.md 갱신 완료
- GitHub Release: {RELEASE_URL}

다음 단계:
- 릴리스 노트를 팀에 공유하세요
- 프로덕션 배포를 진행하세요 (필요한 경우)
```

### 릴리스 불필요 메시지

```
마지막 태그({LAST_TAG}) 이후 새 커밋이 없습니다.
릴리스가 필요하지 않습니다.

마지막 릴리스 정보:
- 태그: {LAST_TAG}
- 날짜: {LAST_TAG_DATE}
- 커밋: {LAST_TAG_HASH}
```

### 경고 메시지

커밋되지 않은 변경사항:
```
커밋되지 않은 변경사항이 감지되었습니다:
{GIT_STATUS_OUTPUT}

릴리스를 진행하기 전에 변경사항을 커밋하거나 stash해주세요.
```

버전 불일치:
```
프로젝트 내 버전이 일치하지 않습니다:

- Git 태그: {TAG_VERSION}
- package.json: {PKG_VERSION}
- pyproject.toml: {PY_VERSION}

버전을 통일한 후 릴리스를 다시 시도해주세요.
```

Conventional Commits 미사용:
```
Conventional Commits 형식 준수율이 낮습니다 ({COMPLIANCE}%).

키워드 기반으로 분류를 진행했지만, 정확도가 떨어질 수 있습니다.
향후 커밋 시 git-conventional-commits 스킬 사용을 권장합니다.

분류 결과:
{CLASSIFICATION_SUMMARY}

이 분류가 맞나요? 수정이 필요하면 말씀해주세요.
```

### 에러 메시지

gh CLI 미설치:
```
GitHub CLI(gh)가 설치되어 있지 않습니다.
Git 태그는 생성했지만, GitHub Release는 수동으로 생성해야 합니다.

설치 방법:
- macOS: brew install gh
- Windows: winget install --id GitHub.cli
- Linux: https://github.com/cli/cli/blob/trunk/docs/install_linux.md

설치 후 인증:
  gh auth login
```

테스트 실패:
```
테스트가 실패했습니다. 릴리스를 중단합니다.

실패한 테스트:
{TEST_FAILURE_OUTPUT}

테스트를 수정한 후 다시 릴리스를 시도해주세요.
```

태그 중복:
```
태그 v{VERSION}이 이미 존재합니다.

기존 태그 정보:
- 태그: v{VERSION}
- 날짜: {TAG_DATE}
- 커밋: {TAG_HASH}

다른 버전을 지정하거나, 기존 태그를 삭제한 후 다시 시도해주세요.
```
