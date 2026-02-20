---
name: code-formatter
description: Generate language-specific code formatting configuration files (ESLint, Prettier, EditorConfig, CheckStyle) based on existing codebase analysis. Use when users request: (1) Creating ESLint or Prettier configs, (2) Generating formatting rules from code analysis, (3) Setting up code style guides, (4) Creating .editorconfig files, (5) Establishing team coding standards, or (6) Any task involving code formatting configuration generation.
---

# Code Formatter Configuration Generator

Generate language-specific code formatting configuration files by analyzing existing codebases or using predefined style templates.

## Quick Start

When user requests formatting configs:

1. **Ask for style preference** (if not specified):
   - Analyze existing codebase
   - Use predefined template (Google, Airbnb, Standard)
   - Custom specifications

2. **Identify target languages** from user request or project structure

3. **Generate appropriate configs** using templates from [references/templates.md](references/templates.md)

4. **Create files** in project root with proper naming

## Supported Languages

- **JavaScript/TypeScript**: ESLint, Prettier
- **Java**: CheckStyle, EditorConfig
- **Python**: Black, Flake8, .editorconfig
- **General**: .editorconfig (cross-language)

## Workflow

### From Codebase Analysis

When analyzing existing code:

1. Read sample files (5-10 representative files)
2. Identify patterns:
   - Indentation (tabs vs spaces, size)
   - Quote style (single vs double)
   - Semicolons (required vs optional)
   - Brace style (K&R, Allman, etc.)
   - Line breaks and spacing
   - Naming conventions
3. Generate configs matching discovered patterns
4. Use [references/analysis-guide.md](references/analysis-guide.md) for detailed analysis instructions

### From Templates

When using predefined templates:

1. Ask user to choose style:
   - Airbnb (JavaScript/TypeScript)
   - Google (Java, JavaScript, Python)
   - Standard (JavaScript)
   - Custom (specify rules)

2. Load template from [references/templates.md](references/templates.md)

3. Customize if needed

4. Generate config files

## File Generation

Create these files in project root:

### JavaScript/TypeScript Projects
- `.eslintrc.json` - ESLint configuration
- `.prettierrc.json` - Prettier configuration
- `.editorconfig` - Editor settings

### Java Projects
- `checkstyle.xml` - CheckStyle configuration
- `.editorconfig` - Editor settings

### Python Projects
- `pyproject.toml` - Black/Flake8 configuration
- `.editorconfig` - Editor settings

### Multi-language Projects
- `.editorconfig` - Universal editor settings
- Language-specific configs as needed

## Configuration Details

For comprehensive configuration options and templates, see:
- [references/templates.md](references/templates.md) - Complete config templates for all languages
- [references/analysis-guide.md](references/analysis-guide.md) - Codebase analysis methodology

## Output Format

Always explain:
1. What each config file does
2. How to install required tools
3. How to run formatters/linters
4. Integration with IDEs (VS Code, IntelliJ, etc.)

Example output structure:
```
# ESLint Configuration

Created `.eslintrc.json` based on your codebase analysis.

## Key Rules
- Indentation: Tabs
- Quotes: Double quotes
- Semicolons: Required

## Installation
npm install --save-dev eslint

## Usage
npm run lint
```

## Best Practices

- **Consistency**: Match existing codebase patterns when analyzing
- **Team standards**: Prioritize team preferences over personal style
- **Incremental adoption**: Start with core rules, add more gradually
- **Documentation**: Always explain reasoning behind rules
- **IDE integration**: Provide setup instructions for common editors
