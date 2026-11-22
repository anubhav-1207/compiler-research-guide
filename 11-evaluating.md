# Chapter 11 — Evaluation: Producing Evidence

A soundness proof says "it cannot be wrong." Evaluation says "and it is worth
using." You need both, and the evaluation is where most first papers are weakest.

---

## 1. What reviewers actually want from evaluation

Three questions, in order of importance:

1. **Does it work at all?** (A demonstration on a real program.)
2. **What does it cost?** (Overhead vs. a real baseline, including worst case.)
3. **Is the cost model accurate?** (Does the predicted bound match reality?)

Notice that raw speed is *third*. In the safety/control space, a 2% overhead with
a proven bound is a better paper than a 20% speedup with no guarantee. **Do not
chase speed; chase predictability.**

---

## 2. Choosing workloads

**Microbenchmarks** isolate a mechanism. Good for showing the cost of one
operation. Bad for credibility if used alone.

**Macrobenchmarks** are real programs. Essential.

For memory/ownership research, the standard suites:

- **DaCapo** — Java, the classic for GC research. Still the reference.
- **Renaissance** — modern JVM suite, broader.
- **SPEC CPU** — the general-purpose standard. Expensive, respected.
- **PARSEC** — parallel workloads.
- **Real-world codebases** — a compiler, a web server, a game engine. Increasingly
  preferred over synthetic suites because reviewers have seen DaCapo a thousand
  times.

**For your specific area**, the best workloads are ones with the property you
claim to help:

- **Real-time**: audio processing, control loops, anything with a deadline. Show
  the *tail latency*, not the average.
- **Multi-tenant**: a plugin host, a server with per-request budgets. Show that
  one tenant cannot starve another.
- **Embedded**: anything with a hard memory ceiling. Show the proven bound is
  respected.

> **For your paper:** pick 3–5 workloads that *stress your claim*, not 15 that
> flatter it. A reviewer will trust a narrow, honest evaluation far more than a
> broad, suspicious one.

---

## 3. Methodology (the part people get wrong)

Performance measurement is a science. Do it properly.

**Repetitions and variance.** Never report a single run. Report the median and a
confidence interval over ≥10 runs. Java/JVM work needs warmup and many iterations
(see Georges et al. below).

**Pin the environment.** Same machine, same compiler version, same flags, same
CPU governor, no background load. Record all of it in the artifact.

**Compare fairly.** The baseline must be the best realistic alternative, compiled
with the same effort. "Our system vs. unoptimized C" is a strawman.

**Report the distribution.** For latency claims, report p50, p99, p99.9, and max.
Averages hide the pauses that matter.

**Report memory, not just time.** In this field, space is often the binding
constraint. Bytes per object, peak RSS, and fragmentation.

**Separate compile time from run time.** If your analysis is expensive to run,
that is a real cost. Report it.

---

## 4. Evaluating a formal result

If you mechanized a proof, "it compiles" is not enough. Report:

- **What is mechanized.** The calculus? The semantics? The type system? The
  theorems? Be precise about the boundary.
- **What is assumed.** "We assume the back end preserves semantics" — state it.
- **Axioms.** Did you assume anything (`Parameter`, `Axiom`, `Admitted`)? **Count
  them and list them.** A proof with hidden axioms is not a proof.
- **Size.** Lines of specification vs. lines of proof. The ratio tells reviewers
  how much work it was.
- **Which tool and version.** Reproducibility.

The standard for a strong paper: the core calculus is mechanized, the theorems are
complete with no `Admitted`, and the implementation is separate and unverified but
clearly scoped.

---

## 5. The tightness metric (your secret weapon)

For a budget system, the key number nobody else reports is:

> **tightness = proven bound / actual usage**

Report the distribution of this ratio across your workloads: median, worst case.
If your median is 1.2× and your worst is 4×, that is a *strong* result and it
should be a headline number. If your median is 50×, your bound is technically
sound and practically useless — say so, and explain what would tighten it.

This single metric will differentiate your paper from every hand-wavy "bounded
memory" claim in the literature. It is also easy to compute and hard to argue
with.

---

## 6. Threats to validity

Write this section. It is not an admission of weakness; it is a demonstration of
rigor. Standard categories:

- **Internal validity:** did your measurement actually measure what you claim?
  (e.g. did the JIT optimize away the thing you were timing?)
- **External validity:** do your workloads represent real use? (Your 5 benchmarks
  are not the world.)
- **Construct validity:** does your metric capture the property? (Does "average
  allocation" capture "real-time safety"? No — use tail latency.)
- **Conclusion validity:** are your statistics sound? (Enough repetitions?
  Appropriate test?)

A short, honest threats section preempts three reviewer objections.

---

## 7. The artifact

Artifact Evaluation is a separate track at most venues and a strong artifact can
carry a borderline paper.

Minimum viable artifact:

- **One command to build.** `docker build` or `make`. If it needs 12 manual steps,
  you have already lost.
- **One command to reproduce each figure.** A script per figure, producing the
  exact data.
- **A README that states the expected result** for each experiment, so the
  reviewer knows if it worked.
- **Pinned dependencies.** No "latest."
- **A smoke test** that runs in under 5 minutes, so reviewers can sanity-check
  before committing hours.

Budget two full weeks for artifact preparation. It is always more work than
expected, and it is often the difference between "weak accept" and "accept."

---

## 8. A minimal evaluation plan for your first paper

If you have limited time, do exactly this:

1. **One motivating case study.** A real program where your system does something
   the baseline cannot (e.g. a plugin host that provably cannot exceed its
   budget). Qualitative, but powerful.
2. **Three benchmarks** with overhead vs. the best baseline. Median + CI.
3. **The tightness distribution** for your budget bounds.
4. **A worst-case analysis.** Where does your approach degrade, and by how much?
5. **Compile-time cost** of your analysis.

That is a complete, credible evaluation for a workshop or a first OOPSLA
submission. Resist the urge to add more benchmarks; add *depth* instead.

---

## 9. Reading

- **Georges, Buytaert, Eeckhout, *Statistically Rigorous Java Performance
  Evaluation* (OOPSLA 2007)** — the methodology paper. Read it before measuring
  anything on a JVM.
- **Blackburn et al., *The DaCapo Benchmarks* (OOPSLA 2006)** — how memory
  research is evaluated, and why.
- **Hoefler & Belli, *Scientific Benchmarking of Parallel Computing Systems*
  (SC 2015)** — brutal and correct about benchmarking sins.
- **The artifact-evaluation guidelines** of your target venue — read them before
  you design the experiments, not after.

---

**Next:** `12-project-ladder.md` — five concrete projects, from toy to publishable.
