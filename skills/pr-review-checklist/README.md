# PR Review Checklist Generator

`git diff`를 분석하여 변경사항 종류별 맞춤 리뷰 체크리스트를 자동 생성하고, PR description을 작성하여 GitHub PR을 생성하는 Claude Code 커스텀 스킬입니다.

## 스킬 소개

코드 리뷰에서 반복적으로 확인해야 하는 항목들을 자동화합니다. DB 마이그레이션, API 변경, UI 수정, 설정 변경 등 변경의 종류에 따라 꼭 확인해야 할 항목이 다릅니다. 이 스킬은 `git diff`를 분석하여 변경된 파일의 카테고리를 자동으로 파악하고, 해당 카테고리에 맞는 리뷰 체크리스트를 생성합니다.

또한 테스트 파일 누락, 문서 미갱신 등 흔히 놓치기 쉬운 항목을 경고하여, 리뷰 품질을 높이고 배포 사고를 예방합니다.

## 주요 기능

### 1. 변경 파일 자동 분류
`git diff`의 파일 경로와 확장자를 분석하여 다음 카테고리로 자동 분류합니다:
- **DB 변경**: 마이그레이션, 스키마 파일
- **API 변경**: 컨트롤러, 라우트, 핸들러
- **UI 변경**: 컴포넌트, 페이지, 스타일
- **설정 변경**: config, env, Docker
- **테스트 변경**: 테스트 코드, fixture
- **비즈니스 로직**: 서비스, 모델, 유틸리티
- **인프라/CI**: GitHub Actions, Terraform, Kubernetes

### 2. 카테고리별 맞춤 체크리스트
각 카테고리에 해당하는 변경이 있을 때만 관련 체크리스트를 생성합니다:
- DB 변경 → 롤백 전략, 데이터 무결성, 인덱스, lock 영향
- API 변경 → 하위호환성, 인증/인가, 입력 검증, 응답 형식
- UI 변경 → 접근성(a11y), 반응형, 크로스브라우저, 상태 처리
- 설정 변경 → 환경별 분리, 시크릿 노출, 기본값 설정
- 테스트 변경 → 커버리지, 엣지케이스, 모킹 전략

### 3. 누락 항목 경고
- 소스 코드가 변경되었지만 대응하는 테스트 파일이 없는 경우 경고
- API 변경이 있지만 API 문서 갱신이 없는 경우 경고
- 모델 변경이 있지만 마이그레이션이 없는 경우 경고

### 4. PR Description 자동 작성
- 변경 목적 요약 (Summary)
- 변경 파일 통계와 주요 변경사항 (Changes)
- 카테고리별 리뷰 체크리스트 (Review Checklist)
- 누락 경고 (Warnings)
- 테스트 계획 (Test Plan)
- 관련 이슈 번호 자동 추출 (Related Issues)

### 5. PR 크기 분류 및 레이블 추천
- S(1-50줄) / M(51-200줄) / L(201-500줄) / XL(501줄+) 크기 분류
- 변경 카테고리에 따른 GitHub Label 추천
- XL PR에 대한 분할 권고

### 6. GitHub PR 생성 연동
- `gh pr create` 명령으로 직접 PR 생성
- 사용자 확인 후 생성 (자동 생성하지 않음)

## 사용 예시

### 기본 사용

```
사용자: PR 만들어줘

Claude: 변경사항을 분석합니다...

[분석 결과]
현재 브랜치: feature/add-user-profile
Base branch: main
커밋 수: 3
변경 파일: 8개 (신규 3, 수정 5)
변경 줄 수: +187 / -23 (Size: M)

[파일 분류]
- API 변경: 2개 (userController.ts, profileRoutes.ts)
- UI 변경: 3개 (UserProfile.tsx, ProfileEdit.tsx, profile.css)
- 비즈니스 로직: 1개 (userService.ts)
- 테스트: 2개 (userController.test.ts, userService.test.ts)

[PR Description 초안]

## Summary
사용자 프로필 조회 및 수정 기능을 추가합니다. 프로필 이미지 업로드와
자기소개 편집을 지원합니다.

## Changes
### 변경 파일 통계
- 신규: 3개 파일
- 수정: 5개 파일
- 총 변경: +187줄 / -23줄

### 주요 변경사항
- 사용자 프로필 API 엔드포인트 추가 (GET /api/users/:id/profile, PUT /api/users/:id/profile)
- 프로필 조회/수정 UI 컴포넌트 구현
- 프로필 이미지 업로드 서비스 로직 추가

## Review Checklist

#### API 변경 체크리스트
- [ ] 하위 호환성(backward compatibility)이 유지되는가?
- [ ] 인증(authentication) 처리가 적용되어 있는가?
- [ ] 인가(authorization) / 권한 검사가 구현되어 있는가?
- [ ] 요청(request) 입력값 검증(validation)이 있는가?
- [ ] 응답(response) 형식이 일관적인가?
- [ ] 에러 응답 형식이 표준을 따르는가?

#### UI 변경 체크리스트
- [ ] 웹 접근성(a11y)이 준수되는가?
- [ ] 반응형 디자인이 적용되어 있는가?
- [ ] 크로스 브라우저 호환성이 확인되었는가?
- [ ] 로딩 상태(loading state)가 처리되어 있는가?
- [ ] 에러 상태(error state)가 처리되어 있는가?

## Test Plan
- [ ] 프로필 API 단위 테스트 통과 확인
- [ ] 프로필 UI 컴포넌트 렌더링 테스트 확인
- [ ] 이미지 업로드 시나리오 수동 테스트

## Related Issues
- Refs #42

---

이 내용으로 PR을 생성할까요? 수정이 필요하면 말씀해주세요.
```

