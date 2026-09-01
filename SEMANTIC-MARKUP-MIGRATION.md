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
| `\into` | `\morphMono` | 6 / 14 | Mechanical rename |
| `\onto` | `\morphEpi` | 38 / 15 | Mechanical rename |
| *(none — see below)* | `\morphName{X}` | 0 | **Not mechanical** |

**`\morphName` has no legacy macro to rename from.** The papers'
own `\funcname`/`\funcseqname` are commented out in the source
(`% \newcommand \funcname [1] {\mathit{#1}}`) — morphisms are
currently just bare math variables (`$f$`, `$g$`), never wrapped in
any macro. `\funcname`/`\funcseqname` are exactly what `\morphName`/
`\morphSeqName` formalize semantically (a disabled placeholder for
the same concept the new package now names) — but since the papers'
own macro was never turned on, there's no live call site to
mechanically rename *from*. Migrating morphism names means manually
finding and wrapping each bare occurrence — a semantic-judgment task
done sentence by sentence, not a search-and-replace — and is treated
as its own separate, later phase, not blocking the mechanical renames
above.

## Changes

Entries added as each pass actually happens, newest first. Each entry
names the macro(s), the papers touched, the site count, and anything
non-mechanical that came up during the pass (an argument that didn't
fit the expected pattern, a rendering check, etc.).

*(No migration edits made yet — this file was set up in advance of
the first pass.)*
