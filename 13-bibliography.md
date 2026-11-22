# Chapter 13 — Annotated Bibliography

Prioritized. ★ = read this first. Work top to bottom within each section; do not
try to read everything.

---

## Tier 0 — Read before anything else (2 weeks)

★ **Peyton Jones, *How to Write a Great Research Paper*** — free online, 45
minutes, changes how you think. Read twice.

★ **Keshav, *How to Read a Paper*** — the three-pass method. 5 minutes.

★ **Pierce, *Types and Programming Languages*** (TAPL) — chapters 1–9, 15, 22–23.
The type systems book. Work the exercises; they are the point.

★ **Pierce et al., *Software Foundations* vol. 1** — Coq from zero. Do the first
four chapters. This is your entry to mechanization.

---

## Tier 1 — Core semantics and types (4–6 weeks)

★ **Wright & Felleisen, *A Syntactic Approach to Type Soundness* (1994)** —
progress and preservation. Short, canonical, the template for every soundness
proof you will write.

★ **Walker, *Substructural Type Systems*** (in *Advanced Topics in Types and
Programming Languages*, 2005) — linearity, the readable survey. Your "currency"
pillar's foundation.

★ **Appel, *Modern Compiler Implementation in ML*** — SSA, dataflow, register
allocation. Read parts II and III.

**Cooper & Torczon, *Engineering a Compiler*** — the practical alternative;
excellent on dataflow.

**Braun et al., *Simple and Efficient Construction of SSA Form*** ("the SSA
book") — free, short, definitive on SSA construction.

---

## Tier 2 — Memory management (4–6 weeks)

★ **Jones, Hosking, Moss, *The Garbage Collection Handbook*** — chapters 1–4 and
the generational chapter. The definitive reference.

★ **Tofte, Birkedal, Elsman, Hallenberg, *Region-Based Memory Management*
(1997)** — region inference. Your "inheritance" pillar's ancestor. Read this
before claiming novelty.

★ **Reinking, Xie, de Moura, Leijen, *Perceus: Garbage Free Reference Counting
with Reuse* (PLDI 2021)** — the cost-model template. Read for structure as much
as content.

**Lieberman & Hewitt (1983)** and **Ungar (1984)** — the generational hypothesis
and generation scavenging. Short, historical, and the origin of "aging."

**Blackburn et al., *The DaCapo Benchmarks* (2006)** — how memory research is
evaluated.

**Vale / Verdagon, *Generational References*** — the sound dynamic "aging"
mechanism. Blog series is more readable than the paper.

---

## Tier 3 — Ownership and aliasing (4–6 weeks)

★ **Jung et al., *Stacked Borrows: An Aliasing Model for Rust* (POPL 2020)** —
the formal aliasing model. Essential for any provenance-based design.

★ **The NLL RFC** (`rust-lang/rfcs` 2094) — non-lexical lifetimes, the design
document. Surprisingly readable.

★ **The Polonius repository** — the Datalog formulation of borrow checking. The
most practical starting point for extending the checker.

**Tree Borrows** (Villani et al., recent) — the successor model; more permissive.

**Clarke, Potter, Noble, *Ownership Types for Flexible Alias Protection*
(1998)** — ownership types, the formal ancestor of "inheritance of memory."

**Smith, Walker, Morrisett, *Alias Types* (2000)** — unique/aliased pointers,
precursor to borrowing.

**The Rustonomicon** — unsafe Rust and aliasing, practically.

**Hylo / Val papers and blog** — the mutable-value-semantics alternative to
lifetimes. Know it so you can answer "why not just do that?"

---

## Tier 4 — Capabilities and revocation (3–4 weeks)

★ **Klein et al., *seL4: Formal Verification of an OS Kernel* (SOSP 2009)** — the
capability derivation tree and revocation, formally verified. The gold standard.

★ **The seL4 reference manual, capability chapter** — the precise semantics.

**Levy, *Capability-Based Computer Systems* (1984)** — the historical foundation.

**Miller, *Robust Composition* (PhD thesis, 2006)** — the object-capability model.

**Woodruff et al., *The CHERI Capability Model* (2014)** and **Davis et al.,
*CHERI C/C++* (2019)** — capabilities in hardware. Your potential backend.

---

## Tier 5 — Resources and budgets (3–4 weeks)

★ **Hoffmann et al., *Resource-Aware Type Systems*** (RAML and successors) —
amortized analysis. For data-dependent bounds.

**Wadler, *Linear Types Can Change the World!* (1990)** — short, famous.

**The RTSJ specification, scoped memory chapters** — the industrial attempt at
bounded memory in a managed language.

**The Ethereum Yellow Paper, gas sections** — the deployed, successful budget
system. Note that it is static and prepaid.

**Austral language docs** — a small modern linear-types language, readable in a
weekend.

---

## Tier 6 — Concurrency (3–4 weeks)

★ **Boehm & Adve, *Foundations of the C++ Concurrency Memory Model* (PLDI
2008)** — the model you inherit.

★ **McKenney, *Is Parallel Programming Hard...*** — free book; the RCU bible.
Practical and opinionated.

**Fraser, *Practical Lock-Freedom* (2004)** — epoch-based reclamation.

**Michael, *Hazard Pointers* (2004)** — the alternative scheme.

**Jung et al., *Iris from the Ground Up* (2020)** — concurrent separation logic.
The proof technique for anything concurrent.

---

## Tier 7 — Methodology and craft

★ **Georges, Buytaert, Eeckhout, *Statistically Rigorous Java Performance
Evaluation* (OOPSLA 2007)** — read before measuring anything.

★ **Whitesides, *Writing a Paper*** — 2 pages, excellent.

**Hoefler & Belli, *Scientific Benchmarking of Parallel Computing Systems* (SC
2015)** — brutal and correct.

**Sammler et al., *RefinedC* (PLDI 2021)** — a recent, strong paper combining
refinement types, ownership, and separation logic for C. Read it as a model of
the paper you want to write.

**Leroy, *Formal Verification of a Realistic Compiler* (CompCert, 2009)** — what
end-to-end verification costs, and why you will scope it out.

---

## How to work this list

Do not read tier by tier linearly. Instead:

1. **Tier 0 completely.** Non-negotiable.
2. **Then tier 1, chapters 1–2 of TAPL at a time**, with exercises.
3. **Then pick your rung from chapter 12** and read only the tiers that rung
   needs. Rung 3 needs tiers 1, 2, 5. Rung 4 needs tiers 3, 4, 6.
4. **Keep a notes file per paper** (chapter 09 §4).
5. **Reproduce one artifact per month** from tier 2 or 3.

The list is ~40 items. At 2 hours/day, tier 0–2 is about three months. That is the
right pace. Reading faster than you can absorb is the most common way to waste a
year.
