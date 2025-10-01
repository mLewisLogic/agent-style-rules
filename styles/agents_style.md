# AGENTS.md Style Guide

Reference for maintaining AGENTS.md files after code changes.

## Core Principle

AGENTS.md files provide immediate operational context for AI agents working in specific directories. Keep them concise and actionable.

## Hierarchy

- **Root**: Project-wide commands and workflows
- **Directory**: Component patterns and integration points
- **Subdirectory**: Implementation details only if unique

## When to Create/Update

### Create when

- Directory has >3 files with distinct patterns
- Local conventions differ from parent
- Complex integrations need explanation

### Update when

- Architecture changes
- New patterns introduced
- Integration points modified

### Delete when

- Directory deprecated
- Content duplicates parent
- No unique value added

## What Goes In

✅ **Include:**

- Immediate commands (build, test, lint)
- Architecture decisions and rationale
- Integration patterns
- Local conventions that differ from parent
- Performance considerations

❌ **Exclude:**

- API documentation → docs/
- Setup guides → README
- Frequently changing implementation details
- Content already in parent AGENTS.md
- Generic best practices

## Keep It Tight

- One-line purpose statement
- Bullet points over paragraphs
- Commands over explanations
- Link to specs with @prefix for details
- No bold formatting needed

## Simple Template

```markdown
# Component Name

One-line purpose.

## Commands
+ npm test - Run tests
+ npm build - Build component

## Architecture
Key patterns unique to this component.

## Integrations
How this connects to other components.

## See Also
- @specs/detailed-spec.md
```

## Quality Check

Before committing, ask:

1. Can an agent immediately act on this info?
2. Is it unique to this directory?
3. Is it under 100 lines?
4. Does it avoid duplicating the parent?

If any answer is "no", refactor or remove.
