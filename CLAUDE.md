# NeuroimaGym paper

The methods paper for NeuroimaGym (the software `neuroimagym`, formerly fmri-gym). LaTeX only, and the whole paper is one file, `main.tex`, at the root: no `\input` of sections.

## The source

- One sentence per line in `.tex` files, so a diff, a merge and an Overleaf comment each touch one sentence.
- `main.tex` must compile on Overleaf with pdflatex and latexmk: packages from TeX Live only, no shell-escape, no local-only files.
- Every citation key exists in `references.bib`. An entry is copied from a page that shows it (arXiv, the publisher, PubMed); an entry that could not be checked carries a `% UNVERIFIED` comment and is listed in `notes/open-decisions.md`.
- Every number in the text comes from a measurement, a log or the code, and says which. Until it does, it is `\TODO{…}`.
- `\TODO{…}` marks what is missing; the `outline` environment holds the planned content of a section that is not drafted yet, shown in grey in the PDF; `\status{…}` says, in an outline item, how much of it the software does today, as found in its code, and goes once the item is drafted.
- The software is NeuroimaGym in prose; `neuroimagym` only where the text names a command or the package.
- The vocabulary is the one defined in the Framework section (Section 2 of `main.tex`, the vocabulary table). The software's documentation (`docs/overview/concepts.md` in the neuroimagym repository) uses the same words; a change to one is a change to both.
- Describe what the software does today. What is planned is written as planned, with its status (a prototype, a design), never as a feature.

## Git

- `origin` is GitHub (`main`), `overleaf` is Overleaf (`master`). The sync routine is in `README.md`: fetch both, merge both into `main`, push both.
- Never force-push, rebase pushed commits, or rewrite history on either remote.
- Feature branches stay on GitHub; only `main` goes to Overleaf.
- Before resolving a conflict in `.tex` content, stop and ask the user.
- GitHub pushes go through the repository's own credential setting (`gh auth git-credential`); the Overleaf token lives in the credential helper only.

## Notes

`notes/` holds working notes (the plan, the feasibility note of the classical-task backend, open decisions). They are not part of the paper and are never `\input`.
