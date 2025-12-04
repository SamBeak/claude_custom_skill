# AI Agent Rules Templates

Complete rule templates for different platforms and technology stacks.

## Table of Contents

1. [Cursor Rules (.cursorrules)](#cursor-rules-cursorrules)
2. [Windsurf Rules (.windsurfrules)](#windsurf-rules-windsurfrules)
3. [Cline/Claude Code Rules (.clinerules)](#clineclaude-code-rules-clinerules)
4. [Continue Rules (.continuerules)](#continue-rules-continuerules)
5. [Zed Settings (.zed/settings.json)](#zed-settings-zedsettingsjson)
6. [GitHub Copilot Instructions](#github-copilot-instructions)
7. [Antigravity/Kiro Rules](#antigravitykiro-rules)
8. [Universal AI Coding Guide](#universal-ai-coding-guide)
9. [Stack-Specific Templates](#stack-specific-templates)

---

## Cursor Rules (.cursorrules)

### Spring Boot + MyBatis (eGovFrame Style)

**File: `.cursorrules`**

```
You are an expert Java Spring Boot developer working on an eGovFrame-based project with MyBatis.

## Project Context

Technology Stack:
- Java 11
- Spring Boot 2.7 / Spring Framework 5.3
- eGovFrame 3.8 (Korean e-Government Framework)
- MyBatis 3.5
- JSP + jQuery
- MySQL/MariaDB

Architecture Pattern:
- Layered Architecture (MVC)
- Controller → Service → ServiceImpl → Mapper (MyBatis)

## Code Style Guidelines

### Indentation & Formatting
- Use TABS (not spaces) for indentation
- Tab width: 4 spaces equivalent
- Maximum line length: 120 characters
- K&R brace style (opening brace on same line)
- Always use braces for if/for/while, even single statements

### Naming Conventions

**Java:**
- Packages: lowercase (e.g., `egovframework.cona.auth.controller`)
- Classes: PascalCase (e.g., `AuthController`, `UserVO`)
- Methods: camelCase, verb-first (e.g., `selectUserInfo`, `updatePassword`)
- Variables: camelCase (e.g., `userVO`, `responseMap`)
- Constants: UPPER_SNAKE_CASE (e.g., `MAX_SIZE`, `SECRET_KEY`)

**Database:**
- Table names: lowercase snake_case with `tb_` prefix (e.g., `tb_user`, `tb_board`)
- Column names: UPPER_SNAKE_CASE (e.g., `USER_ID`, `REG_DATE`)

**Method Prefixes:**
- `select*`: Database queries (SELECT)
- `insert*`: Database inserts (INSERT)
- `update*`: Database updates (UPDATE)
- `delete*`: Database deletes (DELETE)
- `action*`: User actions (e.g., `actionLogin`)
- `search*`: Search operations
- `store*`: Save/store operations

### File Organization

```
src/main/java/egovframework/cona/
├── [domain]/
│   ├── controller/     # @Controller classes
│   ├── service/        # Service interfaces
│   │   └── impl/       # Service implementations
│   ├── mapper/         # MyBatis Mapper interfaces
│   └── model/          # VO (Value Object) classes
└── cmmn/              # Common utilities
```

## Architecture Guidelines

### Controller Layer
- Annotate with `@Controller`
- Use `@RequestMapping` for URL mapping (`.do` extension)
- Return type: `String` (view name) or `Map<String, Object>` (for AJAX)
- Always use `@ResponseBody` for AJAX responses
- Handle sessions with `HttpSession` or `@SessionAttribute`

**Example:**
```java
@Controller
@RequestMapping("/auth")
public class AuthController {
→
→	@Autowired
→	private AuthService authService;
→
→	@ResponseBody
→	@RequestMapping("/actionLogin.do")
→	public Map<String, Object> userLogin(UserVO userVO, HttpSession session) {
→	→	Map<String, Object> responseMap = new HashMap<>();
→	→	try {
→	→	→	UserVO loginVO = authService.actionLogin(userVO);
→	→	→	session.setAttribute("LoginVO", loginVO);
→	→	→	responseMap.put("result", true);
→	→	→	responseMap.put("returnUrl", "/main.do");
→	→	} catch (Exception e) {
→	→	→	responseMap.put("result", false);
→	→	→	responseMap.put("message", "로그인 실패");
→	→	→	e.printStackTrace();
→	→	}
→	→	return responseMap;
→	}
}
```

**AJAX Response Format:**
Always include:
- `result`: boolean (true/false)
- `message`: String (error or success message)
- `data`: Object (optional, response data)
- `returnUrl`: String (optional, redirect URL)

### Service Layer
- Create interface + implementation pattern
- Interface in `service/` package
- Implementation in `service/impl/` with `Impl` suffix
- Annotate implementation with `@Service`
- Business logic goes here, NOT in controllers
- Handle transactions with `@Transactional` or manual `TransactionManager`

**Example:**
```java
// Interface
public interface AuthService {
→	UserVO actionLogin(UserVO userVO) throws Exception;
}

// Implementation
@Service
public class AuthServiceImpl implements AuthService {
→
→	@Autowired
→	private AuthMapper authMapper;
→
→	@Override
→	public UserVO actionLogin(UserVO userVO) throws Exception {
→	→	// 1. 비밀번호 암호화
→	→	String encrypted = EgovFileScrty.encryptPassword(
→	→	→	userVO.getUserPassword(),
→	→	→	userVO.getUserId()
→	→	);
→	→	userVO.setUserPassword(encrypted);
→	→
→	→	// 2. 로그인 조회
→	→	UserVO loginVO = authMapper.actionLogin(userVO);
→	→
→	→	// 3. 결과 반환
→	→	return loginVO;
→	}
}
```

### Mapper Layer (MyBatis)
- Interface in `mapper/` package
- XML in `src/main/resources/egovframework/sqlmap/[domain]/mappers/`
- Use `@Mapper` annotation or XML configuration
- Method names match XML statement ids

**Java Interface:**
```java
@Mapper
public interface AuthMapper {
→	UserVO actionLogin(UserVO userVO) throws Exception;
→	void insertUser(UserVO userVO) throws Exception;
→	void updateUser(UserVO userVO) throws Exception;
}
```

**MyBatis XML:**
```xml
<mapper namespace="egovframework.cona.auth.mapper.AuthMapper">
→
→	<select id="actionLogin" resultType="UserVO">
→	→	SELECT
→	→	→	USER_NO,
→	→	→	USER_ID,
→	→	→	USER_NAME
→	→	FROM
→	→	→	tb_user
→	→	WHERE
→	→	→	USER_ID = #{userId}
→	→	AND
→	→	→	USER_PASSWORD = #{userPassword}
→	→	AND
→	→	→	DEL_YN = 'N'
→	</select>
→
</mapper>
```

**SQL Style:**
- Keywords: UPPERCASE
- Columns: UPPERCASE with proper indentation (3 tabs from SELECT)
- Tables: lowercase
- Use tabs for indentation in XML

### VO (Value Object) Classes
- POJOs in `model/` package
- Private fields with public getters/setters
- NO business logic
- Include common fields: `regUserId`, `regDate`, `regIp`, `updateUserId`, `updateDate`, `updateIp`, `delYn`

**Example:**
```java
public class UserVO {
→
→	private int userNo;
→	private String userId;
→	private String userName;
→	private String userEmail;
→
→	// Common audit fields
→	private String regUserId;
→	private String regDate;
→	private String regIp;
→	private String updateUserId;
→	private String updateDate;
→	private String updateIp;
→	private String delYn;
→
→	// Getters and setters...
→	public int getUserNo() { return userNo; }
→	public void setUserNo(int userNo) { this.userNo = userNo; }
→	// ... (continue for all fields)
}
```

## Transaction Management

### Programmatic Transactions
```java
@Autowired
private DataSourceTransactionManager txManager;

public void someMethod() {
→	DefaultTransactionDefinition def = new DefaultTransactionDefinition();
→	def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
→	TransactionStatus txStatus = txManager.getTransaction(def);
→
→	try {
→	→	// Business logic here
→	→	mapper.insert(data);
→	→	txManager.commit(txStatus);
→	} catch (Exception e) {
→	→	txManager.rollback(txStatus);
→	→	e.printStackTrace();
→	→	throw e;
→	}
}
```

### Declarative Transactions
```java
@Transactional
public void someMethod() throws Exception {
→	// Business logic here
→	// Auto-commit on success, rollback on exception
}
```

## Common Patterns

### Password Encryption
```java
String encrypted = EgovFileScrty.encryptPassword(password, userId);
```

### Getting Client IP
```java
String clientIp = CommonUtils.getClientIpAddress();
```

### Session Management
```java
// Set session
UserVO loginVO = ...;
session.setAttribute("LoginVO", loginVO);

// Get session
UserVO sessionVO = (UserVO) session.getAttribute("LoginVO");

// Check login
if (sessionVO == null || sessionVO.getUserId() == null) {
→	return "forward:/auth/loginPage.do";
}
```

### Sending Email
```java
String tempPassword = CommonUtils.createRandomPassword(7);
String emailContent = CommonUtils.generateEmailContent(tempPassword);
CommonUtils.sendEmail(email, subject, emailContent);
```

## Frontend (JSP + jQuery)

### JSP Structure
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html>
<head>
→	<meta charset="UTF-8">
→	<title>Page Title</title>
→	<link rel="stylesheet" href="<c:url value='/css/style.css'/>"/>
→	<script src="<c:url value='/js/jquery-3.6.0.min.js'/>"></script>
</head>
<body>
→	<!-- Content -->
</body>
</html>
```

### jQuery AJAX Pattern
```javascript
$.ajax({
→	type: 'POST',
→	url: '/auth/actionLogin.do',
→	data: formData,
→	dataType: 'json',
→	success: function(response) {
→	→	if (response.result) {
→	→	→	window.location.href = response.returnUrl;
→	→	} else {
→	→	→	alert(response.message);
→	→	}
→	},
→	error: function(xhr, status, error) {
→	→	console.error('Error:', error);
→	→	alert('요청 처리 중 오류가 발생했습니다.');
→	}
});
```

### JavaScript Style
- Use `var` (legacy ES5 code)
- Function declarations: `function name() {}`
- Tabs for indentation
- Double quotes for strings
- Required semicolons

## Error Handling

### Controller Exception Handling
```java
try {
→	// Logic
→	responseMap.put("result", true);
} catch (Exception e) {
→	responseMap.put("result", false);
→	responseMap.put("message", "처리 중 오류가 발생했습니다.");
→	e.printStackTrace();
}
return responseMap;
```

### Service Exception Handling
```java
@Override
public UserVO getUserInfo(UserVO userVO) throws Exception {
→	// Let exceptions propagate up
→	// Or wrap in custom exceptions if needed
→	return mapper.selectUserInfo(userVO);
}
```

## Security Guidelines

1. **Always encrypt passwords** using `EgovFileScrty.encryptPassword()`
2. **Validate user input** before database operations
3. **Use parameterized queries** (MyBatis #{} syntax)
4. **Check session** before accessing protected resources
5. **Log IP addresses** for audit trail

## Testing Patterns

When writing new features:

1. Create VO class first
2. Create Mapper interface + XML
3. Create Service interface + implementation
4. Create Controller
5. Test each layer independently

## What to AVOID

- ❌ DO NOT use raw SQL strings (use MyBatis XML)
- ❌ DO NOT put business logic in Controllers
- ❌ DO NOT store passwords in plain text
- ❌ DO NOT use `System.out.println()` (use proper logging)
- ❌ DO NOT skip transaction management for multi-step operations
- ❌ DO NOT use magic numbers/strings (use constants)
- ❌ DO NOT ignore exceptions (always handle or log them)

## Comments

### JavaDoc for public methods
```java
/**
 * @methodName    : actionLogin
 * @author        : DEVELOPER_NAME
 * @date          : 2024.XX.XX
 * @param userVO  : User login information
 * @설명          : 사용자 로그인 처리
 * @return        : UserVO with login result
 * @throws Exception
 */
public UserVO actionLogin(UserVO userVO) throws Exception {
→	// Implementation
}
```

### Inline comments for complex logic
```java
// 1. 비밀번호 암호화
String encrypted = encrypt(password);

// 2. 데이터베이스 조회
UserVO result = mapper.select(userVO);

// 3. 결과 반환
return result;
```

## When in Doubt

- Follow existing patterns in the codebase
- Check similar features for reference
- Maintain consistency with current code style
- Ask for clarification on business logic
```

### React + TypeScript + Spring Boot REST API

**File: `.cursorrules`**

```
You are an expert full-stack developer working on a React TypeScript frontend with Spring Boot REST API backend.

## Project Context

Frontend:
- React 18 + TypeScript 5
- Vite build tool
- React Router v6
- Axios for API calls
- Tailwind CSS / Material-UI

Backend:
- Java 17 + Spring Boot 3
- Spring Data JPA
- PostgreSQL
- RESTful API design

## Code Style

### TypeScript/React
- Use functional components with hooks
- TypeScript strict mode enabled
- Named exports preferred
- Props destructuring
- 2 spaces indentation

### Java
- 4 spaces indentation
- Builder pattern for DTOs
- Record classes for immutable data
- Lombok for boilerplate reduction

## Architecture

### Frontend Structure
```
src/
├── components/     # Reusable UI components
├── pages/          # Page components
├── hooks/          # Custom React hooks
├── services/       # API service layer
├── types/          # TypeScript types/interfaces
├── utils/          # Utility functions
└── stores/         # State management
```

### Backend Structure
```
src/main/java/com/project/
├── controller/     # REST controllers
├── service/        # Business logic
├── repository/     # JPA repositories
├── entity/         # JPA entities
├── dto/            # Data transfer objects
└── config/         # Spring configuration
```

## Frontend Patterns

### Component Pattern
```typescript
import React from 'react';

interface UserCardProps {
  user: User;
  onEdit: (id: string) => void;
}

export const UserCard: React.FC<UserCardProps> = ({ user, onEdit }) => {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <button onClick={() => onEdit(user.id)}>Edit</button>
    </div>
  );
};
```

### API Service Pattern
```typescript
import axios from 'axios';

const api = axios.create({
  baseURL: '/api/v1',
  headers: {
    'Content-Type': 'application/json'
  }
});

export const userService = {
  getAll: () => api.get<User[]>('/users'),
  getById: (id: string) => api.get<User>(`/users/${id}`),
  create: (data: CreateUserDto) => api.post<User>('/users', data),
  update: (id: string, data: UpdateUserDto) => api.put<User>(`/users/${id}`, data),
  delete: (id: string) => api.delete(`/users/${id}`)
};
```

## Backend Patterns

### REST Controller
```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<List<UserDto>> getAll() {
        return ResponseEntity.ok(userService.findAll());
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDto> getById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @PostMapping
    public ResponseEntity<UserDto> create(@Valid @RequestBody CreateUserDto dto) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(userService.create(dto));
    }
}
```

### Service Layer
```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;

    @Transactional(readOnly = true)
    public List<UserDto> findAll() {
        return userRepository.findAll().stream()
            .map(userMapper::toDto)
            .toList();
    }

    @Transactional
    public UserDto create(CreateUserDto dto) {
        User user = userMapper.toEntity(dto);
        User saved = userRepository.save(user);
        return userMapper.toDto(saved);
    }
}
```

## API Response Format

Success:
```json
{
  "data": { ... },
  "timestamp": "2024-01-01T00:00:00Z"
}
```

Error:
```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with id 123 not found",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

## Common Patterns

- Use React Query for server state
- Use Context API for global UI state
- Validate all user input (frontend + backend)
- Use DTOs for all API communication
- Never expose entities directly in APIs
- Use builder pattern for complex object construction
- Prefer composition over inheritance
```

---

## Windsurf Rules (.windsurfrules)

### Spring Boot Structure

**File: `.windsurfrules`**

```yaml
project:
  name: "CONA_CTC"
  type: "Spring Boot + eGovFrame"
  framework: "eGovFrame 3.8"
  language: "Java 11"

codeStyle:
  indentation:
    style: "tab"
    size: 4
  naming:
    classes: "PascalCase"
    methods: "camelCase"
    variables: "camelCase"
    constants: "UPPER_SNAKE_CASE"
    packages: "lowercase"
  braceStyle: "K&R"
  maxLineLength: 120
  quotes: "double"

architecture:
  pattern: "Layered (MVC)"
  layers:
    - name: "Controller"
      package: "controller"
      responsibility: "HTTP request handling, session management"
      annotations: ["@Controller", "@RequestMapping", "@ResponseBody"]
    - name: "Service"
      package: "service"
      responsibility: "Business logic, transaction management"
      pattern: "Interface + Implementation"
    - name: "Mapper"
      package: "mapper"
      responsibility: "Database access via MyBatis"
      annotations: ["@Mapper"]
    - name: "Model"
      package: "model"
      responsibility: "Data transfer objects (VOs)"

database:
  orm: "MyBatis 3.5"
  namingConvention:
    tables: "tb_prefix_snake_case_lower"
    columns: "UPPER_SNAKE_CASE"

patterns:
  controller:
    returnTypes:
      - "String (view name)"
      - "Map<String, Object> (AJAX response)"
    responseFormat:
      result: "boolean"
      message: "string"
      data: "optional object"
      returnUrl: "optional string"
    urlPattern: "*.do"

  service:
    pattern: "Interface + Impl"
    transactions:
      - "Programmatic (TransactionManager)"
      - "Declarative (@Transactional)"

  mapper:
    location: "src/main/resources/egovframework/sqlmap/[domain]/mappers/"
    methodPrefixes:
      - "select*: SELECT queries"
      - "insert*: INSERT queries"
      - "update*: UPDATE queries"
      - "delete*: DELETE queries"

security:
  passwordEncryption: "EgovFileScrty.encryptPassword()"
  sessionManagement: "HttpSession"
  ipLogging: true

constraints:
  - "Always use tabs for indentation"
  - "Never put business logic in controllers"
  - "Always handle exceptions with try-catch"
  - "Use parameterized queries (MyBatis #{})"
  - "Include audit fields in all VOs"

antiPatterns:
  - "Raw SQL strings"
  - "Plain text passwords"
  - "System.out.println for logging"
  - "Business logic in controllers"
  - "Exposing entities directly in responses"
```

---

## Cline/Claude Code Rules (.clinerules)

### Spring Boot + MyBatis Template

**File: `.clinerules`**

```markdown
# Cline/Claude Code Rules for CONA_CTC

## Project Context

Spring Boot 2.7 eGovFrame project with MyBatis ORM, JSP frontend, and jQuery.

## Technology Stack

- Java 11
- Spring Boot 2.7 + eGovFrame 3.8
- MyBatis 3.5 (XML-based SQL)
- JSP + JSTL + jQuery 3.6
- MySQL/MariaDB

## Code Style

### Formatting
- **Indentation**: TABS (not spaces), 4-character width
- **Braces**: K&R style (opening brace on same line)
- **Line length**: Maximum 120 characters
- **Quotes**: Double quotes for strings

### Naming
- **Packages**: lowercase.dot.separated
- **Classes**: PascalCase (AuthController, UserVO)
- **Methods**: camelCase, verb-first (selectUserInfo, updatePassword)
- **Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Database tables**: tb_snake_case (lowercase)
- **Database columns**: UPPER_SNAKE_CASE

## Architecture Pattern

Layered MVC: Controller → Service → ServiceImpl → Mapper

### File Structure
```
[domain]/
├── controller/     # @Controller classes
├── service/        # Service interfaces
│   └── impl/       # ServiceImpl classes
├── mapper/         # @Mapper interfaces
└── model/          # VO classes
```

## Key Patterns

### Controller Pattern
```java
@Controller
@RequestMapping("/auth")
public class AuthController {
	@Autowired
	private AuthService authService;

	@ResponseBody
	@RequestMapping("/actionLogin.do")
	public Map<String, Object> actionLogin(...) {
		Map<String, Object> responseMap = new HashMap<>();
		try {
			// Business logic
			responseMap.put("result", true);
		} catch (Exception e) {
			responseMap.put("result", false);
			responseMap.put("message", e.getMessage());
		}
		return responseMap;
	}
}
```

### Service Pattern
```java
// Interface
public interface AuthService {
	UserVO actionLogin(UserVO userVO) throws Exception;
}

// Implementation
@Service("authService")
public class AuthServiceImpl implements AuthService {
	@Autowired
	private AuthMapper authMapper;

	@Override
	public UserVO actionLogin(UserVO userVO) throws Exception {
		return authMapper.actionLogin(userVO);
	}
}
```

### Mapper Pattern
```java
@Mapper
public interface AuthMapper {
	UserVO actionLogin(UserVO userVO);
}
```

### MyBatis XML
```xml
<mapper namespace="egovframework.cona.auth.mapper.AuthMapper">
	<select id="actionLogin" resultType="UserVO">
		SELECT
			USER_ID,
			USER_NAME
		FROM
			tb_user
		WHERE
			USER_ID = #{userId}
	</select>
</mapper>
```

## Security Rules

1. **Always encrypt passwords**: Use `EgovFileScrty.encryptPassword(password, userId)`
2. **Use parameterized queries**: MyBatis `#{param}` (never `${param}`)
3. **Log IP addresses**: Use `CommonUtils.getClientIpAddress()` for audit fields
4. **Validate sessions**: Check session for protected endpoints

## Anti-Patterns (Avoid)

❌ Business logic in Controllers
❌ Plain text passwords
❌ SQL injection with ${param}
❌ System.out.println (use logger)
❌ Spaces for indentation (use tabs)
❌ Missing audit fields in VOs

## Common Utilities

- `CommonUtils.createRandomPassword(length)` - Generate random password
- `CommonUtils.sendEmail(to, subject, content)` - Send email
- `CommonUtils.getClientIpAddress()` - Get client IP
- `EgovFileScrty.encryptPassword(password, salt)` - Encrypt password
```

---

## Continue Rules (.continuerules)

### JSON Configuration Format

**File: `.continuerules`**

```json
{
  "name": "CONA_CTC Project Rules",
  "version": "1.0.0",
  "description": "Spring Boot eGovFrame project with MyBatis",
  "rules": {
    "codeStyle": {
      "indentation": "tab",
      "tabWidth": 4,
      "lineLength": 120,
      "braceStyle": "K&R",
      "quotes": "double"
    },
    "naming": {
      "packages": "lowercase.dot",
      "classes": "PascalCase",
      "methods": "camelCase",
      "variables": "camelCase",
      "constants": "UPPER_SNAKE_CASE",
      "methodPrefixes": {
        "select": "Database SELECT queries",
        "insert": "Database INSERT operations",
        "update": "Database UPDATE operations",
        "delete": "Database DELETE operations",
        "action": "User action handlers"
      }
    },
    "database": {
      "tables": "tb_snake_case",
      "columns": "UPPER_SNAKE_CASE"
    },
    "architecture": {
      "pattern": "Layered MVC",
      "layers": [
        "controller",
        "service",
        "service/impl",
        "mapper",
        "model"
      ],
      "flow": "Controller → Service → Mapper"
    },
    "patterns": {
      "controller": {
        "annotation": "@Controller",
        "requestMapping": "*.do URLs",
        "returnType": "Map<String, Object> for AJAX",
        "responseKeys": ["result", "message", "data", "returnUrl"]
      },
      "service": {
        "pattern": "Interface + Implementation",
        "interfaceLocation": "service/",
        "implementationLocation": "service/impl/",
        "implementationSuffix": "Impl"
      },
      "mapper": {
        "annotation": "@Mapper",
        "sqlLocation": "src/main/resources/egovframework/sqlmap/[domain]/mappers/",
        "parameterSyntax": "#{paramName}"
      },
      "vo": {
        "suffix": "VO",
        "auditFields": [
          "regUserId",
          "regDate",
          "regIp",
          "updateUserId",
          "updateDate",
          "updateIp",
          "delYn"
        ]
      }
    },
    "security": {
      "passwordEncryption": "EgovFileScrty.encryptPassword(password, userId)",
      "sqlInjectionPrevention": "Use #{param} not ${param}",
      "sessionKey": "LoginVO",
      "ipLogging": "CommonUtils.getClientIpAddress()"
    },
    "frontend": {
      "technology": "JSP + jQuery",
      "javaScriptStyle": "ES5",
      "keywords": ["var", "function"],
      "ajaxPattern": "$.ajax with result/message/data response"
    },
    "antiPatterns": [
      "Business logic in Controllers",
      "Plain text passwords",
      "SQL injection with ${param}",
      "System.out.println for logging",
      "Spaces for indentation",
      "Missing audit fields in VOs",
      "let/const in JavaScript (use var)",
      "Arrow functions (use function keyword)"
    ]
  }
}
```

---

## Zed Settings (.zed/settings.json)

### AI-Integrated Settings

**File: `.zed/settings.json`**

```json
{
  "assistant": {
    "default_model": {
      "provider": "anthropic",
      "model": "claude-sonnet-4"
    },
    "version": "2",
    "button": true,
    "custom_instructions": "You are an expert Java Spring Boot developer working on an eGovFrame-based project with MyBatis.\n\n## Code Style\n- Use TABS for indentation (not spaces)\n- K&R brace style\n- camelCase for methods and variables\n- PascalCase for classes\n- UPPER_SNAKE_CASE for constants\n\n## Architecture\n- Layered MVC: Controller → Service → Mapper\n- Controllers return Map<String, Object> for AJAX\n- Services use Interface + Implementation pattern\n- MyBatis for database access with XML-based SQL\n\n## Patterns\n- Method prefixes: select*, insert*, update*, delete*, action*\n- URL pattern: *.do endpoints\n- Password encryption: EgovFileScrty.encryptPassword()\n- Session management: Store as 'LoginVO'\n\n## Security\n- Always encrypt passwords\n- Use parameterized queries (#{param})\n- Log IP addresses for audit\n- Validate sessions for protected endpoints\n\n## Anti-Patterns to Avoid\n- Business logic in Controllers\n- Plain text passwords\n- SQL injection with ${param}\n- System.out.println (use logger)\n- Spaces for indentation",
    "code_generation_rules": {
      "indentation": "tab",
      "architecture": "MVC",
      "namingConvention": "camelCase",
      "framework": "Spring Boot + MyBatis",
      "orm": "MyBatis",
      "frontend": "JSP + jQuery"
    }
  },
  "languages": {
    "Java": {
      "tab_size": 4,
      "hard_tabs": true,
      "format_on_save": "on"
    },
    "JavaScript": {
      "tab_size": 4,
      "hard_tabs": true,
      "format_on_save": "on"
    },
    "JSP": {
      "tab_size": 2,
      "hard_tabs": true
    }
  }
}
```

---

## GitHub Copilot Instructions

### Markdown Instructions Format

**File: `.github/copilot-instructions.md`**

```markdown
# GitHub Copilot Instructions for CONA_CTC

## Project Context

This is a Spring Boot 2.7 eGovFrame project with MyBatis ORM, following Korean e-Government standards.

**Technology Stack:**
- Java 11
- Spring Boot 2.7 + eGovFrame 3.8
- MyBatis 3.5 (XML-based SQL mapping)
- JSP + JSTL + jQuery 3.6
- MySQL/MariaDB

**Architecture:** Layered MVC (Controller → Service → ServiceImpl → Mapper)

## Code Style Rules

### Formatting
- **ALWAYS use TABS** (not spaces) for indentation
- Tab width: 4 characters
- K&R brace style (opening brace on same line)
- Maximum line length: 120 characters
- Line endings: LF (\n)

### Naming Conventions
- **Java Classes**: PascalCase (e.g., `AuthController`, `UserVO`)
- **Methods**: camelCase with verb prefix (e.g., `selectUserInfo`, `updatePassword`)
- **Variables**: camelCase (e.g., `userName`, `responseMap`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_SIZE`)
- **DB Tables**: `tb_snake_case` (lowercase)
- **DB Columns**: `UPPER_SNAKE_CASE`

### Method Prefixes
- `select*`: SELECT queries
- `insert*`: INSERT operations
- `update*`: UPDATE operations
- `delete*`: DELETE operations
- `action*`: User action handlers (e.g., `actionLogin`)

## Architecture Patterns

### Controller Pattern
```java
@Controller
@RequestMapping("/auth")
public class AuthController {
	@Autowired
	private AuthService authService;

	@ResponseBody
	@RequestMapping("/actionLogin.do")
	public Map<String, Object> actionLogin(...) {
		Map<String, Object> responseMap = new HashMap<>();
		try {
			// Business logic via service
			responseMap.put("result", true);
		} catch (Exception e) {
			responseMap.put("result", false);
			responseMap.put("message", e.getMessage());
		}
		return responseMap;
	}
}
```

### Service Pattern
Use Interface + Implementation:
```java
// service/AuthService.java
public interface AuthService {
	UserVO actionLogin(UserVO userVO) throws Exception;
}

// service/impl/AuthServiceImpl.java
@Service("authService")
public class AuthServiceImpl implements AuthService {
	@Autowired
	private AuthMapper authMapper;

	@Override
	public UserVO actionLogin(UserVO userVO) throws Exception {
		// Encrypt password
		String encrypted = EgovFileScrty.encryptPassword(
			userVO.getUserPassword(),
			userVO.getUserId()
		);
		userVO.setUserPassword(encrypted);
		return authMapper.actionLogin(userVO);
	}
}
```

### Mapper Pattern
```java
@Mapper
public interface AuthMapper {
	UserVO actionLogin(UserVO userVO);
}
```

## Security Best Practices

1. **Password Encryption**: Always use `EgovFileScrty.encryptPassword(password, userId)`
2. **SQL Injection Prevention**: Use `#{param}` syntax (never `${param}`)
3. **IP Logging**: Use `CommonUtils.getClientIpAddress()` for audit fields
4. **Session Management**: Store user as `"LoginVO"` in session

## What to Avoid

❌ Business logic in Controllers (use Services)
❌ Plain text passwords
❌ SQL injection with `${param}`
❌ `System.out.println` for logging (use logger)
❌ Spaces for indentation (use tabs)
❌ Missing audit fields in VOs

## Common Utilities

- `CommonUtils.createRandomPassword(int length)` - Generate random password
- `CommonUtils.sendEmail(String to, String subject, String content)` - Send email
- `CommonUtils.getClientIpAddress()` - Get client IP address
- `EgovFileScrty.encryptPassword(String password, String salt)` - Encrypt password

## Standard VO Structure

All VOs must include these audit fields:
```java
private String regUserId;
private String regDate;
private String regIp;
private String updateUserId;
private String updateDate;
private String updateIp;
private String delYn;
```
```

---

## Antigravity/Kiro Rules

### Universal Markdown Format

**File: `AI_CODING_GUIDE.md` or `.aicoderules`**

```markdown
# AI Coding Rules for CONA_CTC Project

> Guidelines for AI code generation tools (Antigravity, Kiro, and other AI assistants)

## 📋 Project Overview

**Project**: CONA CTC (Clinical Trial Center Management System)
**Framework**: Spring Boot 2.7 + eGovFrame 3.8 (Korean e-Government Framework)
**Architecture**: Layered MVC (Controller → Service → Mapper)
**ORM**: MyBatis 3.5
**Frontend**: JSP + jQuery 3.6
**Database**: MySQL/MariaDB

## 🎨 Code Style

### Indentation & Formatting
- **Indentation**: TABS (not spaces) - 4 character width
- **Brace Style**: K&R (opening brace on same line)
- **Line Length**: Maximum 120 characters
- **Quotes**: Double quotes for strings
- **Line Endings**: LF (\n)

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Java Package | lowercase.dot | `egovframework.cona.auth` |
| Java Class | PascalCase | `AuthController`, `UserVO` |
| Java Method | camelCase (verb-first) | `selectUserInfo`, `updatePassword` |
| Java Variable | camelCase | `userName`, `responseMap` |
| Java Constant | UPPER_SNAKE_CASE | `MAX_SIZE`, `DEFAULT_PAGE` |
| DB Table | tb_snake_case | `tb_user`, `tb_board` |
| DB Column | UPPER_SNAKE_CASE | `USER_ID`, `REG_DATE` |
| File (Controller) | [Domain]Controller.java | `AuthController.java` |
| File (Service) | [Domain]Service.java | `AuthService.java` |
| File (ServiceImpl) | [Domain]ServiceImpl.java | `AuthServiceImpl.java` |
| File (Mapper) | [Domain]Mapper.java | `AuthMapper.java` |
| File (VO) | [Domain]VO.java | `UserVO.java` |

### Method Prefixes

| Prefix | Purpose | Example |
|--------|---------|---------|
| `select*` | Database SELECT | `selectUserInfo`, `selectBoardList` |
| `insert*` | Database INSERT | `insertUser`, `insertBoard` |
| `update*` | Database UPDATE | `updatePassword`, `updateUserInfo` |
| `delete*` | Database DELETE | `deleteUser`, `deleteBoard` |
| `action*` | User actions | `actionLogin`, `actionLogout` |
| `search*` | Search operations | `searchUsers`, `searchBoards` |
| `store*` | Save/persist | `storeReservation` |

## 🏗️ Architecture

### Package Structure
```
egovframework.cona/
├── [domain]/
│   ├── controller/     # @Controller classes
│   ├── service/        # Service interfaces
│   │   └── impl/       # ServiceImpl classes
│   ├── mapper/         # @Mapper interfaces (MyBatis)
│   └── model/          # VO (Value Object) classes
└── cmmn/              # Common utilities
```

### Layer Responsibilities

**Controller Layer** (`controller/` package):
- Handle HTTP requests/responses
- Request validation
- Session management
- Delegate to Service layer
- Return Map<String, Object> for AJAX

**Service Layer** (`service/` and `service/impl/` packages):
- Business logic
- Transaction management
- Call Mapper methods
- Data transformation

**Mapper Layer** (`mapper/` package):
- Database operations
- MyBatis interface with XML SQL

**Model Layer** (`model/` package):
- Data transfer (VOs)
- Always include audit fields

## 📦 Common Patterns

### Controller Pattern
```java
@Controller
@RequestMapping("/auth")
public class AuthController {

	@Autowired
	private AuthService authService;

	@ResponseBody
	@RequestMapping("/actionLogin.do")
	public Map<String, Object> actionLogin(
		@RequestParam("userId") String userId,
		@RequestParam("userPassword") String userPassword,
		HttpSession session
	) {
		Map<String, Object> responseMap = new HashMap<>();

		try {
			UserVO userVO = new UserVO();
			userVO.setUserId(userId);
			userVO.setUserPassword(userPassword);

			UserVO loginVO = authService.actionLogin(userVO);

			if (loginVO != null) {
				session.setAttribute("LoginVO", loginVO);
				responseMap.put("result", true);
				responseMap.put("returnUrl", "/main.do");
			} else {
				responseMap.put("result", false);
				responseMap.put("message", "Invalid credentials");
			}
		} catch (Exception e) {
			responseMap.put("result", false);
			responseMap.put("message", e.getMessage());
			e.printStackTrace();
		}

		return responseMap;
	}
}
```

### Service Pattern
```java
// Interface: service/AuthService.java
public interface AuthService {
	UserVO actionLogin(UserVO userVO) throws Exception;
	void updatePassword(UserVO userVO) throws Exception;
}

// Implementation: service/impl/AuthServiceImpl.java
@Service("authService")
public class AuthServiceImpl implements AuthService {

	@Autowired
	private AuthMapper authMapper;

	@Autowired
	private PlatformTransactionManager txManager;

	@Override
	public UserVO actionLogin(UserVO userVO) throws Exception {
		// Encrypt password
		String encrypted = EgovFileScrty.encryptPassword(
			userVO.getUserPassword(),
			userVO.getUserId()
		);
		userVO.setUserPassword(encrypted);

		return authMapper.actionLogin(userVO);
	}
}
```

### Mapper Pattern
```java
@Mapper
public interface AuthMapper {
	UserVO actionLogin(UserVO userVO);
	void insertUser(UserVO userVO);
	void updatePassword(UserVO userVO);
}
```

### MyBatis XML Pattern
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="egovframework.cona.auth.mapper.AuthMapper">

	<select id="actionLogin" resultType="UserVO">
		SELECT
			USER_NO,
			USER_ID,
			USER_NAME,
			USER_EMAIL
		FROM
			tb_user
		WHERE
			USER_ID = #{userId}
		AND
			USER_PASSWORD = #{userPassword}
		AND
			DEL_YN = 'N'
	</select>

</mapper>
```

## 🔒 Security Practices

1. **Password Encryption**
   ```java
   String encrypted = EgovFileScrty.encryptPassword(password, userId);
   ```

2. **SQL Injection Prevention**
   - Use `#{paramName}` (parameterized)
   - NEVER use `${paramName}` (string substitution)

3. **Session Management**
   ```java
   // Store
   session.setAttribute("LoginVO", userVO);

   // Check
   UserVO sessionVO = (UserVO) session.getAttribute("LoginVO");
   if (sessionVO == null) {
       return "forward:/auth/loginPage.do";
   }
   ```

4. **IP Logging**
   ```java
   String clientIp = CommonUtils.getClientIpAddress();
   userVO.setRegIp(clientIp);
   ```

## ❌ Anti-Patterns (What to Avoid)

1. ❌ Business logic in Controllers → ✅ Use Services
2. ❌ Plain text passwords → ✅ Encrypt with EgovFileScrty
3. ❌ `System.out.println` → ✅ Use proper logger
4. ❌ SQL in Java code → ✅ Use MyBatis XML
5. ❌ `${paramName}` in SQL → ✅ Use `#{paramName}`
6. ❌ Spaces for indentation → ✅ Use tabs
7. ❌ Missing audit fields → ✅ Include all audit fields in VOs
8. ❌ Ignoring exceptions → ✅ Handle with try-catch

## 🛠️ Common Utilities

```java
// Generate random password
String tempPassword = CommonUtils.createRandomPassword(7);

// Send email
CommonUtils.sendEmail(email, "Subject", htmlContent);

// Get client IP
String clientIp = CommonUtils.getClientIpAddress();

// Encrypt password
String encrypted = EgovFileScrty.encryptPassword(password, userId);
```

## 📝 Standard VO Structure

All VOs must include audit fields:
```java
public class UserVO {
	// Business fields
	private String userNo;
	private String userId;
	private String userName;

	// Audit fields (REQUIRED)
	private String regUserId;
	private String regDate;
	private String regIp;
	private String updateUserId;
	private String updateDate;
	private String updateIp;
	private String delYn;

	// Getters and setters...
}
```

## 🌐 Frontend Patterns (JSP + jQuery)

### JavaScript Style (ES5)
```javascript
// Use var (not let/const)
var userName = "test";

// Use function keyword (not arrow functions)
function submitForm() {
	var formData = {
		userId: $("#userId").val(),
		userPassword: $("#userPassword").val()
	};

	$.ajax({
		type: "POST",
		url: "/auth/actionLogin.do",
		data: formData,
		dataType: "json",
		success: function(response) {
			if (response.result) {
				window.location.href = response.returnUrl;
			} else {
				alert(response.message);
			}
		}
	});
}
```

## 🎯 Code Generation Guidelines

When generating code:
1. ✅ Match existing patterns in the codebase
2. ✅ Use TABS for indentation
3. ✅ Include audit fields in all VOs
4. ✅ Encrypt passwords before storage
5. ✅ Use parameterized queries (#{param})
6. ✅ Handle exceptions with try-catch
7. ✅ Return Map<String, Object> for AJAX
8. ✅ Follow method naming prefixes (select*, insert*, update*, delete*)
```

---

## Universal AI Coding Guide

### Comprehensive Markdown Format

**File: `AI_CODING_GUIDE.md`**

```markdown
# AI Coding Guidelines for CONA_CTC Project

This document provides comprehensive context and coding rules for AI assistants working on this project.

## 📚 Project Overview

**Project Name**: CONA CTC (Clinical Trial Center Management System)
**Framework**: Spring Boot 2.7 + eGovFrame 3.8 (Korean e-Government Framework)
**Architecture**: Layered MVC (Controller → Service → Mapper)
**ORM**: MyBatis 3.5
**Frontend**: JSP + jQuery
**Database**: MySQL/MariaDB

## 🏗️ Architecture

### Package Structure
```
egovframework.cona/
├── [domain]/
│   ├── controller/     @Controller classes
│   ├── service/        Service interfaces
│   │   └── impl/       Service implementations
│   ├── mapper/         MyBatis Mapper interfaces
│   └── model/          VO (Value Object) classes
└── cmmn/              Common utilities
```

### Layer Responsibilities

**Controller Layer:**
- Handle HTTP requests/responses
- Session management
- Input validation
- Delegate to Service layer
- Return views or JSON responses

**Service Layer:**
- Business logic
- Transaction management
- Coordinate multiple data sources
- Error handling

**Mapper Layer:**
- Database operations via MyBatis
- SQL queries in XML files
- Parameter mapping

**Model Layer:**
- Data transfer objects (VOs)
- No business logic
- Getters/setters only

## 📝 Code Style Rules

### Java Formatting
- **Indentation**: Tabs (4-space equivalent)
- **Braces**: K&R style (opening on same line)
- **Line length**: 120 characters max
- **Imports**: No wildcards, organized by package

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Package | lowercase | `egovframework.cona.auth.controller` |
| Class | PascalCase | `AuthController` |
| Interface | PascalCase | `AuthService` |
| Method | camelCase (verb-first) | `selectUserInfo()` |
| Variable | camelCase | `userVO` |
| Constant | UPPER_SNAKE | `MAX_SIZE` |
| Table | tb_snake_lower | `tb_user` |
| Column | UPPER_SNAKE | `USER_ID` |

### Method Naming Patterns
- `select*`: Database SELECT operations
- `insert*`: Database INSERT operations
- `update*`: Database UPDATE operations
- `delete*`: Database DELETE operations
- `action*`: User-triggered actions
- `search*`: Search operations
- `store*`: Save/persist operations

## 🎨 Design Patterns

### Controller Pattern
```java
@Controller
@RequestMapping("/auth")
public class AuthController {

    @Autowired
    private AuthService authService;

    // View return
    @RequestMapping("/loginPage.do")
    public String loginPage() {
        return "auth/login";
    }

    // AJAX return
    @ResponseBody
    @RequestMapping("/actionLogin.do")
    public Map<String, Object> actionLogin(UserVO userVO, HttpSession session) {
        Map<String, Object> responseMap = new HashMap<>();
        try {
            UserVO loginVO = authService.actionLogin(userVO);
            session.setAttribute("LoginVO", loginVO);
            responseMap.put("result", true);
            responseMap.put("returnUrl", "/main.do");
        } catch (Exception e) {
            responseMap.put("result", false);
            responseMap.put("message", "Login failed");
            e.printStackTrace();
        }
        return responseMap;
    }
}
```

### Service Pattern
```java
// Interface
public interface AuthService {
    UserVO actionLogin(UserVO userVO) throws Exception;
}

// Implementation
@Service
public class AuthServiceImpl implements AuthService {

    @Autowired
    private AuthMapper authMapper;

    @Override
    public UserVO actionLogin(UserVO userVO) throws Exception {
        // 1. Encrypt password
        String encrypted = EgovFileScrty.encryptPassword(
            userVO.getUserPassword(),
            userVO.getUserId()
        );
        userVO.setUserPassword(encrypted);

        // 2. Query database
        UserVO loginVO = authMapper.actionLogin(userVO);

        // 3. Return result
        return loginVO;
    }
}
```

### MyBatis Mapper Pattern
```java
@Mapper
public interface AuthMapper {
    UserVO actionLogin(UserVO userVO) throws Exception;
    void insertUser(UserVO userVO) throws Exception;
    void updateUser(UserVO userVO) throws Exception;
}
```

**XML Configuration:**
```xml
<mapper namespace="egovframework.cona.auth.mapper.AuthMapper">
    <select id="actionLogin" resultType="UserVO">
        SELECT
            USER_NO,
            USER_ID,
            USER_NAME,
            USER_EMAIL
        FROM
            tb_user
        WHERE
            USER_ID = #{userId}
        AND
            USER_PASSWORD = #{userPassword}
        AND
            DEL_YN = 'N'
    </select>
</mapper>
```

### VO Pattern
```java
public class UserVO {

    // Business fields
    private int userNo;
    private String userId;
    private String userName;
    private String userEmail;

    // Audit fields (ALWAYS include)
    private String regUserId;
    private String regDate;
    private String regIp;
    private String updateUserId;
    private String updateDate;
    private String updateIp;
    private String delYn;

    // Getters and setters
    public int getUserNo() { return userNo; }
    public void setUserNo(int userNo) { this.userNo = userNo; }
    // ... (all fields)
}
```

## 🔄 Common Workflows

### Creating a New Feature

1. **Define VO** in `model/` package
2. **Create Mapper** interface in `mapper/`
3. **Create MyBatis XML** in `resources/egovframework/sqlmap/`
4. **Create Service** interface in `service/`
5. **Implement Service** in `service/impl/`
6. **Create Controller** in `controller/`
7. **Test each layer** independently

### AJAX Response Pattern
Always return consistent structure:
```java
Map<String, Object> response = new HashMap<>();
response.put("result", true);        // boolean: success/failure
response.put("message", "Success");  // string: user message
response.put("data", resultObject);  // optional: response data
response.put("returnUrl", "/next");  // optional: redirect URL
```

### Transaction Management

**Programmatic:**
```java
@Autowired
private DataSourceTransactionManager txManager;

public void someMethod() {
    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    TransactionStatus status = txManager.getTransaction(def);

    try {
        // Business logic
        mapper.insert(data);
        txManager.commit(status);
    } catch (Exception e) {
        txManager.rollback(status);
        throw e;
    }
}
```

**Declarative:**
```java
@Transactional
public void someMethod() throws Exception {
    // Business logic
    // Auto-commit on success, rollback on exception
}
```

## 🔒 Security Practices

1. **Password Encryption**
   ```java
   String encrypted = EgovFileScrty.encryptPassword(password, userId);
   ```

2. **Session Validation**
   ```java
   UserVO sessionVO = (UserVO) session.getAttribute("LoginVO");
   if (sessionVO == null || sessionVO.getUserId() == null) {
       return "forward:/auth/loginPage.do";
   }
   ```

3. **IP Logging**
   ```java
   String clientIp = CommonUtils.getClientIpAddress();
   userVO.setRegIp(clientIp);
   ```

4. **Input Validation**
   - Validate on frontend (JavaScript)
   - Validate on backend (Java)
   - Use parameterized queries (MyBatis #{})

## 🚫 Anti-Patterns (Avoid These)

❌ **DON'T:**
- Put business logic in controllers
- Use plain text passwords
- Use `System.out.println()` for logging
- Use raw SQL strings
- Expose entities directly in responses
- Skip transaction management
- Ignore exceptions
- Use magic numbers/strings

✅ **DO:**
- Keep controllers thin
- Encrypt all passwords
- Use proper logging framework
- Use MyBatis XML for queries
- Use DTOs/VOs for data transfer
- Manage transactions explicitly
- Handle all exceptions
- Use constants for fixed values

## 🧪 Testing Guidelines

- Write unit tests for Service layer
- Integration tests for Mapper layer
- Manual testing for Controller endpoints
- Test transaction rollback scenarios
- Validate all error paths

## 📚 Common Utilities

```java
// Password encryption
EgovFileScrty.encryptPassword(password, salt)

// Random password generation
CommonUtils.createRandomPassword(7)

// Email sending
CommonUtils.sendEmail(to, subject, htmlContent)

// Client IP retrieval
CommonUtils.getClientIpAddress()

// Empty check
CommonUtils.isEmpty(object)
```

## 💡 Best Practices

1. **Consistency**: Follow existing patterns in codebase
2. **Documentation**: Add JavaDoc for public methods
3. **Error Messages**: Use Korean for user-facing messages
4. **Logging**: Log exceptions with stack traces
5. **Code Review**: Check for common mistakes before committing

## 📖 Reference Examples

See existing implementations:
- `AuthController.java` - Login/logout patterns
- `BoardController.java` - File upload/download patterns
- `ReservationController.java` - Complex business logic
- `AuthService.java` - Service layer patterns

---

**Last Updated**: 2024-12-04
**Maintained By**: Development Team
```

---

## Stack-Specific Templates

### Next.js + TypeScript

```
You are working on a Next.js 14 project with TypeScript and App Router.

## Stack
- Next.js 14 (App Router)
- TypeScript 5
- Tailwind CSS
- shadcn/ui components
- React Server Components

## Patterns

### Page Component (Server Component)
```typescript
// app/users/page.tsx
import { UserList } from '@/components/users/user-list';
import { getUsers } from '@/lib/api/users';

export default async function UsersPage() {
  const users = await getUsers();
  return <UserList users={users} />;
}
```

### Client Component
```typescript
'use client';

import { useState } from 'react';

export function UserForm() {
  const [name, setName] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    // Handle submission
  };

  return <form onSubmit={handleSubmit}>...</form>;
}
```

### Server Actions
```typescript
'use server';

import { revalidatePath } from 'next/cache';

export async function createUser(formData: FormData) {
  const name = formData.get('name');
  // Save to database
  revalidatePath('/users');
}
```

## Routing
- Use App Router (`app/` directory)
- Server Components by default
- Add 'use client' only when needed
- Dynamic routes: `[id]/page.tsx`
- Route groups: `(dashboard)/layout.tsx`

## State Management
- Server state: React Server Components
- Client state: useState/useReducer
- Global state: Context API or Zustand
- Form state: react-hook-form

## Data Fetching
- Server Components: Direct database access
- Client Components: SWR or React Query
- Use server actions for mutations

## Styling
- Tailwind CSS utility classes
- shadcn/ui for components
- CSS Modules for complex styles
```

### Django + DRF

```
You are working on a Django REST Framework project.

## Stack
- Python 3.11
- Django 4.2
- Django REST Framework
- PostgreSQL
- Celery for async tasks

## Patterns

### Model
```python
from django.db import models

class User(models.Model):
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        db_table = 'users'
        ordering = ['-created_at']
```

### Serializer
```python
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'email', 'name', 'created_at']
        read_only_fields = ['id', 'created_at']
```

### ViewSet
```python
from rest_framework import viewsets
from rest_framework.permissions import IsAuthenticated

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        serializer.save(created_by=self.request.user)
```

## Code Style
- PEP 8 compliant
- 4 spaces indentation
- snake_case for functions/variables
- PascalCase for classes
- UPPER_SNAKE_CASE for constants

## Project Structure
```
project/
├── apps/
│   ├── users/
│   │   ├── models.py
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── urls.py
│   └── ...
├── config/
│   ├── settings/
│   └── urls.py
└── manage.py
```
```

