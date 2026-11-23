# Program & Repository Decisions

This file records durable structural, workflow, and tooling decisions for the repository. It does **not** override `MASTER_ROADMAP.md`.

---

## DEC-001: Foundational Repository & Governance Setup (2026-11-23)

- **Status:** Accepted
- **Decisions:**
  1. **Single root instruction file:** Use `AGENTS.md` as the sole root Antigravity instruction file; do not create a duplicate `GEMINI.md`.
  2. **Modular supporting rules:** Store focused rules in `.agents/rules/` (`learning-standards.md`, `research-standards.md`, `github-safety.md`) and keep detailed procedures in `.agents/skills/`.
  3. **Root-level curriculum & continuity files:** Place `MASTER_ROADMAP.md`, `PROGRESS.md`, `DECISIONS.md`, `LEARNING_PHILOSOPHY.md`, `BACKEND_KNOWLEDGE_INDEX.md`, and `NEXT_SESSION.md` at the repository root for immediate visibility.
  4. **Authority separation:** ChatGPT owns curriculum, mastery assessment, and final phase approval; Antigravity acts as Research & Engineering Partner; GitHub is the source of truth.
  5. **Just-in-time phase folders:** Create only `phases/phase-01/`. Do not pre-create folders or materials for Phases 2–29.
  6. **Progress integrity:** Track four explicit states (`planned`, `in progress`, `completed`, `approved`). Agent-generated materials never count as completed learning.
  7. **Git safety:** Inspect before modifying, never commit secrets, never overwrite learner work, and never push unreviewed changes directly to `main`.

---

## DEC-002: Three-Level Knowledge-Depth Framework (2026-11-23)

- **Status:** Accepted
- **Decision:** Adopt the approved three-level knowledge-depth framework across `BACKEND_KNOWLEDGE_INDEX.md`, `.agents/rules/learning-standards.md`, and `.agents/skills/mechanism-deep-dive/SKILL.md`:
  1. **Awareness**
  2. **Working Understanding**
  3. **Practical Competence**
- **Clarification:** Mechanism-level depth and production-level understanding are dimensions applied across these three levels, not additional mandatory levels.
