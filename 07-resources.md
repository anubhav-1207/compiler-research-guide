# Chapter 07 — Resources: Turning "Currency" into a Type

Your "currency" idea is the most promising pillar, and the most likely to be
built wrong. This chapter gives you the theory that makes it *sound*: resource
logics, linear types, amortized analysis, and bounded-memory systems. By the end
you should be able to state a budget as a type and prove it holds.

---

## 1. The core insight: resources are not values

A value can be copied. A resource cannot. A file handle, a DMA buffer, a lock, a
stack frame, a byte of heap — these are **conserved quantities**. If you copy a
resource, you have either broken conservation or created something from nothing.

This is not a metaphor. It is a *logic*, and it is the reason linear types exist.

**Linear logic** (Girard, 1987) is the logic of conservation. In ordinary logic:

```
Γ, A ⊢ B          (weakening: you may ignore A)
Γ, A, A ⊢ B       (contraction: you may use A twice)
```

Linear logic **removes** weakening and contraction. Every hypothesis must be used
exactly once. This sounds crippling until you realize: *that is exactly what a
resource is.*

- **Affine logic** removes only contraction (use at most once). Rust is affine.
- **Relevant logic** removes only weakening (use at least once).
- **Linear logic** removes both (exactly once).

**The curry–howard reading:** linear logic is the type theory of resource
conservation. Programs are proofs; resources are hypotheses. If your budget
system is a linear type system, you get conservation *for free, by the
correspondence*, rather than having to prove it case by case.

> **For your paper:** present currency as **linear types with a resource
> annotation**. You inherit Girard, Wadler, Walker, and 40 years of proof
> machinery. Present it as a "market" and you inherit nothing and must prove
> everything. This is the most important strategic choice in your research, and I
> cannot overstate it.

---

## 2. Three ways to type a budget

There are three established designs. Pick one; do not invent a fourth.

### (a) Linear tokens

A scarce resource is represented by a token that must be spent.

```
alloc : (n : Nat) → Token n → (Box n × Token (budget - n))
```

You hold a `Token B` representing "B bytes remaining." Allocating consumes some
of it. The token is linear, so you cannot duplicate it — you cannot spend the
budget twice. **Boundedness is a consequence of linearity.**

This is clean, compositional, and easy to prove. The cost is that every function
must thread the token, which is noisy. (Rust's `&mut` is exactly this noise,
hidden by syntactic sugar.)

### (b) Effect-annotated types

Annotate functions with their allocation *effect*:

```
f : Int → Int  --alloc(k)-->
```

