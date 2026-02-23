# Security Scanner 템플릿

이 문서는 보안 스캔 결과 보고서, 취약점별 수정 가이드, 의존성 감사 보고서, 시크릿 탐지 경고 메시지, pre-commit hook 설정의 템플릿을 정의합니다.

---

## 1. 보안 스캔 보고서 출력 형식

### 전체 보고서 템플릿

```markdown
# 보안 스캔 보고서

> 스캔 일시: {SCAN_DATE}
> 스캔 범위: {SCAN_SCOPE}  (예: git diff --staged, 전체 코드베이스)
> 스캔 대상: {FILE_COUNT}개 파일 ({LANGUAGE_SUMMARY})
> 프로젝트: {PROJECT_NAME}

---

## 요약

| 심각도 | 발견 건수 |
|--------|----------|
| Critical | {CRITICAL_COUNT} |
| High | {HIGH_COUNT} |
| Medium | {MEDIUM_COUNT} |
| Low | {LOW_COUNT} |
| Info | {INFO_COUNT} |
| **합계** | **{TOTAL_COUNT}** |

### 주요 발견 사항

{TOP_FINDINGS_SUMMARY}

---

## 상세 결과

### Critical 취약점

#### [{VULN_ID}] {VULN_TITLE}

- **심각도**: Critical
- **카테고리**: {OWASP_CATEGORY} (예: A03 - Injection)
- **파일**: `{FILE_PATH}:{LINE_NUMBER}`
- **패턴**: {PATTERN_DESCRIPTION}

**취약 코드**:
```{LANGUAGE}
{VULNERABLE_CODE}
```

**설명**:
{VULNERABILITY_DESCRIPTION}

**수정 가이드**:
```{LANGUAGE}
{FIXED_CODE}
```

**참조**:
- {REFERENCE_URL}

---

### High 취약점

(동일 형식 반복)

### Medium 취약점

(동일 형식 반복)

### Low 취약점

(동일 형식 반복)

### Info

(동일 형식 반복)

---

## 시크릿 탐지 결과

| # | 유형 | 파일 | 라인 | 심각도 | 상태 |
|---|------|------|------|--------|------|
| 1 | {SECRET_TYPE} | {FILE_PATH} | {LINE} | {SEVERITY} | {STATUS} |

---

## 의존성 감사 결과

| 패키지 | 현재 버전 | 취약점 | 심각도 | 수정 버전 | CVE |
|--------|----------|--------|--------|----------|-----|
| {PACKAGE} | {CURRENT} | {VULN_DESC} | {SEVERITY} | {FIX_VER} | {CVE_ID} |

---

## 보안 설정 점검

| 항목 | 상태 | 권장 사항 |
|------|------|----------|
| HTTPS 강제 | {STATUS} | {RECOMMENDATION} |
| CORS 설정 | {STATUS} | {RECOMMENDATION} |
| CSP 헤더 | {STATUS} | {RECOMMENDATION} |
| Rate Limiting | {STATUS} | {RECOMMENDATION} |
| 인증 미들웨어 | {STATUS} | {RECOMMENDATION} |
| 보안 헤더 | {STATUS} | {RECOMMENDATION} |

---

## 권장 조치

### 즉시 대응 (Critical)
1. {ACTION_ITEM_1}
2. {ACTION_ITEM_2}

### 단기 대응 (High - 48시간 내)
1. {ACTION_ITEM_3}
2. {ACTION_ITEM_4}

### 중기 대응 (Medium - 1주일 내)
1. {ACTION_ITEM_5}

### 장기 대응 (Low - 계획적)
1. {ACTION_ITEM_6}
```

### 간략 보고서 템플릿 (커밋 전 스캔)

```markdown
## 보안 스캔 결과 (커밋 전 검사)

> 스캔 일시: {SCAN_DATE}
> 변경 파일: {CHANGED_FILES_COUNT}개

### 결과 요약
- Critical: {CRITICAL_COUNT}건
- High: {HIGH_COUNT}건
- Medium: {MEDIUM_COUNT}건
- Low: {LOW_COUNT}건

{IF_CRITICAL_OR_HIGH}
### 커밋 차단 사유

다음 Critical/High 취약점이 발견되어 커밋을 권장하지 않습니다:

{BLOCKING_ISSUES_LIST}

각 항목의 수정 방법은 아래를 참조하세요.
{END_IF}

{IF_CLEAN}
보안 취약점이 발견되지 않았습니다. 커밋을 진행해도 안전합니다.
{END_IF}
```

---

## 2. 취약점별 수정 가이드 템플릿

