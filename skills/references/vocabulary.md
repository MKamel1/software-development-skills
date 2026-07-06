# Vocabulary

A concise, self-contained gloss of the six terms the five skills in this repo
use constantly. This is the working version referenced by every `SKILL.md` and
by `red-flags.md`, `principles.md`, and `review-checklist.md` — enough to use
those documents standalone. The `codebase-design` skill (a separate,
pre-existing skill, not part of this repo) covers the same terms in more depth,
with diagrams, testability guidance, and the deepening/design-it-twice
procedures — read it there if it's installed.

Use these terms exactly; don't substitute "component," "service," "API," or
"boundary" — consistent language is the point.

- **Module** — anything with an interface and an implementation: a function,
  class, package, or tier-spanning slice. Scale-agnostic on purpose.
- **Interface** — everything a caller must know to use the module correctly:
  not just the type signature, but invariants, ordering constraints, error
  modes, required configuration, performance characteristics.
- **Depth** — leverage at the interface: how much behaviour a caller can get
  per unit of interface they have to learn. **Deep** = small interface, large
  implementation. **Shallow** = the interface is nearly as complex as the
  implementation.
- **Seam** _(Michael Feathers)_ — the place where you can alter behaviour
  without editing there; where a module's interface lives. Choosing where the
  seam goes is a separate decision from what sits behind it.
- **Leverage** — what callers get from depth: more capability per unit of
  interface learned.
- **Locality** — what maintainers get from depth: change, bugs, and knowledge
  concentrate in one place instead of spreading across every caller.

## Two working rules used throughout

- **The deletion test.** Imagine deleting the module. If complexity vanishes,
  it was a pass-through. If complexity reappears across N callers, it was
  earning its keep.
- **One adapter is a hypothetical seam; two is a real one.** Don't introduce a
  seam (an abstraction, a port, an interface) unless something actually varies
  across it today. A single implementation behind an interface is speculative
  indirection, not a design win.
