# Chapter 08 — Concurrency: Where These Designs Die

Every memory management scheme works beautifully in a single thread. Then you add
a second core and the whole thing collapses. This chapter explains why, and what
the field does about it — because **your paper will be judged on its concurrency
story**, whether you like it or not.

If you skip this chapter, a reviewer will find the hole in ten minutes.

---

## 1. Why concurrency breaks everything

Three independent problems:

**1. Interleaving.** Two threads execute instructions in an arbitrary order. An
invariant that holds "between statements" may not hold "between two threads'
statements." Your revocation might happen *during* someone's access.

**2. Reordering.** Compilers and CPUs reorder memory operations for performance.
Two writes in program order may become visible in the opposite order to another
core. So "I wrote the pointer before the flag" is not enough — you need a
**fence** or an atomic with the right ordering.

**3. Data races.** Two threads accessing the same location, at least one writing,
without synchronization. In C/C++ this is **undefined behavior** — not "reads a
stale value," but "the compiler may assume it never happens and do anything." This
is why data-race freedom is a *safety* property, not a performance property.

---

## 2. Memory models, in one page

A **memory model** defines which values a read may observe.

- **Sequential consistency (SC):** the interleaving of all operations is some
  total order consistent with program order. The intuitive model. Nobody
  implements it (too slow).
- **Total Store Order (TSO):** x86. Writes are buffered; reads may pass writes.
  Needs fences to recover SC where it matters.
- **Release/acquire:** the portable, minimal synchronization. A *release* store
  makes prior writes visible; an *acquire* load makes subsequent reads see them.
  This is what you should use 95% of the time.
- **Relaxed:** ordering-free atomics. Fast, and a footgun. Only for counters and
  statistics.
- **The C/C++ model** (Boehm & Adve): the standard formalization; data races are
  UB; atomics have explicit orderings. Rust inherits it.

**The only rule you need for research:** if you claim a concurrent algorithm is
correct, you must state **which memory model** you assume, and your proof must be
valid in it. "It works on x86" is not a claim; it is a liability.

---

## 3. Reclamation under concurrency: the four schemes

You cannot free memory while another thread might still read it. Every scheme is
a different answer to "how do I know nobody is looking?"

### Reference counting (atomic)

Increment/decrement are atomic operations. Free when the count hits zero.

- **Correct** if counts are exact.
- **Cost:** an atomic RMW per pointer copy. Under contention this is brutal —
  cache-line ping-pong. This is why `Arc` is slower than `Rc` by a lot.
- **Cycles still leak.** And cycle collection under concurrency is a research
  problem in itself.

### Epoch-based reclamation (EBR)

Threads announce an epoch. Objects retired in epoch `e` are freed once all threads
have advanced past `e`.

```
Thread A: enter epoch e, do work, exit
Thread B: retires object in epoch e
Reclaimer: when all threads have left e, free everything retired in e
```

- **Cost:** two fences per critical section (enter/exit), plus a grace-period
  check. Very cheap in the common case — this is why it is used in Linux RCU and
  Crossbeam.
- **Weakness:** a stalled thread blocks reclamation forever (memory blowup).
  Mitigations exist (bounded EBR, reference counting the epochs).

### Hazard pointers

Each thread publishes the pointers it is currently using. A reclaimer may free an
object only if no thread's hazard list contains it.

- **Cost:** a store per protected pointer, plus a scan of all hazard lists per
  reclamation. Scales poorly with thread count.
- **Strength:** no stalls, bounded memory. Used where EBR's unbounded memory is
  unacceptable.

### RCU (read-copy-update)

Readers never block and never synchronize. Writers create a new version, publish
it, and wait for a **grace period** (all pre-existing readers finished) before
freeing the old.

- **Cost:** zero for readers (the whole point), expensive for writers.
- **Used by:** the Linux kernel, extensively. The most successful concurrent
  reclamation scheme in existence.

**The unifying concept: a quiescence point.** All four schemes are ways of
establishing "no one can be touching this anymore." That is the same problem as
seL4's revocation fence, and it is the same problem you will have.

