# Chapter 01 — The Machine: Pipeline, IR, SSA, Dataflow

You cannot publish about a compiler without knowing where in the compiler your
idea lives. This chapter is the map. Everything here is standard; the goal is
that you can draw it on a whiteboard and point to the exact box your research
occupies.

---

## 1. The pipeline, in one picture

```
source text
   │  lexing        characters → tokens
   ▼
tokens
   │  parsing       tokens → tree
   ▼
AST (abstract syntax tree)
   │  name resolution, type checking
   ▼
typed AST
   │  lowering      high-level constructs → simple operations
   ▼
IR (intermediate representation)  ← most research lives here
   │  optimization passes (dataflow, SSA, inlining, ...)
   ▼
optimized IR
   │  instruction selection, register allocation
   ▼
machine code
```

Two things to internalize:

**The IR is the contract.** Front ends are language-specific and boring. Back
ends are machine-specific and boring. The IR in the middle is where the
interesting, reusable ideas live. When a paper says "we implemented it as an
LLVM pass," it means "we work on the IR."

**Lowering is where semantics get destroyed.** A `for` loop becomes a
conditional branch and a jump. A `Drop` becomes a call. A borrow becomes... what?
If your research is about safety, the moment of lowering is where your invariant
must either be *preserved* or *checked*. This is where most safety bugs in real
compilers come from, and where your proof obligation will live.

---

## 2. Lexing and parsing (the parts you can skim)

**Lexing** turns `x = 42;` into `[ID(x), EQ, INT(42), SEMI]`. A finite automaton.
Boring, solved, and irrelevant to your research.

**Parsing** turns tokens into a tree according to a grammar. You should know two
families:

- **Recursive descent** — one function per grammar rule. What most hand-written
  compilers use. Easy to read, easy to extend.
- **LR / LALR** — table-driven, bottom-up. What `yacc`/`bison` generate. Powerful
  but painful to debug.

You will not do parser research. But you *will* need to add syntax to a language
(a new annotation, a new keyword), so know how to extend a recursive-descent
parser and how to attach source locations to nodes (you will need them for error
messages and for proofs that mention "this expression").

**AST** is just a tree of nodes: `Assign(name, Int(42))`. Name resolution and
type checking walk it. If your idea is a *type system*, this is where it hooks in.

---

## 3. Intermediate representations

An IR is a simplified language designed for analysis and transformation. Design
choices you must recognize:

- **Tree IR vs. linear IR.** Trees are close to syntax; linear IRs (sequences of
  instructions with jumps) are close to machines. Most optimizing compilers use
  a linear IR with basic blocks.
- **Basic block:** a maximal straight-line sequence with one entry and one exit.
  No jumps in the middle.
- **CFG (control-flow graph):** basic blocks as nodes, jumps as edges. *Every*
  analysis you will read about is defined over a CFG. Learn to draw one.
- **Three-address code:** `t1 = a + b; t2 = t1 * c`. At most one operation per
  instruction. Makes dataflow trivial to state.
- **Typed vs. untyped IR:** LLVM IR is typed; some IRs are not. If your research
  is about types, you care a lot about this.

> **For your paper:** know whether your idea needs types available in the IR. If
> it does, and the IR you target has erased them, you have a design problem
> before you have a research problem.

---

## 4. SSA: the one idea you must truly understand

**Static Single Assignment** means: every variable is assigned exactly once.
This sounds like a restriction; it is actually a superpower, because it turns
"what value does `x` have here?" into a lookup instead of a search.

### The problem it solves

```
x = 1
if (c) { x = 2 }
y = x        ← which x?
```

A naive analysis has to track "x might be 1 or 2." That is a *set* of facts, and
sets are where precision dies.

### The fix

Rename each assignment, and introduce a **φ (phi) node** at merge points:

```
x1 = 1
if (c) { x2 = 2 } else { x3 = x1 }
x4 = φ(x2, x3)     ← "whichever branch we came from"
y1 = x4
```

A φ node is a *merge point that asks which predecessor block we arrived from and
selects the corresponding version*. It is not a real machine instruction; it is
a bookkeeping device that disappears during register allocation (usually by
inserting copies on the incoming edges).

### Why it matters to you

- **Def-use chains become explicit.** Every use points at exactly one definition.
  Liveness, dead-code elimination, and constant propagation become trivial
  graph problems.
- **Aliasing analyses are stated on SSA.** "Does this pointer escape?" is much
  easier when every value has a unique definition site.
- **Your research almost certainly needs SSA.** Ownership, escape analysis, and
  budget accounting are all naturally expressed as facts about SSA values.

### The technical bits (enough to be literate)

- **Dominance:** block A dominates B if every path from entry to B goes through
  A. A definition in A is visible in B only if A dominates B.
- **Dominance frontier:** the set of blocks where a definition stops dominating —
  exactly where φ nodes must be placed. (Cytron et al., 1991 — the canonical
  algorithm.)
- **Pruned SSA:** only place φ nodes for variables that are actually live. What
  real compilers do.

You do not need to implement SSA construction to do research, but you must be
able to read "we place φ nodes at the iterated dominance frontier" without
panicking.

---

## 5. Dataflow analysis: the engine of everything

Almost every static analysis is a dataflow analysis. Learn the framework once and
you can read a hundred papers.

### The shape

1. **A lattice** of facts. Each program point gets a fact.
   - Example (liveness): the set of variables live at that point. Ordered by
     subset. "More variables" is *less precise* (safe over-approximation).
