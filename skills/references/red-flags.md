# Red Flags Catalog

Symptoms that a piece of code is probably more complex than it needs to be.
Distilled from all three books — *A Philosophy of Software Design* (APoSD) for
the first table, *The Pragmatic Programmer* (PP) and *The Design of Design*
(DoD) for the second. When you see one, **stop and look for an alternate design
that removes it** — don't rationalize it away. Each flag lists the fix and which
skill acts on it.

Vocabulary (module / interface / seam / depth / leverage / locality):
[vocabulary.md](vocabulary.md) (and the `codebase-design` skill for the full
treatment); this catalog uses those terms.

| # | Red flag | You'll notice… | The fix | Acted on by |
|---|----------|----------------|---------|-------------|
| 1 | **Shallow Module** | the interface is nearly as complicated as the implementation — you learn as much to *call* it as it does for you | Deepen: hide more behind a smaller interface, or merge it into a caller if it adds nothing (deletion test). | design-review, module-design |
| 2 | **Information Leakage** | one design decision (a format, a protocol, a schema) shows up in two+ modules, so they must change together | Encapsulate the decision in one module; or merge the modules that share the secret. | design-review, module-design |
| 3 | **Temporal Decomposition** | module boundaries follow the *order operations run* (read, then process, then write) instead of what knowledge each hides | Re-draw boundaries around information/secrets, not execution sequence. | module-design |
| 4 | **Overexposure** | callers must understand a rarely-used feature just to do the common thing | Make the common case trivial; move rare features behind separate, optional entry points. | module-design |
| 5 | **Pass-Through Method** | a method does almost nothing but forward its args to another method with a near-identical signature | Remove the layer, expose the inner method directly, or give this layer a genuinely different abstraction. | design-review, module-design |
| 6 | **Repetition** | the same nontrivial code appears again and again | Factor it into one place; the repetition means the right abstraction hasn't been found yet. | design-review, inline-authoring |
| 7 | **Special-General Mixture** | special-purpose code is interleaved with a general-purpose mechanism | Separate them: keep the mechanism general, push the special case to a caller/layer above. | module-design |
| 8 | **Conjoined Methods** | you can't understand one method without reading another, and vice-versa | Redesign the seam so each can be read on its own; they were split at the wrong place. | design-review, module-design |
| 9 | **Comment Repeats Code** | the comment restates exactly what the code already says | Delete it, or raise its abstraction to say *what/why*, not *how*. | inline-authoring |
| 10 | **Implementation Documentation Contaminates Interface** | an interface comment leaks details only the implementer needs | Split: interface comments describe usage; implementation comments stay inside the body. | inline-authoring |
| 11 | **Vague Name** | a name (`data`, `obj`, `count`, `process`) conveys little; you keep checking what it holds | Rename to something precise and intuitive that forms a clear mental image. | inline-authoring |
| 12 | **Hard to Pick Name** | you can't find a precise, intuitive name for an entity | Often the entity is ill-defined or does two things — reshape it, don't settle for a fuzzy name. | inline-authoring, module-design |
| 13 | **Hard to Describe** | a variable/method needs a long comment to document it completely | The interface is too complex or the entity has muddled responsibility — simplify it. | inline-authoring, module-design |
| 14 | **Nonobvious Code** | a reader can't quickly tell what a piece of code does or means | Make it obvious: better names, a clarifying comment, or restructure. If it needs a long explanation, redesign. | design-review, inline-authoring |

## Additional red flags (Pragmatic Programmer & Design of Design)

These extend the catalog so a reviewer has one comprehensive symptom list.
PP = *The Pragmatic Programmer* (topic/tip); DoD = *The Design of Design* (ch).

| # | Red flag | You'll notice… | The fix | Acted on by |
|---|----------|----------------|---------|-------------|
| 15 | **Duplicated knowledge (DRY violation)** | the same fact — a rule, a constant, a schema, a bit of logic — is stated in two places, so they can drift | Give every piece of knowledge one authoritative home (PP Topic 9). Note: duplicated *code* isn't always a DRY violation; duplicated *knowledge* is. | design-review, designing-for-change |
| 16 | **Non-orthogonal / coupled change** | a change in one area breaks or forces edits in an unrelated area | Eliminate effects between unrelated things; separate concerns (PP Topic 10, "Eliminate Effects Between Unrelated Things"). | design-review, designing-for-change |
| 17 | **Train-wreck / Law-of-Demeter breach** | chained calls reach through objects (`a.getB().getC().doThing()`), coupling the caller to a deep structure | Tell, don't ask; don't chain method calls across ownership boundaries (PP Topic 28). | design-review, designing-for-change |
| 18 | **Global / shared mutable state** | behaviour depends on data any code can reach or mutate; bugs appear at a distance | Avoid global data; pass what's needed; treat shared state as incorrect state (PP Topics 28, 34). | design-review, designing-for-change |
| 19 | **Programming by coincidence** | code works but nobody can say *why*; it relies on untested assumptions, lucky timing, or undocumented behaviour | Program deliberately — know why it works, rely on documented behaviour, prove assumptions (PP Topic 38). | design-review, inline-authoring |
| 20 | **Broken window** | a known-bad bit of code/design is left "for now"; quality visibly slipping | Fix it or explicitly board it up; entropy compounds once one is tolerated (PP Topic 3). | design-review |
| 21 | **Irreversible decision baked in** | a vendor, protocol, or architectural choice is wired through the code so it can't be swapped | Keep decisions reversible; isolate the choice behind an abstraction (PP Topic 11, "There Are No Final Decisions"). | designing-for-change, module-design |
| 22 | **Temporal coupling** | correctness depends on a hidden required *order* of calls not expressed in the interface | Make ordering explicit or unnecessary; design so operations can be reordered/parallelized (PP Topic 33). | designing-for-change, module-design |
| 23 | **Requirements taken as solutions** | a stated "requirement" is really a chosen implementation; the real need was never surfaced | Dig for the underlying need; requirements are learned in a feedback loop, and nobody knows exactly what they want (PP Topics 45; DoD Ch 4). | design-process |
| 24 | **Waterfall thinking** | design treated as a finished, up-front artifact handed off for implementation, with no path for what's learned while building | Iterate: designs evolve as you build; the linear model is "wrong and harmful" (DoD Ch 3, 5). | design-process |
| 25 | **No conceptual integrity** | the design has many hands' styles with no unifying idea; inconsistent models across parts | Enforce one coherent conceptual model; consistency of idea beats a pile of good-but-unrelated features (DoD Ch 12; APoSD Ch 17). | design-process, design-review |
| 26 | **Undocumented design rationale** | *what* was decided is visible but *why* (and what was rejected) is lost, so future changes relitigate or break intent | Capture the decision trajectory and reasons (DoD Ch 16). | design-process, design-review |
| 27 | **Ignored critical constrained resource** | the design doesn't identify or budget the one scarce resource (latency, memory, cost, attention) that dominates | Name the budgeted resource and design around it; constraints focus the design (DoD Ch 10–11). | design-process, module-design |

## How to use in review

Walk the change against this table. For each hit, name the flag, point at the
line, and propose the fix. Several flags in one place usually point at a single
deeper design problem — fix the design, not each symptom.

Note (from codebase-design): a red flag is a *prompt to look for a better
design*, not an automatic defect. A flag that survives a genuine design-it-twice
pass may be the least-bad option — but you must have looked.
