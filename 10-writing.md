# Chapter 10 — Writing the Paper

A paper is not a report on what you did. It is **an argument that a specific claim
is true, supported by evidence.** Everything below serves that argument.

Two essays to read before anything else, both free and short:

- **Simon Peyton Jones, *How to Write a Great Research Paper*** — the single best
  document on this. Read it twice.
- **George Whitesides, *Writing a Paper*** — on outlining and on knowing your
  one point.

---

## 1. The one-sentence test

Before you write a word, complete this sentence:

> "We show that **___**, which matters because **___**, and we demonstrate it by
> **___**."

If you cannot fill it in, you do not have a paper. If it takes more than one
sentence, you have two papers.

For your work, a plausible version:

> "We show that **allocation budgets and hierarchical revocation can be
> integrated into an ownership type system without runtime checks on the hot
> path**, which matters because **real-time and multi-tenant systems need
> provable resource bounds and safe reclamation**, and we demonstrate it by **a
> mechanized soundness proof, a cost theorem, and a prototype with ≤5% overhead
> on [workload]**."

That is a paper. Note what it is not: it is not "a new paradigm inspired by
biology." That is the *introduction's* framing, not the claim.

---

## 2. Structure, with what each section must accomplish

**Title.** Concrete, with the mechanism named. "Bounded Ownership: Static
Allocation Budgets for an Ownership Type System" beats "Life: A Biological
Approach to Memory." Reviewers search by keyword; give them the keyword.

**Abstract.** Four sentences: problem, gap, idea, result-with-number. No
adjectives. No "novel."

**Introduction.** The most important section. Structure it as a funnel:

1. **The world as it is.** "Real-time systems need bounded memory; multi-tenant
   hosts need safe revocation."
2. **Why existing solutions fail.** Rust has no budgets. seL4 has no language.
   RTSJ is unloved. Be fair and specific.
3. **Your idea, in one paragraph.** No formalism yet.
4. **A motivating example.** A small program that is painful today and pleasant
   under your system. **This is where you win or lose the reader.** Spend real
   time on it.
5. **Contributions, as a bulleted list.** Each bullet must be checkable against
   the rest of the paper. Reviewers literally use this as a checklist.
6. **A "results in brief" sentence** with your headline number.

**Formal core.** The calculus and rules. Keep it minimal. Every construct is a
case in every lemma and a paragraph of explanation. If a construct is not needed
for the theorem, cut it. (You can mention the full language in the
implementation section.)

**Metatheory.** State the theorems *precisely*. Give proof sketches that identify
the hard case. Point to the appendix or the mechanization. Never write "the proof
is straightforward."

**Implementation.** What you actually built, what you approximated, what was
hard. Honesty here builds enormous credibility. "We approximate indirect calls by
assuming the worst case, which costs 12% on one benchmark" is a *good* sentence.

**Evaluation.** Chapter 11.

**Related work.** Position, do not list. For each close work: "X does A but not
B; we do both." If someone did your idea in 1998, say so and explain what is
different. **Hiding a predecessor is the fastest way to get rejected** — the
reviewer is often that predecessor.

**Conclusion.** Restate the claim and the evidence. Future work: 3 bullets max.

---

## 3. Writing a theorem

A theorem statement is a contract. It must be:

- **Precise.** Every symbol defined. Every assumption stated.
- **Falsifiable.** A reader must be able to imagine a counterexample.
- **Bounded in scope.** "For the core calculus, assuming the back end is
  correct" — state the assumption.

Bad: "Our system is safe."
Good: "For any well-typed program `e` and store `μ` satisfying the invariant, if
`⟨e, μ⟩ ⟶* ⟨e', μ'⟩` and `e'` is not a value, then there exists `e''` with
`⟨e', μ'⟩ ⟶ ⟨e'', μ''⟩`."

The second one is *checkable*. That is the difference between a paper and a
brochure.

### Proof sketches

A good sketch is 3–5 sentences:

