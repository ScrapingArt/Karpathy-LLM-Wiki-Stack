# Global Code Style

## Principles

- Explicit over implicit. No magic, no clever tricks.
- Functions do one thing. If "and" appears in the name, split it.
- No abbreviations in identifiers except established conventions: `id`, `url`, `http`, `cli`, `api`, `db`.
- Max function length: 40 lines. Extract if longer.
- Error handling: always explicit. Never silently swallow errors.
- No commented-out code in commits.
- No dead code — delete it.

## Naming

- Variables and functions: `camelCase` (JS/TS), `snake_case` (Python/Go/Shell), `PascalCase` (types/classes).
- Files: `kebab-case`.
- Constants: `SCREAMING_SNAKE_CASE`.
- Booleans: prefix with `is`, `has`, `can`, `should`.

## Structure

- Imports grouped: stdlib → third-party → internal. One blank line between groups.
- Exports at the bottom of the file, not inline.
- No circular dependencies.

## Tests

- Test the behavior, not the implementation.
- One assertion per test where possible. Name tests as: `[unit] [does what] [given what]`.
- All edge cases and error paths must have tests.

## Language-Specific Overrides

<!-- Add per-language rules below. These override or extend the principles above.
     Example:

### TypeScript
- Strict mode always on (`"strict": true`).
- Runtime validation via Zod for all external inputs.
- No `any`. Use `unknown` + type guard instead.
- Prefer `type` over `interface` for data shapes.

### Python
- Type hints on all public functions.
- `dataclass` or `pydantic` for data structures, not plain dicts.
- f-strings only; no `.format()` or `%`.
-->
