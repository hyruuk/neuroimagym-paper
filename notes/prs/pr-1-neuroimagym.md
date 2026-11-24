# PR 1 — The project is neuroimagym: one `neuroimagym` command, a subcommand per tool

Branch `neuroimagym` (local, in `~/GitHub/DBP/dev/fmri-gym-worktrees/neuroimagym`), stacked on PR 0 (`remove-session-editor`): review its own commits only. Not pushed, not opened.

Before pushing: the branch changes `.github/workflows/tests.yml` and `docker-build.yml`, and the `gh` token on this Mac may not push workflow files (`gh auth refresh -h github.com -s workflow`).

Order with tomov: agree on the names (open-decisions.md, section 3) → open this PR → when green, tomov renames the repository (Settings → `neuroimagym`; GitHub redirects the old URLs, the git remotes and the GHCR image name) → merge → tag `v0.2.0`.

---

## Before

- The distribution was `fmri-gym` 0.1.0; seven commands were on `PATH`: `fmri-play`, `fmri-ses`, `rig-setup`, `gamepad-calibration`, `session-logger`, `bids-from-logs` (and `session-editor`, removed by PR 0). Three had generic names, two an `fmri-` prefix, and nothing tied them together.
- Every manifest and session recording recorded the code's version as `versions.fmri_gym`; logs were marked `fmri-gym-log`; controller calibrations lived in `~/.config/fmri-gym/pads/`; the LSL stream defaulted to `fmri_gym`; the Docker image was `fmri-gym:latest`.

## Why that was a problem

The software runs fMRI, MEG, EEG and behavioural sessions, and its name said fMRI. Generic command names on a shared `PATH` belong to nothing and can collide with another package's; a person had to know seven names to find the tools.

## Now

- The distribution is `neuroimagym` 0.2.0 (`pyproject.toml`; `uv.lock` regenerated). The code asks `importlib.metadata` for it through one constant, `provenance.DISTRIBUTION_NAME`.
- One command on `PATH`, `neuroimagym`, with a subcommand per tool (`humans/standalone/cli.py`): `play`, `rig` (`--measure`, `report`, `pool`), `gamepad`, `next-session`, `record` (`start`, `mark`, `stop`), `parse-logs`. `neuroimagym` alone lists them; `neuroimagym <subcommand> --help` gives the tool's own usage, which names it `neuroimagym <subcommand>`. A tool is imported only when named.
- `fmri_play.py` is `humans/play.py`, a layer of its own between `standalone/` and `run.py` (the import contracts say so); `session_script.py` is `next_session.py`. Each tool has a `build_parser(prog)` that builds nothing else, and `main(argv, prog)`.
- The committed session scripts play `uv run neuroimagym play …` and record with `uv run neuroimagym record …`, and their header says neuroimagym.
- Recorded names follow, and what was recorded before still reads: `versions.neuroimagym` (`provenance.package_version_of` reads the old key; the rig measurement's table column is `neuroimagym_version`); the log marker `neuroimagym-log` (`log_format.version` unchanged, which is what readers compare); calibrations in `~/.config/neuroimagym/pads/`, with the old folder read, and said so on the console, when the new one has none of a controller's; the default LSL stream `neuroimagym` (rigs that name theirs, as `rig-setup` writes them, keep it); the Docker image `neuroimagym:latest`.
- Core files touched (rule 2): `humans/play.py` (moved), `config/run.py` (the distribution's name), `save/provenance.py`, `save/logging.py`, `sync/triggers.py`, `inputs/calibration.py`. Each change renames, or adds a read of an old name; none names a backend.

## Why that solves it

One name for the project and its one command; the tools are found with `neuroimagym --help`, and their names cannot collide. Old data, old rig measurements and old calibrations stay readable, so the rename needs nothing done to a dataset.

## How it was verified

TODO: fill from the final run (the suite's summary line, the skip lines, the hand checks below).

## See it yourself

TODO: run both sides as written before posting.

- Before (`origin/main`): `uv sync --extra gui && uv run fmri-play --help` prints the usage of `fmri-play`; a headless run's `manifest.json` has `versions.fmri_gym`.
- After (this branch): `uv sync --extra gui && uv run neuroimagym` lists the subcommands; `uv run neuroimagym play --help`; the same headless run's manifest has `versions.neuroimagym` and `log_format.format == "neuroimagym-log"`.

## Outside this repository (for the release note)

- A session script that is not committed here: `sed -i.bak -e 's/uv run fmri-play/uv run neuroimagym play/g' -e 's/uv run fmri-ses/uv run neuroimagym next-session/g' -e 's/uv run session-logger/uv run neuroimagym record/g' -e 's/uv run rig-setup/uv run neuroimagym rig/g' -e 's/uv run gamepad-calibration/uv run neuroimagym gamepad/g' -e 's/uv run bids-from-logs/uv run neuroimagym parse-logs/g' -e 's/^# fmri-gym session:/# neuroimagym session:/' <script>`. A script that was not updated stops at its first line: `uv` finds no `fmri-play`.
- Every checkout: `uv sync` (with its extras), and delete `fmri_gym.egg-info/` at its root.
- Calibrations: move `~/.config/fmri-gym/pads/*` to `~/.config/neuroimagym/pads/` (the run says so until then).
- A MEG/EEG recorder that selects the LSL stream `fmri_gym` from a rig that does not name it: set `"lsl_stream_name"` in the rig file, or select `neuroimagym`.
- Analysis code: accept `versions.neuroimagym` and `versions.fmri_gym`, and both log markers.
- DBP_Welcome (submodule folder `fmri_gym`, `uv run rig-setup` in its installers, `sessions/demo.sh`), the scheduler (`adapters/fmri_gym.py` matches `uv run fmri-play` and the old header), session-menu's copies of the example sessions, DBP_MEG's `sessions/sub-01_sess-00.txt`, DBP_MTB and DBP_CogTasks help strings: each follows on its side.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