### SQL Injection 수정 가이드

```markdown
## SQL Injection 수정 가이드

### 문제 설명
사용자 입력이 SQL 쿼리에 직접 연결되어 공격자가 임의의 SQL 문을 실행할 수 있습니다.
이를 통해 데이터 유출, 수정, 삭제 또는 서버 제어권 탈취가 가능합니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`
- 코드:
  ```{LANGUAGE}
  {VULNERABLE_CODE}
  ```

### 수정 방법

#### JavaScript (Node.js)
**수정 전**:
```javascript
const query = "SELECT * FROM users WHERE id = " + userId;
db.query(query);
```

**수정 후**:
```javascript
// 방법 1: 파라미터화된 쿼리 (Prepared Statement)
const query = "SELECT * FROM users WHERE id = ?";
db.query(query, [userId]);

// 방법 2: ORM 사용 (Sequelize)
const user = await User.findByPk(userId);

// 방법 3: 쿼리 빌더 (Knex.js)
const user = await knex('users').where('id', userId).first();
```

#### Python
**수정 전**:
```python
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

**수정 후**:
```python
# 방법 1: 파라미터화된 쿼리
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# 방법 2: ORM 사용 (SQLAlchemy)
user = session.query(User).filter(User.id == user_id).first()

# 방법 3: Django ORM
user = User.objects.get(id=user_id)
```

#### Java
**수정 전**:
```java
String query = "SELECT * FROM users WHERE id = " + userId;
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(query);
```

**수정 후**:
```java
// PreparedStatement 사용
String query = "SELECT * FROM users WHERE id = ?";
PreparedStatement pstmt = conn.prepareStatement(query);
pstmt.setInt(1, userId);
ResultSet rs = pstmt.executeQuery();
```

### 참조
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
```

### XSS 수정 가이드

```markdown
## XSS (Cross-Site Scripting) 수정 가이드

### 문제 설명
사용자 입력이 적절한 살균(sanitization) 없이 HTML에 삽입되어,
공격자가 악성 스크립트를 실행할 수 있습니다.
이를 통해 세션 탈취, 피싱, 악성코드 배포 등이 가능합니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`
- 코드:
  ```{LANGUAGE}
  {VULNERABLE_CODE}
  ```

### 수정 방법

#### innerHTML 사용 시
**수정 전**:
```javascript
element.innerHTML = userInput;
```

**수정 후**:
```javascript
// 방법 1: textContent 사용 (HTML 태그 불필요 시)
element.textContent = userInput;

// 방법 2: DOMPurify로 살균 (HTML 태그 필요 시)
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(userInput);

// 방법 3: DOM API 사용
const textNode = document.createTextNode(userInput);
element.appendChild(textNode);
```

#### React dangerouslySetInnerHTML 사용 시
**수정 전**:
```jsx
<div dangerouslySetInnerHTML={{ __html: userContent }} />
```

**수정 후**:
```jsx
// 방법 1: DOMPurify 적용
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userContent) }} />

// 방법 2: 마크다운 렌더러 사용 (마크다운 입력 시)
import ReactMarkdown from 'react-markdown';
<ReactMarkdown>{userContent}</ReactMarkdown>

// 방법 3: 일반 텍스트로 렌더링
<div>{userContent}</div>
```

#### 서버 측 렌더링 시
**수정 전**:
```javascript
// Express + EJS
res.render('page', { content: req.body.content });
// EJS 템플릿: <%- content %>  (미이스케이프)
```

**수정 후**:
```javascript
// EJS 템플릿: <%= content %>  (자동 이스케이프)
// 또는 수동 이스케이프 함수 적용
const escapeHtml = (str) => str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
```

### 추가 방어 조치
- Content-Security-Policy (CSP) 헤더 설정
- HttpOnly 플래그가 설정된 쿠키 사용
- X-XSS-Protection 헤더 설정

