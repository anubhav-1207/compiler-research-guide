# Chapter 05 — Ownership: What Rust Actually Does

Rust is the reference point for "safe and controllable," so you must understand
it mechanically, not reverentially. The good news: **the borrow checker is a
dataflow analysis.** It is not magic, and once you see it as liveness over loans,
you can extend it — which is exactly what your research will do.

---

## 1. The three rules

1. Every value has exactly one **owner**.
2. When the owner goes out of scope, the value is **dropped** (deterministic
   reclamation).
3. You may have **either** many immutable borrows **or** one mutable borrow —
   never both.

Rule 3 is the famous one, and it is best understood as: **aliasing XOR
mutability.** It is the condition under which local reasoning about memory is
possible. If nobody else can see or change a location while you hold a mutable
reference, you can reason about it as if it were a local variable.

That is not an arbitrary restriction. It is the *precondition for optimization*.
This is why Rust code can be as fast as C: the compiler has the aliasing
information it needs. **Safety and speed are the same fact here.**

---

## 2. Move, copy, clone, drop

- **Move:** transferring ownership. The source becomes unusable. No runtime cost
  (it is a static fact; the bytes may not even move).
- **`Copy`:** types that are bitwise-duplicable (integers, `bool`). Assignment
  copies instead of moves.
- **`Clone`:** explicit deep copy. Costs what it says.
- **`Drop`:** the destructor, called at a statically-known point when the owner
  dies.

The crucial design decision: **Rust is affine, not linear.** You may drop a value
without using it. That is why `Drop` is meaningful. A truly linear language would
require you to explicitly consume every value — stricter, sometimes useful,
usually annoying.

---

## 3. The borrow checker as a dataflow analysis

This is the demystification. Here is the algorithm, in plain terms.

**Loans.** A borrow `&x` creates a *loan* — a permission to read (shared) or
write (unique) the place `x`, valid for a *region* of the program.

**The analysis:**

1. **Compute loans.** Every borrow expression introduces a loan with a region
   (a set of program points where it might be live).
2. **Compute liveness of references.** For each reference, the set of points
   where it is used later. This is a standard backward liveness analysis
   (chapter 01).
3. **Check conflicts.** For every point, check that no two loans conflict:
   - a unique loan overlaps any other loan on the same place → error;
   - a shared loan overlaps a write or a unique loan → error.
4. **Check outlives.** Every loan's region must be contained in the region of the
   place it borrows (you cannot borrow something that dies first).
5. **Check moves.** A move invalidates all outstanding loans on that place.

That is the whole thing. It is a constraint system solved by a dataflow fixpoint.

**Non-Lexical Lifetimes (NLL)** was the big improvement: instead of tying loan
regions to lexical scopes, compute them from *actual liveness*. A loan ends at
its last use, not at the closing brace. This made a huge class of previously
rejected programs compile, and it is why modern Rust feels much better than
2015-era Rust.

