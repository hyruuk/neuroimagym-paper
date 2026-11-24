# neuroimagym: the methods paper

The LaTeX source of the paper that presents [neuroimagym](https://github.com/tomov/fmri-gym) (the software is being renamed from fmri-gym): step-based environments as one medium for people and artificial agents in neuroimaging experiments, with their logging, multimodal integration and modular design. An arXiv preprint first, a journal after.

The same project lives in two places:

| Remote | Branch | Used for |
|---|---|---|
| `origin`: GitHub, `hyruuk/neuroimagym-paper` (private) | `main` | editing from a terminal or by an agent, review, history |
| `overleaf`: `https://git.overleaf.com/<project id>` | `master` (Overleaf has only this one) | reading the PDF, commenting and editing in the browser |

## Layout

| Path | What it is |
|---|---|
| `main.tex` | the document: preamble, title, authors, the sections in order, the bibliography |
| `sections/*.tex` | one file per section, numbered in reading order |
| `references.bib` | every reference the text cites |
| `figures/` | the figures, and `figures/README.md` for where each one comes from |
| `arxiv.sty` | the arXiv preprint style (MIT licence, `arxiv-sty-LICENSE.txt`) |
| `notes/` | working notes, decisions to take, the plan; not part of the paper |

## Building the PDF

- On Overleaf: open the project; `main.tex` is the main document.
- Locally, with a TeX distribution: `latexmk -pdf main.tex`; or with [Tectonic](https://tectonic-typesetting.github.io): `tectonic main.tex`.
- On GitHub: a workflow that builds `main.pdf` on every push is ready in `notes/pdf-workflow.yml`; it goes to `.github/workflows/` once the GitHub token may push workflows (`notes/open-decisions.md`).

## Keeping GitHub and Overleaf in sync

Before and after a working session:

```bash
git fetch origin overleaf
git merge origin/main
git merge overleaf/master
git push origin main
git push overleaf main:master
```

- Never force-push and never rewrite history on either remote (Overleaf refuses it anyway).
- Feature branches live on GitHub only; Overleaf sees `main`.
- A conflict in `.tex` content is resolved by a person, not by an agent: an agent stops and asks.
- The Overleaf remote authenticates with the username `git` and an Overleaf Git token, entered once at the prompt so that the credential helper keeps it; the token never goes into a URL or a file.

## Writing

The rules for the source (one sentence per line, citations, numbers, vocabulary) are in [`CLAUDE.md`](CLAUDE.md); they hold for people too.