### 참조
- [OWASP XSS Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
```

### Command Injection 수정 가이드

```markdown
## Command Injection 수정 가이드

### 문제 설명
사용자 입력이 시스템 명령에 직접 삽입되어 공격자가 서버에서 임의 명령을 실행할 수 있습니다.
이를 통해 서버 제어권 탈취, 데이터 유출, 시스템 파괴 등이 가능합니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`

### 수정 방법

#### Node.js
**수정 전**:
```javascript
const { exec } = require('child_process');
exec(`convert ${filename} output.pdf`);
```

**수정 후**:
```javascript
// 방법 1: execFile 사용 (쉘 미경유)
const { execFile } = require('child_process');
execFile('convert', [filename, 'output.pdf']);

// 방법 2: spawn 사용
const { spawn } = require('child_process');
const proc = spawn('convert', [filename, 'output.pdf']);

// 방법 3: 입력 화이트리스트 검증 추가
const allowedChars = /^[a-zA-Z0-9._-]+$/;
if (!allowedChars.test(filename)) {
    throw new Error('유효하지 않은 파일명');
}
```

#### Python
**수정 전**:
```python
import subprocess
subprocess.call(f"convert {filename} output.pdf", shell=True)
```

**수정 후**:
```python
import subprocess
import shlex

# 방법 1: shell=False + 리스트 인자
subprocess.call(["convert", filename, "output.pdf"], shell=False)

# 방법 2: shlex.quote로 이스케이프 (shell=True 필수인 경우)
subprocess.call(f"convert {shlex.quote(filename)} output.pdf", shell=True)

# 방법 3: 입력 화이트리스트 검증
import re
if not re.match(r'^[a-zA-Z0-9._-]+$', filename):
    raise ValueError("유효하지 않은 파일명")
subprocess.call(["convert", filename, "output.pdf"])
```

### 참조
- [OWASP OS Command Injection Defense](https://cheatsheetseries.owasp.org/cheatsheets/OS_Command_Injection_Defense_Cheat_Sheet.html)
```

### 시크릿 노출 수정 가이드

```markdown
## 시크릿/자격증명 노출 수정 가이드

### 문제 설명
API 키, 비밀번호, 토큰 등이 소스 코드에 하드코딩되어 있습니다.
코드가 유출되면 해당 자격증명을 통해 서비스에 무단 접근이 가능합니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`
- 유형: {SECRET_TYPE}

### 수정 방법

#### 1단계: 환경 변수로 이전

**수정 전**:
```python
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
DATABASE_URL = "postgres://admin:password@db.example.com/mydb"
```

**수정 후**:
```python
import os

AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY_ID")
DATABASE_URL = os.environ.get("DATABASE_URL")

# 필수 환경 변수 검증
if not AWS_ACCESS_KEY:
    raise EnvironmentError("AWS_ACCESS_KEY_ID 환경 변수가 설정되지 않았습니다")
```

#### 2단계: .env 파일 설정

```bash
# .env 파일 생성 (Git에 커밋하지 않음)
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
DATABASE_URL=postgres://admin:password@db.example.com/mydb
```

#### 3단계: .gitignore에 .env 추가

```gitignore
# 환경 변수 파일
.env
.env.local
.env.production
.env.*.local

# 키 파일
*.pem
*.key
*.p12
*.pfx
```

#### 4단계: .env.example 생성 (팀 공유용)

```bash
# .env.example - 실제 값을 포함하지 않는 템플릿
AWS_ACCESS_KEY_ID=your-access-key-here
DATABASE_URL=postgres://user:password@host:5432/dbname
```

#### 5단계: 노출된 자격증명 교체

노출된 자격증명은 반드시 즉시 교체(rotate)해야 합니다:

- **AWS**: IAM 콘솔에서 Access Key 재발급
- **GitHub**: Settings > Developer settings에서 토큰 재발급
- **데이터베이스**: 비밀번호 변경 및 기존 세션 종료

#### 6단계: Git 히스토리에서 제거

```bash
# BFG Repo-Cleaner 사용
bfg --delete-files .env
bfg --replace-text passwords.txt

# 또는 git filter-branch (주의: 히스토리 재작성)
git filter-branch --force --index-filter \
    'git rm --cached --ignore-unmatch .env' \
    --prune-empty --tag-name-filter cat -- --all
```

### 참조
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
```

### 역직렬화 취약점 수정 가이드

```markdown
## 안전하지 않은 역직렬화 수정 가이드

### 문제 설명
신뢰할 수 없는 데이터를 역직렬화하면 임의 코드가 실행될 수 있습니다.
공격자가 조작된 직렬화 데이터를 전송하여 서버에서 원격 코드 실행(RCE)이 가능합니다.

### 발견 위치
- 파일: `{FILE_PATH}:{LINE_NUMBER}`

### 수정 방법

#### Python - pickle
**수정 전**:
```python
import pickle
data = pickle.loads(user_data)  # 위험: RCE 가능
```

**수정 후**:
```python
import json
data = json.loads(user_data)  # 안전: 코드 실행 불가

# 또는 marshmallow로 스키마 검증
from marshmallow import Schema, fields
class UserSchema(Schema):
    name = fields.Str(required=True)
    age = fields.Int(required=True)

schema = UserSchema()
data = schema.loads(user_data)
```

#### Python - YAML
**수정 전**:
```python
import yaml
data = yaml.load(content)  # 위험: 코드 실행 가능
```

**수정 후**:
```python
import yaml
data = yaml.safe_load(content)  # 안전: SafeLoader 사용
```

#### Java
**수정 전**:
```java
ObjectInputStream ois = new ObjectInputStream(inputStream);
Object obj = ois.readObject();  // 위험: 역직렬화 공격
```

**수정 후**:
```java
// 방법 1: JSON 사용
ObjectMapper mapper = new ObjectMapper();
User user = mapper.readValue(jsonString, User.class);

// 방법 2: 화이트리스트 기반 역직렬화
ObjectInputStream ois = new ValidatingObjectInputStream(inputStream);
((ValidatingObjectInputStream) ois).accept(User.class);
Object obj = ois.readObject();
```

### 참조
- [OWASP Deserialization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html)
```

### SSRF 수정 가이드

```markdown
## SSRF (Server-Side Request Forgery) 수정 가이드

### 문제 설명
사용자가 제공한 URL로 서버가 요청을 보내어 내부 네트워크의 리소스에
접근하거나 클라우드 메타데이터를 탈취할 수 있습니다.

### 수정 방법

**수정 전**:
```javascript
app.get('/fetch', async (req, res) => {
    const response = await fetch(req.query.url);  // 위험
    res.json(await response.json());
});
```

**수정 후**:
```javascript
const { URL } = require('url');

// URL 화이트리스트
const ALLOWED_HOSTS = ['api.example.com', 'cdn.example.com'];

// 내부 IP 차단
function isInternalIP(hostname) {
    const ip = hostname;
    return ip.startsWith('10.') ||
           ip.startsWith('172.16.') || ip.startsWith('172.17.') ||
           ip.startsWith('192.168.') ||
           ip === '127.0.0.1' || ip === 'localhost' ||
           ip === '0.0.0.0' ||
           ip.startsWith('169.254.');  // AWS 메타데이터
}

app.get('/fetch', async (req, res) => {
    try {
        const parsedUrl = new URL(req.query.url);

        // 프로토콜 검증
        if (!['http:', 'https:'].includes(parsedUrl.protocol)) {
            return res.status(400).json({ error: '허용되지 않는 프로토콜' });
        }

        // 호스트 화이트리스트 확인
        if (!ALLOWED_HOSTS.includes(parsedUrl.hostname)) {
            return res.status(400).json({ error: '허용되지 않는 호스트' });
        }

        // 내부 IP 차단
        if (isInternalIP(parsedUrl.hostname)) {
            return res.status(400).json({ error: '내부 네트워크 접근 불가' });
        }

        const response = await fetch(parsedUrl.toString());
        res.json(await response.json());
    } catch (error) {
        res.status(400).json({ error: '유효하지 않은 URL' });
    }
});
```

### 참조
- [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
```

### Broken Access Control 수정 가이드

```markdown
## Broken Access Control 수정 가이드

### 문제 설명
API 엔드포인트에 적절한 인증/인가 검사가 적용되지 않아
권한이 없는 사용자가 리소스에 접근할 수 있습니다.

### 수정 방법

#### Express.js - 인증 미들웨어 적용
**수정 전**:
```javascript
// 인증 없이 사용자 데이터 접근
app.get('/api/users/:id', async (req, res) => {
    const user = await User.findById(req.params.id);
    res.json(user);
});
```

**수정 후**:
```javascript
// 인증 + 인가 미들웨어 적용
const authenticate = require('./middleware/auth');
const authorize = require('./middleware/authorize');

app.get('/api/users/:id', authenticate, async (req, res) => {
    // 본인 또는 관리자만 접근 가능
    if (req.user.id !== req.params.id && req.user.role !== 'admin') {
        return res.status(403).json({ error: '접근 권한이 없습니다' });
    }
    const user = await User.findById(req.params.id);
    res.json(user);
});
```

#### IDOR 방지
**수정 전**:
```javascript
// 직접 객체 참조 - 다른 사용자의 주문 접근 가능
app.get('/api/orders/:orderId', authenticate, async (req, res) => {
    const order = await Order.findById(req.params.orderId);
    res.json(order);
});
```

**수정 후**:
```javascript
// 소유자 확인 추가
app.get('/api/orders/:orderId', authenticate, async (req, res) => {
    const order = await Order.findOne({
        _id: req.params.orderId,
        userId: req.user.id  // 소유자 확인
    });
    if (!order) {
        return res.status(404).json({ error: '주문을 찾을 수 없습니다' });
    }
    res.json(order);
});
```

### 참조
- [OWASP Access Control Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Access_Control_Cheat_Sheet.html)
```

---

## 3. 의존성 감사 보고서 형식

### npm audit 보고서 템플릿

```markdown
## 의존성 보안 감사 보고서 (npm)

> 감사 일시: {AUDIT_DATE}
> 프로젝트: {PROJECT_NAME}
> 패키지 매니저: npm {NPM_VERSION}
> 총 패키지 수: {TOTAL_PACKAGES}

### 요약

| 심각도 | 건수 |
|--------|------|
| Critical | {CRITICAL_COUNT} |
| High | {HIGH_COUNT} |
| Moderate | {MODERATE_COUNT} |
| Low | {LOW_COUNT} |

### 취약 패키지 상세

#### {PACKAGE_NAME}@{CURRENT_VERSION}

- **심각도**: {SEVERITY}
- **취약점**: {VULNERABILITY_TITLE}
- **CVE**: {CVE_ID}
- **경로**: {DEPENDENCY_PATH}
- **영향**: {IMPACT_DESCRIPTION}
- **수정 버전**: {FIXED_VERSION}
- **수정 명령**:
  ```bash
  npm install {PACKAGE_NAME}@{FIXED_VERSION}
  # 또는
  npm audit fix
  ```

### 자동 수정 가능 항목

```bash
# 자동 수정 실행
npm audit fix

# Breaking change 포함 수정
npm audit fix --force
```

### 자동 수정 불가 항목

수동으로 패키지 업데이트 또는 대체 라이브러리 사용이 필요합니다:

| 패키지 | 현재 버전 | 원인 | 권장 조치 |
|--------|----------|------|----------|
| {PACKAGE} | {VERSION} | {REASON} | {RECOMMENDATION} |
```

### pip audit 보고서 템플릿

```markdown
## 의존성 보안 감사 보고서 (pip)

> 감사 일시: {AUDIT_DATE}
> 프로젝트: {PROJECT_NAME}
> Python 버전: {PYTHON_VERSION}
> 총 패키지 수: {TOTAL_PACKAGES}

### 취약 패키지

| 패키지 | 현재 버전 | CVE | 심각도 | 설명 | 수정 버전 |
|--------|----------|-----|--------|------|----------|
| {PACKAGE} | {VERSION} | {CVE} | {SEVERITY} | {DESCRIPTION} | {FIX_VERSION} |

### 수정 명령

```bash
# 특정 패키지 업데이트
pip install {PACKAGE}=={FIX_VERSION}

# requirements.txt 업데이트 후 재설치
pip install -r requirements.txt --upgrade
```
```

---

## 4. 시크릿 탐지 경고 메시지

### Critical 시크릿 발견 시

```markdown
## [CRITICAL] 시크릿 노출 감지

하드코딩된 자격증명이 발견되었습니다. 즉시 조치가 필요합니다.

### 발견 내용

| # | 유형 | 파일 | 라인 | 마스킹된 값 |
|---|------|------|------|-----------|
| 1 | {SECRET_TYPE} | `{FILE_PATH}` | {LINE} | `{MASKED_VALUE}` |

### 즉시 조치 사항

1. **커밋을 중단하세요** - 이 자격증명이 Git 히스토리에 포함되면 안 됩니다
2. **자격증명을 교체하세요** - 노출된 키/토큰을 즉시 무효화하고 새로 발급받으세요
3. **환경 변수로 이전하세요** - 코드에서 자격증명을 제거하고 환경 변수를 사용하세요
4. **.gitignore를 업데이트하세요** - `.env` 파일이 추적되지 않도록 하세요

### 자격증명 교체 방법

{SECRET_TYPE}에 따른 교체 절차:

#### AWS Access Key
1. AWS IAM 콘솔 접속
2. 해당 사용자의 Access Key 비활성화
3. 새 Access Key 생성
4. 환경 변수 또는 AWS Credentials 파일 업데이트

#### GitHub Token
1. GitHub Settings > Developer settings > Personal access tokens
2. 노출된 토큰 삭제 (Revoke)
3. 새 토큰 생성
4. 환경 변수 업데이트

#### 데이터베이스 자격증명
1. 데이터베이스 서버에서 비밀번호 변경
2. 기존 세션 모두 종료
3. 환경 변수 업데이트
4. 애플리케이션 재시작
```

### .env 파일 커밋 시도 경고

```markdown
## [CRITICAL] .env 파일 커밋 시도 감지

`.env` 파일이 스테이징 영역에 포함되어 있습니다.
이 파일에는 민감한 환경 변수가 포함될 수 있으므로 커밋해서는 안 됩니다.

### 감지된 파일
- `{ENV_FILE_PATH}`

### 조치 방법

```bash
# 1. 스테이징에서 제거
git reset HEAD {ENV_FILE_PATH}

# 2. .gitignore에 추가
echo ".env" >> .gitignore
echo ".env.local" >> .gitignore
echo ".env.production" >> .gitignore

# 3. 이미 트래킹 중인 경우 트래킹 해제
git rm --cached {ENV_FILE_PATH}

# 4. .env.example 생성 (팀 공유용)
# 실제 값 대신 플레이스홀더 사용
```
```

### Private Key 파일 감지 경고

```markdown
## [CRITICAL] Private Key 파일 감지

Private Key 파일이 프로젝트에 포함되어 있습니다.
이 파일이 Git 저장소에 커밋되면 보안 사고로 이어질 수 있습니다.

### 감지된 파일
- `{KEY_FILE_PATH}`
- 유형: {KEY_TYPE} (RSA/EC/DSA/OpenSSH)

### 조치 방법

1. **파일을 안전한 위치로 이동하세요** (프로젝트 디렉토리 외부)
2. **.gitignore에 추가하세요**:
   ```gitignore
   *.pem
   *.key
   *.p12
   *.pfx
   id_rsa
   id_ecdsa
   id_ed25519
   ```
3. **이미 커밋된 경우 Git 히스토리에서 제거하세요**
4. **해당 키를 교체(rotate)하세요** - 노출된 키는 더 이상 안전하지 않습니다
```

---

## 5. Pre-commit Hook 설정 예시

### Git Pre-commit Hook (쉘 스크립트)

```bash
#!/bin/bash
# .git/hooks/pre-commit
# 보안 스캔 Pre-commit Hook
# 설치: cp pre-commit .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit

set -e

echo "========================================="
echo "  보안 스캔 실행 중..."
echo "========================================="

STAGED_FILES=$(git diff --staged --name-only --diff-filter=ACMR)
ISSUES_FOUND=0

# ===== 1. .env 파일 커밋 방지 =====
ENV_FILES=$(echo "$STAGED_FILES" | grep -i '\.env' || true)
if [ -n "$ENV_FILES" ]; then
    echo ""
    echo "[CRITICAL] .env 파일이 스테이징되어 있습니다:"
    echo "$ENV_FILES"
    echo "  -> git reset HEAD <파일명> 으로 스테이징에서 제거하세요."
    ISSUES_FOUND=1
fi

# ===== 2. Private Key 파일 커밋 방지 =====
KEY_FILES=$(echo "$STAGED_FILES" | grep -E '\.(pem|key|p12|pfx)$|id_rsa|id_ecdsa|id_ed25519' || true)
if [ -n "$KEY_FILES" ]; then
    echo ""
    echo "[CRITICAL] Private Key 파일이 스테이징되어 있습니다:"
    echo "$KEY_FILES"
    echo "  -> .gitignore에 추가하고 스테이징에서 제거하세요."
    ISSUES_FOUND=1
fi

# ===== 3. 하드코딩된 시크릿 탐지 =====
if [ -n "$STAGED_FILES" ]; then
    # AWS Access Key
    AWS_KEYS=$(git diff --staged -U0 | grep -E '^\+.*AKIA[0-9A-Z]{16}' || true)
    if [ -n "$AWS_KEYS" ]; then
        echo ""
        echo "[CRITICAL] AWS Access Key가 코드에 포함되어 있습니다."
        echo "  -> 환경 변수를 사용하세요."
        ISSUES_FOUND=1
    fi

    # GitHub Token
    GH_TOKENS=$(git diff --staged -U0 | grep -E '^\+.*(ghp_[a-zA-Z0-9]{36}|github_pat_)' || true)
    if [ -n "$GH_TOKENS" ]; then
        echo ""
        echo "[CRITICAL] GitHub 토큰이 코드에 포함되어 있습니다."
        echo "  -> 환경 변수를 사용하세요."
        ISSUES_FOUND=1
    fi

    # Private Key 내용
    PRIVATE_KEYS=$(git diff --staged -U0 | grep -E '^\+.*BEGIN.*PRIVATE KEY' || true)
    if [ -n "$PRIVATE_KEYS" ]; then
        echo ""
        echo "[CRITICAL] Private Key가 코드에 포함되어 있습니다."
        echo "  -> 키 파일을 프로젝트 외부에 보관하세요."
        ISSUES_FOUND=1
    fi

    # 일반 패스워드 하드코딩
    PASSWORDS=$(git diff --staged -U0 | grep -iE '^\+.*(password|passwd|pwd)\s*[:=]\s*["\x27][^"\x27]{4,}["\x27]' | grep -viE '(example|placeholder|your|todo|changeme|xxx|env|process\.env|os\.environ|getenv)' || true)
    if [ -n "$PASSWORDS" ]; then
        echo ""
        echo "[HIGH] 하드코딩된 비밀번호가 발견되었습니다."
        echo "  -> 환경 변수를 사용하세요."
        ISSUES_FOUND=1
    fi

    # 데이터베이스 연결 문자열
    DB_STRINGS=$(git diff --staged -U0 | grep -E '^\+.*(mongodb|postgres|mysql|redis)://[^/]+:[^/]+@' || true)
    if [ -n "$DB_STRINGS" ]; then
        echo ""
        echo "[CRITICAL] 데이터베이스 연결 문자열에 자격증명이 포함되어 있습니다."
        echo "  -> 환경 변수를 사용하세요."
        ISSUES_FOUND=1
    fi

    # JWT 토큰
    JWT_TOKENS=$(git diff --staged -U0 | grep -E '^\+.*eyJ[A-Za-z0-9_-]{10,}\.eyJ[A-Za-z0-9_-]{10,}\.' || true)
    if [ -n "$JWT_TOKENS" ]; then
        echo ""
        echo "[HIGH] JWT 토큰이 코드에 하드코딩되어 있습니다."
        echo "  -> 환경 변수 또는 동적 생성을 사용하세요."
        ISSUES_FOUND=1
    fi
fi

# ===== 4. 위험 패턴 탐지 (JavaScript/TypeScript) =====
JS_FILES=$(echo "$STAGED_FILES" | grep -E '\.(js|jsx|ts|tsx|mjs|cjs)$' || true)
if [ -n "$JS_FILES" ]; then
    EVAL_USAGE=$(git diff --staged -U0 -- $JS_FILES | grep -E '^\+.*\beval\s*\(' | grep -v '// safe' | grep -v 'test' || true)
    if [ -n "$EVAL_USAGE" ]; then
        echo ""
        echo "[CRITICAL] eval() 사용이 감지되었습니다 (JavaScript/TypeScript)."
        echo "  -> JSON.parse() 또는 전용 파서를 사용하세요."
        ISSUES_FOUND=1
    fi

    INNERHTML=$(git diff --staged -U0 -- $JS_FILES | grep -E '^\+.*\.innerHTML\s*=' | grep -v 'DOMPurify' | grep -v 'sanitize' || true)
    if [ -n "$INNERHTML" ]; then
        echo ""
        echo "[HIGH] innerHTML 직접 할당이 감지되었습니다 (XSS 위험)."
        echo "  -> textContent 또는 DOMPurify를 사용하세요."
        ISSUES_FOUND=1
    fi
fi

# ===== 5. 위험 패턴 탐지 (Python) =====
PY_FILES=$(echo "$STAGED_FILES" | grep -E '\.py$' || true)
if [ -n "$PY_FILES" ]; then
    PICKLE_USAGE=$(git diff --staged -U0 -- $PY_FILES | grep -E '^\+.*pickle\.(loads?|Unpickler)\s*\(' || true)
    if [ -n "$PICKLE_USAGE" ]; then
        echo ""
        echo "[CRITICAL] pickle 역직렬화 사용이 감지되었습니다 (Python)."
        echo "  -> json.loads()를 사용하세요."
        ISSUES_FOUND=1
    fi

    SHELL_TRUE=$(git diff --staged -U0 -- $PY_FILES | grep -E '^\+.*subprocess\..*(shell\s*=\s*True)' || true)
    if [ -n "$SHELL_TRUE" ]; then
        echo ""
        echo "[HIGH] subprocess에서 shell=True 사용이 감지되었습니다 (Python)."
        echo "  -> shell=False와 리스트 인자를 사용하세요."
        ISSUES_FOUND=1
    fi

    YAML_UNSAFE=$(git diff --staged -U0 -- $PY_FILES | grep -E '^\+.*yaml\.load\s*\(' | grep -v 'SafeLoader' | grep -v 'safe_load' || true)
    if [ -n "$YAML_UNSAFE" ]; then
        echo ""
        echo "[HIGH] yaml.load() 사용이 감지되었습니다 (SafeLoader 미적용)."
        echo "  -> yaml.safe_load()를 사용하세요."
        ISSUES_FOUND=1
    fi
fi

# ===== 결과 출력 =====
echo ""
if [ $ISSUES_FOUND -eq 1 ]; then
    echo "========================================="
    echo "  보안 이슈가 발견되었습니다!"
    echo "  위의 문제를 수정한 후 다시 커밋하세요."
    echo "========================================="
    echo ""
    echo "스캔을 건너뛰려면: git commit --no-verify"
    echo "(주의: 보안 검사를 건너뛰는 것은 권장하지 않습니다)"
    exit 1
else
    echo "========================================="
    echo "  보안 스캔 통과! 이슈가 발견되지 않았습니다."
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
    "*.{js,jsx,ts,tsx}": [
      "eslint --rule 'no-eval: error' --rule 'no-implied-eval: error'",
      "bash -c 'grep -l \"innerHTML\\s*=\" \"$@\" && echo \"[ERROR] innerHTML 사용 감지\" && exit 1 || exit 0' --"
    ],
    "*.{env,pem,key}": [
      "bash -c 'echo \"[ERROR] 민감 파일 커밋 시도 감지: $@\" && exit 1' --"
    ]
  }
}
```

### pre-commit 프레임워크 설정 (Python)

```yaml
# .pre-commit-config.yaml
repos:
  # 시크릿 탐지
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
        exclude: 'package-lock\.json|\.env\.example'

  # Git 시크릿 탐지
  - repo: https://github.com/awslabs/git-secrets
    rev: master
    hooks:
      - id: git-secrets

  # 보안 린트 (Python)
  - repo: https://github.com/PyCQA/bandit
    rev: '1.7.5'
    hooks:
      - id: bandit
        args: ['-c', 'pyproject.toml']
        additional_dependencies: ['bandit[toml]']

  # 보안 린트 (JavaScript)
  - repo: local
    hooks:
      - id: eslint-security
        name: ESLint Security Check
        entry: npx eslint --no-eslintrc --rule 'no-eval: error' --rule 'no-implied-eval: error'
        language: system
        files: '\.(js|jsx|ts|tsx)$'

  # .env 파일 차단
  - repo: local
    hooks:
      - id: no-env-files
        name: Block .env files
        entry: bash -c 'echo "ERROR: .env 파일은 커밋할 수 없습니다" && exit 1'
        language: system
        files: '\.env$'
        exclude: '\.env\.example$|\.env\.template$'

  # Private Key 차단
  - repo: local
    hooks:
      - id: no-private-keys
        name: Block private key files
        entry: bash -c 'echo "ERROR: Private Key 파일은 커밋할 수 없습니다" && exit 1'
        language: system
        files: '\.(pem|key|p12|pfx)$|id_rsa|id_ecdsa|id_ed25519'
