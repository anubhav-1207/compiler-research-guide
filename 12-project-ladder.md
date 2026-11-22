# Chapter 12 — The Project Ladder: Five Papers, Toy to Publishable

Knowledge without a project evaporates. This chapter converts the guide into
work. Each rung is a **complete, scoped paper-sized unit** with a claim, a formal
core, an implementation, an evaluation, and a stated risk.

Do them in order. Rung 1 is non-negotiable — it is the proof you can do the rest.
Rungs 3–5 are the ones that get published; pick **one**, not all.

---

## Rung 0 — Reproduce something (2–4 weeks)

**Goal:** build the mechanical skill and the credibility.

Pick a paper with an artifact: Perceus in Koka, Stacked Borrows in Miri, or a
DaCapo GC result. Build it, run it, match one number.

**Deliverable:** a short write-up: "I reproduced figure 4 of X; here is my
number, here is theirs, here is the delta and why."

**Why it matters:** it teaches you evaluation methodology, gives you something
concrete to show a potential advisor, and it is the fastest way to discover
whether you actually enjoy this work. If you hate rung 0, stop — the rest is
more of the same, harder.

---

## Rung 1 — A mechanized toy: budget types on STLC (2–3 months)

**This is the single most important project. Do not skip it.**

**Claim:** "We extend the simply-typed lambda calculus with linear allocation
tokens and prove (a) safety — well-typed programs never get stuck, and (b)
boundedness — a well-typed program's allocation is bounded by a function of its
type."

**Formal core:** STLC + a heap + `alloc`/`free` + a linear `Token n` type.
About 10 rules. Two theorems.

**Implementation:** none. This is pure Coq or Lean 4.

**Evaluation:** the proof itself. Report: lines of spec, lines of proof, axioms
(zero), and the statement of the two theorems.

**Why this specific project:** it contains, in miniature, *every* skill you need:
- defining a calculus (chapter 03),
- designing typing rules (chapter 02),
- proving preservation/progress (chapter 03),
- stating a cost theorem (chapter 07),
- mechanization (chapter 03).

When you finish it, you will be able to read the formal core of any paper in this
area, because you will have written one.

**Risk:** low. This is well-trodden; Software Foundations will carry you.
**If it takes 4 months, that is normal.** Do not be discouraged.

**What it is not:** novel. Nobody will publish it. That is fine. It is your
apprenticeship piece.

---

## Rung 2 — A prototype checker on a real IR (3–6 months)

**Goal:** prove you can implement, not just formalize.

**Project:** implement an **escape analysis pass** on LLVM IR (or on a small
language you write), and measure how many allocations it moves to the stack.

**Claim:** "Our escape analysis identifies X% of allocations as non-escaping on
these workloads, enabling stack allocation at zero runtime cost."

