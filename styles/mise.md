# mise Style Guide

## Core Principle

mise ensures consistency across all execution contexts: developer, agent, pre-commit hooks, and CI/CD.

**Golden Rule**: ALL commands MUST be executed via mise. Never invoke executables directly.

## Configuration Files

```text
.mise.toml          → Base config: tools, tasks, shared env vars
.mise.{env}.toml    → Environment overrides: dev, ci, staging, prod, testing
```

Environment selection:

- CLI: `mise run --env ci test`
- Env var: `MISE_ENV=ci mise run test`
- Multiple: `MISE_ENV=ci,testing mise run test` (last wins)

## Tools

```toml
[tools]
go = "latest"
bun = "1.2.3"
golangci-lint = "latest"
```

mise manages ALL tool versions. Never use language-specific version managers.

## Environment Variables

### Declaration

```toml
[env]
APP_NAME = "myapp"
NODE_ENV = "production"
_.path = ["./bin", "{{ config_root }}/scripts"]
```

### Secrets (SOPS Mandatory)

```toml
# .mise.dev.toml
[settings]
sops.age_key_file = "~/.keys/project/dev.age.key"

[env]
_.file = { path = "secrets/dev.enc.yaml", redact = true }
```

All secrets MUST be encrypted with SOPS. Never commit unencrypted secrets.

### Templates

```toml
PROJECT_NAME = "{{ cwd | basename }}"
BIN_PATH = "{{ config_root }}/bin"
USER_DIR = "{{ env.HOME }}/projects"
ARCH = "{{ arch() }}"
```

## Tasks

### Namespace Pattern

Use `<category>:<action>:<mode>` structure:

```toml
[tasks."go:lint:fix"]
run = "golangci-lint run --fix"
dir = "backend"
description = "Lint and fix Go code"

[tasks."go:lint:nofix"]
run = "golangci-lint run"
dir = "backend"
description = "Lint Go code"
```

### Hybrid Approach

**Semantic tasks** for common workflows:

```toml
[tasks.build]
run = "go build -o dist/app ./cmd"
description = "Build application"

[tasks.test]
depends = ["lint:all:nofix"]
run = "go test ./..."
description = "Run all tests"
```

**Thin wrappers** for ad-hoc use:

```toml
[tasks.go]
run = "go"
description = "Run Go"

[tasks.bun]
run = "bun"
description = "Run Bun"
```

Then: `mise run go version` or `mise run bun install`

### Task Configuration

```toml
[tasks.example]
run = "command"                    # Required
description = "Task description"   # Recommended
depends = ["other-task"]           # Dependencies
dir = "backend"                    # Working directory
sources = ["src/**/*.go"]          # Input files
outputs = ["dist/binary"]          # Output files
raw = true                         # Direct shell connection
env = { DEBUG = "true" }           # Task-specific env
```

### Complex Tasks

For multi-step tasks, use scripts directory:

```toml
[tasks.deploy]
run = "./scripts/deploy.sh"
description = "Deploy application"
raw = true
```

**Never use file-based tasks** (`.mise/tasks/` directory). Always use TOML + scripts.

## Environments

- `dev` - Local development with secrets
- `ci` - CI/CD pipelines (GitHub Actions, etc.)
- `staging` - Pre-production environment
- `prod` - Production environment
- `testing` - Test execution context

## Hooks

Available for automation (use when beneficial):

- `enter` - Project entry setup
- `preinstall` / `postinstall` - Tool setup
- `watch_files` - Auto-rebuild on changes

```toml
[hooks]
enter = "echo 'Welcome to {{ env.APP_NAME }}'"
```

## Monorepo

Enable with `experimental_monorepo_root = true` in root `.mise.toml`.

Task execution:

```bash
mise //projects/api:build        # Absolute path from root
mise //...:test                  # All projects
mise '//projects/*:lint'         # Pattern matching
```

Each sub-project gets its own `.mise.toml` with tools and tasks.

## ANTI-PATTERNS

### NEVER

❌ **Never use `.mise.local.toml`** - Banned. Use environment-specific configs instead.

❌ **Never invoke executables directly** - Always via mise:

```bash
# BAD
go test ./...
bun install
npm run build

# GOOD
mise run go test ./...
mise run bun install
mise run build
```

❌ **Never use language version managers** - No nvm, rbenv, pyenv, etc. mise manages all tools.

❌ **Never create file-based tasks** - Use `scripts/` directory + TOML tasks.

❌ **Never hardcode paths** - Use templates: `{{ config_root }}`, `{{ env.HOME }}`

❌ **Never bypass mise in CI/CD** - Breaks environment consistency.

❌ **Never modify PATH manually** - Use `_.path` in env config.

❌ **Never install tools globally** - All tools in `[tools]` section.

❌ **Never commit unencrypted secrets** - SOPS mandatory.

❌ **Never upgrade tools outside mise** - No `bun upgrade`, `go get -u` directly.

❌ **Never set env vars outside mise** - Breaks golden path consistency.

## Quick Reference

```bash
mise install                     # Install tools from .mise.toml
mise run <task>                  # Execute task
mise run --env ci <task>        # Run with environment
mise tasks                       # List available tasks
mise set KEY=value              # Set environment variable
mise ls                          # List installed tools
mise up                          # Upgrade tools
```

## Validation

Before committing:

1. All commands run via mise tasks
2. Environment variables in mise config
3. Secrets encrypted with SOPS
4. Tasks use namespace pattern
5. No `.mise.local.toml` files
