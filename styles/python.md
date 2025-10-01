# Python Style Guide

## Version & Tooling

**Target runtime:** CPython 3.13+
**Project metadata:** `pyproject.toml` (PEP 621)

- Set `requires-python = ">=3.13"`
- Use `ruff check` and `ruff format` in CI
- Type check with `pyright` or `mypy --strict`
- Pin dependencies with hashes in `requirements.lock`
- Prefer `uv` or `pip` with `--no-deps` in reproducible workflows

## Module Layout

**Package exports:** Explicit via `__all__`
**Name style:** snake_case modules, CamelCase classes

- Keep `src/` layout with isolated virtual environment
- Group related functions in modules, avoid >500 lines
- Use relative imports only within the same package layer
- `__init__.py` exposes stable API surface

## Typing

**Type aliases:** `type Vector = list[float]`
**Generics:** PEP 695 syntax (`class Box[T]: ...`)

- Always annotate public APIs, dataclass fields, and comprehensions
- Prefer `Protocol`, `TypedDict`, `Self`, `@override`
- Use `typing.LiteralString` for SQL or shell inputs
- `Final`, `Literal`, and `TypeAliasType` capture invariants
- Avoid `Any`; fall back to `Unpack`, `Never`, `NotRequired`

## Functions & APIs

**Parameters:** positional-only for identifiers, keyword-only for flags
**Returns:** dataclasses or `NamedTuple` for structured data

- Default arguments must be immutable (`None`, `()`, `frozenset()`)
- Provide context managers instead of paired `open/close`
- Use `match` for multi-branch business logic
- Expose pure functions where possible; isolate side effects

## Async & Concurrency

**Primary runtime:** `asyncio` with `TaskGroup`
**Cancellation:** `asyncio.timeout()` and `Task.cancel()`

- `async with` for network and I/O clients
- Convert blocking calls via `anyio.to_thread.run_sync`
- Never ignore `CancelledError`; propagate after cleanup
- Await `gather()` with `return_exceptions=False`

## Data Modeling

**Preferred:** `@dataclass(slots=True, kw_only=True)`
**Enums:** `enum.StrEnum` for string constants

- Validate input using `pydantic` or `attrs` where strictness needed
- Use `functools.cached_property` for expensive computed fields
- Serialize via `msgspec`/`pydantic` with explicit schemas
- Pattern match on `dataclass` and `Enum` instances

## Error Handling

**Custom exceptions:** Derive from domain-specific base
**Context:** `raise ... from err`

- Use `ExceptionGroup`/`except*` for parallel failures
- Log via `structlog` with structured context
- Avoid blanket `except Exception`; handle specific subclasses
- Use `contextlib.suppress` sparingly with comment

## Testing

**Framework:** `pytest` with `pytest-asyncio`
**Coverage:** 90% line coverage minimum

- Use fixtures and `@pytest.mark.parametrize`
- Snapshot modern output using `syrupy` or `pytest-approvaltests`
- Test async code with `pytest.mark.asyncio` or `anyio` plugin
- Record property-based tests with `hypothesis`

## Performance

**Measurement:** `py-spy`, `perf`, `cProfile`
**Caching:** `functools.cache`/`lru_cache(maxsize=None)`

- Prefer vectorized operations via `numpy`/`polars`
- Replace `json` with `orjson` for hot paths
- Use `multiprocessing.shared_memory` for large data
- Guard SIMD-sensitive code with `if sys.platform` compatibility

## Forbidden

- No `from module import *`
- No bare `except` or silent pass
- No runtime `eval`/`exec`
- No mutable globals or module-level singletons
- No print-based debugging in committed code
