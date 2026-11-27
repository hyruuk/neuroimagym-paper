# Open decisions

Decisions that need a person, gathered since 2026-11-24. Each one has a recommendation, so that "yes" is an answer.

## 1. Overleaf: connected (2026-11-24)

The project was made on Overleaf from the zip of 61efcf8, and its history joined `main` in 9a9683b. Overleaf's git bridge names its one branch `main`, so `main` goes to `main` on both remotes; the routine is in `README.md`.

Still optional, for a PDF built on GitHub at every push: the `gh` token here has no `workflow` scope, so GitHub refuses any commit that adds a file under `.github/workflows/`. Run `gh auth refresh -h github.com -s workflow` (it opens a browser), and I move `notes/pdf-workflow.yml` into place. Overleaf compiles the paper anyway.

## 2. The paper

- **Title**: yours, 2026-11-24: "NeuroimaGym: Gymnasium environments as neuroimaging tasks for neuro-AI".
- **Authors** (yes for now, you said): Yann Harel, Momchil S. Tomov, and the DBP contributors, affiliations to fill in. The order and the contributor list are yours.
- **Spelling**: the draft uses British spelling (behaviour, synchronise), like the software's AGENTS.md. Journals accept either; say if you prefer American.
- **References to check by hand** (marked `% UNVERIFIED` in `references.bib`, each comment saying what is missing): Kempka 2016 (ViZDoom; pages and DOI), Paugam 2026 (eLife volume, article number and DOI), Zhang 2020 (Atari-HEAD; proceedings volume and pages), Gerstgrasser 2022 (CrowdPlay; first names), Stanford 2026 (Digital Brain Project; exact title). Schaul 2013 (VGDL) is written from library catalogue records, not the IEEE page. The file holds 71 entries, of which the text cites 22; the other 49 come from the research dossier.
- **The comparison table** (Table 4, a placeholder in Section 8.1): the tools (PsychoPy, Psychtoolbox, OpenSesame, goxpyriment, jsPsych, E-Prime and Presentation, NeuroGym, Gymnasium and ALE) and the 13 features are a first proposal; every cell but NeuroimaGym's is "?" until checked against each tool's documentation and cited. To decide: which tools a reviewer will expect (Expyriment? CNeuroMod's game setup? Unity-based task frameworks?), and whether the table belongs in the introduction or the discussion.
- **What the outline now says the software does** (the `\status{}` lines, read in the code at 9a58bfd):
  - Seeds: implemented. Every environment phase has a base seed, pinned in the curriculum or derived from the run's name and the phase's position (31 bits of SHA-256); episode e plays seed + e; the manifest says pinned or derived, each episode's log its seed. Open: what the seed determines in each backend.
  - Byte-identical replay: implemented and tested for every backend the suite plays, in a second process under another hash seed, pixels, logged variables, reward, ends and audio samples compared; savestates restored and played on; resumed runs too. Not tested: across machines, operating systems or library versions.
  - The rig measurement suite: implemented (display, frame pacing under load, triggers, controls, photodiode and sound; short and long; reports and a pooled table; latencies filed beside the rig). Not yet published: numbers from the acquisition rigs.
- **The vocabulary** (Framework, Table 3) is what the documentation follows; please read it first. The choices in it:
  - **environment** for what is played (a game or a task), and **backend** for the integration of an engine: issue #48 proposed renaming backend to environment, which would make one word mean two things; the draft keeps both words, each for one thing.
  - **engine time** and **human time** (the software's `run_time` is human time).
  - **fixed-rate**, **turn-based**, **flip-locked** for the three stepping regimes; the third is planned, and the paper says so.
  - **curriculum** kept for the file a run plays (the code's word); **phase** for an entry of it, **block** for an environment phase as played.
  - **participant** and **agent** for the two kinds of player.

## 3. The rename and the documentation (for you, then tomov)

- **NeuroimaGym in the documentation too?** The paper writes the software NeuroimaGym; the site, the README and the help still write `neuroimagym`, the command's name, everywhere. Recommendation: NeuroimaGym for the project in prose (README title, site name, the first line of each page), `neuroimagym` for the command, the package and the repository. One pass over PR 1 and PR 2 once you say so.
- **PR 0, removing the session editor**, needs tomov's agreement before anything else. It also removes what only the editor used: the session script's reader and writer (`fmri-ses` stays), saving a curriculum back with its includes (`save_curriculum` and helpers in `config/curriculum.py`, core), the keys table of `widgets.py`. Say if you would rather keep any of it.
- **Pushing PR 1 and PR 2 needs the `workflow` scope**: PR 1 edits `.github/workflows/tests.yml` (ruff no longer names `fmri_play.py`, which moved) and `docker-build.yml` (the image's name); PR 2 adds `docs.yml`. `gh auth refresh -h github.com -s workflow`, in a browser, once.
- **Run numbers in the committed session scripts** count across tasks (`--run 1` to `6` for six different games), while the removed editor numbered runs per task. BIDS accepts both; the documentation needs one rule. Recommendation: per task, as BIDS intends (`run-` counts repetitions of one task), and fix the examples in the docs PR.
- **`parse-logs`** is the name you gave for `bids-from-logs`; tomov may want another.
- **Recorded names**: the plan changes the log marker to `neuroimagym-log` (readers accept both) and the default LSL stream name to `neuroimagym` (rigs made with the form keep the name they wrote). Both are yours and tomov's call; the alternative is to keep `fmri-gym-log` and `fmri_gym` for continuity.
- **No licence** in the repository: the paper's code-availability statement and the documentation both need one. tomov's choice.

## 4. Code issues found while writing the documentation: filed on tomov/fmri-gym (2026-11-24)

Each was checked on `main` (9a58bfd) before filing; they are fixed separately from the rename.

| Issue | What |
|---|---|
| [#269](https://github.com/tomov/fmri-gym/issues/269) | `bids-from-logs` without `--subject` stops with `KeyError: 'start_epoch'` on a data root that holds a rig measurement (reproduced) |
| [#270](https://github.com/tomov/fmri-gym/issues/270) | a curriculum phase ignores a misspelled key and plays its default (reproduced: `"durations": 3` plays a 2 s fixation) |
| [#271](https://github.com/tomov/fmri-gym/issues/271) | `--subject` silently defaults to `sub-test` |
| [#272](https://github.com/tomov/fmri-gym/issues/272) | `rewind_on` on a backend without savestates stops on the block's first frame, not at the desk |
| [#273](https://github.com/tomov/fmri-gym/issues/273) | the rig form greys out the window size in fullscreen, where it is the canvas the screen shows |
| [#274](https://github.com/tomov/fmri-gym/issues/274) | the rig measurement's screen says bottom-right whatever the patch's corner; `--audio-click`'s help says every flash, the code clicks every other |
| [#275](https://github.com/tomov/fmri-gym/issues/275) | the config tests skip the team curricula in `configs/<team>/curricula/` (the seven BC ones pass today) |
| [#276](https://github.com/tomov/fmri-gym/issues/276) | the VLM policy's prompt names Crafter's genre for every game; a failed call is counted as an invalid reply too |
| [#277](https://github.com/tomov/fmri-gym/issues/277) | `run_docker.sh` mounts another project's folders; the Dockerfile has an unused SuperTuxKart path |

Settled, no issue: **the photodiode offset's sign**. The code measures it as the diode's edge minus the flip (`photodiode.match_edges`), a positive number, so light appears at `flip_time` plus the offset, as the new pages and the paper say; the old README's "subtract it from `flip_time`" was wrong, and the docs PR replaces it. The photodiode module's docstring ("subtracted in analysis") is ambiguous and could say the same.
