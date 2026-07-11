# Seams & Characterization Tests

The primary-source treatment of the **seam** — the term this repo's vocabulary
already uses ([vocabulary.md](vocabulary.md)) — plus how to safely change code you
didn't write. Distilled from Michael Feathers, *Working Effectively with Legacy
Code* (2004), which originated the concept. Shared across jobs: `module-design`
uses it to place a seam while designing; `codebase-audit` uses it to change what an
audit finds; [deepening.md](deepening.md) uses it for dependency handling.

## Seam

**A seam is a place where you can alter behavior without editing there** (Ch. 4).
Every seam has an **enabling point** — the *other* place (a constructor argument, a
build/classpath setting, a compiler flag) where you choose which behavior the seam
uses. A seam with no reachable enabling point is useless for testing; you need both.
The mindset shift: a program isn't a flat sheet of text you can only edit where you
read it — it's full of substitution points already there to exploit.

## Seam types (Ch. 4)

- **Object seam** *(the most useful in OO)* — a call like `cell.recalculate()` doesn't
  say which method body runs; polymorphism lets you vary the target from outside. It's
  a seam only if there's an enabling point to inject a different object/class without
  editing the calling method (a constructor/parameter, a subclass-and-override point).
  Creating and using the object in the same method with no injection is *not* a seam.
- **Link seam** — substitute a whole library/class at build/link time (or via the
  classpath) — e.g. link a stub graphics library for tests. Enabling point lives
  *outside* the source (build script), so make the test/production difference obvious.
  Best for near-pure "tell" APIs (graphics, hardware) where return values don't matter.
- **Preprocessing seam** *(C/C++ only)* — swap definitions via the macro preprocessor
  (`#ifdef TESTING`). Powerful but a last resort: the source diverges from what
  actually compiles.

Feathers' ranking: **object seams first**; link/preprocessing seams are useful but
harder to maintain because the substitution is less explicit.

## Sensing vs. separation (Ch. 3)

Two reasons to break a dependency, often needed together:
- **Sensing** — you can't otherwise *observe* an effect your code computes (it's
  buried in a collaborator you can't interrogate).
- **Separation** — you can't even *compile/run* the class in a test harness (its
  constructor drags in hardware, a DB, the whole app).

A **fake** (an object impersonating a collaborator) is the standard tool: extract a
narrow interface for the behavior invoked, depend on the interface, supply a fake in
tests. A **mock** is a fake that also asserts on the calls it received. A seam is
exactly where the fake gets substituted without editing the class under test.

## Characterization tests (Ch. 13) — how to change what an audit finds

A characterization test pins down **actual current behavior**, not a spec. It only
fails when behavior *changes* (which may or may not be a bug — that's fine). The
literal procedure:
1. Get the code into a harness.
2. Write an assertion you *know is wrong* (assert an absurd value).
3. Run it — the failure message tells you the actual behavior.
4. Change the assertion to expect that actual value.
5. Repeat for other inputs/branches.

If you find a bug while characterizing: if the system was never shipped, just fix it;
if it's live, **don't reflexively "fix" it** — something downstream may depend on the
current behavior. Mark it and investigate. Before a refactor, only the specific logic
you're about to move strictly needs characterizing — and bias toward tests that
exercise each branch and each **type conversion** along the path you'll touch (an
Extract Method can silently turn an `int` into a truncating `double`). This is the
safe on-ramp to changing a hotspot a `codebase-audit` flags.

## Top dependency-breaking techniques (Ch. 6, 25)

Name-and-purpose only — reach for the lightest that works:
- **Extract Interface** — pull the methods actually used into an interface; inject a
  fake. "One of the safest" (a missed step just fails to compile). Extract incrementally,
  driven by compiler errors, not all public methods up front.
- **Parameterize Constructor** — a constructor that `new`s a collaborator gets a
  parameter for it (old constructor kept as an overload) so tests inject a fake.
- **Sprout Method / Class** — write genuinely new behavior as a new, test-first method
  or class called from the old untested code, instead of splicing logic inline.
- **Wrap Method / Class** — rename the old method and add a new one with the old name
  that calls it plus new behavior (or the class-level decorator version) — for behavior
  that must run alongside existing behavior without tangling the implementations.
- **Introduce Instance Delegator / Static Setter** — restore an object seam around
  static/singleton "static cling."

## Restraint

Feathers treats every introduced seam as a deliberate, slightly-invasive incision to
enable a *specific current* test — "you often have to suspend your sense of aesthetics
a bit… there might be a scar" (Ch. 2) — not a default design move, and he extracts
interfaces incrementally rather than wholesale (Ch. 25). This is the same instinct as
this repo's rule: **one adapter is a hypothetical seam; two is a real one**
([vocabulary.md](vocabulary.md), [deepening.md](deepening.md)). Introduce a seam for
an actual need to sense or separate — not speculatively, in case a second
implementation shows up later.
