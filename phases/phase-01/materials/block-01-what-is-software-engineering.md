# Phase 1, Block 1 — What Is Software Engineering?

> **Artifact Type:** Self-Contained Teaching Lesson *(author-synthesized conceptual lesson, backend walkthroughs, exercises, and mastery checks)*
> **Estimated Learner Study Time:** **3 focused hours (180 minutes)** *(estimate, not a hard deadline; mastery takes priority)*
> **Continuity:** Starts from the **very first concept** of Block 1. *(Phase 1 Diagnostic is already `completed` — do **not** repeat it.)*
> **Canonical Outcome Coverage:** Primary support for **Outcome 1** (*Explain the difference between programming and software engineering*), with an early bridge to **Outcome 10** (*Take responsibility for technical decisions and their consequences*).
> **Learner Output Location:** Record your notes, inline check answers, exercises, and mastery responses in `phases/phase-01/notes/block-01-notes.md`.
> **Progress Rule:** Reading this lesson does **not** mark Block 1 `completed` or `approved`, and AI preparation time does **not** count toward your 3-hour study time.

---

## Learning Objectives & Target Knowledge Levels

Each objective has **one** target level from our official three-level framework (`Awareness`, `Working Understanding`, `Practical Competence`), with mechanism-level depth and production relevance applied as dimensions:

| ID | Target Level | What You Should Be Able to Do |
| :--- | :--- | :--- |
| **LO 1.1** | `Working Understanding` | Distinguish **programming**, **software development**, and **software engineering** using operational criteria, recognizing this as a course working distinction rather than a universal ISO taxonomy. |
| **LO 1.2** | `Working Understanding` | Trace how **time and change**, **scale and efficiency**, and **trade-offs and costs** change engineering decisions across a system's lifespan. |
| **LO 1.3** | `Working Understanding` | Explain the mechanisms, consequences, and limits of **Hyrum's Law** and **shifting left** in long-lived backend systems *(consolidated via Exercise 1)*. |
| **LO 1.4** | `Awareness` | Situate code construction within the broader **18 Knowledge Areas of IEEE SWEBOK V4.0 (2024)** and explain why syntax speed alone is not software engineering. |

### Realistic 3-Hour Study Budget (180 Minutes)

| Stage | Activity | Target Time |
| :---: | :--- | :---: |
| **1** | Read & study this self-contained lesson (Sections 1–7) + complete the 3 inline Knowledge Checks | 65 min |
| **2** | Complementary Mandatory Reading: *Software Engineering at Google* Ch. 1 (~30 min) + SWEBOK V4.0 KA skim (~5 min) | 35 min |
| **3** | Complete Practical Reasoning Exercises 1–3 independently in `notes/block-01-notes.md` | 55 min |
| **4** | Answer the 5 Mastery Assessment questions closed-book in `notes/block-01-notes.md` | 25 min |
| **Total** | — | **180 min (3 h)** |

### Epistemic Labels Used in This Lesson
To keep claims honest and verifiable, this lesson distinguishes three kinds of statements:
- **`[Course Working Definition]`** — A deliberate framing adopted for this program to sharpen engineering judgment (not a single universally standardized ISO/legal definition).
- **`[Source-Derived Model]`** — An analytical framework or empirical observation drawn from a cited engineering authority (e.g., Winters et al., 2020), subject to context and exceptions.
- **`[Established Fact / Standard]`** — A verifiable specification, published standard structure, or documented language/database rule.

---

## 1. Why "Working Code" Keeps Failing in Production

Imagine a developer writes a Node.js endpoint backed by PostgreSQL to summarize customer invoices. On Day 1, it passes every manual test: valid HTTP status, accurate totals, fast response. Everyone celebrates and moves on.

Fourteen months later:
- The original developer left the company.
- The Node.js major version reaches End-of-Life (EOL) and stops receiving security patches.
- A database engineer adds an index to speed up reporting, and suddenly the mobile app displays invoices in the wrong order because it silently relied on an unstated query order.
- A product manager asks to add multi-currency support, and the team realizes changing the pricing logic requires editing 14 files with no automated tests to verify nothing broke.

Nothing broke on Day 1. The *programming* worked. What failed was **software engineering**.

### The Three-Way Distinction `[Course Working Definition]`

