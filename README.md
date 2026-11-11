# Local-Coordinate-Spaces

Mathematical papers on local coordinate spaces, manifolds, fiber
bundles, atlases, and morphisms between them.

Author: Seymour J. Metz (H: Shmuel שמואל בן לייביש ולאה)
(<https://mason.gmu.edu/~smetz3>)

## Papers

### Local Coordinate Spaces: a proposed unification of manifolds with fiber bundles, and associated machinery

- arXiv: [1801.05775](https://arxiv.org/abs/1801.05775)
- PDF: <https://arxiv.org/pdf/1801.05775>

### Towards a taxonomy of atlases and of morphisms between them

- arXiv: [1906.11690](https://arxiv.org/abs/1906.11690)
- PDF: <https://arxiv.org/pdf/1906.11690>

## Building

The papers are typeset with pdfLaTeX and BibTeX. Both load
`lcs-common.sty`, their shared preamble, which in turn loads the
`shmuelsemtex` package from
[LaTeX-Semantic-Markup](https://github.com/shmuelmetz/LaTeX-Semantic-Markup).

Both repositories sit side by side and share one directory for `.sty`
files, `../texmf/tex/latex` (STYDIR):

```
texmf/tex/latex/          shmuelsemtex.sty, lcs-common.sty (copied by make)
LaTeX-Semantic-Markup/    package source
Local-Coordinate-Spaces/  this repository
```

```sh
make -C ../LaTeX-Semantic-Markup install   # shmuelsemtex.sty -> STYDIR
make                                       # lcs-common.sty -> STYDIR, then both papers
```

## Repository structure

```
LCS.arXiv.V2.tex / .pdf              -- "Local Coordinate Spaces" paper
M-atlas.tex / .pdf                    -- "Towards a taxonomy of atlases..." paper
lcs-common.sty                        -- preamble shared by both papers
GNUmakefile                           -- builds the papers (see Building)
Local.Coordinate.Space.arXiv.abstract -- abstract for the LCS paper
M-atlas.abstract                      -- abstract for the M-atlas paper
SEMANTIC-MARKUP-MIGRATION.md          -- rationale for adopting LaTeX-Semantic-Markup
```

## Public identifiers

| Service | Identifier |
|---------|-----------|
| ORCID   | [0000-0002-6358-8257](https://orcid.org/0000-0002-6358-8257) |
| arXiv   | [metz_s_1](http://arxiv.org/a/metz_s_1) |

## License

MIT License. See [LICENSE](LICENSE).
