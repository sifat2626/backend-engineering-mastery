# Phase 1 Research Dossier — Engineering Mindset & AI-Augmented Engineering

> **Document Type:** Research & Preparation Dossier (Pre-Material Stage — **not** final learning materials)
> **Phase Authority:** ChatGPT (Curriculum Owner & Instructor) — **26 focused learner-study hours total** (block targets are estimates, not hard deadlines; mastery takes priority)
> **Continuity Context:** Phase 1 Diagnostic is `completed` (must **not** be repeated). The learner has confirmed starting Phase 1 learning **from scratch** at Block 1 (*"What Is Software Engineering?"*), beginning from its first concept with no prior Block 1 learning content assumed completed. Phase 1 is `not approved`.
> **Knowledge-Depth Model:** Uses the repository's three official knowledge levels (`Awareness`, `Working Understanding`, `Practical Competence`). Mechanism-level depth and production relevance are dimensions applied across these levels, not additional levels.

---

## 1. Research Methodology

### Selection Criteria
1. **Primary & Authoritative Sources First:** Prioritized international standards (ISO/IEC/IEEE, IETF RFCs, NIST, OWASP, ACM/IEEE-CS), peer-reviewed empirical papers (ICSE, IEEE Computer, IEEE Software, IEEE TDSC, ACM CCS, USENIX Security), and foundational engineering texts/guides from experienced practitioners (Google SWE/SRE, Ousterhout, Kleppmann, Brooks, Richards & Ford, Gregg, Wiegers, Hickey, Fowler, Nygard).
2. **Free Accessibility First for Mandatory Resources:** Every **Mandatory** resource in this dossier is freely accessible online without a paywall. Commercial books and paywalled standards are classified as **Supplementary** or **Reference**, ensuring no learner is blocked by purchase requirements while still pointing to high-value deep-dive texts.
3. **Enduring Foundations Paired with Current Empirical Evidence:** Classic works (Dijkstra 1969/1970, Brooks 1987, Avižienis et al. 2004, Hickey 2011) establish durable invariants; current standards and empirical studies (ISO/IEC 25010:2023, SWEBOK V4.0 2024, NIST SP 800-218A 2024, DORA 2024/2025, OWASP LLM Top 10 2025, Spracklen et al. USENIX Security 2025, Kleppmann & Riccomini 2026) ensure modern relevance—especially for AI-augmented engineering.
4. **Curated Workload Fit:** Reading volume is intentionally bounded to fit realistically within the **26-hour** learner budget (respecting the one-primary-topic-at-a-time study discipline and leaving substantial time for reasoning, exercises, and the Block 8 laboratory).
5. **Scope Discipline:** Examples connect to backend engineering (and Node.js / PostgreSQL contexts where natural) without prematurely teaching deep technical subjects assigned to later phases (e.g., OS internals in Phase 3, networking in Phase 4, JS internals in Phase 5, DB internals in Phase 10, backend security exploits in Phase 19, or advanced AI/ML engineering reserved for the post-Phase 29 program).
6. **Strict Non-Fabrication & Verification Policy:** All URLs, DOIs, publication years, editions, standard titles, and verified chapter/section breakdowns were checked via live search on `2026-11-23`. Every detail that could not be directly verified (such as exact chapter/page numbers of closed books or numeric ISO catalog IDs) is explicitly labeled `[Unverified]`.

---

## 2. Phase 1 Block Overview (Total: 26 Hours)

*These block targets are estimates, not hard deadlines; mastery takes priority. The authoritative Phase 1 allocation remains **26 focused learner-study hours**.*

| Block | Title | Target Estimate | Primary Target Level | Core Mastery Outcomes Supported |
| :---: | :--- | :---: | :--- | :--- |
| 1 | What Is Software Engineering? | 3 hours | Working Understanding | 1, 10 |
| 2 | Understanding the Problem Before Coding | 3 hours | Practical Competence | 2, 3, 5 |
| 3 | Correctness and Software Quality | 3.5 hours | Working Understanding | 4, 7 |
| 4 | Engineering Constraints and Trade-offs | 3.5 hours | Practical Competence | 5, 6, 10 |
| 5 | Evidence-Based Engineering | 2.5 hours | Working Understanding | 7 |
| 6 | Simplicity, Complexity and Maintainability | 2.5 hours | Working Understanding | 4, 6, 8 |
| 7 | AI-Augmented Engineering | 4 hours | Practical Competence | 7, 9, 10 |
| 8 | Engineering Ownership and Practical Laboratory | 4 hours | Practical Competence | 1–10 (Integrative) |
| **Total** | **8 Research-Dossier Blocks** | **26 hours** | — | **All 10 Outcomes** |

---

## 3. Block-by-Block Research Breakdown

---

### Block 1: What Is Software Engineering? (3 hours)

*Starting from scratch at the first concept (Diagnostic already completed; do not repeat).*

#### Learning Objectives
- **LO 1.1 (`Working Understanding`):** Distinguish clearly between **programming** (producing executable code for a task), **software development** (building and shipping software products/features), and **software engineering** (applying systematic, disciplined, quantifiable engineering practices to design, evolve, and sustain software across time, scale, and competing constraints). *Note: This three-way distinction is a **course working definition** grounded in Winters et al. and SWEBOK to sharpen engineering judgment, rather than a single universally standardized industry taxonomy.*
- **LO 1.2 (`Working Understanding`):** Explain the three core axes separating programming from software engineering: **time and change**, **scale and efficiency**, and **trade-offs and costs** (production & organizational dimension).
- **LO 1.3 (`Working Understanding`):** Explain **Hyrum's Law** and **shifting left** as concrete mechanisms explaining why software behaves differently over long lifespans and large user bases. *(Teaching note: introduce at awareness level during reading, then build to working understanding via the Block 1 scenario exercise.)*
- **LO 1.4 (`Awareness`):** Understand the breadth of software engineering as a formal discipline (as organized in SWEBOK V4.0) and why engineering judgment cannot be reduced to writing syntax.

#### Terminology Inventory
| Term | Target Level | Why It Matters in Phase 1 |
| :--- | :--- | :--- |
| Programming vs. Software Development vs. Software Engineering *(course working distinction)* | `Working Understanding` | Establishes a practical mental boundary between producing working code and owning a durable production system (note: not a single universally standardized ISO taxonomy). |
| Software Lifecycle & Lifespan | `Working Understanding` | Short-lived scripts and multi-year backend services require fundamentally different engineering practices. |
| Software Sustainability *(Winters et al. working definition)* | `Working Understanding` | The capability of a codebase/organization to safely react to every valuable technical or business change needed over its expected lifespan (distinct from environmental sustainability). |
| Hyrum's Law | `Working Understanding` | Given enough users of an API, every observable behavior—even undocumented quirks—will be depended upon. |
| Shifting Left | `Working Understanding` | Detecting defects and design flaws earlier in the lifecycle drastically reduces cost and blast radius. |
| Technical Scope vs. System Boundaries | `Awareness` | Recognizes that code operates inside organizational, operational, and economic environments. |

#### Authoritative Resources & Reading Instructions