In industry job postings and casual conversation, *"programmer,"* *"software developer,"* and *"software engineer"* are often used interchangeably. There is no universal international standard separating those three titles. However, conflating the **activities** behind them causes real failure. In this program, we use a sharp working distinction grounded in Titus Winters et al. (*Software Engineering at Google*, 2020) and IEEE SWEBOK:

| Dimension | Programming | Software Development | Software Engineering |
| :--- | :--- | :--- | :--- |
| **Core Activity** | Translating a defined task into executable code. | Building, integrating, and shipping software features or products for users. | Sustaining, verifying, evolving, and operating software systems reliably across **time**, **scale**, and **competing constraints**. |
| **Driving Question** | *"Does this code run and compute the right answer right now?"* | *"Does this feature work and ship to users?"* | *"Can a changing team safely modify, upgrade, verify, and operate this system over its expected lifespan within real constraints?"* |
| **Primary Risk Ignored If Stopped Here** | Fails once requirements, dependencies, or callers evolve. | Ships features today while accumulating fragile contracts, upgrade debt, or operational blind spots. | *Its own failure mode:* applying heavy engineering process to trivial, short-lived tasks (overengineering). |

#### Concrete Step-by-Step Backend Evolution (Node.js + PostgreSQL)

Watch how the **same domain problem** (fetching customer orders) changes across these three lenses:

1. **Programming mindset (Point-in-time task):**
   You write a small Node.js script that connects to PostgreSQL, executes `SELECT id, total_cents, status FROM orders WHERE user_id = $1`, and prints the JSON rows. If the output matches your test user today, you are done.
2. **Software development mindset (Shipping a product feature):**
   You wrap that query in `GET /api/orders`, hook up authentication middleware, format JSON for the frontend team, deploy the service, and close the ticket once the UI renders orders properly.
3. **Software engineering mindset (Sustaining a system over time, scale, and constraints):**
   Before and after shipping `GET /api/orders`, you ask lifecycle questions:
   - *What happens when a user has 50,000 orders instead of 5?* (Missing pagination becomes an availability/memory hazard.)
   - *What observable behavior might clients silently rely on?* (No `ORDER BY` means ordering can change after an index or table rewrite.)
   - *How will a new engineer know whether changing the `orders` schema breaks `GET /api/orders`?* (Automated tests and explicit response contracts.)
   - *How will we upgrade Node.js, the `pg` driver, and PostgreSQL next year without freezing feature work?* (Upgrade discipline and dependency hygiene.)

**Cause and Effect:** Software engineering does not replace programming—you must be able to program well. Rather, software engineering recognizes that **initial authoring is usually the shortest phase of a production backend system's life.**

---

## 2. Time, Change, and Software Sustainability

Software does not rust mechanically, yet untouched production software still breaks. Why? **Because the environment surrounding the software never stops changing.**

### The Cause-and-Effect Chain of Environmental Drift `[Source-Derived Model]`

Even if your team commits zero lines of code for 18 months:
1. **Security threats evolve** → new vulnerabilities (CVEs) are discovered in Node.js, OpenSSL, or transitive `npm` packages.
2. **Dependencies move forward** → Node.js LTS versions reach EOL, PostgreSQL versions deprecate behaviors, cloud providers retire runtimes.
3. **Data and traffic grow** → a query that took 4 ms on 1,000 rows takes 2 seconds on 5 million rows.
4. **Business rules and callers change** → new tax rules, new client apps, and new compliance constraints arrive.

Because change is unavoidable for long-lived systems, Winters et al. (2020) frame software engineering as **"programming integrated over time"** and introduce a practical working definition of **software sustainability**:

> **Software Sustainability `[Course Working Definition — Winters et al.]`:** A codebase and organization are sustainable when they are capable of safely reacting to every valuable technical or business change needed over the software's expected lifespan. *(Note: This refers strictly to technical and organizational lifecycle sustainability, distinct from environmental/green-computing sustainability.)*

### How Unsustainable Systems Collapse (The Deferral Spiral)

Consider a backend team that avoids dependency upgrades and refactoring because *"nothing is broken right now"*:
- **Month 6:** Skipping one minor Node.js and `pg` driver upgrade saves 4 hours today.
- **Month 18:** Five breaking changes across Node.js, TypeScript/linters, and 30 `npm` packages have stacked up. Nobody remembers why certain workarounds exist, and test coverage is thin.
- **Month 24:** A critical remote-code-execution CVE hits an EOL runtime or dependency. Upgrading now requires weeks of high-risk surgery under emergency pressure—or leaving production exposed.

