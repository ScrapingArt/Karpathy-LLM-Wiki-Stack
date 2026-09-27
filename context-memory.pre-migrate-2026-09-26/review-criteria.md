# Adversarial Review Criteria

Use this checklist on every diff review. Report **all** findings. Do not suppress findings based on perceived low severity. Zero findings requires a mandatory re-analysis pass — this is almost always a sign of insufficient scrutiny.

## Security

- [ ] No secrets, tokens, or API keys hardcoded or in logs
- [ ] All inputs validated before use (type, range, format)
- [ ] Auth checks cannot be bypassed by request manipulation
- [ ] No PII written to logs or error messages
- [ ] SQL/shell/template injection paths checked
- [ ] File paths sanitized (no path traversal)

## Correctness

- [ ] Off-by-one errors in loops, slices, and pagination
- [ ] Null / undefined / zero value dereferences
- [ ] Error paths handled explicitly — no silent swallowing
- [ ] Concurrency hazards: shared mutable state, missing locks, TOCTOU
- [ ] Async/await correctness: missing await, unhandled rejections, race conditions
- [ ] Edge cases: empty input, maximum values, type coercion surprises

## Style Conformance

- [ ] Follows `~/.agents/memory/style.md` rules for the target language
- [ ] No dead code introduced
- [ ] No commented-out code committed
- [ ] Function length within limits; no accidental abstraction collapse

## Scope

- [ ] No changes outside the declared phase scope in `.context/PLAN.md`
- [ ] No unintended modifications to files outside the plan's scope list
- [ ] No accidental refactors bundled into the diff

## Test Coverage

- [ ] New logic paths have tests
- [ ] Error paths have tests
- [ ] No tests deleted without a documented reason

## Findings Format

Emit findings as a table:

| # | Severity | File:Line | Finding | Recommendation |
|---|----------|-----------|---------|----------------|
| 1 | critical | src/auth.ts:42 | Token logged in error handler | Remove log.error(token) |
| 2 | medium   | src/api.ts:89  | Missing await on async call | Add await or return the promise |

**Severity levels:** `critical` · `high` · `medium` · `low` · `nitpick`

After the table, state one of:
- `APPROVED` — all findings are nitpick-level or the diff has no issues.
- `NEEDS CHANGES` — one or more high/critical findings must be addressed before merge.
