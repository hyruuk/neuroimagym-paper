# Progress log

## 2026-11-24 night (Claude, alone; nothing opened, nothing pushed to the software's remotes)

- Paper repository created (private), with the skeleton: `main.tex`, one file per section, `references.bib` (21 entries; 13 checked against their pages, 8 marked UNVERIFIED), the PDF workflow, these notes.
- Framework section drafted in full (`sections/02-framework.tex`): environments, engine time and human time, the three stepping regimes with a TikZ figure, decision points and actions, people and agents on one environment, what a run records, and the vocabulary table. The other sections are outlines.
- Overleaf waits for you: `open-decisions.md`, section 1.
- The first commit on GitHub (c084834) is labelled "Framework…" but holds the whole skeleton: rebuilding the local commits went wrong after the workflow had to be taken out (the `gh` token may not push workflows), and the result was pushed before I saw it. Left as is, since pushed history is never rewritten here.
- Installed Tectonic (`brew install tectonic`, 20 MB; remove with `brew uninstall tectonic`) to compile the paper locally: `tectonic -X compile main.tex`. It builds with no undefined reference or citation; Overleaf (pdflatex) will be the final check.
