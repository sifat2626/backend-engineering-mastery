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

1. Confirm the active phase in `PROGRESS.md` and place all experiment files under `phases/phase-XX/lab/<experiment-name>/`.
2. Never commit credentials, `.env` files, or production data. Use synthetic local data and `.env.example` if configuration is needed.
3. Do not overwrite existing learner experiment code without permission.

## Experiment Structure

Every lab experiment must follow a clear scientific loop:

1. **Hypothesis:** State the exact expected behavior and the mechanism being tested (including runtime/DB versions).
2. **Minimal Setup:** Create the smallest reproducible Node.js script, SQL file, or command sequence needed to observe the behavior. Avoid unnecessary frameworks or boilerplate.
3. **Execution & Measurement:** Run the experiment and capture raw observable evidence (stdout, timings, `EXPLAIN (ANALYZE, BUFFERS)`, lock views, memory/CPU stats, or traces).
4. **Failure / Edge-Case Probe:** Perturb one variable (e.g., concurrency, payload size, isolation level, index absence, timeout) to observe the break point.
5. **Takeaway:** Have the learner articulate what the evidence proved or disproved, and what production trade-off it reveals.

## Verification & Cleanup

- Verify the experiment runs cleanly from a documented command in the lab's local `README.md`.
- Remember: running an agent-scaffolded lab does not automatically mark the block `completed` in `PROGRESS.md`.
