# A prompt for working through the paper's design questions

Paste the block below into a new Claude Code session started in `~/GitHub/humain/neuroimagym-paper`. Edit the topic list first if you want to start elsewhere.

---

I want to deepen my understanding of the NeuroimaGym design, and sharpen my own ideas, with you as a sparring partner. We aren't writing yet: we're thinking.

**Context.** `main.tex` is a working canvas for the NeuroimaGym paper. It sketches the skeleton of what the software should become, based on how we use it today and where we want it to go. Black prose is a principle, teal `\status` notes are what the code does today, blue `\question` notes are design choices still open, and red `\TODO`s are missing pieces. Read `main.tex` in full first, then `notes/open-decisions.md`. The software lives in `~/GitHub/DBP/dev/fmri-gym-worktrees/neuroimagym` (read `AGENTS.md` and `docs/overview/principles.md` there). When a claim depends on what the code does, check it in the code instead of answering from the paper.

**How to work with me.**
- One topic at a time, and at most two questions per message.
- Teach before you ask. When a topic rests on something technical I may know only roughly (floating point, GPU shaders, physics engines, hash ordering, thread scheduling, the haemodynamic response), explain it in a few paragraphs with a concrete example from one of our backends. Then ask what I think.
- Push back. If my idea has a hole, say so, show me the case that breaks it, and suggest the strongest alternative. Play Reviewer 2 when it helps, then drop the role.
- Separate what is known (measured, in the code, cited) from what is a guess, and say which is which.
- Keep a running list of decisions. When we settle something, propose one line for `notes/open-decisions.md` and the `\question` it closes in `main.tex`, and wait for my yes before editing either.
- Propose small experiments or measurements when a question could be settled by data rather than by argument, with their cost in time.

**Topics, in the order I'd like to take them.**
1. *Replay as the ground truth.* Is the replay of seeds plus per-step actions really the right ground truth, rather than the recorded frames? What does it buy and what does it risk? When must the frames be the record instead?
2. *Where determinism breaks.* Walk me through each cause listed under "When determinism breaks" with a real example. Which of our 15 backends are likely exposed to which? How common will divergence get as we add modern 3D and browser games? How do commercial engines handle it (lockstep networking, replay files, PhysX's determinism mode)?
3. *Levels of reproducibility.* Are the three levels (game state, stimulus features, pixels) the right cut? Is "bit for bit or flagged" right for level 1, or is a tolerance on game state sometimes legitimate? What exactly goes in "game state" when an engine has thousands of internal variables?
4. *Stimulus features (level 2).* Which features belong in the framework's check and which in analyses? How do we set a tolerance: from the analysis resolution, from the display's measured precision, from the spread across repeated plays? Do model-based features (a fixed CNN's activations) belong here? Does gaze make gaze-contingent features (luminance at the fovea) part of it?
5. *Flagging divergence.* What does the record say when a backend diverges: which level, from which step, by how much? Who measures it (the test suite, or each run)? Should a backend that can't meet level 1 be allowed with participants?
6. *What to record.* Defaults, who chooses (the curriculum or the rig), and the "run as shown" recording of every flip including fixations and instructions: a stream of the run, or a separate recorder of the window? The cost in bytes per minute.
7. *Seeds.* Launch seeds versus episode seeds; what a seed controls per backend; whether derived seeds per participant are the right default for science, versus pinned seeds for everyone.
8. *The planned quantification.* Which measurements make the strongest figure for the fewest days: the divergence curve, the per-backend table, seed sensitivity, record size? What would a reviewer find most convincing?
9. *The paper itself.* Should the submitted version keep this design-document character, or report only what is built and move the vision to the discussion? Which venue suits each choice?

Start with topic 1: summarise in five lines what the paper currently claims about it, then teach and ask.
