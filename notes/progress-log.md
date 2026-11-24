# Progress log

## 2026-11-24 night (Claude, alone; nothing opened, nothing pushed to the software's remotes)

- Paper repository created (private), with the skeleton: `main.tex`, one file per section, `references.bib` (21 entries; 13 checked against their pages, 8 marked UNVERIFIED), the PDF workflow, these notes.
- Framework section drafted in full (`sections/02-framework.tex`): environments, engine time and human time, the three stepping regimes with a TikZ figure, decision points and actions, people and agents on one environment, what a run records, and the vocabulary table. The other sections are outlines.
- Overleaf waits for you: `open-decisions.md`, section 1.
- The first commit on GitHub (c084834) is labelled "Framework…" but holds the whole skeleton: rebuilding the local commits went wrong after the workflow had to be taken out (the `gh` token may not push workflows), and the result was pushed before I saw it. Left as is, since pushed history is never rewritten here.
- Installed Tectonic (`brew install tectonic`, 20 MB; remove with `brew uninstall tectonic`) to compile the paper locally: `tectonic -X compile main.tex`. It builds with no undefined reference or citation; Overleaf (pdflatex) will be the final check.

### The software: PR 0 and PR 1, committed locally, nothing pushed

Worktree `~/GitHub/DBP/dev/fmri-gym-worktrees/neuroimagym` (its `.venv` has the `gui`, `triggers` and `crafter` extras), on top of `origin/main` 9a58bfd:

| Commit | Branch | What |
|---|---|---|
| 545aefb | `remove-session-editor` (PR 0) | the session editor removed, with what only it used; the committed session scripts tested instead |
| 1a3e538 | `neuroimagym` (PR 1) | the distribution is neuroimagym 0.2.0; records name its version so, old records still read |
| 6a8995b | `neuroimagym` | one `neuroimagym` command: `play`, `rig`, `gamepad`, `next-session`, `record`, `parse-logs`; `fmri_play.py` → `humans/play.py` |
| f700551 | `neuroimagym` | recorded names: `neuroimagym-log`, LSL default `neuroimagym`, calibrations in `~/.config/neuroimagym/pads` (old folder read, and said), Docker image |
| 15cd3e9 | `neuroimagym` | the remaining names: windows, docs, comments, session headers |

Verified on this Mac: the suite (`--changed origin/main` and `--backend gym,crafter`: 1581 passed on the last commit), ruff, lint-imports. Not played here: the backends whose engines are not installed (minihack, vizdoom, coom, baba_auto, vgdl, arc3, aigamestore, rushhour, stk, retro, ale, nethack); CI plays them once the PRs are open and ready. The PR descriptions are in `notes/prs/`; their "See it yourself" steps were run on both refs on 2026-11-25, and their outputs are quoted there.

### The documentation: PR 2, committed locally (2026-11-25, early morning)

Branch `docs-site`, on top of PR 1, six commits (5c546ef … 9c38830): the mkdocs site (33 pages, a strict build with no warning, HTML and PDF by `./build_docs.sh`), the backend READMEs holding their setup, the 80-line README, QUICKSTART and old `docs/*.md` folded in, AGENTS.md and the troubleshooting skill pointing to the docs, `tests/test_docs.py`, three adapter docstrings corrected. The planned PR 3 (generated command reference, backends table) is folded in: every page links to the reference. Suite on the branch: 1685 passed.

Drafted by four helper agents from the README, QUICKSTART, the DBP_Welcome pages (made DBP-agnostic) and the code; I wrote the overview pages, the README, the tests and the generator, and reviewed and integrated the rest. Their findings where the code disagreed with the old docs are in `notes/prs/pr-2-docs-site.md`; the code issues they found are in `open-decisions.md`, section 4. To read the site: `uvx --with mkdocs-material --with mkdocs-print-site-plugin mkdocs serve` in the worktree, or `dist/neuroimagym-docs.pdf` there.

The paper's Framework was corrected on one point the review found: only key presses get the operating system's timestamp today.

Scratch worktree `~/GitHub/DBP/dev/fmri-gym-worktrees/main-9a58bfd` (detached at PR 0's commit, with its own `.venv`) was used for the before/after checks; remove it with `git -C ~/GitHub/DBP/fmri-gym worktree remove ../dev/fmri-gym-worktrees/main-9a58bfd` when it is no longer needed.

## 2026-11-24, day (with you back)

- **The paper is one file**, `main.tex` (bbeb6bc): the sections are in it, in reading order, and `sections/` is gone. The software is NeuroimaGym in prose; the working title is technical (alternatives in `open-decisions.md`, section 2); authors for now: Yann Harel, Momchil S. Tomov, the DBP contributors.
- **Three outlined subsections** for what you asked to add, each with a `\status{}` line read in the code at 9a58bfd: Seeds (4.1), Byte-identical replay (4.2), The rig measurement suite (5.1). All three are implemented; what is not established is said (what the seed determines per backend; replay across machines; published rig numbers).
- **Table 4**, a placeholder comparing NeuroimaGym with experiment libraries, task collections and RL environments (Section 8.1), cited from the introduction's gap.
- It compiles with Tectonic (10 pages, no undefined reference); the sources are plain ASCII, as Overleaf's pdflatex needs.
- **The Overleaf zip**: `~/Desktop/neuroimagym-paper.zip`, `git archive` of `main` at the commit that adds this entry.
- **Issues #269 to #277** on tomov/fmri-gym, one per code problem the documentation work found (`open-decisions.md`, section 4), each checked on `main` (9a58bfd); #269 and #270 reproduced there. The photodiode sign needed no issue: the code settles it.
- **The documentation**: commit 5b739ba on `docs-site` adds five figures and seven screenshots and `tools/docs_images.py` (the PR 2 description says what and where). The suite on the branch, `--backend gym,crafter`: 1687 passed, 9 skipped.
