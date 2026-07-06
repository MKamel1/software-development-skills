---
name: design-review
description: Use when evaluating existing code, a diff, or a pull request for design quality — reviewing changes, judging whether a module is well-designed, spotting complexity, shallow modules, information leakage, or unclear naming, or deciding if code is too tactical. Applies A Philosophy of Software Design as a review rubric.
---

# Design Review

Judge existing code by the complexity it will impose on the *next* person to
read or change it — not by whether it works. Working code isn't enough; the
question is whether the design stays easy to change. This is the review side of
*A Philosophy of Software Design*.

Vocabulary (module / interface / seam / depth / leverage / locality):
[../references/vocabulary.md](../references/vocabulary.md) — the
`codebase-design` skill covers it in more depth. Use these terms exactly. The
complexity frame and the principles live in
[../references/principles.md](../references/principles.md);
the full symptom catalog is in [../references/red-flags.md](../references/red-flags.md).
Worked example: [../references/examples.md](../references/examples.md#design-review).

**For a thorough or reviewer-agent review, run the comprehensive rubric:**
[../references/review-checklist.md](../references/review-checklist.md) — seven
dimensions spanning all three source books (APoSD, Pragmatic Programmer, Design
of Design), designed to be run top to bottom against a diff, a PR, or another
agent's output. The quick procedure below is the fast path; the checklist is the
complete one.

## What you're looking for

Complexity shows up three ways — scan for each:

- **Change amplification** — would a simple conceptual change force edits in many places? A design decision duplicated across files is the tell.
- **Cognitive load** — how much must a reader hold in their head to work here safely? Count what they must *know*, not lines of code.
- **Unknown unknowns** (the worst) — is it non-obvious what you'd need to know or touch to change this safely? Hidden invariants, conventions you must already know, implicit ordering.

## Review procedure

1. **Frame the change** — tactical or strategic? A "get it working" patch that adds a little complexity everywhere is a tactical-tornado smell (Ch 3). Say so.
2. **Walk the red-flags catalog** against the diff — see [../references/red-flags.md](../references/red-flags.md). For each hit: name the flag, cite the line, propose the fix.
3. **Apply the deletion test** to any thin module: if you deleted it, does complexity vanish (it was a pass-through) or reappear across N callers (it earned its keep)?
4. **Check obviousness** (Ch 18) — could a new reader predict what this does without running it? If it needed a long explanation to you, it's Nonobvious Code.
5. **Cluster the findings** — several flags in one spot usually trace to *one* deeper design problem. Report the root cause, not every symptom.

## Report shape

Per finding: **flag → location → why it raises complexity → concrete fix.**
Lead with the highest-leverage design issue, not a list of nits. A red flag is a
prompt to find a better design, not an automatic defect — if a flag survives a
real design-it-twice pass, say why it's the least-bad option.

## Reviewing changes to existing code (Ch 16)

- **Stay strategic** — every change is a chance to improve the design, not just the smallest edit that works. Flag "just patch it" changes that entrench a bad structure.
- **Keep comments near the code they describe**, in the code (not the commit log), and check the diff for comments that no longer match.

## Siblings & when to hand off

- Found a module that needs redesigning, not just patching? → **module-design** (deepen it, design its interface twice).
- The smells are about coupling, duplication, or reversibility? → **designing-for-change**.
- The problem is upstream — wrong requirement, waterfall thinking, no conceptual integrity? → **design-process**.
- The problem is a name, a comment, or local obviousness while code is being written? → **inline-authoring**.
- Need the precise term for a seam/adapter/depth distinction? → **codebase-design**.

## Red flags in your own review

- You explained why a smell is "fine here" without sketching an alternative — do the design-it-twice pass first.
- You listed nits but missed that they share one root cause.
- You judged by "does it work" instead of "what will the next change cost."
