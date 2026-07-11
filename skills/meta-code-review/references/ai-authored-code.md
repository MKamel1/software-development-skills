# Reviewing AI-Authored Code — the design-taste lens

A design-quality complement to [failure-modes.md](failure-modes.md) (which is the
empirical failure taxonomy). Where that file asks "is this factually wrong / insecure /
incomplete," this one asks "is the *design judgment* sound" — the axis on which strong
models fail most. Distilled by applying John Ousterhout's *A Philosophy of Software
Design* (2nd ed.) design lens to AI-authored code.

**Provenance note:** Ousterhout's text is about design in general, not AI. The mapping
to producer types below is this repo's synthesis, not his claim — but it's a clean fit,
because the smells he names are exactly the ones capable models over-produce.

## The one-line calibration

- **Strong/frontier AI** — failures are **taste failures, not correctness failures**:
  code is right at the call site but over-specialized, over-decomposed, and cluttered
  with the unimportant. It *looks* production-quality. Spend review budget on design
  judgment, not bug-hunting.
- **Weak/small AI and juniors** — the same surface smells *plus* real bugs in the
  specialized path, and they tend to **under**-decompose and **under**-comment (the
  opposite tics). See [failure-modes.md](failure-modes.md) for their mechanical failures.

## What to check (Ousterhout's lens, tied to producer)

- **Over-specialization / false abstraction** (APoSD Ch. 6) — a method shaped exactly
  around one call site (`backspace()` instead of a general `delete(range)`), or a
  wrapper whose plausible name suggests it hides something but whose callers must still
  read the body to trust it. Strong AI produces this readily — it names things fluently
  even when the decomposition hides nothing. Test (Ch. 6): *what's the simplest interface
  covering current needs? in how many call sites is this used?* One call site → red flag.
  (This is the design-review flag *Shallow Module* / *Pass-Through* seen in AI output —
  don't restate it, cite `design-review`'s red-flags.)
- **Over-decomposition / "conjoined functions"** (APoSD §9.8) — many tiny single-caller
  helpers that read fine alone but scatter one coherent piece of logic. Ousterhout:
  "first make functions deep, then short enough to read — don't sacrifice depth for
  length." **This is diagnostic of a *capable* model** (trained on Clean-Code style),
  not a weak one. Test: is this helper called from exactly one place and only
  understandable with its caller in mind? Then inline, don't document.
- **Special-case branching instead of redesign** (APoSD §6.8) — edge cases handled by
  piling up `if (x == null)` / `if (!hasSelection)` guards rather than a representation
  that removes the case (an *empty* selection as a real value). Strong AI guards each
  case correctly but doesn't ask whether the case should exist; weak AI/junior guards
  are more often incomplete or wrong.
- **Comment drift** (APoSD §12.6) — AI writes fluent, confident comments fast, and
  because it optimizes for plausible text rather than tracking truth across edits,
  comments silently fall out of sync with code. The junior risk is *under*-commenting;
  the AI risk is *trustworthy*-commenting — so verify comments against current behavior
  line by line, not just that comments are present. Also flag verbose clause-like
  identifiers doing a one-line comment's job.
- **Clutter vs missing information — "good taste"** (APoSD Ch. 21) — the two mistakes:
  (1) treating too much as important → cluttered interfaces (optional args, exposed
  config knobs 95% of callers never touch, defaults that should be computed silently);
  (2) failing to see something *is* important → hidden or omitted, creating unknown-
  unknowns. **Strong AI leans to (1)** (symmetric over-exposure — the taste failure);
  **weak AI / junior lean to (2)** (missing load-bearing state → latent bugs). A good
  general solution has *leverage* — would this same code solve a nearby, slightly
  different problem unchanged? If clearly not, suspect over-fit.

## How this changes the meta-review

When the code producer is a **strong** model, a review that only checked for bugs and
found none is **not** a clean bill of health — the likely defects are architectural
(over-specialization, over-decomposition, over-exposure), and a review that never
raised them has a coverage gap worth flagging in your "missed issues" section. When the
producer is **weak/junior**, weight the mechanical checks in [failure-modes.md](failure-modes.md)
first, and judge the review by whether it mentored on edge cases and testing rather than
nitpicking style.
