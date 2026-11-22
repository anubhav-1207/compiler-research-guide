# Chapter 06 — Capabilities and Revocation

Your "overpowering" pillar — one object kicks another out of memory — is called
**revocation**, and it is one of the most rigorously studied mechanisms in
systems research. seL4 has a *machine-checked proof* of it. This chapter gives you
that literature so you can build on it instead of reinventing it badly.

---

## 1. What a capability is

A **capability** is an **unforgeable token that designates a resource and conveys
specific rights to it.**

Three properties define it:

1. **Unforgeable.** You cannot manufacture one; you can only receive one.
2. **Designates.** It names a specific object (not an address you computed).
3. **Conveys rights.** Read, write, execute, grant, revoke — a bitmask.

This is a **capability-based** system, as opposed to an **access-control-list**
system. The distinction:

- ACL: "the object keeps a list of who may access it." Checking is a lookup on
  every access, against a global policy.
- Capability: "the accessor holds a token." Checking is possession. **Authority
  is local**, which is why capability systems compose and scale.

Memory safety falls out: **if you can only name memory you hold a capability
for, you cannot forge a pointer into memory you do not own.** That is a stronger
guarantee than "no use-after-free" — it is "no unauthorized access at all."

---

## 2. The object-capability model and attenuation

In the **object-capability (ocap)** model, capabilities are the *only* way to
interact with anything. There are no ambient authorities (no global variables
that grant power).

**Attenuation** is the key operation: derive a *weaker* capability from a
stronger one. You can drop rights, never add them.

```
cap_rw  ──attenuate(read)──▶  cap_r  ≤  cap_rw
```

Attenuation is **monotone downward**. This is what makes reasoning tractable: the
authority graph is a DAG that only shrinks along edges.

**Why this matters to you:** this is a *precise* notion of "inheritance" — a
derived capability **inherits** its parent's authority, minus what was
attenuated. Your "inheritance of memory" pillar is, formally, **capability
derivation**. That is a much sharper concept than OOP inheritance, and it has a
proof theory.

---

## 3. The capability derivation tree (seL4)

seL4 is a microkernel with a full formal verification of functional correctness.
Its memory model is the gold standard for revocation. Here is the mechanism.

**The CDT (capability derivation tree).** Every capability has a parent. When you
derive (copy, attenuate, or retype) a capability, the new one becomes a **child**
in the CDT. The structure is a tree rooted at the original capability.

```
        [root: memory region]
              │
      ┌───────┴───────┐
   [cap A: rw]     [cap B: r]
      │
   [cap C: r]
```

**Revocation.** To revoke a capability, you **delete its entire subtree**:

- Recursively remove all descendants.
- Any object whose last capability is removed is **destroyed** (its memory
  returned to the pool).

That is "overpowering," precisely: **a parent can destroy everything derived from
it, transitively, and nothing else.**

**Three properties that make it sound:**

1. **Hierarchical.** Authority only flows down. You can only revoke what descends
   from a capability you hold. You cannot revoke a sibling's subtree.
2. **Monotone.** Revocation only removes authority; it never grants. So the
   authority set shrinks monotonically, which makes invariants inductive.
3. **Transitive.** The whole subtree goes, so there is no "orphan" that keeps a
   dangling reference alive.

**The fence (the hard part).** What if another core is *currently* using a
capability you are revoking? seL4 handles this with:

- a **global lock** on the CDT during revocation, and
- **IPI-based TLB shootdown** to force other cores to drop stale translations.

This is the **quiescence problem**, and it is unavoidable in any concurrent
revocation scheme. You cannot revoke safely until you know no one is mid-access.

> **For your paper:** this is the crux. A language-level revocation must answer:
> *what is the fence?* Options: a global epoch (cheap, coarse), per-object
> refcounts (fine, expensive), or a static proof that no concurrent access is
> possible (free, restrictive). **Choosing and proving the fence is a paper in
> itself.**

---

## 4. CHERI: capabilities in hardware

