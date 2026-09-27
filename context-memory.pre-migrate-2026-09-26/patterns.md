# Cross-Project Patterns

Distilled from session archives across all registered projects.
Maintained by `scripts/distill.sh --apply`. Human-reviewed before promotion.

Do not edit the body below the section headers manually (this instruction is for human editors) —
use `scripts/distill.sh` to propose additions and promote them here after review.

**Runtime load path:** Agents load this file from `~/.agents/memory/patterns.md` directly.
The copy in `.context/memory/patterns.md` is not read at runtime — edits there are ignored.

---

## Logic Traps

Recurring operational gotchas. Each entry is sourced from at least one session archive.

- [grep-bsd-option-parsing]: `grep -qF "pattern"` silently misbehaves on macOS BSD grep when the pattern starts with `-` (treated as an option flag). Always use `grep -qF -- "pattern"` (POSIX `--` terminates option parsing). Applies to all scripts in this codebase. Source: Phase 1, registry implementation.

- [context-gitignored]: `.context/` is listed in `.gitignore`. Tool-level file reads (e.g. `read_file`) respect `.gitignore` and return empty or error. Always use shell tools (`cat`, `grep`, `awk`) to read any file under `.context/`. Source: multiple sessions.

- [build-sh-requires-absolute-paths]: `scripts/build.sh` must be invoked with absolute paths for `--project` and `--agents`. Relative paths embed the calling CWD into generated files (CLAUDE.md, GEMINI.md, etc.), producing machine-specific and worktree-specific paths. Always invoke as: `bash scripts/build.sh --platform all --project /abs/path --agents /abs/path`. Source: multiple sessions.

- [worktree-paths-in-generated-files]: Generated files (CLAUDE.md, GEMINI.md, copilot-instructions.md, agents.mdc) built inside a git worktree embed the worktree path, not the main repo path. Always rebuild from the main repo after merging a worktree branch. Source: worktree sessions.

- [find-project-root-consistency]: `find_project_root()` walks to the nearest `.git` directory. Any script that needs the project root (e.g. `agents unregister`) must use the same `find_project_root()` function as `run_init()` to guarantee the path matches what was registered. Do not substitute `pwd` or `$PWD`. Source: Phase 1 planning.

- [copy-default-skip-behavior]: `copy_default` in `scripts/init.sh` silently skips files that already exist in `~/.agents/memory/`. New memory file templates will not overwrite user-customized versions on re-init. `scripts/distill.sh` must not assume `~/.agents/memory/patterns.md` content matches the template — users may have edited it. Source: ADR-003, Phase 2 planning.

---

## Recurring Decisions

Architectural patterns seen across ≥2 projects. These are not ADRs (those are per-project) — they are stack-level conventions.

- [shell-no-external-deps]: Cross-repo scripts must work with POSIX shell tools only (`grep`, `sed`, `awk`, `sort`, `date`, `mktemp`). Do not require `jq`, `yq`, `python3`, or any language runtime. Registry parsing, session extraction, and sync output must all use pure bash + coreutils.

- [dry-run-default]: Scripts that write to global state (`~/.agents/memory/patterns.md`, Obsidian vault) must default to dry-run (stdout only). The `--apply` or equivalent flag is required to write. This prevents accidental mutation of shared state.

- [idempotent-writes]: Any script that writes to a shared file (registry, patterns.md, Obsidian notes) must be safe to run twice in a row with identical output. Check-before-write on append; full-overwrite on idempotent notes.

### Distilled 2026-03-19 (auto-added by distill.sh)
- [distilled]: [AGENTS] NEEDS CHANGES now ends the session; follow-up is "implement remediation for phase N"
- [distilled]: [AGENTS] {current_branch} must be re-read after git checkout -b (options a/b) before writing to PLAN.md — stale value would record the pre-checkout branch (e.g. main) instead of the new feature branch
- [distilled]: [AGENTS] HALT after Documenter routing in develop/step-04-adversarial.md is a contradictory instruction; build.sh must be run after any agent/workflow source change before committing GEMINI.md
- [distilled]: [AGENTS] Explorer Restrictions wording must use an explicit trigger condition ("only if CONTEXT.md Language field was blank at the start of this step") — vague "mode" references will be ignored by the agent; Manual path in step-01 must reference Dispatcher Check #4 by name rather than duplicating question list
- [distilled]: [AGENTS] Plan task said "write to conventions.md on confirm" but auto-detect has no signals for conventions; A6 correctly omits it — plan wording overstated scope; context_populated polarity is inverted (false=already-populated, true=just-populated) — future consumers must read as "did this step populate context?" not "is context populated?"
- [distilled]: [AGENTS] context_populated polarity is inverted — false=already-populated (skipped), true=just-populated (this step did the work); future step-02 consumers must read as "did this step populate?" not "is context populated?"
- [distilled]: [AGENTS] numbered sub-item inside a conditional is weaker than bold "Before writing:" precondition for agent compliance — use the latter for all archival instructions
- [distilled]: [AGENTS] step-04-adversarial.md still uses unsanitized [started-value] for session archive filename (medium, not in Phase 1 scope); Dispatcher Plan Creation archive missing tests_run/logic_traps/proof_of_work fields — non-compliant with new verify standard
- [distilled]: [AGENTS] find_project_root() walks to nearest .git — --workspace must bypass it entirely; single-quoted heredoc required for bash YAML templates; Dispatcher Plan Creation step 1.5 guard must be explicit to avoid prompting in non-workspace sessions
- [distilled]: [AGENTS] build.sh --agents . produces relative paths in CLAUDE.md; use --agents /abs/path to preserve absolute paths per ADR-001 convention
- [distilled]: [AGENTS] In worktree sessions, all git operations must target the main repo path explicitly; `git checkout -b` from worktree CWD creates branches in the worktree's git context, not the main repo — commit landed on main as intended but branch tracking was confused
- [distilled]: [AGENTS] step-02-format §3 "skip blocks 3 and 4" was ambiguous after renumbering (new §4 = Archive PR); fixed to explicit section skip with forward reference — PR archive must always run regardless of issue_body state
- [distilled]: [AGENTS] build.sh requires --project PATH (no default); invoke as `bash scripts/build.sh --platform all --project .` from repo root; build.sh resolves AGENTS_ROOT to absolute path — generated workflow paths are absolute and machine-specific (pre-existing in agent source files)
- [distilled]: [AGENTS] slug derivation at step-01 position 1.5 may prompt for slug before bug description is captured when no initial description provided — two consecutive prompts; step-02-bisect.md state_in documentation gap closed by adding it to Phase 1 scope
- [distilled]: [AGENTS] Generated files in worktree embed worktree paths (not main repo paths); rebuild required from main repo after merge with `scripts/build.sh --project /Users/ludovic/AGENTS`
- [distilled]: [AGENTS] read_file tool respects .gitignore which blocks access to .context/ — must use shell tools (cat/grep) to read project state
- [distilled]: [AGENTS] grep -qF "pattern" fails on macOS BSD grep when pattern starts with `-`; fixed with grep -qF -- "pattern" (POSIX -- terminates option parsing); applies to all future scripts in this repo
- [distilled]: [AGENTS] build-output-count-verification: use grep -c to verify propagation count equals number of agents (11), not just presence check
