# Design It Twice

Your first interface idea is rarely the best (APoSD Ch 11). When a module is
worth getting right — usually a chosen deepening candidate (see
[deepening.md](deepening.md)) — sketch two or more **radically different**
interfaces and compare, rather than committing to the first. Uses the terms in
[vocabulary.md](vocabulary.md) — **module**, **interface**, **seam**, **depth**,
**leverage**, **locality**. Acted on by `module-design` and `design-process`.

This is the repo's own working version. The `codebase-design` skill, if
installed, has a fuller treatment (including the parallel sub-agent variant);
this file is enough to run the procedure standalone.

## The lightweight version (default)

For most decisions, do it in your own head or notes:

1. Sketch **2–3 genuinely different** interfaces for the module — not three
   spellings of the same idea. Force real variety by giving each a different
   governing constraint (see the constraints below).
2. For each, write down: the entry points (types, methods, params) **plus** the
   invariants, ordering, and error modes a caller must know; a one-line usage
   example; and what it hides behind the seam.
3. Compare on **depth** (leverage per unit of interface), **locality** (where
   change concentrates), and **seam placement**. Pick the strongest, or
   hybridize if elements combine well. Be opinionated — the point is a decision,
   not a menu.

## The parallel version (for a high-stakes interface)

When the interface is consequential enough to justify the spend, explore the
alternatives in parallel sub-agents instead of serially:

1. **Frame the problem space** — the constraints any interface must satisfy, the
   dependencies and their category (see [deepening.md](deepening.md)), and a
   rough code sketch that grounds the constraints (not a proposal, just something
   concrete).
2. **Spawn 3+ sub-agents**, each with a brief to produce a *radically different*
   interface under a different governing constraint:
   - **Minimal** — 1–3 entry points max; maximize leverage per entry point.
   - **Flexible** — support many use cases and extension points.
   - **Common-case-optimized** — make the default caller's path trivial.
   - **Ports & adapters** (if it has cross-seam dependencies) — design around the
     seam.
   Each sub-agent outputs: the interface (with invariants/ordering/errors); a
   usage example; what it hides behind the seam; its dependency/adapter strategy
   (see [deepening.md](deepening.md)); and its trade-offs (where leverage is
   high, where it's thin).
3. **Present and compare** the designs, contrasting on depth, locality, and seam
   placement, then give a clear recommendation (or a hybrid). Keep vocabulary
   consistent so the designs are actually comparable.
