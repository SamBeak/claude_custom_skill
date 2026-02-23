# Tech Debt Tracker

## 스킬 소개

**Tech Debt Tracker**는 Claude Code 커스텀 스킬로, 코드베이스에 축적된 기술 부채를 체계적으로 스캔, 분류, 추적합니다. TODO/FIXME 등 부채 마커, deprecated API 사용, 타입 안전성 갭, 코드 건강도를 종합 분석하고, Impact x Effort 매트릭스 기반의 우선순위 분류와 개선 로드맵을 생성합니다.

### 주요 기능

- **부채 마커 스캔**: TODO, FIXME, HACK, XXX, TEMP, WORKAROUND 및 lint 억제 주석 탐지
- **Git blame 기반 연령 분석**: 각 부채 마커의 생성일, 작성자, 경과 일수 추적
- **Deprecated API 감지**: npm deprecated 패키지, Python deprecated stdlib, React/Next.js/Express deprecated API
- **타입 안전성 분석**: TypeScript `any` 사용, @ts-ignore 억제, Python 타입 힌트 누락, Java raw types
- **코드 건강도 메트릭**: 테스트 부채, 문서 부채, 의존성 부채, 보안 부채 종합 평가
- **Impact x Effort 매트릭스**: P1~P4 우선순위 분류 및 스프린트 단위 로드맵 생성
- **다중 스킬 연계**: refactor-advisor, test-coverage-analyzer, security-scanner와 통합 분석

---

## 분석 대상 요약

### 부채 마커

| 카테고리 | 탐지 대상 | 심각도 |
|---------|----------|--------|
| 코드 주석 | TODO, FIXME, HACK, XXX, TEMP, WORKAROUND, BUG | Medium~Critical |
| 어노테이션 | @deprecated, @Deprecated, DeprecationWarning | Medium |
| TS 억제 | @ts-ignore, @ts-expect-error, @ts-nocheck | High |
| Lint 억제 | eslint-disable, prettier-ignore, noqa, noinspection | Medium |
| Java 억제 | @SuppressWarnings | Low~Medium |

### 부채 연령 분류

| 연령 | 분류 | 의미 |
|------|------|------|
| 0~30일 | Fresh | 최근 추가, 빠른 처리 가능 |
| 31~90일 | Aging | 부채 누적 시작, 주의 필요 |
| 91~180일 | Stale | 장기 방치, 즉시 계획 수립 |
| 181~365일 | Chronic | 만성화, 팀 차원 대응 |
| 365일+ | Fossil | 유효성 재검증 필요 |

### 타입 안전성 분석

| 언어 | 분석 항목 | 지표 |
|------|----------|------|
| TypeScript | `any` 사용, @ts-ignore, @ts-nocheck | any 카운트, 억제 비율 |
| Python | 타입 힌트 누락, `# type: ignore` | 힌트 커버리지 % |
| Java | Raw types, @SuppressWarnings("unchecked") | 미제네릭 사용 수 |

---

## 사용 예시

### 예시 1: 전체 기술 부채 분석

요청:
```
기술 부채 분석해줘
```

결과 요약 예시:
```
=== 기술 부채 분석 보고서 ===

부채 마커: 총 147건
  - TODO: 82건 (56%)
  - FIXME: 31건 (21%)
  - HACK: 12건 (8%)
  - 기타: 22건 (15%)

부채 연령:
  - 90일 이상 방치: 45건 (31%)
  - 최다 방치: src/legacy/parser.js:42 (387일)

Lint 억제: 총 68건
  - @ts-ignore: 23건
  - eslint-disable: 34건
  - noqa: 11건

Deprecated API: 총 15건
  - React Class Component: 8개 파일
  - moment.js 사용: 12개 파일
  - ReactDOM.render: 3개 파일

타입 안전성:
  - any 사용: 156건 (src 디렉토리)
  - @ts-ignore: 23건

의존성 부채:
  - outdated 패키지: 24개
  - deprecated 패키지: 3개
  - CVE 보유: 2개 (Critical 1, High 1)

우선순위 로드맵:
  P1 (즉시): 5건
  P2 (다음 스프린트): 12건
  P3 (틈새 시간): 48건
  P4 (장기 검토): 8건
```

### 예시 2: TODO 정리

요청:
```
TODO 정리해줘. 90일 이상 방치된 것들 위주로
```

결과 예시:
```
=== 장기 방치 TODO 목록 (90일+) ===

[387일] src/legacy/parser.js:42
  // TODO: 새로운 파서 구현으로 교체 (작성자: kim)

[245일] src/utils/format.ts:18
  // FIXME: 한국어 로케일 포맷 버그 (작성자: park)

[156일] src/api/auth.ts:87
  // HACK: 토큰 갱신 로직 임시 처리 (작성자: lee)
  ...

총 45건 / 권장 조치: 다음 스프린트에서 최소 15건 해소
```

### 예시 3: deprecated 분석

요청:
```
deprecated API 사용 현황 분석해줘
```

### 예시 4: 타입 안전성 검사

요청:
```
타입 안전성 검사해줘. any 사용이 얼마나 되는지 파악하고 싶어
```

---

## 에러 처리

| 상황 | 대응 |
|------|------|
| Git 저장소가 아닌 경우 | blame 분석을 건너뛰고 마커 스캔만 수행 |
| npm/pip 미설치 | 의존성 분석을 건너뛰고 수동 확인 안내 |
| 대규모 코드베이스 | 최근 변경 파일 또는 주요 디렉토리만 우선 스캔 |
| git blame 타임아웃 | 상위 100개 파일만 샘플링 분석 |
| 지원하지 않는 언어 | 일반 부채 마커(TODO/FIXME)만 탐지 |

---

## 관련 문서

- [SKILL.md](SKILL.md) - 스킬 정의 및 상세 워크플로우
- [references/analysis-guide.md](references/analysis-guide.md) - 분석 방법론, 언어별 패턴, false positive 기준
- [references/templates.md](references/templates.md) - 보고서 템플릿, 로드맵 형식, pre-commit hook 설정
