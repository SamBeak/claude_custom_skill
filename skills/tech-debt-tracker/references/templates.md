# Tech Debt Tracker 템플릿

이 문서는 기술 부채 분석 보고서 출력 형식, 부채 연령 보고서, Impact x Effort 매트릭스 출력, 개선 로드맵, 건강도 대시보드, pre-commit hook 설정 템플릿을 정의합니다.

---

## 1. 기술 부채 종합 보고서

### 전체 보고서 템플릿

```markdown
# 기술 부채 분석 보고서

> 분석 일시: {SCAN_DATE}
> 분석 범위: {SCAN_SCOPE} (예: 전체 코드베이스, src/ 디렉토리)
> 분석 대상: {FILE_COUNT}개 파일 ({LANGUAGE_SUMMARY})
> 프로젝트: {PROJECT_NAME}

---

## 종합 건강도

| 영역 | 점수 | 등급 |
|------|------|------|
| 부채 마커 | {MARKER_SCORE}/100 | {MARKER_GRADE} |
| 타입 안전성 | {TYPE_SCORE}/100 | {TYPE_GRADE} |
| Deprecated API | {DEPRECATED_SCORE}/100 | {DEPRECATED_GRADE} |
| 테스트 부채 | {TEST_SCORE}/100 | {TEST_GRADE} |
| 의존성 부채 | {DEPENDENCY_SCORE}/100 | {DEPENDENCY_GRADE} |
| **종합** | **{TOTAL_SCORE}/100** | **{TOTAL_GRADE}** |

---

## 부채 마커 현황

### 요약

| 마커 유형 | 건수 | 비율 |
|----------|------|------|
| TODO | {TODO_COUNT} | {TODO_PCT}% |
| FIXME | {FIXME_COUNT} | {FIXME_PCT}% |
| HACK | {HACK_COUNT} | {HACK_PCT}% |
| XXX | {XXX_COUNT} | {XXX_PCT}% |
| TEMP | {TEMP_COUNT} | {TEMP_PCT}% |
| WORKAROUND | {WORKAROUND_COUNT} | {WORKAROUND_PCT}% |
| 기타 | {OTHER_COUNT} | {OTHER_PCT}% |
| **합계** | **{TOTAL_MARKERS}** | **100%** |

### 연령 분포

| 연령 구간 | 건수 | 비율 |
|----------|------|------|
| 0~30일 (Fresh) | {FRESH_COUNT} | {FRESH_PCT}% |
| 31~90일 (Aging) | {AGING_COUNT} | {AGING_PCT}% |
| 91~180일 (Stale) | {STALE_COUNT} | {STALE_PCT}% |
| 181~365일 (Chronic) | {CHRONIC_COUNT} | {CHRONIC_PCT}% |
| 365일+ (Fossil) | {FOSSIL_COUNT} | {FOSSIL_PCT}% |

### 트렌드 (최근 6개월)

| 월 | 추가 | 제거 | 순변동 | 추세 |
|----|------|------|--------|------|
| {MONTH_1} | +{ADDED_1} | -{REMOVED_1} | {NET_1} | {TREND_1} |
| {MONTH_2} | +{ADDED_2} | -{REMOVED_2} | {NET_2} | {TREND_2} |
| ... | ... | ... | ... | ... |

### 장기 방치 목록 (Top 10)

| # | 경과일 | 파일 | 라인 | 마커 | 작성자 | 내용 |
|---|--------|------|------|------|--------|------|
| 1 | {DAYS} | `{FILE}` | {LINE} | {MARKER} | {AUTHOR} | {DESCRIPTION} |
| ... | ... | ... | ... | ... | ... | ... |

### 작성자별 분포

| 작성자 | 건수 | 비율 | 최다 마커 |
|--------|------|------|----------|
| {AUTHOR_1} | {COUNT_1} | {PCT_1}% | {TOP_MARKER_1} |
| ... | ... | ... | ... |

---

## Lint 억제 현황

| 카테고리 | 건수 | 심각도 |
|---------|------|--------|
| @ts-ignore | {TS_IGNORE_COUNT} | High |
| @ts-expect-error | {TS_EXPECT_COUNT} | Medium |
| eslint-disable (전체) | {ESLINT_ALL_COUNT} | Critical |
| eslint-disable (규칙별) | {ESLINT_RULE_COUNT} | Medium |
| prettier-ignore | {PRETTIER_COUNT} | Low |
| noqa | {NOQA_COUNT} | Medium |
| @SuppressWarnings | {SUPPRESS_COUNT} | Low |
| **합계** | **{TOTAL_SUPPRESSION}** | - |

---

## Deprecated API 사용 현황

| # | 카테고리 | deprecated API | 대체 API | 파일 수 | 심각도 |
|---|---------|---------------|---------|--------|--------|
| 1 | {CATEGORY} | {OLD_API} | {NEW_API} | {FILE_COUNT} | {SEVERITY} |
| ... | ... | ... | ... | ... | ... |

### EOL 런타임

| 런타임 | 현재 버전 | 최신 LTS | 상태 |
|--------|----------|---------|------|
| {RUNTIME} | {CURRENT} | {LATEST_LTS} | {STATUS} |

---

## 타입 안전성

### TypeScript any 사용 현황

- **총 any 사용 수**: {ANY_TOTAL}건
- **파일 수**: {ANY_FILE_COUNT}개

| # | 파일 | any 개수 | 주요 사용 패턴 |
|---|------|---------|--------------|
| 1 | `{FILE}` | {COUNT} | {PATTERN} |
| ... | ... | ... | ... |

### 타입 억제 현황

| 파일 | @ts-ignore | @ts-expect-error | 합계 |
|------|-----------|-----------------|------|
| `{FILE}` | {IGNORE} | {EXPECT} | {TOTAL} |
| ... | ... | ... | ... |

---

## 의존성 부채

| 패키지 | 현재 버전 | 최신 버전 | 차이 | 심각도 | CVE |
|--------|----------|---------|------|--------|-----|
| {PACKAGE} | {CURRENT} | {LATEST} | {DIFF_TYPE} | {SEVERITY} | {CVE_COUNT} |
| ... | ... | ... | ... | ... | ... |

---

## Impact x Effort 매트릭스

### P1 - 즉시 처리 (High Impact, Low Effort)
| # | 부채 항목 | 영향 | 예상 공수 |
|---|----------|------|----------|
| 1 | {ITEM} | {IMPACT_DESC} | {EFFORT_EST} |

### P2 - 계획 수립 (High Impact, High Effort)
| # | 부채 항목 | 영향 | 예상 공수 |
|---|----------|------|----------|
| 1 | {ITEM} | {IMPACT_DESC} | {EFFORT_EST} |

### P3 - 틈새 시간 (Low Impact, Low Effort)
| # | 부채 항목 | 영향 | 예상 공수 |
|---|----------|------|----------|
| 1 | {ITEM} | {IMPACT_DESC} | {EFFORT_EST} |

### P4 - 장기 검토 (Low Impact, High Effort)
| # | 부채 항목 | 영향 | 예상 공수 |
|---|----------|------|----------|
| 1 | {ITEM} | {IMPACT_DESC} | {EFFORT_EST} |

---

## 개선 로드맵

### 현재 스프린트
- [ ] {ACTION_ITEM_P1_1}
- [ ] {ACTION_ITEM_P1_2}

### 다음 스프린트
- [ ] {ACTION_ITEM_P2_1}

### 지속 개선
- [ ] {ACTION_ITEM_P3_1}

### 분기 계획
- [ ] {ACTION_ITEM_P4_1}
```

