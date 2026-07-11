# Software Design Skills — from three books

A comprehensive, interlocking set of skills that put three design books into the
hands of a coding/design agent — and, together, serve as a **design-quality
rubric a reviewer agent applies to other agents' work** (including work produced
under `codebase-design` or a superpowers workflow). Distilled in our own words
with chapter/topic citations; the book texts are not reproduced.

## Sources

The canonical source list for this repo — `README.md` and
`references/principles.md` link here rather than restating it.

- **APoSD** — Ousterhout, John K. — *A Philosophy of Software Design*
  (2018/2019, Yaknyam Press).
- **PP** — Hunt, Andrew & Thomas, David — *The Pragmatic Programmer*, 20th
  Anniversary Edition (2020, Pearson).
- **DoD** — Brooks, Frederick P. — *The Design of Design: Essays from a
  Computer Scientist* (2010, Addison-Wesley) — agent-usable subset, Ch 1–16;
  the house/hardware case studies and "great designers" management chapters
  are out of scope for a coding agent.

Additional primary sources distilled into the **review/audit** skills' reference
files (paraphrased with chapter citations, no reproduced text; provenance and
per-book influence recorded in `docs/design/`):

- **Tornhill** — *Software Design X-Rays* (2018) & *Your Code as a Crime Scene* 2nd
  ed. (2024, Pragmatic Bookshelf) → `codebase-audit/references/behavioral-analysis.md`.
- **Ford, Parsons & Kua** — *Building Evolutionary Architectures* (2017, O'Reilly)
  → `codebase-audit/references/fitness-functions.md`.
- **Giordani** — *Clean Architectures in Python* (2nd ed., 2023)
  → `codebase-audit/references/dependency-layering.md`.
- **Feathers** — *Working Effectively with Legacy Code* (2004, Prentice Hall)
  → `references/seams.md` (shared).
- **APoSD 2nd ed.** — Ousterhout → `meta-code-review/references/ai-authored-code.md`.

## The skills

Keyed to **what the agent is doing** (reliable triggering), not to which book an
idea came from. Two families:

- **Five design skills** — `design-process`, `module-design`,
  `designing-for-change`, `design-review`, `inline-authoring`.
- **Two review/audit skills** — `codebase-audit` (repo-scale as-built review,
  hotspot-first) and `meta-code-review` (judging a review's soundness, calibrated
  to weak-AI / strong-AI / junior-developer producers). Both apply the same design
  rubric as `design-review` at the scales it doesn't cover, and are grounded in the
  research captured under `docs/design/`.

See `README.md`'s skill tables for the full breakdown. The shared vocabulary and the
deepening/design-it-twice procedures the skills build on are absorbed locally into
`references/` (`vocabulary.md`, `deepening.md`, `design-it-twice.md`); the separate
`codebase-design` skill, if installed, covers the same ground in more depth as
optional enrichment.

## Shared references (the comprehensive corpus)

Every skill points to these; a reviewer runs on them directly:

- `references/red-flags.md` — symptom catalog across all three books.
- `references/principles.md` — positive principles across all three books, unified under the "Easier To Change" master lens.
- `references/review-checklist.md` — **the reviewer-agent rubric**: seven dimensions spanning all three books, run top to bottom against another agent's output. Entered via `design-review`.

## Scope discipline

Comprehensive on *design quality*, but **defers, doesn't duplicate**:
- Debugging, TDD, test-writing, git/CI → owned by the **superpowers** skills.
- Deep-module vocabulary (module/interface/seam/depth/leverage/locality) and the
  deepening/design-it-twice procedures → absorbed locally into
  `references/vocabulary.md`, `references/deepening.md`, and
  `references/design-it-twice.md` so this set is self-sufficient standalone;
  **codebase-design**, if installed, is optional enrichment on top.

## Cross-linking (the safety net)

Each SKILL.md (a) carries a working version of cross-cutting ideas so it's
self-sufficient, (b) points to the shared references for depth, (c) names its
siblings so an agent can hop doors, (d) points to `references/vocabulary.md`,
`references/deepening.md`, and `references/design-it-twice.md` for the shared
terms and procedures — and to `codebase-design` as optional deeper reading if
it's installed. Entering through any one skill is fine.

## Install

See `README.md`'s "Installing" section — this repo installs as a Claude Code
plugin (skills and agents ship together, no manual copying).