**Practical Consequence:** A system that "works" today but terrifies every engineer who considers modifying or upgrading it is already failing as an engineered system.

> ### Knowledge Check 1 *(2–3 min — write answers in your notes before checking Section 11)*
> 1. Why can a backend service become *unsustainable* even if it has zero open bug tickets and 100% uptime this month?
> 2. A coworker says: *"We shouldn't touch our authentication service's dependencies—if it ain't broke, don't fix it."* Explain the flaw in that reasoning using cause and effect, and state one situation where *"don't touch it"* actually *would* be reasonable.

---

## 3. Implicit Contracts and Hyrum's Law

One of the sharpest mechanisms explaining why time and scale make change hard is **Hyrum's Law** `[Source-Derived Model — Hyrum Wright / Winters et al., 2020]`:

> *"With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody."*

### Why Hyrum's Law Happens (Mechanism, Not Magic)

Hyrum's Law is not caused by "bad" or "lazy" client developers. It emerges naturally from how software interacts across boundaries:
1. **Callers observe concrete behavior, not your internal intent.** When a client integrates with your HTTP endpoint or library, they test against what it *actually returns*, not just what you meant to promise.
2. **Unconstrained degrees of freedom leak out.** If you do not explicitly control row order, error phrasing, JSON key formatting, or response latency, whatever your current implementation happens to produce becomes visible on the wire.
3. **Client code and tests harden around those observations.** Someone writes a regex matching your error string, grabs `items[0]` assuming chronological order, or hardcodes a 200 ms timeout because your endpoint always responded in 40 ms.
4. **Future internal changes collide with hidden consumers.** When you later make a completely valid internal change, you break callers you didn't even know existed.

### Step-by-Step Backend Walkthrough: How an Unordered SQL Query Breaks Production

Suppose your Node.js + PostgreSQL backend implements `GET /api/orders` like this:

```js
// Day 1 implementation (simplified Node.js + pg example)
async function getUserOrders(pool, userId) {
  const result = await pool.query(
    'SELECT id, total_cents, status FROM orders WHERE user_id = $1',
    [userId]
  );
  return result.rows;
}
```

- **Established SQL Fact `[Established Fact / Standard]`:** In SQL (and PostgreSQL specifically), a `SELECT` query without an `ORDER BY` clause makes **no guarantee** about the order in which rows are returned.
- **Step 1 (Months 1–12):** Because rows were inserted sequentially into a small table with no updates or secondary index scans, PostgreSQL *happens* to return them in creation order.
- **Step 2 (Hidden Coupling):** Your API documentation says *"Returns the user's orders"* (making no order promise). Still, the iOS app displays `orders[orders.length - 1]` as *"Most Recent Order,"* and a partner's billing script asserts on `orders[0]`. Both work in testing and production.
- **Step 3 (Day 400 — Legitimate Internal Change):** As the `orders` table grows, you add a composite index on `(user_id, status)` or update older rows *(note: physical heap/index scan mechanics belong to Phase 10; here, focus solely on the observable contract shift)*. PostgreSQL now chooses a faster query plan that returns rows ordered by `status` instead of insertion time.
- **Step 4 (Production Failure):** Your query is still 100% SQL-valid and honors your written API doc—yet mobile users see cancelled orders from last year at the top of their screen, and the partner's invoicing script fails.

#### Concrete Engineering Countermeasures
1. **Close accidental degrees of freedom explicitly:** Add deterministic ordering (`ORDER BY created_at DESC, id DESC`) and document it in the API contract—or explicitly design pagination tokens so order is intentional.
2. **Expose structured contracts, not internal leaks:** Return stable machine-readable error codes (`{ "code": "ORDER_NOT_FOUND" }`) rather than raw PostgreSQL driver error strings that clients might parse with regexes.
3. **Map database rows explicitly:** Avoid `SELECT *` and returning raw DB objects directly; explicitly project fields so adding an internal DB column doesn't silently alter API responses.