### 간략 보고서 템플릿 (요약)

```markdown
## 기술 부채 요약 보고서

> 분석 일시: {SCAN_DATE}
> 종합 건강도: {TOTAL_GRADE} ({TOTAL_SCORE}/100)

### 핵심 지표
- 부채 마커: {TOTAL_MARKERS}건 (90일+ 방치: {STALE_COUNT}건)
- Lint 억제: {TOTAL_SUPPRESSION}건
- Deprecated API: {DEPRECATED_COUNT}건
- any 사용: {ANY_TOTAL}건
- 의존성 부채: outdated {OUTDATED_COUNT}개, CVE {CVE_COUNT}개

### 즉시 조치 필요 (P1)
{P1_SUMMARY_LIST}

### 트렌드
{TREND_DIRECTION}: 지난 3개월간 부채 마커 순변동 {NET_CHANGE}건
```

---

## 2. 부채 연령 보고서

```markdown
## 부채 연령 분석 보고서

> 분석 일시: {SCAN_DATE}
> 분석 대상: {TOTAL_MARKERS}건의 부채 마커

### 연령 분포 히스토그램

```
0~30일   | ████████████████ {FRESH_COUNT}건
31~90일  | ████████████ {AGING_COUNT}건
91~180일 | ████████ {STALE_COUNT}건
181~365일| ████ {CHRONIC_COUNT}건
365일+   | ██ {FOSSIL_COUNT}건
```

### 평균 연령: {AVG_AGE}일
### 중앙값 연령: {MEDIAN_AGE}일
### 최장 방치: {MAX_AGE}일 ({MAX_AGE_FILE}:{MAX_AGE_LINE})

### 작성자별 장기 방치 현황

| 작성자 | 총 부채 | 90일+ | 평균 연령 |
|--------|--------|-------|----------|
| {AUTHOR} | {TOTAL} | {STALE} | {AVG}일 |
| ... | ... | ... | ... |

### 월별 부채 증감 트렌드

```
추가 ████████████ +{M1_ADD}
제거 ██████████   -{M1_REM}
     {MONTH_1}: 순변동 {M1_NET}

