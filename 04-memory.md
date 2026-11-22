# Chapter 04 — Memory: Every Strategy and Its Price

This is your home chapter. By the end you should be able to name any memory
management scheme, state its guarantee, and state its cost — in one sentence
each. That fluency is what lets you position your work against prior art without
being embarrassed.

---

## 1. The fundamental problem

A program allocates objects. Each object has a **live range**: from allocation to
last use. The memory manager's job is to reclaim memory after the live range
ends, without reclaiming it *during* the live range.

The difficulty: **the last use is undecidable in general.** (It is reducible to
the halting problem.) Every memory management scheme is therefore a *sound
approximation* of liveness — and the schemes differ only in **who computes the
approximation and how**.

That is the entire taxonomy. Let's walk it.

---

## 2. The strategies

### Manual (C)

The programmer decides. `malloc` / `free`.

- **Guarantee:** none. Safety is a social contract.
- **Cost:** ~20ns per alloc, zero per access, zero metadata beyond allocator
  headers, fully deterministic.
- **Control:** maximal. Layout, timing, addresses, all yours.
- **Failure mode:** use-after-free, double-free, leaks, aliasing bugs. The most
  expensive bug class in software history.

### Reference counting (RC)

Each object stores a count of references. Increment on copy, decrement on drop;
at zero, free.

- **Guarantee:** memory safety, *if* the counts are correct.
- **Cost:** a counter per object (8 bytes), an atomic increment/decrement per
  pointer copy (expensive under contention), and **cycles leak**.
- **Control:** deterministic destruction at the moment the last reference dies.
  This is RC's superpower and why it is used in real-time systems.
- **Used by:** Swift (ARC), C++ `shared_ptr`, Rust `Rc`/`Arc`, Objective-C.

**Why it matters to you:** RC is the only scheme with *deterministic* reclamation
and *no* pauses. That combination is why it dominates latency-sensitive domains.
If your "life" model wants deterministic death, RC is the baseline you must beat.

### Tracing garbage collection

Periodically walk the heap from roots; anything unreachable is garbage.

- **Mark–sweep:** mark reachable, sweep the rest. Fragments.
- **Copying:** copy live objects to a new space; dead ones vanish. Compacts, but
  uses 2× space.
- **Generational:** the big idea. See §4.
- **Concurrent / incremental:** do the work alongside the mutator to reduce
  pauses. Costs barriers on every pointer write.
- **Guarantee:** memory safety, and **cycles are collected** (unlike RC).
- **Cost:** throughput overhead, unpredictable pauses (unless real-time GC),
  memory headroom, and *no* deterministic destruction (finalizers are a lie).
- **Control:** poor. No addresses, no layout guarantees, no timing guarantees.

### Regions / arenas

Allocate into a region; free the whole region at once.

- **Guarantee:** safety if you can prove no object escapes the region.
- **Cost:** near-zero. Bump allocation, one free for the whole region.
  This is the fastest allocation strategy that exists.
- **Control:** excellent, and *deterministic*.
- **Used by:** MLKit (region inference), Cyclone (region types), arena allocators
  everywhere (games, parsers, compilers), Rust's `bumpalo`.
- **Limitation:** objects with different lifetimes in one region force the region
  to live as long as the longest — over-retention.

**This is the closest existing thing to your "inheritance" pillar**, and it is
*proven*: Tofte & Talpin showed regions can be *inferred* from the program with
no annotations. Read that before you claim novelty.

### Linear / ownership types

The type system proves each object has exactly one owner; reclamation happens at
a statically-known point.

- **Guarantee:** memory safety, proven statically.
- **Cost:** zero at runtime (that is the point). Compile-time cost: the borrow
  checker, and programmer effort.
- **Control:** excellent — you still get deterministic destruction and no
  metadata.
- **Used by:** Rust, Austral, linear Haskell (partially).

### Capability-based (seL4, CHERI)

Authority to access memory is an unforgeable token. Reclamation = revoking
tokens.

