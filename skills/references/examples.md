# Worked Examples

One tight before→after per skill, each naming the red flag it illustrates
([red-flags.md](red-flags.md)) and the fix ([principles.md](principles.md)).
These are illustrative snippets, not book excerpts — pattern-match against
them, don't treat them as the only shape a fix can take.

## design-process

**Flag:** *Requirements taken as solutions* (#23).

> **Before:** "Add an 'Export to CSV' button to the orders table." The team
> builds exactly that: a button, a CSV file, one format, one table.
>
> **After:** Dig one level down — *why* CSV, *why* this table? The real need
> turns out to be "get order data into the finance team's accounting tool
> without a human retyping it." That reframes the work as a general export
> mechanism (format chosen by destination, reusable across tables) instead of
> a single hard-wired button — and it surfaces a second, cheaper option
> (a scheduled feed) the literal requirement never would have.

**Fix:** surface the underlying need before designing to the stated ask; the
stated form is often just the first implementation that came to the
requester's mind.

## module-design

**Flag:** *Shallow Module* / *Pass-Through Method* (#1, #5).

```typescript
// Before — interface is as complex as the implementation; callers must
// orchestrate three calls in the right order to do the one common thing.
class OrderRepo {
  findById(id: string): OrderRow { /* ... */ }
  findLineItems(orderId: string): LineItemRow[] { /* ... */ }
  hydrate(row: OrderRow, items: LineItemRow[]): Order { /* ... */ }
}
// caller: repo.hydrate(repo.findById(id), repo.findLineItems(id))

// After — one call, common case trivial, assembly hidden inside.
class OrderRepo {
  get(id: string): Order { /* finds, fetches items, hydrates internally */ }
}
// caller: repo.get(id)
```

**Fix:** pull the orchestration down into the module so every caller doesn't
re-derive it; the deletion test on the old three-method version shows the
complexity would just reappear at every call site.

## designing-for-change

**Flag:** *Train-wreck / Law-of-Demeter breach* (#17).

```typescript
// Before — caller reaches through Order into Customer into Address; a change
// to any of those internals breaks every caller that chains through them.
const zip = order.getCustomer().getAddress().getZipCode();

// After — Tell, Don't Ask: Order owns the question it can answer directly.
const zip = order.shippingZip();
```

**Fix:** give the object a method that answers the question, instead of
exposing the chain of objects needed to compute the answer yourself; this
also means only `Order` needs to know how a zip code is derived.

## design-review

**Flag:** clustering — several symptoms, one root cause.

> A diff adds a *Vague Name* (`data`), a `Comment Repeats Code` ("// process
> the data"), and a `Pass-Through Method` that just forwards `data` to another
> function with the same shape. Reported as three unrelated nits, a fix-list
> of three feels thorough but misses the point.
>
> **After clustering:** all three trace to one cause — the entity being
> passed around was never given a real shape or name, so nothing about it
> could be named, commented, or exposed well. The report leads with *that*
> ("model the domain object explicitly") instead of three separate polish
> items.

**Fix:** report the root design problem, not each symptom it produced —
fixing the root usually resolves the others for free.

## inline-authoring

**Flag:** *Vague Name* + *Comment Repeats Code* (#11, #9).

```typescript
// Before
let data = process(obj); // process the object
data.forEach(d => save(d));

// After
const normalizedOrders = normalizeLegacyOrders(rawOrderBatch);
normalizedOrders.forEach(order => persist(order));
```

**Fix:** name the actual thing (`normalizedOrders`, not `data`) so the code
reads without the comment, then delete the comment — it said nothing the new
name doesn't already say.