### Limitations and Exceptions of Hyrum's Law
- **Scale dependence:** Hyrum's Law requires *"a sufficient number of users."* A private helper function called in 2 places inside one repository you fully control does **not** suffer the same ossification as a public API with 400 external integrators.
- **Not a mandate to freeze forever:** Hyrum's Law does **not** mean you can never change observable behavior. It means you must *expect* hidden dependencies, measure impact (telemetry, deprecation warnings, staged rollouts), and decide deliberately whether the benefit of change justifies the migration cost.

---

## 4. Scale, Organizational Friction, and Shifting Left

### How Scale Breaks Policies That Worked Yesterday `[Source-Derived Model]`

Software engineering must account for multiple kinds of scaling: **codebase size**, **number of engineers**, **number of users/requests**, and **number of dependencies**.

Consider a concrete policy: *"Whenever we upgrade a shared validation helper, the author manually messages every team on Slack and asks them to update their endpoints."*
- **At 2 engineers and 5 endpoints:** Takes 10 minutes. Totally reasonable!
- **At 40 engineers and 300 endpoints:** Coordination stalls for weeks, half the teams ignore the Slack message, old and new versions coexist indefinitely, and every future upgrade compounds the mess.

**Qualified Engineering Principle:** Recurring maintenance, verification, and coordination policies whose human effort grows linearly or super-linearly with codebase size or team headcount eventually saturate engineering capacity. For **high-frequency, broad-scope policies** (formatting, linting, dependency checks, test execution, routine rollouts), engineers seek automation and tooling whose human cost grows slowly—ideally sub-linearly—as the system grows.

**Important Qualification (Avoid Overgeneralizing):** Not *every* engineering practice can or should be automated to scale sub-linearly. High-judgment activities—such as scoping ambiguous business problems, reviewing novel architecture trade-offs, investigating unprecedented production incidents, and mentoring engineers—inherently require human reasoning. Good engineering automates repetitive mechanical policies so scarce human judgment is preserved for decisions that genuinely need it.

### Shifting Left: Catching Flaws Earlier (Mechanism & Limits)

Every software change moves through stages from left to right:

```text
Problem Framing & Design → Coding → Automated Checks/Tests → Code Review → Staging → Production
  [Cheaper to revise]  ------------------------------------------------------>  [Costlier to repair]
```

**Why catching defects earlier is often substantially cheaper (Cause and Effect):**
1. **Context is fresh:** Fixing a mistaken assumption during design or right after writing a function takes minutes because the mental model is still active in your head.
2. **Blast radius is zero:** No external API consumers have integrated against a flawed schema, and no corrupted financial rows have been written into PostgreSQL.
3. **No emergency coordination cost:** Fixing a bug in a local test run does not wake on-call engineers, require data-repair migrations, or trigger customer incident reports.

**Important Qualification & Limits of Shifting Left:**
- **Beware overly absolute formulas:** Textbooks and blog posts frequently assert that production bugs are *"always exponentially (10x–100x) more expensive"* than design bugs. Treat that as a **directional heuristic, not a universal empirical law**—actual cost ratios vary wildly depending on domain, release automation, feature flags, and reversibility. A minor CSS typo deployed behind continuous delivery can be fixed in production in 3 minutes; a corrupted billing ledger schema can cost months of reconciliation.
- **When shifting left goes too far:** Demanding a 15-page formal design specification and 3 committee approvals for a small, reversible experiment slows learning and increases opportunity cost. **Shift left proportionally to blast radius and irreversibility.**

> ### Knowledge Check 2 *(2–3 min — write answers in your notes before checking Section 11)*
> 1. Give one example of a recurring backend policy that breaks down as a team grows from 2 to 30 engineers, and one engineering activity that *cannot* simply be automated into a sub-linear policy.
> 2. Why is it inaccurate to claim that *"catching a bug earlier is always exponentially cheaper"*? Give one counterexample where fixing a minor issue after release is cheap, and one case where catching it early saves massive cost.

---

## 5. Trade-offs, Multi-Dimensional Costs, and Calibrating Rigor

A defining trait of software engineering is rejecting simplistic rules like *"always use microservices"* or *"always maximize performance."* Every choice trades one set of costs and risks for another.

### Six Real Cost Dimensions `[Source-Derived Model — Winters et al., 2020]`

When evaluating any backend choice, weigh all six costs—not just cloud infrastructure bills:

