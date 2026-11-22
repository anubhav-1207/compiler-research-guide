# Chapter 00 — Orientation: The Field, the Paper, the Gap

## 1. What a compiler actually is

A compiler is a program that translates a source language into a target language
*while preserving meaning*. That is the whole definition, and the word
"preserving" is where all the research lives.

- If the source is C and the target is x86, "preserving meaning" means "the
  observable behavior matches the C standard's abstract machine."
- If the source has a garbage collector and the target does not, "preserving"
  means the compiler must *insert* the reclamation machinery.
- If the source has ownership and the target has raw addresses, "preserving"
  means the compiler must *prove* that the ownership rules make the raw accesses
  safe.

Every safety paper is, at bottom, an argument about this word. **You are not
writing a translator. You are writing an argument that a translation is
meaning-preserving.**

That reframing is important: it means your contribution is usually a *proof
obligation*, not a feature.

---

## 2. The two cultures inside PL research

Programming languages research splits roughly in two, and you need literacy in
both even if you specialize.

**The semantics / types culture.** Concerned with *what programs mean* and *what
can be proven*. Writes calculi (tiny formal languages), typing rules, and
theorems. Venues: POPL, ICFP, LICS, CPP/ITP. Papers look like: "Here is a
calculus. Here are the rules. Theorem: well-typed programs don't get stuck.
Proof: appendix, 14 pages."

**The systems / implementation culture.** Concerned with *what actually runs
fast*. Writes compilers, runtimes, benchmarks. Venues: PLDI, OOPSLA, ASPLOS,
OSDI. Papers look like: "Here is an optimization. Here is its implementation in
LLVM. Here is 12% on SPEC. Here is the artifact."

**Your work is inherently in the middle**, because "safety × control" is exactly
the seam between them. This is both an opportunity (fewer people can do both)
and a hazard (reviewers from each culture will judge you by the other's
standards). Expect a systems reviewer to ask "where are the numbers?" and a
types reviewer to ask "where is the soundness proof?" **You need both.**

> **For your paper:** plan for a mechanized (or at least rigorous) soundness
> argument *and* a measured overhead. If you have only one, you are writing for
> half the community and the other half will reject you.

---

## 3. What makes a paper "meaningful"

A publishable contribution in this area almost always has three components.
Miss one and it is a blog post.

**(a) A new abstraction.** A concept that was not nameable before. Examples:
"region" (Tofte–Talpin), "borrow" as a first-class static permission (Rust),
"capability derivation tree" (seL4), "generational reference" (Vale). Note: none
of these is a *mechanism* — they are *vocabularies that make a proof possible*.

**(b) A soundness argument.** A precise statement of what cannot go wrong, and a
proof. Increasingly this means a machine-checked proof, because reviewers have
been burned by hand-waved ones.

**(c) Evidence.** Either measurements (overhead, throughput, memory) or a
demonstration on real programs, or both.

The strongest papers in this space have a fourth thing: **a cost model**. Not
just "it is safe," but "it is safe *and* here is the exact price, expressed in
the language's own terms." Perceus (Koka/Lean) is the canonical example: it
gives you a *precise* reference-counting cost, not a hand-wave.

> **For your paper:** the "life" model's best chance is (a) + (b) + (d). The
> vocabulary is your contribution. The mechanism is borrowed. Be explicit about
> that; reviewers respect honesty about what is new.

---

## 4. Anatomy of a PL paper

Almost every paper in this area has this skeleton. Learn it and you can read
anything.

1. **Abstract** — problem, idea, result, number. Four sentences.
2. **Introduction** — why the world is broken, what everyone else tried, what
   you do, a bulleted list of contributions. The contributions list is a
   contract; reviewers check it.
3. **Motivating example** — one small program that is painful today and pleasant
   under your system. This is where you win or lose the reader.
4. **Formal core** — the calculus, the typing/effect rules, the operational
   semantics. Usually 2–4 pages of rules.
5. **Metatheory** — the theorems, proof sketches, and a pointer to the appendix
   or the mechanization. Usually 1–3 pages.
6. **Implementation** — how it maps to a real compiler, what was hard, what was
   approximated.
7. **Evaluation** — benchmarks, overhead, comparison to baseline.
8. **Related work** — positioned honestly. Reviewers will check that you cited
   their paper.
9. **Conclusion / future work.**

The proportion matters: **the formal core plus metatheory is usually 40% of the
paper and 90% of the risk.** Budget your time accordingly.

---