2. **Transfer functions** per instruction. "Given the fact before, what is the
   fact after?" Must be **monotone** with respect to the lattice order.
3. **A merge rule** at join points: union (may-analysis) or intersection
   (must-analysis).
4. **A fixpoint iteration:** start from ⊥ (or ⊤), apply transfer functions around
   the CFG until nothing changes.

The lattice guarantees termination: monotone functions on a finite-height
lattice reach a fixpoint in bounded steps. That is the Knaster–Tarski story in
one sentence.

### The canonical examples

| Analysis | Direction | Fact | Used for |
|---|---|---|---|
| Reaching definitions | forward | set of defs that may reach here | constant prop, SSA |
| Liveness | backward | set of vars used later | register allocation, dead code |
| Available expressions | forward | expressions already computed | CSE |
| Very busy expressions | backward | expressions needed later | hoisting |
| **Escape analysis** | forward | does this object escape? | stack allocation, your research |
| **Borrow liveness** | backward | which loans are still needed? | Rust's borrow checker |

That last row is the punchline: **Rust's borrow checker is a liveness analysis
over loans.** It is not magic. It is a dataflow problem with a cleverly designed
lattice. Chapter 05 unpacks this fully.

### Precision vs. cost

Every analysis trades precision for speed. A *may* analysis over-approximates
(says "maybe escapes" when it doesn't) — sound but imprecise. A *must* analysis
under-approximates — precise but unsound if used for safety. **Safety requires
over-approximation.** If your analysis ever says "definitely dead" and is wrong,
you have a use-after-free. Write that sentence on your wall.

---

## 6. Optimization passes you should recognize by name

- **Inlining** — replace a call with the body. Enables everything else, and
  destroys modularity of analysis (a classic research tension).
- **Constant propagation / folding** — replace known values.
- **Dead code elimination** — remove code with no observable effect.
- **Common subexpression elimination (CSE)** — reuse a computed value.
- **Loop-invariant code motion (LICM)** — hoist out of loops.
- **Scalar replacement of aggregates (SROA)** — split a struct into fields; the
  enabler for escape analysis.
- **Devirtualization** — resolve dynamic dispatch.

Notice: most optimizations *depend on aliasing information*. "Can I move this
load?" is "does anything write to that location?" That is why alias analysis is
the bottleneck of compiler performance, and why your memory-model research has
immediate practical relevance. **A more precise aliasing model is a faster
compiler.** Say that in your introduction; it is true and it sells.

---

## 7. Back end: where the invariants get cashed out

- **Instruction selection** — IR → machine ops (tree tiling).
- **Register allocation** — assign values to a finite register set; spills go to
  stack. Usually graph coloring or linear scan. **This is where φ nodes vanish.**
- **Instruction scheduling** — reorder for pipelines.
- **Peephole / post-RA optimization.**

The key research point: **the back end is where your safety invariant must
survive.** If your source language proves "no use-after-free," but register
allocation reorders a free before a use, you have a bug. Real verified compilers
(CompCert, CakeML) spend enormous effort proving the back end preserves
semantics. You probably will not verify a back end; you will *assume* the back
end is correct and say so explicitly. That is an acceptable, standard scoping
decision — as long as you state it.

---

## 8. Where memory management actually lives

Here is the map you should be able to draw. Each arrow is a decision point:

```
allocation site
   │  1. WHERE? stack / heap / region / pool
   ▼
lifetime determination
   │  2. WHEN does it die? static scope / refcount / GC / revoke / budget
   ▼
reclamation insertion
   │  3. WHO inserts the code? compiler / runtime / programmer
   ▼
alias & concurrency reasoning
   │  4. WHAT may access it concurrently? borrows / epochs / locks
   ▼
lowering to target
```

Your "life" pillars map onto these:

| Pillar | Decision point | Chapter |
|---|---|---|
| Inheritance | 1 + 2 (policy and lifetime propagate) | 04, 06 |
| Aging | 2 (when does it die — as a *hint* only) | 04 |
| Currency | 1 (how much may be allocated) | 07 |
| Overpowering | 2 + 4 (revocation + fence) | 06, 08 |

> **For your paper:** the strongest, most defensible position is to own
> decision point **2** and **4** together — *when does it die, and what makes
> that safe under concurrency*. That is where the open problems are, and where
> nobody has a clean language-level answer.

---

## 9. What to read for this chapter

- **Appel, *Modern Compiler Implementation in ML*** — the classic textbook;
  chapters on SSA, dataflow, and register allocation.
- **Cooper & Torczon, *Engineering a Compiler*** — more practical, excellent on
  dataflow and SSA.
- **The SSA Book** (Braun et al., *Simple and Efficient Construction of Static
  Single Assignment Form*) — free, short, and the definitive practical reference.
- **LLVM's Kaleidoscope tutorial** — build a tiny compiler end-to-end in a
  weekend. Do this; it converts theory into intuition faster than any book.

**Exercise:** take this program and convert it to SSA by hand, placing φ nodes:

```
a = 0
while (a < 10) {
  if (a % 2 == 0) { b = a } else { b = a * 2 }
  a = a + 1
}
print(b)
```

If you can do that and explain why each φ node is where it is, you understand
more SSA than most working programmers.

---

**Next:** `02-types.md` — judgments, linearity, lifetimes, and how to read the
symbol soup.