**Polonius** is the next step: reformulate the analysis as **Datalog rules** over
a set of facts (loans, points, subsets). The advantage is that Datalog makes the
analysis *declarative and extensible* — you can add a rule and the engine handles
the fixpoint. It also fixes some remaining false positives (the "conditional
control flow" problem). If you want to extend the borrow checker, Polonius's
formulation is where you start.

> **For your paper:** if you want to add revocation or budgets to the borrow
> checker, express them as **extra Datalog relations and rules** in the Polonius
> style. That gives you a clean implementation story and a clean metatheory story
> (the rules *are* the specification). This is the single most practical piece of
> advice in this guide for implementation.

---

## 4. The escape hatches (and why they matter to your research)

Rust's safety is not absolute; it has controlled holes. Know them, because your
design will need analogous holes.

- **`unsafe`** — a block where the compiler stops checking. The programmer
  assumes a *proof obligation* (e.g. "this pointer is valid, aligned, and
  non-aliased"). This is a **gradual boundary with a social proof obligation.**
- **`Rc<T>`** — shared ownership via refcounting. Safe, but cycles leak, and it
  is not `Send`.
- **`RefCell<T>`** — interior mutability with *runtime* borrow checking. Moves a
  static check to a dynamic one. Panics on violation. This is exactly the
  "gradual boundary" pattern.
- **`Cell<T>`** — copy-in/copy-out, no borrows at all.
- **Raw pointers** — inside `unsafe`, arbitrary.

The pattern: **static check → dynamic check → unchecked-with-obligation.** That
ladder is the realistic architecture for any language that wants both safety and
flexibility, including yours.

---

## 5. Stacked Borrows and Tree Borrows: the aliasing model

Here is a subtle and important problem. Rust's *type system* checks safe code.
But `unsafe` code can do anything. So what does "correct `unsafe` code" *mean*?
You need a **precise aliasing model** — a formal definition of which pointer
accesses are allowed.

**Stacked Borrows** (Jung et al., POPL 2020) is the leading model:

- Every pointer gets a **tag** (a provenance identifier).
- Each memory location has a **stack of tags** — the "borrow stack."
- Creating a borrow pushes a tag; using a pointer pops everything above it.
- Accessing through a tag that is not on the stack is **undefined behavior**.

The intuition: a mutable borrow "invalidates" all previous pointers to that
location, permanently. A shared borrow allows other shared borrows but not
writes.

**Tree Borrows** is the newer, more permissive model that fixes some false
positives by using a tree instead of a stack. It is likely to become the
standard.

**Why this matters to you enormously:** it means "memory safety" in Rust is not
just "no use-after-free" — it is **provenance-based**. A pointer carries the
authority it was created with, and using it outside that authority is UB even if
the address happens to be valid. **This is a capability model in disguise.** Your
"overpowering"/revocation pillar is closer to Rust's real semantics than you might
think.

**Miri** is the interpreter that implements Stacked/Tree Borrows and detects UB in
tests. If you implement a memory model, **you must test it against Miri-style
execution.** That is the accepted evaluation method for aliasing models.

---

## 6. The known pain points = your research openings

Rust's model is good, not perfect. The failures are well documented, and each one
is a paper.

1. **Self-referential structures.** A struct holding a reference to its own field
   is inexpressible without `Pin` or unsafe. Fundamental to async and to intrusive
   data structures.
2. **Graphs and doubly-linked lists.** The canonical "Rust is painful" examples.
   Solutions today: arenas + indices, `Rc<RefCell<>>`, or unsafe. All are
   compromises.
3. **Conditional control flow false positives.** "Borrow in one branch, move in
   the other" sometimes gets rejected. Polonius addresses some of this.
4. **Lifetime annotation burden** in APIs with many references.
5. **No way to express "this borrow is valid until revoked."** ← **This is
   literally your overpowering pillar.** Rust has no revocation concept. Adding
   one, soundly, with a fence, is open.
6. **No way to express "this function allocates at most N bytes."** ← **This is
   literally your currency pillar.** Rust has no budgets. Adding them is open.
7. **FFI.** Lifetimes stop at the boundary.

Points 5 and 6 are your openings. They are *specific*, *well-motivated*, and
*unaddressed by the most popular safe systems language*. That is exactly the shape
of a good research problem.

---

## 7. Ownership without lifetimes: the Hylo/Val counterpoint

Worth knowing because it is a serious challenge to the whole lifetime approach.
**Hylo** (formerly Val) argues that **mutable value semantics** gives you memory
safety with *no lifetime annotations at all*:

- Values are copied on assignment (semantically), optimized to moves.
- Mutation requires **exclusive access** to a value, enforced by the "law of
  exclusivity."
- No references persist across function boundaries; borrowing is purely local.

The claim: you get safety and performance without the borrow checker's annotation
burden. The cost: you lose cheap shared mutable structures, and you must copy or
move more.

Why you should care: it is a genuine alternative point in the design space, and
reviewers will ask "why not just do value semantics?" Have an answer. (Likely
answer: because your users need shared mutable structures and explicit budgets,
which value semantics does not provide.)

---

## 8. Reading

- **The Rustonomicon** — the unsafe/aliasing book. Read the "Borrowing" and
  "Data layout" chapters.
- **The NLL RFC** (`rust-lang/rfcs` 2094) — the design document for non-lexical
  lifetimes. Surprisingly readable.
- **Jung et al., *Stacked Borrows* (POPL 2020)** — the aliasing model.
- **The Polonius repository** — the Datalog rules are short and readable.
- **Matsakis, *The Rust Borrow Checker: A Deep Dive*** — talks and blog posts.
- **Hylo/Val papers and blog** — the value-semantics alternative.

---

**Next:** `06-capabilities.md` — seL4 and CHERI, where "overpowering" is already
a proven, formal mechanism.
