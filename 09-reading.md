# Chapter 09 — Reading Papers

You will read 100–300 papers before you publish. Reading them badly wastes
months. This chapter is the method.

---

## 1. The three-pass method (Keshav)

The standard technique. Do not read linearly.

**Pass 1 — 5 minutes. Triage.**
Read: title, abstract, introduction, section headings, conclusions, figures.
Answer: *What problem? What is the idea? What is the result?* If you cannot
answer, either the paper is bad or you need a different paper. **Do not continue
past pass 1 unless the answer excites you.**

**Pass 2 — 1 hour. Comprehension.**
Read the body, skip proofs. Look at every figure and table carefully (they carry
most of the information). Mark unread references. Answer: *What are the
assumptions? Is the approach sound? What is the evidence?* Now you can summarize
the paper to someone else.

**Pass 3 — 4–5 hours. Reconstruction.**
Only for papers you will build on. Re-derive the theorems yourself. Re-implement
the key idea mentally. Challenge every assumption. This is where you find the
hole that becomes your contribution.

**Rule of thumb:** 10 papers at pass 1, 3 at pass 2, 1 at pass 3, per week. Do not
invert that ratio.

---

## 2. How to read a PL paper specifically

PL papers have a predictable structure (chapter 00). Exploit it.

**Find the calculus first.** Somewhere there is a box with 6–12 rules. That is
the paper. Everything else is motivation or evidence. Read the rules *before* the
prose about them.

**Then find the theorem.** Search for "Theorem." There will be 1–4. Write each
one down verbatim. If you cannot state the theorem, you do not understand the
paper.

**Then find the proof sketch.** Usually 1–3 sentences plus "see appendix." Ask:
*what is the hard case?* The hard case is where the research is. If the paper
glosses it, that is a gap — and possibly your paper.

**Then find the cost model.** Does the paper say what it costs? In what units? If
it does not, it is incomplete, and you can improve on it.

**Then find the evaluation.** What is the baseline? What is the workload? Is the
overhead reported honestly (including worst case)?

### The type-rule reading habit

When you hit a block of rules, do this:

1. **Identify the judgment.** What is being derived? (`Γ ⊢ e : τ`? `⟨e,μ⟩ ⟶ ⟨e',μ'⟩`?)
2. **Identify the context.** What is in `Γ`? What extra structure (store typing,
   capability set, budget) has been added?
3. **Find the interesting rule.** The one that is *new*. It is usually 3–5 rules
   out of 20.
4. **Check the side conditions.** The `where` clauses. That is where the
   subtlety lives — "provided `x ∉ dom(Γ)`", "provided the region is live."
5. **Ask: what breaks if I delete this rule?** If nothing breaks, the rule is
   decoration. If everything breaks, that is the load-bearing idea.

---

## 3. How to read a systems paper specifically

Different checklist:

1. **What is the baseline?** If the baseline is naive, the speedup is meaningless.
2. **What is the workload?** Microbenchmarks or real programs? Both is ideal;
   only micro is a red flag.
3. **What is the overhead on the *worst* case?** Average speedups hide cliffs.
4. **Is the artifact available?** If not, treat the numbers as provisional.
5. **What did they not measure?** Memory? Compile time? Tail latency? The missing
   metric is often where your contribution is.

---

## 4. Note-taking system

Keep one file per paper, with fixed fields. This is not optional; you will forget.

```markdown
# Author Year — Short Title
Venue:
Pass: 1 / 2 / 3

## Problem
## Key idea (one sentence)
## Calculus / mechanism
## Theorem(s)
## Cost model
## Evaluation (baseline, workload, headline number)
## What I believe
## What I doubt
## How it relates to my work
## References to chase
```

The two fields that matter most are **"What I doubt"** and **"How it relates."**
Those are the seeds of your contribution. Everything else is recall.

Use a reference manager (Zotero is fine) but **do not** rely on it for thinking.
The markdown notes are the thinking.

---

## 5. Reading groups and reproduction

If you can find or form a reading group, do it. Explaining a paper to someone
else is 5× more effective than reading it alone.

**Reproduce one result per month.** Pick a paper with an artifact, run it, and try
to match one number. This teaches you more about evaluation methodology than any
book, and it gives you a concrete credential ("I reproduced X") that matters when
you approach potential co-authors.

---

## 6. Red flags (train your suspicion)

In a PL paper:

- **"The proof is straightforward and omitted."** Usually means it was not done.
- **A theorem with no proof sketch at all.** Same.
- **No cost model** but a performance claim. How would you know it is not
  accidentally quadratic?
- **A calculus with 30 constructs.** Probably an implementation paper wearing a
  formal hat; the metatheory is likely shallow.
- **Related work that omits the obvious predecessor.** Check. Always check.
- **"We leave the concurrent case to future work."** Fine if stated, fatal if the
  claim implies concurrency.

In a systems paper:

- **Baseline is a strawman.**
- **Only microbenchmarks.**
- **Speedup reported without variance or repetitions.**
- **No artifact, no reproduction instructions.**
- **"Our prototype" with no indication of what was approximated.**

---

## 7. Building your reading list

Start from three seeds and expand by **backward and forward citation**:

1. **Backward:** read the related-work section of a good recent paper, and follow
   the 5 most-cited references.
2. **Forward:** use Google Scholar's "cited by" on a foundational paper to find
   what came after. The *recent* citations are your competition.
3. **Venue browsing:** read the last 3 years of PLDI/OOPSLA/POPL tables of
   contents. You are looking for the paper that is closest to your idea — because
   you need to know if you have been scooped, and because it is your best model
   for how to write yours.

Chapter 13 gives you a prioritized starting list. Work it in order.

---

## 8. The one habit that matters

**After every paper, write one sentence: "The gap this leaves is ___."**

That sentence is either a research idea or a reason to stop reading that paper.
After 50 papers you will have a list of 50 gaps, and 2 or 3 of them will be real.
That list *is* your research program. This is how the field actually works.

---

**Next:** `10-writing.md` — turning a gap into an accepted paper.
