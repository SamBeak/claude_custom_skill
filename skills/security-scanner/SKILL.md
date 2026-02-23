---
name: security-scanner
description: 코드 변경사항에서 보안 취약점을 사전 탐지하고, OWASP Top 10 기반으로 위험 요소를 식별하여 수정 가이드를 제공하는 스킬. 사용자가 다음을 요청할 때 활성화: (1) 보안 스캔/검사 실행, (2) 취약점 확인/분석, (3) 시크릿/API 키 탐지, (4) security scan 또는 vulnerability check, (5) 커밋/PR 전 보안 리뷰, (6) 의존성 보안 감사, (7) OWASP 기반 코드 점검, (8) 하드코딩된 자격증명 검사.
---

# Security Scanner

코드 변경사항 및 전체 코드베이스에서 보안 취약점을 사전 탐지하고, OWASP Top 10 기반으로 위험 요소를 식별하여 심각도별 분류 및 수정 가이드를 제공합니다.

## Quick Start

사용자가 보안 스캔을 요청하면 다음 워크플로우를 실행합니다:

1. **스캔 범위 결정**:
   ```bash
   # 스테이징된 변경사항 스캔
   git diff --staged

   # 또는 전체 코드베이스 스캔
   find . -type f \( -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.java" -o -name "*.sql" \) -not -path "*/node_modules/*" -not -path "*/.git/*"
   ```

2. **파일별 언어 감지** 후 언어별 취약점 패턴 매칭

3. **시크릿 패턴 탐지** (API 키, 토큰, 자격증명)

4. **의존성 감사 실행** (npm audit, pip audit 등)

5. **결과를 심각도별 분류** (Critical / High / Medium / Low / Info)

6. **수정 가이드 및 보고서 생성** ([references/templates.md](references/templates.md) 형식 사용)

## OWASP Top 10 취약점 패턴 매칭

각 OWASP 카테고리별로 코드 패턴을 분석합니다. 상세 분석 방법론은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

### A01: Broken Access Control (접근 제어 취약점)

**탐지 패턴**:
- 인가 검사 없는 API 엔드포인트
- 직접 객체 참조 (IDOR) 패턴
- 디렉토리 트래버설 경로

```regex
# 인가 미들웨어 누락 확인
(app\.(get|post|put|delete|patch)\s*\([^)]*\)\s*(?!.*auth)(?!.*protect)(?!.*guard))
# 디렉토리 트래버설
(\.\./|\.\.\\|%2e%2e%2f|%2e%2e\/|\.%2e\/|%2e\.\/)
# 직접 파일 접근
(fs\.(readFile|writeFile|unlink|rmdir)\s*\(\s*(req\.(params|query|body)))
```

**검증 항목**:
- 모든 API 라우트에 인증/인가 미들웨어 적용 여부
- 사용자 입력으로 파일 경로를 직접 구성하는지 확인
- RBAC(역할 기반 접근 제어) 구현 여부

### A02: Cryptographic Failures (암호화 실패)

**탐지 패턴**:
```regex
# 약한 해시 알고리즘
(md5|sha1|SHA1)\s*\(
# 하드코딩된 암호화 키
(secret|key|password|passwd|pwd)\s*[:=]\s*['"][^'"]{8,}['"]
# HTTP 사용 (HTTPS 미사용)
http://(?!localhost|127\.0\.0\.1|0\.0\.0\.0)
# 약한 암호화 알고리즘
(DES|RC4|Blowfish|ECB)
# 솔트 없는 해시
(hashlib\.(md5|sha1)\(|MessageDigest\.getInstance\s*\(\s*["']MD5["']\))
```

**검증 항목**:
- bcrypt, argon2, scrypt 등 강력한 해시 함수 사용 여부
- TLS 1.2 이상 강제 여부
- 암호화 키의 안전한 관리 (환경 변수 또는 키 관리 서비스)

### A03: Injection (인젝션)

가장 광범위한 취약점 카테고리입니다. 언어별로 세분화하여 탐지합니다.

**SQL Injection**:
```regex
# 문자열 연결을 통한 SQL 쿼리 구성
(["']SELECT\s.*\+\s*(req|request|params|query|args|input|user))
(["']INSERT\s.*\+\s*(req|request|params|query|args|input|user))
(["']UPDATE\s.*\+\s*(req|request|params|query|args|input|user))
(["']DELETE\s.*\+\s*(req|request|params|query|args|input|user))
# f-string 또는 format을 통한 SQL 구성 (Python)
(f["'].*SELECT.*\{|\.format\(.*SELECT)
# String concatenation SQL (Java)
(["'].*SELECT.*["']\s*\+\s*\w+|Statement\s+\w+\s*=.*createStatement)
```

