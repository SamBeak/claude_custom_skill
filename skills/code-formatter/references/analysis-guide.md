# Codebase Analysis Guide

Comprehensive methodology for analyzing existing codebases to extract formatting rules.

## Analysis Workflow

### Step 1: File Selection

Choose representative files:
- **5-10 files** from different modules
- Mix of large and small files
- Include main application code (not generated/vendor code)
- Prioritize recently modified files

### Step 2: Pattern Analysis

Analyze these key patterns:

#### 1. Indentation

**What to check:**
```
Look at nested blocks:
- Are they using tabs or spaces?
- How many spaces/tabs per level?
- Is it consistent across files?
```

**Detection method:**
```javascript
// Count leading whitespace
if (line.startsWith('\t')) → Tabs
if (line.match(/^    /)) → 4 spaces
if (line.match(/^  /)) → 2 spaces
```

**Common patterns:**
- Java: Tabs or 4 spaces
- JavaScript: Tabs, 2 spaces, or 4 spaces
- Python: 4 spaces (PEP 8)
- HTML/CSS: 2 spaces or tabs

#### 2. Quote Style

**What to check:**
```javascript
// JavaScript/TypeScript
var x = "double";  // Double quotes
var y = 'single';  // Single quotes
var z = `template`; // Template literals

// Java
String s = "double only";

// Python
x = "double" or 'single'
```

**Count occurrences:**
- Count single vs double quote usage
- Exclude strings containing quotes (e.g., "it's" needs single outer)
- Template literals don't count for style preference

#### 3. Semicolons (JavaScript/TypeScript)

**What to check:**
```javascript
// Required style
var x = 1;
return value;

// Optional style (ASI)
var x = 1
return value
```

**Detection:**
- Check statement-ending semicolons
- Ignore semicolons in for loops: `for (i=0; i<10; i++)`

#### 4. Brace Style

**K&R (same line):**
```javascript
function foo() {
    if (condition) {
        // code
    } else {
        // code
    }
}
```

**Allman (next line):**
```javascript
function foo()
{
    if (condition)
    {
        // code
    }
    else
    {
        // code
    }
}
```

**Detection:**
- Count opening braces on same line vs next line
- Check else/catch placement

#### 5. Spacing Rules

**Operators:**
```javascript
// Spaces around operators
var sum = a + b;
var result = x === y;

// No spaces
var sum=a+b;  // (rare)
```

**Keywords:**
```javascript
// Space after keywords
if (condition)
for (var i = 0)
while (true)

// No space
if(condition)  // (less common)
```

**Function calls:**
```javascript
// No space before parentheses
function foo()
method(arg)

// Space before parentheses (rare)
function foo ()
method (arg)
```

**Object literals:**
```javascript
// Space after colon
{name: "value", age: 30}

// No space
{name:"value", age:30}  // (rare)
```

#### 6. Naming Conventions

**JavaScript/TypeScript:**
```javascript
// Variables/functions: camelCase
var userName = "test";
function getUserInfo() {}

// Classes: PascalCase
class UserAccount {}

// Constants: UPPER_SNAKE_CASE or camelCase
const MAX_COUNT = 100;
const maxCount = 100;
```

**Java:**
```java
// Packages: lowercase
package com.example.app;

// Classes: PascalCase
public class UserService {}

// Methods/variables: camelCase
public void getUserInfo() {}
private String userName;

// Constants: UPPER_SNAKE_CASE
public static final int MAX_SIZE = 100;
```

**Python:**
```python
# Variables/functions: snake_case
user_name = "test"
def get_user_info():

# Classes: PascalCase
class UserAccount:

# Constants: UPPER_SNAKE_CASE
MAX_COUNT = 100
```

**Database (SQL):**
```sql
-- Table names: check for pattern
tb_user, user, USER, users

-- Column names: check for pattern
USER_ID, user_id, userId
```

#### 7. Line Breaks

**Empty lines between:**
```javascript
// Between functions
function foo() {}
                    // ← Empty line
function bar() {}

// Between logical blocks
var x = 1;
var y = 2;
                    // ← Empty line
if (condition) {
    // code
}
```

**Count:**
- 0 lines (no separation)
- 1 line (standard)
- 2+ lines (excessive)

#### 8. Line Length

**Check for wrapping:**
```javascript
// Short lines (<80 chars)
var shortLine = "value";

// Medium lines (80-120 chars)
var mediumLine = someFunction(arg1, arg2, arg3, arg4);

// Long lines (>120 chars)
var longLine = someFunction(argument1, argument2, argument3, argument4, argument5, argument6);
```

**Common limits:**
- 80 characters (traditional)
- 100 characters (common)
- 120 characters (modern)
- No limit (rare)

### Step 3: Statistical Analysis

For each pattern, count occurrences:

**Example for indentation:**
```
Files using tabs: 8/10 (80%)
Files using 4 spaces: 2/10 (20%)
Files using 2 spaces: 0/10 (0%)

→ Conclusion: Use tabs
```

**Example for quotes:**
```
Double quotes: 245 occurrences
Single quotes: 12 occurrences

→ Conclusion: Use double quotes
```