> **For your paper:** state your **quiescence protocol** explicitly. "Revocation
> is safe because we wait for an epoch grace period, and here is the proof that
> the grace period is sufficient." That sentence, with a proof, is a contribution.
> Without it, your revocation is a use-after-free waiting to happen.

---

## 4. The revocation × concurrency problem (your hardest problem)

Here is the precise difficulty, stated for your model.

You have:
- an object `o`, owned by parent `p`;
- a borrow `b` of `o`, held by thread T;
- a revocation of `p` on thread U.

What must be true for `revoke(p)` to be safe?

**Option A: static exclusion.** The type system proves `b` and `revoke(p)` cannot
be concurrent. This is free at runtime but requires the borrow to be
thread-local or otherwise provably disjoint. **This is the Rust answer** (via
`Send`/`Sync` and lifetimes), and it is the strongest option when it applies.

**Option B: dynamic quiescence.** `revoke(p)` waits until no thread holds a
borrow. This requires tracking borrows at runtime (a refcount or an epoch) and
paying for it. **This is the seL4/EBR answer.**

**Option C: cooperative cancellation.** The revoked object's users are notified
and must respond (like async cancellation). Sound only if every user cooperates —
which means it is a *convention*, not a guarantee.

**Option D: checked access.** Every access validates a generation counter (Vale's
generational references). Sound, uniform, costs a check per access.

**There is no fifth option.** Every safe design is one of these four. Choose one,
state it, prove it. This taxonomy alone will make your related-work section
credible.

---

## 5. Linearizability and how to state a concurrency theorem

For concurrent data structures, the correctness criterion is **linearizability**:
every operation appears to take effect atomically at some point between its
invocation and its response.

You probably will not prove linearizability of a data structure. But you *will*
need to state your theorem in a concurrent setting. The standard shape:

> **Theorem (Concurrent Safety).** For any interleaving of well-typed threads, if
> the program starts in a state satisfying the invariant, then every reachable
> state satisfies the invariant.

The proof technique is almost always **concurrent separation logic** (Iris), where
each thread's local proof is composed using the frame rule. If you are doing
concurrency, learn Iris. There is no substitute.

**A cheaper, very respectable alternative:** prove safety for the *single-threaded*
core, and prove that your concurrency discipline (e.g. `Send`/`Sync`-style
ownership transfer) *reduces* concurrent programs to sequentially-consistent
single-threaded ones. That is a two-layer proof: "the discipline serializes" +
"the serial core is safe." Many strong papers do exactly this, and it is much
more tractable than full CSL. Consider it.

---

## 6. A checklist for your concurrency section

Before you submit, answer all of these in writing:

1. What memory model do you assume? (Name it. "C++20" or "SC" or "TSO".)
2. What is your quiescence protocol, and what is its cost?
3. What happens if a thread stalls forever? (EBR's known weakness — address it.)
4. Is your reclamation bounded in space? (If not, say so and justify.)
5. Can revocation race with a borrow? What prevents it? (Options A–D above.)
6. Are your atomics acquire/release/relaxed, and is each one justified?
7. Does your proof cover the case where revocation and access are *concurrent*,
   not just sequential? (Reviewers will check this first.)

If you cannot answer #7, you do not have a concurrency story, and you should
scope the paper to single-threaded and say so explicitly. That is a legitimate
scope. Pretending otherwise is not.

---

## 7. Reading

- **Boehm & Adve, *Foundations of the C++ Concurrency Memory Model* (PLDI 2008)**
  — the model you inherit.
- **McKenney, *Is Parallel Programming Hard, And, If So, What Can You Do About
  It?*** — free book, the RCU bible, extremely practical.
- **Fraser, *Practical Lock-Freedom* (PhD thesis, 2004)** — EBR.
- **Michael, *Hazard Pointers* (2004)** — the other scheme.
- **Jung et al., *Iris from the Ground Up* (2020)** — concurrent separation
  logic, the proof technique.
- **Herlihy & Shavit, *The Art of Multiprocessor Programming*** — linearizability
  and the fundamentals.

---

**Next:** `09-reading.md` — how to read a PL paper without drowning.
