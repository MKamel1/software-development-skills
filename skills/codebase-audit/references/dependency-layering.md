# Dependency Direction & Layering — detecting violations

How `codebase-audit` checks that dependencies point the right way. Distilled from
Leonardo Giordani, *Clean Architectures in Python* (2nd ed., 2023) — language-agnostic
here even though the book uses Python. Terminology note: Giordani writes in the "clean
architecture" tradition (his **interface** = a *port*; his **gateway/external-system
implementation** = an *adapter*); both vocabularies map cleanly.

## The dependency rule (the one to check first)

**Source-code dependencies point only inward — toward higher-level policy.** Inner
layers are oblivious to outer ones (Ch. 2). Giordani's phrasing: *talk inward with
simple structures, talk outward through interfaces.* Passing data toward the core can
use plain types and domain entities; reaching outward must go through an interface,
never a concrete outer-layer name. This is what makes an outer layer (a DB, a web
framework) replaceable without touching business logic.

**First check of any layering audit:** does anything in an inner/lower layer reference
a name (class, module, package) from an outer/higher layer, not via that layer's
declared interface? If yes, it's a violation.

## Layer model (Ch. 2)

Concentric, innermost first:

| Layer | Is | May depend on | May NOT depend on |
|---|---|---|---|
| **Entities** | Domain models — business concepts, no storage/framework/serialization methods | Other entities | Anything outward (no DB, no framework, no use cases) |
| **Use cases** | The business logic — one small, testable action | Entities, other use cases | Gateways/external systems *only through an interface*, never directly |
| **Gateways** | Interfaces defining access to external systems (e.g. a repository interface) | Entities, use cases | (rarely calls a use case itself) |
| **External systems** | Concrete implementations behind gateway interfaces (a DB driver, a web framework, a CLI) | Everything inward | — (outermost) |

**Asymmetry the auditor must respect (Ch. 2):** the two directions are not the same.
*Use case → external system* (needs storage) is outward → **requires an interface**.
*External system → use case* (a controller/endpoint/CLI invoking a use case, a user
click) is inward → **allowed directly, no interface**. So a controller that imports and
calls a use case is *correct*, not a violation. The violation shape is specifically
**inner code reaching outward** without an interface between.

## Ports & adapters (Ch. 1, 6)

- **Port** = a gateway's declared interface (e.g. a repository exposing `list(filters)`).
  The use case is written against the signature only.
- **Adapter** = a concrete external-system class implementing that interface for one
  technology (an in-memory repo for tests; a Postgres repo for production) — identical
  API, different `__init__`.
- **Crossing the boundary:** the concrete adapter is chosen in the *composition root*
  (the main script) and injected into the use case, not hard-coded inside it. That's
  why swapping Postgres → Mongo is near-drop-in.

## Detecting violations by import inspection

Build the import graph, label each module with its layer, and flag edges pointing from
an inner label to an outer label. Concrete tells:
- An **entities/domain** module importing anything from use-case, gateway, ORM, web, or
  infrastructure packages.
- A **use-case** module importing a concrete driver/framework (`psycopg2`, `pymongo`,
  `requests`, an ORM model, a Flask/Django import) instead of referencing an injected
  object — or importing the adapter at module top level rather than receiving it as a
  parameter (dependency injection bypassed even if a "repository interface" exists).
- A **gateway/interface** module that itself imports a specific driver — the port and
  adapter have collapsed into one file.
- **Legitimate, don't flag:** an outer module (controller, CLI, endpoint) importing and
  calling an inner use case/entity directly.

Tooling that automates this: import-linter (Python), dependency-cruiser / madge (JS/TS),
ArchUnit (Java); or grep the import graph and label by directory. Turn a confirmed
violation into a fitness function so it can't return ([fitness-functions.md](fitness-functions.md)).

## Real boundary vs incidental structure (Ch. 2)

A **real** boundary is a fixed interface where the far side is genuinely swappable
(different DB, different presentation) with the inner side's code and tests unaffected —
test: could you swap the technology behind it and leave the inner code untouched?
**Incidental** structure is directory tidiness with no such swappability. Giordani's own
tell for an over-drawn boundary: if code repeatedly punches *through* the interface for
performance, the boundary may not earn its cost — *"consider removing one layer of
abstraction, merging the two layers."* An **undocumented** cross-interface direct call
is worse than a documented one.

## When this is overkill (restraint)

The clean architecture "pushes abstraction to its limits" and costs upfront effort and
some runtime performance; Giordani is explicit that you may break the rules where those
costs aren't worth it — *provided you record why*, so a maintainer doesn't "fix" it.
Signals the audit should treat as premature abstraction, not praise:
- A gateway/interface with **exactly one adapter** ever implemented and no near-term
  second — collapse it back into one module (matches this repo's **one adapter is a
  hypothetical seam; two is a real one**).
- A full four-layer split imposed on a domain type too simple to have behavior — the
  entities layer only earns its keep once a concept is complex enough to need its own
  representation (Ch. 2).
