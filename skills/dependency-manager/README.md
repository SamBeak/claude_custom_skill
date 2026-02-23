# Dependency Manager

## 스킬 소개

**Dependency Manager**는 Claude Code 커스텀 스킬로, 프로젝트의 의존성을 종합적으로 분석하여 오래된 패키지를 탐지하고, 메이저 업그레이드 시 마이그레이션 가이드를 제공하며, 라이선스 호환성을 검증하고, 우선순위 기반 업그레이드 플랜을 생성합니다.

### 주요 기능

- **패키지 매니저 자동 감지**: package.json, requirements.txt, pom.xml, go.mod, Cargo.toml, composer.json, Gemfile 등 7개 이상의 패키지 매니저 지원
- **오래된 패키지 분류**: SemVer 기준 patch/minor/major 3단계 분류
- **Breaking Change 영향 분석**: 메이저 업그레이드 대상에 대해 프로젝트 내 사용 패턴 분석 및 영향 범위 산정
- **마이그레이션 가이드 생성**: 주요 패키지(React, Express, Django 등)의 버전별 마이그레이션 절차 제공
- **라이선스 호환성 검증**: MIT/Apache/GPL/AGPL 등 라이선스 간 호환성 매트릭스 기반 검증
- **우선순위 기반 업그레이드 플랜**: 보안(CVE), 호환성, 영향도, 최신성을 종합한 P0~P3 우선순위 생성
- **Security Scanner 연동**: CVE 탐지 결과를 실행 가능한 업그레이드 경로로 변환
- **Test Coverage Analyzer 연동**: 업그레이드 전 테스트 커버리지 확인 및 보강 권장

---

## 지원 패키지 매니저

| 언어 | 매니페스트 파일 | 패키지 매니저 | Lock 파일 |
|------|---------------|-------------|----------|
| JavaScript/TypeScript | `package.json` | npm, yarn, pnpm | `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml` |
| Python | `requirements.txt`, `Pipfile`, `pyproject.toml` | pip, pipenv, poetry | `Pipfile.lock`, `poetry.lock` |
| Java | `pom.xml` | Maven | - |
| Java/Kotlin | `build.gradle`, `build.gradle.kts` | Gradle | `gradle.lockfile` |
| Go | `go.mod` | go modules | `go.sum` |
| Rust | `Cargo.toml` | Cargo | `Cargo.lock` |
| PHP | `composer.json` | Composer | `composer.lock` |
| Ruby | `Gemfile` | Bundler | `Gemfile.lock` |

---

## 사용 예시

### 예시 1: 오래된 패키지 확인

사용자 요청:
> "의존성 업데이트 확인해줘"

분석 결과:
```
============================================================
          의존성 분석 보고서
============================================================

프로젝트: my-app
패키지 매니저: npm (Node.js)
분석 일시: 2026-02-23 14:30
총 의존성: 42개 (직접: 18개, 개발: 24개)

------------------------------------------------------------
오래된 패키지 (15건)
------------------------------------------------------------

[Major] 3건
  express        4.18.2  →  5.0.1   (Breaking Change 포함)
  react          18.2.0  →  19.1.0  (Breaking Change 포함)
  typescript     4.9.5   →  5.7.3   (Breaking Change 포함)

[Minor] 5건
  axios          1.6.2   →  1.7.9
  lodash         4.17.20 →  4.17.21
  jest           29.6.0  →  29.7.0
  eslint         8.50.0  →  8.57.0
  dotenv         16.3.0  →  16.4.7

[Patch] 7건
  cors           2.8.4   →  2.8.5
  helmet         7.1.0   →  7.1.1
  uuid           9.0.0   →  9.0.1
  ...
```

### 예시 2: 라이선스 호환성 확인

사용자 요청:
> "프로젝트 라이선스 호환성 확인해줘"

분석 결과:
```
============================================================
          라이선스 호환성 보고서
============================================================

프로젝트 라이선스: MIT

------------------------------------------------------------
라이선스 충돌 (2건)
------------------------------------------------------------

[Critical] 라이선스 비호환
  패키지: some-gpl-lib@2.1.0
  라이선스: GPL-3.0
  영향: MIT 프로젝트에서 GPL-3.0 의존성 사용 불가
  권장: GPL 미사용 대안 패키지로 교체

[High] 라이선스 미명시
  패키지: unknown-lib@1.0.0
  라이선스: UNLICENSED
  영향: 법적 리스크 불확실
  권장: 패키지 관리자에게 라이선스 명시 요청 또는 대안 검토
```

### 예시 3: 메이저 업그레이드 마이그레이션 가이드

사용자 요청:
> "React 19 마이그레이션 가이드 알려줘"

분석 결과:
```
============================================================
          마이그레이션 가이드: react 18 → 19
============================================================

영향 분석:
  영향받는 파일: 23개
  Deprecated API 사용: 5건
  삭제된 API 사용: 2건
  영향도: High

------------------------------------------------------------
Breaking Changes
------------------------------------------------------------

1. ReactDOM.render() → createRoot()
   영향 파일: src/index.tsx
   수정 방법:
     수정 전: ReactDOM.render(<App />, document.getElementById('root'))
     수정 후: createRoot(document.getElementById('root')).render(<App />)

2. Deprecated lifecycle 메서드 제거
   영향 파일: src/components/Legacy.tsx 외 4건
   ...

------------------------------------------------------------
마이그레이션 단계
------------------------------------------------------------

Phase 1: 사전 준비
  - [ ] 현재 테스트 통과 확인
  - [ ] Deprecated API 사전 교체

Phase 2: 업그레이드 실행
  - [ ] npm install react@19 react-dom@19
  - [ ] Breaking Change 코드 수정

Phase 3: 검증
  - [ ] 전체 테스트 실행
  - [ ] E2E 테스트 실행
```

---

## 에러 처리

| 에러 상황 | 대응 |
|----------|------|
| 매니페스트 파일 미존재 | 지원 매니페스트 파일 목록 안내 |
| 패키지 매니저 미설치 | 설치 가이드 제공, 수동 분석 수행 |
| Lock 파일 미존재 | 매니페스트 기반 분석 전환, lock 파일 생성 권장 |
| 네트워크 오류 | 로컬 캐시 데이터 활용 |
| Private 레지스트리 인증 실패 | 인증 설정 안내, 공개 패키지만 분석 |
| 의존성 >1000개 | 직접 의존성만 우선 분석 |
| SemVer 미준수 패키지 | 수동 확인 안내 |
| Monorepo 구조 | 워크스페이스 단위 분리 분석 |

---

## 관련 문서

- [SKILL.md](SKILL.md) - 스킬 정의 및 상세 워크플로우
- [references/analysis-guide.md](references/analysis-guide.md) - 분석 방법론, 버전 비교 알고리즘, 패키지 매니저별 심화 가이드
- [references/templates.md](references/templates.md) - 보고서 템플릿, 마이그레이션 가이드 템플릿, 라이선스 보고서 형식
