---
name: codebase-audit
description: Use when reviewing an existing repository's architecture as-built — auditing a whole codebase or large subsystem (not a single diff or module) for design quality, finding hotspots, dependency cycles, layering violations, and where complexity is actually costing maintenance. Applies to code written by AI agents or humans. Reports by design-quality dimension, hotspot-first.
---

# Codebase Audit

Review a repository's architecture **as it actually stands**, at repo scale — the
gap that `design-review` (diff/module scope) and `principal-design-reviewer`
(pre-code planning artifacts) both leave open. The goal isn't a linter dump; it's
an architectural narrative that leads with **where complexity is costing the most
to change**, judged by the same design lens as the rest of this repo.

Executable metric commands: [references/metrics.md](references/metrics.md). Design
vocabulary and rubric come from the `design-review` skill's
`references/vocabulary.md`, `references/red-flags.md`, and
`references/review-checklist.md` — invoke `design-review` to load them (its base
directory is shown on load; cite relative to that, not by bare path).

## When to use vs siblings

- Whole repo / large subsystem, code already written → **here**.
- A single diff, PR, or module → **design-review**.
- A pre-code design doc, spec, or plan → **principal-design-reviewer** agent.
- You found a module that needs redesigning → hand the specific module to
  **module-design**; coupling/duplication across modules → **designing-for-change**.

## Procedure

Run top to bottom. Steps 0–6 are cheap and git/grep-based (see
[references/metrics.md](references/metrics.md)); don't skip the baseline.

0. **Scope & baseline.** Repo age, contributor count (bus factor), languages,
   size. A two-week AI-agent scaffold and a five-year legacy system get different
   expectations — establish which before judging anything.
1. **Module inventory.** Map top- and second-level directories to candidate
   modules; get file count and LOC each. This is the map you annotate in every
   later step.
2. **Churn.** Most-changed files over the last 6–12 months (git log). Exclude
   generated/vendored files first.
3. **Complexity proxy.** LOC per file (cheap, what CodeScene itself uses), or a
   grep decision-point count, or `scc` if available. A ranking signal, not a score.
4. **Hotspots = churn × complexity.** Lead with the intersection: files both
   frequently changed and complex. This is the ~1–2% of the codebase that drives
   most maintenance cost — the report's headline.
5. **Change coupling.** Files that change together in the same commits but have no
   import/call relationship — hidden temporal coupling. Flag pairs that cross a
   module or layer boundary; that's an architecture smell, not a nit.
6. **Dependency & layering.** Build a lightweight import graph. Detect cycles,
   layer-direction violations (lower layer importing higher), and instability
   `I = Ce/(Ce+Ca)` outliers — a depended-upon module that itself depends on
   volatile ones is fragile. If intended layers are undocumented (common in
   AI-built repos), state them as a hypothesis and flag violations against it.
7. **Judge hotspots by dimension.** For each hotspot/finding, apply the
   `design-review` rubric — is it a *Shallow Module*, *Information Leakage*,
   *Temporal Decomposition*, a DRY violation? Run the module's **deletion test**.
   Report the design cause, not just the metric.

## Prioritize hotspot-first (don't drown the report in nits)

1. **Hotspot intersection** — high churn ∧ high complexity ∧ (crosses a boundary or
   sits in a cycle). Lead here, unconditionally.
2. **Structural violations** independent of churn — cycles, layering violations,
   inverted instability. Latent risk; they predict *future* hotspots.
3. **Isolated complexity/size outliers** (complex but stable) — mention, low priority.
4. **Style/naming/nits** — only if asked, and batched, never itemized per file.

For a repo too large for one context window, dispatch one Sonnet subagent per
module (each capped in output), then dedupe and synthesize — don't force the whole
tree through one pass.

## Report shape

```
## Codebase audit: <repo/subsystem>

## Scope & baseline
<age, size, contributors, languages — the expectation-setting>

## Hotspots (lead here)
- <file:line> — <churn/complexity signal> → <design dimension + red flag> → <fix>
  (top N only, ranked; severity × confidence each)

## Structural findings
<cycles, layering violations, instability outliers — file:line cited>

## Looks bad but actually fine  (REQUIRED, non-empty)
<the strongest candidate you chose NOT to flag, and why it holds up — steelman the
status quo before publishing. An empty section is itself a failure of the audit.>

## Prioritized fix list
1. <highest-leverage, hotspot-first>

## Open questions
<what you couldn't determine — undocumented intended layers, unclear ownership>
```

## What NOT to do

- Don't itemize every nit — rank by "does fixing this reduce future change cost."
- Don't re-run linting/formatting the toolchain already does.
- Don't treat a red flag as an automatic defect — it's a prompt to look for a better
  design; if a flag survives a genuine design-it-twice pass, say why it's least-bad.
- Don't force a hotspot narrative onto a repo that has none — for a weak-agent
  scaffold the real finding is often "no layering exists yet"; say that plainly.
- Don't fabricate metrics — every number traces to a command you actually ran.

## Red flags in your own audit

- Your report is a flat list, not ranked hotspot-first.
- The "looks bad but fine" section is empty — you didn't actually weigh alternatives.
- A finding has no `file:line` — "the code generally…" is not an audit finding.
- You reported metrics without naming the design cause behind them.
