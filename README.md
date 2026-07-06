# Software Development Skills

Software-design knowledge, distilled into skills and an agent team for coding agents (built for [Claude Code](https://claude.com/claude-code), using its Skills and Subagents features).

Three books — John Ousterhout's *A Philosophy of Software Design* (APoSD), Hunt & Thomas's *The Pragmatic Programmer* (PP), and Fred Brooks's *The Design of Design* (DoD) — distilled into principles an agent can actually apply, plus a senior review agent that applies them to your designs before you write code.

Principles are paraphrased with chapter/topic citations; no book text is reproduced.

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

agents/
  principal-design-reviewer.md     Senior reviewer for pre-code planning artifacts
  design-specialist-reviewer.md    Deep-dive persona it spawns when warranted
  design-skeptic.md                Devil's-advocate + pessimist persona it spawns when warranted

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
- Spawns `design-specialist-reviewer` (a deep-dive persona) or `design-skeptic` (devil's-advocate + pessimist) **only when genuinely warranted** — not a fixed always-on panel
- Resolves any disagreement itself, citing a specific red flag or principle first; only escalates to one capped rebuttal round for a genuine judgment call
- Reports a verdict (`ready to build` / `needs revision`), findings by dimension, and a prioritized fix list — its own judgment, never a vote

See `docs/superpowers/specs/2026-07-05-principal-design-reviewer-design.md` for the full design rationale.

## Installing

Skills and agents are plain files — Claude Code picks them up from `~/.claude/skills/` and `~/.claude/agents/`.

```bash
# Skills (copy — edit here, then re-copy to update)
cp -r skills/* ~/.claude/skills/

# Agents (symlink — edits here take effect immediately, no re-copy needed)
ln -s "$(pwd)/agents/principal-design-reviewer.md" ~/.claude/agents/
ln -s "$(pwd)/agents/design-specialist-reviewer.md" ~/.claude/agents/
ln -s "$(pwd)/agents/design-skeptic.md" ~/.claude/agents/
```

## Using it

**Skills** trigger automatically based on what you're doing — no command needed. Mention designing a module, reviewing a diff, or naming something, and the relevant skill loads itself.

**The reviewer agent** is on-demand only — ask for it naturally:

> "Have the principal design reviewer look at this design doc before I start coding."
> "Get a senior architecture review of ARCHITECTURE.md and PRD.md."

It decides on its own whether to spawn either subagent — you never invoke those two directly.

## Scope

These skills judge **design quality** — they deliberately don't cover debugging, TDD mechanics, test-writing, or git/CI process (that's [superpowers](https://github.com/obra/superpowers)' territory), and they don't redefine vocabulary already owned by `codebase-design`.

## Sources

- Ousterhout, John K. — *A Philosophy of Software Design* (2018/2019, Yaknyam Press)
- Hunt, David & Thomas, Andrew — *The Pragmatic Programmer*, 20th Anniversary Edition (2020, Pearson)
- Brooks, Frederick P. — *The Design of Design: Essays from a Computer Scientist* (2010, Addison-Wesley) — chapters 1–16 only; the house/hardware case studies and "great designers" chapters are out of scope for a coding agent.
