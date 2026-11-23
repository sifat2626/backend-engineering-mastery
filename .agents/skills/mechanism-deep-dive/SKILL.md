---
name: mechanism-deep-dive
description: >-
  Use this skill when the learner wants a deep, mechanism-level exploration of a
  backend concept, protocol, OS primitive, runtime behavior, or database internal
  within the active phase. Enforces coverage of mechanisms, invariants, failure
  modes, ecosystem alternatives, trade-offs, and a verification path.
---

# Mechanism Deep Dive

Guide a rigorous, mechanism-level investigation of a topic inside the active phase.

## Authority & Scope Checks

1. Check `PROGRESS.md` and `NEXT_SESSION.md` to confirm the topic belongs to the active phase and block.
2. Do not mark the topic `completed` after generating an explanation; mastery requires learner synthesis and verification.
3. Do not overwrite existing learner notes unless explicitly asked.

## Required Investigation Structure

Address the topic across six concrete dimensions:

1. **Problem & Definition:** What exact engineering problem this concept or technology solves, and what it is (no hand-waving).
2. **Underlying Mechanism:** How it actually works under the hood (data structures, memory layout, OS syscalls, wire framing, scheduling, concurrency control, or storage engine mechanics). Cite primary sources or version-specific details where relevant.
3. **Invariants & Edge Cases:** What assumptions must hold, and where naive mental models break.
4. **Production Failure Modes:** How this fails under real load, partial failure, resource exhaustion, contention, or misconfiguration—and how failures manifest in telemetry/logs.
5. **Alternatives & Trade-Offs:** Compare against broader backend ecosystem alternatives (including Node.js / PostgreSQL context where applicable), naming what is gained and sacrificed.
6. **Independent Verification:** Propose a minimal, concrete experiment, query, trace, or primary-source check so the learner can verify the mechanism firsthand.

## Optional Follow-Up

- Ask 1–3 targeted probing questions to test the learner's mental model rather than assuming understanding.
- When adding or updating an entry in `BACKEND_KNOWLEDGE_INDEX.md`:
  - Use only the three official depth levels (`Awareness`, `Working Understanding`, `Practical Competence`).
  - Clearly distinguish researched or AI-generated content from learner-verified understanding (only mark an entry learner-verified / `completed` after the learner has reviewed and confirmed their understanding).
  - Do not mark an entry `approved` without explicit ChatGPT sign-off.
