---
name: ai-agent-rules
description: Generate custom AI coding rules and guidelines for AI-powered IDEs and agents (Cursor, Windsurf, Cline, Continue, Zed, Copilot, Antigravity, Kiro, etc.) by analyzing project codebases. Automatically detects IDE from existing rule files or asks user to select. Use when users request: (1) Creating .cursorrules, .windsurfrules, .clinerules, .continuerules, or similar AI agent configuration files, (2) Generating project-specific coding guidelines for AI assistants, (3) Documenting code style and architecture patterns for AI agents, (4) Setting up context and constraints for AI pair programming, (5) Creating custom instructions for AI code generation, or (6) Any task involving AI agent behavior customization for development projects.
---

# AI Agent Rules Generator

Generate project-specific rules and guidelines for AI-powered IDEs and coding agents by analyzing existing codebases and documenting patterns, conventions, and constraints.

## Quick Start

When user requests AI agent rules:

1. **Auto-detect or ask for target IDE**:

   **Auto-detection steps:**
   - Check for existing rule files in project root:
     - `.cursorrules` → Cursor IDE
     - `.windsurfrules` → Windsurf IDE
     - `.clinerules` or `.clauiderules` → Cline/Claude Code
     - `.zed/settings.json` → Zed IDE
     - `.github/copilot-instructions.md` → GitHub Copilot
   - Check VS Code extensions in `.vscode/extensions.json`:
     - `cursor-ai` → Cursor
     - `windsurf-ai` → Windsurf
     - `continue.continue` → Continue

   **If no IDE detected, ask user:**
   Use AskUserQuestion tool with these options:
   - Cursor (most popular, natural language)
   - Windsurf (YAML/structured format)
   - Cline/Claude Code (markdown instructions)
   - Continue (JSON format)
   - Zed (JSON settings)
   - GitHub Copilot (markdown comments)
   - Antigravity/Kiro (markdown format)
   - Universal (AI_CODING_GUIDE.md for all IDEs)

2. **Analyze codebase** or **use user specifications**

3. **Generate appropriate rules file** based on detected/selected IDE:
   - Code style and formatting patterns
   - Architecture and design patterns
   - Naming conventions
   - Framework-specific guidelines
   - Project constraints and preferences

4. **Create file** with correct name in project root:
   - Cursor → `.cursorrules`
   - Windsurf → `.windsurfrules`
   - Cline/Claude Code → `.clinerules` or project-specific instructions
   - Continue → `.continuerules` (JSON)
   - Zed → `.zed/settings.json`
   - GitHub Copilot → `.github/copilot-instructions.md`
   - Antigravity/Kiro → `.aicoderules` or `AI_CODING_GUIDE.md`
   - Universal → `AI_CODING_GUIDE.md`

## Supported Platforms

- **Cursor**: `.cursorrules` (natural language instructions, markdown)
- **Windsurf**: `.windsurfrules` (YAML/JSON structured rules)
- **Cline/Claude Code**: `.clinerules` or custom instructions (markdown)
- **Continue**: `.continuerules` (JSON configuration)
- **Zed**: `.zed/settings.json` (JSON settings)
- **GitHub Copilot**: `.github/copilot-instructions.md` (markdown)
- **Antigravity**: `.aicoderules` or `AI_CODING_GUIDE.md` (markdown)
- **Kiro**: `AI_CODING_GUIDE.md` (markdown)
- **Universal**: `AI_CODING_GUIDE.md` (platform-agnostic markdown)

## Rule Categories

### 1. Code Style & Formatting
- Indentation (tabs/spaces)
- Naming conventions
- Quote style, semicolons
- Line length, formatting

### 2. Architecture Patterns
- Layer structure (MVC, Clean Architecture)
- File organization
- Dependency injection patterns
- Error handling approaches

### 3. Framework-Specific
- Spring Boot conventions
- React/Vue patterns
- Database query patterns
- API design standards

### 4. Project Constraints
- Technology stack limitations
- Performance requirements
- Security guidelines
- Testing requirements

### 5. Domain Knowledge
- Business logic patterns
- Database schema relationships
- API endpoint conventions
- Common workflows

## Workflow

### From Codebase Analysis

1. **Sample code** (read 10-15 representative files)
2. **Extract patterns**:
   - Controller/Service layer patterns
   - DTO/VO usage
   - Exception handling
   - Transaction management
   - Authentication/Authorization
3. **Identify conventions**:
   - URL patterns (`.do` endpoints, RESTful)
   - Response formats (Map, ResponseEntity)
   - Database naming (tb_ prefix, UPPER_CASE columns)
4. **Document architecture**:
   - Package structure
   - Layer responsibilities
   - Common utilities
5. **Generate rules** using templates from [references/templates.md](references/templates.md)

### From User Specifications

