# Code Formatter Configuration Generator

코드베이스 분석 또는 사전 정의된 스타일 템플릿을 기반으로 언어별 코드 포매팅 설정 파일을 자동 생성하는 스킬입니다.

## 개요

기존 코드베이스의 패턴을 분석하거나 표준 스타일 가이드(Google, Airbnb, Standard)를 적용하여 ESLint, Prettier, EditorConfig, CheckStyle 등의 설정 파일을 자동으로 생성합니다.

## 주요 기능

- **코드베이스 분석**: 기존 코드의 스타일 패턴 자동 감지
- **템플릿 기반 생성**: 업계 표준 스타일 가이드 적용
- **다양한 언어 지원**: JavaScript/TypeScript, Java, Python 등
- **IDE 통합 안내**: VS Code, IntelliJ 등 주요 IDE 설정 방법 제공

## 지원 언어 및 도구

### JavaScript/TypeScript
- **ESLint**: 코드 품질 및 스타일 검사
- **Prettier**: 코드 포매팅
- **EditorConfig**: 에디터 설정

### Java
- **CheckStyle**: 코드 스타일 검사
- **EditorConfig**: 에디터 설정

### Python
- **Black**: 코드 포매터
- **Flake8**: 린터
- **EditorConfig**: 에디터 설정

### 범용
- **EditorConfig**: 모든 언어에 적용 가능한 에디터 설정

## 사용 사례

다음과 같은 요청 시 이 스킬이 활성화됩니다:

1. ESLint 또는 Prettier 설정 파일 생성
2. 코드 분석을 통한 포매팅 규칙 생성
3. 코드 스타일 가이드 설정
4. `.editorconfig` 파일 생성
5. 팀 코딩 표준 수립
6. 코드 포매팅 설정 관련 모든 작업

## 작업 흐름

### 코드베이스 분석을 통한 생성

1. **샘플 파일 읽기**: 대표적인 파일 5-10개 분석
2. **패턴 식별**:
   - 들여쓰기 (탭 vs 스페이스, 크기)
   - 따옴표 스타일 (작은따옴표 vs 큰따옴표)
   - 세미콜론 (필수 vs 선택)
   - 중괄호 스타일 (K&R, Allman 등)
   - 줄바꿈 및 간격
   - 네이밍 컨벤션
3. **설정 생성**: 발견된 패턴에 맞는 설정 파일 생성

### 템플릿을 통한 생성

1. **스타일 선택**: 사용자에게 스타일 가이드 선택 요청
   - Airbnb (JavaScript/TypeScript)
   - Google (Java, JavaScript, Python)
   - Standard (JavaScript)
   - Custom (사용자 정의)
2. **템플릿 로드**: 선택된 템플릿 가져오기
3. **커스터마이징**: 필요시 규칙 수정
4. **파일 생성**: 프로젝트 루트에 설정 파일 생성

## 생성 파일

### JavaScript/TypeScript 프로젝트
- `.eslintrc.json` - ESLint 설정
- `.prettierrc.json` - Prettier 설정
- `.editorconfig` - 에디터 설정

### Java 프로젝트
- `checkstyle.xml` - CheckStyle 설정
- `.editorconfig` - 에디터 설정

### Python 프로젝트
- `pyproject.toml` - Black/Flake8 설정
- `.editorconfig` - 에디터 설정

### 다중 언어 프로젝트
- `.editorconfig` - 범용 에디터 설정
- 언어별 설정 파일 추가

## 출력 형식

생성된 설정 파일과 함께 다음 정보를 제공합니다:

1. 각 설정 파일의 역할 설명
2. 필요한 도구 설치 방법
3. 포매터/린터 실행 방법
4. IDE 통합 방법 (VS Code, IntelliJ 등)

### 예시

```
# ESLint 설정

코드베이스 분석을 기반으로 `.eslintrc.json`을 생성했습니다.

## 주요 규칙
- 들여쓰기: 탭
- 따옴표: 큰따옴표
- 세미콜론: 필수

## 설치
npm install --save-dev eslint

## 사용
npm run lint
```

## 참고 문서

- [SKILL.md](SKILL.md) - 스킬 전체 문서
- [references/templates.md](references/templates.md) - 언어별 설정 템플릿
- [references/analysis-guide.md](references/analysis-guide.md) - 코드베이스 분석 방법론

## 사용 예시

```
사용자: "프로젝트에 ESLint 설정 파일을 만들어줘"
→ 코드베이스를 분석하여 .eslintrc.json 생성

사용자: "Airbnb 스타일 가이드로 Prettier 설정해줘"
→ Airbnb 템플릿 기반 .prettierrc.json 생성

사용자: "Java 프로젝트에 CheckStyle 설정 추가해줘"
→ checkstyle.xml과 .editorconfig 생성
```

## 베스트 프랙티스

- **일관성**: 코드베이스 분석 시 기존 패턴과 일치시키기
- **팀 표준 우선**: 개인 스타일보다 팀 선호도 우선
- **점진적 도입**: 핵심 규칙부터 시작하여 점진적으로 추가
- **문서화**: 규칙 선택 이유 설명
- **IDE 통합**: 주요 에디터 설정 방법 제공

## 추가 자산

프로젝트 템플릿 파일:
- [assets/package.json.template](assets/package.json.template) - npm 스크립트 예시
- [assets/vscode-settings.json.template](assets/vscode-settings.json.template) - VS Code 설정 예시
