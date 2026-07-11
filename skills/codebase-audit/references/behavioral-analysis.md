# Behavioral Code Analysis — methods & thresholds

The "what it means and what the numbers should be" companion to
[metrics.md](metrics.md) (which has the runnable git/grep commands). Distilled from
Adam Tornhill's *Software Design X-Rays* (2018) and *Your Code as a Crime Scene*
2nd ed. (2024). Behavioral analysis reads a repo's **git history** as evidence of
where change and risk actually concentrate — the backbone of `codebase-audit`.

## Hotspots — do this first

**Hotspot = change frequency × complexity.** Rank files by both; the ones high on
both are the priority set. The point is triage, not judgment: complex code nobody
touches is *not* a hotspot — it's changing code that costs.

- **Complexity proxy = lines of code.** Tornhill argues *against* cyclomatic/Halstead
  as the default: they need a per-language parser, and studies show LOC predicts
  defects about as well while being language-neutral and trivial to compute (X-Rays
  ch.2; Crime Scene ch.3). Don't over-invest in a fancier metric.
- **X-Ray:** re-run the identical frequency×complexity ranking at *function* scope
  inside a hotspot file, to turn "refactor this 3000-line file" into a small target
  (X-Rays ch.2).
- **Analysis window:** default ~12 months; shrink to a month for very high-churn
  repos, or anchor to a major architectural event (Crime Scene ch.3).

**Why it works (numbers you can cite):** change activity in essentially every repo
follows a power law — a small fraction of files absorb most commits. Hotspots are
typically **~1–5% of the code but 25–75% of the defects** (one case: 4% of files
held 72% of defects; Crime Scene ch.3, citing MPS08). Lead the audit report with
that small set.

## Change coupling — the hidden architecture

Files that repeatedly change **in the same commit** with no code-level dependency
reveal the real, as-built structure — including coupling that crosses module/layer
boundaries invisibly.

- **Threshold (X-Rays ch.3):** treat two files as change-coupled when they change
  together in **≥ ~20 commits** *and* in **≥ 50% of the commits to either file**.
  (Tooling defaults differ — code-maat ships `min-revs 5`, `min-coupling 30%`; the
  X-Rays text argues the stricter 20-commit / 50% bar for reporting. Pick a bar and
  state it.)
- **Degree of coupling** = shared-commits ÷ commits-to-either-file, as a %.
- **Sum of Coupling (SOC)** = how many times a module was co-changed with *any*
  other, summed over all commits. High SOC marks the **architecturally significant**
  modules — the place to *start* an architectural review (Crime Scene ch.9).
- **Algorithm choice:** prefer the plain "% of shared commits" over a recency-weighted
  variant — Tornhill's data shows code more often decays than improves, so recency
  weighting adds no predictive value (Crime Scene ch.9).
- **Decay signal:** coupling *within* a component is expected; a coupling edge that
  **crosses a component/layer boundary** is the architecture smell to flag. For a CI
  omission-catcher, ~80% degree is a usable (configurable) bar (X-Rays ch.9).
- **Limits:** only sees same-commit coupling (misses code-now/test-later splits);
  plain git + code-maat don't follow renames (a move looks like delete+add) — only
  commercial tooling does. State this when you report it.

## Complexity trends — "is this hotspot getting worse?"

Compute complexity at each historic revision of a hotspot and look at the trend —
distinguishes "large but stable" from "actively deteriorating."

- **Metric for trends: indentation / "negative space."** Sum leading-whitespace
  units per line (correlates with McCabe/Halstead across 278 projects; unaffected by
  minor style). The **max** indentation per file is often the most diagnostic —
  it flags deeply nested islands even when the average looks fine (Crime Scene ch.5).
- **Threshold: ~10% relative rise** vs the file's *own* prior baseline is a warning.
  **Never an absolute universal cutoff** — that fires constantly in legacy code and
  desensitizes everyone (X-Rays ch.10).
- **Three shapes** to classify a hotspot: *Deteriorating* (climbing → refactor),
  *Refactored* (dip then flat), *Stable* (flat → leave it alone).
- **Rising hotspot:** run the ranking now vs truncated to N months ago; a file that
  climbed ≥10 positions is worth an early walkthrough before it's a top hotspot.

## Code age

Age = time since last modification. Old code is usually safer (a module a year older
has ~⅓ fewer faults, GKMS00); the danger zone is the **middle** — not recent enough
to be in anyone's head, not old enough to be a trusted black box. Tests are the
exception: old tests *lose* regression-catching power, don't treat "old = safe" for
them. Use age to deprioritize an old-but-still-highly-ranked hotspot (it's stopped
costing you) and to spot packages mixing wildly different ages (a split-by-topic vs
split-by-rate-of-change smell). (X-Rays ch.5.)

## Social / knowledge signals (use when the audit scope includes team risk)

- **Truck/bus factor:** min people who could leave before knowledge loss makes the
  code unmaintainable. Greedy heuristic (Crime Scene ch.14): per file, main developer
  = author with most *historically added* lines; remove top contributors one by one
  until >50% of files are "abandoned." (React: 1,600 contributors, truck factor 2.)
- **Knowledge maps / ownership patterns:** single-owner (consistent), balanced-with-a-
  main-owner (higher main-owner share → fewer defects, BNMG11), or many-minor-
  contributors-no-owner (strongest defect predictor). (Crime Scene ch.15.)
- **Conway's Law check:** build an author↔author graph from shared-file commits, map
  authors→teams; healthy = communication stays within teams, unhealthy = dense
  cross-team edges the org chart says shouldn't exist (Crime Scene ch.13).

## Data-quality checklist (run before trusting any ranking)

Need ~150–200 commits minimum. Exclude generated/vendored files. Watch for: VCS-
migration or merge-only author misattribution; history-losing repo splits; squashed
commits (destroy change-coupling data); pair/mob work (git credits one committer);
multiple author aliases (fix with `.mailmap`). When social data is dubious, fall back
to the technical analyses (hotspots, coupling) — they're far less bias-sensitive.
(X-Rays ch.10.)

## AI-authored and mixed codebases (Crime Scene 2nd ed., ch.16)

There's **no special exemption** for AI-generated code — run the same hotspots and
complexity trends over it. Tornhill's point: the bottleneck was always *comprehension*,
not typing speed, so code generated faster without regard to understandability is
"more of a legacy-code generator." In mixed human/AI repos, "who holds the mental
model?" gets harder — which strengthens the case for knowledge-map/truck-factor
analysis. Practical: consider filtering commits from known AI-assistant bot accounts
before author-based social analysis, the same way you filter generated files from
hotspot rankings.

## Tool note
Everything above is reproducible with plain `git` + `cloc` (+ small scripts); see
[metrics.md](metrics.md). code-maat automates the same queries; rename-aware coupling
and time-decayed variants need commercial tooling (CodeScene) — say so when relevant.