```

### GitHub Actions 보안 스캔 워크플로우

```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: 시크릿 탐지 (detect-secrets)
        uses: reviewdog/action-detect-secrets@v0.15
        with:
          reporter: github-pr-review

      - name: npm audit (Node.js)
        if: hashFiles('package-lock.json') != ''
        run: |
          npm audit --audit-level=high
        continue-on-error: true

      - name: pip audit (Python)
        if: hashFiles('requirements.txt') != ''
        run: |
          pip install pip-audit
          pip-audit -r requirements.txt
        continue-on-error: true

      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: '${{ github.repository }}'
          path: '.'
          format: 'HTML'

      - name: 결과 업로드
        uses: actions/upload-artifact@v4
        with:
          name: security-scan-report
          path: reports/
```

---

## 6. 보안 헤더 설정 템플릿

### Express.js (helmet)

```javascript
const helmet = require('helmet');

app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'", "'nonce-{RANDOM}'"],
            styleSrc: ["'self'", "'unsafe-inline'"],
            imgSrc: ["'self'", "data:", "https:"],
            connectSrc: ["'self'"],
            fontSrc: ["'self'"],
            objectSrc: ["'none'"],
            mediaSrc: ["'self'"],
            frameSrc: ["'none'"],
        },
    },
    crossOriginEmbedderPolicy: true,
    crossOriginOpenerPolicy: true,
    crossOriginResourcePolicy: { policy: "same-site" },
    dnsPrefetchControl: true,
    frameguard: { action: "deny" },
    hidePoweredBy: true,
    hsts: { maxAge: 31536000, includeSubDomains: true, preload: true },
    ieNoOpen: true,
    noSniff: true,
    referrerPolicy: { policy: "strict-origin-when-cross-origin" },
    xssFilter: true,
}));
```

### Nginx

```nginx
# 보안 헤더 설정
add_header X-Frame-Options "DENY" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self';" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;

# 서버 정보 숨기기
server_tokens off;
```
