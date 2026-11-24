# Open decisions

Decisions that need a person, gathered during the night of 2026-11-24 to 25. Each one has a recommendation, so that "yes" is an answer.

## 1. Overleaf (needs you: your project and your token)

The GitHub side is ready. To connect Overleaf without a conflict on `main.tex`:

1. On Overleaf: **New Project → Upload Project**, with a zip of the repository as it is: `git -C ~/GitHub/humain/neuroimagym-paper archive -o ~/Desktop/neuroimagym-paper.zip HEAD`. (A blank project also works, but its template `main.tex` would conflict with ours on the first merge, and that is a `.tex` conflict you asked to decide yourself.)
2. Give me the project ID (the last part of the project's URL).
3. I add the remote and the branch mapping: `git remote add overleaf https://git.overleaf.com/<ID>` and `git config remote.overleaf.push refs/heads/main:refs/heads/master`.
4. In your own terminal (not through me), run `git -C ~/GitHub/humain/neuroimagym-paper fetch overleaf` once, and answer `git` as the username and your Overleaf Git token as the password (Overleaf: Account settings → Git integration). The keychain keeps it (`osxkeychain` is the credential helper of Homebrew's git here); it never goes into a URL or a file.
5. I run the first sync: `git merge --allow-unrelated-histories overleaf/master`, then push to both.

Also, for a PDF built on GitHub at every push: the `gh` token here has no `workflow` scope, so GitHub refuses any commit that adds a file under `.github/workflows/`. Run `gh auth refresh -h github.com -s workflow` (it opens a browser), and I move `notes/pdf-workflow.yml` into place. Optional: Overleaf compiles the paper anyway.

## 2. The paper

- **Working title**: "neuroimagym: step-based environments as a common medium for people and artificial agents in neuroimaging experiments". Alternatives: "Playing the same game: …"; "One environment for people and models: …". Recommendation: keep the working title until the abstract is final.
- **Authors and order**: left as a TODO in `main.tex`. Probably you, Momchil Tomov, and the DBP contributors; yours to decide.
- **Spelling**: the draft uses British spelling (behaviour, synchronise), like the software's AGENTS.md. Journals accept either; say if you prefer American.
- **References to check by hand** (marked `% UNVERIFIED` in `references.bib`; their publisher pages could not be read by the fetching tool, so they are written from memory): Peirce 2019 (PsychoPy2), Gorgolewski 2016 (BIDS), Mnih 2015 (DQN), Sonkusare 2019, Nastase 2020, Cross 2021, Bridges 2020, Tomilin 2023 (COOM). Kempka 2016 (ViZDoom) lacks pages and DOI. The Lab Streaming Layer paper is not in the file yet (not verified).
- **The vocabulary** (Framework, Table 3) is what the documentation will follow; please read it first. The choices in it:
  - **environment** for what is played (a game or a task), and **backend** for the integration of an engine: issue #48 proposed renaming backend to environment, which would make one word mean two things; the draft keeps both words, each for one thing.
  - **engine time** and **human time** (the software's `run_time` is human time).
  - **fixed-rate**, **turn-based**, **flip-locked** for the three stepping regimes; the third is planned, and the paper says so.
  - **curriculum** kept for the file a run plays (the code's word); **phase** for an entry of it, **block** for an environment phase as played.
  - **participant** and **agent** for the two kinds of player.

## 3. The rename and the documentation (for you, then tomov)

- **PR 0, removing the session editor**, needs tomov's agreement before anything else.
- **`parse-logs`** is the name you gave for `bids-from-logs`; tomov may want another.
- **Recorded names**: the plan changes the log marker to `neuroimagym-log` (readers accept both) and the default LSL stream name to `neuroimagym` (rigs made with the form keep the name they wrote). Both are yours and tomov's call; the alternative is to keep `fmri-gym-log` and `fmri_gym` for continuity.
- **`--subject` defaults to `sub-test`** when left out: a silent default that files data under the wrong participant. The plan proposes a separate small PR to make it required.
- **No licence** in the repository: the paper's code-availability statement and the documentation both need one. tomov's choice.
