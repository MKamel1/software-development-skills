# New Skills — Research & Design (codebase-audit, meta-code-review)

Phase 9 of the review-findings work. Grounded in three parallel research passes
(trusted GitHub skills; AI-code failure modes; repo-scale audit methods). This
file is the design rationale (DoD Ch 16) the SKILL.md files will be built from.

Distinctive requirement driving both skills: they must review work produced by
**weak/small AI agents, strong/big AI agents, and junior developers** — three
different failure profiles, so the skills calibrate to the producer.

## What the research established

### Skill-authoring conventions to follow (from obra/superpowers, anthropics/skills, exemplar review skills)
- Frontmatter = `name` + `description`; description states **triggering only**
  ("Use when…"), keyword-rich, never the workflow.
- Progressive disclosure: lean SKILL.md (<~500 words core), detail pushed to
  `references/*.md` loaded only when needed.
- Standard shape: Overview → When to use → core procedure → red-flags/checklist
  table → **fixed, capped, triaged report template** → what NOT to do → siblings.
- Findings must be **`file:line`-cited**, never vague.
- Two independent axes: **severity × confidence** (not one blended score).
- A **required "looks bad but actually fine" / steelman** section — the strongest
  anti-shallow-analysis device found (tech-debt-skill); an empty one is itself a
  failure.
- State explicitly **what NOT to review** (don't redo linting; don't redo the
  original review).
- Repo-scale audits **parallelize by module via subagents**, then dedupe/synthesize.

Exemplars worth stealing from: `ksimback/tech-debt-skill` (audit structure +
steelman section), `ahmed-aisyst/architects-review-protocol` (map→review→synthesize
+ capped report), `awesome-skills/code-review-skill` (severity taxonomy + "what not
to review"), `mattpocock/skills/improve-codebase-architecture` (deletion test +
Strength badge), Google `eng-practices` (the normative review standard),
Augment Code's AI-review write-up (precision/recall as an explicit design choice).

