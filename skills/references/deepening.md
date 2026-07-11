# Deepening

How to deepen a cluster of shallow modules safely, given its dependencies.
Uses the terms in [vocabulary.md](vocabulary.md) — **module**, **interface**,
**seam**, **depth**, **leverage**, **locality**. Acted on mainly by
`module-design`; `design-process` and `design-review` reach for it too.

This is the repo's own working version of the deepening procedure. The
`codebase-design` skill, if installed, covers the same ground with more depth
(diagrams, testability walkthroughs) — but everything you need to apply it is
here.

## Dependency categories

When assessing a candidate for deepening, classify its dependencies first — the
category determines how the deepened module is tested across its seam, and
whether a seam belongs at its external interface at all.

1. **In-process** — pure computation, in-memory state, no I/O. Always deepenable:
   merge the modules and test through the new interface directly. No adapter,
   no port.
2. **Local-substitutable** — dependencies with a local test stand-in (an
   in-memory database, a temp filesystem). Deepenable if the stand-in exists;
   the deepened module is tested with the stand-in running. The seam is
   internal — no port at the module's external interface.
3. **Remote but owned (ports & adapters)** — your own services across a network
   boundary. Define a **port** (interface) at the seam; the deep module owns the
   logic, the transport is injected as an **adapter**. Tests use an in-memory
   adapter; production uses an HTTP/gRPC/queue adapter. Shape of the
   recommendation: *"define a port at the seam, an HTTP adapter for production
   and an in-memory adapter for tests, so the logic stays in one deep module
   even though it's deployed across a network."*
4. **True external (mock)** — third-party services you don't control (payments,
   messaging). The deepened module takes the dependency as an injected port;
   tests provide a mock adapter.

## Seam discipline

- **One adapter is a hypothetical seam; two is a real one.** Don't introduce a
  port unless at least two adapters are justified (typically production + test).
  A single-adapter seam is just indirection. (This is the same rule stated in
  [vocabulary.md](vocabulary.md) — apply it here.)
- **Internal seams vs external seams.** A deep module can have internal seams
  (private to its implementation, used by its own tests) as well as the external
  seam at its interface. Don't expose an internal seam through the interface just
  because tests use it.

For the primary-source treatment of seam *types* (object / link / preprocessing),
sensing vs. separation, and the dependency-breaking techniques that introduce a seam,
see [seams.md](seams.md) (Feathers).

## Testing strategy: replace, don't layer

- Old unit tests on the shallow modules become waste once tests exist at the
  deepened module's interface — delete them.
- Write new tests at the deepened module's interface. **The interface is the
  test surface.**
- Assert on observable outcomes through the interface, not on internal state, so
  tests survive internal refactors. If a test has to change when the
  implementation changes, it was testing past the interface.