| Cost Type | What It Measures | Concrete Backend Example |
| :--- | :--- | :--- |
| **Financial** | Direct money spent (cloud, SaaS, licenses). | Paying for a managed PostgreSQL instance vs. self-hosting on raw VMs. |
| **Resource** | Compute, memory, storage, network bandwidth. | Returning full nested JSON payloads instead of paginated summaries increases DB I/O and bandwidth. |
| **Personnel** | Engineering time to build, review, debug, and operate. | Self-hosting a database saves monthly cloud fees but consumes 10 engineering hours/month in backups, patching, and on-call toil. |
| **Transaction** | Friction and risk required to *perform* a change or migration. | Splitting a table or renaming a live API field requires dual-writes, versioning, and coordinated rollouts. |
| **Opportunity** | Valuable product or reliability work *not* done because capacity was spent here. | Spending 3 weeks building a custom internal framework instead of shipping core customer billing fixes. |
| **Societal / User** | Real-world impact on users (harm, data exposure, lost money, downtime). | Skipping input validation or dependency security patching exposes customer PII. |

**Revisiting Decisions:** Because inputs (team size, traffic, revenue, regulatory scope) change over time, **good engineering decisions have expiration dates**. Choosing a single Node.js service and a single PostgreSQL database at 200 users is usually smart; revisiting that choice only when measured evidence shows a bottleneck or organizational boundary problem is sound engineering.

### Contrasting Scenarios: When Rigor Is Necessary vs. When It Is Overhead

Engineering maturity is **not** applying maximum process everywhere. It is calibrating rigor to **expected lifespan**, **scale**, and **blast radius**.