### 대규모 변경 (경고 포함)

```
사용자: 풀 리퀘스트 생성해줘

Claude: 변경사항을 분석합니다...

[분석 결과]
현재 브랜치: feature/payment-system
Base branch: main
커밋 수: 12
변경 파일: 24개 (신규 15, 수정 8, 삭제 1)
변경 줄 수: +1,523 / -89 (Size: XL)

경고: PR 크기가 매우 큽니다 (XL: 1,612줄 변경).
가능하다면 여러 개의 작은 PR로 분할하는 것을 권장합니다.

[파일 분류]
- DB 변경: 3개 (마이그레이션, 스키마)
- API 변경: 5개 (결제 API)
- 비즈니스 로직: 7개 (결제 서비스)
- 설정 변경: 2개 (환경 변수)
- UI 변경: 4개 (결제 UI)
- 테스트: 3개

경고: 다음 파일에 대응하는 테스트 파일이 변경 또는 추가되지 않았습니다.
  - src/services/refundService.ts → 테스트 없음
  - src/services/settlementService.ts → 테스트 없음

경고: API 엔드포인트가 변경되었지만 API 문서 갱신이 감지되지 않았습니다.

## Review Checklist

#### DB / 마이그레이션 체크리스트
- [ ] 롤백(down) 마이그레이션이 작성되어 있는가?
- [ ] 기존 데이터와의 호환성이 보장되는가?
- [ ] 필요한 인덱스가 추가되었는가?
- [ ] 대용량 테이블 변경 시 lock 영향을 고려했는가?

#### API 변경 체크리스트
- [ ] 하위 호환성이 유지되는가?
- [ ] 인증/인가 처리가 적용되어 있는가?
- [ ] 입력값 검증이 있는가?
- [ ] Rate limiting이 적용되어 있는가?

#### 설정 변경 체크리스트
- [ ] 시크릿/민감 정보가 하드코딩되지 않았는가?
- [ ] 환경별 분리가 되어 있는가?
- [ ] .env 파일이 .gitignore에 포함되어 있는가?

#### UI 변경 체크리스트
- [ ] 접근성(a11y)이 준수되는가?
- [ ] 반응형 디자인이 적용되어 있는가?
- [ ] 결제 관련 에러 상태가 사용자에게 명확하게 표시되는가?

## Warnings
- 테스트 누락: refundService.ts, settlementService.ts에 대한 테스트가 없습니다
- 문서 누락: API 변경에 대한 문서 갱신이 필요합니다
- PR 크기: XL (분할 권장)

...
```

### 리뷰 체크리스트만 요청

```
사용자: 리뷰 체크리스트만 만들어줘

Claude: 변경사항을 분석하여 체크리스트를 생성합니다...

[변경 카테고리: 설정 변경, 인프라/CI]

#### 설정 변경 체크리스트
- [ ] 환경별(dev/staging/production) 분리가 되어 있는가?
- [ ] 시크릿/민감 정보가 코드에 하드코딩되지 않았는가?
- [ ] .env 파일이 .gitignore에 포함되어 있는가?
- [ ] 환경 변수 변경 시 배포 문서가 갱신되었는가?
- [ ] 기본값(fallback)이 적절하게 설정되어 있는가?
- [ ] 팀원에게 새로운 환경 변수 추가가 공유되었는가?

#### 인프라/CI 변경 체크리스트
- [ ] CI 파이프라인이 정상적으로 동작하는가?
- [ ] 배포 스크립트 변경 시 롤백 절차가 확인되었는가?
- [ ] 보안 그룹/네트워크 규칙이 올바른가?
- [ ] 모니터링/알림 설정이 포함되어 있는가?
```