where `k` is a bound on bytes (or on the number of allocations). Composition adds
effects; a top-level function's effect is its total budget. **This is the
"currency as a type signature" version**, and it is what most real systems do
(seL4's memory pre-allocation is the manual analogue).

The cost: effect inference (or annotation burden), and the bound must be
compositional — so you need a *summary* for every function, including indirect
calls.

### (c) Amortized / potential-based analysis

Attach a **potential function** (a number) to each data structure, representing
the "credit" stored in it. Operations consume credit; the invariant is that
potential never goes negative.

This is **Amortized Resource Analysis (AARA)** (Hoffmann and colleagues). It is
the most powerful of the three, because it handles *data-dependent* bounds: "this
sort allocates O(n log n) because the potential of the input list is n log n."

The cost: the analysis is a linear program, and it is hard to automate for
complex code. But it is the only approach that gives you **asymptotic** bounds
rather than constants.

> **Recommendation for a first paper:** (b) effect annotations, with a simple
> composition rule. It is the easiest to prove sound, the easiest to implement,
> and the easiest for reviewers to understand. Save (c) for the sequel.

---

## 3. Bounded memory in the real world (steal from these)

You are not inventing this from nothing. Four systems have done pieces of it:

**seL4.** The kernel performs **no dynamic allocation at all.** All memory is
pre-allocated and handed to components as capabilities. This is the most extreme
form of "budget": zero. It is why the kernel is verifiable — there is no allocator
to reason about.

**Real-Time Specification for Java (RTSJ).** `MemoryArea` objects (scoped,
immortal, physical) with **size checks at allocation**. The runtime throws
`OutOfMemoryError` deterministically. Escape rules prevent references from
outliving their area. Clunky, unloved, but *it exists and it is specified*.

**Ethereum gas.** The most successful "currency in a VM" ever deployed. Note what
it is: a **static, deterministic, prepaid budget** per operation, checked before
execution. Not a market. Not dynamic pricing. Static and conservative. This is
the design that works.

**WASM memory limits / cgroups.** Hard ceilings enforced by the host. Simple,
effective, and the reason multi-tenant systems survive noisy neighbors.

The lesson from all four: **budgets work when they are static, deterministic, and
checked before the operation.** Budgets fail when they are dynamic, priced, and
negotiated.

---

## 4. Perceus: the cost-model gold standard

If you read one paper to learn how to present a cost model, read **Perceus**
(Reinking, Xie, de Moura, Leijen; PLDI 2021).

What it does:

- **Precise reference counting**: it inserts `dup` and `drop` exactly where
  needed, and proves the counts are exact (no leaks, no over-counting).
- **Reuse analysis**: when a value is consumed and a same-sized value is
  produced, it *reuses the allocation in place* (FBIP — Functional But In-Place).
- **A precise cost theorem**: the number of `dup`/`drop` operations is a function
  of the program's types, not of runtime behavior.

Why it matters to you: it is the model for **"soundness theorem + cost theorem +
implementation + evaluation"** in one paper. Copy its structure:

1. Calculus with reference counting.
2. Operational semantics with counts.
3. Typing with borrow annotations.
4. **Theorem 1 (Soundness):** well-typed programs don't leak or double-free.
5. **Theorem 2 (Cost):** the number of RC operations is bounded by the typing
   derivation.
6. Implementation in Koka/Lean.
7. Benchmarks showing the bound is tight.

Your paper should have the same two-theorem shape.

---

## 5. The failure modes of "currency"

Be explicit about these in your paper; reviewers will raise them.

**1. Who sets the price?**
- Compiler → you have a static type system (fine).
- Runtime → you have an allocator policy (fine, but it is not a *language*
  feature).
- Programmer → you have manual memory management with extra steps and worse
  ergonomics (bad).
- **Market** → nondeterminism, untestability, DoS surface (reject).

**2. Correctness must not depend on wealth.** If a program is safe when it has
budget and unsafe when it does not, you have built a language where *correctness
is a function of resources*. That is a catastrophic property. The budget must
cause a **predictable, safe failure** (an error return, a trap, a static
rejection) — never corruption.

**3. Composition.** Budgets must compose: if `f` costs `a` and `g` costs `b`,
`f; g` costs `a + b`. If your rule is not compositional, you cannot analyze
libraries, and the system is useless in practice.

**4. The inference burden.** Fully annotated budgets are unbearable. You need
inference (or defaults, or a solver). Budget this as real work.

**5. Bounds that are too loose.** A bound of "1 TB" is technically sound and
practically worthless. Reviewers will ask how tight your bounds are on real code.
**Report the ratio of proven bound to actual usage.** That is your key metric.

---

## 6. The theorem you should aim for

Here is a clean, defensible formal statement for a budget system:

> **Theorem (Bounded Allocation).** If `⊢ e : τ --alloc(B)-->`, and `e` runs to
> completion in a state with at least `B` bytes of free budget, then it never
> fails with an allocation error, and the total bytes allocated is at most `B`.

> **Theorem (Safety).** If `⊢ e : τ --alloc(B)-->` and the budget is exhausted,
> `e` reaches a *defined* error state — it never reaches a stuck state.

Two theorems, both by induction on the typing derivation. The second is the one
that matters: **running out of budget is a defined behavior, not a crash.**

That pair — "bounded" and "fails safely" — is the formal content of your currency
pillar. It is a real contribution and it is provable.

---

## 7. Reading

- **Walker, *Substructural Type Systems*** — the readable linear-logic-for-types
  survey. Start here.
- **Wadler, *Linear Types Can Change the World!* (1990)** — the famous (short)
  paper.
- **Hoffmann et al., *Resource-Aware Type Systems*** (various; start with the
  RAML paper) — amortized analysis.
- **Reinking et al., *Perceus* (PLDI 2021)** — the template.
- **The RTSJ specification, scoped memory chapters** — the industrial attempt.
- **The Ethereum Yellow Paper, gas sections** — the deployed attempt.
- **Austral language documentation** — a modern linear-types language, small and
  readable.

---

**Next:** `08-concurrency.md` — where every one of these designs dies if you get
the fence wrong.
