# Principal Design Reviewer — Team-of-Agents Design

## Purpose

A senior-reviewer agent that evaluates a design/architecture/plan **before code
gets written** — the codebase's proposed structure, its modules, its task
segmentation — using the five software-design skills already built
(`design-process`, `module-design`, `designing-for-change`, `design-review`,
`inline-authoring`, plus the shared `codebase-design` vocabulary and the
`review-checklist.md` rubric). It can spawn a small team of subagents, but only
when it judges that's actually warranted — this is explicitly *not* the
council system's always-heavy philosophy.

## Prior art considered

`llm-council` (`/home/omar/projects/llm-council/.claude/skills/council/`) is a
working "team of agents" pattern in this environment: a skill orchestrates fixed
parallel `Agent` calls to persona files, always running the full roster. It's
proven but the wrong shape here — this system needs adaptive, judgment-driven
delegation, not a fixed pipeline. We reuse the *mechanism* (persona files
dispatched via the `Agent` tool) but not the *policy* (always-heavy) or the
*orchestrating-skill layer* (a separate skill driving fixed stages). Here, the
**lead agent itself** holds `Agent`-tool access and decides at runtime.

## Roster

| Agent | Model | Tools | Role |
|---|---|---|---|
| `principal-design-reviewer` | Opus | Read, Grep, Glob, Bash, Skill, Agent | Team lead. Does the review itself by default; spawns the other two only when warranted; rules on disagreements; produces the final report. |
| `design-specialist-reviewer` | Sonnet | Read, Grep, Glob, Bash, Skill | Generic deep-dive persona. The lead writes a tailored brief naming which skill(s)/dimension to focus on. Reused across tasks — no separate file per skill. |
| `design-skeptic` | Sonnet | Read, Grep, Glob, Bash, Skill | Combined devil's-advocate + pessimist. Argues the single sharpest way the design fails outright, and the slow-decay case (entropy, scope creep, maintenance burden). Advisory only, never a veto. |

No agent beyond the lead holds `Agent` tool access — delegation is exactly one
level deep, no recursive spawning.

## What it reviews

- Design/spec documents (e.g. what `superpowers:brainstorming` produces, or
  existing `PRD.md` / `ARCHITECTURE.md`-style files).
- Implementation plans / task breakdowns (e.g. what `writing-plans` produces) —
  are tasks well-segmented, sized right, sequenced sensibly, not hiding a design
  problem inside "just code it"?
- Existing codebase structure, when the work extends something that already
  exists — read for context, not as the primary review target.

The lead is expected to proactively look for and read surrounding project
context on its own initiative (PRD, ARCHITECTURE.md, roadmap/future-phases
docs, CONTEXT.md) even if not explicitly pointed at them, so its verdict
accounts for where the project is headed, not just the artifact in isolation.

## Decision procedure ("only if necessary," made concrete)

1. **Always**: the lead invokes the relevant design skills via the `Skill` tool
   and runs `review-checklist.md`'s seven dimensions itself. This is the
   default, no-spawn path, and will resolve the large majority of reviews.
2. **Spawn the specialist** only if: a dimension is large or ambiguous enough
   that the lead's own confidence is low, or the user explicitly asked for a
   thorough/deep review.
3. **Spawn the skeptic** only if: a reviewed decision is expensive to reverse,
   or the lead doubts the design's long-run robustness under real usage/scale/
   maintenance, or the user explicitly asked for adversarial stress-testing.
4. **Disagreement handling**: if the lead's own read conflicts with a spawned
   subagent's finding, or the specialist and skeptic conflict with each other,
   the lead first tries to **rule directly** — checking whether a specific
   red-flag or principle from the shared references settles it outright (most
   disagreements resolve this way; e.g. the "one adapter = hypothetical seam,
   two = real" test cited in `codebase-design`/`designing-for-change`). Only
   when it's a genuine judgment call the skills don't resolve outright does it
   fall back to **one rebuttal round**: relay each side's argument to the
   other, ask each to rebut or revise, then rule. Capped at one round — no
   open-ended back-and-forth, and never delegated to the subagents to resolve
   between themselves (they can't message each other directly in this
   harness; the lead always mediates).

## Output

A single structured markdown report from the lead:

```
## Verdict: <ready to build | needs revision>

## Findings (by review-checklist dimension)
- [Dimension N — name] location → why it costs changeability → fix
  ...

## What to fix before coding starts
1. ...
2. ...

## What the specialist/skeptic added
(only present if spawned — includes any disagreement raised and how,
per the decision procedure above, it was resolved)
```

The verdict and prioritization is the lead's own judgment — it is not a
majority vote across the roster, matching how `council-chairman` is never
bound by majority sentiment either.

## Scope discipline

- Judges **design quality**, not process execution — debugging, TDD mechanics,
  test-writing, and git/CI stay owned by the superpowers skills.
- Vocabulary (module/interface/seam/depth/leverage/locality) stays owned by
  `codebase-design`; this system's agents use it, not redefine it.
- No wiring into `superpowers:brainstorming`'s approval gate — invocation is
  on-demand only, via the `Agent` tool, either at the user's request or when
  the top-level assistant judges a design substantial enough to warrant it.

## Files & install

```
software-development-skills/agents/
  principal-design-reviewer.md
  design-specialist-reviewer.md
  design-skeptic.md
```

Authored here (project, source of truth), then **symlinked** into
`~/.claude/agents/` — matching the convention the existing `council-*` agents
already use (symlinked from `llm-council`'s project `.claude/agents/`), rather
than copied like the design skills were. A symlink keeps the global copy
automatically in sync with the project source with no manual re-sync step.

`software-development-skills/` is not a git repo, so the "commit the design doc
to git" step from the brainstorming process is skipped for this spec; noting
that explicitly rather than silently omitting it.
