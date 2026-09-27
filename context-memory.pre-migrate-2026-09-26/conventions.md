# Global Conventions

## Commit Format

Conventional Commits v1.0.0:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:** `feat` · `fix` · `refactor` · `docs` · `test` · `chore` · `perf` · `ci`

- Subject line: imperative mood, max 72 chars, no period at end.
- Body: wrap at 100 chars. Explain *why*, not *what*.
- Breaking changes: footer `BREAKING CHANGE: <description>`.

**Examples:**
```
feat(auth): add OAuth2 PKCE flow
fix(db): prevent connection leak on query timeout
refactor(api): extract pagination logic to shared helper
```

## Branch Naming

```
<type>/<short-description>
```

**Examples:** `feat/oauth-login` · `fix/session-timeout` · `refactor/extract-repo-layer`

- Lowercase, hyphens only, no slashes beyond the type prefix.
- Keep short: ≤ 5 words after the prefix.

## PR Structure

**Title:** same format as commit subject line (max 72 chars).

**Body:**

```markdown
## What
[One paragraph: what changed and why.]

## How to test
- [ ] Step 1
- [ ] Step 2

## Breaking changes
None / [describe if any]
```

## CHANGELOG Format

Keep a Changelog (https://keepachangelog.com):

```markdown
## [version] — YYYY-MM-DD

### Added
### Changed
### Deprecated
### Removed
### Fixed
### Security
```

- Unreleased changes go under `## [Unreleased]`.
- One entry per logical change, written for a developer reader.
