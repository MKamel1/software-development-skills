---
name: meta-code-review
description: Use when judging the quality of a code review that was itself produced by an AI agent or a person — verifying its findings against the actual code, checking for hallucinated or mis-cited findings, severity miscalibration, false positives, and real bugs it missed. Calibrates to who produced the code and the review (weak AI, strong AI, or junior developer). Applies the design-review rubric plus AI/junior-developer failure-mode research and Google's code-review standard.
---

# Meta Code Review

Judge whether a **code review can be trusted** — you're reviewing the review, not
re-reviewing the code. AI-produced (and junior) reviews fail in specific ways:
findings that don't hold up against the actual code, mis-cited line numbers,
severity inflated or deflated, style preferences dressed as defects, and real bugs
left unmentioned. Your output is a verdict on the review's soundness plus what it
got wrong.

Design rubric (for judging design-level findings) comes from the `design-review`
skill's `references/*.md` — invoke it to load them (base directory shown on load;
cite relative to that, not by bare path). Failure-mode taxonomy by producer:
[references/failure-modes.md](references/failure-modes.md).

## First: identify the producer, then route effort

AI code fails differently by capability, and juniors differently again. Identify
who produced the **code** and who produced the **review** (if unknown, hedge toward
the union). This decides where you spend verification budget — see
[references/failure-modes.md](references/failure-modes.md):

- **weak/small AI** → verify facts mechanically: resolve every import/package/API,
  re-run/trace execution, check numeric and boundary logic. Hallucination and
  fabrication dominate; assume low self-consistency.
- **strong/frontier AI** → verify *logic* and *intent*: trace real execution paths,
  check spec-intent (not just letter), and challenge whether each abstraction is
  necessary. Code looks production-quality even when subtly wrong.
- **junior developer** → verify edge cases, error handling, testing discipline, and
  architecture; **mentor** rather than nitpick style, and credit genuine fresh-eyes
  catches. They rarely hallucinate APIs.

## Procedure

1. **Verify each finding independently.** Re-derive it from the actual diff — don't
   accept the review's restated claim as evidence. Resolve every cited `file:line`;
   citation drift (a finding pointing at the wrong or nonexistent line, or claiming
   a symbol is missing when it exists) is a documented AI-review failure. If the
   review says something is untested/unhandled, grep for the test/handler first.
2. **Resolve external claims.** Any API/package/spec behavior the review asserts —
   resolve it against the real signature/registry, same rigor as hunting hallucinated
   packages in the code.
3. **Calibrate severity — both directions.** Does "critical" match real blast radius
   (security, data loss, crash), or is it an inflated nit? And the inverse: a real
   critical bug downgraded to "minor" or omitted because the reviewer never traced the
   runtime path.
4. **Detect over-reach / false positives.** Style-preference-as-defect; findings with
   no concrete failure attached. Note the review's profile: noisy-high-recall vs
   quiet-high-precision — and whether that matches what the team needs (developers
   stop trusting a review much past ~10–30% false positives).
5. **Scan for missed bugs.** Actively check the categories the review didn't mention —
   concurrency/races, error-handling completeness, security boundary crossings,
   resource cleanup, and **compliance with the original ticket/spec** (a distinct,
   commonly-missed category). AI-authored code shows elevated defect rates exactly
   here.
6. **Steelman the review.** (REQUIRED) Argue the strongest case that the review is
   right where you're inclined to fault it. Skipping this produces shallow meta-review.

Ground each judgment in facts over preference. When a finding (or a rebuttal to it)
is disputed, adjudicate by the ladder: **technical fact/data → the project's stated
standard → principled design rationale → codebase consistency → mere preference**
(Google's engineering-practices standard). A review comment that rests only on the
bottom rung is a nit, not a defect.

## Report shape

```
## Meta-review verdict: <trustworthy | trust with caveats | not trustworthy>

## Producer calibration
<who produced code / review, and where you therefore focused>

## Per-finding audit
- <review's finding> → <VERIFIED | UNVERIFIED | WRONG> → <your re-rated severity>
  → <evidence: the file:line you checked>
  (repeat per finding; group if many share a cause)

## Missed issues (the review didn't mention)
- <file:line> → <category: concurrency/error-handling/security/spec-compliance/…> → <why it matters>

## Steelman  (REQUIRED, non-empty)
<the review's strongest correct call, or the best defense of a finding you doubted>

## Overall
<is the review safe to act on as-is, act-on-with-fixes, or redo — and the single
most important correction>
```

## What NOT to do

- Don't re-review the code from scratch — judge the review's soundness. (Note real
  missed bugs in the dedicated section; don't turn the whole thing into a fresh review.)
- Don't accept AI self-attestation ("verified", "tested", a cited number) without an
  independent check.
- Don't inflate agreeable-sounding nits to look thorough, or soften a real miss.
- Don't judge a junior's review by a frontier-model bar — calibrate and mentor.

## Siblings & when to hand off

- Judging a *design* review specifically, or the design itself → **design-review**.
- The review is over a whole repo's architecture → **codebase-audit**.
- You're the one *receiving* review feedback on your own work → that's the
  superpowers `receiving-code-review` skill, not this one.

## Red flags in your own meta-review

- You agreed a finding was real without opening the cited line.
- You never named the producer, so you verified the wrong things.
- Your steelman section is empty — you didn't genuinely weigh that the review is right.
- You re-reviewed the code instead of judging the review.
