# Design Principles & The Complexity Frame

The shared conceptual layer for all three Philosophy-of-Software-Design skills.
Distilled from APoSD. Vocabulary (module / interface / seam / depth / leverage /
locality): [vocabulary.md](vocabulary.md) (and the `codebase-design` skill for
the full treatment).

## What we are fighting: complexity

**Complexity** = anything about a system's structure that makes it hard to
understand or change. It's what the reader experiences, not a property you can
feel while writing. It accumulates incrementally — no single change looks like
the problem, so you must *sweat the small stuff* (Ch 1–2).

### Three symptoms (how complexity shows up)

- **Change amplification** — a simple conceptual change forces edits in many
  places. Good design shrinks the code touched per decision.
- **Cognitive load** — how much a developer must hold in their head to make a
  change. Fewer lines can mean *more* load; optimize for what the reader must
  know, not line count.
- **Unknown unknowns** — it's not even obvious what you must know or change to
  do a task safely. This is the **worst** symptom: with the other two you at
  least know where to look. Good design makes systems **obvious**, killing
  unknown unknowns.

### Two causes (where complexity comes from)

- **Dependencies** — when a piece of code can't be understood or changed in
  isolation because it's tied to others. You can't eliminate dependencies, but
  design makes them simpler and more obvious.
- **Obscurity** — when important information isn't apparent (a vague name, an
  undocumented invariant, a convention you must already know). Fixed with better
  naming, well-placed comments, and consistency.

## Tactical vs strategic (Ch 3)

- **Tactical programming** optimizes for getting *this* feature working now.
  Each shortcut adds a little complexity; the "tactical tornado" ships fast and
  leaves a mess others clean up.
- **Strategic programming** treats *working code as insufficient* — the goal is
  a great design that stays easy to change. Invest ~10–20% extra continuously
  (small investments, not big rewrites) to fight complexity as you go.

This split is the lens for `design-review` (is this change tactical or
strategic?) and `inline-authoring` (am I taking a shortcut that adds
complexity?).

## The 15 design principles

1. **Complexity is incremental** — sweat the small stuff; every shortcut counts.
2. **Working code isn't enough** — a clean, changeable design is the real deliverable.
3. **Make continual small investments** to improve design (~10–20% overhead), not big-bang cleanups.
4. **Modules should be deep** — a lot of capability behind a small interface (see `codebase-design`).
5. **Design interfaces to make the common case simple**; push rare cases out of the way.
6. **A simple interface matters more than a simple implementation** — accept internal complexity to spare every caller.
7. **General-purpose modules are deeper** — "somewhat general" interfaces are usually simpler *and* more reusable than special-purpose ones.
8. **Separate general-purpose and special-purpose code** — don't let special cases pollute a general mechanism.
9. **Different layers should have different abstractions** — if adjacent layers look the same, one is probably a pass-through.
10. **Pull complexity downward** — it's better for the module (one implementer) to suffer complexity than every caller.
11. **Define errors (and special cases) out of existence** — redesign semantics so the error can't arise; beats handling it everywhere (see below).
12. **Design it twice** — your first interface idea is rarely the best; sketch two+ radically different ones and compare on depth/locality/seam.
13. **Comments should describe what isn't obvious from the code** — capture the *why* and the design intent the code can't (see `inline-authoring`).
14. **Design for ease of reading, not ease of writing** — code is read far more than written; obscurity is a defect.
15. **The increments of development should be abstractions, not features** — when adding a feature, invest in the right abstraction rather than bolting on.

## Define errors out of existence (Ch 10)

Exceptions are a major source of complexity. Four techniques, best-first:

1. **Define the error away** — redefine the API so the "error" is normal
   behavior (e.g. deleting a nonexistent key is a no-op; a substring past the
   end clamps rather than throwing). Fewest special cases for callers.
2. **Mask it** — handle the exception at a low level so higher levels never see
   it (e.g. transparent retry on a transient network blip).
3. **Aggregate handling** — catch many exceptions in one high-level place rather
   than at every call site.
4. **Just crash** — for truly unrecoverable errors, fail fast and clearly
   instead of threading recovery logic everywhere.

Fewer places that throw, and fewer exception types, means fewer special cases
callers must reason about.