| Dimension | Scenario A: High Rigor Necessary | Scenario B: Heavy Rigor Is Harmful Overhead |
| :--- | :--- | :--- |
| **System** | Public B2B webhook & billing API serving 400 paying companies over a 5-year lifespan. | A 25-line Node.js script run once locally by one engineer to normalize capitalization in a one-time CSV import. |
| **If It Fails** | Duplicate charges, broken partner pipelines, legal/financial liability, lost trust. | Script prints an error or misformats a local output file; engineer inspects a sample, fixes line 12, and re-runs before importing. |
| **Appropriate Rigor** | Explicit versioned API contract, deterministic `ORDER BY`, structured error codes, automated integration tests, dependency upgrade cadence, staged rollouts, clear ownership. | Clear input/output verification on a test copy of the CSV, spot-check of edge cases (commas/quotes/nulls), run once, discard or archive with a 3-line comment. |
| **Failure Mode** | **Under-engineering:** skipping tests, leaking undocumented behaviors (Hyrum's Law), letting Node.js go EOL. | **Over-engineering:** spending 2 days building a modular plugin architecture, CI pipeline, Docker image, and semantic versioning for a script that dies at 5:00 PM today. |

> ### Knowledge Check 3 *(2–3 min — write answers in your notes before checking Section 11)*
> 1. A startup saves $300/month in cloud bills by self-managing its own database cluster on raw servers instead of using a managed database, but spends ~15 hours/month of senior engineering time handling patches, backups, and alerts. Which cost dimensions were traded off, and why might this be a net loss?
> 2. Why is over-engineering a 25-line throwaway script a failure of *software engineering*, not merely "being extra thorough"?

---

## 6. Why Coding Is Only 1 of 18 Areas (SWEBOK V4.0 Orientation)

As an `[Established Fact / Standard]`, the IEEE Computer Society's ***Guide to the Software Engineering Body of Knowledge* (SWEBOK V4.0, 2024)** organizes software engineering into **18 Knowledge Areas** (adding *Software Architecture*, *Software Engineering Operations*, and *Software Security* in V4):

1. Software Requirements · 2. Software Architecture *(V4)* · 3. Software Design · 4. **Software Construction** *(writing code)* · 5. Software Testing · 6. Software Engineering Operations *(V4)* · 7. Software Maintenance · 8. Software Configuration Management · 9. Software Engineering Management · 10. Software Engineering Process · 11. Software Engineering Models and Methods · 12. Software Quality · 13. Software Security *(V4)* · 14. Software Engineering Professional Practice · 15. Software Engineering Economics · 16. Computing Foundations · 17. Mathematical Foundations · 18. Engineering Foundations.

**Why This Matters Today (`Awareness`):**
Writing syntax (*Software Construction*) is **one** of eighteen knowledge areas. Modern AI assistants can generate syntax rapidly—which makes *Software Construction* cheaper in keystrokes while making the surrounding areas (framing requirements, verifying correctness, evaluating trade-offs, protecting security, operating systems, and maintaining sustainability) **more decisive than ever**. An engineer who cannot judge whether generated code fits system boundaries, preserves contracts, or remains maintainable over time is still only operating at the programming layer.

---

## 7. Common Misconceptions & Real-World Failure Patterns

1. **The "Cleaner Syntax = Software Engineering" Illusion**
   - *Failure pattern:* Obsessing over variable naming or clever abstractions while shipping an undocumented, unpaginated endpoint with no tests and an EOL runtime.
   - *Reality:* Clean code helps readability (one slice of maintainability), but software engineering spans lifecycle sustainability, verification, trade-offs, security, and operations.
2. **The "Maximum Rigor Everywhere" Cargo Cult**
   - *Failure pattern:* Copying hyperscaler processes (heavy RFC gates, multi-service splits, elaborate abstractions) onto tiny prototypes or one-off scripts.
   - *Reality:* Good engineering calibrates process cost to lifespan, scale, and blast radius.
3. **The "Don't Touch Working Code" Freeze Trap**
   - *Failure pattern:* Treating untouched legacy services as "safe" until an EOL runtime or security vulnerability forces a catastrophic rewrite.
   - *Reality:* Refusing small, steady upgrades converts manageable maintenance into high-risk transaction cost.
4. **The "Not In Our Docs, Not Our Fault" Fallacy**
   - *Failure pattern:* Breaking dozens of clients by changing an undocumented response behavior and arguing *"they violated our spec."*
   - *Reality:* Under Hyrum's Law, observable behavior becomes de facto dependency; breaking paying users is still your operational and business problem.
5. **The "Permanent Best Practice" Dogma**
   - *Failure pattern:* Treating a past architecture choice as sacred long after the team size, traffic, or budget that justified it has changed.
   - *Reality:* Decisions are hypotheses bound to historical inputs and constraints; engineers revisit them when inputs shift.

---

## 8. Practical Reasoning Exercises (~55 Minutes Total)

> **Ownership Rule:** Write all responses independently in `phases/phase-01/notes/block-01-notes.md`. Do **not** ask AI to draft your answers.

### Exercise 1: Hyrum's Law, Contracts & Shifting Left (~20 minutes)

**Scenario:**
Your team maintains a Node.js + PostgreSQL backend exposing `GET /api/orders`. The API documentation says only: *"Returns a JSON array of the authenticated user's orders."* Internally, the query omits `ORDER BY`. For 18 months, PostgreSQL returned rows in creation order, so both your mobile app and two enterprise partners quietly relied on `orders[0]` being the oldest order and `orders[orders.length - 1]` being the newest. After adding a new performance index on `orders`, PostgreSQL returns rows in a different order *(focus on the observable API contract—do not dive into Phase 10 storage internals)*. Mobile users now see old cancelled orders first, and a partner's billing script pulls the wrong invoice.

**Tasks:**
1. **Cause-and-Effect Diagnosis:** Trace the chain of events connecting the Day-1 omission, Hyrum's Law, and the Day-540 outage.
2. **Programmer vs. Software Engineer Response:** Contrast a purely point-in-time programmer's reaction (*"We never documented an order; clients must fix their code"*) with a software engineer's immediate mitigation and long-term contract fix.
3. **Proportional Shift Left:** Identify **two** earlier lifecycle checkpoints where this risk could have been caught cheaply—and state **one** heavyweight safeguard that would have been unnecessary overhead for an early MVP.

---

### Exercise 2: Calibrating Engineering Rigor Across Four Backend Artifacts (~20 minutes)

Evaluate these four backend artifacts:
- **Artifact A:** A 30-line Node.js script cleaning duplicate whitespace in one historical CSV file before a single local import today.
- **Artifact B:** An internal nightly cron job used by 3 operations staff to email a summary of failed shipments each morning.
- **Artifact C:** A public B2B billing webhook and invoice API integrated by 400 external paying companies.
- **Artifact D:** The company's central authentication and session-validation service called on every backend request.

**Tasks:**
For **each** artifact (A–D):
1. Estimate its expected **lifespan** and **blast radius** if it fails.
2. Specify the appropriate level of engineering rigor (testing, contracts/versioning, upgrade policy, review depth).
3. Give **one concrete failure of under-engineering** AND **one concrete failure of over-engineering** for that specific artifact.

---

### Exercise 3: Multi-Dimensional Cost & Sustainability Analysis (~15 minutes)

**Scenario:**
A 3-engineer backend team runs a production Node.js API on a version reaching End-of-Life (EOL) in 4 months. Upgrading now takes ~1.5 engineering days (dependency audit, test verification, staged rollout). A manager says: *"Skip the upgrade for two years so we ship features faster—nothing is broken right now."*

**Tasks:**
1. Analyze the manager's proposal across **at least four** of Winters et al.'s six cost dimensions (financial, resource, personnel, transaction, opportunity, societal/user).
2. Explain the cause-and-effect impact of a 2-year deferral on **software sustainability**.
3. Describe **one realistic scenario** where deferring the upgrade for a short, bounded window (e.g., 3 weeks) *would* be a sound engineering trade-off—and specify the exact condition/commitment required so it does not turn into indefinite deferral.

---

## 9. Closed-Book Mastery Assessment (~25 Minutes)

Close this lesson and external tabs. Write original, clear answers to all five questions in `phases/phase-01/notes/block-01-notes.md`, then submit your work to **ChatGPT (Curriculum Owner & Instructor)** for evaluation:

1. **(LO 1.1 / Outcome 1)** Using a concrete backend example, explain the operational difference between **programming**, **software development**, and **software engineering**. Why do we treat this three-way split as a course working distinction rather than a universally standardized ISO taxonomy?
2. **(LO 1.2 / Outcome 1)** Explain how **time and change**, **scale and efficiency**, and **trade-offs and costs** each force decisions that do not arise in a single-use program.
3. **(LO 1.3)** Explain how **Hyrum's Law** works mechanistically, why **shifting left** reduces many (though not universally all) defect costs, and where each concept has realistic limits.
4. **(LO 1.2 / Outcome 10)** Define **software sustainability** as used in this program and explain why *"working code nobody dares to touch"* represents an engineering failure with real consequences.
5. **(LO 1.4)** Referring to SWEBOK V4.0's breadth (18 Knowledge Areas), explain why generating code faster—whether by hand or with AI—does not by itself solve software engineering.

### Block 1 Readiness Checklist (Before Moving to Block 2)
- [ ] Studied Sections 1–7 and completed all 3 inline Knowledge Checks (checking against Section 11 afterward).
- [ ] Completed complementary mandatory reading (*Software Engineering at Google* Ch. 1) and 5-minute SWEBOK V4.0 KA skim.
- [ ] Independently written answers to Exercises 1–3 in `phases/phase-01/notes/block-01-notes.md`.
- [ ] Independently answered Mastery Assessment Questions 1–5 closed-book.
- [ ] Submitted Block 1 work to ChatGPT for feedback and confirmation before starting Block 2.

---

## 10. Concise Glossary

| Term | Basis | Definition |
| :--- | :--- | :--- |
| **Programming / Software Development / Software Engineering** | `[Course Working Definition]` | Writing executable code for a task vs. shipping product features vs. sustaining, verifying, evolving, and operating software across time, scale, and constraints. |
| **Software Lifespan** | `[Source-Derived Model]` | Expected operational duration of a codebase/artifact (minutes to decades), which governs how much lifecycle rigor is justified. |
| **Software Sustainability** | `[Course Working Definition]` | Winters et al.'s lifecycle definition: ability of a codebase and organization to safely react to every valuable technical or business change needed over its expected lifespan (distinct from environmental sustainability). |
| **Hyrum's Law** | `[Source-Derived Model]` | Empirical observation that with enough API users, every observable system behavior—documented or accidental—will eventually be depended upon. |
| **Shifting Left** | `[Source-Derived Model]` | Moving defect detection, validation, and design critique earlier in the software lifecycle to reduce repair cost and blast radius, proportional to risk. |
| **Technical Scope vs. System Boundaries** | `[Course Working Definition]` | Recognizing that code lives inside broader organizational, operational, dependency, user, and economic boundaries. |
| **SWEBOK V4.0** | `[Established Fact / Standard]` | IEEE Computer Society's *Guide to the Software Engineering Body of Knowledge* (2024), organizing software engineering into 18 Knowledge Areas. |

---

## 11. Inline Knowledge Check Guidance *(Instructor / Self-Check Key)*

> *Check these only **after** writing your own answers to Knowledge Checks 1–3. (No solutions are provided here for Section 8 Exercises or Section 9 Mastery Assessment—those remain learner-owned for ChatGPT review.)*

- **Knowledge Check 1:**
  1. *Strong answer includes:* External dependencies, runtimes (Node.js/PostgreSQL), security vulnerabilities, data volume, and business needs keep evolving even when the code is untouched. If the team cannot safely upgrade or modify the service when those changes arrive, it is unsustainable despite zero current tickets.
  2. *Strong answer includes:* Deferring authentication dependency upgrades lets security CVEs and breaking changes accumulate until an unavoidable upgrade becomes a high-risk emergency. A reasonable exception for *"don't touch it"* is a genuinely short-lived, isolated artifact scheduled for retirement before EOL/security exposure—or deferring cosmetic refactoring that adds risk without lifecycle value.
- **Knowledge Check 2:**
  1. *Strong answer includes:* Manual cross-team coordination checklists (e.g., manually notifying every consumer on Slack to update a shared helper or manually running tests on every engineer's laptop) break down as headcount grows. Activities like ambiguous problem framing, novel architectural trade-off reviews, and unprecedented incident diagnosis inherently require human judgment and cannot simply be automated away.
  2. *Strong answer includes:* Actual fix cost depends on blast radius, reversibility, and deployment tooling—not a universal exponential formula. A minor typo in an internal UI label behind fast CI/CD is cheap to fix post-release; a flawed database schema or public API contract breaking 400 billing integrations is vastly more expensive to fix after release than during design.
- **Knowledge Check 3:**
  1. *Strong answer includes:* The startup reduced direct **financial** cost ($300/month) by dramatically increasing **personnel** cost (15 senior hours/month) and **opportunity** cost (product/reliability work not done), plus potential **societal/user** cost if manual backup/patching errors cause data loss.
  2. *Strong answer includes:* Software engineering optimizes for lifecycle outcomes under constraints. Spending days on versioning, CI, and abstractions for a script that runs once and is discarded wastes personnel and opportunity cost without reducing meaningful risk—failing to match rigor to lifespan and blast radius.

---

## 12. Complementary Reading & Authoritative Sources

### Mandatory Complementary Reading (~30 min — Read After Sections 1–7)
- **Titus Winters, Tom Manshreck, Hyrum Wright**, *Software Engineering at Google: Lessons Learned from Programming Over Time* (O'Reilly, 2020; freely available HTML edition):
  - **Direct Link:** [Chapter 1 — What Is Software Engineering?](https://abseil.io/resources/swe-book/html/ch01.html)
  - **Exact Sections:** *Time and Change* (*Hyrum's Law*, *Example: Hash Ordering*, *Why Not Just Aim for "Nothing Changes"?*), *Scale and Efficiency* (*Policies That Don't Scale*, *Policies That Scale Well*, *Example: Compiler Upgrade*, *Shifting Left*), *Trade-offs and Costs* (*Example: Markers*, *Inputs to Decision Making*, *Example: Distributed Builds*, *Example: Deciding Between Time and Scale*, *Revisiting Decisions, Making Mistakes*), *Software Engineering Versus Programming*, *Conclusion*, and *TL;DRs*.
  - **How to use it:** Focus on the Google-scale case studies (hash ordering, compiler upgrades, markers, distributed builds) to reinforce and stress-test the concepts taught in Sections 1–5.

### Reference Skim (~5 min)
- **IEEE Computer Society**, *Guide to the Software Engineering Body of Knowledge (SWEBOK V4.0)* (2024):
  - **Direct Link:** [https://www.computer.org/education/bodies-of-knowledge/software-engineering-swebok](https://www.computer.org/education/bodies-of-knowledge/software-engineering-swebok)
  - **Exact Scope:** Overview and the 18 Knowledge Areas list only.

### Optional Supplementary Reading (Outside the 3-Hour Budget)
- **Mary Shaw**, *"Prospects for an Engineering Discipline of Software"*, *IEEE Software*, Vol. 7, No. 6, pp. 15–24 (November 1990): [CMU Figshare Archive](https://doi.org/10.1184/R1/6608717) | [IEEE DOI 10.1109/52.60586](https://doi.org/10.1109/52.60586) — Historical perspective on software's maturation from craft to engineering discipline.
