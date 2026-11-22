# Chapter 02 — Types: Judgments, Linearity, Lifetimes

This chapter is the single highest-leverage thing in the guide. Type systems are
the language in which safety papers are written. If you can read typing rules,
you can read 70% of the literature. If you can *write* them, you can write a
paper.

We go slowly, and we start by teaching you the notation, because the notation is
the actual barrier — not the ideas.

---

## 1. What a type is

A type is a **static approximation of a set of values and the operations valid on
them**. That is all. `Int` approximates "values that support `+`, `-`, `<`".
`String` approximates "values that support concatenation."

The approximation is *conservative*: if the type system says "this is an `Int`,"
then at runtime it really is an `Int`, always. That guarantee — **well-typed
programs do not go wrong** — is the entire point.

A type system is a *proof system*. Checking types is *proving a theorem about
your program*. Once you see it that way, the notation stops being scary.

---

## 2. Decoding the symbol soup

Keep this table open while you read any paper.

| Symbol | Name | Meaning |
|---|---|---|
| `Γ` | Gamma | **typing context / environment**: what variables are in scope and their types. "Γ = x:Int, y:Bool" |
| `Δ` | Delta | often a *store* or *heap* typing, or a second context |
| `Σ` | Sigma | often the heap/store, or a substitution |
| `⊢` | turnstile | "derives" / "entails". `Γ ⊢ e : τ` reads "**under context Γ, expression e has type τ**" |
| `τ`, `σ` | tau, sigma | a type |
| `e` | — | an expression |
| `v` | — | a value |
| `:` | — | "has type" |
| `→` | arrow | function type `τ₁ → τ₂`, or a step relation `e → e'` |
| `⟶` | long arrow | a single evaluation step |
| `⟶*` | — | zero or more steps |
| `⇓` | double down arrow | big-step evaluation: `e ⇓ v` |
| `⊥` / `⊤` | bottom / top | least / greatest element of a lattice; also "diverges" |
| `≤` | — | subtype: `τ₁ ≤ τ₂` means a `τ₁` can be used where `τ₂` is expected |
| `∈`, `⊆` | — | membership, subset |
| `⊎` | — | disjoint union (crucial: Γ₁ ⊎ Γ₂ means the contexts share no variables) |
| `↦` | maps-to | a heap cell: `l ↦ v` means location `l` holds value `v` |
| `∗` | separating conjunction | "and, with disjoint heaps" (separation logic) |
| `⊗` | tensor | linear logic's "and" (both resources consumed) |
| `&` | with | linear logic's additive "and" (choose one) |
| `⊕` | plus | linear logic's additive "or" (produce one) |
| `⊸` | lollipop | linear implication: `A ⊸ B` = "consume an A, produce a B" |
| `!` | bang | "of course": a value that may be used any number of times |
| `∀`, `∃` | — | for all, exists |
| `λ` | lambda | anonymous function |
| `φ` | phi | SSA merge node (unrelated to the above) |
| `α`, `β` | — | type variables (in polymorphism) |

### How to read a rule

A typing rule looks like a fraction:

```
    Γ ⊢ e₁ : Int    Γ ⊢ e₂ : Int
    ─────────────────────────────
         Γ ⊢ e₁ + e₂ : Int
```

Read it as: **"if the things above the line hold, then the thing below the line
holds."** The things above are *premises*; below is the *conclusion*. Read rules
bottom-up when checking a program, top-down when reasoning about it.

That's it. That's the whole skill. Every paper's formal section is made of these.

---

## 3. The progression of type systems (and what each buys)

You should be able to place any paper on this ladder.

**Simply typed lambda calculus (STLC).** Functions, base types, variables.
Guarantees: no "stuck" terms (you never apply an `Int` to an argument).
*Limitation:* no recursion, no state, no polymorphism.

**Polymorphism (System F).** `∀α. α → α` is the identity function for any type.
*Buys:* generic code, parametricity ("theorems for free").

**Subtyping.** `τ₁ ≤ τ₂` means "a `τ₁` is acceptable where a `τ₂` is expected."
*Buys:* records/objects, and — crucially — **lifetime subtyping**: a reference
valid for `'long` can be used where `'short` is needed, because `'long: 'short`.
This single fact is most of Rust's lifetime system.

**Effects.** Types annotated with what a computation *does*: allocates, throws,
reads, writes, diverges. `Int --alloc--> Int`. *Buys:* you can prove "this
function does not allocate," which is exactly a budget property. **Effects are
the natural home for your currency pillar.**

**Linearity / affinity.** The big one for memory. See below.

**Refinement types.** Types plus predicates: `{v: Int | v > 0}`. *Buys:* precise
invariants, used in Liquid Haskell, RefinedC.

**Dependent types.** Types can mention values: `Vec n`. *Buys:* full
specifications; costs: proof burden. Probably overkill for your first paper.

---

## 4. Linearity: the heart of safe memory

This is the most important concept in the chapter.

**A linear type** means: a value of that type **must be used exactly once**. Not
zero times, not twice. Exactly once.

**An affine type** means: **at most once**. You may use it once or drop it.

Rust is **affine**, not linear: you can drop a `String` without using it. That is
why `Drop` exists — dropping is an *implicit use*.

### Why linearity gives you memory safety

Think of a heap object as a **resource**. If you have exactly one token
representing "ownership of this object," then:

- You cannot free it twice (you'd need two tokens).
- You cannot use it after freeing (freeing *consumes* the token).
- You cannot alias it mutably (aliasing would duplicate the token).

Memory safety falls out of *resource conservation*. This is not a metaphor; it is
a proof: linearity is the proof theory of resource conservation, and the heap is
a resource.

```
   x : Box<Int>            (one ownership token)
   ─────────────────
   consume(x)  →  ()       (token spent; x no longer usable)
```

The formal machinery is **substructural logic**: ordinary logic lets you use a
hypothesis any number of times (weakening, contraction). Linear logic removes
those rules. Removing *contraction* gives linearity; removing *weakening* gives
relevance; removing both gives... exactly-once. Walker's survey (*Substructural
Type Systems*, 2005) is the readable entry point and is in the bibliography.

> **For your paper:** your "currency" pillar is *already* linear logic, whether
> you call it that or not. If you present it as linear types with a cost
> annotation, you inherit 40 years of proof machinery for free. If you present
> it as a market, you inherit nothing and must prove everything from scratch.
> This is the single most important strategic decision in your research.

### Fractional permissions

A refinement: instead of one token, split ownership into *fractions* that sum to
1. A read-only borrow is a small fraction; a write requires the whole 1. This is
how modern separation logics handle aliasing precisely, and how you would model
"many readers, one writer" without losing the ability to reason.

---

## 5. Lifetimes as static approximations of liveness

Here is the honest, demystified version.

At runtime, an object is *live* from allocation to last use. The compiler cannot
know the last use in general (it is undecidable). So it uses a **static
approximation**: a *lifetime*, which is a region of the program text (or a
constraint variable) that over-approximates the object's live range.

```
{
    let x = String::new();     // 'a begins
    let r = &x;                // r : &'a String
    use(r);
}                              // 'a ends; x dropped
```

`'a` is *not* a runtime thing. It is a **compile-time constraint variable**. The
borrow checker solves a system of constraints:

1. `'a` must cover every use of `r`.
2. `'a` must not outlive `x` (you cannot reference a dead object).
3. `x` may only be dropped after `'a` ends.

This is a **constraint satisfaction problem**, and solving it is a dataflow
analysis (chapter 01) over a lattice of lifetime constraints. That's the whole
trick. It feels magical because the error messages are bad, not because the
theory is deep.

**Variance** is how lifetimes interact with subtyping:

- `&'a T` is **covariant** in `'a`: a longer lifetime can be used where a
  shorter one is expected. (`'long ≤ 'short`, so `&'long T ≤ &'short T`.)
- `&'a mut T` is **invariant** in `T`: you can read *and write*, so you cannot
  assume anything about the type. This is why `&mut` is so restrictive.
- `fn(T)` is **contravariant** in `T`: a function accepting a *more general*
  type can be used where a *more specific* one is expected.

Variance is a common source of confusing errors, and understanding it removes
about half the pain of fighting the borrow checker.

---

## 6. Ownership types and alias types (the research lineage)

You should know these names because your paper will cite them:

- **Ownership types** (Clarke, Potter, Noble, 1998): each object is owned by
  another; the type system tracks the ownership tree statically. This is
  *literally your "inheritance" pillar*, formalized in 1998.
- **Alias types** (Smith, Walker, Morrisett, 2000): a type system that tracks
  *unique* and *aliased* pointers, with the ability to "focus" on a location.
  Precursor to borrowing.
- **Region types** (Tofte–Talpin, 1997): allocation into named regions whose
  lifetimes are inferred; regions are deallocated wholesale. Precursor to
  arenas and to Rust's scopes.

If your "inheritance" idea is ownership types + regions, say so. Reviewers will
respect the lineage far more than a claim of novelty.

---

## 7. How to design a type rule (the four questions)

When you invent a new construct, answer these in order:

1. **Formation** — when is `τ` a well-formed type? (What are its parameters?)
2. **Introduction** — how do you *create* a value of `τ`? (The constructor rule.)
3. **Elimination** — how do you *use* a value of `τ`? (The destructor/accessor
   rules.)
4. **Computation** — what happens when you eliminate a freshly introduced value?
   (The β/ι rule — this is what makes the semantics and the types agree.)

If you cannot answer all four crisply, your construct is underspecified, and the
soundness proof will fail at the step you skipped. **Every failed paper in this
area failed at step 4.**

---

## 8. Gradual typing: the pragmatic escape hatch

The realistic architecture for a language with dynamic mechanisms is **gradual**:

- Static where provable.
- Dynamic checks at a clearly-marked boundary.
- A **proof obligation at the boundary** (this is the part everyone forgets).

Rust's `unsafe` is a gradual boundary with a *social* proof obligation (the
programmer promises). TypeScript's `any` is a gradual boundary with *no* proof
obligation (and it leaks). Your design should make the boundary explicit and
state exactly what must be true at it. That statement is itself a contribution.

---

## 9. Reading exercise

Read the typing rules for `let` in any textbook and answer:

- Which rule introduces a binding?
- Which rule consumes it?
- In a linear system, what changes? (Hint: the context splits: `Γ = Γ₁ ⊎ Γ₂`.)

Then read the subtyping rule for references in the Rust reference or in a paper on
region types, and identify the variance.

If you can do those two things, you have the literacy this chapter promises.

---

## 10. What to read

- **Pierce, *Types and Programming Languages* (TAPL)** — chapters 1–9, then 15
  (subtyping), then 22–23 (System F). This is *the* book. Work the exercises;
  they are the point.
- **Walker, *Substructural Type Systems*** (in *Advanced Topics in Types and
  Programming Languages*) — the linearity chapter you need.
- **Pierce et al., *Software Foundations*, volume 1** — for the Coq mechanics.
- **The Rust Reference, "Lifetime elision" and "Variance"** — short, practical.

---

**Next:** `03-semantics.md` — how to define what a program *means*, and how to
prove your type system is not lying.
