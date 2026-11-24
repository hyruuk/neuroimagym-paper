# PR 2 — The documentation is a website: docs/, built with mkdocs

Branch `docs-site` (local, in `~/GitHub/DBP/dev/fmri-gym-worktrees/neuroimagym`), stacked on PR 1 (`neuroimagym`): review its own six commits. Not pushed, not opened. Pushing it needs the `workflow` scope too (it adds `.github/workflows/docs.yml`). The planned PR 3 (generated reference) is folded in, since every page links to the generated command reference.

---

## Before

- One 888-line README for every reader (installing, rigs, sessions, the curriculum format, nine per-game sections, the design, the data format, timing, models, a TODO list), plus QUICKSTART, `docs/DEPENDENCIES.md`, `docs/SAVESTATES.md`, `docs/MACHINE_REQUIREMENTS.md`, two DBP game lists, and a dated test record. The rig's reference sat under an H4 called "Games without a curriculum"; session scripts under "Curriculum format".
- The files disagreed with each other and with the code: the start key was SPACE in six places and RETURN in the code; the task label was "the file name" in one section and the curriculum's camelCase path in the code; a curriculum's `triggers` section was documented and refused; `pip install -e ".[dbp]"` was documented and does not resolve; adding a backend was "one adapter, nothing else changes" in one place and a seven-step checklist in another; and so on (the full list is below).
- Each error message's fix was in an agent skill, which the README sent people to; the documentation written for experimenters lived in another repository.

## Why that was a problem

Whatever a reader came for, they had to read past the rest, and what they found could be wrong. Every pull request added its caveat where the main instructions were, so the drift grew. Nothing checked that a documented command or link still worked.

## Now

- `docs/`: 33 pages, rendered by mkdocs-material as a website (`mkdocs.yml`), and as an offline HTML folder and a PDF by `./build_docs.sh`; `.github/workflows/docs.yml` builds both on every pull request that touches them and publishes the site on GitHub Pages from `main`.
  - **Overview**: what it is and is not, its features (each linked to its page), the concepts and their words (those of the methods paper's framework), and the principles and recommendations, with a section for an AI assistant operating the tools.
  - **Getting started**, **the rig** (the rig file with the mock and acquisition rigs, making it, measuring it, controllers, triggers), **sessions** (curricula, session scripts, session launchers, running a session, the session recording), **the data** (layout, BIDS, replay, timing), **backends**, **playing with a model**, **how it works**, **reference** (the commands, the curriculum format, the configs folder, a glossary), **troubleshooting** (every start-up message and its fix), **contributing** (architecture, adding a backend, tests and CI).
  - `tools/docs_tables.py` writes what the code defines: every subcommand's `--help` (`docs/reference/cli.md`) and the backends table.
- `README.md` is an 80-line front door. QUICKSTART and the old `docs/*.md` are folded into the pages and removed; `docs/local-testing/` is removed (it stays in the history); the DBP game lists move to `configs/BC/`.
- Each backend's README (`configs/curricula/<backend>/common/README.md`) holds its setup, moved from the README's per-game sections.
- AGENTS.md points people and agents to the docs and adds the documentation's own rules to Rule 3; the troubleshooting skill keeps only the agent's procedure and points to `docs/troubleshooting.md`.
- `tests/test_docs.py` (engine-free, so it runs in CI's core job): links lead to files, every page is in the menu, former names appear only where a line says what they became, the generated parts are current, and every `neuroimagym` command a page shows parses with that subcommand's own parser.
- Code: no behaviour change. Help texts of `play` and `rig` say what the reference shows; three adapter docstrings are corrected; pointers to README sections point to pages.

## Why that solves it

A reader finds a page per task; each fact has one page, which the others link to; the parts the code defines are generated from it; and the test suite fails when a link, a name, a command or a generated part goes stale.

## How it was verified

- `mkdocs build --strict`: no warning. `./build_docs.sh`: the HTML zip and the PDF (cover, contents, every page).
- `uv run pytest tests/test_docs.py`: 104 passed. The suite on the branch, `uv run pytest --backend gym,crafter`: 1685 passed, 9 skipped (the Linux session reader and backends not selected). ruff and lint-imports clean.

## See it yourself

- `uvx --with mkdocs-material --with mkdocs-print-site-plugin mkdocs serve`, then http://127.0.0.1:8000.
- `./build_docs.sh`: `dist/neuroimagym-docs.pdf`, `dist/neuroimagym-docs-html.zip`.
- `uv run python tools/docs_tables.py --check`; `uv run pytest tests/test_docs.py -q`.

## What the pages say differently from the old README, because the code says so

The start key is RETURN, and CTRL+ESCAPE quits (ESCAPE alone does nothing); `minimal.json` is fullscreen; a calibration file needs `"format": 1`; the console prints `NOT SET: sync.mode, backend`; a curriculum has only the `curriculum` key; the task label is the curriculum's path after the last `curricula/`, in camelCase (so a team curriculum and a shared one at the same relative path share a label); a block folder holds `game_events.jsonl`, `frames.h5` and `audio.h5`; only ale, retro, vgdl and crafter take a savestate every frame, the others one at the block's end by replay (stk none); a phase muted with `has_audio: false` neither plays nor records its sound (a rig-level mute records it); only key presses get the session recording's operating-system timestamp; models play the participants' episodes only when the curriculum pins its seeds; `agents/agent_replay.py` and `agents/vlm/` exist; `pip install -e ".[dbp]"` does not resolve; `fps` is required; `--dummy-trigger` does not skip a survey; SuperTuxKart relaunches for every episode after the first; Rush Hour's `puzzle` and `binary` are `env_kwargs`; stable-retro ids end in `-v0`.

## For you to look at (also in open-decisions.md)

- The photodiode offset: the old README said to subtract the flip-to-light delay from `flip_time`; the pages say light reaches the screen at `flip_time` plus the delay. Check the sign you use in analysis.
- `neuroimagym parse-logs` without `--subject` crashes (`KeyError: 'start_epoch'`) on a data root that holds rig measurements (`sub-rig`); the page tells readers to convert one participant at a time. A one-line fix for a follow-up PR.
- Windows was not tried; the install page's Windows limits are read from the packages' wheels.
- Whether the GHCR image is public, and its name after the rename.
- The session launchers' limits with neuroimagym's scripts are written as facts on `sessions/launchers.md`; their reconciliation is the planned follow-up.

🤖 Generated with [Claude Code](https://claude.com/claude-code)
