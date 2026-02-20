# Configuration Templates

Complete configuration templates for all supported languages and style guides.

## Table of Contents

1. [JavaScript/TypeScript](#javascripttypescript)
2. [Java](#java)
3. [Python](#python)
4. [EditorConfig (Universal)](#editorconfig-universal)

---

## JavaScript/TypeScript

### ESLint - Tab-based Indentation (Custom)

**File: `.eslintrc.json`**

```json
{
  "env": {
    "browser": true,
    "es2021": true,
    "node": true,
    "jquery": true
  },
  "extends": "eslint:recommended",
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module"
  },
  "rules": {
    "indent": ["error", "tab", { "SwitchCase": 1 }],
    "quotes": ["error", "double"],
    "semi": ["error", "always"],
    "comma-spacing": ["error", { "before": false, "after": true }],
    "keyword-spacing": ["error", { "before": true, "after": true }],
    "space-before-function-paren": ["error", {
      "anonymous": "never",
      "named": "never",
      "asyncArrow": "always"
    }],
    "space-in-parens": ["error", "never"],
    "object-curly-spacing": ["error", "never"],
    "array-bracket-spacing": ["error", "never"],
    "brace-style": ["error", "1tbs", { "allowSingleLine": true }],
    "camelcase": ["error", { "properties": "always" }],
    "no-multiple-empty-lines": ["error", { "max": 1, "maxEOF": 0 }],
    "eol-last": ["error", "always"],
    "no-trailing-spaces": "error",
    "no-var": "off",
    "prefer-const": "off"
  }
}
```

### ESLint - Airbnb Style

**File: `.eslintrc.json`**

```json
{
  "extends": "airbnb-base",
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module"
  },
  "rules": {
    "indent": ["error", 2],
    "quotes": ["error", "single"],
    "semi": ["error", "always"]
  }
}
```

**Installation:**
```bash
npm install --save-dev eslint eslint-config-airbnb-base eslint-plugin-import
```

### ESLint - Google Style

**File: `.eslintrc.json`**

```json
{
  "extends": "google",
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "parserOptions": {
    "ecmaVersion": 12,
    "sourceType": "module"
  },
  "rules": {
    "indent": ["error", 2],
    "quotes": ["error", "single"],
    "semi": ["error", "always"],
    "max-len": ["error", { "code": 100 }]
  }
}
```

**Installation:**
```bash
npm install --save-dev eslint eslint-config-google
```

### Prettier - Tab-based

**File: `.prettierrc.json`**

```json
{
  "useTabs": true,
  "tabWidth": 4,
  "semi": true,
  "singleQuote": false,
  "quoteProps": "as-needed",
  "trailingComma": "none",
  "bracketSpacing": false,
  "arrowParens": "always",
  "endOfLine": "lf",
  "printWidth": 120
}
```

### Prettier - Airbnb Style

**File: `.prettierrc.json`**

```json
{
  "useTabs": false,
  "tabWidth": 2,
  "semi": true,
  "singleQuote": true,
  "quoteProps": "as-needed",
  "trailingComma": "es5",
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf",
  "printWidth": 100
}
```

**Installation:**
```bash
npm install --save-dev prettier
```

### TypeScript ESLint

**File: `.eslintrc.json`**

```json
{
  "parser": "@typescript-eslint/parser",
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "parserOptions": {
    "ecmaVersion": 2021,
    "sourceType": "module",
    "project": "./tsconfig.json"
  },
  "rules": {
    "indent": ["error", "tab"],
    "quotes": ["error", "double"],
    "semi": ["error", "always"],
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}
```

**Installation:**
```bash
npm install --save-dev @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

---

## Java

### CheckStyle - Google Style

**File: `checkstyle.xml`**

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">

<module name="Checker">
  <property name="charset" value="UTF-8"/>
  <property name="severity" value="warning"/>
  <property name="fileExtensions" value="java"/>

  <module name="TreeWalker">
    <!-- Naming Conventions -->
    <module name="PackageName">
      <property name="format" value="^[a-z]+(\.[a-z][a-z0-9]*)*$"/>
    </module>
    <module name="TypeName">
      <property name="format" value="^[A-Z][a-zA-Z0-9]*$"/>
    </module>
    <module name="MethodName">
      <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
    </module>
    <module name="ConstantName">
      <property name="format" value="^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$"/>
    </module>
    <module name="LocalVariableName">
      <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
    </module>

    <!-- Indentation -->
    <module name="Indentation">
      <property name="basicOffset" value="4"/>
      <property name="braceAdjustment" value="0"/>
      <property name="caseIndent" value="4"/>
      <property name="throwsIndent" value="4"/>
      <property name="lineWrappingIndentation" value="4"/>
      <property name="arrayInitIndent" value="4"/>
    </module>

    <!-- Whitespace -->
    <module name="WhitespaceAround">
      <property name="allowEmptyConstructors" value="true"/>
      <property name="allowEmptyMethods" value="true"/>
    </module>
    <module name="WhitespaceAfter"/>
    <module name="NoWhitespaceBefore"/>

    <!-- Braces -->
    <module name="LeftCurly">
      <property name="option" value="eol"/>
    </module>
    <module name="RightCurly">
      <property name="option" value="same"/>
    </module>
    <module name="NeedBraces"/>

    <!-- Line Length -->
    <module name="LineLength">
      <property name="max" value="120"/>
    </module>

    <!-- Imports -->
    <module name="AvoidStarImport"/>
    <module name="UnusedImports"/>
  </module>
</module>
```

### CheckStyle - Tab-based (Custom)

**File: `checkstyle.xml`**

```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
    "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
    "https://checkstyle.org/dtds/configuration_1_3.dtd">

<module name="Checker">
  <property name="charset" value="UTF-8"/>
  <property name="severity" value="warning"/>
  <property name="fileExtensions" value="java"/>

  <module name="TreeWalker">
    <!-- Tab Indentation -->
    <module name="RegexpSinglelineJava">
      <property name="format" value="^\t* "/>
      <property name="message" value="Indent must use tabs, not spaces"/>
      <property name="ignoreComments" value="true"/>
    </module>

    <!-- Naming Conventions -->
    <module name="PackageName">
      <property name="format" value="^[a-z]+(\.[a-z][a-z0-9]*)*$"/>
    </module>
    <module name="TypeName">
      <property name="format" value="^[A-Z][a-zA-Z0-9]*$"/>
    </module>
    <module name="MethodName">
      <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
    </module>
    <module name="ConstantName">
      <property name="format" value="^[A-Z][A-Z0-9]*(_[A-Z0-9]+)*$"/>
    </module>
    <module name="LocalVariableName">
      <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
    </module>
    <module name="ParameterName">
      <property name="format" value="^[a-z][a-zA-Z0-9]*$"/>
    </module>

    <!-- Whitespace -->
    <module name="WhitespaceAround">
      <property name="allowEmptyConstructors" value="true"/>
      <property name="allowEmptyMethods" value="true"/>
    </module>
    <module name="WhitespaceAfter"/>
    <module name="NoWhitespaceBefore"/>

    <!-- Braces (K&R Style) -->
    <module name="LeftCurly">
      <property name="option" value="eol"/>
    </module>
    <module name="RightCurly">
      <property name="option" value="same"/>
    </module>
    <module name="NeedBraces"/>

    <!-- Line Length -->
    <module name="LineLength">
      <property name="max" value="120"/>
    </module>
  </module>
</module>
```

**Maven Integration (pom.xml):**

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-checkstyle-plugin</artifactId>
  <version>3.1.2</version>
  <configuration>
    <configLocation>checkstyle.xml</configLocation>
    <encoding>UTF-8</encoding>
    <consoleOutput>true</consoleOutput>
    <failsOnError>true</failsOnError>
  </configuration>
  <executions>
    <execution>
      <id>validate</id>
      <phase>validate</phase>
      <goals>
        <goal>check</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

---

## Python

### Black Configuration

**File: `pyproject.toml`**

```toml
[tool.black]
line-length = 100
target-version = ['py38', 'py39', 'py310']
include = '\.pyi?$'
extend-exclude = '''
/(
  # directories
  \.eggs
  | \.git
  | \.hg
  | \.mypy_cache
  | \.tox
  | \.venv
  | build
  | dist
)/
'''
```

### Flake8 Configuration

**File: `.flake8`**

```ini
[flake8]
max-line-length = 100
extend-ignore = E203, W503
exclude =
    .git,
    __pycache__,
    .venv,
    venv,
    build,
    dist
per-file-ignores =
    __init__.py:F401
```

### Pylint Configuration

**File: `.pylintrc`**

```ini
[MASTER]
jobs=4

[MESSAGES CONTROL]
disable=C0111,C0103,R0903

[FORMAT]
indent-string=\t
max-line-length=100
```

**Installation:**
```bash
pip install black flake8 pylint
```

---

## EditorConfig (Universal)

### Tab-based (General)

**File: `.editorconfig`**

```ini
# EditorConfig is awesome: https://EditorConfig.org

root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = tab
indent_size = 4

[*.{json,yml,yaml}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

### Space-based (Airbnb/Google Style)

**File: `.editorconfig`**

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.{java,py}]
indent_size = 4

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab
```

### Multi-language Project

**File: `.editorconfig`**

```ini
root = true

# Defaults
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

# JavaScript/TypeScript
[*.{js,jsx,ts,tsx}]
indent_style = tab
indent_size = 4

# Java
[*.java]
indent_style = tab
indent_size = 4

# Python
[*.py]
indent_style = space
indent_size = 4

# Web files
[*.{html,css,scss,less}]
indent_style = tab
indent_size = 2

# Config files
[*.{json,yml,yaml,xml}]
indent_style = space
indent_size = 2

# SQL
[*.sql]
indent_style = tab
indent_size = 2

# Markdown
[*.md]
trim_trailing_whitespace = false

# Shell scripts
[*.sh]
indent_style = tab

# Makefiles
[Makefile]
indent_style = tab
```

---

## NPM Scripts for JavaScript/TypeScript

Add to `package.json`:

```json
{
  "scripts": {
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "lint:fix": "eslint . --ext .js,.jsx,.ts,.tsx --fix",
    "format": "prettier --write \"**/*.{js,jsx,ts,tsx,json,css,md}\"",
    "format:check": "prettier --check \"**/*.{js,jsx,ts,tsx,json,css,md}\""
  },
  "devDependencies": {
    "eslint": "^8.0.0",
    "prettier": "^3.0.0"
  }
}
```

## VS Code Integration

**File: `.vscode/settings.json`**

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[java]": {
    "editor.defaultFormatter": "redhat.java"
  },
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter"
  }
}
```

## IntelliJ IDEA Integration

### For Java (CheckStyle)

1. Install CheckStyle-IDEA plugin
2. Settings → Tools → Checkstyle
3. Add Configuration File → Select `checkstyle.xml`
4. Enable "Scan Scope: All sources"

### For JavaScript (ESLint + Prettier)

1. Settings → Languages & Frameworks → JavaScript → Code Quality Tools → ESLint
2. Check "Automatic ESLint configuration"
3. Settings → Languages & Frameworks → JavaScript → Prettier
4. Check "On save" for automatic formatting
