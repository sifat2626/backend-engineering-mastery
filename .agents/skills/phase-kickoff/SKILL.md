---
name: phase-kickoff
description: >-
  Use this skill when the learner is ready to initialize a newly unlocked phase
  after ChatGPT has approved completion of the previous phase. Scaffolds only
  the single target phase directory from MASTER_ROADMAP.md and updates
  PROGRESS.md and NEXT_SESSION.md without generating learning materials in advance.
---

# Phase Kickoff

Initialize a single newly unlocked phase while strictly preserving curriculum authority and one-phase-at-a-time discipline.

## Authority Checks (Required Before Acting)

1. Read `AGENTS.md`, `MASTER_ROADMAP.md`, `PROGRESS.md`, and `NEXT_SESSION.md`.
2. Enforce explicit phase-unlocking rules:
   - Phase 1 may be initialized because it is the explicitly authorized starting phase.
   - For Phase N (N > 1), Phase N-1 must be marked `approved` by ChatGPT in `PROGRESS.md`.
   - Never unlock or initialize a future phase without ChatGPT's explicit approval.
   - Transferring already-approved roadmap content into the currently active phase is not phase unlocking.
3. Never scaffold future phases beyond the single target phase.
4. Never invent or alter phase titles, blocks, scope, allocated hours, or progression criteria from `MASTER_ROADMAP.md`.

## Steps

1. Locate the exact entry for Phase N in `MASTER_ROADMAP.md`. If the phase definition is missing, stop and ask the learner to transfer the approved definition from ChatGPT first.
2. Before editing an existing phase `README.md` or `checkpoint.md`, inspect its contents. Preserve existing learner work and previous feedback. Never overwrite them without explicit confirmation.
3. Create (or update without overwriting learner work) `phases/phase-XX/` containing only structural scaffolding:
   - `README.md` — Copy the approved phase title, scope, allocated hours, blocks, and progression criteria verbatim from `MASTER_ROADMAP.md`.
   - `notes/.gitkeep`
   - `lab/.gitkeep`
   - `checkpoint.md` — Blank mastery evidence template awaiting learner work and ChatGPT sign-off (if not already present).
4. Update `PROGRESS.md` to set Phase N to `in progress` (never `completed` or `approved`).
5. Update `NEXT_SESSION.md` with the exact starting block and immediate next step.
6. Do **not** generate conceptual notes, answers, or lab solutions during kickoff.

## Validation

- Confirm via `git status` that only the single target phase folder and tracking files were touched.
- Summarize every created/modified file and wait for learner confirmation before committing or pushing.
