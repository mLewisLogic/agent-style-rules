# Go Style Guide

## Error Handling

**Wrap for:** Context and debugging
**Return raw for:** Type assertions, validation

- Wrap with `fmt.Errorf("operation failed: %w", err)`
- Use `errors.Is()` and `errors.As()` for checking
- No `panic()` in library code (only `main` packages)
- Return errors as last value

## Module Organization

**Domain-driven:** Business logic in domain packages
**Avoid:** Deep nesting (max 3 levels)

- One module per repository
- Re-export important types at package root
- Use `internal/` for implementation details
- Group related functionality in packages

## Testing

- Table-driven tests with `t.Run()`
- Standard library testing preferred (testify optional)
- `_test.go` files in same package
- Test public behavior, not implementation
- Benchmark critical paths with `testing.B`

## Naming

**Packages:** Short, lowercase, single word
**Interfaces:** -er suffix (`Reader`, `Writer`)
**Constants:** CamelCase or ALL_CAPS for exported

- No stuttering: `log.Info()` not `log.LogInfo()`
- Boolean functions: `Is`, `Has`, `Can`, `Should`
- Getters: `Name()` not `GetName()`

## Concurrency

**Use channels for:** Communication between goroutines
**Use mutexes for:** Protecting shared state

- `sync.WaitGroup` for goroutine coordination
- `context.Context` for cancellation and timeouts
- `sync.Once` for one-time initialization
- Buffered channels for producer/consumer patterns

## Interface Design

**Small interfaces:** 1-3 methods maximum
**Accept interfaces:** Parameters should be interfaces
**Return structs:** Return concrete types

- Define interfaces at usage point
- Prefer composition over large interfaces
- Use `io.Reader`, `io.Writer` when possible

## Performance

- Use `strings.Builder` for string concatenation
- Pool expensive objects with `sync.Pool`
- Prefer value receivers for small structs
- Pointer receivers for large structs or mutations

## Dependencies

- `go mod tidy` before commits
- Pin major versions in `go.mod`
- Use standard library when possible
- Go 1.22+: Prefer `net/http` over third-party routers

## Formatting

- `gofmt` and `goimports` in CI
- Line length ~100 characters
- Group imports: stdlib, external, internal

## Forbidden

- No `init()` functions without clear necessity
- No global mutable state
- No empty error handling (handle or propagate)
- No goroutine leaks (always have exit strategy)
