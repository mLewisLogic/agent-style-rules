# TypeScript Style Guide

## Type Annotations

**Explicit for:** Public APIs, complex destructuring
**Infer for:** Simple assignments with clear types

## Interface vs Type

**Interface:** Domain models, props, state
**Type:** Unions, computed types, conditionals

## Strict Checking

```json
"strict": true
"noUncheckedIndexedAccess": true  
"exactOptionalPropertyTypes": true
```

## Best Practices

- Optional chaining `?.` and nullish coalescing `??`
- Prefer `null` over `undefined`
- Always async/await over raw promises
- Type-only imports: `import type {}`
- Named exports (default only for pages)

## Naming

- PascalCase: Components, types, interfaces
- camelCase: Variables, functions
- SCREAMING_SNAKE: Constants

## Forbidden

- No `any` (use `unknown`)
- No `@ts-ignore` (use `@ts-expect-error` with reason)
- No `!` without comment
- No synchronous file ops in async context
