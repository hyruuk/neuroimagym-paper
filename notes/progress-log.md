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

Verified on this Mac: the suite (`--changed origin/main` and `--backend gym,crafter`: 1581 passed on the last commit), ruff, lint-imports. Not played here: the backends whose engines are not installed (minihack, vizdoom, coom, baba_auto, vgdl, arc3, aigamestore, rushhour, stk, retro, ale, nethack); CI plays them once the PRs are open and ready. The PR descriptions are drafted in `notes/prs/`; their "See it yourself" steps still have to be run on both refs before posting.
