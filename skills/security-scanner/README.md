# Security Scanner

## 스킬 소개

**Security Scanner**는 Claude Code 커스텀 스킬로, 코드 변경사항 및 전체 코드베이스에서 보안 취약점을 사전 탐지합니다. OWASP Top 10 보안 위험을 기반으로 취약점 패턴을 식별하고, 심각도별 분류와 함께 구체적인 수정 가이드를 제공합니다.

### 주요 기능

- **OWASP Top 10 기반 취약점 탐지**: 10가지 주요 보안 위험 카테고리별 패턴 매칭
- **시크릿/API 키 탐지**: 하드코딩된 자격증명, API 키, 토큰, Private Key 감지
- **다중 언어 지원**: JavaScript, TypeScript, Python, Java, SQL 등
- **의존성 보안 감사**: npm audit, pip audit, Maven dependency-check 연동
- **보안 설정 검증**: HTTPS, CORS, CSP, Rate Limiting 등 설정 점검
- **심각도별 분류**: Critical / High / Medium / Low / Info 5단계 분류
- **수정 가이드 제공**: 각 취약점별 구체적인 수정 방법 및 안전한 코드 예시

---

## OWASP Top 10 요약

OWASP(Open Web Application Security Project)는 웹 애플리케이션 보안에서 가장 치명적인 10가지 위험을 정의합니다. 이 스킬은 각 카테고리에 대해 코드 레벨에서 패턴을 탐지합니다.

| 순위 | 카테고리 | 설명 | 주요 탐지 대상 |
|------|---------|------|---------------|
| A01 | Broken Access Control | 접근 제어 취약점 | 인가 미들웨어 누락, IDOR, 디렉토리 트래버설 |
| A02 | Cryptographic Failures | 암호화 실패 | MD5/SHA1 사용, 하드코딩된 키, HTTP 사용 |
| A03 | Injection | 인젝션 | SQL/XSS/Command/NoSQL/LDAP 인젝션 |
| A04 | Insecure Design | 안전하지 않은 설계 | Rate limiting 미적용, Threat modeling 부재 |
| A05 | Security Misconfiguration | 보안 설정 오류 | 디버그 모드, 기본 자격증명, CORS 와일드카드 |
| A06 | Vulnerable Components | 취약한 구성 요소 | 알려진 CVE, EOL 버전, 취약한 의존성 |
| A07 | Auth Failures | 인증 실패 | 약한 비밀번호 정책, JWT 미검증, 세션 관리 |
| A08 | Integrity Failures | 무결성 실패 | 안전하지 않은 역직렬화, SRI 미사용 |
| A09 | Logging Failures | 로깅 실패 | 감사 로그 미적용, 민감 데이터 로그 출력 |
| A10 | SSRF | 서버 측 요청 위조 | 사용자 입력 기반 URL 요청, 내부 네트워크 접근 |

---

## 탐지 예시

### 예시 1: SQL Injection (A03)

**취약한 코드**:
```javascript
// Express.js - 사용자 입력을 SQL 쿼리에 직접 연결
app.get('/users', (req, res) => {
    const userId = req.query.id;
    const query = "SELECT * FROM users WHERE id = " + userId;
    db.query(query, (err, results) => {
        res.json(results);
    });
});
```

**스캔 결과**:
```
[CRITICAL] SQL Injection 취약점 발견
  파일: src/routes/users.js:4
  패턴: 문자열 연결을 통한 SQL 쿼리 구성
  카테고리: A03 - Injection
```

**수정된 코드**:
```javascript
// 파라미터화된 쿼리 사용
app.get('/users', (req, res) => {
    const userId = req.query.id;
    const query = "SELECT * FROM users WHERE id = ?";
    db.query(query, [userId], (err, results) => {
        res.json(results);
    });
});
```

---

### 예시 2: 하드코딩된 시크릿 (Secret Detection)

**취약한 코드**:
```python
# config.py - API 키가 코드에 하드코딩됨
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
DATABASE_URL = "postgres://admin:supersecret@db.example.com:5432/mydb"
```

**스캔 결과**:
```
[CRITICAL] AWS Access Key 하드코딩 발견
  파일: config.py:2
  패턴: AKIA[0-9A-Z]{16}

[CRITICAL] AWS Secret Key 하드코딩 발견
  파일: config.py:3
  패턴: 40자리 자격증명 문자열

[CRITICAL] 데이터베이스 연결 문자열 내 자격증명 노출
  파일: config.py:4
  패턴: postgres://[user]:[password]@[host]
```

**수정된 코드**:
```python
# config.py - 환경 변수 사용
import os

AWS_ACCESS_KEY = os.environ.get("AWS_ACCESS_KEY_ID")
AWS_SECRET_KEY = os.environ.get("AWS_SECRET_ACCESS_KEY")
DATABASE_URL = os.environ.get("DATABASE_URL")
```

---

### 예시 3: XSS (A03)

**취약한 코드**:
```jsx
// React 컴포넌트 - dangerouslySetInnerHTML 사용
function UserComment({ comment }) {
    return (
        <div dangerouslySetInnerHTML={{ __html: comment.body }} />
    );
}
```

