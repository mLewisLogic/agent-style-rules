# Go Style Guide

## Core Philosophy

- **Stdlib first**: Use standard library before external dependencies
- **Simplicity wins**: Clear code > clever code
- **References**: [Google Go Style Guide](https://google.github.io/styleguide/go/), [Code Review Comments](https://go.dev/wiki/CodeReviewComments)

## Error Handling

Wrap with context using `%w`:

```go
if err != nil {
    return fmt.Errorf("processing %s: %w", id, err)
}
```

Multiple errors (Go 1.20+):

```go
err := errors.Join(err1, err2, err3)
if errors.Is(err, target) { ... }

// Unwrap joined errors
if me, ok := err.(interface{ Unwrap() []error }); ok {
    for _, e := range me.Unwrap() { ... }
}
```

**Avoid sentinel errors** (6x slower, tight coupling). Use custom error types instead.

**Always check errors**:

- Return as last value
- No naked error handling (`_ = err`)
- No `panic()` in library code (only `main`)

## Logging (slog)

Use structured logging with `log/slog`:

```go
logger := slog.New(slog.NewJSONHandler(os.Stderr, &slog.HandlerOptions{
    Level: slog.LevelInfo,
}))

// Request-scoped fields
logger = logger.With("request_id", id, "user_id", uid)
logger.Info("processing request", "duration", elapsed)
```

Sanitize secrets with `LogValuer`:

```go
type User struct { ID, Password string }

func (u User) LogValue() slog.Value {
    return slog.GroupValue(slog.String("id", u.ID))
    // Password omitted
}
```

## Context

**Always first parameter**, never store in structs:

```go
func Process(ctx context.Context, data Data) error
```

Use timeouts for external calls:

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()

result, err := externalAPI(ctx)
if errors.Is(err, context.DeadlineExceeded) { ... }
```

Context values (request-scoped only):

```go
type ctxKey struct{}

func WithRequestID(ctx context.Context, id string) context.Context {
    return context.WithValue(ctx, ctxKey{}, id)
}
```

**Not for**: config, dependencies, optional parameters

## Testing

Table-driven tests:

```go
func TestParse(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    Result
        wantErr bool
    }{
        {"valid", "foo", Result{Value: "foo"}, false},
        {"empty", "", Result{}, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            t.Parallel() // If independent

            got, err := Parse(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("error = %v, wantErr %v", err, tt.wantErr)
            }
            // ...
        })
    }
}
```

Modern utilities (Go 1.24+):

- `t.TempDir()` - auto-cleanup temp directory
- `t.Context()` - auto-canceled context
- `t.Cleanup(func)` - deferred cleanup
- `t.Helper()` - mark helper functions

Benchmarking:

```bash
go test -bench . -benchmem
go test -cpuprofile cpu.prof -memprofile mem.prof
go tool pprof -http=:8000 cpu.prof
```

## Concurrency

Use `errgroup` for coordinated goroutines:

```go
import "golang.org/x/sync/errgroup"

g, ctx := errgroup.WithContext(ctx)
g.SetLimit(10) // Max concurrent

for _, item := range items {
    item := item // Capture
    g.Go(func() error {
        return process(ctx, item)
    })
}

return g.Wait() // Returns first error
```

Channel rules:

- Close from sender side only
- Check closed: `val, ok := <-ch`
- Buffered for producer/consumer

**Always define goroutine exit strategy**. Run with `-race` flag in CI.

## Performance

Preallocate slices when size known (70% faster):

```go
items := make([]Item, 0, expectedSize)
for ... { items = append(items, item) }

// Reuse slices
items = items[:0] // Reset without realloc
```

String concatenation:

```go
var sb strings.Builder
sb.WriteString("foo")
sb.WriteString("bar")
result := sb.String()
```

**Profile before optimizing**: Use `pprof`, validate with benchmarks.

Pointer vs value receivers:

- Value: Small structs (<64 bytes), immutable
- Pointer: Large structs, mutations

## Project Structure

Start simple, add structure when needed:

```
# Small (<100 files)
myapp/
├── main.go
└── go.mod

# Medium (library/CLI)
myapp/
├── cmd/myapp/main.go
├── internal/      # Private packages
└── go.mod

# Large (service)
myapp/
├── cmd/
├── internal/
│   ├── api/
│   └── domain/
└── go.mod
```

Key directories:

- `cmd/` - Executables (thin main, import from internal)
- `internal/` - Private packages (compiler-enforced)
- `pkg/` - Public packages (optional, debated)

**Avoid deep nesting** (max 3 levels)

## Naming

**Packages**: Short, lowercase, single word (`http`, `json`, `user`)

**Interfaces**: `-er` suffix for single-method (`Reader`, `Writer`)

**Functions**:

- No stuttering: `user.New()` not `user.NewUser()`
- Getters: `Name()` not `GetName()`
- Booleans: `Is`, `Has`, `Can`, `Should`

**Constants**: CamelCase for exported

## Interface Design

**Small interfaces** (1-3 methods):

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

**Accept interfaces, return structs**:

```go
func Process(r io.Reader) (*Result, error) // Flexible
```

Define interfaces at usage point (consumer side, not implementer).

## Modern Go (1.23+)

**Tool directives** (Go 1.24, replaces `tools.go`):

```go
// go.mod
tool (
    golang.org/x/tools/cmd/stringer
    github.com/golangci/golangci-lint/cmd/golangci-lint
)

// Run: go tool golangci-lint run
```

**Range-over-func iterators** (Go 1.23):

```go
func (c *Collection) All() iter.Seq[Item] {
    return func(yield func(Item) bool) {
        for _, item := range c.items {
            if !yield(item) { return } // Check return
        }
    }
}

// Usage
for item := range collection.All() { ... }
```

**PGO** (Profile-Guided Optimization, Go 1.22+): 2-14% improvement

## Dependencies

- `go mod tidy` before commits
- Pin major versions in `go.mod`
- Use stdlib when possible

## Formatting

- `gofmt` and `goimports` in CI
- Line length ~100 characters
- Group imports: stdlib, external, internal

## Linting

Use `golangci-lint` with strict settings:

```yaml
linters:
  enable:
    - errcheck
    - gosec
    - govet
    - staticcheck
    - unused
    - gocritic
    - revive
```

## File I/O

Atomic writes (write to temp + rename):

```go
tmp, _ := os.CreateTemp(filepath.Dir(path), ".tmp-*")
tmp.Write(data)
tmp.Close()
os.Rename(tmp.Name(), path) // Atomic
```

Always `defer file.Close()` immediately after open.

Cross-platform paths: `filepath.Join("dir", "file")`

## Time Handling

- Store as `time.Time` (not strings/timestamps)
- UTC for storage: `time.Now().UTC()`
- Serialize with `time.RFC3339` (ISO 8601)

## Forbidden

- No `init()` without clear necessity
- No global mutable state
- No empty error handling
- No goroutine leaks (define exit strategy)
- No deep nesting (>3 levels)
- No magic numbers (use constants)
- No naked returns in complex functions
- No `else` after `return` (early returns preferred)
- No reflection unless necessary
