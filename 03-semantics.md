# Chapter 03 — Semantics and Soundness: How to Prove Safety

A safety paper without a soundness argument is an opinion. This chapter teaches
you what "sound" means precisely, how to state it, how to prove it, and how to
mechanize the proof so reviewers believe you.

This is the chapter that most self-taught researchers skip, and it is the reason
their papers get rejected. Do not skip it.

---

## 1. Why you need semantics at all

You cannot prove "this program cannot use-after-free" unless you have a
*definition* of what running the program means. That definition is the
**operational semantics**: a set of rules saying how a program state evolves,
one step at a time.

Without it, "safe" is a vibe. With it, "safe" is a theorem.

---

## 2. Configurations and small-step semantics

A **configuration** is the complete state of a running program: the expression
being evaluated plus the heap/store.

```
⟨e, μ⟩        e = current expression, μ = current store (heap)
```

**Small-step semantics** defines a relation `⟨e, μ⟩ ⟶ ⟨e', μ'⟩`: "this
configuration takes one step to that one." Rules look like:

```
    μ(l) = v
    ─────────────────────
    ⟨!l, μ⟩ ⟶ ⟨v, μ⟩            (dereference)
```

Read: "if location `l` holds `v` in the store, then dereferencing `l` steps to
`v`." That's a whole semantic rule. They compose.

**Big-step semantics** (`e ⇓ v`) says "e evaluates to v in one judgment." Easier
to read, harder to reason about for concurrency and for intermediate states. Most
safety papers use small-step.

**Why small-step matters for you:** safety violations are *intermediate states*.
A use-after-free is a configuration where you dereference a location that is no
longer in the store. Small-step semantics lets you talk about exactly that
moment. Big-step hides it.

---

## 3. The stuck state: the definition of a bug

Define: a configuration is **stuck** if it is not a value and no rule applies.

```
stuck(⟨e, μ⟩)  ≜  e is not a value  ∧  ∄ e', μ'. ⟨e, μ⟩ ⟶ ⟨e', μ'⟩
```

Every safety violation is a stuck state:

- **Use-after-free:** you dereference `l`, but `l ∉ dom(μ)`. No rule applies. Stuck.
- **Double free:** you free `l`, but `l ∉ dom(μ)`. Stuck.
- **Out of bounds:** the address arithmetic rule has no case for it. Stuck.
- **Type confusion:** you apply an `Int` as a function. No rule. Stuck.

So the entire safety question reduces to:

> **Can a well-typed program reach a stuck state?**

That is the theorem. Everything else is machinery to prove it.

---

## 4. Progress and preservation (the Wright–Felleisen method)

The standard decomposition (Wright & Felleisen, 1994):

**Theorem (Progress).** If `⊢ e : τ` (closed, well-typed), then either `e` is a
value, or there exist `e', μ'` with `⟨e, μ⟩ ⟶ ⟨e', μ'⟩`.

*Meaning:* a well-typed program never gets stuck *right now*.

**Theorem (Preservation).** If `⊢ e : τ` and `⟨e, μ⟩ ⟶ ⟨e', μ'⟩`, then
`⊢ e' : τ` (with an appropriately updated store typing).

*Meaning:* stepping preserves well-typedness, so you can apply Progress again,
forever.

**Corollary (Safety).** Well-typed programs never get stuck. By induction on the
number of steps: Progress gives you a step or a value; Preservation keeps you
well-typed; repeat.

That is the whole architecture of a soundness proof. **You will write this
theorem, in some form, in your paper.** Learn the shape now.

### Why the store typing matters

For a language with a heap, `Γ` alone is not enough — you also need to know what
types the heap locations have. That is `Σ` (the store typing), and Preservation
must update it. Getting this right is where most hand proofs have holes, and it
is the #1 reason people move to a proof assistant.

---

## 5. Where syntactic proofs break down

Progress/Preservation works beautifully for STLC. It breaks for:

- **Recursive types** (equi-recursive): you need coinduction.
- **State and aliasing:** the store typing must be *invariant* across steps, and
  aliasing makes that hard.
- **Concurrency:** interleaving means you must reason about *other* threads
  stepping between your steps.
- **Unsafe/FFI boundaries:** the foreign code is not well-typed in your system.

For these, the field uses **semantic soundness**:

**Logical relations.** Instead of proving "well-typed programs don't get stuck"
syntactically, you define a *relation* `τ` between types and sets of values that
"behave well," then prove: if `⊢ e : τ` then `e ∈ ⟦τ⟧`. This scales to
polymorphism, recursion, and (with step-indexing) to state and concurrency.

**Step-indexing.** A technique for making recursive logical relations well-founded
by indexing them with "number of steps remaining." It is the standard tool for
reasoning about higher-order state. If you read one advanced technique, read this
one; it is the gateway to Iris.

**Iris.** A Coq framework for concurrent separation logic with step-indexing,
used in a large fraction of modern PL papers about memory. If your paper involves
concurrency and safety, you will very likely use or imitate Iris.

---

## 6. Separation logic: the right logic for heaps

Classical logic cannot express "these two pointers point to *different* cells."
Separation logic can, and it is the standard tool.