## 5. The paper economy (read this before you dream)

- **Top venues** (POPL, PLDI, OOPSLA, ICFP) accept roughly 15–25%. Two to four
  reviewers. Reviews are long and technical. Rebuttals exist but rarely flip a
  decision on their own.
- **OOPSLA** (now part of PACMPL) has a *two-phase* review and is generally
  considered friendlier to implementation-heavy work with a solid artifact.
  **This is likely your best first target.**
- **Workshops** (e.g. at SPLASH, ICFP, PLDI) are where first papers should go.
  Low prestige, high value: you get feedback, a citation, and a line on a CV.
- **arXiv** is where you establish priority and get feedback before submission.
  Post early, post often.
- **Artifact Evaluation (AE)** is a separate track at most venues. A strong,
  reproducible artifact can carry a paper. Budget real time for it (Docker,
  scripts, one-command reproduction).

**The single most important structural fact:** you almost certainly need a
co-author who has published before. Not for prestige — for *calibration*. First
authors without mentorship systematically overclaim, under-evaluate, and
misjudge reviewer expectations. Find someone (a professor, a senior engineer at
a compiler company, a PhD student) and offer them a concrete, narrow idea plus
your labor. That trade is usually accepted.

---

## 6. Where the actual gaps are

Here is an honest map of open problems at the safety × control seam. These are
your candidate territories. Note how each is a *narrowing* of the life metaphor.

**(1) Bounded memory with static proof.** No mainstream language lets you say
"this component allocates at most N bytes, and I can prove it." Rust has no
budget. C has no proof. Real-time Java's scoped memory is close but clunky and
unloved. **This is the most underserved gap and the most tractable.** A type
system for allocation budgets, integrated with ownership, with a soundness
theorem and a measured cost, is a paper.

**(2) Hierarchical revocation for linear/ownership types.** seL4 proves
revocation for *capabilities* in a kernel. Nobody has cleanly integrated
transitive, monotone revocation into a *language* with ownership types, where the
borrow checker must reason about revocation invalidating borrows. The fence
problem (when is it safe to revoke?) is genuinely open in a language setting.
**This is your "overpowering" pillar, and it is publishable.**

**(3) Generational references with low overhead.** Vale's generational references
give dynamic safety without a GC, at the cost of a check per dereference. Can you
get the same guarantee with fewer bits, or with checks hoisted out of loops by
the compiler? This is a well-defined optimization problem with a clear baseline.

**(4) Ownership without lifetimes.** Hylo/Val argues you can get memory safety
from *mutable value semantics* alone, with no lifetime annotations. Whether that
scales to systems code, and what it costs, is open.

**(5) Escape analysis quality.** Every dynamic scheme is only affordable because
escape analysis proves most objects don't escape. Better escape analysis is
unglamorous and extremely valuable, and it is the enabling technology for
everything else in this list.

**(6) Concurrency + revocation.** Every reclamation scheme has a quiescence
protocol. Integrating revocation with epoch-based reclamation in a *typed*
language, with a proof, is hard and open.

> **For your paper:** pick exactly one of these. The life metaphor should be the
> *framing*, not the contribution. "Life-like" gets you a nice introduction;
> "a sound type system for X with a cost model" gets you accepted.

---

## 7. The trilemma, restated for researchers

Every paper you read is choosing two of:

1. **Control** — predictable layout, addresses, no hidden cost.
2. **Static safety** — proven, no runtime checks.
3. **Unrestricted dynamic reclamation** — objects die whenever.

When you read a paper, immediately identify which corner it gives up and whether
it admits it. When you write one, state your corner in the introduction. Papers
that pretend to have all three are either wrong or hiding a runtime check
somewhere.

This single habit will make you a better reader than most reviewers.

---

## 8. Honest timeline

| Phase | Duration | Output |
|---|---|---|
| Study (this guide) | 3–5 months | Literacy; can read papers |
| Toy formalization | 2–3 months | A mechanized calculus with a theorem |
| Prototype | 3–6 months | A pass or checker on a real IR |
| Workshop paper | 1–2 months | Feedback, first citation |
| Full paper | 4–8 months | Submission, likely 1–2 rejections before acceptance |

Total to a first top-tier acceptance: **typically 2–4 years** from a standing
start without an advisor, faster with one. That is not a discouragement; it is a
calibration. Plan for it and you will not burn out at month eight.

---

**Next:** `01-the-machine.md` — the compilation pipeline, from characters to
registers, with the places where memory management lives.
