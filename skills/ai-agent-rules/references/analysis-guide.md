# Codebase Analysis Guide for AI Agent Rules

Comprehensive methodology for analyzing codebases to extract AI agent rules.

## Analysis Workflow

### Phase 1: Project Discovery (15-20 minutes)

#### Step 1: Identify Technology Stack

**Check these files first:**
- `pom.xml` / `build.gradle` (Java/Spring)
- `package.json` (JavaScript/TypeScript)
- `requirements.txt` / `pyproject.toml` (Python)
- `go.mod` (Go)
- `.csproj` (C#/.NET)

**Extract:**
- Framework versions (Spring Boot 2.7, React 18, etc.)
- Key dependencies (MyBatis, JPA, Redux, etc.)
- Build tools (Maven, Gradle, npm, pip)
- Language versions (Java 11, Node 18, Python 3.11)

**Example for Spring Boot:**
```xml
<!-- pom.xml analysis -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.0</version>  <!-- ✓ Spring Boot version -->
</parent>

<dependencies>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>  <!-- ✓ MyBatis ORM -->
        ...
    </dependency>
</dependencies>
```

**Document:**
```markdown
## Technology Stack
- Java 11
- Spring Boot 2.7
- MyBatis 3.5
- JSP + jQuery
- MySQL
```

#### Step 2: Understand Project Structure

**Map directory structure:**
```bash
# Run these commands
find src/main/java -type d -maxdepth 4
find src/main/resources -type d -maxdepth 3
```

**Look for patterns:**
- MVC structure: `controller/`, `service/`, `repository/`
- Layered: `presentation/`, `business/`, `data/`
- Clean Architecture: `entities/`, `usecases/`, `adapters/`
- Domain-Driven: `domain/`, `application/`, `infrastructure/`

**Example analysis:**
```
src/main/java/egovframework/cona/
├── auth/
│   ├── controller/      ✓ Controllers here
│   ├── service/
│   │   └── impl/        ✓ Service implementation pattern
│   ├── mapper/          ✓ MyBatis mappers
│   └── model/           ✓ VOs/DTOs here
└── cmmn/               ✓ Common utilities

→ Architecture: Layered MVC with Interface+Impl pattern
```

#### Step 3: Identify Architecture Pattern

**Read 2-3 controller files:**
- Check annotations (`@Controller`, `@RestController`)
- Look at method signatures
- Check return types
- Identify dependency injection pattern

**Read 2-3 service files:**
- Interface vs Implementation
- Transaction management approach
- Business logic patterns

**Example findings:**
```java
// Controller pattern discovered
@Controller  // ← Not @RestController
@RequestMapping("/auth")
public class AuthController {

    @Autowired  // ← Field injection
    private AuthService authService;

    @ResponseBody  // ← AJAX responses
    @RequestMapping("/actionLogin.do")  // ← .do URLs
    public Map<String, Object> login(...) {  // ← Map responses
```

**Document:**
```markdown
## Architecture Pattern
- Layered MVC
- Controller → Service → Mapper (MyBatis)
- Interface + Implementation for Services
- Field injection with @Autowired
- .do URL pattern (legacy Spring MVC)
- Map<String, Object> for AJAX responses
```

---

### Phase 2: Code Pattern Analysis (20-30 minutes)

#### Step 4: Analyze Naming Conventions

**Sample 10-15 files across different layers:**

**Controllers:**
```java
AuthController.java
BoardController.java
ReservationController.java

→ Pattern: [Domain]Controller
→ Methods: actionLogin, selectUserInfo, updatePassword
→ Prefix patterns: action*, select*, insert*, update*, delete*
```

**Services:**
```java
AuthService.java (interface)
AuthServiceImpl.java (implementation)

→ Pattern: [Domain]Service + [Domain]ServiceImpl
→ Suffix: Impl for implementations
```

**Mappers/Repositories:**
```java
AuthMapper.java
BoardMapper.java

→ Pattern: [Domain]Mapper
→ Annotation: @Mapper (MyBatis)
```

**Models/VOs/DTOs:**
```java
UserVO.java
BoardVO.java
ReservationVO.java

→ Pattern: [Domain]VO
→ Suffix: VO (Value Object)
```

**Database:**
```sql
-- Table names
tb_user
tb_board
tb_reservation

-- Columns
USER_ID
USER_NAME
REG_DATE

→ Tables: tb_[domain]_snake_case (lowercase)
→ Columns: UPPER_SNAKE_CASE
```

**Document:**
```markdown
## Naming Conventions

### Java
- Classes: PascalCase
- Methods: camelCase (verb-first)
- Variables: camelCase
- Constants: UPPER_SNAKE_CASE
- Packages: lowercase.dot.separated

### Method Prefixes
- select*: Database SELECT
- insert*: Database INSERT
- update*: Database UPDATE
- delete*: Database DELETE
- action*: User actions (actionLogin)
- search*: Search operations
- store*: Save/persist operations

### Database
- Tables: tb_domain_snake (lowercase)
- Columns: UPPER_SNAKE_CASE
```

#### Step 5: Extract Common Patterns

**Look for repeated code structures:**

**1. Controller Response Pattern:**
```java
// Found in: AuthController, BoardController, ReservationController
Map<String, Object> responseMap = new HashMap<>();
try {
    // logic
    responseMap.put("result", true);
    responseMap.put("returnUrl", "/main.do");
} catch (Exception e) {
    responseMap.put("result", false);
    responseMap.put("message", "Error message");
    e.printStackTrace();
}
return responseMap;

→ Standard AJAX response pattern
```

**2. Transaction Pattern:**
```java
// Found in: Multiple service implementations
DefaultTransactionDefinition def = new DefaultTransactionDefinition();
def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
TransactionStatus txStatus = txManager.getTransaction(def);

try {
    // business logic
    txManager.commit(txStatus);
} catch (Exception e) {
    txManager.rollback(txStatus);
    throw e;
}

→ Programmatic transaction management
```

**3. Session Management:**
```java
// Found in: Multiple controllers
UserVO sessionVO = (UserVO) session.getAttribute("LoginVO");
if (sessionVO == null || sessionVO.getUserId() == null) {
    return "forward:/auth/loginPage.do";
}

→ Session validation pattern
```

**4. VO Structure:**
```java
// Found in: All VO classes
private String regUserId;
private String regDate;
private String regIp;
private String updateUserId;
private String updateDate;
private String updateIp;
private String delYn;

→ Standard audit fields in all VOs
```

**Document these as "Common Patterns" section**

#### Step 6: Analyze Database Access Layer

**Check ORM/Data access:**
- JPA entities vs MyBatis mappers
- Query location (annotations vs XML)
- SQL style and formatting

**MyBatis Example:**
```java
// Interface
@Mapper
public interface AuthMapper {
    UserVO actionLogin(UserVO userVO);
}
```

```xml
<!-- XML file location pattern -->
src/main/resources/egovframework/sqlmap/[domain]/mappers/Auth_SQL.xml

<!-- SQL formatting pattern -->
<select id="actionLogin" resultType="UserVO">
    SELECT
        USER_NO,
        USER_ID,
        USER_NAME
    FROM
        tb_user
    WHERE
        USER_ID = #{userId}
    AND
        USER_PASSWORD = #{userPassword}
</select>

→ Keywords: UPPERCASE
→ Columns: Indented 3 tabs from SELECT
→ Tables: lowercase
→ Tabs for indentation
```

**Document:**
```markdown
## Database Access

### ORM: MyBatis 3.5

### Mapper Pattern
- Interface: `@Mapper` annotated
- XML Location: `src/main/resources/egovframework/sqlmap/[domain]/mappers/`
- Method name = XML statement id

### SQL Style
- Keywords: UPPERCASE (SELECT, FROM, WHERE)
- Columns: UPPERCASE_SNAKE_CASE, indented
- Tables: lowercase_snake_case
- Indentation: Tabs
- Parameters: #{paramName} syntax
```

#### Step 7: Identify Security Patterns

**Search for:**
- Password handling
- Authentication/authorization
- Session management
- Input validation

**Example findings:**
```java
// Password encryption
String encrypted = EgovFileScrty.encryptPassword(password, userId);

// IP logging
String clientIp = CommonUtils.getClientIpAddress();
userVO.setRegIp(clientIp);

// Session check
UserVO sessionVO = (UserVO) session.getAttribute("LoginVO");
if (sessionVO == null) {
    // redirect to login
}
```

**Document:**
```markdown
## Security Practices

1. **Password Encryption**
   - Always use `EgovFileScrty.encryptPassword(password, salt)`
   - Salt is typically the userId

2. **IP Logging**
   - Log IP for all registration/update operations
   - Use `CommonUtils.getClientIpAddress()`

3. **Session Management**
   - Store user in session as "LoginVO"
   - Check session before protected operations

4. **Input Validation**
   - Validate in controller layer
   - Use parameterized queries (MyBatis #{})
```

---

### Phase 3: Frontend Analysis (10-15 minutes)

#### Step 8: Frontend Stack & Patterns

**Identify frontend technology:**
- React/Vue/Angular components
- JSP files
- Template engines (Thymeleaf, Handlebars)

**For JSP + jQuery example:**
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<script src="<c:url value='/js/jquery-3.6.0.min.js' />"></script>

→ JSP with JSTL
→ jQuery for interactions
```

**Analyze JavaScript patterns:**
```javascript
// Common AJAX pattern
$.ajax({
    type: 'POST',
    url: '/auth/actionLogin.do',
    data: formData,
    dataType: 'json',
    success: function(response) {
        if (response.result) {
            window.location.href = response.returnUrl;
        } else {
            alert(response.message);
        }
    }
});

→ jQuery $.ajax
→ Expecting JSON response with result/message/returnUrl
→ var keyword (ES5)
→ function keyword (not arrow functions)
```

**Document:**
```markdown
## Frontend

### Stack
- JSP + JSTL
- jQuery 3.6
- Legacy ES5 JavaScript

### Patterns
- Use `var` (not let/const)
- Function declarations (not arrow functions)
- jQuery for DOM manipulation and AJAX
- Response handling: check `response.result` boolean

### AJAX Pattern
```javascript
$.ajax({
    type: 'POST',
    url: '/controller/action.do',
    data: formData,
    dataType: 'json',
    success: function(response) {
        if (response.result) {
            // success handling
        } else {
            alert(response.message);
        }
    },
    error: function(xhr, status, error) {
        console.error('Error:', error);
    }
});
```
```

---

### Phase 4: Documentation (15-20 minutes)

#### Step 9: Identify Anti-Patterns to Avoid

**Look for:**
- Common mistakes in code
- Commented-out sections with explanations
- TODO/FIXME comments
- Code reviews or PR comments (if accessible)

**Example anti-patterns found:**
```java
// DON'T: Business logic in controller
@RequestMapping("/bad.do")
public Map<String, Object> badExample(...) {
    // Complex calculation here ← Should be in service
    int result = complexCalculation();
    return map;
}

// DON'T: Plain text passwords
userVO.setPassword("password123"); ← Should be encrypted

// DON'T: System.out for logging
System.out.println("User logged in"); ← Use proper logger
```

**Document:**
```markdown
## Anti-Patterns (Avoid These)

❌ **DON'T:**
1. Put business logic in controllers
2. Store plain text passwords
3. Use System.out.println() for logging
4. Use raw SQL strings (use MyBatis XML)
5. Expose entities directly in API responses
6. Skip transaction management
7. Ignore exceptions
8. Use magic numbers/strings

✅ **DO:**
1. Keep controllers thin (delegate to services)
2. Encrypt all passwords
3. Use proper logging framework
4. Use MyBatis for database access
5. Use DTOs/VOs for data transfer
6. Manage transactions explicitly
7. Handle all exceptions with try-catch
8. Use constants for fixed values
```

#### Step 10: Extract Common Utilities

**Search for utility classes:**
```bash
find . -name "*Util*.java" -o -name "*Helper*.java" -o -name "Common*.java"
```

**Example findings:**
```java
// CommonUtils.java
public class CommonUtils {
    public static String createRandomPassword(int length)
    public static void sendEmail(String to, String subject, String content)
    public static String getClientIpAddress()
    public static boolean isEmpty(Object obj)
}

// EgovFileScrty.java
public class EgovFileScrty {
    public static String encryptPassword(String password, String salt)
}
```

**Document:**
```markdown
## Common Utilities

### CommonUtils
- `createRandomPassword(length)`: Generate random password
- `sendEmail(to, subject, content)`: Send email
- `getClientIpAddress()`: Get client IP from request
- `isEmpty(obj)`: Check if object is null/empty

### EgovFileScrty
- `encryptPassword(password, salt)`: Encrypt password with salt

### Usage Examples
```java
// Generate temp password
String tempPassword = CommonUtils.createRandomPassword(7);

// Send email
CommonUtils.sendEmail(email, "Subject", htmlContent);

// Get client IP
String ip = CommonUtils.getClientIpAddress();

// Encrypt password
String encrypted = EgovFileScrty.encryptPassword(password, userId);
```
```

---

## Analysis Checklist

Use this checklist to ensure comprehensive analysis:

### Project Discovery
- [ ] Technology stack identified (framework, versions)
- [ ] Build tool and dependencies documented
- [ ] Project structure mapped
- [ ] Architecture pattern identified

### Code Patterns
- [ ] Naming conventions extracted
- [ ] Method naming patterns documented
- [ ] Common code patterns identified
- [ ] Database naming conventions analyzed

### Layer Analysis
- [ ] Controller patterns documented
- [ ] Service layer patterns identified
- [ ] Data access patterns extracted
- [ ] Model/VO structure analyzed

### Security & Best Practices
- [ ] Authentication/authorization patterns
- [ ] Password handling approach
- [ ] Session management pattern
- [ ] Input validation approach

### Frontend (if applicable)
- [ ] Frontend stack identified
- [ ] JavaScript patterns analyzed
- [ ] AJAX communication pattern
- [ ] UI framework/library identified

### Documentation
- [ ] Common utilities documented
- [ ] Anti-patterns identified
- [ ] Example code extracted
- [ ] Constraints and requirements noted

---

## Quick Analysis Commands

### Java/Spring Projects
```bash
# Find all controllers
find . -name "*Controller.java" | head -5

# Find all services
find . -name "*Service*.java" | grep -v "Test"

# Check Spring Boot version
grep -A5 "spring-boot-starter-parent" pom.xml

# Find MyBatis mappers
find . -name "*Mapper.java"

# Check database access
find . -path "*/resources/*" -name "*.xml" | grep -i sql
```

### JavaScript/TypeScript Projects
```bash
# Check framework
cat package.json | grep -E "(react|vue|angular|next)"

# Find component files
find src -name "*.jsx" -o -name "*.tsx" | head -10

# Check build tool
ls -la | grep -E "(webpack|vite|parcel)"

# Find API calls
grep -r "axios\|fetch" src/ --include="*.js" --include="*.ts"
```

### Python Projects
```bash
# Check framework
cat requirements.txt | grep -E "(django|flask|fastapi)"

# Find models
find . -name "models.py"

# Find views/routes
find . -name "views.py" -o -name "routes.py"

# Check async usage
grep -r "async def" . --include="*.py"
```

---

## Output Template

After analysis, generate rules using this structure:

```markdown
# AI Coding Rules for [Project Name]

## Project Overview
[Technology stack, architecture, key frameworks]

## Code Style
[Formatting, naming, indentation rules]

## Architecture Guidelines
[Layer structure, patterns, responsibilities]

## Common Patterns
[Controller pattern, service pattern, etc. with examples]

## Database Access
[ORM/query patterns, naming conventions]

## Security Practices
[Authentication, authorization, encryption]

## Frontend Patterns
[If applicable: JS/framework patterns]

## Common Utilities
[Available utility functions]

## Anti-Patterns
[What to avoid with explanations]

## Examples
[Real code examples from the project]

## Constraints
[Technology limitations, performance requirements]
```

---

## Platform-Specific Adjustments

### For Cursor (.cursorrules)
- Use natural, conversational language
- Include "You are..." statement at top
- Provide clear examples with code
- Explain WHY, not just WHAT

### For Windsurf (.windsurfrules)
- Use structured YAML/JSON format
- Organize by categories
- Include explicit rule definitions
- Provide enumerated options

### For Universal (Markdown)
- Comprehensive documentation
- Include table of contents
- Use tables for quick reference
- Provide extensive examples

---

## Tips for Better Analysis

1. **Start broad, then narrow**: Understand overall architecture before diving into details

2. **Look for consistency**: If pattern appears 3+ times, it's a standard

3. **Check recent files**: Newer files show current standards better than old code

4. **Read tests**: Test files often show intended usage patterns

5. **Check configuration**: Config files reveal framework choices and conventions

6. **Follow imports**: See what utilities/libraries are commonly used

7. **Look for comments**: Inline comments often explain non-obvious patterns

8. **Check git history**: Recent commits show active development patterns

9. **Identify edge cases**: Find how errors and special cases are handled

10. **Validate findings**: Cross-reference patterns across multiple files