1. **Ask clarifying questions**:
   - Framework/stack being used
   - Specific patterns to enforce
   - Common mistakes to avoid
2. **Select template** from [references/templates.md](references/templates.md)
3. **Customize** based on requirements
4. **Generate rules file**

## Analysis Guide

For detailed codebase analysis methodology, see [references/analysis-guide.md](references/analysis-guide.md).

Key aspects to analyze:
- **Architecture layers** (Controller → Service → DAO/Mapper)
- **Naming patterns** (method prefixes: select*, insert*, update*, delete*)
- **Response handling** (Map<String, Object> vs DTOs)
- **Exception patterns** (try-catch, throws, transaction management)
- **Database patterns** (MyBatis XML, JPA, raw SQL)
- **Frontend patterns** (JSP, React, Vue, template structure)

## Output Structure

Generated rules should include:

```markdown
# Project Overview
- Technology stack
- Architecture pattern
- Key frameworks

# Code Style Rules
- Formatting preferences
- Naming conventions
- File organization

# Architecture Guidelines
- Layer structure
- Responsibility separation
- Common patterns

# Framework-Specific Rules
- Spring annotations usage
- Transaction management
- API design patterns

# Common Patterns
- Error handling
- Logging approach
- Response formats

# Constraints
- What to avoid
- Performance considerations
- Security requirements

# Examples
- Sample controller
- Sample service
- Sample DAO/repository
```

## Platform-Specific Formats

### Cursor (.cursorrules)

Natural language, conversational style. Focus on clear instructions:

```markdown
You are working on a Spring Boot eGovFrame project with MyBatis.

## Code Style
- Use tabs for indentation
- camelCase for Java methods and variables
- PascalCase for classes
...

## Architecture
- Follow Controller → Service → Mapper pattern
- Controllers handle HTTP requests, return Map<String, Object>
- Services contain business logic
...

## Common Patterns
When creating a new feature:
1. Create VO in model package
2. Create Mapper interface
3. Create Service interface and implementation
...
```

### Windsurf (.windsurfrules)

Structured YAML or JSON format:

```yaml
project:
  type: "Spring Boot"
  framework: "eGovFrame"

codeStyle:
  indentation: "tab"
  naming:
    methods: "camelCase"
    classes: "PascalCase"

architecture:
  pattern: "Layered (MVC)"
  layers:
    - controller
    - service
    - mapper/dao

patterns:
  controller:
    returnType: "Map<String, Object>"
    annotations: ["@Controller", "@RequestMapping"]
  ...
```

### Cline/Claude Code (.clinerules)

Markdown format with clear sections:

```markdown
# Project Context

Spring Boot eGovFrame project with MyBatis ORM.

## Code Guidelines

- Tabs for indentation
- Controller → Service → Mapper architecture
- Map<String, Object> for AJAX responses

## Patterns

[Detailed patterns with examples]
```

### Continue (.continuerules)

JSON configuration format:

```json
{
  "rules": {
    "codeStyle": {
      "indentation": "tab",
      "namingConvention": "camelCase"
    },
    "architecture": {
      "pattern": "MVC",
      "layers": ["controller", "service", "mapper"]
    }
  }
}
```

### Zed (.zed/settings.json)

JSON settings integrated with Zed's AI:

```json
{
  "assistant": {
    "default_model": {
      "provider": "openai",
      "model": "gpt-4"
    },
    "custom_instructions": "You are working on a Spring Boot project...",
    "code_generation_rules": {
      "indentation": "tab",
      "architecture": "MVC"
    }
  }
}
```

### GitHub Copilot (.github/copilot-instructions.md)

Markdown instructions for Copilot:

```markdown
# GitHub Copilot Instructions

## Project Context
Spring Boot eGovFrame with MyBatis

## Code Style
- Use tabs (not spaces)
- camelCase for methods
- PascalCase for classes

## Architecture
Controller → Service → Mapper pattern
```

### Antigravity/Kiro (AI_CODING_GUIDE.md or .aicoderules)

Universal markdown format:

```markdown
# AI Coding Guidelines for [Project Name]

This document provides context and rules for AI coding assistants working on this project.

## Technology Stack
- Java 11 + Spring Boot 2.7
- MyBatis 3.5
- JSP + jQuery
...

## Architecture
[Detailed architecture description]

## Code Patterns
[Common patterns with examples]
...
```

## Best Practices

1. **Be specific**: Include actual code examples from the project
2. **Explain why**: Context helps AI make better decisions
3. **Include anti-patterns**: What NOT to do
4. **Keep updated**: Update rules as project evolves
5. **Test generated code**: Verify AI follows the rules

## Templates

For complete rule templates by platform and stack, see [references/templates.md](references/templates.md).

For analysis methodology, see [references/analysis-guide.md](references/analysis-guide.md).
