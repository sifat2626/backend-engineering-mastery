---
name: mechanism-deep-dive
description: >-
  Use this skill when the learner wants a deep, mechanism-level exploration of a
  backend concept, protocol, OS primitive, runtime behavior, or database internal
  within the active phase. Enforces prerequisite-aware, discovery-first prose
  progressing from intuition into mechanisms, invariants, failure modes,
  ecosystem alternatives, trade-offs, and verification.
---

# Mechanism Deep Dive

Guide a rigorous, mechanism-level investigation inside the active phase so the learner builds a connected mental model rather than memorizing isolated technical facts.

## Authority & Scope Checks

1. Read `PROGRESS.md`, `NEXT_SESSION.md`, and `LEARNING_JOURNEY.md` to confirm the topic belongs to the active phase and block—then inspect relevant learner-authored notes, exercises, and verified lab evidence before treating any prerequisite as mastered. Status tracking provides useful context, but learner evidence determines what can safely be assumed.
2. Do not mark the topic `completed` after generating an explanation; mastery requires learner synthesis and verification.
3. Do not overwrite existing learner notes unless explicitly asked.

## Internal Preparation (Do Not Dump Onto the Learner)

Before writing, classify technical terms internally as **Prerequisite** (must be understood before proceeding), **Supporting** (explain in 1–2 clear, accessible Bangla sentences inline; internals come later), or **Context** (ecosystem purpose only). Plan how each idea follows naturally from the previous one.

## Guided Discovery Flow

Write in connected, natural Bangla instructor prose while keeping original English technical terms (explained clearly in Bangla on first introduction), code, commands, APIs, and authoritative English sources—never Romanized Bangla or stiff literal translations (use tables/lists only when they genuinely clarify)—moving naturally through:

1. **Concrete Situation & Problem:** Open with a real scenario where naive assumptions fail, weaving in any missing **Prerequisite** concepts right where needed.
2. **Intuition Before Formalism:** Build a clear mental model with a simple example, then name the formal concept once the underlying idea is clear.
3. **Step-by-Step Mechanism & Cause-and-Effect:** Unpack how it works under the hood (data structures, memory, OS syscalls, wire framing, scheduling, concurrency control, or storage mechanics) with primary sources or version-specific details where relevant.
4. **Invariants, Limits & Production Failure Modes:** Explain what must hold, where naive mental models break, and how real load, contention, partial failure, or misconfiguration manifests.
5. **Alternatives & Trade-Offs:** Compare honestly against broader backend ecosystem alternatives (highlighting Node.js / PostgreSQL context where applicable).
6. **Forward Connection & Independent Verification:** Show how this reframes the problem and what unanswered question comes next, then propose a minimal experiment, query, trace, or primary-source check—plus 1–3 probing questions.

## Quality Gate & Index Hygiene

- **Read-aloud check:** Ensure every transition has an intellectual reason and no unexplained terms interrupt flow.
- When adding or updating `BACKEND_KNOWLEDGE_INDEX.md`, populate its relational columns (`Problem Solved & Definition`, `Prerequisites`, `Enables / Connects To`), use only the three official depth levels (`Awareness`, `Working Understanding`, `Practical Competence`), distinguish researched content from learner-verified understanding, and never mark `approved` without ChatGPT sign-off.
