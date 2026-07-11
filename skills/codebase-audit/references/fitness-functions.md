# Fitness Functions — turn audit findings into guardrails

How to stop an audit finding from rotting back in. Distilled from Neal Ford, Rebecca
Parsons & Patrick Kua, *Building Evolutionary Architectures* (2017). Used by
`codebase-audit`: a finding is a one-time snapshot; a fitness function is the durable,
CI-runnable check that keeps it from regressing.

## Definition

An **architectural fitness function** is an objective, executable assessment of some
architectural characteristic (Ch. 2). A dependency cycle you removed, a layering rule
you restored, a complexity ceiling you set — each becomes a fitness function so the
build fails the next time someone violates it, instead of the next audit re-finding it.

**Core rule for the audit: a structural finding isn't "done" until it has a
corresponding fitness function checked into CI.** The finding is the human-readable
symptom; the fitness function is the artifact that lasts.

## Finding → fitness function (the recipe, Ch. 2–4, 6)

1. **Name the characteristic at risk** — which "-ility" or structural rule (changeability
   for a cycle; layering integrity for a dependency-direction breach).
2. **Make the rule explicit as data, not prose** — an allowed module-dependency graph,
   or a "zero cycles" invariant. Explicit beats a coding-standards doc: "developers
   have a hard time remembering bureaucratic policies in the heat of coding" (Ch. 4).
3. **Pick the category** — almost always **atomic + triggered** for structural findings
   (a unit test that runs in CI on every change).
4. **Automate and wire into the pipeline** so it runs every time the system changes, not
   only when someone remembers to re-audit.
5. **Classify Key / Relevant / Not-Relevant** (Ch. 2) so you don't turn every trivial
   finding into a permanent CI gate — reserve gates for things that shape design.
6. **Give it an owner and a review cadence** (revisit at least yearly as the system
   evolves).

## Taxonomy (Ch. 2) — tag each function

- **Atomic vs holistic** — one aspect in isolation (a single dependency-direction test)
  vs several together to catch emergent conflicts (caching added for speed breaks a
  freshness rule needed for security). Most audit-derived checks are atomic.
- **Triggered vs continual** — runs on a commit/build vs runs constantly against the
  live system. Static structural checks are triggered; latency/resiliency want continual
  monitors (out of scope for a static repo audit — flag them as "needs a continual check").
- **Static vs dynamic** — fixed threshold (complexity ≤ N) vs one that shifts with
  context. Start static.
- **Temporal** — a check that force-fails on a library upgrade so a workaround gets
  re-evaluated. Good for "this shim exists because of version X."

## Codeable shapes (Ch. 3–4) — what to scaffold

- **Layering / dependency-direction rule** — declare the allowed module graph as data,
  then assert the real graph matches (ArchUnit `layeredArchitecture()`, import-linter
  contracts, dependency-cruiser rules). Turns a "layering violation" finding into a
  build failure the moment an illegal import creeps in. Detection heuristics:
  [dependency-layering.md](dependency-layering.md).
- **Cycle check** — run a dependency-cycle analyzer (SCC over the module graph), assert
  zero cycles. Ford's exact guidance: once you've done the laborious work of removing
  cycles, put the zero-cycles test in place to guard against their return (Ch. 4).
- **Complexity ceiling** — a metrics check that fails the build if any function exceeds
  a cyclomatic/indentation threshold; baseline against the file's own history where
  possible (see [behavioral-analysis.md](behavioral-analysis.md)).
- **Coupling / instability bound** — compute Ca/Ce (fan-in/out) per module, fail if a
  module's instability crosses a set bound or a god-module signature appears.

## Appropriate coupling, not zero coupling (Ch. 4–7)

The goal is coupling that matches the problem's natural granularity, not its
elimination. Before flagging coupling as a defect, ask whether it's *appropriate* for
the domain — a heavily transactional system is legitimately more coupled than an
independent one; forcing microservice-level decoupling onto a transactional domain is
itself a design error. Two useful ideas:
- **Architectural quantum** — the smallest independently deployable/testable unit with
  high cohesion. It sets the lower bound on how incrementally the system can change; an
  artificially large quantum (from over-coupling) is a real audit finding.
- **Transactional coupling** binds units together invisibly to import/call analysis
  ("a strong nuclear force"). A structural audit that only measures code coupling will
  miss it — call that out rather than declaring a system more decomposable than it is.
- **Reuse-as-coupling** — shared code across independently-evolving domains raises
  coupling and can be worse than duplication ("the more reusable, the less usable").
  Don't reflexively praise a cross-domain shared model as DRY. (This is the *acceptable
  redundancy* principle from the other side: duplication across genuinely different
  jobs can be correct.)

## Connascence

Not in this book — its coupling vocabulary is afferent/efferent coupling + architectural
quantum. If a connascence taxonomy is wanted later, it needs a different source.
