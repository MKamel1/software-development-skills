---
name: inline-authoring
description: Use while writing or editing code — choosing names, writing comments, deciding what to document, keeping style consistent, or making a piece of code obvious to the next reader. Applies A Philosophy of Software Design's naming, comments, and obviousness guidance line by line.
---

# Inline Authoring

Write for the next reader. Code is read far more often than written, so optimize
for ease of reading, not ease of writing. This is the moment-of-writing side of
*A Philosophy of Software Design*: names, comments, consistency, obviousness.

Vocabulary (module / interface / seam):
[../references/vocabulary.md](../references/vocabulary.md) (and the
`codebase-design` skill for the full treatment); principles and the red-flags
catalog are in
[../references/principles.md](../references/principles.md) and
[../references/red-flags.md](../references/red-flags.md). Worked example:
[../references/examples.md](../references/examples.md#inline-authoring).

## Naming (Ch 14; PP Topic 44)

A name should create an accurate **mental image** and be precise enough that a
reader rarely has to look up what it holds. **Rename the moment a name stops
fitting** (PP Tip 74, "Name Well; Rename When Needed") — a stale name is a
lie the reader trusts.

- **Precise + intuitive.** `blockCount` beats `count`; `newestChild` beats `node`. If the best you can do is `data`, `obj`, `process`, that's a *Vague Name*.
- **Can't find a good name?** (*Hard to Pick Name*) The entity is often ill-defined or does two things — reshape it, don't ship a fuzzy label.
- **Consistent** — the same concept gets the same name everywhere; different concepts get different names. Don't reuse one name for two meanings.

## Comments (Ch 12–13, 15)

**Comments describe what is *not* obvious from the code** — never restate it.

- **What & why, not how.** Capture intent, invariants, units, ranges, and the reasoning the code can't show. If the comment just re-says the code, it's *Comment Repeats Code* — delete or raise its abstraction.
- **Interface vs implementation comments.** Interface comments tell a *caller* what they need to use the thing (behaviour, args, errors) and must not leak internals — leaking them is *Implementation Documentation Contaminates Interface*. Implementation comments explain the tricky *how/why* inside the body.
- **Different abstraction than the code.** A good comment says something the code doesn't: higher-level ("what/why") for interfaces, precise detail (units, invariants) where the code is terse.
- **Write comments first** (Ch 15) — writing the interface comment before the body is a design check: if it's hard to describe, the interface is too complex (*Hard to Describe*). Fix the design, not the wording.
- **Keep them near the code** and update them in the same diff (Ch 16). A comment that no longer matches is worse than none.

## Consistency (Ch 17)

Similar things done in similar ways lower cognitive load and kill unknown
unknowns. Follow the conventions already in the file/codebase — names,
formatting, error handling, patterns — even if you'd have chosen differently.
When you establish a new convention, apply it everywhere. Don't introduce a
second way to do something that already has one.

## Obviousness (Ch 18) — the acceptance test

Before you move on, ask: **could a reader who's never seen this predict what it
does and why, without running it?** If it took *you* a paragraph to explain,
it's *Nonobvious Code*. Make it obvious by:

- better names and a clarifying comment for genuine intent,
- avoiding surprises (hidden side effects, misleading names, magic values),
- restructuring when no comment can rescue it.

Obscurity is a defect, not a style preference.

**Program deliberately, not by coincidence** (PP Topic 38). If code "works" but
you can't say *why* — it relies on lucky timing, an undocumented side effect, or
an unproven assumption — that's a latent bug, not working code. Rely only on
documented behaviour, and prove assumptions before building on them.

## Siblings & when to hand off

- The problem is bigger than a name/comment — the *interface* is wrong? → **module-design**.
- Reviewing someone else's finished change against these standards? → **design-review**.
- Need the exact term for a seam/adapter/depth idea? → **codebase-design**.

## Red flags in your own writing

- A comment that restates the line above it — delete or lift it.
- You reached for `data`/`tmp`/`obj`/`doIt` — name the actual thing.
- You couldn't name it or had to write a long doc — the entity, not the label, is the problem.
- You added a second way to do something the codebase already does one way.
