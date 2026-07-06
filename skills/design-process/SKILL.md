---
name: design-process
description: Use when approaching a design problem or task before the modules are known — figuring out what to build, turning vague requirements into a plan, deciding how to sequence a build, handling constraints, or capturing why a design choice was made. Applies The Design of Design (Brooks) and The Pragmatic Programmer's design-attitude tips.
---

# Design Process

How to *approach* a design problem — the moment after "build X" and before you
know the modules. The other skills shape a module once you know what it should
do; this one is about deciding what to do and how to proceed. Sources: Brooks'
*The Design of Design* (Ch 1–16) and *The Pragmatic Programmer*'s design tips.

Full principles: [../references/principles.md](../references/principles.md).
Vocabulary: the `codebase-design` skill.

## The core stance

**Design is iterative, not linear.** The waterfall/rational model — gather
complete requirements, then design, then build — is "wrong and harmful" (DoD
Ch 3). You learn *while* building; the problem and the solution co-evolve. So
plan to iterate, keep decisions reversible, and don't over-commit before you've
learned. The master test for any choice: **does it make the system easier to
change?** (PP Tip 14).

## Approaching the problem

1. **Find the real requirement, not the stated one** (DoD Ch 4; PP Topics 45).
   No one knows exactly what they want, and stated "requirements" are often
   disguised implementation choices. Dig for the underlying need. Requirements
   are learned in a feedback loop — expect them to change, and design so they can.
2. **Commit to an explicit user model, even if partly wrong** (DoD Ch 9) —
   "better wrong than vague." A concrete model of who uses this and how can be
   tested and corrected; a vague one can't.
3. **Seek the constraints — they're friends** (DoD Ch 11). Constraints shrink
   the search space and often force the better idea. State them explicitly
   rather than resenting them.
4. **Name and budget the scarce resource** (DoD Ch 10). Identify the one
   resource that dominates (latency, memory, cost, human attention) and design
   around its budget.

## Proceeding to build

5. **Fire a tracer bullet, don't (only) prototype** (PP Topics 12–13). Build a
   thin *working* end-to-end path first, then grow it — it validates the whole
   architecture and you keep it. Use a throwaway prototype only to answer one
   risky question, then discard it. Don't confuse the two: tracer code is kept,
   prototype code is thrown away.
6. **Build end-to-end, in small reversible steps** (PP Tips 42, 68). Keep
   decisions reversible ("there are no final decisions", PP Topic 11) so what you
   learn while building can change the design.
7. **Design it twice** (APoSD Ch 11) — sketch two+ radically different
   approaches before committing; your first idea is rarely best. For interface
   alternatives use `codebase-design`'s DESIGN-IT-TWICE.md.

## Guard the whole

8. **Protect conceptual integrity** (DoD Ch 12; APoSD Ch 17) — one coherent
   idea, held by one mind or a tightly-aligned few, beats a pile of individually
   good but unrelated features. Consistency of *concept* is the paramount virtue.
9. **Capture the rationale** (DoD Ch 16) — record not just what you decided but
   *why*, and what you rejected. Future changes need the reasons. Put it where
   the next reader will find it (a design note, an ADR, a comment near the seam).

## Siblings & when to hand off

- Now designing a specific module/interface? → **module-design**.
- Designing for change/decoupling specifically? → **designing-for-change**.
- Reviewing a design someone already produced? → **design-review** (and [../references/review-checklist.md](../references/review-checklist.md)).
- Executing a plan / TDD / debugging? → the superpowers skills own process execution; this skill is about design judgment, not running the build.

## Red flags in your own process

- You started designing modules before pinning the real requirement.
- You treated a stated requirement as fixed truth (PP: no one knows what they want).
- You planned a linear "design then build" with no path for what building teaches (waterfall thinking).
- You can't name the one scarce resource the design must respect.
- You committed to a design without a second alternative or a recorded reason.
