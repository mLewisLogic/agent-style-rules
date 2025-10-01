# Rust Style Guide

## Error Handling

- Libraries: `thiserror` with structured variants
- Applications: `anyhow` with `.context()`
- No `unwrap()` in library code

## Async

- `tokio` exclusively
- `tokio::spawn` for concurrent tasks
- `tokio::try_join!` for fallible ops

## Module Organization

- Max 3 levels deep
- Re-export at crate root
- `pub(crate)` for internal APIs

## Conventions

- No `_` wildcard in enum matching
- Avoid `get_` prefix
- `is_`/`has_` for booleans
- Builder pattern for 4+ fields
- `Arc<T>` for cross-thread sharing

## Serde

- `#[serde(rename_all = "snake_case")]` for JSON
- `skip_serializing_if` for optionals

## Unsafe

- Document SAFETY invariants
- Minimal unsafe blocks
- Safe wrapper functions