**스캔 결과**:
```
[HIGH] XSS 취약점 - dangerouslySetInnerHTML 사용
  파일: src/components/UserComment.jsx:4
  패턴: 사용자 입력이 HTML로 직접 렌더링
  카테고리: A03 - Injection (XSS)
```

**수정된 코드**:
```jsx
// DOMPurify로 입력 살균
import DOMPurify from 'dompurify';

function UserComment({ comment }) {
    const sanitized = DOMPurify.sanitize(comment.body);
    return (
        <div dangerouslySetInnerHTML={{ __html: sanitized }} />
    );
}
```

---

### 예시 4: Command Injection (A03)

**취약한 코드**:
```python
# 사용자 입력으로 시스템 명령 실행
import subprocess

def convert_file(filename):
    subprocess.call(f"convert {filename} output.pdf", shell=True)
```

**스캔 결과**:
```
[HIGH] Command Injection 취약점 발견
  파일: utils/converter.py:5
  패턴: subprocess with shell=True + 사용자 입력
  카테고리: A03 - Injection (Command)
```

**수정된 코드**:
```python
# shell=False 사용 + 리스트 형태 인자 전달
import subprocess
import shlex

def convert_file(filename):
    # 입력 검증 추가
    if not filename.isalnum() and '.' not in filename:
        raise ValueError("유효하지 않은 파일명")
    subprocess.call(["convert", filename, "output.pdf"], shell=False)
```

---

### 예시 5: Insecure Deserialization (A08)

**취약한 코드**:
```python
# pickle을 사용한 역직렬화
import pickle

def load_user_data(data):
    return pickle.loads(data)  # 위험: 임의 코드 실행 가능
```

**스캔 결과**:
```
[CRITICAL] 안전하지 않은 역직렬화 - pickle.loads() 사용
  파일: services/data_service.py:5
  패턴: pickle.loads()에 외부 데이터 전달
  카테고리: A08 - Software and Data Integrity Failures
```

**수정된 코드**:
```python
# JSON 등 안전한 직렬화 형식 사용
import json

def load_user_data(data):
    return json.loads(data)  # 안전: 코드 실행 불가
```

---

### 예시 6: Security Misconfiguration (A05)

**취약한 코드**:
```javascript
// CORS 전체 허용 + 디버그 모드
const app = express();
app.use(cors());  // 모든 오리진 허용
app.set('env', 'development');

app.use((err, req, res, next) => {
    res.status(500).json({
        error: err.message,
        stack: err.stack  // 스택 트레이스 노출
    });
});
```

**수정된 코드**:
```javascript
// 특정 오리진만 허용 + 프로덕션 에러 처리
const app = express();
app.use(cors({
    origin: ['https://example.com', 'https://app.example.com'],
    credentials: true
}));

app.use((err, req, res, next) => {
    console.error(err.stack);  // 서버 로그에만 기록
    res.status(500).json({
        error: process.env.NODE_ENV === 'production'
            ? '서버 오류가 발생했습니다'
            : err.message
    });
});
```

---

## 지원 언어

| 언어 | 파일 확장자 | 주요 탐지 항목 |
|------|-----------|---------------|
| JavaScript | `.js`, `.jsx`, `.mjs`, `.cjs` | eval, innerHTML, prototype pollution, XSS, 동적 코드 실행 |
| TypeScript | `.ts`, `.tsx` | JavaScript와 동일 + 타입 우회 패턴 |
| Python | `.py` | pickle, exec, eval, subprocess, yaml.load, os.system |
| Java | `.java` | Runtime.exec, SQL Statement, ObjectInputStream, XXE |
| SQL | `.sql` | 동적 쿼리, GRANT, xp_cmdshell, LOAD_FILE |
| YAML | `.yaml`, `.yml` | 안전하지 않은 역직렬화, 자격증명 노출 |
| JSON | `.json` | 자격증명 노출, 민감 설정 |
| XML | `.xml` | XXE, 자격증명 노출 |
| Shell | `.sh`, `.bash` | 명령어 인젝션, 하드코딩된 자격증명 |
| Go | `.go` | SQL 인젝션, 명령어 인젝션, 안전하지 않은 TLS |
| Ruby | `.rb` | eval, system, send, 역직렬화 |
| PHP | `.php` | eval, exec, SQL 인젝션, 파일 포함 |
| 설정 파일 | `.env`, `.ini`, `.conf`, `.cfg` | 자격증명 노출, 민감 설정 |
| 인증서/키 | `.pem`, `.key`, `.p12`, `.pfx` | Private key 노출 |

---

## 사용 방법

다음 명령어를 사용하여 보안 스캔을 실행할 수 있습니다:

- "보안 스캔 실행해줘"
- "이 코드의 취약점을 확인해줘"
- "커밋 전에 보안 검사 해줘"
- "시크릿이 코드에 포함되어 있는지 확인해줘"
- "의존성 보안 감사를 실행해줘"
- "OWASP 기준으로 코드를 점검해줘"

---

## 관련 파일

- [SKILL.md](SKILL.md) - 스킬 정의 및 상세 워크플로우
- [references/analysis-guide.md](references/analysis-guide.md) - 분석 방법론 및 탐지 패턴 상세
- [references/templates.md](references/templates.md) - 보고서 템플릿 및 수정 가이드