추가 ██████████ +{M2_ADD}
제거 ████████████████ -{M2_REM}
     {MONTH_2}: 순변동 {M2_NET}
...
```
```

---

## 3. 타입 안전성 보고서

```markdown
## 타입 안전성 분석 보고서

> 분석 일시: {SCAN_DATE}
> 분석 대상: {TS_FILE_COUNT}개 TypeScript 파일

### 요약

| 지표 | 수치 | 등급 |
|------|------|------|
| any 사용 | {ANY_COUNT}건 | {ANY_GRADE} |
| @ts-ignore | {IGNORE_COUNT}건 | {IGNORE_GRADE} |
| @ts-expect-error | {EXPECT_COUNT}건 | {EXPECT_GRADE} |
| @ts-nocheck | {NOCHECK_COUNT}건 | {NOCHECK_GRADE} |
| 미타이핑 함수 | {UNTYPED_COUNT}건 | {UNTYPED_GRADE} |
| **종합 점수** | **{TYPE_SCORE}/100** | **{TYPE_GRADE}** |

### any 사용 상위 파일

| # | 파일 | any 개수 | 사용 패턴 |
|---|------|---------|----------|
| 1 | `{FILE}` | {COUNT} | {PATTERNS} |
| ... | ... | ... | ... |

### 개선 권장 사항

#### 즉시 대응
{IMMEDIATE_ACTIONS}

#### 단계적 개선
{GRADUAL_ACTIONS}
```

---

## 4. Deprecated API 보고서

```markdown
## Deprecated API 분석 보고서

> 분석 일시: {SCAN_DATE}
> 발견 건수: {TOTAL_DEPRECATED}건

### 프레임워크별 현황

| 프레임워크 | deprecated 건수 | 영향 파일 수 |
|-----------|----------------|------------|
| {FRAMEWORK} | {COUNT} | {FILE_COUNT} |
| ... | ... | ... |

### 상세 목록

#### {FRAMEWORK_1}

| deprecated API | 대체 API | 파일 | 라인 | 마이그레이션 가이드 |
|---------------|---------|------|------|-----------------|
| {OLD_API} | {NEW_API} | `{FILE}` | {LINE} | {GUIDE_LINK} |
| ... | ... | ... | ... | ... |

### EOL 런타임 경고

{EOL_WARNINGS}

### 마이그레이션 로드맵

| 순서 | 대상 | 예상 공수 | 우선순위 |
|------|------|----------|---------|
| 1 | {MIGRATION_TARGET} | {EFFORT} | {PRIORITY} |
| ... | ... | ... | ... |
```

---

## 5. 개선 로드맵 상세 템플릿

