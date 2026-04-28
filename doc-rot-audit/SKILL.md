---
name: doc-rot-audit
description: Audit project documentation against the actual codebase to find and report doc rot — claims in README, ARCHITECTURE, CLAUDE.md, or any other doc that no longer match the code. Use this skill whenever the user mentions stale docs, doc rot, doc audit, doc drift, doc health, outdated documentation, or "the docs feel out of date." Also use it whenever the user is returning to a project after a pause, has just finished a refactor and wants to check what fell behind, is onboarding a new contributor, or asks for a periodic doc-health check on a long-lived project. Detects four kinds of rot: fictional content (docs describing things that don't exist), aspirational content presented as fact (docs describing what someone hopes will be true), stale content from past refactors (claims that were once true but the code has moved on), and inventory drift (file lists that have fallen behind the codebase). Stack-agnostic — works on Node, Python, Go, Rust, anything with a code/docs split. Produces a structured report with file:line evidence; does not auto-fix.
---

# Doc-rot audit

Documentation rots silently. Code moves forward; docs don't. Six months later, the docs describe an architecture that no longer exists, list files that have been deleted, recommend patterns that have been refactored away, and prescribe setups that were never installed.

This skill catches that. It does not "review" docs for quality or readability. It compares every claim the docs make against the actual code, treats the code as ground truth, and reports every mismatch.

## Core principle

**The code is the truth. Every claim in a doc either checks out against the code or it doesn't.**

Doc-rot detection is not a reading-comprehension exercise — it's a verification exercise. If a doc says "we have 9 stores," count the actual files. If a doc says "we use library X," grep for the import. If a doc says "after a refactor, do Y," verify Y still exists.

Vague verification fails. Specific, falsifiable verification works. Always produce file:line evidence, grep output, or directory listings — never "I checked, looks good."

## The four kinds of rot

Each is detected differently. Run all four checks; one type does not rule out the others.

### 1. Fictional content

Docs describing systems, files, libraries, or features that do not exist in the codebase. Often the result of templates copied from another project, or aspirational early-stage planning that never materialized.

**How to detect:**

- For each major doc, list the things it claims exist (files, packages, endpoints, services, components).
- Verify each one: does the file exist? Is the package in the manifest (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.)? Does the import resolve? Is the endpoint actually called anywhere?
- Anything claimed but absent is fictional.

**Examples to look for:**

- A doc describing an API layer when the project has no backend.
- A library described in detail that isn't in the dependency manifest.
- A "configuration file" referenced everywhere but not present at the path given.

### 2. Aspirational content presented as fact

Docs written in present tense ("we use X", "the system does Y") describing things that were planned but never built. Distinct from fictional content because the intent was real; the implementation never happened.

**How to detect:**

- Look for prescriptive setup docs (testing, CI, monitoring, security, deployment).
- Verify the setup actually exists: is the framework installed? Are the test files present? Is the CI config in place?
- A doc describing a setup step-by-step is suspect until verified.

**Examples to look for:**

- A testing strategy doc with full Jest/Vitest/pytest configs but zero test files in the repo.
- A monitoring guide referencing a Sentry DSN that's commented out everywhere.
- A deployment guide for infrastructure that was never provisioned.

### 3. Stale content from refactors

Docs that were correct when written but the code has changed. This is the most dangerous kind, because individual sentences can each have been true at some point — the doc as a whole now actively misleads.

**How to detect:**

- Find architectural claims in the docs ("uses LRU cache", "has 9 stores", "fetches from /api/users", "single-cache design").
- Verify each one against the current code: is the library still imported? Is the count still right? Is the architecture still the one described?
- After finding one stale claim, **grep all other docs for the old terminology**. Stale architectural facts cascade: change "dual cache" to "single cache" in one place and three other docs still mention the dual cache in passing.

**Examples to look for:**

- A doc describing a "dual cache" architecture when the code was refactored to a single cache.
- A doc citing limits ("max 500 items") when the limit has been raised.
- A doc describing a state shape that has been normalized.

### 4. Inventory drift

File lists, store inventories, type catalogs, and component catalogs that have fallen behind the codebase. New files added without doc updates, old files deleted without doc updates, files renamed without doc updates.

**How to detect:**

- For each directory the docs claim to describe, run a directory listing.
- Compare the actual file list against the documented list.
- Report mismatches in both directions: undocumented files that exist, documented files that don't (ghosts).

**Examples to look for:**

- A "ghost" file documented in ARCHITECTURE.md but absent from the filesystem.
- A new module added to the code but missing from the documented inventory.
- Renamed files where the doc still uses the old name.

## The procedure

Run the steps in order. Each step builds context for the next.

### Step 0: Scope the audit

- List every doc file in the project (typically `*.md` at root and under `docs/`, plus any `*.rst`, `*.adoc`, or wiki files in version control).
- For each file, note in one line what it claims to describe (architecture, components, deployment, testing, etc.).
- This is your audit map. Skip clearly-historical files (changelogs, release notes, archived docs in an `archive/` folder) — they are intentionally frozen and rot-checking them is noise.

### Step 1: Inventory drift check

For each directory the docs claim to describe:

- Run a directory listing on the actual directory.
- Extract the list of files claimed in the corresponding doc.
- Diff the two lists. Report:
  - **Undocumented files:** present in code, missing from doc.
  - **Ghost files:** documented but absent from code.
  - **Renamed files:** different name in code vs. doc, same purpose.

Apply this to every list you find: modules, packages, services, hooks, utility libraries, components, routes, config files, workers, types.

### Step 2: Fictional content check

For each major doc:

- Identify the systems, libraries, files, or features it describes as if they exist.
- Verify each one:
  - Files: does the path exist?
  - Libraries: is the package in the manifest? Is it actually imported anywhere (grep the source tree)?
  - Endpoints: are they called from the codebase?
  - Configs: do they exist at the documented paths?
- Flag anything that fails verification.

### Step 3: Aspirational content check

Look specifically at docs that prescribe setup, process, or workflow:

- Testing strategy
- CI/CD process
- Monitoring/observability
- Security policy
- Deployment runbook

For each, verify the prescribed thing actually exists:

- Test docs: are tests present? Is the framework installed?
- CI docs: is the CI config in the repo? Does it match what's documented?
- Monitoring: is the SDK installed and called?
- Security: are the documented headers/sanitizers in the code?

Aspirational content isn't always wrong to keep — but it must be honestly labeled. A doc describing a "planned" setup is fine; a doc describing the same setup as if it's current is rot.

### Step 4: Stale architectural claims

For each architecturally-significant doc (caching, state management, data flow, system architecture):

- Extract the specific claims: library names, version numbers, limits, counts, structural descriptions.
- For each claim, find the corresponding code (usually a single file or directory).
- Verify the claim line-by-line against the code.

When you find a stale claim:

- Note the doc and line.
- Note the source file and line that contradicts it.
- **Then grep the rest of the docs** for the old terminology. Cascade rot is the rule, not the exception.

### Step 5: Cascade sweep

After identifying any stale architectural claim, grep all docs for:

- The old terminology (e.g., "dual cache", "LRU cache" if removed).
- The old library name (if a dependency was replaced).
- The old structural names (if a refactor renamed things).

Stop when grep returns zero matches outside intentional history notes or archives.

### Step 6: Report

Produce a single report with:

- **Summary counts:** N fictional, N aspirational, N stale, N inventory drift.
- **Each finding individually**, with: file:line of the doc claim, file:line of the contradicting code, recommended fix.
- **Cascade-discovered findings** separately tagged so the human knows these are second-order.
- **Three things you're least confident about** — places where verification was incomplete or where you suspect more rot than you found.
- **Gut call:** are the docs healthy, salvageable, or in serious drift?

Do not soften. Doc rot is invisible until someone audits for it; the audit's value is in being honest.

## Verification rules

These apply throughout the procedure:

1. **Every claim in your report must be backed by file:line evidence or grep output.** Not "I checked X and it looks correct" — show the line of code that proves it.

2. **Directory listings and grep are your primary tools.** Reading docs alone never finds rot; only comparing docs to code does.

3. **When verifying a library is "in use", check imports, not just the manifest.** A package can be installed but unused (often a sign of an abandoned refactor — itself a rot finding).

4. **Distinguish "documented and correct" from "documented and incorrect" from "undocumented".** All three are useful signals. Undocumented isn't always rot, but it's worth surfacing.

5. **Self-verification is biased.** When you re-verify your own fixes (i.e., audit a refactor you yourself did earlier in the session), name that bias in the report and recommend an outside reader for a cold-read pass.

## When to use this skill

- **After any significant refactor.** Code change → cascade rot. Run this immediately to catch what shifted.
- **At the start of work on a project after a pause.** Long-dormant projects accumulate the most rot.
- **Periodically on long-lived projects.** Monthly is reasonable for active projects; quarterly for stable ones.
- **Before onboarding a new contributor.** They will read the docs cold and trip over every piece of rot.
- **When CLAUDE.md or any major doc "feels stale".** That feeling is usually right.

## What this skill does not do

- It does not improve writing quality. Rot is about truthfulness, not style.
- It does not enforce conventions. Whether a project should have a CLAUDE.md or how it's organized is out of scope.
- It does not auto-fix. It produces a report; the human or a follow-up agent decides what to fix.
- It does not catch logic errors in the code. The code is treated as ground truth even if the code itself has bugs.
- It does not validate links between docs and external resources. Only against the project's own code.

## Output format

Always end with the report from Step 6. The report is the deliverable — it is what the human acts on.