## 체크리스트 예시 (실제 프로젝트)

### 예시 1: Next.js 프론트엔드 변경

변경 파일:
- `src/app/dashboard/page.tsx` (Modified)
- `src/components/Chart.tsx` (Added)
- `src/styles/dashboard.module.css` (Added)

생성되는 체크리스트:

```markdown
#### UI 변경 체크리스트
- [ ] 웹 접근성(a11y)이 준수되는가? (aria 속성, 키보드 내비게이션)
- [ ] 반응형 디자인이 적용되어 있는가? (모바일, 태블릿, 데스크탑)
- [ ] 크로스 브라우저 호환성이 확인되었는가?
- [ ] 로딩 상태(loading state)가 처리되어 있는가?
- [ ] 에러 상태(error state)가 처리되어 있는가?
- [ ] 빈 상태(empty state)가 처리되어 있는가?
- [ ] 성능 (불필요한 리렌더링, 메모이제이션 등)이 고려되었는가?
```

경고:
```
경고: 다음 파일에 대응하는 테스트 파일이 없습니다.
  - src/components/Chart.tsx → 테스트 파일 없음
```

### 예시 2: Django REST API + DB 마이그레이션

변경 파일:
- `apps/orders/migrations/0015_add_coupon_field.py` (Added)
- `apps/orders/models.py` (Modified)
- `apps/orders/serializers.py` (Modified)
- `apps/orders/views.py` (Modified)
- `apps/orders/urls.py` (Modified)
- `tests/orders/test_views.py` (Modified)

생성되는 체크리스트:

```markdown
#### DB / 마이그레이션 체크리스트
- [ ] 롤백(down) 마이그레이션이 작성되어 있는가?
- [ ] 기존 데이터와의 호환성이 보장되는가?
- [ ] 필요한 인덱스가 추가되었는가?
- [ ] NOT NULL 컬럼 추가 시 기본값이 설정되어 있는가?
- [ ] 대용량 테이블 변경 시 lock 영향을 고려했는가?

#### API 변경 체크리스트
- [ ] 하위 호환성(backward compatibility)이 유지되는가?
- [ ] 인증(authentication) 처리가 적용되어 있는가?
- [ ] 인가(authorization) / 권한 검사가 구현되어 있는가?
- [ ] 요청(request) 입력값 검증(validation)이 있는가?
- [ ] 응답(response) 형식이 일관적인가?
- [ ] API 문서(Swagger/OpenAPI 등)가 갱신되었는가?
```

### 예시 3: CI/CD 파이프라인 + Docker 설정 변경

변경 파일:
- `.github/workflows/deploy.yml` (Modified)
- `Dockerfile` (Modified)
- `docker-compose.prod.yml` (Modified)
- `.env.example` (Modified)

생성되는 체크리스트:

```markdown
#### 설정 변경 체크리스트
- [ ] 환경별(dev/staging/production) 분리가 되어 있는가?
- [ ] 시크릿/민감 정보가 코드에 하드코딩되지 않았는가?
- [ ] .env 파일이 .gitignore에 포함되어 있는가?
- [ ] 기본값(fallback)이 적절하게 설정되어 있는가?
- [ ] Docker/인프라 설정 변경 시 영향 범위가 확인되었는가?
- [ ] 팀원에게 새로운 환경 변수 추가가 공유되었는가?

#### 인프라/CI 변경 체크리스트
- [ ] CI 파이프라인이 정상적으로 동작하는가?
- [ ] 배포 스크립트 변경 시 롤백 절차가 확인되었는가?
- [ ] 보안 그룹/네트워크 규칙이 올바른가?
- [ ] 환경 변수/시크릿 관리가 적절한가?
- [ ] 모니터링/알림 설정이 포함되어 있는가?
```

## 에러 처리

| 에러 | 대응 |
|------|------|
| git 저장소가 아님 | 안내 메시지 출력 후 중단 |
| base branch 없음 | 사용자에게 base branch 확인 요청 |
| 변경사항 없음 | "변경사항이 없습니다" 안내 |
| 커밋되지 않은 변경 | 커밋 먼저 하도록 안내 |
| gh CLI 미설치 | 체크리스트만 생성, PR 생성은 수동 안내 |
| gh 인증 안됨 | `gh auth login` 안내 |

## 관련 문서

- [SKILL.md](SKILL.md) - Claude Code용 상세 지침 (체크리스트 로직, 파일 분류 규칙 등)
- [references/analysis-guide.md](references/analysis-guide.md) - 파일 분류 기준, 위험도 판단, PR 크기 분류 방법론
- [references/templates.md](references/templates.md) - PR description 템플릿, 체크리스트 템플릿, 경고 메시지 템플릿