- **Guarantee:** memory safety + *authority* safety (you can't even name memory
  you don't have a capability for).
- **Cost:** capability registers / tagged memory (hardware), or a kernel call.
  CHERI adds ~1 bit per 16 bytes of memory and a tag check per access.
- **Control:** excellent, and revocation is *hierarchical and provable*.
- **This is your "overpowering" pillar, and it is the most formally mature one.**

### Budgeted / bounded

The system pre-allocates a fixed pool; allocation either succeeds within budget or
fails predictably.

- **Guarantee:** bounded memory use, provable.
- **Cost:** whatever the underlying allocator costs, but *bounded*.
- **Used by:** seL4 (no dynamic allocation at all in the kernel), real-time
  systems, WASM hosts, smart contracts (gas), embedded.
- **This is your "currency" pillar, and it is the least served by mainstream
  languages.**

---

## 3. The comparison table (memorize this)

| Scheme | Safety | Determinism | Runtime cost | Cycles | Control |
|---|---|---|---|---|---|
| Manual | ✗ | ✓ | minimal | n/a | max |
| Refcount | ✓ | ✓ | per-copy atomic | ✗ leak | high |
| Tracing GC | ✓ | ✗ | throughput + pauses | ✓ | low |
| Regions | ✓ (if proven) | ✓ | near zero | n/a | high |
| Linear types | ✓ (proven) | ✓ | zero | n/a | high |
| Capabilities | ✓ + authority | ✓ | tag check / call | n/a | high |
| Budgeted | ✓ (bounded) | ✓ | allocator cost | n/a | high |

Notice the pattern: **determinism and safety together require a proof** — either
static (linear/regions) or dynamic (capabilities/refcount). Everything that skips
the proof (GC) buys convenience at the cost of determinism.

> **For your paper:** your model is a *hybrid*: regions (inheritance) + refcount
> or epochs (death) + budget (currency) + capability revocation (overpowering).
> That is a legitimate and interesting combination. But each piece exists. Your
> contribution must be the **integration and its proof**, not the pieces.

---

## 4. The generational hypothesis — and why "aging" is real

This is the empirical fact your "aging" pillar is reaching for, and it is one of
the most solid results in the field:

> **The generational hypothesis:** most objects die young.

Measured repeatedly since the 1980s. Consequences:

- **Generational GC** (Lieberman & Hewitt 1983; Ungar 1984): split the heap into
  generations. New objects go in the nursery. Survivors get *promoted* to older
  generations. Collect the nursery often (cheap, most objects are dead); collect
  old generations rarely.
- This is **literally aging**: objects accumulate "age" by surviving collections,
  and age determines placement and collection frequency.

**Critical distinction for your research:**

| Use of age | Sound? |
|---|---|
| Age determines *where* an object lives (promotion, arena class) | ✓ Sound — it is a placement heuristic |
| Age determines *when* an object is freed | ✗ Unsound — you can free a live object |

The second row is a use-after-free generator. If your language frees on a timer,
you have C's safety with extra steps. **Age may move; only a liveness proof may
free.**

There is a sound dynamic version: **generational references** (Vale). Each object
carries a generation counter; each reference carries the generation it was made
at. Dereference checks they match; on free, the counter increments. This catches
use-after-free *dynamically*, with a check per dereference, no GC. It is a real,
working design and it is the closest thing to a sound "aging" mechanism. Study it.

---

## 5. Escape analysis: the linchpin

Every dynamic scheme above is affordable only because most objects **do not
escape** their allocating function. If the compiler can prove an object never
escapes, it can:

- allocate it on the **stack** (zero cost, freed on return),
- or in a **region** with a known lifetime,
- or **scalar-replace** it entirely (the object ceases to exist).

So escape analysis is the enabling technology for everything in this chapter. It
is also, unglamorously, the place where your research will succeed or fail.

**How it works (the standard formulation):** a constraint-based analysis over the
SSA graph. Each allocation gets a "connectivity" node; assignments and field
stores create edges; a node *escapes* if it reaches a parameter, a return value,
a global, or a field of an escaping object. Solve to a fixpoint; anything not
reaching an escape point is stack-allocatable.

**Why it is hard:** the analysis is inherently *whole-program-ish* (or requires
summaries for modularity), and it is imprecise for anything involving dynamic
dispatch, collections, or threads. That imprecision is why Java still allocates
most things on the heap.

> **For your paper:** if you can state and prove a *precise* escape condition for
> your ownership model — "under these typing rules, this value provably does not
> escape, therefore it costs zero" — that is a genuinely publishable theorem with
> a clean cost model. This may be the most tractable contribution in your whole
> program. Keep it in mind for chapter 12.

---

## 6. Cost models: how to talk about price honestly

A cost model is a *theorem-shaped statement about resources*. Three kinds:

**Allocation cost.** "Each `new` costs O(1) bump + O(1) bookkeeping." State the
constants you measured.

**Reclamation cost.** Amortized: "each object is freed exactly once, so total
free cost is O(n)." Or worst-case: "a collection may pause for T." Both are
legitimate; be explicit which you claim.

**Space overhead.** Bytes per object (headers, counters, tags), plus fragmentation
and headroom. **Report this.** Reviewers in this area care about space more than
about speed, because embedded and real-time users care about space.

The gold standard for cost-model papers is **Perceus** (Koka/Lean): it gives a
*precise* reference-counting cost, proves it, and shows the compiler achieves it
without leaks. Read it as a template for "soundness theorem + cost theorem +
implementation + evaluation."

---

## 7. What "life" maps to, precisely

| Your pillar | The real mechanism | The real literature | Sound version |
|---|---|---|---|
| Inheritance | region subtyping / ownership types | Tofte–Talpin, Clarke et al. | child lifetime ⊆ parent, statically checked |
| Aging | generational promotion | Lieberman–Hewitt, Ungar, Vale | age *moves*, never frees |
| Currency | linear budgets / effects | linear logic, AARA, RTSJ, gas | compile-time allocation bound |
| Overpowering | capability revocation | seL4, CHERI | hierarchical, transitive, with a fence |

Every row has a sound version. None of them is a metaphor — they are all type
systems or kernel designs. **Your paper is the integration of these four sound
versions under one vocabulary, with a proof that they compose.** That is a real
contribution, and it is honest.

---

## 8. Reading

- **Jones, Hosking, Moss, *The Garbage Collection Handbook*** — the definitive
  reference. Read chapters 1–4 and the generational chapter.
- **Tofte, Birkedal, Elsman, Hallenberg, *Region-Based Memory Management* (1997)**
  — the region paper.
- **Blackburn et al., *The DaCapo Benchmarks* (2006)** — how memory research is
  evaluated.
- **Reinking et al., *Perceus* (PLDI 2021)** — the cost-model template.
- **Vale's generational references** — search "Vale generational references
  Verdagon"; the blog series is more readable than the eventual paper.

---

**Next:** `05-ownership.md` — what Rust's borrow checker actually is, mechanically.