1. "By induction on the derivation of the step relation."
2. "The interesting case is [X], where [the subtle thing happens]."
3. "It is resolved by [lemma Y], which holds because [reason]."
4. "All other cases are routine."

Naming the hard case *is* the contribution of the sketch. It tells the reviewer
you actually did the proof.

---

## 4. Writing a cost model section

Give it its own subsection. State:

1. **The unit.** Bytes? Allocations? Cycles? Be explicit.
2. **The accounting rule.** What counts and what does not. (Does the allocator
   header count? The metadata?)
3. **The theorem.** "Total allocation ≤ f(types)."
4. **The constants.** Measured, with methodology.
5. **The tightness.** "The bound is within 1.4× of actual usage on average, 3×
   worst case." **This number is more important than your speedup.** A loose
   bound is a weak result; report it honestly and it becomes a strength.

---

## 5. The rebuttal

Most top venues have a short author response. Rules:

- **Answer the factual objections first.** If a reviewer says "you did not cite
  X," cite X (or explain precisely why not).
- **Concede gracefully.** "The reviewer is right that we do not handle case Y; we
  have added a paragraph scoping this." Reviewers reward honesty.
- **Do not argue about taste.** "We believe our design is cleaner" wins nothing.
- **Be short.** Long rebuttals read as defensive.
- **Never promise work you have not done.** "We will add" is weak; "we have added"
  (in the revision) is strong.

Rebuttals rarely flip accept/reject on their own. They matter most for borderline
papers, and they matter enormously for the *next* submission — reviewers remember
who was gracious.

---

## 6. Dealing with rejection

You will be rejected. The acceptance rate is 15–25%; the expected number of
submissions to a first acceptance is 2–3. This is normal and it is not a verdict
on you.

**What to do with reviews:**

1. Wait 72 hours before reading them carefully. Seriously.
2. Extract the *actionable* criticisms. Ignore the tone.
3. Categorize: (a) real flaw, (b) missing experiment, (c) misunderstanding
   (your writing's fault), (d) taste.
4. Fix (a) and (b). Fix (c) by rewriting — if a smart reviewer misunderstood, so
   will others. Ignore (d).
5. Resubmit, usually to the next venue, with the fixes. **Most accepted papers
   were rejected somewhere first.**

---

## 7. Ethics of claiming soundness

This deserves its own paragraph because it is where this field gets corrupted.

- **Do not claim more than you proved.** If you proved the core calculus and not
  the implementation, say "the core calculus is verified; the implementation is
  not."
- **Do not call a test suite a proof.** Testing finds bugs; it does not establish
  safety.
- **Do not hide the runtime check.** If your "zero-cost" abstraction has a check
  in one path, report it.
- **Do not omit the predecessor.** Cite the 1998 paper. It costs you one sentence
  and buys you the reviewer's trust.
- **Do not present a bound you know is vacuous.** "At most 1 TB" is technically
  true and intellectually dishonest. Report tightness.

A reputation for honesty is the only durable asset in this field. Papers get
forgotten; "that person's numbers are trustworthy" does not.

---

## 8. A writing checklist

Before submitting, verify:

- [ ] The one-sentence claim is in the abstract, intro, and conclusion, worded
      consistently.
- [ ] Every contribution bullet is delivered somewhere in the paper.
- [ ] Every theorem is stated precisely, with its assumptions.
- [ ] Every theorem has a proof sketch naming the hard case.
- [ ] The cost model has a unit, a rule, a theorem, constants, and tightness.
- [ ] The evaluation has a real baseline, real workloads, and variance.
- [ ] Related work cites the obvious predecessors.
- [ ] Limitations are stated explicitly (a "Limitations" paragraph is a strength,
      not a weakness).
- [ ] The artifact builds from a clean checkout with one command.
- [ ] A colleague outside your subfield can read the introduction and explain
      your idea back to you.

That last one catches more problems than all the others combined.

---

**Next:** `11-evaluating.md` — the evidence half of the argument.
