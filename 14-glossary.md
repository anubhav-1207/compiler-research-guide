# Chapter 14 — Glossary

One line each. Alphabetical. Keep this open while reading papers.

**AARA (Amortized Resource Analysis)** — type-based analysis that assigns
potential functions to data to prove asymptotic resource bounds.

**Affine type** — a value usable *at most* once (Rust's ownership). Weaker than
linear.

**Aliasing** — two references to the same location. The source of most memory
bugs, and the reason alias analysis is the bottleneck of optimization.

**Attenuation** — deriving a weaker capability from a stronger one (dropping
rights). Monotone downward.

**Basic block** — straight-line code with one entry and one exit; the node of a
CFG.

**Borrow** — a temporary, scoped permission to access a value without taking
ownership.

**Borrow checker** — the dataflow analysis that enforces aliasing-XOR-mutability
and loan liveness.

**Budget (allocation)** — a static upper bound on bytes or allocation count,
checked at compile or admission time.

**Capability** — an unforgeable token designating a resource and conveying
rights.

**CDT (capability derivation tree)** — the parent/child tree of derived
capabilities; revocation deletes a subtree.

**CFG (control-flow graph)** — basic blocks as nodes, jumps as edges.

**Copy-on-write (COW)** — share memory until a write, then duplicate.

**CSL (concurrent separation logic)** — separation logic extended to threads; the
proof technique for data-race freedom.

**Data race** — two threads accessing the same location, at least one writing,
without synchronization. UB in C/C++/Rust.

**Dataflow analysis** — a lattice-based fixpoint computation over a CFG; the
engine of static analysis.

**Dominance** — block A dominates B if every path to B goes through A.

**Dominance frontier** — where a definition stops dominating; where φ nodes go.

**Drop** — deterministic destruction at a statically-known point (Rust).

**EBR (epoch-based reclamation)** — free retired memory once all threads have
advanced past its epoch.

**Effect system** — types annotated with what a computation does (allocates,
throws, reads, writes).

**Escape analysis** — proving an object does not outlive its allocating function,
enabling stack allocation or scalar replacement.

**Fat pointer** — a pointer with extra metadata (bounds, provenance, generation).

**Frame rule** — separation logic's rule that a proof composes with any disjoint
resource; the basis of local reasoning.

**Generation (GC)** — an age class; survivors are promoted to older generations.

**Generational hypothesis** — most objects die young. The empirical basis of
generational GC.

**Generational reference** — a pointer carrying the generation it was created at;
dereference checks the counter, catching use-after-free dynamically.

**Grace period** — the interval after which all pre-existing readers have
finished (RCU).

**Hazard pointer** — a per-thread published pointer that a reclaimer must not
free.

**Heap typing (store typing, Σ)** — the types of heap locations; required for
preservation in languages with state.

**IR (intermediate representation)** — the compiler's internal language between
source and machine code.

**Iris** — a Coq framework for concurrent separation logic with step-indexing.

**Judgment** — a statement of the form "under these assumptions, this holds";
written `Γ ⊢ e : τ`.

**Lattice** — a partially ordered set with joins/meets; the domain of a dataflow
analysis.

**Linear type** — a value usable *exactly* once. Stronger than affine.

**Linearizability** — every concurrent operation appears to take effect atomically
between invocation and response.

**Loan** — the permission created by a borrow; the unit of borrow checking.

**Logical relation** — a type-indexed relation on values used to prove semantic
soundness; the technique that scales where syntactic proofs fail.

**Liveness (analysis)** — which variables are used later at a program point.

**Memory model** — the specification of which values a read may observe under
concurrency (SC, TSO, release/acquire, C++).

**Monotone framework** — a dataflow analysis whose transfer functions are
monotone; guarantees a fixpoint exists and iteration terminates.

**Move semantics** — transferring ownership without copying; the source becomes
invalid.

**NLL (non-lexical lifetimes)** — loan regions computed from actual liveness
rather than lexical scope.

**Operational semantics** — a definition of program execution as state
transitions.

**Ownership type** — a type system where each object is owned by exactly one
other object or scope.

**Perceus** — precise reference counting with reuse analysis; the cost-model
exemplar.

**Phi (φ) node** — an SSA merge that selects a value based on the predecessor
block.

**Polonius** — the Datalog formulation of borrow checking; the extensible
starting point.

**Preservation** — stepping a well-typed program yields a well-typed program.

**Progress** — a well-typed program is a value or can step (never stuck).

**Provenance** — the authority a pointer carries from its creation; accessing
outside it is UB even if the address is valid.

**Quiescence** — a state where no thread can be mid-access; the precondition for
safe reclamation or revocation.

**RCU (read-copy-update)** — readers never block; writers wait a grace period
before freeing the old version.

**Refinement type** — a type with a predicate: `{v : Int | v > 0}`.

**Region** — a named allocation area freed wholesale; the basis of arena
allocation.

**Revocation** — removing authority (and usually memory) from a capability and
its descendants, transitively.

**Separation logic** — a program logic where `∗` asserts disjoint heaps; the
standard tool for aliasing.

**Small-step semantics** — one evaluation step at a time; exposes intermediate
states, where bugs live.

**SSA (static single assignment)** — every variable assigned exactly once; makes
def-use chains explicit.

**Stacked Borrows** — Rust's aliasing model: per-location stacks of pointer tags;
using a popped tag is UB.

**Stuck state** — a configuration that is not a value and cannot step. Every
safety violation is one.

**Subtyping** — `τ₁ ≤ τ₂`: a `τ₁` is acceptable where a `τ₂` is expected.

**Tightness** — proven bound divided by actual usage; the metric that makes a
budget result meaningful.

**Tree Borrows** — the successor to Stacked Borrows; a tree instead of a stack,
more permissive.

**Type soundness** — well-typed programs do not go wrong; usually progress +
preservation, or a logical relation.

**Variance** — how subtyping of a parameter affects subtyping of the constructed
type (covariant, contravariant, invariant).

**WCET (worst-case execution time)** — a proven upper bound on execution time;
the temporal analogue of an allocation budget.