```markdown
## 기술 부채 개선 로드맵

> 생성 일시: {DATE}
> 대상 기간: {QUARTER} (예: 2025 Q3)
> 총 부채 항목: {TOTAL_ITEMS}건

---

### Phase 1: 현재 스프린트 (P1 - 즉시 처리)

**목표**: High Impact, Low Effort 항목 해소
**예상 공수**: {P1_EFFORT}

| # | 작업 | 유형 | 파일 | 담당자 | 상태 |
|---|------|------|------|--------|------|
| 1 | {TASK_DESC} | {TYPE} | `{FILE}` | {ASSIGNEE} | [ ] |
| ... | ... | ... | ... | ... | ... |

**완료 기준**:
- [ ] 모든 Critical CVE 패치 완료
- [ ] @ts-ignore 중 사유 불명확한 항목 제거
- [ ] Deprecated API 단순 대체 완료

---

### Phase 2: 다음 스프린트 (P2 - 계획 수립)

**목표**: High Impact, High Effort 항목 착수
**예상 공수**: {P2_EFFORT}

| # | 작업 | 유형 | 범위 | 담당자 | 상태 |
|---|------|------|------|--------|------|
| 1 | {TASK_DESC} | {TYPE} | {SCOPE} | {ASSIGNEE} | [ ] |
| ... | ... | ... | ... | ... | ... |

**완료 기준**:
- [ ] TypeScript any 사용 {CURRENT_ANY} -> {TARGET_ANY}건으로 감소
- [ ] 테스트 커버리지 {CURRENT_COV}% -> {TARGET_COV}% 달성
- [ ] 주요 deprecated API 마이그레이션 완료

---

### Phase 3: 지속 개선 (P3 - 틈새 시간)

**목표**: Low Impact, Low Effort 항목을 일상 작업 중 해소
**방법**: 코드 터치 시 인접 부채 함께 해결 (Boy Scout Rule)

| # | 작업 | 유형 | 빈도 |
|---|------|------|------|
| 1 | 90일+ TODO/FIXME 처리 | 부채 마커 | 스프린트당 10건 |
| 2 | 미문서화 공개 API 문서 추가 | 문서 부채 | 스프린트당 5건 |
| 3 | 불필요한 lint 억제 주석 제거 | 코드 품질 | 코드 터치 시 |
| ... | ... | ... | ... |

---

### Phase 4: 분기 계획 (P4 - 장기 검토)

**목표**: Low Impact, High Effort 항목에 대한 장기 전략 수립

| # | 작업 | 유형 | 예상 공수 | 의사결정 시점 |
|---|------|------|----------|-------------|
| 1 | {MAJOR_MIGRATION} | 프레임워크 | {EFFORT} | {DECISION_DATE} |
| ... | ... | ... | ... | ... |

**검토 기준**:
- ROI 분석 완료
- 팀 합의 도출
- 마이그레이션 플랜 문서화
```

---

## 6. Pre-commit Hook 설정

### 기술 부채 관련 Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit
# 기술 부채 증가 방지 Pre-commit Hook

set -e

echo "========================================="
echo "  기술 부채 검사 실행 중..."
echo "========================================="

STAGED_FILES=$(git diff --staged --name-only --diff-filter=ACMR)
ISSUES_FOUND=0
WARNINGS=0

# ===== 1. 기한 없는 TODO/FIXME 추가 방지 =====
NEW_TODOS=$(git diff --staged -U0 | grep -E '^\+.*\b(TODO|FIXME)\b' | grep -vE '\(#\d+\)|\(\d{4}-\d{2}\)|\(\w+,\s*#\d+\)' || true)
if [ -n "$NEW_TODOS" ]; then
    echo ""
    echo "[WARNING] 기한 또는 이슈 번호 없는 TODO/FIXME가 추가되었습니다:"
    echo "$NEW_TODOS"
    echo ""
    echo "  권장 형식: // TODO(#123): 설명"
    echo "  또는: // TODO(2025-06): 설명"
    WARNINGS=$((WARNINGS + 1))
fi

# ===== 2. @ts-ignore 추가 경고 =====
NEW_TS_IGNORE=$(git diff --staged -U0 | grep -E '^\+.*@ts-ignore' || true)
if [ -n "$NEW_TS_IGNORE" ]; then
    echo ""
    echo "[WARNING] @ts-ignore가 추가되었습니다:"
    echo "$NEW_TS_IGNORE"
    echo ""
    echo "  권장: @ts-expect-error와 함께 사유를 명시하세요."
    echo "  또는: 적절한 타입 정의로 대체하세요."
    WARNINGS=$((WARNINGS + 1))
fi