**XSS (Cross-Site Scripting)**:
```regex
# JavaScript
(\.innerHTML\s*=|\.outerHTML\s*=|document\.write\s*\(|\.insertAdjacentHTML\s*\()
# React
(dangerouslySetInnerHTML)
# 템플릿 엔진 미이스케이프 출력
(\{\{\{.*\}\}\}|<%[-=].*%>|\$\{.*\}|v-html\s*=)
```

**Command Injection**:
```regex
# Node.js
(child_process\.(exec|execSync|spawn|spawnSync)\s*\(.*\+|child_process\.(exec|execSync)\s*\(\s*`[^`]*\$\{)
# Python
(os\.system\s*\(.*\+|subprocess\.(call|run|Popen)\s*\(.*shell\s*=\s*True|os\.popen\s*\()
# Java
(Runtime\.getRuntime\(\)\.exec\s*\(.*\+|ProcessBuilder\s*\(.*\+)
```

**LDAP Injection**:
```regex
(ldap_search\s*\(.*\+|SearchFilter\s*\(.*\+|ldap\.search\s*\(.*\+)
```

**NoSQL Injection**:
```regex
(\$where\s*:|\.find\s*\(\s*\{.*req\.(body|query|params)|\.aggregate\s*\(\s*\[.*req\.(body|query|params))
```

### A04: Insecure Design (안전하지 않은 설계)

**탐지 패턴**:
```regex
# Rate limiting 미적용
(app\.use\s*\((?!.*rateLimit)(?!.*rate-limit)(?!.*throttle))
# CAPTCHA 없는 인증 폼
(login|signin|sign-in|authenticate)(?!.*captcha)(?!.*recaptcha)
# 비밀번호 복잡성 검증 미적용
(password)(?!.*validate)(?!.*strength)(?!.*complexity)(?!.*regex)
```

**검증 항목**:
- Threat modeling 수행 여부
- Rate limiting 적용 여부
- 비즈니스 로직 검증 (금액 한도, 횟수 제한 등)
- 다단계 인증 (MFA) 지원 여부

### A05: Security Misconfiguration (보안 설정 오류)

**탐지 패턴**:
```regex
# 디버그 모드 활성화
(DEBUG\s*=\s*True|debug\s*:\s*true|NODE_ENV.*development)
# 기본 자격증명 사용
(admin:admin|root:root|test:test|password:password|user:password)
# 불필요한 기능 활성화
(X-Powered-By|Server:|stacktrace|stack_trace|verbose.*error)
# 디렉토리 리스팅 허용
(autoindex\s+on|Options\s+Indexes|directory-listing.*true)
# CORS 와일드카드
(Access-Control-Allow-Origin.*\*|cors\(\s*\)|origin\s*:\s*['"]?\*['"]?)
```

**검증 항목**:
- 프로덕션 환경에서 디버그 모드 비활성화 여부
- 불필요한 HTTP 헤더 제거 여부
- 최소 권한 원칙 적용 여부
- 보안 헤더 설정 (CSP, X-Frame-Options, X-Content-Type-Options 등)

### A06: Vulnerable and Outdated Components (취약한 구성 요소)

**탐지 방법**:
```bash
# Node.js
npm audit --json 2>/dev/null
npx audit-ci --config audit-ci.json 2>/dev/null

# Python
pip audit --format=json 2>/dev/null
safety check --json 2>/dev/null

# Java (Maven)
mvn org.owasp:dependency-check-maven:check 2>/dev/null

# Java (Gradle)
./gradlew dependencyCheckAnalyze 2>/dev/null
```

**검증 항목**:
- package.json, requirements.txt, pom.xml 내 알려진 CVE
- 지원 종료(EOL) 버전 사용 여부
- 라이선스 호환성 확인

### A07: Identification and Authentication Failures (인증 실패)

**탐지 패턴**:
```regex
# 약한 비밀번호 정책
(minLength\s*[:=]\s*[1-5][^0-9]|password.*length.*[<]\s*8)
# 세션 고정 공격
(session\.id\s*=|JSESSIONID|connect\.sid)(?!.*regenerate)
# JWT 미검증
(jwt\.decode\s*\((?!.*verify)|jsonwebtoken.*verify\s*=\s*false|algorithms\s*=\s*\[['"]none['"]\])
# 기본 인증 하드코딩
(Authorization.*Basic\s+[A-Za-z0-9+/=]+)
# 비밀번호 평문 저장
(password\s*=.*getText\(\)|password\s*=.*value\b(?!.*hash|.*bcrypt|.*encrypt))
```

**검증 항목**:
- 비밀번호 해시 (bcrypt/argon2) 사용 여부
- 세션 타임아웃 설정
- 다단계 인증 지원
- 계정 잠금 정책

### A08: Software and Data Integrity Failures (무결성 실패)

**탐지 패턴**:
```regex
# 안전하지 않은 역직렬화
(pickle\.loads?\s*\(|yaml\.load\s*\((?!.*Loader\s*=\s*yaml\.SafeLoader)|ObjectInputStream|unserialize\s*\(|JSON\.parse\s*\(.*req\.)
# CDN Subresource Integrity 미사용
(<script\s+src=["']https?://(?!.*integrity\s*=))
# 서명 미검증 업데이트
(auto.?update(?!.*verify)(?!.*signature)(?!.*checksum))
# CI/CD 파이프라인 보안
(curl\s+.*\|\s*(bash|sh)|wget\s+.*\|\s*(bash|sh))
```

**검증 항목**:
- Subresource Integrity (SRI) 사용 여부
- 코드 서명 검증 여부
- CI/CD 파이프라인 보안 설정

### A09: Security Logging and Monitoring Failures (로깅 실패)

**탐지 패턴**:
```regex
# 인증 이벤트 로깅 미적용
(login|authenticate|signin)(?!.*log)(?!.*audit)(?!.*track)
# 민감 데이터 로그 출력
(console\.log\s*\(.*password|logger?\.(info|debug|warn|error)\s*\(.*password|print\s*\(.*password|log\.(info|debug)\s*\(.*token|console\.log\s*\(.*secret)
# 에러 로깅 미적용
(catch\s*\(\s*\w+\s*\)\s*\{(?!\s*.*log)(?!\s*.*console)(?!\s*.*logger))
```

**검증 항목**:
- 인증 성공/실패 이벤트 로깅
- 민감 데이터 마스킹
- 감사 로그 무결성 보장
- 알림 임계값 설정

### A10: Server-Side Request Forgery (SSRF)

**탐지 패턴**:
```regex
# 사용자 입력을 URL로 직접 사용
(fetch\s*\(\s*(req\.(body|query|params)|request\.(body|query|params))|axios\.(get|post)\s*\(\s*(req\.|request\.)|urllib\.request\.urlopen\s*\(.*input|requests\.(get|post)\s*\(\s*\w*url)
# URL 화이트리스트 미적용
(http\.get\s*\(.*req\.|https\.get\s*\(.*req\.)
# DNS rebinding 가능 패턴
(new\s+URL\s*\(.*req\.)
```

**검증 항목**:
- URL 화이트리스트/블랙리스트 적용 여부
- 내부 네트워크 접근 차단 여부
- DNS 확인 후 요청 전송 여부

## 시크릿/API 키 탐지

하드코딩된 자격증명, API 키, 토큰 등을 탐지합니다. 상세 정규식 패턴은 [references/analysis-guide.md](references/analysis-guide.md)를 참조하세요.

### 탐지 대상

| 카테고리 | 패턴 설명 | 심각도 |
|---------|----------|--------|
| AWS Access Key | `AKIA[0-9A-Z]{16}` | Critical |
| AWS Secret Key | 40자리 base64 문자열 | Critical |
| GitHub Token | `ghp_[a-zA-Z0-9]{36}` / `github_pat_` | Critical |
| Google API Key | `AIza[0-9A-Za-z\-_]{35}` | High |
| Slack Token | `xox[baprs]-[a-zA-Z0-9-]+` | High |
| JWT | `eyJ[A-Za-z0-9-_]+\.eyJ[A-Za-z0-9-_]+` | High |
| Private Key | `-----BEGIN (RSA|DSA|EC|OPENSSH) PRIVATE KEY-----` | Critical |
| 일반 패스워드 | `password\s*[:=]\s*['"][^'"]+['"]` | High |
| 데이터베이스 연결 문자열 | `(mongodb|postgres|mysql|redis)://[^/\s]+:[^/\s]+@` | Critical |
| Azure 키 | `[a-zA-Z0-9+/]{86}==` (Azure Storage) | High |
| Stripe Key | `sk_(live|test)_[a-zA-Z0-9]{24,}` | Critical |
| SendGrid Key | `SG\.[a-zA-Z0-9_-]{22}\.[a-zA-Z0-9_-]{43}` | High |
| Twilio | `SK[a-f0-9]{32}` | High |

### .env 파일 커밋 방지 확인

```bash
# .gitignore에 .env 포함 여부 확인
grep -q '\.env' .gitignore 2>/dev/null
echo $?

# 스테이징된 .env 파일 확인
git diff --staged --name-only | grep -i '\.env'

# 트래킹 중인 .env 파일 확인
git ls-files | grep -i '\.env'
```

### Private Key 파일 감지

```bash
# Private key 파일 탐지
find . -type f \( -name "*.pem" -o -name "*.key" -o -name "*.p12" -o -name "*.pfx" -o -name "id_rsa" -o -name "id_ecdsa" -o -name "id_ed25519" \) -not -path "*/.git/*" -not -path "*/node_modules/*"

# 파일 내용에서 private key 패턴 탐지
grep -rl "BEGIN.*PRIVATE KEY" . --include="*.js" --include="*.ts" --include="*.py" --include="*.java" --include="*.json" --include="*.yaml" --include="*.yml" --include="*.xml" --include="*.conf" --include="*.cfg" 2>/dev/null
```

## 언어별 취약점 스캔

### JavaScript / TypeScript

| 위험 패턴 | 설명 | 심각도 |
|----------|------|--------|
| `eval()` | 임의 코드 실행 | Critical |
| `Function()` 생성자 | 동적 코드 실행 | Critical |
| `innerHTML =` | XSS 취약점 | High |
| `dangerouslySetInnerHTML` | React XSS 취약점 | High |
| `document.write()` | DOM 기반 XSS | High |
| `__proto__`, `constructor.prototype` | Prototype Pollution | High |
| `new RegExp(userInput)` | ReDoS 취약점 | Medium |
| `setTimeout(string)` | 문자열 기반 타이머 (코드 실행) | Medium |
| `setInterval(string)` | 문자열 기반 타이머 (코드 실행) | Medium |
| `require(variable)` | 동적 모듈 로드 | Medium |
| `window.location = userInput` | 오픈 리다이렉트 | Medium |
| `postMessage('*')` | Cross-origin 통신 | Medium |
| `crypto.createCipher` | 폐기된 암호화 API | Medium |

```regex
# JavaScript/TypeScript 위험 패턴 통합 정규식
(eval\s*\(|new\s+Function\s*\(|\.innerHTML\s*=|dangerouslySetInnerHTML|document\.write\s*\(|__proto__|constructor\s*\[\s*['"]prototype['"]\]|setTimeout\s*\(\s*['"`]|setInterval\s*\(\s*['"`]|require\s*\(\s*\w+[^'")])
```

### Python

| 위험 패턴 | 설명 | 심각도 |
|----------|------|--------|
| `pickle.loads()` / `pickle.load()` | 안전하지 않은 역직렬화 | Critical |
| `exec()` | 임의 코드 실행 | Critical |
| `eval()` | 임의 코드 실행 | Critical |
| `subprocess` with `shell=True` | 명령어 인젝션 | High |
| `yaml.load()` without SafeLoader | 역직렬화 취약점 | High |
| `os.system()` | 명령어 인젝션 | High |
| `os.popen()` | 명령어 인젝션 | High |
| `input()` (Python 2) | 코드 실행 | High |
| `__import__()` | 동적 모듈 임포트 | Medium |
| `compile()` + `exec()` | 동적 코드 컴파일 | Medium |
| `marshal.loads()` | 안전하지 않은 역직렬화 | High |
| `tempfile.mktemp()` | Race condition | Medium |
| `assert` (보안 검증) | 프로덕션에서 비활성화 가능 | Medium |

```regex
# Python 위험 패턴 통합 정규식
(pickle\.(loads?|Unpickler)\s*\(|exec\s*\(|eval\s*\(|subprocess\.(call|run|Popen)\s*\(.*shell\s*=\s*True|yaml\.load\s*\((?!.*SafeLoader)|os\.(system|popen)\s*\(|__import__\s*\(|marshal\.loads\s*\(|tempfile\.mktemp\s*\()
```

### Java

| 위험 패턴 | 설명 | 심각도 |
|----------|------|--------|
| `Runtime.exec()` | 명령어 인젝션 | High |
| `Statement` (vs PreparedStatement) | SQL 인젝션 | Critical |
| SQL 문자열 연결 | SQL 인젝션 | Critical |
| `ObjectInputStream.readObject()` | 역직렬화 | Critical |
| `XMLInputFactory` (XXE) | XML External Entity | High |
| `SecureRandom` 미사용 | 약한 난수 생성 | Medium |
| `X509TrustManager` 빈 구현 | 인증서 검증 우회 | Critical |
| `setAllowFileAccess(true)` | WebView 파일 접근 | High |
| `addJavascriptInterface` | WebView 코드 실행 | High |

```regex
# Java 위험 패턴 통합 정규식
(Runtime\.getRuntime\(\)\.exec\s*\(|createStatement\s*\(|ObjectInputStream|XMLInputFactory|\.readObject\s*\(|TrustManager|setAllowFileAccess\s*\(\s*true|addJavascriptInterface)
```

### SQL

| 위험 패턴 | 설명 | 심각도 |
|----------|------|--------|
| 동적 쿼리 구성 | SQL 인젝션 가능성 | High |
| `GRANT ALL` | 과도한 권한 부여 | High |
| `GRANT ... WITH GRANT OPTION` | 권한 위임 | High |
| `EXECUTE AS` | 권한 상승 | Medium |
| `xp_cmdshell` | OS 명령 실행 (MSSQL) | Critical |
| `LOAD_FILE()` | 파일 읽기 (MySQL) | High |
| `INTO OUTFILE` | 파일 쓰기 (MySQL) | High |
| `OPENROWSET` | 원격 데이터 접근 (MSSQL) | High |

```regex
# SQL 위험 패턴
(GRANT\s+ALL|WITH\s+GRANT\s+OPTION|EXECUTE\s+AS|xp_cmdshell|LOAD_FILE\s*\(|INTO\s+OUTFILE|OPENROWSET|BULK\s+INSERT)
```

## 의존성 취약점 확인

### 실행 명령

프로젝트 유형을 감지한 후 해당 패키지 매니저의 감사 명령을 실행합니다:

```bash
# 프로젝트 유형 감지
if [ -f "package.json" ] || [ -f "package-lock.json" ]; then
    echo "Node.js 프로젝트 감지"
    npm audit --json 2>/dev/null || echo "npm audit 실행 실패"
fi

if [ -f "requirements.txt" ] || [ -f "Pipfile" ] || [ -f "pyproject.toml" ]; then
    echo "Python 프로젝트 감지"
    pip audit --format=json 2>/dev/null || safety check --json 2>/dev/null || echo "Python 감사 도구 미설치"
fi

if [ -f "pom.xml" ]; then
    echo "Maven 프로젝트 감지"
    mvn org.owasp:dependency-check-maven:check 2>/dev/null || echo "dependency-check 미설치"
fi

if [ -f "build.gradle" ] || [ -f "build.gradle.kts" ]; then
    echo "Gradle 프로젝트 감지"
    ./gradlew dependencyCheckAnalyze 2>/dev/null || echo "dependency-check 미설치"
fi

if [ -f "go.mod" ]; then
    echo "Go 프로젝트 감지"
    govulncheck ./... 2>/dev/null || echo "govulncheck 미설치"
fi
```

### 결과 분석 기준

| 심각도 | CVSS 점수 | 조치 |
|--------|-----------|------|
| Critical | 9.0 - 10.0 | 즉시 업데이트 필수 |
| High | 7.0 - 8.9 | 가능한 빨리 업데이트 |
| Medium | 4.0 - 6.9 | 계획적 업데이트 |
| Low | 0.1 - 3.9 | 다음 릴리스에 포함 |

## 보안 설정 검증

### HTTPS 강제

```regex
# HTTP 링크 사용 (비보안)
http://(?!localhost|127\.0\.0\.1|0\.0\.0\.0|10\.|172\.(1[6-9]|2[0-9]|3[01])\.|192\.168\.)
# HSTS 미설정
(?!.*Strict-Transport-Security)
```

### CORS 설정

```regex
# 와일드카드 오리진 (위험)
(Access-Control-Allow-Origin\s*:\s*\*|origin\s*:\s*['"]?\*['"]?|cors\(\s*\))
# credentials와 와일드카드 동시 사용 (불가능하지만 시도하는 패턴)
(credentials\s*:\s*true.*origin\s*:\s*\*|origin\s*:\s*\*.*credentials\s*:\s*true)
```

### CSP 헤더

```regex
# unsafe-inline 또는 unsafe-eval 사용
(Content-Security-Policy.*unsafe-inline|Content-Security-Policy.*unsafe-eval)
# CSP 미설정 확인 (helmet 또는 수동 설정 필요)
```

### 인증/인가 미들웨어 확인

```bash
# Express.js 인증 미들웨어 확인
grep -rn "passport\|jwt\|auth.*middleware\|isAuthenticated\|requireAuth\|protect.*route" --include="*.js" --include="*.ts" .

# Django 인증 확인
grep -rn "@login_required\|@permission_required\|IsAuthenticated\|IsAdminUser" --include="*.py" .

# Spring Security 확인
grep -rn "@PreAuthorize\|@Secured\|@RolesAllowed\|SecurityConfig\|WebSecurityConfigurerAdapter" --include="*.java" .
```

### Rate Limiting 설정

```bash
# Express.js rate limiting
grep -rn "express-rate-limit\|rateLimit\|rate-limit\|throttle" --include="*.js" --include="*.ts" .

# Django throttling
grep -rn "DEFAULT_THROTTLE_RATES\|throttle_classes\|AnonRateThrottle\|UserRateThrottle" --include="*.py" .

# Spring Boot rate limiting
grep -rn "RateLimiter\|@RateLimit\|Bucket4j\|resilience4j.*ratelimiter" --include="*.java" .
```

## 심각도 분류 기준

스캔 결과를 다음 기준으로 분류합니다:

### Critical (즉시 대응)

- 원격 코드 실행(RCE) 가능 취약점
- SQL Injection 확인
- 하드코딩된 프로덕션 자격증명
- Private key 노출
- 알려진 CVE (CVSS >= 9.0)
- 인증 우회 가능

### High (48시간 내 대응)

- XSS 취약점
- SSRF 가능성
- 안전하지 않은 역직렬화
- 약한 암호화 사용
- 과도한 권한 부여
- 알려진 CVE (CVSS 7.0-8.9)

### Medium (1주일 내 대응)

- 보안 헤더 미설정
- 불완전한 입력 검증
- 약한 비밀번호 정책
- 디버그 모드 활성화 가능성
- 알려진 CVE (CVSS 4.0-6.9)

### Low (계획적 대응)

- 정보 노출 (서버 버전, 기술 스택)
- 사소한 설정 미비
- 알려진 CVE (CVSS < 4.0)
- 베스트 프랙티스 미준수

### Info (참고 사항)

- 보안 모범 사례 권장
- 코드 품질 관련 제안
- 최신 보안 가이드 참조

## 수정 가이드

각 취약점 유형별 수정 방법을 제공합니다. 상세 템플릿은 [references/templates.md](references/templates.md)를 참조하세요.

### 공통 수정 원칙

1. **입력 검증**: 모든 외부 입력에 대해 화이트리스트 기반 검증 적용
2. **파라미터화된 쿼리**: SQL 쿼리에 바인딩 변수 사용
3. **출력 인코딩**: 사용자 입력을 출력할 때 컨텍스트에 맞는 인코딩 적용
4. **최소 권한**: 필요한 최소한의 권한만 부여
5. **심층 방어**: 여러 계층의 보안 제어 적용
6. **안전한 기본값**: 기본 설정이 보안적으로 안전하도록 구성
7. **환경 변수**: 자격증명은 환경 변수 또는 시크릿 관리 서비스 사용

### 인젝션 수정 예시

**취약한 코드** (SQL Injection):
```javascript
// 위험: 문자열 연결
const query = "SELECT * FROM users WHERE id = " + req.params.id;
db.query(query);
```

**수정된 코드**:
```javascript
// 안전: 파라미터화된 쿼리
const query = "SELECT * FROM users WHERE id = ?";
db.query(query, [req.params.id]);
```

### XSS 수정 예시

**취약한 코드**:
```javascript
// 위험: innerHTML 직접 사용
element.innerHTML = userInput;
```

**수정된 코드**:
```javascript
// 안전: textContent 사용 또는 DOMPurify 적용
element.textContent = userInput;
// 또는
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);
```

## 워크플로우 상세

```
1. 스캔 범위 결정
   ├─ git diff --staged (커밋 전 스캔)
   ├─ git diff HEAD~1 (최근 변경 스캔)
   └─ 전체 코드베이스 스캔

2. 파일 수집 및 언어 감지
   ├─ .js, .ts, .jsx, .tsx → JavaScript/TypeScript
   ├─ .py → Python
   ├─ .java → Java
   ├─ .sql → SQL
   ├─ .yaml, .yml, .json, .xml → 설정 파일
   └─ .env, .pem, .key → 민감 파일

3. 취약점 패턴 매칭
   ├─ OWASP Top 10 패턴 적용
   ├─ 언어별 위험 함수 탐지
   └─ 컨텍스트 분석 (false positive 최소화)

4. 시크릿 패턴 탐지
   ├─ API 키 / 토큰 패턴 매칭
   ├─ .env 파일 커밋 확인
   └─ Private key 파일 감지

5. 의존성 감사
   ├─ npm audit / pip audit
   ├─ 알려진 CVE 확인
   └─ EOL 버전 확인

6. 보안 설정 검증
   ├─ HTTPS, CORS, CSP
   ├─ 인증/인가 미들웨어
   └─ Rate limiting

7. 결과 분류 및 보고서 생성
   ├─ 심각도별 분류 (Critical/High/Medium/Low/Info)
   ├─ 수정 가이드 첨부
   └─ 보고서 출력 (references/templates.md 형식)
```

## 트리거 조건

이 스킬은 다음 상황에서 활성화됩니다:

- 사용자가 "보안 스캔", "보안 검사", "취약점 확인", "취약점 분석" 요청 시
- "security scan", "vulnerability check", "security audit" 영문 요청 시
- "시크릿 검사", "API 키 탐지", "자격증명 확인" 요청 시
- "의존성 감사", "dependency audit" 요청 시
- 커밋/PR 전 "보안 리뷰" 요청 시
- "OWASP 점검", "OWASP 스캔" 요청 시

## 에러 처리

| # | 에러 상황 | 감지 방법 | 대응 |
|---|----------|----------|------|
| 1 | Git 저장소가 아닌 경우 | `git status` 실패 | 전체 디렉토리 스캔으로 전환 |
| 2 | 지원하지 않는 언어 | 파일 확장자 미매칭 | 일반 시크릿 탐지만 수행 |
| 3 | 감사 도구 미설치 | 명령어 실행 실패 | 수동 확인 가이드 제공 |
| 4 | 대규모 코드베이스 | 파일 수 > 10,000 | 변경된 파일만 우선 스캔 |
| 5 | 바이너리 파일 포함 | 파일 유형 검사 | 바이너리 파일 제외 |
| 6 | 권한 부족 | 파일 읽기 실패 | 읽기 가능한 파일만 스캔 |
| 7 | 패턴 오탐 (False Positive) | 컨텍스트 분석 | [references/analysis-guide.md](references/analysis-guide.md)의 FP 판단 기준 적용 |
| 8 | 네트워크 오류 (CVE 조회) | API 호출 실패 | 오프라인 패턴 매칭만 수행 |

## 참조 문서

- [references/analysis-guide.md](references/analysis-guide.md) - 상세 분석 방법론, 시크릿 탐지 정규식, false positive 판단 기준
- [references/templates.md](references/templates.md) - 보고서 출력 형식, 수정 가이드 템플릿, pre-commit hook 설정

## 모범 사례

1. **커밋 전 스캔**: 모든 커밋 전에 `git diff --staged` 기반 스캔 실행
2. **정기적 전체 스캔**: 주 1회 이상 전체 코드베이스 스캔
3. **의존성 업데이트**: Critical/High CVE 발견 시 즉시 업데이트
4. **시크릿 관리**: 자격증명은 반드시 환경 변수 또는 시크릿 관리 서비스 사용
5. **보안 교육**: 팀원에게 스캔 결과 공유 및 보안 인식 향상
6. **CI/CD 통합**: pre-commit hook 또는 CI 파이프라인에 스캔 자동화
7. **점진적 수정**: Critical부터 순서대로 수정하되 모든 심각도에 대응
