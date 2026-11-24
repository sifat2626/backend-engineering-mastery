---
name: lab-experiment
description: >-
  Use this skill when designing, scaffolding, or running a minimal reproducible
  Node.js, PostgreSQL, or systems experiment inside the active phase's lab/
  directory to empirically verify a backend mechanism, edge case, or failure mode.
---

# Lab Experiment

Build and run small, falsifiable experiments so backend behavior is verified empirically rather than accepted on faith.

## Authority & Safety Checks

1. Confirm the active phase in `PROGRESS.md` (and its conceptual scope in `LEARNING_JOURNEY.md`) and place all experiment files under `phases/phase-XX/lab/<experiment-name>/`.
2. Never commit credentials, `.env` files, or production data. Use synthetic local data and `.env.example` if configuration is needed.
3. Do not overwrite existing learner experiment code without permission.

## Experiment Structure

Connect the experiment directly to the active block's motivating problem (and the recurring promotional-code/commerce application where suitable, without forcing it onto low-level topics):

1. **Problem Context & Prerequisite Boundary:** State what failure or question in the active block this experiment tests, which concepts are prerequisites, and treat any incidental setup tooling (e.g., a helper runner or local container command) strictly as **Supporting** so the learner focuses on the target mechanism.
2. **Hypothesis:** State the exact expected behavior and mechanism being tested (including runtime/DB versions).
3. **Minimal Setup:** Create the smallest reproducible Node.js script, SQL file, or command sequence needed to observe the behavior. Avoid unnecessary frameworks or boilerplate.
4. **Execution & Measurement:** Run the experiment and capture raw observable evidence (stdout, timings, `EXPLAIN (ANALYZE, BUFFERS)`, lock views, memory/CPU stats, or traces).
5. **Failure / Edge-Case Probe:** Perturb one variable (e.g., concurrency, payload size, isolation level, index absence, timeout) to observe the break point.
6. **Takeaway:** Have the learner articulate what the evidence proved or disproved, how it updates their mental model, and what production trade-off it reveals.

## Verification & Cleanup

- Verify the experiment runs cleanly from a documented command in the lab's local `README.md`.
- Remember: running an agent-scaffolded lab does not automatically mark the block `completed` in `PROGRESS.md`.
