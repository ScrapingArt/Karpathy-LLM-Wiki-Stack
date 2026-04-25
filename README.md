---
tags: [guide, obsidian, claude-code, karpathy, pkm, llm-wiki]
created: 2026-04-12
status: reference
version: 1.1
---

# Karpathy's LLM Wiki Stack — Complete Technical Blueprint

A comprehensive, build-ready reference for constructing a high-performance personal knowledge system using Obsidian and Claude Code, grounded in Andrej Karpathy's "LLM Wiki" pattern (April 2026).

> **How to use this document:** Read the Quick Start if you want a working system in 15 minutes. Read the full document to understand every architectural decision. Share this entire document with Claude Code and say: *"Set up an LLM Wiki for me based on this blueprint."*

---

## Table of Contents

1. [The Paradigm Shift](#1-the-paradigm-shift)
2. [The Three-Layer Architecture](#2-the-three-layer-architecture)
3. [Full Vault Directory Tree](#3-full-vault-directory-tree)
4. [CLAUDE.md — The Master Schema](#4-claudemd--the-master-schema)
5. [Core Operations: Ingest, Query, Lint](#5-core-operations-ingest-query-lint)
6. [Obsidian Setup and Plugins](#6-obsidian-setup-and-plugins)
7. [Claude Code Integration Strategies](#7-claude-code-integration-strategies)
8. [The Obsidian CLI Advantage](#8-the-obsidian-cli-advantage)
9. [Indexing and Logging](#9-indexing-and-logging)
10. [Frontmatter Schemas and Dataview](#10-frontmatter-schemas-and-dataview)
11. [Search at Scale: qmd](#11-search-at-scale-qmd)
12. [Custom Slash Commands](#12-custom-slash-commands)
13. [Advanced: Self-Evolving Loop](#13-advanced-self-evolving-loop)
14. [Vault Evolution & Migration](#14-vault-evolution--migration)
15. [Graph Architecture: Tree Shape Over Cluster Shape](#15-graph-architecture-tree-shape-over-cluster-shape)
16. [Use Cases and Configuration Variants](#16-use-cases-and-configuration-variants)
17. [Community Best Practices and Failure Modes](#17-community-best-practices-and-failure-modes)
18. [Quick Start: 15-Minute Build](#18-quick-start-15-minute-build)
19. [Known Repos and Starter Kits](#19-known-repos-and-starter-kits)
20. [Sources](#20-sources)

---

## 1. The Paradigm Shift

### From Karpathy's Gist (April 2026)

Karpathy introduced this pattern in a tweet and then published an *idea file* — a GitHub gist intentionally kept abstract so that any LLM agent can instantiate a version tailored to your exact setup and domain.

> "Most people's experience with LLMs and documents looks like RAG: you upload a collection of files, the LLM retrieves relevant chunks at query time, and generates an answer. This works, but the LLM is rediscovering knowledge from scratch on every question. There's no accumulation."
> — Andrej Karpathy, April 2026

His alternative: instead of retrieving from raw documents at query time, the LLM **incrementally builds and maintains a persistent wiki** — a structured, interlinked collection of `.md` files that sits between you and the raw sources.

> "The knowledge is compiled once and then *kept current*, not re-derived on every query."

The result: **"the wiki is a persistent, compounding artifact."** Cross-references pre-built. Contradictions already flagged. Synthesis reflecting everything you have ever read.

### The IDE Metaphor

- **Obsidian** is the IDE.
- **The LLM** is the programmer.
- **The wiki** is the codebase.

### Comparison: RAG vs. Human Notes vs. LLM Wiki

| Dimension | Traditional RAG | Human Note-Taking | LLM Wiki |
| :--- | :--- | :--- | :--- |
| **When knowledge is processed** | Query time (re-derived every question) | Manual write time | Ingest time (once per source) |
| **Cross-references** | Discovered ad-hoc per query | Manual, often abandoned | Pre-built and auto-maintained |
| **Contradictions** | May not be noticed | Rarely caught | Flagged during ingestion |
| **Knowledge accumulation** | None — resets each query | Grows but degrades | Compounds with every source |
| **Output format** | Ephemeral chat responses | Static markdown | Persistent, evolving markdown |
| **Who maintains it** | Black-box system | Human (burns out) | LLM (never bored) |
| **Retrieval method** | Vector similarity (embedding) | Human memory / search | Index file → targeted reads |
| **Examples** | NotebookLM, ChatGPT uploads | Obsidian vaults, Notion | This pattern |

### Why It Works (Karpathy's Words)

> "The tedious part of maintaining a knowledge base is not the reading or the thinking — it's the bookkeeping. Humans abandon wikis because the maintenance burden grows faster than the value. LLMs don't get bored, don't forget to update a cross-reference, and can touch 15 files in one pass."

### Division of Labour

| Human | LLM |
| :--- | :--- |
| Curate sources | Summarize and extract |
| Direct analysis | Cross-reference and file |
| Ask good questions | Maintain consistency |
| Think about meaning | Update on contradiction |
| Decide what to explore next | Suggest new investigations |

---

## 2. The Three-Layer Architecture

Karpathy defines three layers with strict ownership rules.

### Layer 1 — Raw Sources (`raw/`)

**Owner: Human (write). LLM: read-only. Rule: NEVER modify.**

Your curated collection of source documents — articles, papers, images, data files, transcripts, web clips, repo READMEs. These are **immutable**. This is your source of truth. If the LLM makes a mistake in the wiki, you can always trace back to the raw source and correct it.

**Accepted file types:** `.md`, `.pdf`, `.txt`, `.csv`, `.json`, `.png`, `.jpg`, `.mp3` (transcript), and anything else your LLM agent can read.

### Layer 2 — The Wiki (`wiki/`)

**Owner: LLM (write and maintain). Human: read and explore.**

A directory of LLM-generated markdown files. Summaries, entity pages, concept pages, comparisons, an overview, a synthesis. The LLM creates pages, updates them when new sources arrive, maintains cross-references, and keeps everything consistent.

**You read it. The LLM writes it.**

### Layer 3 — The Schema (`CLAUDE.md` / `AGENTS.md`)

**Owner: Human + LLM (co-evolved). Read by: LLM at every session start.**

A configuration document that tells the LLM how the wiki is structured, what conventions to follow, and what workflows to execute for each operation. This is what makes the LLM a **disciplined, consistent wiki maintainer** rather than a generic chatbot. Without it, every session resets to zero. With it, the LLM inherits your entire operational context instantly.

> "You and the LLM co-evolve this over time as you figure out what works for your domain." — Karpathy

**Note:** For OpenAI Codex, use `AGENTS.md`. For OpenCode/Pi, use `OPENCODE.md`. Content is identical — only the filename changes based on which agent reads it.

### The Routing Test: L1 vs L2

As you maintain your stack, you must decide whether a piece of knowledge belongs in the **L1 Memory directory** (Claude Code's `.claude/` or `~/.agents/memory/`) or the **L2 Wiki** (`wiki/`).

Apply the **Dangerous or Embarrassing Test**:
- **L1 (Auto-loaded):** "Would the LLM making a mistake without this knowledge be **dangerous or embarrassing**?" (e.g., security credentials, hard behavioral constraints, "never use standard markdown links").
- **L2 (On-demand):** "Would a mistake be merely **inconvenient**?" (e.g., a specific concept detail, a source summary).

If everything goes in the wiki, the LLM may already have made a critical mistake before it "remembers" to load the relevant wiki page. Keep L1 lean, but keep it protective.

---

## 3. Full Vault Directory Tree

```
knowledge-vault/                    ← Vault root, Claude Code working dir
│
├── CLAUDE.md                       ← Master schema (Layer 3). Read first.
├── VAULT-INDEX.md                  ← Live dashboard: open projects, status
│
├── raw/                            ← Layer 1. Immutable. Human drops files here.
│   ├── articles/                   ← Clipped web articles (via Obsidian Web Clipper)
│   ├── papers/                     ← PDFs, academic papers
│   ├── repos/                      ← README.md, architecture notes from code repos
│   ├── transcripts/                ← Meeting notes, podcast/video transcripts
│   ├── data/                       ← CSV, JSON, benchmark results
│   └── assets/                     ← Downloaded images (Obsidian attachment dir)
│
├── wiki/                           ← Layer 2. LLM-owned. Do not hand-edit.
│   ├── index.md                    ← Master catalog of all wiki pages (updated on every ingest)
│   ├── log.md                      ← Append-only chronological activity record
│   ├── hot.md                      ← Rolling session hot cache (~500 words, read silently at session start)
│   ├── overview.md                 ← High-level synthesis of the entire knowledge base
│   │
│   ├── sources/                    ← One summary page per ingested raw source
│   │   └── summary-{slug}.md
│   │
│   ├── entities/                   ← Pages for people, companies, products, organizations
│   │   └── {entity-name}.md
│   │
│   ├── concepts/                   ← Pages for ideas, methods, techniques, theories
│   │   └── {concept-name}.md
│   │
│   ├── comparisons/                ← Analytical comparison pages (filed from queries)
│   │   └── {comparison-slug}.md
│   │
│   └── syntheses/                  ← Longer essays/analyses filed from explorations
│       └── {synthesis-slug}.md
│
├── .claude/                        ← Claude Code config (NOT Obsidian vault content)
│   ├── settings.json               ← Agent settings, model prefs, max tokens
│   └── skills/                     ← Custom slash commands (one .md file per command)
│       ├── my-world.md
│       ├── today.md
│       ├── close.md
│       ├── trace.md
│       ├── ghost.md
│       └── recall.md
│
└── .obsidian/                      ← Obsidian config (auto-generated, ignored by LLM)
    ├── app.json                    ← Vault-level settings (ignore filters, etc.)
    └── plugins/                    ← Plugin configs
```

### Key `.obsidian/app.json` settings

```json
{
  "userIgnoreFilters": [
    "node_modules/",
    ".git/",
    ".claude/"
  ],
  "newFileLocation": "folder",
  "newFileFolderPath": "raw/articles",
  "attachmentFolderPath": "raw/assets"
}
```

> **Why ignore `.claude/`?** Skills, plans, and agent memory live there. They are LLM operational files — not human knowledge. Keeping them out of Obsidian prevents graph and search pollution.

---

## 4. CLAUDE.md — The Master Schema

This is the most important file in the vault. Paste this at the root as `CLAUDE.md`. Customize the `## Domain` section for your topic.

```markdown
# LLM Wiki — Master Schema

## Domain
[REPLACE WITH YOUR TOPIC, e.g.: "Machine Learning Research, 2024–2026"]

## Project Structure
- `raw/` — immutable source documents. NEVER modify any file in raw/.
- `wiki/` — LLM-generated wiki. You own this layer entirely.
- `wiki/index.md` — master catalog. Update on EVERY ingest.
- `wiki/log.md` — append-only activity log. Never delete entries.
- `wiki/overview.md` — high-level synthesis. Revise after major ingests.
- `CLAUDE.md` — this file. Re-read at the start of every session.
- `wiki/hot.md` — session hot cache (~500 words). Read silently at session start BEFORE responding.

## Page Conventions
Every wiki page MUST have YAML frontmatter. Use these schemas:

### Source Summary Pages (wiki/sources/)
---
type: source
title: "Article/Paper Title"
slug: summary-{slug}
source_file: raw/articles/{filename}.md
author: "Author Name"
date_published: YYYY-MM-DD
date_ingested: YYYY-MM-DD
key_claims: [claim1, claim2, claim3]
related: [[concept1]], [[concept2]]
confidence: high | medium | low
---

### Concept Pages (wiki/concepts/)
---
type: concept
title: "Concept Name"
aliases: [alt-name, abbreviation]
sources: [[[source1]], [[source2]]]
related: [[concept2]], [[entity1]]
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: high | medium | low
cluster: {cluster-name}
cluster_role: hub | member   # hub pages must have an "In this cluster" body section
---

### Entity Pages (wiki/entities/)
---
type: entity
entity_type: person | company | product | org
title: "Entity Name"
sources: [[source1]], [[source2]]
related: [[concept1]], [[entity2]]
created: YYYY-MM-DD
updated: YYYY-MM-DD
cluster: {cluster-name}
contradictions: []
open_questions: []
---

### Comparison Pages (wiki/comparisons/)
---
type: comparison
title: "Comparing X vs Y"
sources: [[source1]], [[source2]]
related: [[concept1]]  # Back-link upward to tier-3 member pages
filed_from_query: true
date: YYYY-MM-DD
---

### Synthesis Pages (wiki/syntheses/)
---
type: synthesis
title: "Synthesis Title"
sources: [[source1]], [[source2]]
related: [[concept1]]  # REQUIRED: Back-link upward to tier-3 member pages
filed_from_query: true
date: YYYY-MM-DD
---

## Ingest Workflow
When I say "ingest [filename]" or "ingest raw/[path]":
1. Read the source file from raw/.
2. Discuss key takeaways with me (3–5 bullet points).
3. Create wiki/sources/summary-{slug}.md with full summary.
4. Update wiki/index.md — add new page under its cluster section.
5. Update ALL relevant concept and entity pages with new info.
6. If new info contradicts an existing page, flag it explicitly using a > [!contradiction] callout block.
7. Create new concept/entity pages if the source introduces them.
   - Assign each new page a `cluster:` field.
   - If the page is a hub (`cluster_role: hub`), add an `## In this cluster` body section listing members.
   - If the page is a member, link it from its cluster hub's `## In this cluster` table.
   - If a new cluster is needed, add it to wiki/index.md and wiki/overview.md.
8. Append a structured entry to wiki/log.md (see Log Format below).
9. A single ingest should touch 5–15 wiki pages.

## Query Workflow
When I ask a question:
1. Read wiki/index.md to identify relevant pages.
2. Read those pages directly.
3. Synthesize an answer with [[wiki-link]] citations.
4. If the answer is a valuable analysis, offer to file it as a new
   page in wiki/comparisons/ or wiki/syntheses/.
   - If a synthesis is filed, add a `see also: [[synthesis-slug]]` back-link to the original concept pages it drew from.
5. Update wiki/log.md with a query entry.

## Lint Workflow
When I say "lint" or "health check":
1. Scan for contradictions between pages. List them.
2. Find orphan pages (0 inbound links). List them.
3. List concepts mentioned 3+ times but lacking their own page.
4. Check for stale claims that newer sources may have superseded.
5. **Cluster health check:**
   - Every concept/entity page has a `cluster:` field. List any missing.
   - Every `cluster_role: hub` page has an `## In this cluster` section. List any missing.
   - Every cluster hub is listed in wiki/index.md under its cluster section. List gaps.
6. **Synthesis back-link check:**
   - Every synthesis page is back-linked from the concept pages it drew on. List near-orphan syntheses (≤1 inbound link).
7. Suggest 3–5 new questions or sources to investigate.
8. Append a lint entry to wiki/log.md.

## Log Format
Each log entry MUST start with this prefix for parsability:
## [YYYY-MM-DD] {ingest|query|lint} | {title/description}

Example:
## [2026-04-12] ingest | Mixture of Experts Efficiency Study
Source: raw/articles/2026-04-moe-efficiency.md
Pages created: wiki/sources/summary-moe-efficiency.md
Pages updated: wiki/concepts/mixture-of-experts.md,
               wiki/concepts/scaling-laws.md
Contradictions flagged: wiki/concepts/dense-vs-sparse.md (see note)

## Safety Rules
- NEVER write to raw/. This is a hard constraint with no exceptions.
- NEVER delete wiki pages. Mark as deprecated in frontmatter instead.
- Always update wiki/index.md and wiki/log.md on every operation.
- When uncertain about a claim's accuracy, set confidence: low.
- Cross-reference all new pages to at least 2 existing pages.
```

---

## 5. Core Operations: Ingest, Query, Lint

These are the three operations Karpathy defines. Each is a complete, repeatable workflow.

### The 10-Source Test for Query Readiness

A common mistake is querying the wiki too early. A wiki with 90% coverage can be **actively misleading** because the LLM has high-confidence gaps — it doesn't know what it doesn't know. 

**Rule:** Do not rely on the wiki for decision-making until you have reached meaningful coverage (the "10-Source Test"). Validate that the wiki yields cross-source insights before trusting its syntheses. *Empirical finding:* A 90%-finished wiki performed 17% worse than a complete one in production testing due to high-confidence gaps.

### The Single-Writer Rule (Concurrency)

LLM agents can suffer from silent data corruption (lost content, corrupted characters) when multiple sessions or agents write to the same wiki file simultaneously.

**Rule:** Treat wiki files as a shared resource. Default to single-agent sequential writes. If you must run parallel sessions, ensure they are operating on disjoint sets of files.

### Operation 1: Ingest

**Trigger:** You drop a file into `raw/` and tell Claude to process it.

> "A single source might touch 10–15 wiki pages." — Karpathy

**Exact prompt to use:**
```
Ingest raw/articles/2026-04-my-article.md
```

**What the LLM should do (defined in CLAUDE.md):**
1. Read the source.
2. Discuss 3–5 key takeaways with you.
3. Create `wiki/sources/summary-{slug}.md`.
4. Update `wiki/index.md`.
5. Update all relevant concept and entity pages.
6. Flag contradictions explicitly using a > [!contradiction] callout block.
7. Append to `wiki/log.md`.

**Karpathy's preferred style:** ingest one source at a time and stay involved. Read the summaries, check updates, guide emphasis. Batch ingestion is possible but produces lower-quality cross-referencing.

---

### Operation 2: Query

**Trigger:** You ask a question.

**Example prompt:**
```
What do we know about mixture-of-experts routing strategies
and their efficiency tradeoffs? Cite wiki pages.
```

**The compounding insight (from Karpathy):**
> "Good answers can be filed back into the wiki as new pages. A comparison you asked for, an analysis, a connection you discovered — these are valuable and shouldn't disappear into chat history."

**Follow-up to file an answer:**
```
File this analysis as a wiki comparison page.
```

This is the compounding loop: external sources enrich the wiki on ingest; your own explorations enrich it on query. **The wiki grows from both directions.**

---

### Operation 3: Lint

**Trigger:** Periodic health check (recommend: weekly).

**Exact prompt:**
```
Lint the wiki.
```

**What the LLM should produce:**
- List of contradictions between pages (with specific files and claims).
- Orphan pages (no inbound links — candidates for linking or deletion).
- Concepts mentioned 3+ times without their own page.
- Stale claims from older sources not yet updated.
- 3–5 suggested investigations or new sources.

**Example lint output format:**
```
Wiki Health Report — 2026-04-12

CONTRADICTIONS (2)
- concepts/dense-vs-sparse.md: claims dense > sparse below 10B params
  BUT sources/summary-moe-efficiency.md shows opposite.
  Recommendation: update with nuance from new source.

ORPHAN PAGES (3)
- concepts/tokenization.md — no inbound links
- sources/summary-old-bert.md — no references in concepts or entities

MISSING PAGES (4)
- "RLHF" mentioned 14 times → no concept page
- "KV Cache" mentioned 7 times → no concept page

SUGGESTED INVESTIGATIONS
- No sources on inference optimization post-2025
- Entity page for Meta AI is thin (1 source only)
```

---

## 6. Obsidian Setup and Plugins

### Vault Configuration Steps
1. Install Obsidian from [obsidian.md](https://obsidian.md).
2. Open `knowledge-vault/` as a new vault (not as a folder on an existing vault).
3. Go to Settings → Files and links:
   - Set **Default location for new notes** → `raw/articles`
   - Set **Attachment folder path** → `raw/assets`
4. Go to Settings → Hotkeys → Search "Download" → Bind `Ctrl+Shift+D` to **"Download attachments for current file"**. This downloads all remote images in a clipped article to `raw/assets/`.

> **Why download images locally?** Remote image URLs break over time. Local images let the LLM view and reference them directly. Note: LLMs can't read inline markdown images in one pass — the workaround is to have the LLM read text first, then view referenced images separately.

### Essential Plugins

| Plugin | Type | Purpose |
| :--- | :--- | :--- |
| **Obsidian Web Clipper** | Browser extension | Clip articles to markdown, save to `raw/articles/` |
| **Dataview** | Core plugin | Query frontmatter as a database |
| **Templater** | Core plugin | Enforce consistent frontmatter on new pages |
| **Graph View** (built-in) | Core | Visualize wiki topology; identify hubs and orphans |
| **Marp Slides** | Optional | Generate slide decks from wiki content |
| **File Explorer++** | Optional | Better folder management |
| **Smart Connections** | Optional | Semantic similarity sidebar |
| **Folder Note** | Optional | `index.md` per folder as a navigable root |

### Obsidian Web Clipper Tips
- Configure clip destination to `raw/articles/`.
- Set up a template with frontmatter: `source_url`, `author`, `date_clipped`.
- After clipping, immediately hit `Ctrl+Shift+D` to download images before ingesting.

### Graph View Usage
Use graph view after ingests to visually inspect:
- **Hub nodes** (large, many connections) = central concepts working correctly.
- **Orphan nodes** (isolated) = pages that need cross-references or can be removed.
- **Clusters** = emergent themes in your knowledge base.

---

## 7. Claude Code Integration Strategies

Five distinct patterns exist in the community. Choose based on your use case.

| Strategy | Setup | Pros | Cons | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **1. Vault as Working Dir** | Open Claude Code directly in vault root | Simplest setup; CLAUDE.md auto-loaded | Wiki and dev files in same space | Personal knowledge base (recommended default) |
| **2. Symlinked Docs** | `ln -s ~/vault/wiki ./docs` from a project repo | Unified search across code + notes | Mobile sync issues; recursive indexing risk | Developer documentation wikis |
| **3. MCP Bridge** | `obsidian-claude-code-mcp` plugin, WebSocket port 22360 | Clean repo separation; remote vault access | More complex setup; extra dependency | Teams; vault on different machine |
| **4. One Vault Per Repo** | Separate vault per project | Simple; total isolation | No cross-project knowledge search | Isolated project wikis |
| **5. QMD + Session Sync** | qmd MCP server + `/recall` skill | 60%+ token reduction; semantic pre-load | Requires qmd install and index build | Large vaults (200+ sources) |

> **Recommended for most users:** Strategy 1 (Vault as Working Dir). Point Claude Code at the vault root. `CLAUDE.md` is auto-detected and loaded. Start with this and graduate to Strategy 5 when the vault grows large enough to hit context limits.

> **Symlink warning:** Symlinks can cause recursive indexing loops in Obsidian's sync engine and break mobile clients. If you use them, exclude symlinked directories from Obsidian's file watcher. Strategy 3 (MCP Bridge) is cleaner for cross-repo access.

### Mandatory Behavioral Layer: The Karpathy Skills Plugin

Regardless of the connection strategy, every LLM Wiki maintainer should install the behavioral guardrails from Karpathy's own observations on LLM coding pitfalls. This ensures the agent acts with the surgical discipline of a senior engineer.

**Install via Claude Code:**
```bash
claude plugin marketplace add forrestchang/andrej-karpathy-skills
claude plugin install andrej-karpathy-skills@karpathy-skills
```

This plugin enforces four core principles globally: **Think Before Coding**, **Simplicity First**, **Surgical Changes**, and **Goal-Driven Execution**.

---

## 8. The Obsidian CLI Advantage

Obsidian 1.12 introduced a native CLI that provides programmatic access to Obsidian's internal caching database — bypassing OS-level filesystem searches entirely.

### Benchmarks: Methodology & Provenance

The following performance metrics are derived from community testing and typical tokenization patterns:

- **54× CLI Speedup:** Benchmarked by [@drrobcincotta] on a 4,663-file, 16 GB research vault. Compares native Obsidian SQLite cache access vs. standard `grep` / `glob` filesystem scans.
- **3× Token Efficiency (Defuddle):** A "Chrome-to-Content Ratio" based on typical web articles (2k–6k tokens) vs. clean markdown content (400–1,200 tokens).
- **60%+ Token Reduction (qmd):** Reported by Kevin Lee and ArtemXTech. Compares semantic pre-loading (reading only the top 5 relevant pages) vs. naive whole-vault indexing or unranked keyword search.

---

### Plugin Stability & The Filesystem Fallback

Third-party plugins (`qmd`, `obsidian-skills`, `cli-patch`) are performance accelerators, not core requirements. To ensure vault stability across tool updates:

**The Fallback Rule:** If any plugin, MCP server, or CLI command returns an error, returns empty data silently, or hangs for >10 seconds, the agent MUST fallback to standard Unix tools:
- **Search Fallback:** Use `ripgrep` (`rg`) or `grep -r` for keyword discovery.
- **Read Fallback:** Use `cat` or `read_file` for direct filesystem access.
- **Write Fallback:** Use `printf >>` or `write_file` for atomic append/write.

*Never allow the failure of a performance tool to halt the ingestion or query pipeline.*

---

### Knowledge Layers: Behavioral L1 vs Wiki L2

Knowledge in this stack is stored across two distinct layers:

1. **L1 (Behavioral Guardrails):** Stored in \`CLAUDE.md\` or as global Claude Code plugins. This is "how the agent works." These rules are auto-loaded and enforced on every session. Use L1 for surgical maintenance rules, safety protocols, and formatting constraints.
2. **L2 (Domain Wiki):** Stored in \`wiki/\`. This is "what the agent knows." These pages are loaded on-demand via targeted search or \`qmd\`. Use L2 for source summaries, concepts, and syntheses.

**The Test:** If a piece of knowledge is needed *before* the agent starts its first operation (e.g., "always use surgical updates"), it belongs in **L1**. If it's needed *during* an operation (e.g., "what is MoE?"), it belongs in **L2**.

### Step A — Install Kepano's Official Obsidian Skills (Do This First)

Before installing the CLI patch, install the official skills from Obsidian's CEO (Steph Ango / Kepano). Released January 2026, this is a **44-skill repository** that teaches Claude the correct proprietary syntax for every Obsidian file type. Without it, Claude will silently break files:

| Without `obsidian-skills` | With `obsidian-skills` |
| :--- | :--- |
| Creates wikilinks in wrong format, breaks graph view | Correct `[[Note Title]]` wikilink syntax enforced |
| Generates invalid JSON for `.base` files | Correct Bases schema with valid views, filters, formulas |
| Writes `.canvas` files that won't open | Correct JSON Canvas spatial format |
| Guesses frontmatter format | Enforces Obsidian's exact properties syntax |

**Install via Claude Code marketplace:**
```bash
claude plugin marketplace add kepano/obsidian-skills
```

**Or install manually (recommended — keeps skills in your vault for offline use):**
```bash
# From your vault root
git clone https://github.com/kepano/obsidian-skills.git /tmp/obsidian-skills
cp -r /tmp/obsidian-skills/.claude/* .claude/
```

**The three core skills included:**

- **`obsidian-markdown`** — Rules for `.md` files: correct wikilink syntax, frontmatter/properties format, callout syntax, embed syntax, tag formatting, and when to use wikilinks vs standard markdown links.
- **`obsidian-bases`** — Rules for `.base` files (Obsidian's database layer): view types, filter expressions, formula syntax, summary config. Without this, Claude generates invalid JSON that Obsidian rejects silently.
- **`json-canvas`** — Rules for `.canvas` spatial map files: node types (`text`, `file`, `link`, `group`), edge config, position schema. Prevents Claude from writing canvas files that crash on open.
- **`defuddle`** — Web page cleaning skill (see Section 8c below for full coverage).

> **Why this matters for the LLM Wiki specifically:** Every wiki page you build depends on correct wikilink cross-references. If Claude uses `[Concept Name](wiki/concepts/concept-name.md)` instead of `[[Concept Name]]`, your entire graph topology breaks silently — links appear correct in markdown but Obsidian's graph and backlink engine cannot parse them.

### Step B — Install the CLI Silent-Failure Patch (Mandatory for Automation)

The Obsidian CLI 1.12 has **13 documented silent failures** — commands that exit with code `0` (success) but return empty or wrong data. This is the most dangerous class of bug for an automated wiki agent: Claude believes the command worked, receives no error, and proceeds with incorrect or missing information.

**The 13 Silent Failures (from 57-scenario test suite, Obsidian Forum Feb 2026):**

| Broken Command | What Happens | Correct Command |
| :--- | :--- | :--- |
| `tasks todo` | 0 results (defaults to "active file" scope; no active file in CLI) | `tasks all todo` |
| `tasks done` | 0 results (same scoping bug) | `tasks all done` |
| `tags counts` | "No tags found" (same active-file default) | `tags all counts` |
| `properties format=json` | Returns YAML despite the flag | `properties format=tsv` |
| `create name="x" content="y"` | Opens Obsidian GUI instead of creating silently | Add `silent` flag |
| `search query="x"` (no vault flag) | Searches wrong vault | Always specify `vault="My Vault"` |
| `read file="x"` (ambiguous name) | Returns first match silently | Use full path `file="folder/x.md"` |
| `backlinks file="x"` | Returns partial results at large vault size | Use `--limit 0` to disable cap |
| `daily:read` (no date) | Returns yesterday's note | Add explicit `date=today` |
| `append file="x"` (no newline) | Corrupts last line of file | Always prefix content with `
` |
| `template` (template has variables) | Silently omits unfilled variables | Use `--fill-vars` flag |
| `move` across vault boundaries | Exits 0 but leaves duplicate | Verify with `read` after every move |
| `property:set` on locked file | Exits 0, write silently dropped | Check `property:get` to confirm |

**Install the patch:**
```bash
# Claude Code (auto-updates)
claude plugin marketplace add jackal092927/obsidian-official-cli-skills

# Any agent (Claude Code, Codex, Cursor, 20+ others)
npx skills add jackal092927/obsidian-official-cli-skills
```

**What the patch provides:**
- Correct syntax for all 34 CLI commands.
- Explicit warnings for all 13 silent failure patterns embedded as skill rules.
- Safety rules (e.g., never run `eval`, always add `silent` to `create`).
- Error-handling patterns (since exit codes cannot be trusted for validation).

> **Critical:** Do not rely on exit codes to verify CLI operations. Always use a read-back command (`obsidian read file="..."` or `obsidian property:get`) to confirm that writes took effect.

### Three Connection Methods (Ranked)

1. **Obsidian CLI** (recommended) — fastest, uses Obsidian's internal cache.
2. **REST API** — good for remote vaults, requires Obsidian to be running.
3. **Filesystem (grep/glob)** — fallback; works without Obsidian running but slow at scale.
### 8c. The Defuddle Ingestion Pre-processor

Defuddle is an open-source library created by Kepano specifically for Obsidian Web Clipper. It strips web page chrome — ads, navigation bars, sidebars, comment sections, cookie banners, and footers — before content reaches your vault, leaving only the primary article content as clean markdown.

**Why it matters: 3× Token Efficiency**

A raw web clip of a typical article contains 2,000–6,000 tokens of noise (navigation, related articles, social buttons, cookie notices). Defuddle reduces this to 400–1,200 tokens of clean content — a **3× efficiency gain**. At 10 ingests per week, that is 15,000–45,000 tokens saved weekly — roughly the equivalent of 10–30 extra article ingests per week at no extra cost.

**Install:**
```bash
npm install -g @kepano/defuddle
```

**CLI usage (pipe into vault):**
```bash
# Clean a URL directly to markdown
defuddle https://example.com/article --md > raw/articles/article-clean.md

# With metadata extraction
defuddle https://example.com/article --md --json | jq '{title, author, date, contentMarkdown}'

# Batch clean all files in a staging folder
for url in $(cat raw/staging/urls.txt); do
  slug=$(echo $url | sed 's/[^a-zA-Z0-9]/-/g' | cut -c1-40)
  defuddle "$url" --md > raw/articles/$slug.md
done
```

**Key Defuddle flags:**

| Flag | Effect |
| :--- | :--- |
| `--md` | Output as Markdown (default is HTML) |
| `--json` | Output JSON with both HTML and `contentMarkdown` fields |
| `-p title` | Extract only the title metadata |
| `-p title,author,date` | Extract multiple metadata fields |
| `--removeExactSelectors` | Remove elements matching ad/social button selectors (default: true) |
| `--removePartialSelectors` | Remove partial selector matches (default: true) |
| `--debug` | Show what was removed and why |

**Install as a Claude Code Skill (`/defuddle`):**

Save as `.claude/skills/defuddle.md`:
```markdown
# /defuddle

## Purpose
Strip a web URL to clean markdown before saving to raw/articles/.
Use instead of WebFetch for any article, blog post, or documentation page.
WebFetch returns raw HTML with full page chrome — always use /defuddle instead.

## Workflow
1. Take the URL as input.
2. Run: `defuddle {url} --md --json`
3. Parse the JSON: extract `title`, `author`, `date`, `domain`, `contentMarkdown`.
4. Write the clean markdown to raw/articles/{slug}.md with frontmatter:
   ---
   source_url: {url}
   title: "{title}"
   author: "{author}"
   date_clipped: {today}
   date_published: {date}
   domain: {domain}
   ---
5. Confirm the file was written and report word count saved vs. raw HTML.

## When to Use
Always use /defuddle as the first step before any ingest from a web URL.
```

**Integrated Web-to-Wiki Funnel (with Defuddle):**

```
Web URL
  │
  ▼
/defuddle → strips chrome, returns clean markdown + metadata
  │
  ▼
raw/articles/{slug}.md  (clean, token-efficient, with frontmatter)
  │
  ▼
ingest raw/articles/{slug}.md  (LLM reads clean content, not 5,000 tokens of nav)
  │
  ▼
wiki/sources/, wiki/concepts/, wiki/entities/ — updated
```

> **Defuddle vs. Obsidian Web Clipper:** Use Obsidian Web Clipper for clipping from your browser manually while browsing. Use the Defuddle CLI (or `/defuddle` skill) when directing Claude Code to fetch and ingest a URL automatically — it runs headlessly without needing a browser open. The two complement each other perfectly.



---

## 9. Indexing and Logging

Two special files are critical infrastructure for how the LLM navigates the wiki. Both are defined by Karpathy in the gist.

### `wiki/index.md` — The Content Catalog

**Purpose:** Replace RAG embedding search with a human-readable, LLM-readable catalog. At moderate scale (~100 sources, ~hundreds of pages), a well-maintained index works better than a vector database — no embedding infrastructure required.

**Structure:** Cluster-organized, not a flat list. Navigation flows `index → overview → cluster hub → member page` — a tree, not a catalogue dump.

```markdown
# Wiki Index — [Domain Name]

**Navigation:** Start here → [[overview]] (cluster map) → hub concept → member pages → syntheses.

---

## Cluster: {cluster-name}
> One-line description of what this cluster covers.

**Hub:** [[hub-concept]] — one-line description (N sources)

| Page | Summary |
| :--- | :--- |
| [[member-concept-1]] | description (N sources) |
| [[member-concept-2]] | description (N sources) |

**People in this cluster:** [[entity-1]] · [[entity-2]]
**Syntheses:** [[synthesis-1]] · [[synthesis-2]]

---

## Cluster: {another-cluster}
> ...

---

## Syntheses
- [[synthesis-slug]] — YYYY-MM-DD — description (N sources)

## Source Summaries
- [[summary-slug]] — YYYY-MM-DD — description

## Navigation Files
- [[overview]] — cluster navigation hub; start here for orientation
- [[hot]] — rolling session context; open questions; recent operations
- [[log]] — append-only activity record
```

**Rule:** The LLM updates this file on EVERY ingest, adding new pages under their cluster section. It reads it first on EVERY query to identify relevant pages.

### `wiki/overview.md` — The Cluster Navigation Hub

**Purpose:** Tier-1 navigation page. Humans and LLM agents read this to understand the vault's shape before drilling into a topic. Describes each cluster in 2–3 sentences and links to its hub concept. Updated after major corpus changes.

```markdown
# Wiki Overview — [Domain Name]

> Cluster navigation hub. Read this before diving into any topic.
> For the full page listing, see [[index]].

---

## What This Wiki Is

[1–2 sentence domain description and current state: N sources, M pages, K clusters.]

---

## Cluster Map

[ASCII or text diagram showing cluster hierarchy and navigation paths]

---

## Cluster: {cluster-name}

**Enter at:** [[hub-concept]]

[2–3 sentences: what this cluster covers, when to enter it, key questions it answers.]

**Key entry points:**
- "Question 1?" → [[page-1]]
- "Question 2?" → [[page-2]]

**Synthesis:** [[synthesis-slug]] — one-line description

---

## Graph Health

**Last analysis:** [[graph-structure-analysis-YYYY-MM-DD]] (N pages)

[One sentence on current graph shape and any known structural debt.]
```

**Rule:** `wiki/overview.md` must be linked from `wiki/index.md`. It is a navigation page — it should never be an orphan.

### `wiki/log.md` — The Activity Timeline

**Purpose:** Chronological record of all operations. Gives the LLM context about what has been done recently when starting a new session. Parseable with standard Unix tools.

**New Convention: The Failure Log**
The system compounds knowledge best when it also tracks failure. Use the `!failure` tag in log entries to record:
- Failed queries (no relevant information found).
- Ingest errors (source too noisy, formatting broken).
- Confirmed hallucinations found during lint.
This ensures future sessions don't repeat the same investigative dead-ends.

```markdown
# Activity Log

## [2026-04-12] ingest | MoE Efficiency Study
Source: raw/articles/2026-04-moe-efficiency.md
Pages created: wiki/sources/summary-moe-efficiency.md
Pages updated: wiki/concepts/mixture-of-experts.md, wiki/concepts/scaling-laws.md
Contradictions flagged: wiki/concepts/dense-vs-sparse.md

## [2026-04-12] query | MoE Routing Comparison
Question: Compare routing strategies across MoE sources
Pages read: concepts/mixture-of-experts.md, 3 source summaries
Output filed: wiki/comparisons/moe-routing-strategies.md

## [2026-04-10] lint | Weekly Health Check
Contradictions found: 2 | Orphans: 3 | Missing pages suggested: 4
```

**Karpathy's tip:** The `## [YYYY-MM-DD]` prefix makes every entry greppable:
```bash
grep "^## \[" wiki/log.md | tail -5
```
---

## 9b. The Hot Cache Pattern (`wiki/hot.md`)

The "hot cache" solves the **recap problem**: at the start of each new session, Claude has no memory of what was discussed previously. Without a warm-up mechanism, users spend their first 10–15 exchanges recapping context — wasting tokens and degrading momentum.

### What It Is

`wiki/hot.md` is a **~500-word rolling session history file** that Claude reads silently at every session start, before any other operation. It is not a journal (too long) and not a task list (too narrow). It is a compressed, structured snapshot of recent context: decisions made, concepts currently under investigation, open questions, and the last few ingest/query actions.

### Add to Your CLAUDE.md

Add this block to your master schema in `CLAUDE.md`:

```markdown
## Hot Cache (`wiki/hot.md`)
Read `wiki/hot.md` silently at the start of EVERY session, before responding.
This file contains ~500 words of recent session context. Do not summarize it
to the user — just use it to restore your operating context.

After EVERY session (or when the user says /close), update wiki/hot.md:
- Keep total length under 500 words.
- Overwrite (do not append).
- Structure:

### Current Focus
[1–2 sentences: what we are actively investigating right now]

### Open Questions
[Bullet list of unresolved questions or next ingests to do]

### Recent Decisions
[Bullet list: key decisions or conclusions from the last 1–2 sessions]

### Last Operations
[3–5 lines from wiki/log.md — the most recent ingest/query/lint entries]

### Active Pages
[List of wiki pages currently being developed or recently updated]
```

### Example `wiki/hot.md`

```markdown
---
type: hot-cache
updated: 2026-04-12
---

### Current Focus
Deep-diving MoE routing strategies. Comparing top-k, expert-choice,
and hash routing across efficiency benchmarks from 4 recent papers.

### Open Questions
- Does expert-choice routing degrade at inference time? No source yet.
- Check if Anthropic's MoE paper from Q1 2026 is in raw/ yet.

### Recent Decisions
- Filed moe-routing-strategies.md as a comparison page (2026-04-12)
- Confidence on dense-vs-sparse.md demoted to "low" — contradiction flagged
- Scaling laws section needs update from 2026 Chinchilla revision

### Last Operations
[2026-04-12] ingest | MoE Efficiency Study
[2026-04-12] query | MoE Routing Comparison → filed as comparison page
[2026-04-10] lint | Weekly Health Check

### Active Pages
- wiki/concepts/mixture-of-experts.md (under revision)
- wiki/comparisons/moe-routing-strategies.md (newly filed)
- wiki/concepts/dense-vs-sparse.md (contradiction flagged, needs fix)
```

> **Token cost:** A 500-word hot cache costs roughly 650–700 tokens per session to read. This replaces 2,000–4,000 tokens of recap conversation. Net saving: ~3× per session.

> **Rule:** `wiki/hot.md` is the **only** wiki file the LLM writes without being explicitly asked. All other writes are triggered by ingest, query, or lint operations.



---

## 10. Frontmatter Schemas and Dataview

### Complete Frontmatter Schema Set

**Source Summary (`wiki/sources/`)**
```yaml
---
type: source
title: "Paper: Efficient MoE Routing — 2026"
slug: summary-moe-routing-2026
source_file: raw/papers/moe-routing-2026.pdf
author: "Jane Smith et al."
date_published: 2026-03-15
date_ingested: 2026-04-12
key_claims:
  - Expert-choice routing achieves 3.4x throughput at same quality
  - Dense models remain superior below 3B parameters
related: [[mixture-of-experts]], [[scaling-laws]]
confidence: high
---
```

**Concept Page (`wiki/concepts/`)**
```yaml
---
type: concept
title: "Mixture of Experts"
aliases: [MoE, sparse mixture]
sources:
  - [[summary-moe-efficiency-2026]]
  - [[summary-switch-transformer]]
related:
  - [[scaling-laws]]
  - [[attention-mechanism]]
  - [[google-deepmind]]
created: 2026-03-20
updated: 2026-04-12
confidence: high
cluster: implementation
contradictions: []      # List of [[source-link]] showing conflicting info
open_questions: []      # Bullet list of unresolved gaps
---
```

**Entity Page (`wiki/entities/`)**
```yaml
---
type: entity
entity_type: company
title: "Anthropic"
sources:
  - [[summary-claude-3-paper]]
  - [[summary-constitutional-ai]]
related:
  - [[openai]]
  - [[claude-series]]
  - [[constitutional-ai]]
created: 2026-03-01
updated: 2026-04-10
cluster: core-pattern
---
```

**Comparison/Synthesis (`wiki/comparisons/`, `wiki/syntheses/`)**
```yaml
---
type: synthesis
title: "MoE Routing Strategies: Top-K vs Expert-Choice vs Hash"
sources:
  - [[summary-moe-efficiency-2026]]
  - [[summary-switch-transformer]]
  - [[summary-gshard-paper]]
related:
  - [[mixture-of-experts]]  # REQUIRED: Back-link upward to tier-3 member pages
filed_from_query: true
date: 2026-04-12
---
```

### Dataview Queries

**Dashboard: All concepts by source depth**
```dataview
TABLE length(sources) AS "Sources", confidence, updated
FROM "wiki/concepts"
SORT length(sources) DESC
```

**Find low-confidence pages needing review**
```dataview
TABLE title, sources, updated
FROM "wiki"
WHERE confidence = "low"
SORT updated ASC
```

**Recently updated pages (last 7 days)**
```dataview
LIST
FROM "wiki"
WHERE updated >= date(today) - dur(7 days)
SORT updated DESC
```

**All comparison and synthesis pages**
```dataview
TABLE date, sources
FROM "wiki/comparisons" OR "wiki/syntheses"
SORT date DESC
```

---

## 11. Search at Scale: qmd

At small scale (~50 sources), `wiki/index.md` is sufficient. As the wiki grows, the index itself becomes too large to read in one context window. This is when you add `qmd`.

### What qmd Is

`qmd` was built by Tobi Lütke (CEO of Shopify). It is a local, on-device search engine for markdown files combining three strategies:

- **BM25 full-text search** — keyword precision.
- **Vector semantic search** — finds conceptually related pages even without keyword match.
- **LLM re-ranking** — highest quality; LLM scores results for relevance.

All models run locally via `node-llama-cpp` with GGUF models. No data leaves your machine.

### Install and Configure

```bash
# Install globally
npm install -g @tobilu/qmd

# Add your wiki as a named collection
qmd collection add ./wiki --name my-research

# Rebuild the index after ingests
qmd index rebuild my-research
```

### Usage Patterns

```bash
# BM25 keyword search
qmd search "mixture of experts routing"

# Semantic search (finds related concepts)
qmd vsearch "how do sparse models handle efficiency tradeoffs"

# Hybrid search with LLM re-ranking (best quality, slowest)
qmd query "what are the tradeoffs of top-k vs expert-choice routing"

# JSON output for piping to agents
qmd query "scaling laws" --json

# Run as MCP server (Claude Code uses it as a native tool)
qmd mcp
```

### Integration with Claude Code

Add to `.claude/skills/recall.md`:
```markdown
# /recall

Before starting any new session topic, run:
`qmd query "{topic}" --json`

Parse the output and pre-load the top 5 most relevant wiki pages
into context before responding. This ensures answers are grounded
in existing wiki knowledge rather than hallucinated from training data.
```

> **Token savings:** Using qmd to pre-select relevant pages before loading them reduces token usage by 60%+ compared to naively reading all pages or relying on large context windows.

---

## 12. Custom Slash Commands

Defined in `.claude/skills/` as individual markdown files. Each file describes a workflow Claude Code executes on demand.

| Command | Trigger | Workflow |
| :--- | :--- | :--- |
| `/my-world` | New session start | Read `VAULT-INDEX.md`, `wiki/index.md`, last 10 `wiki/log.md` entries. Provide session briefing. |
| `/today` | Morning | Read daily notes, active tasks. Suggest 3 priority focuses based on wiki state. |
| `/close` | Evening | Summarize session activity. Export key decisions to wiki. Update `wiki/log.md`. |
| `/trace [concept]` | Exploration | Find all wiki pages mentioning concept. Reconstruct how understanding evolved across ingest dates. |
| `/ghost [draft]` | Writing | Read 20+ of user's vault notes. Rewrite or generate text in user's voice and style. |
| `/recall [topic]` | Session prep | Run `qmd query` for topic. Pre-load top 5 relevant wiki pages before answering. |

### Example Skill File: `/recall`

Save as `.claude/skills/recall.md`:

```markdown
# /recall

## Purpose
Pre-load relevant wiki context before responding to a new topic.
Reduces hallucination and improves answer quality.

## Workflow
1. Take the user's query or topic as input.
2. Run: `qmd query "{topic}" --json`
3. Parse the JSON response. Extract top 5 file paths.
4. Read each of those wiki pages.
5. Summarize what the wiki already knows about this topic.
6. Then proceed to answer the user's question with that context loaded.

## When to Use
Run /recall at the start of any new session or topic shift.
```

---

## 13. Advanced: Self-Evolving Loop

The compounding loop has three entry points. Once all three operate consistently, the wiki improves automatically without user intervention.

### Automation Safety & Recovery Protocols

Automated ingestion (via hooks or batching) introduces the risk of **partial writes** and **token exhaustion**.

**1. The Partial Write Detection:** Before every operation, the agent should check the last entry in `wiki/log.md`. If the last entry is an "ingest" but the corresponding `wiki/sources/` page is missing or incomplete, the agent MUST flag the failure and offer to resume or rollback before starting new work.

**2. Token Budget Management:** For batch ingests (>3 files), the agent should estimate total token usage. If the estimated usage exceeds 70% of the model's output context limit, the agent MUST split the task into sequential sessions to avoid mid-file truncation.

**3. Hook Failure Recovery:** If a `post-save` hook fails, the agent MUST write a `!failure` entry to `wiki/log.md`. This prevents the "silent rot" of skipped ingests that would otherwise go unnoticed until the next manual lint.

---

## 14. Vault Evolution & Migration

As your vault grows, the flat schema that worked at 10 sources will begin to saturate your context window.

### Inflection Points

- **10 Sources (The Validation Point):** Run your first cross-source query to confirm the compounding effect is working.
- **50 Sources (The Structure Point):** Move from a flat `wiki/concepts/` folder to sub-folders (e.g., `concepts/methods/`, `concepts/frameworks/`) if navigation slows.
- **100 Sources (The Search Point):** Install `qmd` and transition to semantic pre-loading. Switch from whole-vault `grep` to targeted reads.
- **200+ Sources (The Refactor Point):** Extract heavy procedural instructions from `CLAUDE.md` into separate `agents/` or `skills/` files to maintain a lean baseline context.

### The "Schema Refactor" Workflow

When the vault reaches an inflection point, run the following:
```
Analyze the current vault structure against the 200-source migration guide.
Propose a new directory structure or schema refactor to improve context efficiency.
Do not move files until the plan is approved.
```

---

## 15. Graph Architecture: Tree Shape Over Cluster Shape

The long-term navigability of a wiki depends on its graph topology. Without deliberate design, wikis collapse into **cluster-shaped graphs**: one gravity-well concept (usually the top-level idea) accumulates all inbound links, cross-cluster connections run only through that hub, and new pages attach wherever feels closest rather than where they structurally belong.

**The target shape is a tree:**

```
Tier 0: wiki/index.md          — entry point; cluster-organized
Tier 1: wiki/overview.md       — cluster navigation hub
Tier 2: hub concepts           — one per cluster; list their members
Tier 3: member concepts        — know their cluster; link up to hub
Tier 4: wiki/sources/*         — evidence behind concepts
Tier 5: wiki/syntheses/*       — built from tier-3; back-link upward
```

A tree shape means: given any page, a human or LLM agent can navigate upward to its cluster hub, then laterally to sibling pages, then downward to sources or syntheses — without needing to pass through a single gravity-well hub.

### Why This Matters for LLM Agents

When an agent loads `wiki/index.md` and sees a cluster-organized structure, it can immediately identify which cluster is relevant to the current task and read only those pages. A flat alphabetical list forces the agent to read more pages to build the same mental model — wasting tokens and degrading focus.

### Cluster Design Rules

1. **Assign every concept and entity a `cluster:` field** — the agent should never have to infer cluster membership from prose.
2. **One hub per cluster** — the hub page lists all members in `## In this cluster`. Members link back up to the hub.
3. **Syntheses are leaves, not hubs** — they draw from tier-3 pages and link back to them. Nothing should depend on a synthesis page for navigation.
4. **Sources are evidence, not navigation** — `wiki/sources/*` pages are referenced from concept frontmatter `sources:` lists. They should not appear in `wiki/index.md` as primary navigation targets.
5. **Cross-cluster links are allowed** — but they are secondary. The primary navigation path is the tree.

### Cluster Splitting: Preventing Gravity Wells

As a vault grows, hub nodes (like `llm-wiki.md`) can become "gravity wells" that accumulate too many inbound links, making them noisy and hard to navigate.

**Rule:** When a hub page exceeds 15 members in its `## In this cluster` section, split the cluster. Identify a sub-theme within the members and create a new sub-hub. This maintains the tree's navigability by preventing any single node from dominating the graph topology.

### Detecting Drift

Run the cluster health lint checks (see Lint Workflow) after every 10 ingests. Signs of cluster drift:
- A single concept accumulates >5× the inbound links of the next hub → split or redistribute
- New pages created without a `cluster:` field → orphans in the making
- Synthesis pages with ≤1 inbound link → back-link from the concept pages they drew on
- `wiki/overview.md` has 0 inbound links → link it from `wiki/index.md`

---

## 16. Use Cases and Configuration Variants

### A. Research Knowledge Base

**Config changes from default:**
- Add `wiki/timeline.md` — a chronological record of the field's development.
- Concept pages include `consensus: high | contested | emerging` frontmatter field.
- Lint check includes: "Are all claims within the last 6 months flagged for recency?"

**Best sources:** arXiv abstracts (Web Clipper), paper PDFs, GitHub READMEs, conference talk transcripts.

### B. Developer Documentation Wiki

**Config changes from default:**
- Strategy 2 (symlinked docs): `ln -s ~/vault/wiki ./docs` from each project repo.
- Add `wiki/decisions/` directory for Architecture Decision Records (ADRs).
- Source types include: PR descriptions, commit messages, meeting transcripts.

**Best sources:** PR descriptions exported to markdown, `git log --pretty=format:"%H %s"`, meeting transcripts, error logs with context.

### C. Content / Newsletter Brain

**Config changes from default:**
- Add `wiki/angles/` — synthesis pages for article angles and takes.
- Add `wiki/audience/` — entity pages for readers, personas, communities.
- `/ghost` skill is essential here.

**Best sources:** Competitor articles (Web Clipper), reader feedback exports, analytics summaries.

### D. Personal Second Brain

**Config changes from default:**
- Add `wiki/journal-themes/` — concept pages synthesized from journal entries.
- Add `wiki/goals/` — entity pages for long-term goals, with `status: active | paused | achieved`.
- `/today` and `/close` skills are essential here.

**Best sources:** Daily journal entries (markdown), health/fitness exports (CSV), podcast transcripts.

### E. Business / Team Wiki

**Config changes from default:**
- Git repo shared on GitHub/GitLab; team members push new raw sources.
- Human-in-the-loop review before LLM writes to wiki (use PR review process).
- Add `wiki/decisions/` for company decision log.
- Strategy 3 (MCP Bridge) for multi-user access.

**Best sources:** Slack export (JSON → markdown converter), meeting transcripts (Otter.ai), CRM exports, project docs.

---

## 17. Community Best Practices and Failure Modes

### The Golden Rules

1. **"Agents read, humans write" — for raw sources only.**
   Human knowledge lives in `raw/` (human drops) and is curated by the human. LLM writes `wiki/`. Never have the LLM generate content that goes into `raw/` — it contaminates your source of truth with AI-generated material.

2. **Never pollute the vault with AI journaling or AI-generated "thoughts."**
   Keep Claude's plans, memory, and operational files in `.claude/`. Obsidian is for human knowledge and LLM-maintained wiki. The boundary is strict.

3. **Ingest one source at a time when you can.**
   Karpathy's preference. Staying involved produces richer cross-referencing and catches errors the LLM would miss without your domain context.

4. **Schema is persistent memory. Evolve it.**
   Start with the template in Section 4. After every 20–30 ingests, review the CLAUDE.md and refine it: add new page types you discovered you needed, update workflows that didn't work, document exceptions.

5. **Use modular, conditional CLAUDE.md, not a monolith.**
   As the schema grows, split it into included sub-files: `CLAUDE-ingest.md`, `CLAUDE-query.md`, `CLAUDE-schema.md`. Reference them from the root `CLAUDE.md`. This keeps each section focused and makes individual sections easier to evolve.

6. **Commit after every ingest.**
   ```bash
   git add wiki/ && git commit -m "ingest: [source title]"
   ```
   The wiki is a codebase. Treat it like one.

### Common Failure Modes

| Failure Mode | Symptom | Fix |
| :--- | :--- | :--- |
| **No schema file** | LLM reinvents conventions every session | Write and maintain `CLAUDE.md` |
| **Monolithic CLAUDE.md** | Schema grows to 5,000+ tokens; LLM ignores parts | Split into modular sub-files |
| **Infrequent linting** | Contradictions accumulate; wiki becomes inconsistent | Run lint weekly |
| **Skipping index updates** | Index drifts from reality; query quality degrades | Enforce index update in CLAUDE.md workflow |
| **AI writing raw/ files** | Source of truth contaminated | Hard rule: LLM never writes to `raw/` |
| **No git history** | Bad ingest corrupts pages; no rollback | `git init` at vault root on day one |
| **Symlinks + mobile sync** | Recursive loops; sync client errors | Use MCP bridge instead of symlinks |
| **Oversized wiki/index.md** | Index itself hits context limit | Add qmd; switch to semantic pre-loading |

### When NOT to Use This Stack

- You need real-time, sub-second search across millions of documents → use a proper vector DB.
- Your team has no Git discipline → file chaos will defeat the compounding effect.
- You need multi-modal, image-first knowledge (e.g., diagrams are primary) → the LLM image workflow is clunky (read text, view images separately).
- You want a zero-maintenance system → this requires ongoing curation of `raw/` sources. The LLM handles wiki maintenance, but you must curate inputs.

---

## 18. Quick Start: 15-Minute Build

### Step 1 — Provision the Directory

```bash
mkdir -p ~/knowledge-vault/raw/{articles,papers,repos,transcripts,data,assets}
mkdir -p ~/knowledge-vault/wiki/{sources,entities,concepts,comparisons,syntheses}
mkdir -p ~/knowledge-vault/.claude/skills

touch ~/knowledge-vault/wiki/index.md
touch ~/knowledge-vault/wiki/log.md
touch ~/knowledge-vault/wiki/overview.md

cd ~/knowledge-vault
git init
```

### Step 2 — Configure Obsidian

```bash
mkdir -p .obsidian
cat > .obsidian/app.json << 'EOF'
{
  "userIgnoreFilters": ["node_modules/", ".git/", ".claude/"],
  "newFileLocation": "folder",
  "newFileFolderPath": "raw/articles",
  "attachmentFolderPath": "raw/assets"
}
EOF
```

Open `~/knowledge-vault/` in Obsidian as a new vault. Install Dataview and Templater plugins.

### Step 3 — Create CLAUDE.md

Paste the full CLAUDE.md schema from Section 4 into `~/knowledge-vault/CLAUDE.md`. Edit the `## Domain` section for your topic.

### Step 4 — Initialize Claude Code

```bash
# Point Claude Code at the vault root
cd ~/knowledge-vault
claude init

# Optional: install Obsidian CLI skills
claude plugin marketplace add jackal092927/obsidian-official-cli-skills
```

### Step 5 — Bootstrap Prompt

Open Claude Code in the vault directory and run:

```
Here is my CLAUDE.md schema. Please:
1. Read it fully.
2. Confirm you understand the directory structure and all three operations.
3. Initialize wiki/index.md with the correct header and empty category sections.
4. Initialize wiki/log.md with the log format header.
5. Initialize wiki/hot.md with the correct structure (Current Focus, Open Questions,
   Recent Decisions, Last Operations, Active Pages). Mark it as freshly initialized.
6. Write a brief wiki/overview.md stating the domain and that the wiki is newly initialized.
7. Tell me what to drop into raw/ for the first ingest.
```

### Step 6 — First Ingest

Install Obsidian Web Clipper. Clip one article related to your domain. Save to `raw/articles/`. Hit `Ctrl+Shift+D` to download images. Then:

```
Ingest raw/articles/[your-article-filename].md
```

### Step 7 — Verify and Commit

Open Obsidian. Check:
- `wiki/sources/` has a new summary page.
- `wiki/index.md` has been updated.
- `wiki/log.md` has a new entry.
- Graph view shows the new pages with connections.

```bash
git add . && git commit -m "init: vault bootstrap + first ingest"
```

### Step 8 — First Query

```
What do we know about [topic from your first article]?
Cite wiki pages. If the answer is valuable, file it.
```

The wiki is now live and compounding.

---

## 19. Known Repos and Starter Kits

| Resource | Type | Notes |
| :--- | :--- | :--- |
| [Karpathy's original gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) | Idea file | The canonical source. Copy-paste to any LLM agent. |
| `forrestchang/andrej-karpathy-skills` | Claude Plugin | Mandatory behavioral layer enforcing surgical coding principles |
| `ballred/obsidian-claude-pkm` | GitHub repo | Goal cascading (yearly/monthly/weekly) variant |
| `huytieu/COG-second-brain` | GitHub repo | Self-evolving template with hooks |
| `ksanderer/claude-vault` | GitHub repo | Git-based cloud sync variant |
| `heyitsnoah/claudesidian` | GitHub repo | Pre-configured vault structure |
| `SamurAIGPT/llm-wiki-agent` | GitHub repo | Full agent implementation |
| Karpathy's `.brain` pattern | Gist discussion | Lightweight project-scoped variant (see CLAUDE.md discussion tab) |

---

## 20. Sources

1. Andrej Karpathy, "LLM Wiki" — GitHub Gist, April 2026: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
2. Antigravity.codes, "Karpathy's LLM Wiki: The Complete Guide to His Idea File," April 2026: https://antigravity.codes/blog/karpathy-llm-wiki-idea-file
3. Starmorph, "Obsidian + Claude Code: The Complete Integration Guide," April 2026: https://blog.starmorph.com/blog/obsidian-claude-code-integration-guide
4. Matt Paige, "Andrej Karpathy Just Showed Us How to Build an AI Second Brain," April 2026: https://mattpaige68.substack.com/p/andrej-karpathy-just-showed-us-how
5. Mehmet Goekce, "I Built Karpathy's LLM Wiki with Claude Code and Logseq," April 2026: https://mehmetgoekce.substack.com/p/i-built-karpathys-llm-wiki-with-claude
6. Obsidian Changelog, "Desktop v1.12.4," February 2026: https://obsidian.md/changelog/2026-02-27-desktop-v1.12.4/
7. Kepano (Steph Ango), "obsidian-skills — Agent skills for Obsidian," GitHub, January 2026: https://github.com/kepano/obsidian-skills
8. Kepano (Steph Ango), "defuddle — Get the main content of any page as Markdown," GitHub: https://github.com/kepano/defuddle
9. jackal092927, "obsidian-official-cli-skills — prevents 13 silent failures," Obsidian Forum, February 2026: https://forum.obsidian.md/t/open-source-agent-skill-for-obsidian-cli-prevents-13-silent-failures/111169
10. Repovive, "Official Obsidian Skills for Claude Code," February 2026: https://repovive.com/roadmaps/claude-code/obsidian-claude-code-your-ai-powered-second-brain/obsidian-skills
11. Forrest Chang, "andrej-karpathy-skills — Surgical coding guardrails for LLM agents," GitHub, April 2026: https://github.com/forrestchang/andrej-karpathy-skills
