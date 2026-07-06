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

## The five skills (+ codebase-design)

Keyed to **what the agent is doing** (reliable triggering), not to which book an
idea came from. See `README.md`'s "The five skills" table for the full
skill-by-skill breakdown; `codebase-design` (pre-existing, not part of this
repo) supplies the shared vocabulary, mirrored locally in
`references/vocabulary.md`.

## Shared references (the comprehensive corpus)

Every skill points to these; a reviewer runs on them directly:

- `references/red-flags.md` — symptom catalog across all three books (27 flags).
- `references/principles.md` — positive principles across all three books, unified under the "Easier To Change" master lens.
- `references/review-checklist.md` — **the reviewer-agent rubric**: seven dimensions spanning all three books, run top to bottom against another agent's output. Entered via `design-review`.

## Scope discipline

Comprehensive on *design quality*, but **defers, doesn't duplicate**:
- Debugging, TDD, test-writing, git/CI → owned by the **superpowers** skills.
- Deep-module vocabulary (module/interface/seam/depth/leverage/locality) →
  mirrored locally in `references/vocabulary.md` so this set is self-sufficient
  standalone, with the full treatment owned by **codebase-design**.

## Cross-linking (the safety net)

Each SKILL.md (a) carries a working version of cross-cutting ideas so it's
self-sufficient, (b) points to the shared references for depth, (c) names its
siblings so an agent can hop doors, (d) points to `references/vocabulary.md`
for the shared terms and to `codebase-design` for the full treatment.
`codebase-design` links back to all five; entering through any one is fine.

## Install

See `README.md`'s "Installing" section — this repo installs as a Claude Code
plugin (skills and agents ship together, no manual copying).