# ===== 3. any 타입 추가 경고 =====
TS_FILES=$(echo "$STAGED_FILES" | grep -E '\.(ts|tsx)$' || true)
if [ -n "$TS_FILES" ]; then
    NEW_ANY=$(git diff --staged -U0 -- $TS_FILES | grep -E '^\+.*:\s*any\b|^\+.*<any>|^\+.*as\s+any\b' || true)
    if [ -n "$NEW_ANY" ]; then
        echo ""
        echo "[WARNING] any 타입이 추가되었습니다:"
        echo "$NEW_ANY"
        echo ""
        echo "  권장: 구체적인 타입을 사용하세요."
        WARNINGS=$((WARNINGS + 1))
    fi
fi

# ===== 4. eslint-disable (전체) 추가 차단 =====
NEW_ESLINT_ALL=$(git diff --staged -U0 | grep -E '^\+.*/\*\s*eslint-disable\s*\*/' || true)
if [ -n "$NEW_ESLINT_ALL" ]; then
    echo ""
    echo "[ERROR] eslint-disable (전체 규칙 비활성화)가 추가되었습니다:"
    echo "$NEW_ESLINT_ALL"
    echo ""
    echo "  -> 특정 규칙만 비활성화하세요: /* eslint-disable specific-rule */"
    ISSUES_FOUND=1
fi

# ===== 5. HACK/XXX 추가 경고 =====
NEW_HACK=$(git diff --staged -U0 | grep -E '^\+.*\b(HACK|XXX)\b' || true)
if [ -n "$NEW_HACK" ]; then
    echo ""
    echo "[WARNING] HACK 또는 XXX 마커가 추가되었습니다:"
    echo "$NEW_HACK"
    echo ""
    echo "  HACK/XXX는 높은 심각도의 기술 부채입니다."
    echo "  가능하면 근본적인 해결책을 적용하세요."
    WARNINGS=$((WARNINGS + 1))
fi

# ===== 결과 출력 =====
echo ""
if [ $ISSUES_FOUND -eq 1 ]; then
    echo "========================================="
    echo "  기술 부채 검사 실패!"
    echo "  위의 ERROR 항목을 수정한 후 다시 커밋하세요."
    echo "========================================="
    echo ""
    echo "건너뛰려면: git commit --no-verify"
    exit 1
elif [ $WARNINGS -gt 0 ]; then
    echo "========================================="
    echo "  기술 부채 경고 ${WARNINGS}건 (커밋 진행 가능)"
    echo "  위의 WARNING 항목을 검토하세요."
    echo "========================================="
    exit 0
else
    echo "========================================="
    echo "  기술 부채 검사 통과!"
    echo "========================================="
    exit 0
fi
```

### Husky + lint-staged 설정

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "bash -c 'grep -l \"@ts-ignore\" \"$@\" && echo \"[WARNING] @ts-ignore 사용 감지 - @ts-expect-error 사용 권장\" || exit 0' --",
      "bash -c 'any_count=$(grep -c \": any\\b\" \"$@\" 2>/dev/null || echo 0); [ \"$any_count\" -gt 5 ] && echo \"[WARNING] any 타입 ${any_count}건 감지\" || exit 0' --"
    ]
  }
}
```

### GitHub Actions 워크플로우

