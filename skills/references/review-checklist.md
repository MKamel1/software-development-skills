# Comprehensive Design-Review Checklist

The reviewer-agent rubric. Run this top to bottom to evaluate a design, a diff, a
PR, or another agent's output (e.g. work produced under `codebase-design` or a
superpowers workflow). It unifies all three source books; each item cites where
the full treatment lives.

- APoSD = *A Philosophy of Software Design* (Ousterhout)
- PP = *The Pragmatic Programmer* (Hunt & Thomas)
- DoD = *The Design of Design* (Brooks)

Symptom definitions: [red-flags.md](red-flags.md). Positive principles:
[principles.md](principles.md). Vocabulary: [vocabulary.md](vocabulary.md)
(and the `codebase-design` skill for the full treatment).

**Out of scope — defer, don't duplicate:** debugging technique, test-writing,
TDD mechanics, git/CI → the superpowers skills own these. This rubric judges
*design quality*, not process execution. Note where those are missing, but don't
re-teach them here.

## How to run it

1. Score each dimension: **OK / Concern / Blocker**, with a line + a fix.
2. Lead the report with the highest-leverage design issue, not nit order.
3. Cluster symptoms: several flags in one spot usually share one root cause —
   report the cause.
4. A red flag is a prompt to find a better design, not an automatic defect. If a
   flag survives a genuine design-it-twice pass, note it as accepted with the
   reason.
5. The master test behind every dimension (PP Tip 14): **does this make the
   system easier to understand and change, or harder?**

## Dimension 1 — Complexity & obviousness (APoSD)

- [ ] Would a simple conceptual change touch many places? (**change amplification**)
- [ ] How much must a reader know to work here safely? (**cognitive load**)
- [ ] Is it non-obvious what you'd need to know/touch to change this? (**unknown unknowns** — the worst)
- [ ] Could a fresh reader predict what the code does without running it? (obviousness, APoSD Ch 18) → else *Nonobvious Code*.

## Dimension 2 — Module depth & interfaces (APoSD + codebase-design)

- [ ] Is each module **deep** — lots of behaviour behind a small interface? Flag *Shallow Module*, *Pass-Through Method*.
- [ ] Does the interface make the **common case simple** and keep rare features out of the way? Flag *Overexposure*.
- [ ] Is complexity **pulled downward** (into the module) rather than pushed onto callers?
- [ ] Do adjacent layers have **different abstractions**?
- [ ] Passes the **deletion test** — would deleting this module make complexity vanish (pass-through) or reappear across callers (earns keep)?

## Dimension 3 — Information hiding & coupling (APoSD + PP)

- [ ] Is each design decision **hidden in one module**? Flag *Information Leakage* (a decision reflected in 2+ modules).
- [ ] Are module boundaries drawn around **information hiding**, not execution order? Flag *Temporal Decomposition*.
- [ ] **Orthogonal** — do unrelated things avoid affecting each other? (PP Topic 10)
- [ ] **Decoupled** — Tell-Don't-Ask honored, no train-wreck call chains (Law of Demeter), no avoidable global/shared mutable state? (PP Topics 28, 34)
- [ ] No **temporal coupling** — does correctness secretly depend on call order? (PP Topic 33)

## Dimension 4 — Duplication & change-friendliness (PP)

- [ ] **DRY** — is any piece of *knowledge* (rule, constant, schema) stated in more than one authoritative place? (PP Topic 9)
- [ ] **Reversibility** — are vendor/protocol/architecture choices isolated so they can be swapped? Flag *Irreversible decision baked in*. (PP Topic 11)
- [ ] **ETC** — of the plausible designs here, does this one keep the most future options open? (PP Tip 14)
- [ ] Any **broken windows** left unaddressed? (PP Topic 3)

## Dimension 5 — Correctness posture (APoSD + PP)

- [ ] Are errors **defined out of existence** where possible, rather than thrown-and-caught everywhere? (APoSD Ch 10)
- [ ] **Design by Contract** — are preconditions/postconditions/invariants clear at each interface? (PP Topic 23)
- [ ] Does it **crash early** on broken assumptions rather than limp on? (PP Topic 24)
- [ ] Any **programming by coincidence** — code that works but nobody can say *why*, relying on undocumented behaviour? (PP Topic 38)

## Dimension 6 — Naming, comments, consistency (APoSD + PP)

- [ ] Names precise and intuitive? Flag *Vague Name*, *Hard to Pick Name*. (APoSD Ch 14, PP Topic 44)
- [ ] Comments describe **what isn't obvious / the why**, never restate code? Flag *Comment Repeats Code*, *Implementation Documentation Contaminates Interface*. (APoSD Ch 13)
- [ ] **Consistent** with existing conventions; no second way to do an existing thing? (APoSD Ch 17)

## Dimension 7 — Design process & intent (DoD)

Applies when reviewing a *design* or a substantial change, not a one-line fix.

- [ ] Do the stated **requirements reflect the real need**, or are they disguised implementation choices? Flag *Requirements taken as solutions*. (DoD Ch 4, PP Topic 45)
- [ ] Was the design allowed to **iterate** with what building taught, or handed down waterfall-style? Flag *Waterfall thinking*. (DoD Ch 3, 5)
- [ ] Does the whole have **conceptual integrity** — one coherent model, not a patchwork? (DoD Ch 12, APoSD Ch 17)
- [ ] Is the **critical constrained resource** named and budgeted? (DoD Ch 10–11)
- [ ] Is the **design rationale** recorded — why this, what was rejected? Flag *Undocumented design rationale*. (DoD Ch 16)
- [ ] Was more than one interface **considered** (design-it-twice)? (APoSD Ch 11)

## Report shape

For each finding: **dimension → flag/principle → location → why it raises
complexity or lowers changeability → concrete fix.** End with an overall read:
is this design *easier or harder to change* than the alternative, and the single
most valuable thing to fix.
