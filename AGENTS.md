# AGENTS.md

Documentation-only repository. There is **no** build, test, lint, or CI — do not
hunt for `package.json`, a `Makefile`, or a toolchain. The deliverable is
Markdown prose.

## What this is

A 15-chapter self-study curriculum (`00-*.md` … `14-*.md`, plus `README.md`) for
researching memory safety versus low-level control in compilers. Public repo,
branch `main`, licensed CC BY 4.0.

## Keep these in sync (the easy-to-miss part)

- `README.md` → the fenced tree under **"The directory"** is a *manual index*.
  Adding, renaming, or removing a chapter requires updating that tree, the
  chapter ranges in **"How to use this guide"**, and the **"Suggested cadence"**
  table.
- Chapters `00`–`12` each end with a `**Next:** \`NN-slug.md\` — one-liner`
  footer forming a chain. Inserting a chapter means re-pointing the previous
  chapter's footer. `13` and `14` intentionally have no footer.
- Chapters cross-reference each other as "chapter NN §N". Renaming a referenced
  heading silently breaks those pointers.

## Content invariants

- **Never invent citations.** The guide names ~40 specific papers, venues, and
  years. Add a reference only if it is verifiable; otherwise extend
  `13-bibliography.md` from a real source. Fabricated citations are the worst
  failure mode here.
- **No implementations.** This is prose with tiny illustrative snippets only. Do
  not add code, build steps, or "let's build the compiler" sections.
- **Preserve the critical tone.** The guide deliberately refuses to overclaim
  (it calls the "life" idea roughly 60% rediscovery). Do not soften it into
  marketing copy.
- The four "life" pillars map to fixed chapters and to the table in
  `04-memory.md` §7: inheritance → regions/ownership subtyping, aging →
  promotion only, currency → linear budgets, overpowering → capability
  revocation. Edits must keep that mapping consistent across `04`, `06`, `07`,
  and `12`.

## Style

- Chapter heading: `# Chapter NN — Title`.
- `> **For your paper:**` blockquote callouts mark research-relevant takeaways.
- Bibliography priority marker: `★` means read first.
- Glossary: one line per term, alphabetical, no paragraphs.
- Chapters run ~1000–2000 words. Match surrounding density; no padding.

## Git

- The repo is **public** and CC BY 4.0. Do not commit personal notes, secrets,
  or session artifacts.
- Commit identity is repo-local (`Anubhav G.` + GitHub noreply address), not
  global.
- Do not copy the user-level "Royal Protocol" from `~/.config/opencode/AGENTS.md`
  into repo files — it is personal guidance and this repo is public.
