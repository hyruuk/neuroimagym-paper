# Classical tasks in fmri-gym: feasibility report

Date: 2026-11-23. Author: Claude (Opus 5.5 / Fable 5.1) with Hugo. Status: feasibility established by two spikes; no code in fmri-gym yet.

## 1. The question

fmri-gym should run the classical neuroimaging tasks and localizers (DBP_CogTasks_fMRI, DBP_MTB, DBP_MEG, and more later) with everything the games get: one rig file, one trigger path, one eye-tracker integration, one log format, humans and agents on the same picture, bit-exact replay. Ideally an **existing experiment script runs unchanged**, so no task has to be translated.

Two candidate foundations were on the table: goxpyriment ([arXiv:2604.15245](https://arxiv.org/abs/2604.15245), Pallier et al., the runtime DBP_MEG is built on) and PsychoPy (what DBP_CogTasks_fMRI and DBP_MTB are built on).

## 2. Verdict

**Feasible, and the "unchanged script" goal holds for PsychoPy scripts.** The seam is the screen flip: run the script as a coroutine whose `win.flip()` hands the frame to the host and waits for the next `step()`. With a virtual clock and a key queue, an unmodified task becomes a Gymnasium-style env where one step is one flip and the action is a response with a reaction time. Verified end to end on WCST (DBP_CogTasks_fMRI) in two independent ways (section 5): 219 flips, 430 s of task time in 0.3 s (fake renderer) or 8.6 s (real PsychoPy), identical output CSV from both.

Recommended solution: **keep real PsychoPy and patch only clock, events and flip** (solution B, section 6). Pixels are PsychoPy's own, so the runs are replicated exactly; the patch is small.

Neither goxpyriment nor PsychoPy is the *foundation*; each is a front-end whose scripts run inside fmri-gym. The Go programs need a cooperation from upstream (section 6.2); the Python ones do not.

## 3. What exists today

### fmri-gym (the host)

- Every backend renders **into an array in memory**; `humans/outputs/display.py` blits it to the pygame window; `agents/agent_play.py` hands it to a model. No game owns the window. The seam is `humans/adapters/base.py` (`EnvAdapter`: `reset`, `step`, `render`, `sound`, `capture`, `restore`, `native_fps`).
- `humans/run.py` paces a block at a fixed `fps` (`frame_period`) or, in turn-based mode, one step per press; it timestamps flips with `perf_counter`, fires a frame trigger on the flip, and logs every step.
- Two backends already run a game as a **separate process in lockstep**: `aigamestore` (browser games through playwright) and `stk` (SuperTuxKart's game server). That is the shape a Go program would take.
- AGENTS.md rules that matter: game logic lives in a `gymnasium.Env` under `gym/<GAME>/`, adapters are thin; core changes are additive, default-off and nameless; anything unsupported must fail at start-up, never run silently degraded.

### The task codebases (what a compatibility layer must cover)

| Repo | Runtime | Size | API surface actually used |
|---|---|---|---|
| DBP_CogTasks_fMRI (WCST, VCST, ARC, syllogism) | PsychoPy 2026.2.5, `pyproject.toml`, `.venv` present (Python 3.13) | ~2,600 lines | `TextStim`, `Rect`, `ShapeStim`, `Polygon`, `Circle`; `core.Clock/wait/getTime/quit`; `event.getKeys/waitKeys/clearEvents`; `logging`; `monitors`; `pix2deg/deg2pix`; `win.callOnFlip`, `win.winHandle.push_handlers` (pyglet, Ctrl+Return → `os._exit(0)`); `buttons.py` polls pygame joysticks directly |
| DBP_MTB | PsychoPy via the MultiTaskBattery package (23 task classes, ~4,900 lines) + `dbp_tasks.py` (1,300 lines) | ~6,200 lines | all of the above plus `ImageStim`, `Line`, `MovieStim` (videos), `sound.Sound` (beeps, audio files), `Aperture` (stencil), `BoxStim` + `LightSource` (3D maze, raw `pyglet.gl`), `gui.Dlg`, its own `ttl_clock.py` (counts `t` presses), `gamepad.py`, `gc.collect()` around movies |
| DBP_MEG | **goxpyriment v0.28** (Go), 17 programs, 170–520 lines each | ~5,000 lines Go | `exp.Screen.Clear/Update`, `exp.Show/ShowTS/Wait`, `exp.Keyboard`, `exp.Marker.Fire/Arm` (TTL + photodiode), `exp.Data`; stimuli: text lines, rectangles, polylines, fixation cross, pictures |

Timing idioms in the Python tasks, all of which the virtual clock handles: `core.wait(d)` after a flip; `while clock.getTime() < end: getKeys(); core.wait(0.001)` response loops; `waitKeys(keyList=['t'])` for the scanner trigger; `ttl_clock.wait_until(t)` loops polling `t` presses. No threads, no timers.

Most DBP_MEG programs are MEG versions of tasks that also exist in MTB/CogTasks (ARC, VCS, Wisconsin, mental rotation, theory of mind, spatial/verbal WM, visual search, reading sentences, intuitive physics).

### goxpyriment (the paper, and the code DBP_MEG pins)

- Paper: a Go library over SDL3 for behavioural experiments, single static binary, OS-timestamped input, GC paused in timing loops, Black Box Toolkit validation, ~40 examples (Stroop, RSVP, masked priming, retinotopy, QUEST). Imperative API, **no agent/RL interface**, no web deployment, video timing and eye tracking listed as missing. The paper says GPLv3; **the v0.28 source and `LICENSE.txt` say Apache-2.0**.
- v0.28 (`~/go/pkg/mod/github.com/chrplr/goxpyriment@v0.28.0`, 45k lines) is well beyond the paper: an `eyetracker/` package with EyeLink and Tobii **bridges driven as Python subprocesses over newline-JSON** (`eyetracker/protocol.go`), many TTL drivers, vblank timing on Linux/macOS, a `//go:build js` split for a WebAssembly target. The split isolates exactly the functions a host would have to replace: `apparatus/screen_newscreen_{js,notjs}.go`, `screen_present_{js,notjs}.go`, `glrenderer_{js,notjs}.go`, `control/marker_ttl_{js,notjs}.go`, `control/platform_{js,notjs}.go`.
- `Experiment.Wait` loops on package-level `getTicks`/`clock.Wait` variables (already swapped in its tests); keyboard waits call `sdl.WaitEventTimeout` directly (`apparatus/keyboard.go:41`).

## 4. The mechanism

A script and a Gymnasium env control time in opposite ways: the script calls `flip()` and blocks until the frame is shown; an env waits for its caller's `step()`. Bridge: run the script in a **greenlet** and make the flip the switch point.

```
env.reset()/step(action)  ──switch──▶  script runs: draws, waits, polls keys
        ▲                                    │  win.flip()
        │                                    ▼
   frame array  ◀──────switch────────  flip: stamp onset, read back pixels
```

- **Clock.** `core.getTime`/`Clock`/`wait` read one clock object. *Virtual mode* (agents, tests, replay): `wait(d)` adds `d`; a flip rounds time up to the next vsync (1/60 s); nothing sleeps. *Real mode* (a participant): wall time; `wait` sleeps while pumping the rig's inputs.
- **Keys.** `getKeys`/`waitKeys`/`clearEvents` read a queue of `(key, time)`. Virtual mode: `step((key, rt))` schedules the key at `flip_time + rt`; a bounded `waitKeys` jumps straight to the next scheduled press; an unbounded `waitKeys` (the scanner trigger) switches out and asks the host, which injects the trigger key. Real mode: the rig's `InputDevice` events, named as the script expects through the phase's `controls`.
- **Flip.** Stamp the onset, grab the frame, run `callOnFlip` hooks (they read the clock, so stamps are exact), switch to the host. The script never knows.
- **Decision points for free.** The agent acts once per flip, not once per 16 ms frame; the script itself says where responses are possible because that is where it polls keys. No per-task annotation.

## 5. The spikes

Both run `DBP_CogTasks_fMRI/WCS/wcst_run.py` **without modifying it**, with a seeded random policy (90 % of flips answer with a key and an RT in 0.4–1.5 s). Code: `~/GitHub/DBP/dev/psych-backend-spike/` (`run_wcst_env.py` with `fakepsy/`, `run_wcst_real.py`).

### Spike A: fake `psychopy` (reimplemented drawing with PIL)

- ~250 lines for the 25 symbols WCST uses, plus a stub `pyglet.window.key`.
- 219 flips, 430.0 s virtual, **0.33 s wall**, 1920×1080 frames, onsets quantised to 1/60 s, WCST's own log and results CSV written (54 trials, RTs exact).
- Shows the mechanism; the pixels are not PsychoPy's (e.g. the ✗ feedback glyph drew as a tofu box).

### Spike B: real PsychoPy 2026.2.5, four things patched

Patched: `psychopy.clock.getTime` and `psychopy.core.getTime`, `psychopy.core.wait`/`psychopy.clock.wait`, `psychopy.event.getKeys/waitKeys/clearEvents`, `Window.flip` (read back the back buffer with `win._getFrame(buffer="back")`, call the original flip so `callOnFlip` hooks run, switch out). Everything else is PsychoPy's own code: every stimulus, units, colours, the window.

- 219 flips, 430.0 s virtual, **8.6 s wall**; readback 17.5 ms median per flip at the Retina backing size (3840×2160).
- **Results CSV byte-identical to spike A's** (same policy seed), so the clock/key semantics are consistent across both implementations and match PsychoPy's `timeStamped` conventions.
- Gotchas found:
  - `Window.__init__` calls `getActualFrameRate`, which flips until real time has passed; on a virtual clock it spins forever (99 % CPU in Cocoa). Fix: return the nominal rate in virtual mode.
  - A **hidden window hangs on macOS** at the first readback (`set_visible(False)` then `_getFrame`). A visible window works. Headless agent runs on macOS need a visible window; on Linux, a virtual display (as `stk` already needs).
  - macOS Retina: the GL buffer is 2× the requested window; readback size must be handled (`useRetina=False` or downsample) so humans and agents get the same resolution.
  - greenlet + pyglet/Cocoa window creation works (tested in and out of the greenlet).

## 6. Solutions compared

### 6.1 Python (PsychoPy scripts)

| | A. Reimplement PsychoPy's API | **B. Real PsychoPy, patch clock/events/flip** | C. Two renderers (real PsychoPy for humans, shim for agents) |
|---|---|---|---|
| Pixels | ours, approximate | PsychoPy's, exact | differ between human and agent |
| Code to write/maintain | every stimulus class, forever chasing | ~100 lines of patches + the env | both A and the human path |
| Dependencies in fmri-gym | none | PsychoPy as an extra (lazy import, like every backend) | PsychoPy |
| GL context needed | no | yes (CI needs a virtual display) | yes |
| Speed headless | 1300× real time | 50× real time | — |
| Movies, apertures, sound, 3D | each one new code | PsychoPy's own; sound and TTL need routing to fmri-gym's audio/triggers | — |
| Replicates existing runs exactly | no | **yes** | humans yes, agents no |

**B** is the recommendation. A's spike stays useful as a dependency-free reference of the mechanism and as a CI fallback for timing logic tests (identical CSV proves the semantics match).

### 6.2 Go (goxpyriment / DBP_MEG programs)

A DBP_MEG binary opens an SDL window, presents frames, reads SDL keys and ticks, fires TTLs. To be a backend, those four must be redirected, as in Python, but across a process boundary: a **subprocess in lockstep** (the `aigamestore`/`stk` shape). Each `present()` writes the frame to a pipe and blocks until the host replies with the key events since the last frame and the clock value to adopt; markers go down the same pipe to fmri-gym's triggers.

| | (i) `driver` build mode upstream | (ii) Fork goxpyriment | (iii) Port the 17 programs to Python (solution B) | (iv) Screen capture + OS key injection |
|---|---|---|---|---|
| Who does the Go work | Pallier (with a precise ask), or a PR from us | us, forever | nobody; Python rewrite instead | nobody |
| Fit with upstream design | good: the `js`/`notjs` split already isolates present/newscreen/renderer/marker; clock vars already swappable; his eye-tracker bridge already speaks newline-JSON to a subprocess | diverges from a 45k-line, fast-moving codebase | n/a | n/a |
| Exact replication of the MEG runs | yes | yes | no (same paradigm, new code) | no (timing uncontrolled, no virtual clock) |
| Effort | ~1–2k lines Go in `apparatus/` + `control/`, plus the Python host side | same + maintenance | 17 × (170–520 lines) | rejected |

Recommendation: ask Pallier for (i) now (cheap; the answer changes the plan) and do not block on it. If the paradigms rather than the exact MEG runs are what matters, (iii) covers most of DBP_MEG through the PsychoPy twins in MTB/CogTasks.

## 7. How it lands in fmri-gym

- `gym/psychopy_script/`: a plain `gymnasium.Env`. `reset()` starts the script (greenlet, virtual or real clock); `step(action)` with `action = (key, rt) | None`; observation = the flipped frame; `info` carries the flip onset and the script's own log lines. The patches live here. Knows nothing about fMRI (AGENTS.md rule 1); the scanner trigger is "the host answers an unbounded key wait".
- `humans/adapters/psychopy.py`: thin. Builds the env from the phase (`script`, `expInfo`, `controls`), maps rig controls to the key names the script expects, routes `sound.Sound` to `EnvAdapter.sound` and the script's TTL calls to `block_extra`/triggers.
- Curriculum: one script = one `GamePhase` (decided). fmri-gym owns trigger, duration, rig, data location; the script's own trigger wait receives the injected key; `core.quit()`/`sys.exit`/`os._exit` are intercepted (the Ctrl+Return handler in CogTasks calls `os._exit(0)`); the script's own CSV/log land under the block's BIDS directory next to fmri-gym's log.
- **Core change needed (rule 2, additive, default-off): variable step duration.** `run.py` paces at `frame_period`; a script's flips are irregular (219 flips over 430 s). The env reports each frame's onset and the loop must present at that time, log per flip, and keep the frame trigger on the flip. This is the part a reviewer must read closely.
- Agents: `agent_play.py` works unchanged once the adapter exists; action space is (key, rt).
- Replay: seed `random`/`numpy` inside the script's greenlet; log per flip (not per 60 Hz frame); the frame is then a pure function of (script, inputs). CogTasks/MTB read jitters from CSVs, which helps.
- Install: `psychopy` as an extra (`--extra psychopy`), pinned to what the tasks pin (2026.2.5), lazily imported in `_make`.

## 8. Risks and open questions

1. **Real-clock mode is untested.** The spikes ran the virtual clock only. In the scanner, `core.wait` and `waitKeys` must sleep while pumping the rig's inputs and keeping vsync, with `perf_counter` stamps like the games. Same patches, second clock backend.
2. **Timing precision for classical tasks.** fmri-gym's loop was built for games; classical tasks want flip-locked onsets and ms-level RTs. Measure with the photodiode on the BC rig before trusting any human run.
3. **API long tail** (MTB): `MovieStim` (per-frame video timing), `Aperture`, `BoxStim`/`LightSource` + raw `pyglet.gl` (3D maze), `gui.Dlg`, MTB's `ttl_clock` and `gamepad.py`, `buttons.py`'s direct pygame joystick use (collides with fmri-gym's own pygame). Under B these mostly work since PsychoPy draws them; the input modules need the rig's devices instead of their own.
4. **Scripts with side effects**: `os._exit`, their own file naming, `sys.path` surgery, module-level globals (`buttons.py`) that survive between blocks in one process. Running each script in a fresh subprocess instead of a greenlet is the fallback (slower start, cleaner).
5. **Platform**: hidden windows hang on macOS; Linux needs a virtual display for headless runs; Windows untested.
6. **Maintenance**: an allowlist of supported PsychoPy symbols checked at start-up, failing loudly on anything else (rule 4), keeps the layer honest as tasks evolve.
7. **goxpyriment** depends on Pallier's answer.

## 9. Validation protocol for milestone 1 (decided: replicate the CogTasks runs exactly)

For each of WCST, VCST, ARC, syllogism:

1. Run the task standalone (its own `run.py`, real PsychoPy) with a scripted responder, and under fmri-gym with the same scripted responses.
2. Diff the task's own results CSV and log: identical except file names and dates.
3. Pixel-diff the frames at each flip (standalone frames captured with the same `_getFrame` readback): identical.
4. Replay the fmri-gym log in a second process: bit-exact (the existing `tests.verify_replay`).
5. One human run on the laptop rig and one on the BC rig with the photodiode: onsets within a frame of the logged flip.

## 10. Decisions so far

- Picture: one renderer for humans and agents, PsychoPy's own (solution B).
- Block model: one script = one `GamePhase`; fmri-gym owns trigger, duration, rig and data location.
- Milestone 1: the four CogTasks reproduced exactly, then MTB's long tail (movies, sound, apertures, 3D), then DBP_MEG depending on Pallier.
- Pending: email Pallier about a `driver` build mode in goxpyriment.

## 11. Pointers

- Spikes: `~/GitHub/DBP/dev/psych-backend-spike/` (`run_wcst_env.py` + `fakepsy/` = A; `run_wcst_real.py` = B; `probe.py` = the greenlet/clock isolation test). Run B with `DBP_CogTasks_fMRI/.venv/bin/python run_wcst_real.py --visible`.
- fmri-gym seam: `humans/adapters/base.py`; loop: `humans/run.py` (`_episode`, `_show`); display readback target: `humans/outputs/display.py:draw_frame`.
- PsychoPy patch points (2026.2.5): `psychopy/clock.py:37` (`getTime`), `:533` (`wait`); `psychopy/event.py:368/479/984`; `psychopy/visual/window.py:1254` (`flip`), `:653` (`getActualFrameRate` in `__init__`).
- goxpyriment seam: `apparatus/screen_present_notjs.go:34` (`present`), `apparatus/screen_newscreen_notjs.go:284`, `apparatus/keyboard.go:41` (`waitSDLKeyEvent`), `control/experiment.go:1042–1094` (`pumpFrame`, `Wait`, swappable `getTicks`), `eyetracker/protocol.go` (the newline-JSON bridge protocol to copy).
- Task API inventories: section 3 table; the grep commands are trivial to rerun (`grep -ohE "\b(visual|core|event|logging|data|monitors|sound)\.[A-Za-z_]+"`).
