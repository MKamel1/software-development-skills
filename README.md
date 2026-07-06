# Software Development Skills

A Claude Code plugin that puts three software-design books into an agent's hands: five skills that **trigger automatically** while you design, write, or review code, plus a **senior reviewer agent team** you can call on-demand to judge a design before code gets written.

Three books — John Ousterhout's *A Philosophy of Software Design* (APoSD), Andrew Hunt & David Thomas's *The Pragmatic Programmer* (PP), and Fred Brooks's *The Design of Design* (DoD) — distilled into principles an agent can actually apply. Principles are paraphrased with chapter/topic citations; no book text is reproduced.

## Installing

This repo is a [Claude Code plugin](https://claude.com/claude-code) (`.claude-plugin/plugin.json` + `marketplace.json`) — skills and agents install and update together, with no manual copying or symlinking to keep in sync.

```
/plugin marketplace add MKamel1/software-development-skills
/plugin install software-development-skills
```

Restart Claude Code (or reopen `/plugin`) to pick up the new skills and agents. Check `/plugin list` — you should see `software-development-skills` with 5 skills and 4 agents. Update it later with `/plugin update software-development-skills`, or a marketplace refresh; either picks up any change to this repo automatically, with no separate "re-copy" step. Your `~/.claude/CLAUDE.md` can still route agents to these skills by name — that routing is plugin-agnostic.

**Developing locally:** to test edits before pushing, point the marketplace at your working copy instead of GitHub:

```
/plugin marketplace add /path/to/software-development-skills
/plugin install software-development-skills
```

## Triggering it

**The five skills trigger automatically** — there's no slash command and nothing to invoke by name. Claude Code matches what you're doing against each skill's description and loads the matching one on its own. Say what you're actually trying to do, in your own words:

| Say something like… | Triggers |
|---|---|
| "I have a rough idea for X, help me figure out how to approach building it" | `design-process` |
| "Design the interface for this `OrderRepository` class" | `module-design` |
| "This module and that one keep changing together, how do I decouple them?" | `designing-for-change` |
| "Review this diff / is this PR well-designed?" | `design-review` |
| "What should I name this variable / is this comment adding anything?" | `inline-authoring` |

**The reviewer agent is on-demand only** — ask for it by name, naturally, before code-writing starts:

> "Have the principal design reviewer look at this design doc before I start coding."
> "Get a senior architecture review of ARCHITECTURE.md and PRD.md."

`principal-design-reviewer` decides on its own whether to dispatch any of its three subagents (`design-specialist-reviewer`, `review-skeptic`, `change-minimizer`) — you never invoke those directly, and there's no separate command to trigger them.

## What's in this repo

```
skills/
  design-process/          Approaching a design problem before modules are known
  module-design/            Designing an interface/API up front
  designing-for-change/     Coupling, duplication, reversibility across modules
  design-review/            Evaluating existing code/diffs/PRs for design quality
  inline-authoring/         Naming, comments, consistency while writing code
  references/
    red-flags.md            27 design-smell symptoms spanning all 3 books
    principles.md           Positive principles, unified under "Easier To Change"
    review-checklist.md     7-dimension rubric — the reviewer-agent's rubric
    vocabulary.md           Module/interface/seam/depth/leverage/locality gloss
    examples.md             One before→after worked example per skill

agents/
  principal-design-reviewer.md     Senior reviewer; forms a provisional verdict, then adjudicates
  design-specialist-reviewer.md    Pre-verdict deep-dive persona, dispatched when warranted
  review-skeptic.md                Post-verdict red-team: challenges a review that was too lenient
  change-minimizer.md              Post-verdict red-team: challenges a fix list that over-prescribes change

docs/superpowers/
  specs/    Design spec for the agent team
  plans/    Implementation plan for the agent team
```

## The five skills

Keyed to **what you're doing**, not to which book an idea came from — this is what makes them trigger reliably instead of requiring you to know which chapter applies.

| Skill | Use when | Draws mainly on |
|---|---|---|
| `design-process` | Approaching a design problem, before you know the modules | DoD Ch 1–16, PP design-attitude tips |
| `module-design` | Designing a specific interface/API/module | APoSD Ch 4–11, PP Design-by-Contract |
| `designing-for-change` | Reducing coupling/duplication across modules | PP DRY/orthogonality/reversibility, APoSD info hiding |
| `design-review` | Evaluating existing code, a diff, or a PR | All three books — the reviewer-agent's hub |
| `inline-authoring` | Writing code right now (names, comments) | APoSD Ch 12–18, PP naming |

A separate, pre-existing skill called `codebase-design` supplies the shared vocabulary (module/interface/seam/depth/leverage/locality) these five build on. It isn't part of this repo, but each skill here links to it and to its siblings, so entering through any one of them works.

## The agent team

`principal-design-reviewer` is a senior reviewer for **planning artifacts** — design docs, specs, implementation plans, task breakdowns — reviewed *before* code gets written. It:

- Reads the artifact you point it at, plus any `PRD.md`/`ARCHITECTURE.md`/`ROADMAP.md`/`CONTEXT.md` it finds nearby, on its own initiative
- Applies the five skills above and runs `review-checklist.md`'s seven dimensions itself, by default
- May dispatch `design-specialist-reviewer` (a pre-verdict deep-dive persona) **only when genuinely warranted**, then forms a **provisional** verdict, findings, and fix list from its own pass
- May then dispatch a post-verdict red-team — `review-skeptic` (challenges a verdict that was too lenient) and/or `change-minimizer` (challenges a fix list that over-prescribes change) — each reading the artifact and the provisional report independently, with no steering brief, so their challenge isn't anchored to how the principal framed it
- Adjudicates every challenge in one round: accepts and revises, or rejects and justifies with a cited red flag or principle
- Reports a **final** verdict (`ready to build` / `needs revision`), findings by dimension, a prioritized fix list, and how any challenges were resolved — its own judgment, never a vote

See `docs/superpowers/specs/2026-07-05-principal-design-reviewer-design.md` for the original design rationale — that spec predates the post-verdict red-team described above; `agents/principal-design-reviewer.md` is the current source of truth for its behavior.

## Scope

These skills judge **design quality** — they deliberately don't cover debugging, TDD mechanics, test-writing, or git/CI process (that's [superpowers](https://github.com/obra/superpowers)' territory), and they don't redefine vocabulary already owned by `codebase-design`.

## Sources

See `skills/DESIGN.md`'s Sources section for the full citations (edition, publisher, and scope notes) — this repo distills Ousterhout's *A Philosophy of Software Design*, Andrew Hunt & David Thomas's *The Pragmatic Programmer*, and Fred Brooks's *The Design of Design*.
