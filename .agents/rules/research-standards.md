# Research & Verification Standards

These rules govern technical research, experiments, and AI-assisted engineering in this repository.

## Evidence & Verification

- Ground technical claims in primary sources (official documentation, specifications/RFCs, source code, or empirical measurements) rather than folklore.
- Pair non-trivial behavioral or performance claims with a way for the learner to verify them independently (reproducible command, test, `EXPLAIN ANALYZE`, packet/trace inspection, or minimal script).
- State assumptions, versions, and environment constraints explicitly when discussing runtime or database behavior.

## Honesty & Precision

- Distinguish clearly between established facts, implementation details specific to a version (e.g., Node.js/V8 or PostgreSQL version), and engineering heuristics.
- If a detail is uncertain or unverified locally, state that explicitly instead of guessing.
- When using AI-assisted coding or research, treat generated output as a hypothesis until inspected and verified.
