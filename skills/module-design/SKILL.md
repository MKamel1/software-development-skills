---
name: module-design
description: Use when designing a module, class, function, or API before or while writing it — choosing an interface, deciding where a seam goes, splitting or merging responsibilities, handling errors, or making something deep and reusable. Applies A Philosophy of Software Design to up-front design.
---

# Module Design

Design **deep modules**: a lot of capability behind a small interface. The best
interface makes the common case trivial and hides complexity from every caller.
This is the up-front-design side of *A Philosophy of Software Design*.

Vocabulary (module / interface / seam / depth / leverage / locality):
[../references/vocabulary.md](../references/vocabulary.md) — the
`codebase-design` skill covers the same terms in more depth, and its
DEEPENING.md / DESIGN-IT-TWICE.md cover dependency categories and the
parallel-agent interface search. Don't redefine those here; use them.
Principles and the error techniques are in
[../references/principles.md](../references/principles.md). Worked example:
[../references/examples.md](../references/examples.md#module-design).

## The one working idea you need here

A module is **deep** when its interface (everything a caller must know: types,
invariants, ordering, error modes) is small relative to the behaviour it
delivers. **Shallow** = the interface is nearly as complex as the
implementation. Deep modules give callers **leverage** and maintainers
**locality**. Everything below serves depth.

## Design moves (in priority order)

1. **Make the common case simple** (Ch 4–5) — design the interface around the
   90% caller; push rare features to separate, optional entry points so they
   don't tax everyone (avoids *Overexposure*).
2. **Prefer a simple interface over a simple implementation** (Ch 8, "pull
   complexity downward") — if complexity must live somewhere, put it inside the
   module (one implementer pays) rather than in every caller.
3. **Hide the secret** (Ch 5) — each module should encapsulate one design
   decision (a format, an algorithm, a schema). If a decision shows up in two
   modules, that's *Information Leakage* — merge them or move the secret.
4. **Make it somewhat general-purpose** (Ch 6) — a slightly general interface
   ("insert/delete a range of text") is usually both simpler and more reusable
   than a special-purpose one ("backspace"). General is often *deeper*.
5. **Separate general from special** (Ch 6) — keep special-purpose logic out of
   a general mechanism (avoids *Special-General Mixture*).
6. **Give each layer a different abstraction** (Ch 7) — if a method mostly
   forwards to another with the same signature, it's a *Pass-Through Method*:
   remove the layer or make it earn a distinct abstraction.
7. **Together or apart?** (Ch 9) — combine things when they share information,
   are always used together, or overlap; separate when a reader can understand
   one without the other. Watch for *Conjoined Methods* (split at the wrong
   seam) and *Repetition* (missing shared abstraction).

## Choosing where the seam goes

The seam location is its own decision, separate from what sits behind it. Base
it on **information hiding**, not execution order — boundaries drawn around the
*sequence* of operations (read → process → write) are *Temporal Decomposition*.
For seam + dependency handling (in-process, local-substitutable, ports &
adapters, mock), use `codebase-design`'s DEEPENING.md. Rule of thumb: **one
adapter is a hypothetical seam; two is a real one** — don't add a port for a
single implementation.

## Handle errors by removing them (Ch 10)

Before adding a `throw`/`catch`, try to make the error not exist. Best-first:
**define it away** (make the "error" normal behaviour) → **mask** it low →
**aggregate** handling high → **just crash** if unrecoverable (crash early on a
broken assumption — a dead program does less damage than a crippled one, PP
Topic 24). Fewer throwing sites and fewer exception types = fewer special cases
for callers. State each interface's **contract** — preconditions,
postconditions, invariants (Design by Contract, PP Topic 23) — as part of the
interface, not buried in the body. Full techniques in
[../references/principles.md](../references/principles.md).

## Design it twice (Ch 11)

Your first interface is rarely the best. Sketch **two or more radically
different** interfaces (e.g. minimal vs flexible vs common-case-optimized),
then compare on depth, locality, and seam placement, and pick or hybridize. For
a chosen deepening candidate this is worth spinning up parallel sub-agents —
see `codebase-design`'s DESIGN-IT-TWICE.md.

## Siblings & when to hand off

- Still deciding *what* to build, or handling requirements/constraints? → **design-process**.
- The concern is coupling, duplication, or reversibility across modules? → **designing-for-change**.
- Code already exists and you're judging it? → **design-review**.
- Down to naming, comments, and local readability? → **inline-authoring**.
- Need the exact seam/adapter/dependency-category vocabulary and the
  parallel-agent design pattern? → **codebase-design**.

## Red flags in your own design

- You designed only one interface — do the design-it-twice pass.
- The interface grew a method "for one caller" — that's Overexposure creeping in.
- You added a `throw` before asking whether the error could be defined away.
- A new layer just forwards calls — Pass-Through; give it a real abstraction or delete it.
