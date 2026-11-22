# The Safety × Control Compiler Research Guide

> A self-study curriculum for going from "I have an idea about life-like memory
> management" to "I can write and defend a paper about safety and control in a
> compiler."

**Audience:** you have programmed (C, Rust, Python, whatever), you have *not*
studied compiler design or programming language theory formally, and you want to
publish something real.

**Promise:** work through this in order and you will be able to (a) read a PL
paper without drowning, (b) state a soundness theorem and sketch its proof,
(c) build a small prototype that demonstrates a cost model, and (d) know exactly
which claim in your idea is defensible and which is not.

**Non-promise:** this will not make you an expert in nine months. It will make
you *dangerous* in nine months — able to produce a workshop paper and a credible
plan for a full one.

---

## The one-sentence thesis of this guide

> A paper at the safety × control intersection is **a soundness theorem, plus a
> cost model, plus an artifact** — and the novelty is almost never the mechanism,
> it is the *combination* and the *proof*.

If you remember nothing else, remember that. Your "life" idea is not going to
win because the metaphor is beautiful. It wins if you can write:

> "We present a memory model with *hierarchical revocation and static allocation
> budgets*, prove it sound (no use-after-revocation, no unbounded allocation),
> and show it costs ≤ 4% over Rust on these workloads."

That sentence is a paper. Everything below exists to get you to the point where
you can write it honestly.

---

## How to use this guide

**Read 00–01 first, in one sitting.** They orient you and give you the machine.

**Then 02–03.** These are the two hardest chapters and the two that separate
"person with opinions" from "person who can publish." Type systems and semantics
are the grammar of every paper you will read. Do not skip them.

**Then 04–08** in order. These are your actual research territory. Each one ends
with a *"For your paper"* box mapping it onto the life-model pillars.

**09–11 are craft.** Read them before you start writing anything, not after.

**12 is the plan.** It converts knowledge into five concrete projects, sized by
ambition.

**13–14 are reference.** Keep them open.

### Suggested cadence

| Weeks | Chapters | Milestone |
|---|---|---|
| 1–2 | 00, 01 | Can explain SSA and dataflow on a whiteboard |
| 3–5 | 02, 03 | Can read a typing-rules page and state progress/preservation |
| 6–9 | 04, 05 | Can explain every memory strategy and its cost |
| 10–12 | 06, 07 | Can formalize revocation and budgets on paper |
| 13–14 | 08 | Can explain why your design survives threads (or doesn't) |
| 15–16 | 09–11 | Can critique a paper and outline your own |
| 17+ | 12 | Pick a rung, start building |

That is roughly a 4-month *study* phase. The research phase after it is longer.
Be honest with yourself about that; the field rewards patience and punishes
haste with rejections.

---

## The prerequisite reality check

You do **not** need:

- A PhD in math.
- Category theory (it is beautiful and almost never required for this work).
- To have written a production compiler.

You **do** need, and this guide will supply:

- **Induction** (structural and on derivation height). This is the single most
  used proof technique in the field. If you can prove things about lists
  recursively, you can start.
- **Basic logic:** ∧ ∨ ¬ → ∀ ∃, and the idea of a *judgment*.
- **Lattices and fixed points**, at the level of "a set of facts that only grows
  until it stops." One page of Knaster–Tarski is enough.
- **Comfort with notation.** PL papers are dense with Greek letters. Chapter 02
  includes a decoder table. It is a language, and it is learnable in a week.

---

## What "safety" and "control" mean here, precisely

People say these words loosely. In this guide they mean:

**Safety** — the absence of a defined class of undefined behavior. Concretely, at
minimum:

- no use-after-free / use-after-revoke
- no double-free
- no out-of-bounds access
- no data races (or a defined model for them)
- no type confusion

**Control** — the programmer can *predict and constrain* the machine:

- object layout and size
- when allocation happens, and where
- when reclamation happens (determinism)
- absence of hidden metadata and hidden checks on the hot path
- bounded, statically-known resource use

The whole field exists because **you cannot maximize both at once without a
proof**. That is the trilemma from our earlier discussion, and it recurs in every
chapter. Learn to spot which corner a paper is sacrificing.

---

## The directory

```
compiler-research-guide/
├── README.md              ← you are here
├── 00-orientation.md      the field, the paper economy, where the gaps are
├── 01-the-machine.md      pipeline, IR, SSA, dataflow, lowering
├── 02-types.md            judgments, linearity, lifetimes, variance
├── 03-semantics.md        operational semantics, soundness, separation logic, Coq/Lean
├── 04-memory.md           GC, RC, regions, arenas, escape analysis, cost models
├── 05-ownership.md        Rust, borrow checking, NLL, Polonius, Stacked/Tree Borrows
├── 06-capabilities.md     seL4, CHERI, revocation, quiescence
├── 07-resources.md        linear logic, budgets, Perceus, bounded memory
├── 08-concurrency.md      memory models, epochs, hazard pointers, RCU
├── 09-reading.md          the three-pass method, adapted for PL papers
├── 10-writing.md          structure, theorems, rebuttals, rejection
├── 11-evaluating.md       benchmarks, statistics, artifacts, validity
├── 12-project-ladder.md   five projects from toy to publishable
├── 13-bibliography.md     annotated, prioritized
└── 14-glossary.md         every term, one line
```

---

## A warning about scope

The most common failure mode for a first paper in this area is **claiming too
much**. "A new memory paradigm" is a 20-year research program, not a paper. "A
sound type system for bounded allocation in the presence of hierarchical
revocation" is a paper. Chapter 12 exists to help you cut your ambition down to
a defensible size without losing what made the idea interesting.

Cut ruthlessly. A small true theorem beats a grand vague vision, every single
time, in front of a reviewer.

---

## License

This guide is licensed under
[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE).

You may share and adapt it for any purpose, including commercially, as long as
you give appropriate credit and indicate if changes were made.

---

*Maintained by Anubhav G. Corrections and additions are welcome — open an issue
or a pull request.*