| Priority | Access | Resource | Reading Scope (Verified vs. Unverified) | Why Useful & Objectives Supported |
| :--- | :---: | :--- | :--- | :--- |
| **Mandatory** | Free | Titus Winters, Tom Manshreck, Hyrum Wright, *Software Engineering at Google* (O'Reilly, 2020; free HTML edition): [Chapter 1 — What Is Software Engineering?](https://abseil.io/resources/swe-book/html/ch01.html) | **Verified sections:** Full Chapter 1 — *Time and Change* (incl. Hyrum's Law), *Scale and Efficiency* (incl. Shifting Left), *Trade-offs and Costs*, *Software Engineering Versus Programming*, *Conclusion*, and *TL;DRs*. | Clear, authoritative modern articulation of "software engineering is programming integrated over time." Directly supports LO 1.1–1.3 and Outcome 1. |
| **Supplementary** | Free mirror / Paywalled IEEE | Mary Shaw, *"Prospects for an Engineering Discipline of Software"*, *IEEE Software*, Vol. 7, No. 6, pp. 15–24 (Nov 1990): [CMU Figshare](https://doi.org/10.1184/R1/6608717) / [IEEE DOI 10.1109/52.60586](https://doi.org/10.1109/52.60586) | **Verified:** Journal, Vol. 7, No. 6, pp. 15–24, Nov 1990. **Unverified:** Internal subsection headings `[Unverified — full PDF not directly inspected]`. | Grounds "engineering" historically by comparing software against mature engineering disciplines. Supports LO 1.1, LO 1.4. |
| **Reference** | Free | IEEE Computer Society, *Guide to the Software Engineering Body of Knowledge (SWEBOK V4.0)* (2024): [Official SWEBOK Portal](https://www.computer.org/education/bodies-of-knowledge/software-engineering-swebok) | **Verified:** Skim the Introduction and the list of **18 Knowledge Areas** (noting V4 additions: Software Architecture, Software Engineering Operations, Software Security). Do not read all 18 KAs in Phase 1. | Shows that coding (*Software Construction*) is only 1 of 18 knowledge areas in software engineering. Supports LO 1.4. |

#### Misconceptions, Common Mistakes & Key Distinctions
- **Misconception:** *"Software engineering is just writing cleaner code or knowing more frameworks."* → **Correction:** Programming produces a working artifact at a point in time; software development ships product features; software engineering governs sustainability, verification, trade-offs, and safe evolution over time and scale.
- **Misconception:** *"Every script needs full software engineering rigor."* → **Correction:** Applying heavy engineering overhead to a 20-line throwaway script is poor engineering. Judgment means matching rigor to lifespan, blast radius, and scale.
- **Key Distinction:** **Working now** vs. **modifiable and operable two years from now by another engineer**.

#### Realistic Scenarios & Practical Exercises
1. **Lifespan & Hyrum's Law Analysis:** A Node.js backend endpoint accidentally returns items ordered by insertion without documenting any sort guarantee. Two years later, changing the PostgreSQL query breaks three downstream clients. Analyze how Hyrum's Law applies and contrast a programmer's response with an engineer's response.
2. **Rigor Classification Exercise:** Given four backend artifacts (a one-off CSV cleanup script, an internal admin cron job, a public billing API, and an authentication service), determine appropriate engineering rigor for each and justify differences using time, scale, and cost axes.

#### What the Learner Should Be Able to Explain, Analyze, or Demonstrate
- Explain in their own words the operational differences between programming, software development, and software engineering.
- Analyze a realistic backend change across time, scale, and cost, identifying a Hyrum's Law risk and a shift-left opportunity.

---

### Block 2: Understanding the Problem Before Coding (3 hours)

#### Learning Objectives
- **LO 2.1 (`Working Understanding`):** Distinguish the **problem space** (user/business need, domain constraints, failure consequences) from the **solution space** (tools, frameworks, schemas, code structures).
- **LO 2.2 (`Practical Competence`):** Decompose an ambiguous or under-specified engineering request to surface **hidden assumptions**, **ambiguities**, **missing requirements**, **edge cases**, and **non-goals** before writing code.
- **LO 2.3 (`Working Understanding`):** Apply standard criteria for well-formed requirements (especially **unambiguous**, **complete**, **singular**, **feasible**, and **verifiable**) and normative specification language (`MUST`, `SHOULD`, `MAY`).
- **LO 2.4 (`Working Understanding`):** Explain why lightweight written problem framing (e.g., concise design docs) catches expensive misunderstandings before implementation begins.

#### Terminology Inventory
| Term | Target Level | Why It Matters in Phase 1 |
| :--- | :--- | :--- |
| Problem Space vs. Solution Space | `Working Understanding` | Prevents premature technology choices ("let's add Kafka/Redis") before understanding the problem. |
| Requirement (Functional vs. Non-Functional) | `Working Understanding` | Separates *what* the system must do from *how well* it must behave under constraints. |
| Assumption vs. Fact vs. Dependency | `Practical Competence` | Unvalidated assumptions are latent failure triggers; engineers make them explicit and testable. |
| Ambiguity (Reader-Side vs. Stakeholder-Side) | `Working Understanding` | Stakeholder-side ambiguity (two people reading the same words differently) is the deadliest requirements defect. |
| Non-Goals / Explicit Scope Boundaries | `Practical Competence` | Defining what a system will *not* solve prevents scope creep and hidden architectural drift. |
| Verifiability (Acceptance Criteria) | `Practical Competence` | A requirement that cannot be objectively tested or inspected cannot be proved satisfied. |
| Normative Keywords (`MUST` / `SHOULD` / `MAY`) | `Working Understanding` | Eliminates hedging words ("normally", "fast", "support") in engineering specs. |

#### Authoritative Resources & Reading Instructions

| Priority | Access | Resource | Reading Scope (Verified vs. Unverified) | Why Useful & Objectives Supported |
| :--- | :---: | :--- | :--- | :--- |
| **Mandatory** | Free | IETF RFC 2119 (S. Bradner, March 1997): [Key words for use in RFCs to Indicate Requirement Levels](https://datatracker.ietf.org/doc/html/rfc2119) | **Verified:** Full short RFC (~2 pages, Sections 1–6). **Read together with RFC 8174 below.** | Teaches precision in specifying mandatory vs. optional behavior; eliminates vague wording. Supports LO 2.3. |
| **Mandatory** | Free | IETF RFC 8174 (B. Leiba, May 2017): [Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words](https://datatracker.ietf.org/doc/rfc8174/) | **Verified:** Full very short RFC (~2 pages) updating RFC 2119 to clarify that **only all-uppercase forms** (`MUST`, `SHOULD`, `MAY`, etc.) invoke the special **BCP 14** normative meanings, while **lowercase forms retain their ordinary English meanings**. **Read together with RFC 2119.** | Prevents accidental ambiguity when writing requirements in prose alongside RFC 2119. Supports LO 2.3. |
| **Mandatory** | Free | Malte Ubl, *"Design Docs at Google"* (2020): [industrialempathy.com/posts/design-docs-at-google](https://www.industrialempathy.com/posts/design-docs-at-google/) | **Verified:** Full article — focus on *Context and Scope*, *Goals and Non-Goals*, *When Not to Write a Design Doc*, and *Lifecycle of a Design Doc*. | Real-world production blueprint for framing problems, constraints, and non-goals before coding. Supports LO 2.1, LO 2.2, LO 2.4. |
| **Reference** | Paywalled standard | ISO/IEC/IEEE 29148:2018, *Systems and software engineering — Life cycle processes — Requirements engineering*: [ISO Standard 72089](https://www.iso.org/standard/72089.html) | **Verified:** Official catalog URL and the **9 characteristics of individual well-formed requirements** (Necessary, Appropriate, Unambiguous, Complete, Singular, Feasible, Verifiable, Correct, Conforming). **Unverified:** Internal clause numbers `[Unverified — full PDF behind ISO paywall]`. | Authoritative international baseline (replaces IEEE 830); not required to purchase since the 9 criteria and RFC 2119/8174 are covered cleanly in open materials. Supports LO 2.3. |
| **Supplementary** | Commercial book | Karl Wiegers & Joy Beatty, *Software Requirements* (3rd ed., Microsoft Press, 2013) or Karl Wiegers & Candase Hokanson, *Software Requirements Essentials* (2023) | **Verified concepts:** Reader-side vs. stakeholder-side ambiguity, intrinsically ambiguous words, Vision & Scope assumptions/dependencies. **Unverified:** Exact chapter/page numbers `[Unverified — table of contents not directly inspected]`. | Practical catalog of requirement failure patterns for learners who own or borrow the book. Supports LO 2.2, LO 2.3. |

#### Misconceptions, Common Mistakes & Key Distinctions
- **Misconception:** *"Requirements gathering is a Product Manager's job; engineers just implement tickets."* → **Correction:** Product tickets rarely specify concurrency semantics, idempotency, failure handling, latency bounds, or data retention; engineers own uncovering technical ambiguities and edge cases.
- **Misconception:** *"Starting to code immediately is faster."* → **Correction:** Coding against an ambiguous mental model produces fast initial output followed by expensive rework, schema migrations, or broken API contracts.
- **Key Distinction:** **Stakeholder solution proposal** ("add a cron job to send emails") vs. **underlying problem & invariant** ("customers must receive payment receipts reliably once within 5 minutes of payment confirmation").

#### Realistic Scenarios & Practical Exercises
1. **Ambiguous Ticket Interrogation:** Given a realistic vague backend request — *"Build an API endpoint so users can export their order history quickly and securely"* — produce: (a) 8+ specific clarifying questions grouped by behavior, data bounds, security/authorization, and failure/performance constraints; (b) an explicit list of hidden assumptions; (c) explicit non-goals; and (d) 5 rewritten, verifiable acceptance criteria using RFC 2119 keywords.
2. **Problem vs. Solution Refactoring:** Take a ticket phrased purely as a solution (*"Cache user sessions in Redis for 24 hours"*) and reconstruct the underlying problem statement, constraints, and questions that must be answered before choosing any mechanism.

#### What the Learner Should Be Able to Explain, Analyze, or Demonstrate
- Analyze a realistic, under-specified backend request and systematically surface ambiguities, hidden assumptions, missing error/boundary behaviors, and non-goals before proposing code.
- Rewrite vague adjectives (*"fast"*, *"secure"*, *"reliable"*) into verifiable, measurable requirements.

---

### Block 3: Correctness and Software Quality (3.5 hours)

#### Learning Objectives
- **LO 3.1 (`Working Understanding`):** Define **correctness** rigorously in terms of conformance to a specification, **preconditions**, **postconditions**, and **invariants**, explaining why "it works on my machine for the happy path" is not correctness.
- **LO 3.2 (`Working Understanding`):** Explain the **Fault → Error → Failure** causal chain (Avižienis et al. 2004) and the four means of achieving dependability (fault prevention, fault tolerance, fault removal, fault forecasting).
- **LO 3.3 (`Working Understanding`):** Explain **verification** ("building the system right") vs. **validation** ("building the right system") and why testing proves the *presence* of bugs, never their *absence* (Dijkstra).
- **LO 3.4 (`Working Understanding`):** Use the **ISO/IEC 25010:2023** nine-characteristic quality model to articulate multi-dimensional software quality and identify tensions between quality attributes (e.g., security vs. performance efficiency, reliability vs. simplicity).

#### Terminology Inventory
| Term | Target Level | Why It Matters in Phase 1 |
| :--- | :--- | :--- |
| Correctness (Partial vs. Total; Spec Conformance) | `Working Understanding` | Correctness is meaningless without a specification of expected behavior and termination/liveness expectations. |
| Precondition, Postcondition, Invariant | `Working Understanding` | Core mechanism-level vocabulary for reasoning about code, APIs, and database state integrity. |
| Fault, Error, Failure | `Working Understanding` | Precise causal chain separating a latent defect (fault), invalid internal state (error), and external spec violation (failure). |
| Verification vs. Validation | `Working Understanding` | Prevents the common failure of rigorously verifying a system that solves the wrong problem. |
| Quality Attributes (ISO/IEC 25010:2023) | `Working Understanding` | Provides a standardized, current vocabulary (9 characteristics, including Safety, Interaction Capability, Flexibility) instead of vague "high quality." |
| Internal Quality vs. External Quality | `Working Understanding` | Users observe external quality (reliability, latency); engineers pay for poor internal quality (maintainability) over time. |

#### Authoritative Resources & Reading Instructions

| Priority | Access | Resource | Reading Scope (Verified vs. Unverified) | Why Useful & Objectives Supported |
| :--- | :---: | :--- | :--- | :--- |
| **Mandatory** | Free (author copy) | Algirdas Avižienis, Jean-Claude Laprie, Brian Randell, Carl E. Landwehr, *"Basic Concepts and Taxonomy of Dependable and Secure Computing"*, *IEEE TDSC*, Vol. 1, No. 1, pp. 11–33 (Jan–Mar 2004): [Author-Hosted PDF (Carl Landwehr)](https://www.landwehr.org/cs690/readings/avizienis04basic.pdf) / [IEEE DOI 10.1109/TDSC.2004.2](https://doi.org/10.1109/TDSC.2004.2) | **Verified:** Focus on the core definitions of **Fault, Error, Failure** (and their causal chain), the overview of dependability **Attributes** and **Means** (Fault Prevention, Tolerance, Removal, Forecasting). Skip deep hardware fault-tolerance taxonomy subsections in Phase 1. **Unverified:** Exact internal subsection numbering of the PDF `[Unverified — not directly paged through]`. | Gold-standard precision on faults vs. errors vs. failures and how dependability is engineered. Supports LO 3.2. |
| **Mandatory** | Free | arc42 Quality Model — [ISO/IEC 25010:2023 Product Quality Model Overview](https://quality.arc42.org/iso-25010/) | **Verified:** Study the **9 top-level quality characteristics** in the 2023 revision: Functional Suitability, Performance Efficiency, Compatibility, Interaction Capability *(formerly Usability)*, Reliability, Security, Maintainability, Flexibility *(formerly Portability; includes Scalability)*, and Safety *(new in 2023)*. | Freely accessible, authoritative summary of the current 2023 ISO/IEC 25010 quality model without requiring purchase of the paywalled ISO PDF. Supports LO 3.4. |
| **Mandatory** | Free | Edsger W. Dijkstra, *Notes on Structured Programming* (EWD249, 1969/1970): [UT Austin PDF](https://www.cs.utexas.edu/users/EWD/ewd02xx/EWD249.PDF) / [HTML Transcription](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD02xx/EWD249.html) | **Verified:** Read Section 3 (*"On the Reliability of Mechanisms"*) for why testing shows the presence—never the absence—of bugs and why structural reasoning about correctness matters. | Prevents over-reliance on shallow "passing tests = correct software" thinking. Supports LO 3.1, LO 3.3. |
| **Supplementary** | Commercial book | Martin Kleppmann, *Designing Data-Intensive Applications* (1st ed., O'Reilly, March 2017; or 2nd ed. with Chris Riccomini, O'Reilly, Feb 2026) | **Verified:** 1st ed. Chapter 1 (*"Reliable, Scalable, and Maintainable Applications"*, sections *Reliability* and *Maintainability*). 2nd ed. early chapters (*Trade-Offs in Data Systems Architecture* and *Defining Nonfunctional Requirements*). **Unverified:** Exact 2nd ed. chapter/page numbers `[Unverified]`. | Connects quality attributes directly to backend data systems (hardware faults, software faults, human error). Supports LO 3.2, LO 3.4. |
| **Reference** | Paywalled standard | ISO/IEC 25010:2023, *Systems and software engineering — SQuaRE — Product quality model* (2nd ed., Nov 2023; [ISO Catalog](https://www.iso.org/standard/)) | **Unverified:** Exact numeric `iso.org/standard/<id>` URL `[Unverified — behind ISO paywall]`. | Official normative standard behind the free arc42 overview above. Supports LO 3.4. |

#### Misconceptions, Common Mistakes & Key Distinctions
- **Misconception:** *"If all unit tests pass and there are no syntax errors, the software is correct."* → **Correction:** Tests only sample finite inputs; if invariants, concurrency cases, or requirements are wrong, 100% green tests still ship broken software.
- **Misconception:** *"Code cleanliness equals software quality."* → **Correction:** Clean code is only one facet of *Maintainability*. Clean code that violates data invariants, leaks PII, or crashes under partial failure is low-quality software.
- **Key Distinction:** **Fault** (a missing null check or wrong SQL `WHERE` clause in code) vs. **Error** (an incorrect balance stored in memory/DB) vs. **Failure** (double-charging a customer at the API boundary).

#### Realistic Scenarios & Practical Exercises
1. **Fault → Error → Failure Trace:** Trace a realistic backend defect (e.g., an unhandled floating-point rounding bug in currency calculation, or a missing uniqueness constraint on an idempotency key) through: (a) dormant fault, (b) activation condition, (c) internal error state, (d) observable service failure, and (e) which dependability means (prevention, removal, tolerance) could have intercepted each step.
2. **Quality Attribute Tension Matrix:** For an order-processing API, pick 4 characteristics from ISO/IEC 25010:2023 (e.g., Functional Suitability, Performance Efficiency, Security, Maintainability) and describe one concrete design decision where improving one degrades another.

#### What the Learner Should Be Able to Explain, Analyze, or Demonstrate
- Define correctness using specification conformance and invariants, and explain why passing tests is necessary but insufficient.
- Diagnose a concrete backend bug using the Fault → Error → Failure chain and map quality requirements to ISO/IEC 25010:2023 characteristics.

---

### Block 4: Engineering Constraints and Trade-offs (3.5 hours)

#### Learning Objectives
- **LO 4.1 (`Working Understanding`):** Identify and classify real-world **engineering constraints** across technical, resource/cost, schedule, team/organizational, regulatory/compliance, and operational dimensions.
- **LO 4.2 (`Working Understanding`):** Apply Mark Richards and Neal Ford's **First Law of Software Architecture** (*"Everything in software architecture is a trade-off"*) and its corollary (seeking the *"least worst"* design for the specific context rather than universal "best practices").
- **LO 4.3 (`Practical Competence`):** Compare at least two viable technical alternatives systematically across explicit criteria, surfacing what is gained, what is sacrificed, reversibility (one-way vs. two-way doors), and failure modes.
- **LO 4.4 (`Practical Competence`):** Write a structured **Architecture Decision Record (ADR)** capturing Context, Decision, Alternatives Considered, Consequences, and Status.

#### Terminology Inventory
| Term | Target Level | Why It Matters in Phase 1 |
| :--- | :--- | :--- |
| Engineering Constraint (Hard vs. Soft) | `Working Understanding` | Constraints shape the feasible solution space; ignoring them yields theoretically "elegant" but unshippable systems. |
| Trade-off & Opportunity Cost | `Working Understanding` | Every chosen option forecloses other options and spends finite engineering, compute, and complexity budgets. |
| Error Budget / Risk Tolerance | `Awareness` | Illustrates why "100% reliability" is almost always the wrong target due to cost and velocity trade-offs; detailed SRE error-budget engineering is reserved for **Phase 21 (Reliability Engineering)**. |
| Architecture Decision Record (ADR) | `Practical Competence` | Lightweight, version-controlled artifact preserving *why* a trade-off was made under specific historical constraints. |
| Reversibility (One-Way vs. Two-Way Door) | `Working Understanding` | Determines how much upfront analysis and evidence a decision justifies. |
| Context-Dependent Fit ("Least Worst", Richards & Ford) | `Working Understanding` | Counteracts cargo-culting "best practices" copied from hyperscalers into small teams. |

#### Authoritative Resources & Reading Instructions

| Priority | Access | Resource | Reading Scope (Verified vs. Unverified) | Why Useful & Objectives Supported |
| :--- | :---: | :--- | :--- | :--- |
| **Mandatory** | Free | Michael Nygard, *"Documenting Architecture Decisions"* (Cognitect Blog, Nov 15, 2011): [cognitect.com/blog/2011/11/15/documenting-architecture-decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) | **Verified:** Full short article (Context, Decision, Status, Consequences, immutable history). | Original specification of ADRs—directly teaches how engineers record constraints, trade-offs, and consequences. Supports LO 4.4. |
| **Mandatory** | Free | Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy (eds.), *Site Reliability Engineering* (O'Reilly, 2016): [Chapter 3 — Embracing Risk](https://sre.google/sre-book/embracing-risk/) | **Verified (Awareness scope):** Read *Managing Risk* and the conceptual framing in *Risk Tolerance of Services* & *Motivation for Error Budgets* (~15–20 min). Defer detailed SLI/SLO/error-budget calculations and SRE policies to **Phase 21**. | Concrete, production-grounded illustration of why 100% reliability is a bad constraint trade-off. Supports LO 4.1, LO 4.2 (`Awareness` for error budgets). |
| **Mandatory** | Free | Titus Winters et al., *Software Engineering at Google* (2020): [Chapter 1 — Trade-offs and Costs](https://abseil.io/resources/swe-book/html/ch01.html) | **Verified sections:** *Trade-offs and Costs*, *Inputs to Decision Making*, and *Revisiting Decisions, Making Mistakes*. | Catalogs real engineering cost types (financial, resource, personnel, transaction, opportunity, societal) and emphasizes revisiting decisions when inputs change. Supports LO 4.1–4.3. |
| **Supplementary** | Commercial book | Mark Richards & Neal Ford, *Fundamentals of Software Architecture* (2nd ed., O'Reilly, 2025; 1st ed. 2020) | **Verified concepts:** Origin of the **First Law of Software Architecture** (*"Everything in software architecture is a trade-off"*) and *"least worst architecture"*. **Unverified:** Exact chapter/page numbers in 2nd ed. 2025 `[Unverified]`. | Authoritative source of the First Law and architectural trade-off thinking for learners who own or borrow the book. Supports LO 4.2, LO 4.3. |

#### Misconceptions, Common Mistakes & Key Distinctions
- **Misconception:** *"Good engineering means always picking industry 'best practices' (e.g., microservices, 99.999% uptime, zero tech debt)."* → **Correction:** Every "best practice" carries operational and cognitive costs; without matching constraints, it becomes destructive overengineering.
- **Misconception:** *"A trade-off analysis is a pros/cons list where our favorite option has all the pros."* → **Correction:** Honest trade-off analysis explicitly names the concrete downsides of the *chosen* option and the scenarios under which that choice would fail or need reversal.
- **Key Distinction:** **Decision quality** (sound reasoning given constraints and evidence available at the time) vs. **outcome luck**.

#### Realistic Scenarios & Practical Exercises
1. **Constraint-Driven ADR Exercise:** A 2-engineer startup is launching a SaaS backend expected to serve 500 daily active users for the next 12 months with a 6-week launch constraint. Compare Option A (single Node.js service + single PostgreSQL database) vs. Option B (4 microservices + separate DBs + message broker). Write a complete 1-page ADR explicitly listing constraints, trade-offs of *both* options, failure modes, and trigger conditions under which you would revisit the decision.
2. **Reliability Cost Exercise:** Using Google SRE's *Embracing Risk* framing, explain why targeting 99.999% availability (~5.26 minutes downtime/year) for an internal weekly reporting tool harms the business compared to targeting 99.5%.

#### What the Learner Should Be Able to Explain, Analyze, or Demonstrate
- Identify explicit and hidden constraints in an engineering scenario.
- Compare competing alternatives objectively, articulate what is sacrificed by the chosen option, and write a clear ADR that preserves decision rationale for future engineers.

---

### Block 5: Evidence-Based Engineering (2.5 hours)

#### Learning Objectives
- **LO 5.1 (`Working Understanding`):** Distinguish **empirical evidence** (measurements, reproducible experiments, production telemetry, peer-reviewed/systematic studies) from **opinion, authority bias, hype, and anecdote**.
- **LO 5.2 (`Working Understanding`):** Apply the **scientific method** to engineering diagnosis and decision-making (Question → Hypothesis → Prediction → Test → Analysis) and recognize diagnostic **anti-methods** (Blame-Someone-Else, Streetlight, Drunk Man, Random Change, Passive Benchmarking, Traffic Light).
- **LO 5.3 (`Practical Competence`):** Recognize common evidence traps: confounded benchmarks, average vs. tail metrics (e.g., mean vs. p99 conceptually), survivorship bias, correlation vs. causation, and post-hoc rationalization.
- **LO 5.4 (`Working Understanding`):** Explain how **blameless postmortems** use factual timelines and contributing factors (not blame or opinion) to improve systems after failures.

#### Terminology Inventory
| Term | Target Level | Why It Matters in Phase 1 |
| :--- | :--- | :--- |
| Evidence-Based Software Engineering (EBSE) | `Awareness` | Grounds engineering choices in systematic empirical data rather than vendor claims or folklore. |
| Falsifiable Hypothesis | `Practical Competence` | A claim that cannot be disproved by an experiment or measurement is opinion, not engineering analysis. |
| Diagnostic Anti-Methods (Streetlight, Drunk Man, Random Change, Blame-Someone-Else) | `Working Understanding` | Names common failure habits of looking only at familiar tools or tweaking knobs blindly without measurement. |
| Baseline & Controlled Measurement | `Practical Competence` | You cannot prove an optimization or fix worked without measuring before and after under comparable conditions. |
| Correlation vs. Causation | `Working Understanding` | Essential for interpreting metrics, incidents, and empirical studies (including AI productivity claims). |
| Blameless Postmortem | `Working Understanding` | Institutionalizes evidence-based learning from failure rather than hiding mistakes. |

#### Authoritative Resources & Reading Instructions

| Priority | Access | Resource | Reading Scope (Verified vs. Unverified) | Why Useful & Objectives Supported |
| :--- | :---: | :--- | :--- | :--- |
| **Mandatory** | Free | Brendan Gregg, [*Performance Analysis Methodology*](https://www.brendangregg.com/methodology.html) (and conceptual intro of [*The USE Method*](https://www.brendangregg.com/usemethod.html)) | **Verified:** Read the **Anti-Methodologies** (Blame-Someone-Else, Streetlight, Drunk Man, Random Change, Passive Benchmarking, Traffic Light), the **Scientific Method** (Question → Hypothesis → Prediction → Test → Analysis), and the conceptual summary of **USE** (Utilization, Saturation, Errors) and **Problem Statement / Workload Characterization** methods. Defer Linux CLI checklists (`vmstat`, `iostat`, `sar`) to Phases 3, 23, and 25. | Freely accessible, authoritative blueprint from the creator of the USE Method for hypothesis-driven diagnosis over guesswork—replaces the commercial book as mandatory. Supports LO 5.1–5.3. |
| **Mandatory** | Free | Google SRE Book (2016): [Chapter 15 — Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/) | **Verified:** Full Chapter 15 (*Google's Postmortem Philosophy*, *Collaborate and Share Knowledge*, *Introducing a Postmortem Culture*). | Demonstrates evidence-based incident investigation (factual timelines, systemic contributing factors, verifiable action items). Supports LO 5.1, LO 5.4. |
| **Supplementary** | Commercial book | Brendan Gregg, *Systems Performance: Enterprise and the Cloud* (2nd ed., Addison-Wesley, 2020) | **Verified:** **Chapter 2 — Methodologies** (expanded book treatment of the open pages above). **Unverified:** Exact 2nd ed. subsection/page numbers `[Unverified]`. | Optional deeper book reading for learners who own it. Supports LO 5.1–5.3. |
| **Supplementary** | Paywalled IEEE | Barbara A. Kitchenham, Tore Dybå, Magne Jørgensen, *"Evidence-Based Software Engineering"*, *ICSE 2004*: [DOI 10.1109/ICSE.2004.1317449](https://doi.org/10.1109/ICSE.2004.1317449) | **Verified:** ICSE 2004 foundational paper introducing EBSE (adapting Evidence-Based Medicine to software engineering via systematic literature reviews and empirical appraisal). **Unverified:** Exact page numbers `[Unverified — behind IEEE paywall]`. | Academic foundation for evaluating empirical studies and industry claims. Supports LO 5.1. |
| **Reference** | Free | DORA (DevOps Research and Assessment): [dora.dev](https://dora.dev/) and [2024 Accelerate State of DevOps Report](https://dora.dev/dora-report-2024) | **Verified:** Skim the research methodology overview (how DORA distinguishes statistically validated findings from anecdote). | Concrete example of large-scale empirical software engineering research. Supports LO 5.1, LO 5.3. |

#### Misconceptions, Common Mistakes & Key Distinctions
- **Misconception:** *"A benchmark blog post showing Tool X is 3x faster than Tool Y proves we should switch."* → **Correction:** Synthetic microbenchmarks rarely match production workloads, data shapes, memory constraints, or failure modes; unverified benchmarks are weak evidence.
- **Misconception:** *"Senior engineers can rely on intuition instead of measuring."* → **Correction:** Intuition generates *hypotheses*; evidence confirms or falsifies them. Senior engineers get burned by stale mental models when runtimes, hardware, or workloads change.
- **Key Distinction:** **Observation/Data** ("p95 response time rose from 80 ms to 900 ms at 14:02 UTC after deploy #412") vs. **Hypothesis** ("a query plan regressed due to missing index") vs. **Premature Conclusion** ("Node.js is slow, we should rewrite in Go").

#### Realistic Scenarios & Practical Exercises
1. **Anti-Method Audit:** Read a short simulated Slack incident thread where engineers jump to 4 unverified fixes (restarting pods, increasing connection pool size blindly, blaming the cloud provider, adding caching). Identify which of Gregg's anti-methods are occurring and rewrite the response as a 5-step Scientific Method investigation plan.
2. **Claim Appraisal Exercise:** Take a bold engineering claim (e.g., *"ORM X is always too slow for production"* or *"Switching from REST to gRPC cuts latency by 50%"*). List what exact evidence, workload conditions, and controlled experiment would be required before acting on that claim.

#### What the Learner Should Be Able to Explain, Analyze, or Demonstrate
- Separate raw observations, hypotheses, opinions, and verified evidence in a technical discussion or incident report.
- Design a minimal, falsifiable verification check (baseline, single changed variable, measurable criterion) before committing to an optimization or architectural change.

---

### Block 6: Simplicity, Complexity and Maintainability (2.5 hours)

#### Learning Objectives
- **LO 6.1 (`Working Understanding`):** Distinguish **essential complexity** (inherent to the real-world problem domain) from **accidental complexity** (introduced by our tools, abstractions, and design choices — Brooks 1987).
- **LO 6.2 (`Working Understanding`):** Distinguish **simple** (unbraided, few interleaved concerns — objective structural property) from **easy** (familiar, quick to type initially — relative to the developer's background — Hickey 2011).
- **LO 6.3 (`Working Understanding`):** Identify the three observable **symptoms of complexity** (*change amplification*, *cognitive load*, *unknown unknowns*) and their two root causes (*dependencies* and *obscurity*), and contrast **tactical vs. strategic programming** and **deep vs. shallow modules** (Ousterhout).
- **LO 6.4 (`Working Understanding`):** Reason accurately about **maintainability**, **overengineering**, and **technical debt** (using Martin Fowler's Technical Debt Quadrant: prudent/reckless × deliberate/inadvertent).

#### Terminology Inventory
| Term | Target Level | Why It Matters in Phase 1 |
| :--- | :--- | :--- |
| Essential vs. Accidental Complexity | `Working Understanding` | You cannot eliminate essential complexity; engineering skill lies in refusing to add unnecessary accidental complexity. |
| Simple vs. Easy | `Working Understanding` | "Easy" shortcuts (e.g., global mutable state, magic frameworks, copy-pasted AI code) often create tangled, unmaintainable complexity. |
| Change Amplification, Cognitive Load, Unknown Unknowns | `Working Understanding` | Concrete, diagnosable symptoms that replace subjective arguments about "clean vs. ugly code." |
| Tactical vs. Strategic Programming | `Working Understanding` | Explains how small "just get it working quickly" compromises compound into system paralysis. |
| Deep vs. Shallow Modules | `Working Understanding` | Prevents overengineering via "classitis" / excessive thin wrapper layers that hide no complexity. |
| Overengineering (Speculative Generality) | `Working Understanding` | Building abstractions or distributed machinery for hypothetical future scale that imposes immediate complexity cost today. |
| Technical Debt Quadrant | `Working Understanding` | Separates deliberate, prudent engineering trade-offs from reckless messiness. |

#### Authoritative Resources & Reading Instructions

| Priority | Access | Resource | Reading Scope (Verified vs. Unverified) | Why Useful & Objectives Supported |
| :--- | :---: | :--- | :--- | :--- |
| **Mandatory (Primary Talk)** | Free | Rich Hickey, *"Simple Made Easy"* (Strange Loop Keynote, 2011): [InfoQ Video & Transcript](https://www.infoq.com/presentations/Simple-Made-Easy/) | **Verified (~60 min):** Primary Block 6 talk (watch or read transcript) — focus on definitions of *simple* vs. *easy*, *complecting* (braiding concerns together), and why choosing easy over simple degrades maintainability. | Sharpens precision on why "fast/easy to write now" often destroys long-term simplicity. Supports LO 6.2. |
| **Mandatory (Selected Sections)** | Free | John Ousterhout, *"A Philosophy of Software Design"* (*Talks at Google*, 2018): [YouTube Video](https://www.youtube.com/watch?v=bmSAYlu0NcY) | **Scoped to ~35 min (selected sections only — do not watch full Q&A in Block 6):** Complexity & 3 symptoms (~00:00–12:00), Tactical vs. Strategic Programming (~12:00–22:00), and Deep vs. Shallow Modules (~22:00–38:00). `[Unverified exact timestamps — YouTube video lacks official chapter markers; approximate ranges based on talk flow]`. | Freely accessible primary presentation by the book's author—fits cleanly inside the 2.5-hour block alongside Hickey and Fowler. Supports LO 6.3, LO 6.4. |
| **Mandatory** | Free | Martin Fowler, *"Technical Debt Quadrant"* (Oct 14, 2009): [martinfowler.com/bliki/TechnicalDebtQuadrant.html](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html) | **Verified (~5 min):** Full short bliki entry (2×2 matrix: Reckless/Prudent × Deliberate/Inadvertent). | Stops the misuse of "technical debt" as an excuse for sloppy engineering. Supports LO 6.4. |
| **Supplementary** | Commercial book | John Ousterhout, *A Philosophy of Software Design* (2nd ed., Yaknyam Press, 2021) | **Verified chapters & subsections:** **Ch. 2: The Nature of Complexity** (2.1–2.5: change amplification, cognitive load, unknown unknowns; dependencies & obscurity), **Ch. 3: Working Code Isn't Enough** (3.1–3.5: tactical vs. strategic programming), **Ch. 4: Modules Should Be Deep** (4.1–4.8: deep vs. shallow modules, classitis). | Top optional book recommendation if the learner owns or borrows it; expands on the free Talks at Google presentation. Supports LO 6.3, LO 6.4. |
| **Supplementary** | Paywalled IEEE | Fred Brooks, *"No Silver Bullet — Essence and Accident in Software Engineering"*, *IEEE Computer*, Vol. 20, No. 4, pp. 10–19 (April 1987): [DOI 10.1109/MC.1987.1663532](https://doi.org/10.1109/MC.1987.1663532) | **Verified:** *IEEE Computer*, Vol. 20, No. 4, pp. 10–19, April 1987 (four intrinsic properties: complexity, conformity, changeability, invisibility; essential vs. accidental difficulties). | Seminal origin of essential vs. accidental complexity—explains why no tool (including AI) removes conceptual software difficulty. Listed as Supplementary due to IEEE paywall (its core distinctions are summarized in learning materials). Supports LO 6.1 and bridges into Block 7. |

#### Misconceptions, Common Mistakes & Key Distinctions
- **Misconception:** *"More abstraction layers and design patterns make code more maintainable."* → **Correction:** Shallow pass-through layers increase cognitive load and change amplification without hiding complexity (Ousterhout's "classitis").
- **Misconception:** *"Simple means short, minimal lines of code, or familiar."* → **Correction:** Dense clever one-liners are short, not simple; familiar tools are easy, not necessarily simple. Simplicity means untangled responsibilities with minimal hidden dependencies.
- **Key Distinction:** **Prudent, deliberate technical debt** (a documented shortcut taken under a hard deadline with a known repayment plan) vs. **reckless mess** (skipping design/verification out of haste or indifference).

#### Realistic Scenarios & Practical Exercises
*(Time budget note: Mandatory reading/viewing takes ~100–103 min, leaving ~47–50 min within the 2.5-hour block. Exercise 1 is **Mandatory**; Exercise 2 is a **Short Optional Extension**.)*

1. **Overengineering Critique *(Mandatory — ~30–35 min)*:** Inspect a contrived "enterprise" Node.js CRUD feature split into 7 thin pass-through files (`Controller` → `Facade` → `ServiceInterface` → `ServiceImpl` → `RepositoryInterface` → `RepositoryImpl` → `Mapper`) for a single 3-column PostgreSQL table. Diagnose the specific symptoms of complexity (change amplification, cognitive load, shallow modules) and separate essential from accidental complexity.
2. **Simple vs. Easy Classification *(Optional Extension — ~10–15 min)*:** Compare two approaches to configuration or data validation in a small backend service and classify each along *simple vs. easy*, explaining how each behaves when requirements change 6 months later.

#### What the Learner Should Be Able to Explain, Analyze, or Demonstrate
- Diagnose a piece of code or design using Ousterhout's three complexity symptoms and two causes, distinguishing essential from accidental complexity.
- Spot speculative overengineering and explain why "easy to generate quickly" (including with AI) often increases long-term complexity.

---

### Block 7: AI-Augmented Engineering (4 hours)

> **Scope Guardrail:** Covers the practical mindset, engineering workflows, risks, verification methods, and accountability a modern backend engineer needs when using AI coding tools. Does **not** drift into ML model architecture, training, fine-tuning, or advanced AI engineering (reserved for the post-Phase 29 program).

#### Learning Objectives
- **LO 7.1 (`Practical Competence`):** Decide **what to delegate** to AI (boilerplate, test-case brainstorming, unfamiliar syntax exploration, initial refactors with tight specs, code explanation as a starting hypothesis) versus **what requires independent human engineering reasoning** (problem framing, invariant definition, trade-off decisions, threat modeling, architecture boundaries, final verification, production accountability).
- **LO 7.2 (`Practical Competence`):** Apply **context engineering** — giving AI explicit constraints, invariants, interfaces, non-goals, and repository conventions rather than vague prompts, and keeping tasks in small, verifiable batches.
- **LO 7.3 (`Working Understanding`):** Explain concrete **AI failure mechanisms** backed by empirical evidence: confident hallucination/confabulation, **package hallucination / slopsquatting**, silent security vulnerabilities paired with developer overconfidence, plausible-looking logic/edge-case bugs, subtle API/version drift, and systemic amplification of weak engineering practices (DORA 2024/2025). *(Teaching note: practical verification of these failure modes is exercised under LO 7.4 at `Practical Competence`.)*
- **LO 7.4 (`Practical Competence`):** Execute a disciplined **AI verification workflow** (line-by-line code review, dependency/version verification, negative/edge-case testing, security sanity checks, and independent execution) before accepting or committing AI-assisted code.

#### Terminology Inventory
| Term | Target Level | Why It Matters in Phase 1 |
| :--- | :--- | :--- |
| Delegation vs. Abdication | `Practical Competence` | Delegating drafting speed while retaining specification, verification, and ownership vs. blindly trusting output. |
| Context Engineering | `Practical Competence` | Curating the exact specs, constraints, rules, files, and feedback loops the model sees to constrain probabilistic output. |
| Hallucination / Confabulation | `Working Understanding` | Probabilistic generation of plausible-sounding but false facts, nonexistent APIs, wrong flags, or fabricated citations. |
| Package Hallucination ("Slopsquatting") | `Working Understanding` | LLM fabrication of nonexistent `npm`/`PyPI` package names that attackers can register for supply-chain compromise. |
| Automation Bias & Overconfidence Trap | `Working Understanding` | Empirically documented tendency of developers using AI to trust insecure or subtly buggy code more than they should. |
| AI as an Amplifier (DORA 2025) | `Working Understanding` | AI magnifies existing engineering discipline (small batches, version control, tests, reviews) OR magnifies existing organizational chaos and instability. |
| Verification Harness | `Practical Competence` | Deterministic checks (tests, linters, type checks, schema constraints, empirical execution) used to constrain non-deterministic AI output. |

#### Authoritative Resources & Reading Instructions

*Mandatory reading below is scoped to targeted sections/findings (~90–110 min total) so at least half of the 4-hour block remains available for the hands-on audit lab and exercises.*

| Priority | Access | Resource | Reading Scope (Verified vs. Unverified) | Why Useful & Objectives Supported |
| :--- | :---: | :--- | :--- | :--- |
| **Mandatory (Targeted Sections)** | Free | DORA (DevOps Research and Assessment): [2024 Accelerate State of DevOps Report](https://dora.dev/dora-report-2024) | **Verified (~15 min — AI findings summary only):** Survey findings on AI adoption, attitudes, reported productivity/flow benefits alongside reductions in software delivery performance, and the exact trust finding that **39.2% of respondents reported little or no trust in AI**. *Scope note: Large-scale survey research reflecting reported organizational patterns, not a universal outcome for every team or tool.* | Establishes the 2024 empirical baseline of AI adoption benefits alongside delivery-performance trade-offs and trust limits. Supports LO 7.1, LO 7.3. |
| **Mandatory (Targeted Sections)** | Free | DORA (DevOps Research and Assessment): [2025 DORA Report: State of AI-assisted Software Development](https://dora.dev/research/2025/dora-report/) | **Verified (~15 min — executive findings only):** Read the *"AI is an amplifier"* thesis showing that AI magnifies existing organizational strengths and weaknesses rather than fixing broken delivery systems. | Separates the 2025 systemic amplifier thesis cleanly from the 2024 report. Supports LO 7.1, LO 7.3. |
| **Mandatory (Targeted Sections)** | Free | DORA (DevOps Research and Assessment): [DORA AI Capabilities Model (2025 Companion Report)](https://dora.dev/ai/capabilities-model/report/) *(overview: [dora.dev/ai](https://dora.dev/ai/))* | **Verified (~10 min — 7 capabilities overview):** Skim the seven foundational capabilities enabling positive AI ROI (e.g., *Working in Small Batches*, *Strong Version Control Practices*, *User-Centric Focus*, *Clear AI Stance* — enabling organizational capabilities, not evidence of AI-induced batch-size inflation). | Direct link to the companion model so the learner does not have to hunt for the seven capabilities indirectly. Supports LO 7.1, LO 7.3. |
| **Mandatory (Targeted Sections)** | Free | Neil Perry, Megha Srivastava, Deepak Kumar, Dan Boneh (Stanford, ACM CCS 2023): *"Do Users Write More Insecure Code with AI Assistants?"* — [arXiv:2211.03622](https://arxiv.org/abs/2211.03622) / [DOI 10.48550/arXiv.2211.03622](https://doi.org/10.48550/arXiv.2211.03622) | **Verified (~20 min — targeted sections only):** Read Abstract, key empirical security & overconfidence findings, and the prompt/critical-engagement takeaway. *Experimental context: Controlled user study evaluating `codex-davinci-002` on specific security tasks; illustrates the behavioral overconfidence mechanism in that setting rather than a universal defect rate for every current AI assistant.* **Unverified:** Exact internal section numbering of the camera-ready CCS 2023 PDF vs. arXiv preprint `[Unverified]`. | Empirical illustration of the overconfidence trap and why human security skepticism is mandatory. Supports LO 7.3, LO 7.4. |
| **Mandatory (Targeted Sections)** | Free | Joseph Spracklen et al. (USENIX Security 2025 Distinguished Paper): *"We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code Generating LLMs"* — [USENIX Security '25](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen) / [arXiv:2406.10279](https://arxiv.org/abs/2406.10279) | **Verified (~20 min — targeted sections only):** Read Abstract, key quantitative findings (**576,000** generated Python/JS code samples containing **~2.23 million package recommendations**, of which **440,445 package recommendations (19.7%)** were hallucinated — averaging **5.2%** for commercial models and **21.7%** for open-source models, yielding 205,000+ unique fake package names), the slopsquatting threat model, and the mitigations summary. **Clarification:** *19.7% is the proportion of package recommendations, not the proportion of code samples.* *Limitation: These results reflect the paper's 16 evaluated models, specific prompts, and package-repository snapshot (**January 10, 2024**); the paper also acknowledges newer models emerged after its evaluation, so its results must not be treated as a current universal error rate.* | Directly relevant to Node.js (`npm`) backend engineering; demonstrates a concrete supply-chain failure mode of AI code generation. Supports LO 7.3, LO 7.4. |
| **Mandatory (Targeted Sections)** | Free | Birgitta Böckeler (MartinFowler.com, **5 February 2026**): [*Context Engineering for Coding Agents*](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html) | **Verified (~25 min):** Read core sections on curating what the model sees, *Instructions vs. Guidance/Rules*, and structured context/outer harness. | Current practitioner guide connecting context provision directly to the rules/skills workflow used in this repository. Supports LO 7.2, LO 7.4. |
| **Supplementary** | Free | Anthropic Engineering, [*Effective Context Engineering for AI Agents*](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) | **Verified:** Focus on context curation, signal-to-noise degradation ("lost-in-the-middle"), and structured context boundaries. **Unverified:** Exact publication date and internal section headings `[Unverified]`. | Reinforces mechanism-level reasons why dumping massive unstructured context degrades AI output quality. Supports LO 7.2. |
| **Reference** | Free | NIST SP 800-218 (Version 1.1, Feb 2022): [*Secure Software Development Framework (SSDF)*](https://csrc.nist.gov/pubs/sp/800/218/final) | **Verified:** High-level awareness of general SSDF practices (*Prepare the Organization, Protect the Software, Produce Well-Secured Software, Respond to Vulnerabilities*) as the baseline for verifying and securing application code (including AI-assisted application code). | Provides the authoritative general software-verification baseline applicable to AI-assisted application development. Supports LO 7.4 (`Awareness`). |
| **Supplementary** | Free | NIST SP 800-218A (2024): [*Secure Software Development Practices for Generative AI and Dual-Use Foundation Models*](https://csrc.nist.gov/pubs/sp/800/218/a/final) | **Verified:** Awareness-level skim of Executive Summary. *Scope:* An **AI-specific SSDF companion for generative AI / foundation-model development**, with relevance to organizations **producing or acquiring AI systems using those models**. It is **not** the primary checklist for ordinary backend developers consuming third-party model APIs (use general SSDF SP 800-218 principles for application-code verification). | Clarifies the boundary between foundation-model/AI-system lifecycle security vs. general application-code security when consuming third-party model APIs. Supports LO 7.3 (`Awareness`). |
| **Reference** | Free | OWASP GenAI Security Project (2025): [*OWASP Top 10 for LLM Applications*](https://genai.owasp.org/llm-top-10/) | **Verified:** Awareness-level skim of categories relevant to AI-assisted development (Supply Chain, Improper Output Handling, Sensitive Information Disclosure, Misinformation/Hallucination). Detailed LLM application security belongs in Phase 19. | Establishes shared security vocabulary without turning Phase 1 into Phase 19. Supports LO 7.3 (`Awareness`). |

#### Misconceptions, Common Mistakes & Key Distinctions
- **Misconception:** *"If AI generates code that runs and passes the AI's own generated tests, it is verified."* → **Correction:** When AI generates both code and tests from the same flawed assumption, the tests merely confirm the bug. Independent specification checks and adversarial/edge-case tests are required.
- **Misconception:** *"Better prompts eliminate hallucinations and security flaws."* → **Correction:** Better context reduces error rates, but LLMs remain probabilistic; deterministic verification (execution, static checks, dependency audits, human code review) is non-negotiable.
- **Misconception:** *"Using AI means I don't need to understand every line I commit."* → **Correction:** If you cannot explain why every line works, what edge cases break it, and how to debug it at 3 AM, you do not own it.
- **Key Distinction:** **Speed of code generation** vs. **speed of verified, maintainable delivery** (small batches + review + testing vs. dumping 800 unreviewed lines into a PR).

#### Realistic Scenarios & Practical Exercises
1. **AI Output Audit Lab (Node.js / SQL Context):** Audit a realistic AI-generated ~50-line Node.js + SQL snippet intentionally seeded with 4 realistic AI failure modes: (a) a hallucinated or outdated package/API option, (b) a subtle SQL injection or missing input boundary validation, (c) a silent error-swallowing `catch` block or broken edge case (e.g., timezone/null/empty array), and (d) an unstated concurrency/idempotency assumption. The learner audits it independently, verifies packages/APIs against official docs, and writes failing tests exposing each defect before fixing them.
2. **Delegation Boundary Exercise:** Given 6 concrete backend tasks (drafting a schema validation helper, choosing an authentication architecture, brainstorming edge-case tests, designing transaction boundaries for money transfer, explaining an unfamiliar error message, approving a production migration), classify each as *Good AI Delegation (with verification)*, *Collaborative Drafting (human leads constraints)*, or *Human-Owned Decision*, and justify why.
3. **Context Packet Construction:** Compare output quality between a vague 1-line prompt and a structured context packet (goal, non-goals, exact types/constraints, Node/Postgres versions, error-handling invariant, forbidden dependencies) on a small pure-logic backend task—documenting observed differences objectively.

#### What the Learner Should Be Able to Explain, Analyze, or Demonstrate
- Explain empirically (citing DORA 2024/2025, Perry et al., and Spracklen et al.) why AI amplifies risk when paired with weak verification, uncritical trust, or weak engineering systems.
- Construct a structured context brief for an AI coding task and execute an independent verification checklist (logic, edge cases, dependencies, security, maintainability) on AI-generated code.

---

### Block 8: Engineering Ownership and Practical Laboratory (4 hours)

#### Learning Objectives
- **LO 8.1 (`Practical Competence`):** Demonstrate **engineering ownership and professional accountability**: individuals **cannot abdicate responsibility for work they author, approve, or ship** (never deflecting blame to an AI tool or vague ticket), while recognizing that **organizational accountability is distributed** across teams, reviewers, leadership, and operational systems. *(Teaching note: introduce conceptual ethics/accountability principles first, then demonstrate practically through the Block 8 review and ownership sign-off.)*
- **LO 8.2 (`Working Understanding`):** Apply disciplined **code review** standards (both as author preparing reviewable, small, well-explained changes and as reviewer verifying design, functionality, complexity, tests, and readability).
- **LO 8.3 (`Practical Competence`):** Synthesize **all eight Phase 1 blocks** in an end-to-end Practical Laboratory: framing an ambiguous backend problem, surfacing assumptions/constraints, evaluating trade-offs in an ADR, using AI deliberately under strict context and verification discipline, conducting an independent audit, and taking explicit sign-off accountability.

#### Terminology Inventory
| Term | Target Level | Why It Matters in Phase 1 |
| :--- | :--- | :--- |
| Engineering Ownership & Accountability | `Practical Competence` | Owning outcomes end-to-end—from problem framing through verification, deployment impact, and maintenance. |
| Author Responsibility in Code Review | `Practical Competence` | Submitting AI-generated or poorly verified code shifts uncompensated review burden onto teammates and erodes trust. |
| Professional Ethics & Harm Avoidance | `Working Understanding` | Technical decisions affect real users' money, privacy, safety, and trust—never purely abstract puzzles. |
| Defense of Decisions | `Practical Competence` | Being able to explain *why* code/architecture is structured as it is, what alternatives were rejected, and what fails first. |
| Operational Ownership ("You Build It, You Run It") | `Awareness` | Introduces the mindset that writing code is not the finish line; operating and debugging it is part of ownership (expanded in Phases 20–24). |

#### Authoritative Resources & Reading Instructions

| Priority | Access | Resource | Reading Scope (Verified vs. Unverified) | Why Useful & Objectives Supported |
| :--- | :---: | :--- | :--- | :--- |
| **Mandatory** | Free | Google Engineering Practices: [Code Review Developer Guide](https://google.github.io/eng-practices/) | **Verified:** Read *How to do a code review* (*The Standard of Code Review*, *What to look for in a code review*) and *The CL Author's Guide* (clear descriptions, keeping changes small). | Concrete operational definition of author accountability and reviewer rigor—directly applicable to reviewing AI-assisted PRs. Supports LO 8.1, LO 8.2. |
| **Mandatory** | Free | ACM, [*ACM Code of Ethics and Professional Conduct* (2018)](https://www.acm.org/code-of-ethics) | **Verified:** Read Section 1 (General Ethical Principles: 1.1, 1.2 *Avoid harm*, 1.3 *Be honest and trustworthy*) and Section 2 (Professional Responsibilities: 2.1 *Strive to achieve high quality*, 2.2 *Maintain high standards of professional competence*, 2.5 *Give comprehensive and thorough evaluations of computer systems and their impacts*). | Authoritative professional standard framing accountability for technical decisions and their human/business consequences. Supports LO 8.1 and Outcome 10. |
| **Supplementary** | Free | Tom Manshreck & Caitlin Sadowski, *Software Engineering at Google* (2020): [Chapter 9 — Code Review](https://abseil.io/resources/swe-book/html/ch09.html) | **Verified:** Read *Benefits of Code Review* and *Code Review Best Practices*. | Connects code review to long-term sustainability and collective ownership over time. Supports LO 8.1, LO 8.2. |
| **Reference** | Free | ACM / IEEE-CS, [*Software Engineering Code of Ethics and Professional Practice* (1999)](https://ethics.acm.org/code-of-ethics/software-engineering-code/) | **Verified:** Skim the 8 Principles (*Public, Client and Employer, Product, Judgment, Management, Profession, Colleagues, Self*), especially *Product* and *Judgment*. | Software-engineering-specific companion to the 2018 ACM Code. Supports LO 8.1. |

#### Misconceptions, Common Mistakes & Key Distinctions
- **Misconception:** *"Code review is the reviewer's job to catch my bugs."* → **Correction:** The author owns correctness and verification first; code review is a second layer of defense and shared comprehension, not a substitute for author testing.
- **Misconception:** *"If the AI wrote the bug (or the PM wrote a vague ticket), it's not my fault."* → **Correction:** Shipping unverified code or implementing unexamined ambiguities is an engineering ownership failure.
- **Key Distinction:** **Territorial ownership** ("nobody else may touch my code") vs. **accountable stewardship** ("I stand behind the correctness, clarity, and operability of what I ship, and make it safe for others to maintain").

#### Realistic Scenarios & Practical Laboratory Proposal (Integrative Phase 1 Capstone Lab)
**Proposed Lab (`phases/phase-01/lab/` — to be designed/executed only during Block 8, NOT now):**
- **Objective:** Demonstrate engineering reasoning, verification, and ownership within the **4-hour** Block 8 budget—not implementation complexity.
- **Scenario:** A realistic, intentionally ambiguous backend feature request (e.g., *"Add a promo-code redemption operation for an e-commerce backend"*).
- **Strict Capstone Scope Boundary (No Scope Expansion):**
  - **Allowed:** Pure Node.js only (no web framework, no database), a small in-memory model, one narrowly defined redemption operation, a minimal test harness, and the four learner artifacts below.
  - **Explicitly Prohibited:** Production deployment, authentication/authorization, distributed locking, HTTP server frameworks, or advanced database work.
- **Required Learner Artifacts (Integrative Demonstration):**
  1. **Problem & Requirements Brief:** Surface ≥6 ambiguities/assumptions, define explicit non-goals, and write verifiable RFC 2119/8174 acceptance criteria + core domain invariants (Blocks 2 & 3).
  2. **Constraint & Trade-Off ADR:** Compare 2 concrete design approaches under explicit constraints, state downsides of the chosen approach, and explain why overengineered alternatives were rejected (Blocks 4 & 6).
  3. **Controlled AI-Assisted Implementation & Audit Log:** Delegate a bounded sub-task using a written context brief, then independently inspect, test (including adversarial/edge cases), check dependencies/assumptions, and document every flaw found and fixed (Blocks 5 & 7).
  4. **Ownership & Review Statement:** A short engineering sign-off explaining every design choice, how correctness was verified (and what remains unverified), how the component could fail in production, and what evidence would trigger revisiting the decision (Blocks 1 & 8).

#### What the Learner Should Be Able to Explain, Analyze, or Demonstrate
- Defend every requirement interpretation, trade-off, verification step, and line of code in their own lab artifact without deferring to "the AI suggested it."
- Perform a structured code/design review on a sample change using Google's review dimensions (design, functionality, complexity, tests, naming/clarity).

---

## 4. Coverage Mapping Against the Ten Canonical Phase 1 Mastery Outcomes

| # | Canonical Phase 1 Mastery Outcome | Primary Block(s) | Supporting Block(s) | Mapped Block LOs | Core Free Mandatory Sources | Verification / Demonstration Path |
| :---: | :--- | :---: | :---: | :--- | :--- | :--- |
| **1** | Explain the difference between programming and software engineering. | Block 1 | Block 6, Block 8 | `LO 1.1`, `LO 1.2`, `LO 1.3`, `LO 1.4` | *SWE at Google* Ch. 1 (open HTML) | Learner explanation across time, scale, and trade-offs + lifespan rigor classification exercise. |
| **2** | Analyze a request before implementation. | Block 2 | Block 8 | `LO 2.1`, `LO 2.2`, `LO 2.4` | *Design Docs at Google*; RFC 2119 | Problem-space vs. solution-space decomposition of an ambiguous backend ticket before coding. |
| **3** | Identify assumptions, ambiguities, and missing requirements. | Block 2 | Block 7, Block 8 | `LO 2.2`, `LO 2.3` | RFC 2119 (+ RFC 8174); *Design Docs at Google* (plus ISO 29148 criteria) | Ambiguous ticket interrogation: clarifying questions, explicit assumptions, non-goals, verifiable criteria. |
| **4** | Define correctness and meaningful software quality. | Block 3 | Block 6, Block 8 | `LO 3.1`, `LO 3.2`, `LO 3.3`, `LO 3.4` | Avižienis et al. (2004, author PDF); arc42 ISO 25010:2023; Dijkstra EWD249 | Fault→Error→Failure trace + invariant definitions + ISO 25010 quality tension matrix. |
| **5** | Identify engineering constraints. | Block 4 | Block 2, Block 8 | `LO 4.1`, `LO 2.1` | *SWE at Google* Ch. 1; Google SRE Ch. 3 (*Embracing Risk*, awareness scope) | Constraint inventory (cost, time, team, reliability, compliance) inside a realistic ADR scenario. |
| **6** | Compare alternatives and explain trade-offs. | Block 4 | Block 6, Block 8 | `LO 4.2`, `LO 4.3`, `LO 4.4` | Nygard (2011) ADR; *SWE at Google* Ch. 1; Google SRE Ch. 3 | Written 1-page ADR comparing ≥2 viable options with honest downsides of the chosen option and revisit triggers. |
| **7** | Distinguish evidence from opinion. | Block 5 | Block 3, Block 7, Block 8 | `LO 5.1`, `LO 5.2`, `LO 5.3`, `LO 5.4`, `LO 3.3` | Gregg (*Methodology* & *USE Method* web docs); Google SRE Ch. 15; DORA 2024/2025 | Anti-method incident audit + falsifiable verification plan separating observation, hypothesis, and proof. |
| **8** | Recognize unnecessary complexity and overengineering. | Block 6 | Block 4, Block 8 | `LO 6.1`, `LO 6.2`, `LO 6.3`, `LO 6.4` | Hickey (2011) *Simple Made Easy*; Ousterhout (2018) *Talks at Google* (selected sections); Fowler (2009) | Critique of an overengineered design using essential vs. accidental complexity, simple vs. easy, and Ousterhout's 3 symptoms. |
| **9** | Use AI as an engineering partner while independently verifying its output. | Block 7 | Block 5, Block 8 | `LO 7.1`, `LO 7.2`, `LO 7.3`, `LO 7.4` | DORA 2024/2025; Perry et al. (arXiv/CCS 2023); Spracklen et al. (USENIX '25); Böckeler (5 Feb 2026) | AI output audit lab (detecting planted bugs/security flaws/hallucinated deps) + structured context packet + delegation matrix. |
| **10** | Take responsibility for technical decisions and their consequences. | Block 8 | Block 1, Block 4, Block 7 | `LO 8.1`, `LO 8.2`, `LO 8.3` | ACM Code of Ethics (2018); Google Code Review Guide | End-to-end Block 8 laboratory defense & ownership sign-off statement covering risks, verification limits, and failure modes. |

---

## 5. Identified Gaps, Uncertainties, and Resource Limitations

### A. Accessibility & Paywalled Sources (Resolved for Mandatory Tier)
- **All Mandatory resources across Blocks 1–8 are freely accessible online.** No paid book or paywalled ISO/IEEE PDF is mandatory.
- **Paywalled or Commercial Supplementary/Reference Items:**
  - `ISO/IEC/IEEE 29148:2018` (`https://www.iso.org/standard/72089.html`) and `ISO/IEC 25010:2023` official PDFs are paywalled (the exact numeric `iso.org/standard/<id>` URL for `ISO/IEC 25010:2023` was not verified and is marked `[Unverified]`). Open coverage via RFC 2119/8174 and `quality.arc42.org/iso-25010/` satisfies the Phase 1 learning objectives without purchasing the standards.
  - Commercial books (*A Philosophy of Software Design*, *Systems Performance*, *Software Requirements*, *Designing Data-Intensive Applications*, *Fundamentals of Software Architecture*) and IEEE-paywalled papers (Brooks 1987, Kitchenham et al. 2004, Shaw 1990 IEEE version) are classified as **Supplementary** or **Reference**.

### B. Unverified Citation Details (Marked Transparently — Not Fabricated)
- Exact chapter/page numbers for commercial books where publisher ToCs were not directly inspected (Wiegers & Beatty 2013 / Wiegers & Hokanson 2023, Richards & Ford 2nd ed. 2025, Kleppmann & Riccomini 2nd ed. 2026, Gregg 2nd ed. 2020 Ch. 2 internal subsections) are explicitly marked `[Unverified]`.
- Exact internal section numbers of the ACM CCS 2023 camera-ready PDF of Perry et al. (vs. `arXiv:2211.03622`), internal subsection numbers of Avižienis et al. (2004) and Shaw (1990), exact timestamps of Ousterhout's 2018 *Talks at Google* video (no official YouTube chapters), and exact publication date / section headings of Anthropic's *Effective Context Engineering for AI Agents* are marked `[Unverified]`.

### C. Fast-Moving AI Evidence Base & Experimental Context (Block 7)
- Empirical AI studies must be interpreted within their experimental settings rather than generalized as universal current rates for every AI assistant:
  - **Perry et al. (ACM CCS 2023)** conducted a controlled user study with `codex-davinci-002`; it illustrates the behavioral mechanism of automation bias and overconfidence in that setting, not a fixed vulnerability rate for today's models.
  - **Spracklen et al. (USENIX Security 2025)** evaluated 16 models using specific prompts against a **January 10, 2024** package-repository snapshot; its 19.7% / 5.2% / 21.7% figures reflect that evaluation setup, not a universal current rate for today's models.
  - **DORA 2024 & 2025** are large-scale industry surveys capturing self-reported developer and organizational patterns across respondents, not a universal guarantee for every team or tool.
- Learning materials will state model generations, study designs, and snapshot dates explicitly whenever citing empirical AI research (per `.agents/rules/research-standards.md`).

### D. Continuity Note
- Phase 1 Diagnostic is `completed` and must not be repeated.
- Block 1 (*"What Is Software Engineering?"*) will be prepared **from scratch, starting from its first concept**, with no prior Block 1 content assumed completed.
