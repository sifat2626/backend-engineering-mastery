---
name: phase-checkpoint-handoff
description: >-
  Use this skill when the learner has finished studying and experimenting in the
  active phase and wants to compile a structured mastery evidence packet in
  checkpoint.md for review and final approval by ChatGPT (Curriculum Owner and
  Instructor).
---

# Phase Checkpoint Handoff

Prepare a clean, verifiable mastery handoff packet for ChatGPT without fabricating mastery or self-approving the phase.

## Authority Checks (Critical)

1. ChatGPT retains **sole final authority** over mastery assessment and phase completion (`approved` status).
2. Never write the learner's conceptual answers or mastery reflections for them. Only organize, cross-link, and audit evidence the learner actually produced.
3. Generated explanations, AI-created notes, scaffolds, and suggested solutions are **not** evidence of learner mastery.
4. Never mark a phase `approved` in `PROGRESS.md` without explicit ChatGPT sign-off.

## Steps

1. Read `MASTER_ROADMAP.md`, `LEARNING_JOURNEY.md`, `PROGRESS.md`, `phases/phase-XX/README.md`, `phases/phase-XX/checkpoint.md`, and all learner-authored files in `phases/phase-XX/notes/` and `phases/phase-XX/lab/`.
2. Audit completeness against the approved phase criteria:
   - List every required block/objective and link to the learner's corresponding learner-authored notes or verified lab artifacts.
   - Run a **Connected Mental Model Check:** verify whether the learner's notes demonstrate how blocks inside the phase connect into one coherent mental model and surface the unanswered engineering question motivating the next phase in `LEARNING_JOURNEY.md`.
   - Flag any uncovered objectives, unverified claims, weak cross-block connections, or incomplete experiments honestly as open gaps.
3. Update `phases/phase-XX/checkpoint.md` while **preserving all existing learner reflections, lab evidence, and prior ChatGPT feedback**:
   - **Evidence Inventory:** Links to learner-authored notes, verified labs, and `BACKEND_KNOWLEDGE_INDEX.md` entries added in this phase.
   - **Open Gaps / Uncertainties:** Explicit list of anything still needing clarification or Instructor probing.
   - **Instructor Review Section:** Clean placeholder for ChatGPT's assessment, feedback, and `approved` sign-off (preserving any prior feedback).
4. Produce a concise copy-ready summary the learner can paste into ChatGPT for mastery evaluation.
5. Update `NEXT_SESSION.md` to note that the phase checkpoint is awaiting ChatGPT evaluation.
6. A phase may be marked `completed` in `PROGRESS.md` **only** when:
   - Learner-authored evidence supports the required objectives,
   - Open gaps have been explicitly acknowledged by the learner, and
   - The learner confirms submission.
   Never mark an entry or phase `approved` without explicit ChatGPT sign-off.
