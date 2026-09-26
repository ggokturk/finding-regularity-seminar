# Finding Regularity: High-Performance Computing for Graph Algorithms

Research seminar by Gökhan Göktürk.

## Files

- [seminar.pdf](seminar.pdf): compiled slides.
- [seminar.tex](seminar.tex): editable LaTeX Beamer source, including speaker cues.
- [seminar-narrative.md](seminar-narrative.md): spoken narrative and timing plan.
- [seminar-editorial-notes.md](seminar-editorial-notes.md): editorial rationale and rehearsal notes.
- [papers/README.md](papers/README.md): reference-paper provenance. The reference PDFs and manuscript sources are local preparation material and are not included.

The deck contains 26 main slides, followed by a divider and 14 backup slides. All slides are static, giving 41 PDF pages in total. Prepared material totals 30:50; reserve 35 minutes for delivery with pauses and brief interruptions, with 40 minutes as the ceiling in the one-hour seminar and interview session. This leaves roughly 20–25 minutes for discussion and interview questions.

## Build

Install a TeX distribution with `latexmk`, pdfLaTeX, the Metropolis Beamer theme, Source Sans Pro, Source Code Pro, and TikZ. The slide figures are embedded in the LaTeX source; no external image assets or reference PDFs are required.

```sh
latexmk -pdf -interaction=nonstopmode -halt-on-error -outdir=.build/seminar seminar.tex
cp .build/seminar/seminar.pdf seminar.pdf
```

To build a separate PDF with speaker cues on the right-hand page:

```sh
pdflatex -interaction=nonstopmode -halt-on-error -output-directory=.build/seminar -jobname=seminar-notes '\def\seminarnotes{1}\input{seminar.tex}'
pdflatex -interaction=nonstopmode -halt-on-error -output-directory=.build/seminar -jobname=seminar-notes '\def\seminarnotes{1}\input{seminar.tex}'
```
