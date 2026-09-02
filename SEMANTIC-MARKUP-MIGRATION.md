# Semantic-markup migration rationale

Running record of what changed in `LCS.arXiv.V2.tex`/`M-atlas.tex` to
adopt the [LaTeX-Semantic-Markup](https://github.com/shmuelmetz/LaTeX-Semantic-Markup)
package, and why. Kept alongside the papers themselves rather than
only in commit messages, since the migration touches thousands of
call sites across several separate passes and the reasoning for each
pass is worth having in one place, not scattered across commit
history.

## Why

The papers currently define their own presentation-hardcoded macros
(`\catname`, `\cat`, `\compose`, `\into`, ...) — the display choice
(script vs. bold upright, which arrow glyph, ...) is baked into each
macro definition itself. `LaTeX-Semantic-Markup`'s whole design goal,
in the author's own words: *"if you submit a paper to different
journals, you shouldn't have to change your markup to accommodate
different house styles, other than changing setup."* Migrating onto
it means the document body becomes journal-portable — a house-style
change becomes one `\usepackage[style=...]` line, never a body edit.

## Package version adopted

**0.3.0** (2026-09-01), *after* a consistency-check pass and fix —
not the version as originally built. Before any migration edit, every
`\catname`/`\cat` call site in both papers (3413 calls, 17 distinct
arguments) was checked against the package's real
`\tl_if_single:nTF`-based auto-detection via an actual `pdflatex` run,
not assumed correct. Three arguments — `C^2`, `E^i` (1 site each),
and `XY` (44 sites, a product-category symbol) — were called via
`\catname` (the papers' "arbitrary symbol" choice) despite not being
single tokens, meaning the *original* heuristic would have silently
rendered all 46 of those real sites in bold upright instead of
script the moment they were mechanically renamed. Fixed in the
package itself (`\catName` now also treats a `^`/`_`-decorated
argument, or an all-uppercase-letters argument, as "arbitrary" ->
script) before this migration started, verified against all 17 real
arguments through the actual built macro, and pinned with a permanent
regression test (`test-catname-decoration.tex` in that repo). See
that repo's `semantic-markup.dtx`, the "Tightened, 2026-09-01"
correction under "cat vs. catname (resolved)", for the full writeup.

## Macro mapping

Built by reading `semantic-markup.dtx` in full and grepping the real
call sites in both papers — not assumed from the package's public API
alone. Counts below are exact: a plain substring `grep` first gave
inflated numbers for `\compose`/`\into`/`\onto` (matching
`\composeh`/`\composet` as a prefix of `\compose`, and, worse for
`\into`/`\onto`, also matching the plain English words "into"/"onto"
elsewhere on a matching line) — recounted with a backslash-anchored
regex requiring the macro name not be followed by another letter, so
these are the real call-site counts, verified against spot-checked
context.

| Legacy macro (papers) | New macro (package) | Call sites (LCS / M-atlas) | Kind |
|---|---|---|---|
| `\catname{X}`, `\cat{X}` | `\catName{X}` | 1737+123 / 1660+16 | Mechanical rename |
| `\catseqname{X}` | `\catSeqName{X}` | 477 / 37 | Mechanical rename |
| `\compose` (bare or `[label]`) | `\morphCompose` | 575 / 553 | Mechanical rename |
| `\composeh` | `\morphComposeHead` | 10 / 1 | Mechanical rename |
| `\composet` | `\morphComposeTail` | 10 / 1 | Mechanical rename |
| `\into` | `\morphMono` | 6 / 14 | Mechanical rename (second pass, see below) |
| `\onto` | `\morphEpi` | 38 / 15 | Mechanical rename |
| `\funcname{X}` | `\morphName{X}` | 2151 / 2114 | Mechanical rename |
| `\funcseqname{X}` | `\morphSeqName{X}` | 718 / 426 | Mechanical rename |

**Correction, 2026-09-01:** this table previously said `\morphName`
had no legacy macro to rename from, citing only the papers' disabled,
pre-`expl3`-era `% \newcommand \funcname [1] {\mathit{#1}}`. That was
wrong — the user caught it directly: *"the commented out definition
dates from before I was using expl3; look for the second
definition."* A second, active, genuinely `expl3`-based
`\funcname`/`\funcseqname` definition exists further down in both
papers (identical in `LCS.arXiv.V2.tex`/`M-atlas.tex`), used at the
4265 + 1144 real call sites in the table above — not a disabled
placeholder. It auto-detects the display style itself, the same way
`\catname`/`\cat` effectively split on name length: a single-token
argument renders italic (bold italic for the Seq form); anything
longer renders upright roman (bold roman). `\morphName`/
`\morphSeqName` were corrected to match — see that repo's own commit
history for the fix (found the same `f^2`-style decorated-symbol edge
case `\catName`'s own tightening had already needed, one real call
site, fixed by reusing the same heuristic rather than special-casing
it again). **So every macro in this table is now a mechanical
rename** — there is no separate "manual morphism-tagging" phase after
all.

## `\into` — resolved as a real semantic bug in the package, not a style choice

Every other macro in the table above was checked for a rendering
match before renaming (`\catName`/`\morphName` via the consistency-
check tool; `\compose`/`\composeh`/`\composet` by direct comparison of
their default-style glyphs, `\circ`/`\odot`/`\cdot`, against the
papers' own — identical). `\into`/`\onto` needed the same check since
they wrap external symbols (`\mon`/`\epi`, both from Xy-pic) rather
than defining their own glyph, and the check found a real mismatch:
rendered side by side, `\onto`'s `\epi` is visually identical to
`\morphEpi`'s `\twoheadrightarrow` (safe, renamed in the first pass),
but `\into`'s `\mon` is Xy-pic's own thin, small tail-arrow — **not**
the same glyph as `\morphMono`'s original `\hookrightarrow` default.
First pass left `\into` unrenamed pending a decision, rather than
either guess.

**Resolution**: re-reading this paper's own Part II ("Conventions")
section, quoted verbatim, settled it as a real semantic bug in the
package, not a style preference to arbitrate: *"One with a hook
($A \hookrightarrow B$) represents an **inclusion map**. One with a
tail ($A \rightarrowtail B$) represents a **monomorphism**."* These
are two genuinely distinct mathematical notions (every inclusion is a
monomorphism, but not every monomorphism is an inclusion) that this
paper deliberately gives two different dedicated symbols — and
`semantic-markup`'s original `\morphMono` default was using the
paper's own **inclusion** symbol, not its monomorphism one. Fixed in
the package itself (`\lsm_morph_mono:` now renders as `\rightarrowtail`,
an `amssymb` symbol confirmed visually near-identical to `\mon` — see
`LaTeX-Semantic-Markup` commit `549b263`, v0.6.0), not by changing
this paper. `\into` then renamed to `\morphMono` in both papers (20
combined call sites) in a second pass, same day.

Also removed in this same pass, unrelated to `\into` itself: a stale,
auto-generated block of ~80 `% Package: ...` version-snapshot comments
at the top of each file (a leftover compile-log capture from years
ago, already including `thmtools`, which this migration's first pass
had separately determined was unused and removed as actual code) — no
longer accurate or useful, and not referenced by anything. Also
corrected two places where the mechanical rename had (harmlessly, but
confusingly) rewritten the word "`\into`" inside prose explaining the
rename itself and inside a historical per-version changelog comment,
into "`\morphMono`" — those now correctly describe what was actually
done at the time, rather than retroactively renaming history.

## Changes

Entries added as each pass actually happens, newest first. Each entry
names the macro(s), the papers touched, the site count, and anything
non-mechanical that came up during the pass (an argument that didn't
fit the expected pattern, a rendering check, etc.).

### 2026-09-01 -- first pass: everything except `\into`

Both papers, all macros in the table above except `\into` (see
above). Concretely, per paper: preamble gets `\usepackage{semantic-
markup}`; the now-superseded legacy definitions (`\cat`, `\catname`,
`\catseqname`, `\compose`, `\composeh`, `\composet`, `\onto`,
`\funcname`/`\funcname:n`, `\funcseqname`/`\funcseqname:n`) are
removed, each replaced with a one-line comment pointing here;
`\into`'s definition is kept, with a comment explaining why it wasn't
touched.

Three real problems found and fixed along the way, each verified
empirically before and after, not assumed:

1. **Package/paper compatibility bug, fixed in the package.** Loading
   `semantic-markup` alongside the papers' own `stix2` (a
   comprehensive OpenType math font package) hit LaTeX's hard
   "Too many symbol fonts declared" limit — `stix2` already defines
   `\mathscr` and `\twoheadrightarrow`, the two things the package
   needs from `mathrsfs`/`amssymb`, and stacking both symbol-font sets
   overflowed the 16-slot table. Fixed in `LaTeX-Semantic-Markup`
   itself (both `RequirePackage`s now guarded with `\ifdefined`); see
   that repo's commit history, v0.5.0.
2. **Pre-existing, unrelated compile bug, fixed in the papers.**
   Independent of this migration: `amsthm` + `thmtools` + the
   installed `cleveref` v0.21.4 reproducibly break
   `\newtheorem{corollary}[theorem]{Corollary}` with a fatal
   `Command \c@corollary already defined` error (isolated to a 5-line
   minimal reproduction). Confirmed present on the *original,
   unmodified* papers via `git stash` before touching anything.
   `thmtools` is loaded in both papers but nothing from it
   (`\declaretheorem`, `\listoftheorems`, a custom style) is ever
   actually used (confirmed by grep) — removed from both preambles,
   zero functional loss, both papers now compile cleanly end to end
   (verified: two full `pdflatex` passes each, zero "Undefined control
   sequence" errors, real content rendering correctly on inspection).
3. **Rename-script gaps, fixed by hand.** The mechanical rename missed
   two real TeX idioms it didn't account for: (a) a macro invoked as a
   bare subscript target (`\Hom_\catName{C}`) parses differently for
   an `xparse`-defined command than for the papers' own plain
   `\newcommand`-defined ones — 6 such sites (all `\catName`, all
   `\Hom_`/`\Set_`/`\Top_` constructions) needed explicit braces added
   (`\Hom_{\catName{C}}`); (b) a single-letter argument given without
   braces (`\catname C`, valid TeX for a one-character argument) isn't
   matched by a brace-requiring rename regex — 4 such sites (`\catname
   C`, 2 per paper) were converted to the braced form
   (`\catName{C}`) by hand. A full sweep of both files afterward found
   no further occurrences of any renamed macro outside a comment.

Verified: both papers compile cleanly, two `pdflatex` passes each
(needed for cross-references), zero errors, zero "Undefined control
sequence", spot-checked several rendered pages (including dense
category-theory notation) for correct typography. Regenerated
`.pdf`s are part of this same change -- not committed independently.