### AI-code failure taxonomy, by producer (from arXiv + tooling benchmarks)
| Failure mode | Worst in | Cheap detection |
|---|---|---|
| Package/API hallucination | weak AI (21.7% vs 5.2% commercial, USENIX'25) | resolve every new import against lockfile/registry; grep symbols against real exports |
| Insecure patterns | all (~40% Copilot, S&P'22) | run SAST/secret-scan; don't trust "no security issues" prose |
| Plausible-but-wrong logic | strong AI (AI PRs 1.7× more issues, CodeRabbit) | trace one concrete input per branch by hand |
| Incomplete-but-looks-done (stubs/TODO wiring) | weak+strong AI | diff acceptance criteria vs what's wired; grep TODO/stub/NotImplemented |
| Over-engineering (1-impl abstractions) | strong AI | count abstraction layers vs concrete callers |
| Under-engineering / missing edge cases | AI + junior | enumerate senior-expected edge cases, check each |
| Fault mislocalization (symptom file, not root) | weak-mid AI | check sibling callers for same bug |
| Silent constraint/spec violation | all AI, worse long-context | re-check diff vs original spec line by line |
| Programming by coincidence | junior + AI | vary an input not in the prompt; ask "why does this work" |
| Fabrication (invented numbers/behavior/citations) | weak AI | independently verify any "verified/tested" claim |
| Quantization/numeric artifacts | small/quantized models | extra scrutiny on numeric/boundary logic |

Producer calibration (route review effort):
- **weak AI** → verify facts/imports/execution mechanically; assume low self-consistency.
- **strong AI** → verify logic, spec-*intent*, and necessity of abstractions; code
  looks production-quality even when subtly wrong.
- **junior dev** → verify edge cases/testing/architecture; **mentor** (don't nitpick
  style); value their fresh-eyes catches.

### Meta-review checks (judging a review, not the code)
Finding verification (re-derive independently), citation resolution (lines/files
actually match — citation drift is a real LLM-review failure), severity calibration
(inflation *and* deflation), false-positive/over-reach (style-nit-as-defect; is it
noisy-high-recall or quiet-high-precision, and does that match the team's tolerance —
devs distrust >~10-30% FP), missed-bug scan (concurrency, error handling, security
boundaries, resource cleanup, spec compliance), reasoning presence (assertion vs
explained finding correlates with reliability).

### Repo-scale audit method (executable with git/grep/glob)
Hotspots = **churn × complexity** (Tornhill/CodeScene; LOC as the cheap complexity
proxy). Change coupling = files co-changing across module boundaries. Dependency
analysis = import-graph cycles, layer-direction violations, instability
`I = Ce/(Ce+Ca)`. Prioritize hotspot-first (the "1-2% of code drives ~70% of work"
claim). Concrete one-liners captured in the skill's `references/metrics.md`.

## Proposed design

### `skills/codebase-audit/`
As-built, repo-scale architecture review (fills the gap: `design-review` is
diff/module-scoped; `principal-design-reviewer` refuses finished code).
Procedure: (0) scope/baseline — is this a 2-week AI scaffold or 5-year legacy?
(1) module inventory; (2) churn; (3) complexity proxy; (4) hotspots = churn×complexity;
(5) change coupling across boundaries; (6) dependency/layering/cycles/instability;
(7) prioritize hotspot-first (4 tiers); (8) report by the seven review-checklist
dimensions at repo scale. Big repos → parallel Sonnet subagents per module, then
synthesize. Report: exec summary → scope/baseline → hotspots (file:line, dimension,
severity×confidence) → structural findings → **required "looks bad but fine"** →
prioritized fix list → open questions. Reference file: `references/metrics.md`
(exact git/grep one-liners). Composes with `design-review` (per-module depth) and
`designing-for-change` (coupling).

### `skills/meta-code-review/`
Judges the trustworthiness of a code review (esp. AI-produced), calibrated to the
producer. Does **not** redo the review. Procedure: (1) identify producer of code
*and* of the review → route effort per calibration model; (2) verify each finding
independently + resolve citations; (3) resolve external API/package claims
(hallucination check); (4) severity calibration (both directions); (5)
false-positive/over-reach; (6) missed-bug scan of the uncovered categories; (7)
producer-calibration note; (8) **required "steelman the review"** section. Report:
verdict (trustworthy / trust-with-caveats / not-trustworthy) → per-finding audit
(verified/unverified/wrong, re-rated) → missed issues → calibration note → steelman
→ overall. Reference file: `references/failure-modes.md` (the taxonomy above, by
producer). The existing red-team agents (`review-challenger`, `change-minimizer`)
can later cite this skill instead of embedding their own rubric.

### Framing
These are the repo's **review/audit skills**, a family alongside `design-review`
(which stays the diff/module reviewer). Update the "five skills" prose to name the
review/audit additions rather than inflate a fragile count.

## Book recommendation (honest read)
- **Only one book is genuinely worth buying: Adam Tornhill, *Software Design X-Rays*
  (2018) or *Your Code as a Crime Scene* (2nd ed. 2024).** It's the primary source
  for churn×complexity hotspots and change coupling — the backbone of
  `codebase-audit` — and its method is git-log-based (agent-executable). The research
  pass already extracted the working formulas/thresholds, so a strong skill is
  buildable **without** it; the book would add case-study nuance and calibration
  depth if you want the audit skill to be best-in-class.
- **No purchase needed** for the rest: Google `eng-practices` (free, the review
  standard for `meta-code-review`) and the AI-failure-mode arXiv papers (free) cover
  the theory; APoSD / Pragmatic Programmer are already distilled in this repo.

## Book acquisition list (buy → drop in a folder → distilled in a later phase)

Decision: v1 of both skills is built now from the extracted research; these books
feed a later depth-enrichment pass, not the initial authoring.

**Tier 1 — buy (highest value, unique content):**
1. **Adam Tornhill — *Software Design X-Rays: Fix Technical Debt with Behavioral
   Code Analysis*** (Pragmatic Bookshelf, 2018). The core source for
   `codebase-audit`: hotspots (churn × complexity), change/temporal coupling,
   debt prioritization, the exact analyses to run over git history.
2. **Adam Tornhill — *Your Code as a Crime Scene*, 2nd ed.** (Pragmatic Bookshelf,
   2024). Updated companion; more on behavioral/team analysis and current (AI-era)
   framing. ~40% overlap with X-Rays — if buying only one, get X-Rays for the
   metrics; get both if you want the fullest forensic depth.

**Tier 2 — worth it for depth (optional):**
3. **Neal Ford, Rebecca Parsons, Patrick Kua — *Building Evolutionary
   Architectures*, 2nd ed.** (O'Reilly, 2022). Fitness functions → turning
   `codebase-audit` findings into repeatable checks.
4. **Michael Feathers — *Working Effectively with Legacy Code*** (Prentice Hall,
   2004). Seams (already our vocabulary), characterization tests — deepens the
   seam/deepening material and the "safely change what the audit finds" angle.
5. **Robert C. Martin — *Clean Architecture*** (Prentice Hall, 2017). Dependency
   rule, layering, stable-dependencies/abstractions — vocabulary for
   `codebase-audit`'s dependency-direction judgments. (Some overlap with existing
   knowledge; medium value.)

**Do NOT buy (free or already covered):**
- Google Engineering Practices — free at google.github.io/eng-practices (the
  standard `meta-code-review` measures against; distilled from the free source).
- AI-code failure-mode research — free arXiv papers (listed above).
- *A Philosophy of Software Design* / *The Pragmatic Programmer* — already distilled
  in this repo (the 2nd-ed APoSD's AI-code section is a low-priority nice-to-have).
- *The Art of Readable Code* — mostly overlaps `inline-authoring`; skip.
