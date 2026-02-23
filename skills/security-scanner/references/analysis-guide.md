# Security Scanner 분석 가이드

이 문서는 보안 스캔의 상세 분석 방법론, 시크릿 탐지 정규식 패턴, 언어별 위험 함수 목록, false positive 판단 기준, 심각도 분류 매트릭스를 정의합니다.

---

## 1. OWASP 카테고리별 분석 방법

### A01: Broken Access Control 분석

**분석 절차**:

1. 모든 API 라우트/엔드포인트를 식별합니다
2. 각 엔드포인트에 인증/인가 미들웨어가 적용되었는지 확인합니다
3. 사용자 입력이 리소스 식별자(ID)로 직접 사용되는지 확인합니다
4. 파일 경로 구성에 사용자 입력이 포함되는지 확인합니다

**탐지 정규식**:

```regex
# Express.js - 인가 미들웨어 없는 라우트
app\.(get|post|put|delete|patch)\s*\(\s*['"][^'"]+['"]\s*,\s*(?!.*auth|.*protect|.*guard|.*middleware|.*verify|.*check|.*require|.*validate)\s*(async\s+)?\(?\s*(req|ctx)

# Django - 데코레이터 없는 뷰 함수
def\s+(get|post|put|delete|patch|list|create|update|destroy)\s*\(self.*\)(?<!@login_required)(?<!@permission_required)

# Spring - 어노테이션 없는 컨트롤러 메서드
@(Get|Post|Put|Delete|Patch)Mapping(?!.*@PreAuthorize)(?!.*@Secured)(?!.*@RolesAllowed)

# IDOR - 직접 객체 참조
(findById|findOne|get)\s*\(\s*(req\.(params|query|body)\.\w+|request\.getParameter)

# 디렉토리 트래버설
(path\.join|path\.resolve|os\.path\.join)\s*\(.*req\.(params|query|body)
```

**확인 체크리스트**:
- [ ] 모든 엔드포인트에 인증 미들웨어 적용
- [ ] 리소스 접근 시 소유자 확인 로직 존재
- [ ] 관리자 전용 기능에 역할 검증 적용
- [ ] 파일 접근 시 경로 정규화 및 화이트리스트 적용
- [ ] CORS 정책이 특정 도메인으로 제한

### A02: Cryptographic Failures 분석

**분석 절차**:

1. 사용 중인 해시 알고리즘을 식별합니다
2. 암호화 키 관리 방식을 확인합니다
3. TLS/SSL 설정을 검증합니다
4. 민감 데이터의 전송 및 저장 방식을 확인합니다

**탐지 정규식**:

```regex
# 약한 해시 알고리즘
crypto\.createHash\s*\(\s*['"]md5['"]\)
crypto\.createHash\s*\(\s*['"]sha1['"]\)
hashlib\.md5\s*\(
hashlib\.sha1\s*\(
MessageDigest\.getInstance\s*\(\s*["']MD5["']\)
MessageDigest\.getInstance\s*\(\s*["']SHA-1["']\)

# 약한 암호화
crypto\.createCipher\s*\(  # deprecated API
DES\.|DESede\.|RC4\.|Blowfish\.
AES.*ECB  # ECB 모드

# 하드코딩된 암호화 키/솔트
(encryption_key|secret_key|crypto_key|salt)\s*[:=]\s*['"][^'"]{4,}['"]

# 비밀번호 평문 저장
password\s*=\s*(request|req)\.(body|params|query)\.password\s*;?\s*$
(INSERT|UPDATE).*password.*VALUES?\s*\(.*(?!hash|bcrypt|encrypt|argon)

# 안전하지 않은 난수 생성
Math\.random\(\)  # 보안 목적 사용 시
random\.random\(\)  # Python: 보안 목적 사용 시
java\.util\.Random  # SecureRandom 대신 사용 시
```

**확인 체크리스트**:
- [ ] 비밀번호에 bcrypt/argon2/scrypt 사용
- [ ] AES-256-GCM 또는 ChaCha20-Poly1305 등 강력한 암호화 사용
- [ ] TLS 1.2 이상 강제
- [ ] 암호화 키를 환경 변수 또는 KMS로 관리
- [ ] 안전한 난수 생성기 사용 (crypto.randomBytes, secrets, SecureRandom)

### A03: Injection 분석

**분석 절차**:

1. 외부 입력이 사용되는 모든 지점을 식별합니다
2. 각 입력 지점에서 데이터가 어떻게 처리되는지 추적합니다
3. 쿼리, 명령어, HTML 출력에 사용자 입력이 직접 삽입되는지 확인합니다
4. 파라미터화 또는 이스케이핑 적용 여부를 검증합니다

**SQL Injection 탐지 정규식**:

```regex
# JavaScript/TypeScript
["'`]SELECT\s+.*\$\{
["'`]SELECT\s+.*['"`]\s*\+
["'`]INSERT\s+INTO\s+.*\$\{
["'`]UPDATE\s+.*SET\s+.*\$\{
["'`]DELETE\s+FROM\s+.*\$\{
\.query\s*\(\s*["'`].*\+\s*(req|request|params|args)
\.execute\s*\(\s*["'`].*\+\s*(req|request|params|args)

# Python
f["']SELECT\s+.*\{
f["']INSERT\s+.*\{
f["']UPDATE\s+.*\{
f["']DELETE\s+.*\{
["']SELECT\s+.*["']\s*%\s*\(
cursor\.(execute|executemany)\s*\(\s*f["']
cursor\.(execute|executemany)\s*\(\s*["'].*["']\s*%

# Java
["']SELECT\s+.*["']\s*\+\s*\w+
Statement\s+\w+\s*=\s*\w+\.createStatement
(executeQuery|executeUpdate)\s*\(\s*["'].*\+
String\s+\w+\s*=\s*["']SELECT.*["']\s*\+
```

**XSS 탐지 정규식**:

```regex
# DOM 기반 XSS
\.innerHTML\s*=\s*(?!['"]<)
\.outerHTML\s*=\s*(?!['"]<)
document\.write\s*\(
document\.writeln\s*\(
\.insertAdjacentHTML\s*\(

# React XSS
dangerouslySetInnerHTML\s*=\s*\{\{

# Vue.js XSS
v-html\s*=

# Angular XSS
\[innerHTML\]\s*=
bypassSecurityTrust(Html|Script|Url|ResourceUrl|Style)

# 템플릿 엔진
\{\{\{.*\}\}\}  # Handlebars unescaped
<%[-=]\s*.*%>   # EJS
\|\s*safe\b      # Django/Jinja2 safe filter
```

**Command Injection 탐지 정규식**:

```regex
# Node.js
child_process\.(exec|execSync)\s*\(.*(\+|\$\{|concat)
child_process\.spawn\s*\(.*\{.*shell\s*:\s*true
require\s*\(\s*['"]child_process['"]\s*\)\.(exec|execSync)\s*\(\s*`

# Python
os\.system\s*\(\s*(f['"]|['"].*%|.*\+|.*format)
os\.popen\s*\(\s*(f['"]|['"].*%|.*\+|.*format)
subprocess\.(call|run|Popen|check_output)\s*\(.*shell\s*=\s*True
commands\.(getoutput|getstatusoutput)\s*\(

# Java
Runtime\.getRuntime\s*\(\s*\)\s*\.exec\s*\(\s*.*\+
new\s+ProcessBuilder\s*\(\s*.*\+

# PHP
(exec|system|passthru|shell_exec|popen|proc_open)\s*\(.*\$
`.*\$.*`  # backtick execution
```

### A04: Insecure Design 분석

**분석 절차**:

1. 비즈니스 로직의 보안 제어를 확인합니다
2. Rate limiting 적용 여부를 검증합니다
3. 입력 유효성 검사의 포괄성을 평가합니다

**탐지 정규식**:

```regex
# Rate limiting 미적용
app\.(get|post|put|delete)\s*\(.*(?!.*rateLimit|.*rate.?limit|.*throttle|.*limiter)

# 결제/금액 검증 미적용 (서버 측)
(price|amount|total|cost)\s*=\s*(req|request)\.(body|query|params)(?!.*validate|.*verify|.*check)

# 비밀번호 복잡성 미검증
password(?!.*(?:regex|pattern|validate|check|strength|policy|complex|rules|min.*length))
```

### A05: Security Misconfiguration 분석

**분석 절차**:

1. 환경별 설정 파일을 확인합니다
2. 디버그/개발 설정이 프로덕션에 노출되는지 확인합니다
3. 보안 헤더 설정을 검증합니다

**탐지 정규식**:

```regex
# 디버그 모드
DEBUG\s*=\s*True
debug\s*[:=]\s*true
NODE_ENV\s*[:=]\s*['"]?development['"]?

# 스택 트레이스 노출
stack_?trace|stackTrace|err\.stack|error\.stack
res\.(json|send)\s*\(.*err\.(stack|message)

# 기본 자격증명
(username|user)\s*[:=]\s*['"]admin['"]
(password|passwd|pwd)\s*[:=]\s*['"]admin['"]
(password|passwd|pwd)\s*[:=]\s*['"]password['"]
(password|passwd|pwd)\s*[:=]\s*['"]123456['"]

# 불필요한 헤더
X-Powered-By
Server:\s*(Apache|nginx|IIS)

# CORS 완전 개방
cors\s*\(\s*\)
Access-Control-Allow-Origin\s*:\s*\*
origin\s*:\s*['"]?\*['"]?

# 보안 헤더 미설정 (helmet 등 미사용)
# app 초기화 후 helmet 미적용 확인
```

### A06: Vulnerable Components 분석

**분석 절차**:

1. 의존성 목록을 추출합니다
2. 각 의존성의 알려진 취약점(CVE)을 확인합니다
3. EOL(지원 종료) 버전을 식별합니다

**분석 명령어**:

```bash
# package.json 분석
cat package.json | jq '.dependencies, .devDependencies' 2>/dev/null

# requirements.txt 분석
cat requirements.txt 2>/dev/null

# pom.xml 의존성 추출
grep -A2 '<dependency>' pom.xml 2>/dev/null

# lock 파일 기반 정확한 버전 확인
cat package-lock.json | jq '.dependencies | to_entries[] | {name: .key, version: .value.version}' 2>/dev/null
```

### A07: Authentication Failures 분석

**탐지 정규식**:

```regex
# 약한 비밀번호 정책
minLength\s*[:=]\s*[1-7]\b
password.*min.*[1-7]\b
passwordMinLength\s*[:=]\s*[1-7]\b

# JWT 보안 이슈
algorithm\s*[:=]\s*['"]none['"]
jwt\.decode\s*\((?!.*verify)
jwt\.verify\s*\(.*\{\s*algorithms\s*:\s*\[['"]none['"]\]

# 세션 관리 이슈
(cookie|session)(?!.*secure|.*httpOnly|.*sameSite)
maxAge\s*[:=]\s*\d{10,}  # 비정상적으로 긴 세션

# 무차별 대입 방지 미적용
(login|signin|authenticate).*(?!.*lockout|.*attempts|.*block|.*ban|.*captcha)
```

### A08: Integrity Failures 분석

**탐지 정규식**:

```regex
# 안전하지 않은 역직렬화
pickle\.(load|loads)\s*\(
yaml\.load\s*\(\s*[^)]*\)(?!.*Loader\s*=\s*yaml\.SafeLoader)
yaml\.load\s*\(\s*[^)]*\)(?!.*safe_load)
ObjectInputStream
java\.io\.Serializable
unserialize\s*\(  # PHP
Marshal\.load\s*\(  # Ruby

# SRI 미사용
<script\s+src\s*=\s*["']https?://(?!.*integrity)
<link\s+.*href\s*=\s*["']https?://(?!.*integrity)

# CI/CD 파이프라인 보안
curl\s+.*\|\s*(ba)?sh
wget\s+.*\|\s*(ba)?sh
curl\s+.*-o\s+.*&&\s*(ba)?sh
```

### A09: Logging Failures 분석

**탐지 정규식**:

```regex
# 민감 데이터 로그 출력
(console\.log|logger?\.(info|debug|warn|error)|print|System\.out\.print)\s*\(.*(?:password|passwd|pwd|secret|token|api[_-]?key|credit.?card|ssn|social.?security)

# 빈 catch 블록 (에러 무시)
catch\s*\(\s*\w*\s*\)\s*\{\s*\}
except\s*.*:\s*pass\s*$
catch\s*\(\s*\w+\s*\)\s*\{\s*//.*\s*\}

# 인증 이벤트 로깅 미적용
(login|logout|authenticate|signin|signout|register).*(?!.*log|.*audit|.*track|.*record|.*event)
```

### A10: SSRF 분석

**탐지 정규식**:

```regex
# 사용자 입력 기반 HTTP 요청
(fetch|axios\.(get|post|put|delete)|http\.get|https\.get|urllib\.request\.urlopen|requests\.(get|post|put|delete|head|patch))\s*\(\s*(req\.(body|query|params)|request\.(body|query|params|GET|POST)|args\.get)

# URL 구성에 사용자 입력 사용
(new\s+)?URL\s*\(\s*(req\.|request\.|args|params)
url\s*=\s*(f['"].*\{|['"].*%s|.*\+\s*(req|request))

# 내부 IP 접근 미차단
(127\.0\.0\.1|localhost|0\.0\.0\.0|10\.\d|172\.(1[6-9]|2\d|3[01])\.|192\.168\.)(?!.*block|.*deny|.*reject|.*whitelist|.*blacklist)
```

---

## 2. 시크릿 탐지 정규식 패턴

### AWS 자격증명

```regex
# AWS Access Key ID
(A3T[A-Z0-9]|AKIA|AGPA|AIDA|AROA|AIPA|ANPA|ANVA|ASIA)[A-Z0-9]{16}

# AWS Secret Access Key
(?<![A-Za-z0-9/+=])[A-Za-z0-9/+=]{40}(?![A-Za-z0-9/+=])

# AWS Session Token
(?i)(aws.?session.?token|aws.?security.?token)\s*[:=]\s*['"]?[A-Za-z0-9/+=]{100,}
```

### GitHub 토큰

```regex
# GitHub Personal Access Token (classic)
ghp_[a-zA-Z0-9]{36}

# GitHub Personal Access Token (fine-grained)
github_pat_[a-zA-Z0-9]{22}_[a-zA-Z0-9]{59}

# GitHub OAuth Access Token
gho_[a-zA-Z0-9]{36}

# GitHub App Token
(ghu|ghs)_[a-zA-Z0-9]{36}

# GitHub App Installation Token
ghi[ts]_[a-zA-Z0-9]{36}
```

### Google Cloud / Firebase

```regex
# Google API Key
AIza[0-9A-Za-z\-_]{35}

# Google OAuth Client ID
[0-9]+-[a-z0-9_]{32}\.apps\.googleusercontent\.com

# Firebase 설정
(?i)(firebase|firestore).*api[_-]?key\s*[:=]\s*['"]AIza[0-9A-Za-z\-_]{35}['"]
```

### Azure

```regex
# Azure Storage Account Key
(?i)(AccountKey|account_key|storage_key)\s*[:=]\s*['"]?[A-Za-z0-9+/]{86}==

# Azure Connection String
DefaultEndpointsProtocol=https;AccountName=[^;]+;AccountKey=[A-Za-z0-9+/]{86}==

# Azure AD Client Secret
(?i)(client_secret|clientSecret)\s*[:=]\s*['"][A-Za-z0-9~_.]{34,}['"]
```

### JWT (JSON Web Token)

```regex
# JWT 토큰 패턴
eyJ[A-Za-z0-9_-]{10,}\.eyJ[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}

# JWT 시크릿 하드코딩
(?i)(jwt.?secret|jwt.?key|token.?secret)\s*[:=]\s*['"][^'"]{8,}['"]
```

### Slack

```regex
# Slack Bot Token
xoxb-[0-9]{10,}-[0-9]{10,}-[a-zA-Z0-9]{24}

# Slack User Token
xoxp-[0-9]{10,}-[0-9]{10,}-[0-9]{10,}-[a-f0-9]{32}

# Slack Webhook URL
https://hooks\.slack\.com/services/T[A-Z0-9]{8,}/B[A-Z0-9]{8,}/[a-zA-Z0-9]{24}

# Slack App Token
xapp-[0-9]-[A-Z0-9]{10,}-[0-9]{13}-[a-f0-9]{64}
```

### Stripe

```regex
# Stripe Secret Key
sk_(live|test)_[a-zA-Z0-9]{24,}

# Stripe Publishable Key (일반적으로 안전하지만 기록)
pk_(live|test)_[a-zA-Z0-9]{24,}

# Stripe Restricted Key
rk_(live|test)_[a-zA-Z0-9]{24,}
```

### SendGrid

```regex
# SendGrid API Key
SG\.[a-zA-Z0-9_-]{22}\.[a-zA-Z0-9_-]{43}
```

### Twilio

```regex
# Twilio API Key
SK[a-f0-9]{32}

# Twilio Account SID
AC[a-f0-9]{32}

# Twilio Auth Token
(?i)(twilio.?auth.?token|twilio.?token)\s*[:=]\s*['"][a-f0-9]{32}['"]
```

### Private Keys

```regex
# RSA Private Key
-----BEGIN RSA PRIVATE KEY-----

# DSA Private Key
-----BEGIN DSA PRIVATE KEY-----

# EC Private Key
-----BEGIN EC PRIVATE KEY-----

# OpenSSH Private Key
-----BEGIN OPENSSH PRIVATE KEY-----

# PGP Private Key
-----BEGIN PGP PRIVATE KEY BLOCK-----

# PKCS#8 Private Key
-----BEGIN PRIVATE KEY-----

# Encrypted Private Key
-----BEGIN ENCRYPTED PRIVATE KEY-----
```

### 데이터베이스 연결 문자열

```regex
# MongoDB
mongodb(\+srv)?://[^/\s:]+:[^/\s@]+@[^/\s]+

# PostgreSQL
postgres(ql)?://[^/\s:]+:[^/\s@]+@[^/\s]+

# MySQL
mysql://[^/\s:]+:[^/\s@]+@[^/\s]+

# Redis
redis://:[^/\s@]+@[^/\s]+
redis://[^/\s:]+:[^/\s@]+@[^/\s]+

# MSSQL
(Server|Data\s*Source)\s*=\s*[^;]+;\s*(User\s*Id|uid)\s*=\s*[^;]+;\s*(Password|pwd)\s*=\s*[^;]+

# JDBC
jdbc:(mysql|postgresql|oracle|sqlserver)://[^/\s]+.*password\s*=\s*[^&\s]+
```

### 일반 패턴

```regex
# 일반 API 키 할당
(?i)(api[_-]?key|apikey|api[_-]?secret)\s*[:=]\s*['"][A-Za-z0-9\-_.]{16,}['"]

# 일반 시크릿 할당
(?i)(secret|token|credential|auth)\s*[:=]\s*['"][A-Za-z0-9\-_.]{16,}['"]

# 일반 패스워드 할당
(?i)(password|passwd|pwd|pass)\s*[:=]\s*['"][^'"]{4,}['"]

# Bearer 토큰 하드코딩
(?i)Authorization\s*[:=]\s*['"]Bearer\s+[A-Za-z0-9\-_.]+['"]

# Basic Auth 하드코딩
(?i)Authorization\s*[:=]\s*['"]Basic\s+[A-Za-z0-9+/=]+['"]
```

---

## 3. 언어별 위험 함수/패턴 목록

### JavaScript / TypeScript

| 함수/패턴 | 위험 유형 | 심각도 | 안전한 대안 |
|----------|----------|--------|-----------|
| `eval(expr)` | 코드 실행 | Critical | `JSON.parse()`, 전용 파서 |
| `new Function(code)` | 코드 실행 | Critical | 정적 함수 정의 |
| `setTimeout(string, ms)` | 코드 실행 | Medium | `setTimeout(fn, ms)` |
| `setInterval(string, ms)` | 코드 실행 | Medium | `setInterval(fn, ms)` |
| `innerHTML = value` | XSS | High | `textContent`, DOMPurify |
| `outerHTML = value` | XSS | High | `textContent`, DOMPurify |
| `document.write(html)` | XSS | High | DOM API 사용 |
| `dangerouslySetInnerHTML` | XSS | High | DOMPurify 적용 |
| `child_process.exec(cmd)` | 명령어 인젝션 | High | `execFile()` + 인자 분리 |
| `require(variable)` | 동적 로드 | Medium | 정적 import |
| `__proto__` 접근 | Prototype Pollution | High | `Object.create(null)`, `Map` |
| `JSON.parse(untrusted)` | DoS (거대 페이로드) | Low | 크기 제한 적용 |
| `new RegExp(userInput)` | ReDoS | Medium | 정적 정규식, 입력 검증 |
| `crypto.createCipher()` | 약한 암호화 | Medium | `crypto.createCipheriv()` |
| `Math.random()` (보안) | 약한 난수 | Medium | `crypto.randomBytes()` |
| `window.location = input` | 오픈 리다이렉트 | Medium | URL 화이트리스트 |
| `postMessage('*')` | 정보 노출 | Medium | 특정 origin 지정 |

### Python

| 함수/패턴 | 위험 유형 | 심각도 | 안전한 대안 |
|----------|----------|--------|-----------|
| `eval(expr)` | 코드 실행 | Critical | `ast.literal_eval()` |
| `exec(code)` | 코드 실행 | Critical | 전용 파서, 제한된 실행 |
| `pickle.loads(data)` | 역직렬화 RCE | Critical | `json.loads()` |
| `pickle.load(file)` | 역직렬화 RCE | Critical | `json.load()` |
| `yaml.load(data)` | 역직렬화 | High | `yaml.safe_load()` |
| `os.system(cmd)` | 명령어 인젝션 | High | `subprocess.run(list)` |
| `os.popen(cmd)` | 명령어 인젝션 | High | `subprocess.run(list)` |
| `subprocess(shell=True)` | 명령어 인젝션 | High | `shell=False` + 리스트 인자 |
| `input()` (Python 2) | 코드 실행 | High | `raw_input()` 또는 Python 3 |
| `__import__(name)` | 동적 임포트 | Medium | 정적 import |
| `marshal.loads(data)` | 역직렬화 | High | `json.loads()` |
| `tempfile.mktemp()` | Race condition | Medium | `tempfile.mkstemp()` |
| `assert condition` | 보안 검증 | Medium | `if not: raise` |
| `random.random()` (보안) | 약한 난수 | Medium | `secrets.token_hex()` |
| `hashlib.md5()` | 약한 해시 | Medium | `hashlib.sha256()` |
| `compile() + exec()` | 코드 실행 | Critical | 정적 코드 |
| `string.Template` | SSTI | Medium | Jinja2 sandbox |

### Java

| 함수/패턴 | 위험 유형 | 심각도 | 안전한 대안 |
|----------|----------|--------|-----------|
| `Runtime.exec(cmd)` | 명령어 인젝션 | High | `ProcessBuilder` + 리스트 인자 |
| `createStatement()` | SQL 인젝션 | Critical | `prepareStatement()` |
| `executeQuery(str+var)` | SQL 인젝션 | Critical | `PreparedStatement` 바인딩 |
| `ObjectInputStream.readObject()` | 역직렬화 RCE | Critical | JSON/Protobuf, 화이트리스트 |
| `XMLInputFactory` (기본) | XXE | High | `setFeature(DISALLOW_DTD)` |
| `DocumentBuilder` (기본) | XXE | High | DTD 비활성화 |
| `SAXParser` (기본) | XXE | High | 외부 엔티티 비활성화 |
| `java.util.Random` | 약한 난수 | Medium | `SecureRandom` |
| `X509TrustManager` 빈 구현 | 인증서 무시 | Critical | 기본 TrustManager 사용 |
| `HostnameVerifier` 빈 구현 | 호스트명 무시 | Critical | 기본 HostnameVerifier 사용 |
| `addJavascriptInterface` | WebView RCE | High | `@JavascriptInterface` 제한 |
| `setAllowFileAccess(true)` | WebView 파일 | High | 필요 시에만 제한적 허용 |
| `Thread.sleep()` (동기화) | Race condition | Medium | 적절한 동기화 메커니즘 |
| `String.format(sql)` | SQL 인젝션 | High | `PreparedStatement` |

### SQL

| 패턴 | 위험 유형 | 심각도 | 안전한 대안 |
|------|----------|--------|-----------|
| `GRANT ALL PRIVILEGES` | 과도한 권한 | High | 최소 필요 권한만 부여 |
| `WITH GRANT OPTION` | 권한 위임 | High | 관리자만 GRANT OPTION |
| `EXECUTE AS` | 권한 상승 | Medium | 명시적 권한 관리 |
| `xp_cmdshell` | OS 명령 실행 | Critical | 사용 금지, 비활성화 |
| `LOAD_FILE()` | 파일 읽기 | High | 앱 레벨 파일 처리 |
| `INTO OUTFILE` | 파일 쓰기 | High | 앱 레벨 파일 처리 |
| `OPENROWSET` | 원격 접근 | High | Linked Server 제한 |
| `BULK INSERT` | 파일 로드 | Medium | 앱 레벨 데이터 로드 |
| `sp_configure` | 서버 설정 변경 | High | 관리자 접근 제한 |
| `DBCC` | 데이터베이스 관리 | Medium | 관리자 접근 제한 |

---

## 4. False Positive 판단 기준

오탐(false positive)을 최소화하기 위한 판단 기준입니다.

### 제외 조건 (False Positive로 판정)

#### 시크릿 탐지

| 조건 | 설명 | 예시 |
|------|------|------|
| 테스트/예시 파일 | 테스트 코드의 더미 값 | `test_api_key = "test-key-12345"` |
| 주석 내 설명 | 코드 주석에 포함된 예시 | `// API_KEY=your-key-here` |
| 환경 변수 참조 | 실제 값이 아닌 변수 참조 | `os.environ.get("API_KEY")` |
| 설정 템플릿 | `.env.example`, `.env.template` 파일 | `.env.example` 내 `API_KEY=` |
| 문서 파일 | `.md`, `.txt`, `.rst` 파일 | README의 설정 가이드 |
| 플레이스홀더 | 명백한 예시 값 | `"your-api-key-here"`, `"xxx"`, `"TODO"` |
| 공개 키 | Public key (private이 아닌) | `-----BEGIN PUBLIC KEY-----` |
| 해시 결과 | 해시된 값 (원본이 아닌) | `password_hash = bcrypt.hash(...)` |

**플레이스홀더 탐지 정규식**:
```regex
(?i)(your[_-]?(api[_-]?)?key|example|placeholder|dummy|fake|test|sample|todo|fixme|changeme|replace[_-]?me|insert[_-]?here|xxx+|\.\.\.|\*{3,})
```

#### 코드 패턴

| 조건 | 설명 | 예시 |
|------|------|------|
| 테스트 파일 | `*_test.*`, `*.test.*`, `*.spec.*` | `user.test.js`의 eval 사용 |
| Mock/Stub | 테스트 더블 | `jest.mock()` 내부의 패턴 |
| 비활성 코드 | 주석 처리된 코드 | `// eval(dangerous)` |
| 안전한 래퍼 | 검증 로직이 포함된 래퍼 | `safeEval(input)` (내부에서 검증) |
| 리터럴 전용 | 사용자 입력이 아닌 리터럴 | `eval("1 + 1")` (정적 문자열) |
| 라이브러리 코드 | node_modules, vendor 등 | 외부 라이브러리 내부 코드 |

### 컨텍스트 분석 규칙

1. **데이터 흐름 추적**: 사용자 입력이 실제로 위험한 함수에 도달하는지 확인
   - 입력 -> 검증/살균 -> 사용: 안전 (FP)
   - 입력 -> 직접 사용: 위험 (TP)

2. **스코프 확인**: 함수의 호출 컨텍스트 확인
   - 내부 전용 함수에서의 사용: 심각도 하향
   - 외부 노출 API에서의 사용: 심각도 유지

3. **프레임워크 보호**: 프레임워크의 내장 보호 기능 확인
   - ORM 사용 시 SQL Injection: FP 가능성 높음
   - React JSX의 자동 이스케이핑: innerHTML 미사용 시 FP

4. **파일 경로 기반 판단**:
   ```
   # FP 가능성 높은 경로
   */test/*
   */tests/*
   */__tests__/*
   */spec/*
   */mock/*
   */fixture/*
   */example/*
   */demo/*
   */docs/*
   */node_modules/*
   */vendor/*
   */.git/*
   */dist/*
   */build/*
   ```

### 심각도 조정 규칙

| 조건 | 조정 |
|------|------|
| 테스트 파일에서 발견 | 심각도 2단계 하향 (최소 Info) |
| 주석 내 발견 | Info로 하향 |
| 개발 전용 설정 (devDependencies) | 심각도 1단계 하향 |
| 내부 네트워크 전용 서비스 | 심각도 1단계 하향 |
| 프레임워크 보호 적용 | 심각도 1단계 하향 또는 FP |

---

## 5. 심각도 분류 매트릭스

### 분류 기준표

| 심각도 | CVSS 범위 | 공격 복잡도 | 영향 범위 | 대응 시간 | 색상 코드 |
|--------|-----------|-----------|----------|----------|----------|
| Critical | 9.0 - 10.0 | 낮음 | 시스템 전체 | 즉시 | 빨강 |
| High | 7.0 - 8.9 | 낮음~중간 | 주요 기능 | 48시간 | 주황 |
| Medium | 4.0 - 6.9 | 중간 | 일부 기능 | 1주일 | 노랑 |
| Low | 0.1 - 3.9 | 높음 | 제한적 | 1개월 | 파랑 |
| Info | N/A | N/A | 없음 | 계획적 | 회색 |

### 취약점 유형별 기본 심각도

| 취약점 유형 | 기본 심각도 | 조건부 상향 | 조건부 하향 |
|------------|-----------|-----------|-----------|
| Remote Code Execution (RCE) | Critical | - | 테스트 코드 |
| SQL Injection | Critical | - | ORM 사용 시 Medium |
| 프로덕션 자격증명 노출 | Critical | - | 개발 환경 전용 시 High |
| Private Key 노출 | Critical | - | 테스트용 키 시 High |
| Command Injection | High | 외부 입력 직접 사용 시 Critical | shell=False 시 Low |
| XSS (Stored) | High | 인증 없는 페이지 시 Critical | CSP 적용 시 Medium |
| XSS (Reflected) | High | - | CSP 적용 시 Medium |
| SSRF | High | 내부 네트워크 접근 시 Critical | URL 화이트리스트 시 Medium |
| 안전하지 않은 역직렬화 | High | 외부 입력 시 Critical | 내부 데이터 시 Medium |
| 약한 암호화 | Medium | 비밀번호 해시 시 High | 로그 해시 시 Low |
| 디버그 모드 | Medium | 프로덕션 배포 시 High | 개발 환경 시 Low |
| 보안 헤더 미설정 | Medium | - | 내부 서비스 시 Low |
| 정보 노출 | Low | 시스템 구성 노출 시 Medium | 버전 정보만 시 Info |
| 모범 사례 미준수 | Info | 보안 관련 시 Low | - |

### 공격 복잡도 평가

| 요소 | 낮은 복잡도 (위험) | 높은 복잡도 (덜 위험) |
|------|-----------------|-------------------|
| 인증 필요 여부 | 인증 불필요 | 인증 필요 |
| 네트워크 접근 | 인터넷 노출 | 내부 네트워크만 |
| 사용자 상호작용 | 불필요 | 필요 (클릭 등) |
| 특수 조건 | 없음 | 특정 설정 필요 |
| 익스플로잇 가용성 | 공개 익스플로잇 존재 | 없음 |

### 우선순위 결정 공식

```
최종 심각도 = 기본 심각도 + 컨텍스트 조정

컨텍스트 가중치:
  +1 단계: 프로덕션 환경, 인터넷 노출, 인증 불필요
  -1 단계: 개발 환경, 내부 네트워크, 인증 필요
  -2 단계: 테스트 코드, 주석 내 발견
```

---

## 6. 분석 실행 순서

보안 스캔 시 다음 순서로 분석을 실행합니다:

### 1단계: 파일 수집 및 분류

```bash
# 스캔 대상 파일 수집 (바이너리 및 외부 모듈 제외)
find . -type f \
    -not -path "*/.git/*" \
    -not -path "*/node_modules/*" \
    -not -path "*/vendor/*" \
    -not -path "*/dist/*" \
    -not -path "*/build/*" \
    -not -path "*/__pycache__/*" \
    -not -path "*/.venv/*" \
    -not -name "*.min.js" \
    -not -name "*.min.css" \
    -not -name "*.map" \
    | head -5000
```

### 2단계: 시크릿 탐지 (최우선)

시크릿 노출은 즉시 대응이 필요하므로 가장 먼저 실행합니다.

### 3단계: 언어별 위험 함수 스캔

파일 확장자 기반으로 해당 언어의 위험 함수 패턴을 매칭합니다.

### 4단계: OWASP 카테고리별 분석

A01부터 A10까지 순서대로 분석합니다.

### 5단계: 의존성 감사

패키지 매니저의 감사 도구를 실행합니다.

### 6단계: 보안 설정 검증

HTTPS, CORS, CSP, 인증 미들웨어, Rate Limiting 등을 확인합니다.

### 7단계: False Positive 필터링

위의 FP 판단 기준을 적용하여 오탐을 제거합니다.

### 8단계: 심각도 분류 및 보고서 생성

매트릭스를 적용하여 최종 심각도를 결정하고, [references/templates.md](../templates.md)의 형식으로 보고서를 생성합니다.