**Threshold for decision:**
- If one pattern appears in >70% of samples → Use it
- If split 50-60% → Ask user for preference
- If inconsistent → Recommend team discussion

### Step 4: Generate Configuration

Based on analysis results, select appropriate template from [templates.md](templates.md) and customize.

## Analysis Checklist

Use this checklist when analyzing code:

```
[ ] Sample Selection
    [ ] Selected 5-10 representative files
    [ ] Excluded generated/vendor code
    [ ] Files from different modules

[ ] Indentation Analysis
    [ ] Determined: Tabs or Spaces
    [ ] If spaces, determined size (2 or 4)
    [ ] Verified consistency

[ ] Quote Style Analysis (JS/TS/Python)
    [ ] Counted single vs double quotes
    [ ] Determined dominant style

[ ] Semicolon Analysis (JS/TS)
    [ ] Checked if required or optional
    [ ] Verified consistency

[ ] Brace Style Analysis
    [ ] Identified K&R vs Allman vs other
    [ ] Checked else/catch placement

[ ] Spacing Analysis
    [ ] Checked operator spacing
    [ ] Checked keyword spacing
    [ ] Checked function call spacing
    [ ] Checked object literal spacing

[ ] Naming Convention Analysis
    [ ] Variables: camelCase/snake_case
    [ ] Functions: camelCase/snake_case
    [ ] Classes: PascalCase
    [ ] Constants: UPPER_SNAKE_CASE

[ ] Line Break Analysis
    [ ] Empty lines between functions
    [ ] Empty lines between blocks

[ ] Line Length Analysis
    [ ] Typical line length
    [ ] Maximum line length
```

## Language-Specific Notes

### Java

**Additional patterns:**
- Import organization (alphabetical, grouped)
- JavaDoc format
- Annotation placement
- Exception handling style

**Tools to check:**
- Maven/Gradle build files for existing checkstyle config
- IDE settings (.idea/codeStyleSettings.xml)

### JavaScript/TypeScript

**Additional patterns:**
- Arrow function style: `() => {}` vs `function() {}`
- Object shorthand: `{name}` vs `{name: name}`
- Trailing commas in arrays/objects
- Template literals vs concatenation

**Tools to check:**
- Existing .eslintrc or .prettierrc
- package.json scripts

### Python

**Additional patterns:**
- Import organization (stdlib, third-party, local)
- Docstring format (Google, NumPy, reStructuredText)
- Type hints usage
- Line continuation (backslash vs parentheses)

**Standard:** PEP 8 compliance

### SQL

**Additional patterns:**
- Keyword case (UPPER vs lower)
- Table/column naming (snake_case vs camelCase)
- Alias usage
- JOIN formatting

## Quick Analysis Script

Use grep/find commands for quick analysis:

```bash
# Check indentation (tabs vs spaces)
grep -P "^\t" **/*.js | wc -l  # Count tab-indented lines
grep -P "^    " **/*.js | wc -l  # Count 4-space-indented lines

# Check quote style
grep -o "\"[^\"]*\"" **/*.js | wc -l  # Count double quotes
grep -o "'[^']*'" **/*.js | wc -l  # Count single quotes

# Check semicolon usage
grep -c ";\s*$" **/*.js  # Count line-ending semicolons

# Check brace style
grep -c "{$" **/*.js  # K&R style (same line)
grep -c "^{" **/*.js  # Allman style (next line)
```

## Edge Cases

### Mixed Styles

If codebase has mixed styles:
1. **Ask user** which style to standardize to
2. **Suggest** most common pattern
3. **Recommend** gradual migration plan
4. **Provide** both configs as options

### Generated Code

Exclude from analysis:
- node_modules/, vendor/
- build/, dist/, out/
- .min.js files
- Auto-generated files (e.g., protobuf, GraphQL)

### Legacy Code

For legacy codebases:
- Analyze recent files (last 6 months) separately
- Weight recent changes more heavily
- Consider modern standard if migrating

### Framework Conventions

Some frameworks enforce styles:
- **React**: Often uses Prettier + Airbnb
- **Angular**: Official style guide
- **Vue**: Vue style guide
- **Spring Boot**: Google Java Style

Check framework documentation before overriding.

## Output Format

When presenting analysis results to user:

```markdown
# Code Style Analysis Results

Based on analysis of 8 files from your codebase:

## Indentation
- **Pattern**: Tabs (found in 8/8 files)
- **Consistency**: 100%

## Quote Style
- **Pattern**: Double quotes (245 vs 12 single quotes)
- **Consistency**: 95%

## Semicolons
- **Pattern**: Required (found in 98% of statements)
- **Consistency**: 98%

## Brace Style
- **Pattern**: K&R (same line)
- **Consistency**: 100%

## Naming Conventions
- Variables: camelCase
- Functions: camelCase
- Classes: PascalCase
- Constants: UPPER_SNAKE_CASE

## Recommendations
Based on this analysis, I recommend using the tab-based ESLint configuration with double quotes and required semicolons.

Would you like me to generate the configuration files?
```
