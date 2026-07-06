---
name: designing-for-change
description: Use when the concern is coupling and changeability — reducing dependencies between parts, removing duplication (DRY), keeping decisions reversible, breaking temporal coupling, or deciding whether components can change independently. Applies The Pragmatic Programmer's decoupling and DRY tips plus A Philosophy of Software Design's information hiding.
---

# Designing for Change

Optimize for the one thing all good design shares: **Easier To Change** (ETC,
PP Tip 14). Where `module-design` makes a single module deep, this skill governs
the *relationships between* parts — coupling, duplication, and reversibility —
so a change in one place doesn't ripple everywhere. Sources: *The Pragmatic
Programmer* (decoupling/DRY topics) and APoSD information hiding.

Full principles: [../references/principles.md](../references/principles.md).
Symptom catalog: [../references/red-flags.md](../references/red-flags.md).
Vocabulary (module/interface/seam/depth): the `codebase-design` skill.

## The master metric

**ETC — Easier To Change.** When two designs both work, pick the one that leaves
more options open. Every rule below is a special case of ETC.

## Remove duplicated knowledge (DRY)

**Every piece of knowledge has one authoritative representation** (PP Topic 9).

- DRY is about **knowledge, not keystrokes.** Two identical code fragments that
  encode *different* decisions are fine; one business rule expressed in two
  places is a violation even if the code looks different.
- Duplication hides in code, data schemas, docs, and comments that restate
  logic. When a fact changes, exactly one place should need editing.
- Don't over-apply: forcing unrelated things to share code to "avoid repetition"
  creates coupling — the opposite goal. Ask "is this the *same knowledge*?"

## Make components orthogonal

**Eliminate effects between unrelated things** (PP Topic 10). Orthogonal
components can be designed, tested, and changed in isolation. Test: *"if I change
this, what else must change?"* — a good answer is "nothing unrelated." This is
the same aim as APoSD **information hiding**: each module owns one design
decision (its secret); if a decision shows up in two modules, that's
*Information Leakage* — merge them or move the secret.

## Reduce coupling

- **Tell, Don't Ask** (PP Tip 45) — ask an object to do something, don't pull its
  state out and act on it from outside.
- **Don't chain calls through objects you don't own** (Law of Demeter, PP Topic
  28) — `a.getB().getC().doThing()` couples the caller to a whole structure.
  Flag: *train-wreck / Law-of-Demeter breach*.
- **Avoid global and shared mutable state** (PP Topics 28, 34) — shared state is
  incorrect state; it couples everything that touches it and breaks locality.
- **Prefer composition/interfaces over inheritance** (PP Topics 31, "Inheritance
  Tax") — deep inheritance hierarchies couple subclasses to superclass internals.

## Keep decisions reversible

**There are no final decisions** (PP Topic 11). Vendor, protocol, framework, and
architecture choices should sit behind an abstraction so they can be swapped.
Flag: *irreversible decision baked in*. This is where a **seam** (see
`codebase-design`) pays off — but remember: one adapter is a hypothetical seam,
two is a real one. Don't add indirection for a choice that will never change.

## Break temporal coupling

Correctness must not secretly depend on a required *order* of calls that the
interface doesn't express (PP Topic 33). Make ordering explicit in the interface,
or design it away so operations can be reordered or run concurrently. Flag:
*temporal coupling*.

## Siblings & when to hand off

- Making a single module deep / choosing its interface? → **module-design**.
- Approaching the overall design problem / requirements? → **design-process**.
- Reviewing existing code for these smells? → **design-review** (and [../references/review-checklist.md](../references/review-checklist.md)).
- Need the exact seam/adapter/dependency-category vocabulary? → **codebase-design**.

## Red flags in your own design

- The same rule/constant/schema lives in two places (DRY).
- Changing one component forces edits in an unrelated one (non-orthogonal).
- A caller reaches through a chain of objects it doesn't own (Demeter).
- A vendor/protocol choice is wired through many files (irreversible).
- Correctness depends on calling things in a specific undocumented order.
