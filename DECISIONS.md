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

---

## DEC-003: Connected Engineering Journey & Guided Discovery Teaching Model (2026-11-24)

- **Status:** Accepted
- **Decisions:**
  1. **Connected 29-phase journey:** Add high-level `LEARNING_JOURNEY.md` explaining phase relationships, mental models, longitudinal concept dependencies, and forward-reference seams—strictly subordinate to `MASTER_ROADMAP.md` with no changes to roadmap order, scope, or hours.
  2. **Hybrid guided-discovery pedagogy:** Teaching materials must actively teach in connected instructor prose (problem-first, prerequisite-aware, progressive technical depth) rather than reading like reference manuals. The 12-step standard acts strictly as an internal quality checklist alongside the four-question Transition Test and Read-Aloud Quality Gate—never as a rigid document template or learner-facing metadata clutter.
  3. **Internal terminology tiers (distinct from DEC-002):** Authoring materials classify terms internally as **Prerequisite**, **Supporting**, or **Context** (`Awareness` remains reserved exclusively as an official DEC-002 knowledge-depth level).
  4. **Recurring example & transfer discipline:** Evolve a promotional-code redemption application across suitable phases without forcing it where native examples are clearer; require novel transfer scenarios in exercises rather than repeating solved walkthroughs.
  5. **Relational knowledge index & skill alignment:** Add `Problem Solved`, `Prerequisites`, and `Enables / Connects To` columns to `BACKEND_KNOWLEDGE_INDEX.md`, align existing `.agents/skills/*`, and add `.agents/skills/teaching-material-design/SKILL.md`.

---

## DEC-004: Teaching-Language Policy — Natural Bangla with English Technical Terminology (2026-11-24)

- **Status:** Accepted
- **Decision:** Standardize learning materials and teaching explanations across all 29 phases on **natural, technically accurate Bangla prose with original English technical terminology** (codified in `LEARNING_PHILOSOPHY.md`, `.agents/rules/learning-standards.md`, `.agents/skills/teaching-material-design/SKILL.md`, and `.agents/skills/mechanism-deep-dive/SKILL.md`):
  1. Main explanations, exercises, and assessment prompts use clear, natural Bangla (never Romanized Bangla or awkward literal translations) without reducing mechanism-level rigor.
  2. Technical terms, code, commands, identifiers, APIs, configuration, appropriate diagram labels, and authoritative English books/documentation titles remain in English, with technical terms explained clearly in Bangla when first introduced.
  3. English engineering writing and documentation practice is retained wherever the curriculum requires it.
  4. Does not alter `MASTER_ROADMAP.md`, approved hours, learning objectives, or curriculum scope.
