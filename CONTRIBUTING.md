# Contributing to MQL5 Expert Development System

First off, thank you for considering contributing to this project! It's people like you that make this skill better for everyone.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [How Can I Contribute?](#how-can-i-contribute)
- [Development Setup](#development-setup)
- [Contribution Guidelines](#contribution-guidelines)
- [Style Guide](#style-guide)
- [Commit Messages](#commit-messages)
- [Pull Request Process](#pull-request-process)

---

## Code of Conduct

### Our Pledge

We are committed to providing a welcoming and inspiring community for all. Please be respectful and constructive in all interactions.

### Our Standards

**Positive behaviors:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable behaviors:**
- Trolling, insulting/derogatory comments, and personal attacks
- Public or private harassment
- Publishing others' private information without permission
- Other conduct which could reasonably be considered inappropriate

---

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates.

When reporting a bug, include:
- **Clear title**: Describe the issue concisely
- **Steps to reproduce**: Numbered list of exact steps
- **Expected behavior**: What should happen
- **Actual behavior**: What actually happens
- **Environment**: Antigravity version, OS, etc.
- **Code sample**: Minimal code that reproduces the issue
- **Error messages**: Full error text if applicable

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues.

When suggesting an enhancement:
- **Use a clear title**: Describe the enhancement
- **Provide detailed description**: Explain why this would be useful
- **Include examples**: Show how it would work
- **Benefits**: Explain who would benefit and how

### Contributing Code

Types of contributions we welcome:

1. **New Templates**: Add code templates to `resources/config/code_templates.json`
2. **Enhanced Rules**: Improve linting rules in `resources/config/linter_rules.yaml`
3. **Documentation**: Enhance knowledge modules in `resources/reference/`
4. **Core Improvements**: Enhance `SKILL.md` with new patterns
5. **Bug Fixes**: Fix issues in existing templates or documentation
6. **Examples**: Add real-world examples to documentation

---

## Development Setup

### Prerequisites

- Git
- Text editor (VS Code recommended)
- MetaTrader 5 (for testing)
- Antigravity with Gemini 2.0

### Setup Steps

1. **Fork the repository**
   ```bash
   # Click 'Fork' on GitHub, then:
   git clone https://github.com/YOUR_USERNAME/mql5-expert-antigravity-skill.git
   cd mql5-expert-antigravity-skill
   ```

2. **Create a branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make your changes**
   - Edit files as needed
   - Follow the style guide below
   - Test thoroughly

4. **Test your changes**
   - Install the skill in Antigravity
   - Generate test EAs/indicators
   - Verify generated code compiles in MetaTrader 5
   - Check for any errors or warnings

---

## Contribution Guidelines

### Adding Code Templates

When adding to `resources/config/code_templates.json`:

1. **Complete code only**: No placeholders or TODOs
2. **Include safety checks**: All critical validations
3. **Add documentation**: Function contracts where applicable
4. **Test compilation**: Code must compile without errors
5. **Provide context**: Add tags for easy discovery

**Template format:**
```json
{
  "name": "Descriptive Template Name",
  "tags": ["category", "keywords", "for", "search"],
  "code": "// Complete working code here..."
}
```

### Adding Linting Rules

When adding to `resources/config/linter_rules.yaml`:

1. **Clear criteria**: Rule must be objectively checkable
2. **Appropriate severity**: CRITICAL, ERROR, WARNING, or INFO
3. **Helpful message**: Explain what's wrong and how to fix
4. **Examples**: Provide good and bad examples
5. **Justification**: Explain why this rule matters

**Rule format:**
```yaml
- id: RULE001
  name: "RuleName"
  severity: WARNING
  pattern: "regex pattern if applicable"
  message: "Clear explanation of what's wrong and how to fix"
```

### Enhancing Knowledge Modules

When updating `resources/reference/*.md`:

1. **Clear explanations**: Assume reader is learning
2. **Working examples**: All code must be complete and functional
3. **Best practices**: Include "do" and "don't" examples
4. **Cross-references**: Link to related modules
5. **Consistent formatting**: Follow existing markdown style

### Improving Core Skill

When updating `SKILL.md`:

1. **Maintain structure**: Don't break existing sections
2. **YAML frontmatter**: Don't modify unless necessary
3. **Complete patterns**: All code examples must be functional
4. **Safety first**: Never compromise safety for convenience
5. **Test thoroughly**: Verify Antigravity can load and use changes

---

## Style Guide

### Markdown

- Use ATX-style headers (`#` not `===`)
- One blank line between sections
- Use fenced code blocks with language specifier
- Wrap lines at 100 characters (except code blocks and tables)
- Use **bold** for emphasis, *italic* for terms

### Code Blocks

```cpp
// Always specify language
// Include comments explaining complex logic
// Follow MQL5 style from SKILL.md

void ExampleFunction()
{
   // 3-space indentation
   Print("Example");
}
```

### MQL5 Code Style

Follow standards in `SKILL.md`:
- **Indentation**: 3 spaces
- **Braces**: 1TBS style
- **Naming**: `CPascalCase` for classes, `PascalCase` for functions, `camelCase` for variables
- **Comments**: Explain WHY not WHAT
- **Safety**: Always include all mandatory checks

---

## Commit Messages

### Format

```
type(scope): Brief description (50 chars max)

Detailed explanation if needed (wrap at 72 chars)
- Bullet points for multiple changes
- Reference issues with #123

Closes #123
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting, no code change
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance tasks

### Examples

**Good:**
```
feat(templates): Add trailing stop template

- Implements ATR-based trailing stop
- Includes position profit tracking
- Full error handling and cleanup

Closes #42
```

**Bad:**
```
updated stuff
```

---

## Pull Request Process

### Before Submitting

- [ ] Code follows style guide
- [ ] All new code has been tested
- [ ] Generated EAs compile without errors
- [ ] Documentation updated if needed
- [ ] Commit messages follow guidelines
- [ ] Branch is up to date with main

### PR Description Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Documentation update
- [ ] Code refactoring

## Testing
Describe testing performed:
- Generated EA with new template
- Compiled successfully in MT5
- Tested in Strategy Tester
- Results: [describe]

## Checklist
- [ ] Code follows style guide
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] No breaking changes (or noted in description)

## Related Issues
Closes #123
```

### Review Process

1. **Automated checks**: Must pass (if any)
2. **Code review**: At least one maintainer approval
3. **Testing**: Maintainer may test before merging
4. **Discussion**: Address all review comments
5. **Merge**: Squash and merge to main

### After Merge

- Your contribution will be credited in release notes
- Thank you for making this project better!

---

## Recognition

Contributors will be:
- Listed in release notes
- Credited in README (for significant contributions)
- Given a shoutout on project updates

---

## Questions?

- Open a [discussion](https://github.com/yourusername/mql5-expert-antigravity-skill/discussions)
- Ask in an issue
- Review existing documentation

---

**Thank you for contributing! 🎉**

Every contribution, no matter how small, helps make this skill better for the entire community.