```yaml
# .github/workflows/tech-debt-check.yml
name: Tech Debt Check

on:
  pull_request:
    branches: [main, develop]

jobs:
  debt-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: 부채 마커 증감 확인
        run: |
          BASE=${{ github.event.pull_request.base.sha }}
          HEAD=${{ github.event.pull_request.head.sha }}

          ADDED=$(git diff $BASE...$HEAD -U0 | grep -cE '^\+.*\b(TODO|FIXME|HACK|XXX)\b' || echo 0)
          REMOVED=$(git diff $BASE...$HEAD -U0 | grep -cE '^\-.*\b(TODO|FIXME|HACK|XXX)\b' || echo 0)
          NET=$((ADDED - REMOVED))

          echo "추가: ${ADDED}, 제거: ${REMOVED}, 순변동: ${NET}"

          if [ "$NET" -gt 5 ]; then
            echo "::warning::기술 부채 마커가 ${NET}건 순증가했습니다. 부채 해소를 검토하세요."
          fi

      - name: any 타입 증가 확인
        run: |
          BASE=${{ github.event.pull_request.base.sha }}
          HEAD=${{ github.event.pull_request.head.sha }}

          NEW_ANY=$(git diff $BASE...$HEAD -U0 -- '*.ts' '*.tsx' | grep -cE '^\+.*:\s*any\b|^\+.*<any>|^\+.*as\s+any\b' || echo 0)
          REMOVED_ANY=$(git diff $BASE...$HEAD -U0 -- '*.ts' '*.tsx' | grep -cE '^\-.*:\s*any\b|^\-.*<any>|^\-.*as\s+any\b' || echo 0)
          NET_ANY=$((NEW_ANY - REMOVED_ANY))

          echo "any 추가: ${NEW_ANY}, 제거: ${REMOVED_ANY}, 순변동: ${NET_ANY}"

          if [ "$NET_ANY" -gt 3 ]; then
            echo "::warning::any 타입이 ${NET_ANY}건 순증가했습니다. 구체적인 타입 사용을 검토하세요."
          fi

      - name: @ts-ignore 추가 확인
        run: |
          BASE=${{ github.event.pull_request.base.sha }}
          HEAD=${{ github.event.pull_request.head.sha }}

          NEW_IGNORE=$(git diff $BASE...$HEAD -U0 | grep -cE '^\+.*@ts-ignore' || echo 0)

          if [ "$NEW_IGNORE" -gt 0 ]; then
            echo "::warning::@ts-ignore가 ${NEW_IGNORE}건 추가되었습니다. @ts-expect-error 사용을 권장합니다."
          fi

      - name: npm outdated 확인
        if: hashFiles('package-lock.json') != ''
        run: |
          npm outdated --json 2>/dev/null | jq 'to_entries | length' || echo "0"
        continue-on-error: true
```

---

## 7. 건강도 대시보드 템플릿

### 텍스트 기반 대시보드

```markdown
## 코드 건강도 대시보드

> 최종 업데이트: {DATE}

### 종합 점수: {SCORE}/100 ({GRADE})

```
부채 마커    [████████░░] {MARKER_SCORE}/100
타입 안전성  [██████░░░░] {TYPE_SCORE}/100
Deprecated  [█████████░] {DEPRECATED_SCORE}/100
테스트 부채  [███████░░░] {TEST_SCORE}/100
의존성 부채  [████████░░] {DEPENDENCY_SCORE}/100
```

### 핵심 KPI

| KPI | 현재 | 목표 | 추세 |
|-----|------|------|------|
| 총 부채 마커 | {CURRENT_MARKERS} | <{TARGET_MARKERS} | {TREND} |
| 90일+ 방치 | {STALE_COUNT} | 0 | {TREND} |
| any 사용 | {ANY_COUNT} | <{TARGET_ANY} | {TREND} |
| @ts-ignore | {IGNORE_COUNT} | 0 | {TREND} |
| CVE 미패치 | {CVE_COUNT} | 0 | {TREND} |
| Deprecated API | {DEPRECATED_COUNT} | 0 | {TREND} |
| 테스트 비율 | {TEST_RATIO}% | >{TARGET_RATIO}% | {TREND} |

### 지난 달 대비 변화

| 영역 | 지난 달 | 이번 달 | 변화 |
|------|--------|--------|------|
| 부채 마커 | {PREV_MARKERS} | {CURR_MARKERS} | {DIFF_MARKERS} |
| any 사용 | {PREV_ANY} | {CURR_ANY} | {DIFF_ANY} |
| 의존성 부채 | {PREV_DEP} | {CURR_DEP} | {DIFF_DEP} |
```
```

---

## 8. 스킬 연계 통합 보고서 템플릿

```markdown
## 종합 코드 건강도 보고서

> 분석 일시: {DATE}
> 사용 스킬: Tech Debt Tracker + refactor-advisor + test-coverage-analyzer + security-scanner

---

### 1. 기술 부채 (Tech Debt Tracker)
{TECH_DEBT_SUMMARY}

### 2. 코드 품질 (refactor-advisor)
{REFACTOR_SUMMARY}

### 3. 테스트 커버리지 (test-coverage-analyzer)
{TEST_COVERAGE_SUMMARY}

### 4. 보안 취약점 (security-scanner)
{SECURITY_SUMMARY}

---

### 통합 우선순위

| # | 항목 | 출처 스킬 | 영향도 | 노력도 | 우선순위 |
|---|------|----------|--------|--------|---------|
| 1 | {ITEM} | {SKILL} | {IMPACT} | {EFFORT} | P{N} |
| ... | ... | ... | ... | ... | ... |

### 통합 개선 로드맵

{INTEGRATED_ROADMAP}
```