CHERI (Capability Hardware Enhanced RISC Instructions) puts capabilities in the
ISA. Each pointer becomes a **fat pointer**: base, bounds, permissions, and a tag.

- **Bounds checking in hardware.** An out-of-bounds access traps.
- **Provenance.** A pointer can only be derived from another valid pointer, by
  bounded arithmetic. You cannot fabricate one from an integer.
- **Compartmentalization.** You can hand a library a capability scoped to a small
  buffer, and it *physically cannot* touch anything else.

Cost: roughly 1 bit of tag per 16 bytes of memory (tagged memory), wider pointers
(128-bit), and some performance overhead (typically low single-digit percent,
sometimes a win due to reduced bounds checks in software).

**Why it matters to you:** CHERI is the existence proof that **capability
semantics can be a hardware reality, not just a formal model.** If your language
targets CHERI, your revocation story becomes *concrete and fast* rather than
requiring a kernel call. That is a strong, novel, and very publishable angle:
**"a language-level ownership/revocation model that compiles to CHERI
capabilities."**

Also note: **CHERI's provenance model is essentially Rust's Stacked Borrows in
hardware.** The two communities are converging, and there is real research space
at that intersection.

---

## 5. Revocation semantics: the formal statement

If you formalize revocation, here is the shape your definitions should take.

**State.** A heap `H`, an authority graph `G` (a forest), and a set of live
capabilities.

**Invariant (Authority Soundness).** Every capability in `G` has a parent in `G`
or is a root. Every object referenced by a capability is live.

**Revocation rule:**

```
    cap c ∈ G     G' = G \ subtree(c)
    ────────────────────────────────────
    ⟨revoke(c), H, G⟩ ⟶ ⟨H', G'⟩
```

where `H'` frees every object whose last capability was in `subtree(c)`.

**The theorem you must prove:**

> **Revocation Safety.** If `⟨H, G⟩` satisfies the invariant and
> `⟨revoke(c), H, G⟩ ⟶* ⟨H', G'⟩`, then `⟨H', G'⟩` satisfies the invariant, and
> no capability in `G'` references a freed object.

**The hard lemma** is the one connecting revocation to *outstanding references*:
you must show that any reference held by running code is either (a) inside the
revoked subtree (so it dies too), or (b) not derived from `c` (so it is
unaffected). That is the **frame property**, and separation logic gives it to you
almost for free. Use separation logic.

---

## 6. Mapping to your pillars

| Your pillar | Capability concept | Formal object |
|---|---|---|
| Inheritance | derivation + attenuation | CDT edges; monotone downward |
| Overpowering | revocation of a subtree | `G \ subtree(c)` |
| Aging (sound version) | capability expiry with a liveness proof | not native; needs epochs |
| Currency | quota on capability creation | not native; needs a resource logic |

Note the two that capabilities *do not* give you: aging and currency. Those need
separate machinery (chapters 04, 07, 08). **This is the honest gap in your
model**: capabilities give you inheritance and overpowering beautifully, and
nothing for the other two. Say so in your paper; it is a strength to be precise
about scope.

---

## 7. Reading

- **Levy, *Capability-Based Computer Systems* (1984)** — the historical foundation.
- **Miller, *Robust Composition* (PhD thesis, 2006)** — the object-capability
  model, readable and influential.
- **Klein et al., *seL4: Formal Verification of an OS Kernel* (SOSP 2009)** — the
  headline paper. Read the revocation section carefully.
- **The seL4 reference manual, chapter on capabilities and CDT** — the precise
  semantics, surprisingly readable.
- **Woodruff et al., *The CHERI Capability Model* (2014)** and **Davis et al.,
  *CHERI C/C++* (2019)** — the hardware story.
- **Watson et al., *Capability Hardware Enhanced RISC Instructions: CHERI
  Instruction-Set Architecture*** — the spec, for reference.

---

**Next:** `07-resources.md` — turning "currency" into a type, with linear logic
and amortized analysis.