## Pragmatic Programmer — design principles (Hunt & Thomas)

The design-relevant tips (the book has 100; debugging/testing/VCS/teams are left
to superpowers). Cited as PP Topic N.

- **ETC — Easier To Change** (Topic 8, Tip 14) — the master metric. *Good design
  is easier to change than bad design.* When unsure between two designs, pick the
  one that leaves more options open. Every other principle below is a special
  case of ETC.
- **DRY — Don't Repeat Yourself** (Topic 9) — every piece of *knowledge* has a
  single authoritative representation. It's about duplicated knowledge, not
  duplicated keystrokes: two identical code fragments that encode *different*
  decisions aren't a DRY violation; one rule expressed in two places is.
- **Orthogonality** (Topic 10) — eliminate effects between unrelated things.
  Changing one thing shouldn't ripple into another. Orthogonal components can be
  designed, tested, and changed in isolation.
- **Reversibility** (Topic 11) — there are no final decisions. Don't wire
  vendor/protocol/architecture choices through the code; keep them swappable.
- **Tracer bullets vs prototypes** (Topics 12–13) — to find the target, build a
  thin *working* end-to-end path (a tracer bullet) that you keep and grow; use a
  throwaway prototype only to learn a specific risky thing, then discard it.
  Don't confuse the two.
- **Design by Contract** (Topic 23) — state each routine's preconditions,
  postconditions, and invariants; be strict in what you accept and produce.
- **Crash early** (Topic 24) — a dead program does less damage than a crippled
  one; fail fast at the point of the broken assumption.
- **Assertive programming** (Topic 25) — use assertions to prove "impossible"
  things can't happen; don't use them for normal error handling.
- **Decoupling** (Topic 28) — *Tell, Don't Ask*; don't chain calls through
  objects you don't own (Law of Demeter); avoid global data. Decoupled code is
  easier to change.
- **Don't program by coincidence** (Topic 38) — program deliberately: rely only
  on documented behaviour, know *why* your code works, and prove assumptions.
- **Refactor early and often** (Topic 40) — treat design debt like a garden, not
  a construction site; refactor continuously in small steps.
- **Name well** (Topic 44) — names carry intent and expectations; rename the
  moment a name stops fitting. (Overlaps APoSD Ch 14.)

## Design of Design — process principles (Brooks)

How to *approach* a design problem, not just how to shape a module. Cited as
DoD Ch N. (The book's second half — dream tools, "great designers," and
house/hardware case studies — is not reproduced here; it's out of scope for a
coding agent.)

- **The rational/waterfall model is wrong and harmful** (Ch 2–3, 5) — design is
  not a linear march from complete requirements to implementation. You learn
  *while* building; real processes iterate, backtrack, and co-evolve the problem
  and the solution. Design for that feedback loop.
- **The hardest part is deciding what to design** (Ch 4) — requirements are the
  crux and they're always wrong or vague at first ("no one knows exactly what
  they want"). Surface the real need behind stated requirements; expect them to
  change.
- **Conceptual integrity is the paramount virtue** (Ch 12; and Brooks' earlier
  work) — a coherent design by one mind (or a tightly-aligned few) beats a pile
  of independently-good features. Unity of idea > sum of parts.
- **Constraints are friends** (Ch 11) — constraints don't cramp design, they
  focus it; they shrink the search space and often force the better idea. Seek
  and state them rather than resenting them.
- **Budget the scarce resource** (Ch 10) — identify the one resource that
  dominates (latency, memory, cost, human attention) and design around its
  budget explicitly.
- **User models: better wrong than vague** (Ch 9) — commit to an explicit,
  concrete model of who uses this and how, even if it's partly wrong; a vague
  model can't be tested or corrected.
- **Capture the design's trajectory and rationale** (Ch 16) — record not just
  *what* was decided but *why*, and what was rejected. Future changes need the
  reasons, not just the result.

## The master lens: Easier To Change

Across all three books the same idea recurs under different names — deep modules
and information hiding (APoSD), ETC and orthogonality and reversibility (PP),
conceptual integrity and reversible constraints (Brooks). When judging any
design decision, the unifying question is: **does this make the system easier to
understand and change, or harder?**