**The key connective:** `P ∗ Q` — "P holds for one part of the heap, Q holds for a
**disjoint** part." Disjointness is built into the logic. That is exactly what
you need for aliasing.

**The key assertion:** `l ↦ v` — "location `l` holds value `v`." Owning this
assertion *is* owning the cell.

**The frame rule** — the most important rule in the logic:

```
    {P} c {Q}
    ─────────────────────
    {P ∗ R} c {Q ∗ R}
```

"If `c` is safe with resource `P`, then it is safe with `P` plus *anything
disjoint*." This is **local reasoning**: you prove a function correct using only
the resources it touches, and it composes. It is why separation logic scales.

**Concurrent separation logic (CSL)** adds the rule that a thread may only touch
resources it owns — which is precisely a *data-race-freedom* proof. This is the
formal content of "no data races."

> **For your paper:** hierarchical revocation is naturally a separation-logic
> story. "A parent owns a subtree of resources; revoking the parent consumes the
> whole subtree." The frame rule then tells you what revocation *cannot* affect.
> This is the cleanest formal route for your "overpowering" pillar, and it has
> direct precedent in seL4's proofs.

---

## 7. Mechanization: why reviewers demand it

A hand proof in an appendix is worth less every year. The reasons are good:

- Hand proofs of type soundness are routinely found to be wrong, sometimes years
  later.
- Reviewers cannot check 14 pages of cases.
- A mechanized proof is *reproducible evidence*, like a benchmark.

**The tools:**

- **Coq / Rocq** — the traditional choice; huge ecosystem; Iris lives here.
  Software Foundations is the best tutorial.
- **Lean 4** — modern, fast, excellent tooling; growing PL community. If you are
  starting fresh in 2026, **Lean 4 is a reasonable default**.
- **Isabelle/HOL** — strong automation, used by seL4. Great for big engineering
  proofs, less fashionable in PL.
- **Agda** — beautiful dependent types, poor automation.

**What "mechanized" means in practice:** you define the syntax as an inductive
type, the semantics as an inductive relation, the typing judgment as an inductive
relation, and the theorems as `Theorem`/`lemma` with a proof script or term. The
proof assistant checks every step.

**Realistic scoping:** you do not need to mechanize a full compiler. You need to
mechanize **the core calculus** — the 8–12 rules that capture your idea — and
state clearly that the implementation is a *separate*, unverified artifact. That
is what most papers do, and it is accepted.

---

## 8. The anatomy of a soundness proof for a paper

Here is the recipe. Follow it and your proof will have no holes.

1. **Define the calculus.** Syntax, values, configurations. Keep it minimal — 6
   to 10 constructs. Every construct you add is a case in every lemma.
2. **Define the semantics.** Small-step rules, including the heap.
3. **Define the typing judgment.** Including the store typing.
4. **State the invariant.** What does "well-typed" *mean* for the heap? For your
   system this will include your new assertion (e.g. "every live object has a
   valid owner," "every allocation is within budget").
5. **Prove Preservation.** The hard lemma. You will need auxiliary lemmas:
   substitution, weakening, store extension, and — for your system — *the
   invariant's own preservation*.
6. **Prove Progress.** Usually easier, but for your system it must show that
   "in budget" and "not revoked" imply "not stuck."
7. **Corollary: safety.** No stuck states.
8. **State the cost model as a separate theorem.** "Every well-typed program's
   allocation is bounded by f(types)." This is often an *induction on the typing
   derivation* and is completely independent of the safety proof. Two theorems,
   two proofs. Say so.

That separation — **safety theorem + cost theorem** — is the structure of a
strong paper in your area.

---

## 9. A worked mini-example (do this on paper)

Take STLC and add a single primitive: `alloc(v)` which returns a location, and
`free(l)`. Now:

- Write the semantics for `alloc` and `free`.
- Show that `free(l); !l` is stuck.
- Now add a linear type `Box τ` with rules: `alloc : τ → Box τ` (consumes the
  value, produces a linear token) and `free : Box τ → Unit` (consumes the
  token). Dereference requires the token.
- Prove Preservation and Progress.
- Observe: the proof works *because* the token cannot be duplicated. That is the
  entire content of "linearity gives memory safety."

If you can do this exercise, you can read and write the formal core of most
papers in this area. It is four pages of work and it is worth more than ten
papers read passively.

---

## 10. What to read

- **Pierce, *Software Foundations* vol. 1 & 2** — Coq, from zero. Do the
  exercises.
- **Wright & Felleisen, *A Syntactic Approach to Type Soundness* (1994)** — the
  progress/preservation paper. Short, canonical.
- **Jung et al., *Iris from the Ground Up* (2020)** — the modern tutorial for
  concurrent separation logic. Dense but the definitive entry point.
- **Sammler et al., *RefinedC* (PLDI 2021)** — a real, recent system that combines
  refinement types, ownership, and separation logic for C. Read it as a model of
  what a strong paper in your target area looks like.
- **Appel, *Program Logics for Certified Compilers*** — if you want the
  compiler-verification angle.

---

**Next:** `04-memory.md` — every memory strategy, its cost model, and why escape
analysis is the linchpin of your whole idea.
