# Software Design Skills — from three books

A comprehensive, interlocking set of skills that put three design books into the
hands of a coding/design agent — and, together, serve as a **design-quality
rubric a reviewer agent applies to other agents' work** (including work produced
under `codebase-design` or a superpowers workflow). Distilled in our own words
with chapter/topic citations; the book texts are not reproduced.

## Sources

- **APoSD** — Ousterhout, *A Philosophy of Software Design*.
- **PP** — Hunt & Thomas, *The Pragmatic Programmer* (20th Anniv.).
- **DoD** — Brooks, *The Design of Design* (agent-usable subset, Ch 1–16; the
  house/hardware case studies and "great designers" management chapters are out
  of scope).

## The five skills (+ codebase-design)

Keyed to **what the agent is doing** (reliable triggering), not to which book an
idea came from. `codebase-design` (pre-existing) supplies the shared vocabulary.

| Skill | Fires when the agent is… | Main sources |
|---|---|---|
| `design-process` | approaching a design problem before modules are known | DoD Ch 1–16 + PP design-attitude tips |
| `module-design` | designing an interface/API up front | APoSD Ch 4–11 + PP Design-by-Contract |
| `designing-for-change` | reducing coupling/duplication across modules | PP DRY/orthogonality/decoupling/reversibility + APoSD info hiding |
| `design-review` | evaluating existing code/a diff/a PR | all three books (hub for reviewer agents) |
| `inline-authoring` | writing code right now | APoSD Ch 12–18 + PP naming/deliberate-programming |

## Shared references (the comprehensive corpus)

Every skill points to these; a reviewer runs on them directly:

- `references/red-flags.md` — symptom catalog across all three books (27 flags).
- `references/principles.md` — positive principles across all three books, unified under the "Easier To Change" master lens.
- `references/review-checklist.md` — **the reviewer-agent rubric**: seven dimensions spanning all three books, run top to bottom against another agent's output. Entered via `design-review`.

## Scope discipline

Comprehensive on *design quality*, but **defers, doesn't duplicate**:
- Debugging, TDD, test-writing, git/CI → owned by the **superpowers** skills.
- Deep-module vocabulary (module/interface/seam/depth/leverage/locality) → owned by **codebase-design**.

## Cross-linking (the safety net)

Each SKILL.md (a) carries a working version of cross-cutting ideas so it's
self-sufficient, (b) points to the shared references for depth, (c) names its
siblings so an agent can hop doors, (d) defers vocabulary to `codebase-design`.
`codebase-design` links back to all five; entering through any one is fine.

## Install

Plain skills folders. `design-process/`, `designing-for-change/`,
`design-review/`, `module-design/`, `inline-authoring/`, and `references/` are
copied into `~/.claude/skills/` (global) and kept here (local). Relative links
(`../references/…`) resolve because the folders stay siblings. A user-level
`~/.claude/CLAUDE.md` routes all agents (incl. superpowers workflows) to these
skills and names the reviewer entry point.