**Why escape analysis:** it is the linchpin of every dynamic memory scheme
(chapter 04), it is well-defined, it has clear baselines (LLVM's existing
analysis, Java's), and it is *tractable*. It is also the enabling technology for
your later papers.

**Implementation:** an LLVM pass. Constraint graph over SSA values; solve to
fixpoint; annotate allocas. ~2000 lines.

**Evaluation:** % of allocations stack-allocated; runtime delta; compile-time
delta. Compare to LLVM's `-passes=ea` baseline.

**Risk:** medium. LLVM's API is large; budget time for learning it.
**Payoff:** a working artifact and the implementation credibility that makes
rungs 3–5 believable.

---

## Rung 3 — The first real paper: bounded ownership (6–9 months)

**This is your most likely first publication. Target OOPSLA or a workshop first.**

**Claim:** "We present an ownership type system with static allocation budgets.
Well-typed programs (a) never exceed their declared budget, (b) fail safely when
the budget is exhausted, and (c) require no runtime checks on the hot path. We
prove this and implement it, with ≤N% overhead and a median bound tightness of
M×."

**Novelty:** budgets exist (RTSJ, gas, seL4). Ownership exists (Rust). **The
integration, with a soundness proof and a cost theorem, does not exist in a
mainstream-style language.** That is your contribution, and it is honest.

**Formal core:** a Rust-like core calculus (ownership, borrows, drop) + an effect
annotation `--alloc(B)-->` on functions + a linear budget token at the top level.
Theorems: safety (no stuck states, including budget exhaustion) and boundedness.

**Implementation:** a checker for a small Rust-like language, or an annotation
checker over MIR. Start with a standalone type checker on a toy language; porting
to MIR is a stretch goal.

**Evaluation:** three workloads with deadlines (audio, control loop, request
handler); overhead; **tightness distribution** (chapter 11 §5); compile-time cost.

**Risk:** medium-high. The hard part is *composition*: budget summaries for
higher-order and indirect calls. Scope it: "for first-order programs" is an
acceptable limitation if stated.

**Why this is the best first paper:** it is the least crowded gap, it has a clear
industrial motivation (real-time, WASM, embedded), the formal core is small, and
the evaluation is straightforward.

---

## Rung 4 — The ambitious paper: revocation for borrows (9–15 months)

**Claim:** "We extend an ownership type system with hierarchical, transitive
revocation, and prove that revocation cannot invalidate a live borrow. We give a
quiescence protocol with cost O(...) and show it on a multi-tenant plugin host."

**Novelty:** revocation is proven in seL4 (a kernel, not a language). Borrow
checking is proven in Rust (no revocation). **The composition is open.** This is
the strongest novelty of the five, and the hardest.

**Formal core:** ownership + borrows + a capability derivation tree + a revoke
operation + a quiescence relation. Theorems:
1. Revocation safety: no live borrow refers to a revoked object.
2. Progress: revocation always terminates and reaches a quiescent state.
3. (Stretch) Concurrency: the protocol is sound under a stated memory model.

**Implementation:** a runtime with an epoch-based quiescence protocol, plus a
static check that borrows do not cross revocation points where provable.

**Evaluation:** a plugin host where one tenant is revoked mid-execution; measure
revocation latency, throughput under revocation, and the overhead on the
non-revoking path.

**Risk:** high. The concurrency proof is genuinely hard (chapter 08 §4). If you
cannot do the concurrent case, **scope to single-threaded and say so** — that is
still a paper.

**The fence is the paper.** If you can state and prove a quiescence protocol for
language-level revocation, that alone is a contribution.

---

## Rung 5 — The synthesis: the "life" model, done honestly (2+ years)

**Only attempt this after rungs 3 and 4.**

**Claim:** "We present a unified memory model in which ownership (inheritance),
generational placement (aging), static budgets (currency), and hierarchical
revocation (overpowering) are four facets of one type system, with a single
soundness theorem and a compositional cost model."

**This is the paper you actually want to write.** It is also the paper that
requires rungs 3 and 4 as lemmas. Do not start here.

**What makes it publishable rather than grandiose:** the *composition theorem*.
Anyone can list four features. The contribution is: **the four interact safely**,
and you prove it. Specifically:

- Revocation must not break a budget (does revoking free budget? to whom?).
- Aging must not break ownership (promotion must preserve provenance).
- Budgets must compose across revocation boundaries.
- Inheritance must be monotone under all three operations.

Each interaction is a lemma. Four interactions, four lemmas, one theorem. **That
is the paper.** It is a real, hard, and novel contribution — and it is exactly
the "life" idea, stripped of the parts that cannot be proven.

---

## How to get a co-author

You need one. Here is the honest approach:

1. **Finish rung 1 first.** Then you have something to show.
2. **Find the 5 people closest to your idea.** Read their papers (pass 3). Find a
   gap they left.
3. **Email them with a specific, narrow proposal**, not "will you mentor me."
   Something like: "I mechanized budget types on STLC (link). I want to extend it
   with revocation. Would you be interested in collaborating, or in pointing me
   at the right prior work?"
4. **Offer labor.** Implementation, proofs, artifact. Senior people are
   time-poor; a competent implementer is valuable.
5. **Be patient and be useful.** One good interaction beats ten cold emails.

Also: **workshops and doctoral symposia** (SPLASH SRC, PLMW, OOPSLA doctoral
symposium) exist precisely to pair newcomers with mentors. Apply.

---

## A realistic timeline

| Month | Activity | Milestone |
|---|---|---|
| 1–5 | Study (this guide) | Literacy |
| 6–8 | Rung 0 + Rung 1 | Mechanized toy calculus |
| 9–14 | Rung 2 | Working LLVM pass |
| 15–22 | Rung 3 | Workshop paper → OOPSLA submission |
| 23–34 | Rung 4 | Second paper / revision |
| 35+ | Rung 5 | The synthesis |

Two to three years to a first top-tier acceptance, working part-time. Faster with
an advisor, slower alone. **This is normal.** Everyone's first paper took this
long; they just do not tell you.

---

## The decision that determines everything

At every rung, ask: **"what is the smallest true theorem I can state?"**

The temptation is always to make the claim grander. Resist it. A small theorem,
mechanized, with a measured cost, beats a grand vision every time. The grand
vision is the *introduction*. The theorem is the *paper*.

---

**Next:** `13-bibliography.md` — the prioritized reading list.
