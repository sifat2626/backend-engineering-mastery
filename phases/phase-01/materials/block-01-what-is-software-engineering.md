# Phase 1, Block 1 — What Is Software Engineering?

Suppose a small online store asks you to add a promotional code: when a customer types `SAVE10` at checkout, take 10% off the order total.

On Day 1, this feels almost trivial. You write a ten-line JavaScript function that checks whether the code equals `"SAVE10"`, multiplies the total by `0.9`, and returns the discounted price. You test it with a 100-dollar order, see 90 dollars come back, and finish before lunch.

Two weeks later, customers need to redeem `SAVE10` from both the website and the mobile app. A standalone function on your laptop cannot serve live shoppers, so you wrap it inside a **backend service**—a program running continuously on a server that receives requests from client apps over a network and reads or writes saved data in a database. You store active promo codes in a database, wire the service into checkout, test a normal order, and ship it. Real shoppers enter `SAVE10`, get their discount, and everyone celebrates.

Now jump forward fourteen months.

The engineer who originally wrote the promo feature has left the company. Order volume has grown tenfold. Marketing now asks for codes that expire at midnight, apply only to first-time buyers, or cannot be combined with other discounts. When a new developer opens the codebase to add expiration dates, they discover that discount assumptions are scattered across a dozen files, there are no automated tests to prove checkout still works after an edit, and nobody is quite sure which parts of the mobile app depend on the exact response format. A change that should take an afternoon turns into a tense, week-long gamble—where one slip could accidentally discount every order in the store by 100%.

Look closely at what happened. Nothing was broken on Day 1. Your ten-line function computed the right number. Even shipping the Week-2 checkout feature succeeded. What failed was everything that happened *after* Day 1.

That gap is what separates **programming** and **software development** from **software engineering**.

In job postings and everyday conversation, those three titles get used almost interchangeably, and there is no single universal international standard dividing them. Still, blurring the *activities* behind them causes real production failures. Grounded in Titus Winters, Tom Manshreck, and Hyrum Wright (*Software Engineering at Google*, 2020) and IEEE's software engineering body of knowledge, we use a sharp working distinction throughout this program:

| Lens | Core Focus | Driving Question | What Breaks If You Stop Here |
| :--- | :--- | :--- | :--- |
| **Programming** | Turning a clearly defined task into working code. | *"Does this code run and produce the right answer right now?"* | Fails as soon as requirements, callers, or data change. |
| **Software Development** | Building, integrating, and shipping features to users. | *"Does this feature work and get out the door?"* | Accumulates fragile contracts, upgrade debt, and operational blind spots. |
| **Software Engineering** | Sustaining, verifying, evolving, and operating software reliably across **time**, **scale**, and **competing constraints**. | *"Can a changing team safely modify, upgrade, verify, and operate this system over its expected lifespan?"* | *Its own failure mode:* applying heavy engineering rigor to tiny, disposable tasks. |

Notice what software engineering is *not*. Many developers assume software engineering simply means writing cleaner, prettier syntax or knowing more frameworks. Readable code matters—it makes maintenance easier—but spotless variable names cannot save a checkout service that nobody can safely test, upgrade, or modify a year later. Software engineering does not replace programming; you still have to program well. It recognizes that **initial authoring is usually the shortest chapter in a production system's life.**

That raises an immediate pushback. *Wait,* you might ask. *What if marketing never asked for a single new discount rule? If we simply left our working Week-2 promo service completely untouched, wouldn't it keep working safely forever?*

---

## Why Untouched Software Still Breaks

Software does not rust or wear out physically. Yet untouched production services break all the time. The reason is simple: **your code does not live in isolation, and the environment surrounding it never stops moving.**

Even our small `SAVE10` backend sits on top of a living stack. Your JavaScript code runs inside a **runtime**—in our primary stack, Node.js, the runtime environment that executes server-side JavaScript using the V8 JavaScript engine and provides APIs for server-side work. It leans on external libraries written by other people (**dependencies**, such as packages installed via `npm`), reads and writes through a **database** (such as PostgreSQL), and communicates with client apps over the internet.

Now imagine your team commits zero lines of code to the promo service for eighteen months. Watch what still changes around it:

- **Security vulnerabilities are discovered.** Researchers routinely uncover flaws in runtimes and widely used libraries. Many publicly disclosed vulnerabilities receive **CVE** identifiers *(Common Vulnerabilities and Exposures)*, which provide a common way to identify them, and maintainers publish **security patches** to close them (we will study security mechanisms deeply in Phase 19). Untouched code stays exposed to newly discovered attacks.
- **Underlying platforms age out.** Every major runtime version—even Long-Term Support (**LTS**) releases—eventually reaches **End-of-Life (EOL)**, the point where maintainers stop releasing security fixes. Databases and cloud platforms likewise phase out old behaviors and retire older versions.
- **Data and traffic grow.** Looking through 50 promo records takes a fraction of a millisecond; scanning 5 million redemptions slows checkout noticeably (we will learn how to reason about growth rates in Phase 2 and database indexing in Phase 10).
- **The outside world shifts.** Tax laws, payment processors, mobile operating systems, and partner integrations all evolve whether your repository changes or not.

Because external change is guaranteed for any long-lived service, avoiding maintenance creates a predictable trap—the **deferral spiral**. Suppose at Month 6, a routine Node.js and package upgrade would take four hours, so the team skips it because *"nothing is broken right now."* By Month 18, five separate breaking changes across Node.js and dozens of libraries have stacked on top of one another, original authors have moved on, and test coverage is thin. By Month 24, a critical vulnerability hits an EOL runtime—one that could allow attackers to run unauthorized code on your server. Now an upgrade is no longer optional, yet performing it requires weeks of high-risk surgery under emergency pressure while production sits exposed.

This is why Winters et al. (2020) define software engineering as **"programming integrated over time"** and introduce a practical working definition of **software sustainability**: a codebase and organization are sustainable when they can safely react to every valuable technical or business change needed over the software's **expected lifespan**. *(Note that this refers strictly to technical and organizational lifecycle sustainability, distinct from environmental sustainability.)*

A service that runs with zero bug tickets today—yet terrifies every engineer who considers modifying or upgrading it—is already failing as an engineered system. Notice, though, that *"if it ain't broke, don't touch it"* is not *always* wrong: if a small internal script is isolated and scheduled to be thrown away next week before any security or upgrade risk can arrive, leaving it untouched is sensible. Expected **lifespan** changes the engineering decision.

> **Pause & Check 1** *(Write brief answers in `phases/phase-01/notes/block-01-notes.md` before checking the Self-Check Guidance at the end.)*
> 1. Why can a backend service be *unsustainable* even if it has zero open bug tickets and 100% uptime this month?
> 2. A teammate says: *"Don't touch our login service's dependencies—if it ain't broke, don't fix it."* Trace the cause-and-effect flaw in that argument, and name one realistic situation where leaving working code untouched *would* be reasonable.

Accepting that long-lived software *must* change leads straight into an unsettling puzzle. Suppose you do the responsible thing: you keep the promo service maintainable and make a careful, completely valid internal improvement that obeys every single word of your documentation. **How could a fully legal change still break live customers?**

---

## When a "Safe" Change Breaks Users Anyway

Picture a small shopkeeper who tosses each day's paper receipts into a shoebox. The shopkeeper never promised the receipts were kept in chronological order—only that every receipt is in the box. Still, the bookkeeper notices that the newest receipt always happens to land on top, so she builds a daily habit of grabbing the top sheet. One afternoon, the shopkeeper neatly reorganizes the receipts alphabetically into a binder so customer lookups run faster. Not a single promise was broken, and every receipt is still there—yet the bookkeeper's routine silently breaks the next morning.

The exact same failure happens constantly in backend systems.

Return to our growing promo-code service. Suppose it exposes an **API** *(Application Programming Interface — the boundary through which client apps and partner programs send requests and receive responses)* with two behaviors: one endpoint lists a customer's currently eligible promo codes, and another redeems a code—returning an error message if redemption fails. Your documentation makes only two promises: *"Returns a list of the customer's eligible promo codes; returns an error message if a code cannot be redeemed."*

Notice two details you never promised: any particular sort order for the list, and exact English wording for error messages.

For a year, because new promo codes happened to be stored one after another, the list consistently came back oldest-first and newest-last, and expired codes consistently returned the text `"Code expired"`. Neither the mobile app team nor an external partner read your mind; they tested against what your service *actually returned*. Quietly, the mobile app grabbed `codes[codes.length - 1]` to showcase *"Newest Offer"* on the home screen and checked `if (error.message === "Code expired")` to trigger a renewal banner. Both worked flawlessly in testing and production.

At Month 16, a backend engineer optimizes how eligible codes are fetched—changing the internal lookup order—and polishes the error text to `"This promotional code has expired."` Every written promise in the documentation still holds. Yet overnight, mobile shoppers see a year-old promotion labeled *"Newest Offer,"* and the expired-code banner vanishes completely.

Hyrum Wright captured this exact empirical phenomenon in *Software Engineering at Google* (2020), and it is now widely known as **Hyrum's Law**:

> *"With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody."*

Notice that Hyrum's Law is not caused by "bad" or "lazy" client developers. It follows from a clear four-step mechanism across any boundary:

1. **Callers observe concrete behavior, not your internal intent.** When another system integrates with yours, it sees what comes back on real requests.
2. **Unconstrained degrees of freedom leak out.** Whenever you leave sort order, error wording, timing, or extra fields unconstrained, whatever your current implementation happens to emit becomes visible.
3. **Client code and tests harden around those observations.** Downstream engineers write logic and automated assertions matching what consistently works.
4. **Legitimate internal changes collide with hidden consumers.** Months later, a valid internal improvement alters an accidental behavior and breaks callers you never knew depended on it.

Once you see the mechanism, three practical engineering countermeasures follow directly:

- **Close accidental degrees of freedom explicitly.** If an endpoint returns a list of eligible promo codes, choose an intentional product ordering policy and enforce it explicitly—such as newest-created offer first (`created_at DESC`) with a stable ID as a tie-breaker—and document it rather than letting storage layout decide by accident.
- **Expose structured contracts, not raw prose or internals.** Return stable machine-readable error codes—such as `{ "code": "PROMO_EXPIRED", "message": "This promotional code has expired." }`—so clients branch on `PROMO_EXPIRED` instead of brittle English text or raw database errors.
- **Map output fields explicitly.** Choose deliberately which fields cross the API boundary rather than forwarding entire raw internal records, so adding an internal field tomorrow does not silently reshape responses.

Be careful, however, not to turn Hyrum's Law into a superstition. It has two clear limits. First, it is **scale-dependent**: it requires *"a sufficient number of users."* A private helper function called in two places inside a single repository you control does not ossify like a public API used by 400 external partners. Second, it is **not a rule that you can never change observable behavior**. It means you must *expect* hidden dependencies, measure real usage where possible, give callers advance notice before retiring old behavior, and decide deliberately whether the benefit of a change outweighs the migration pain. Arguing *"we never documented that order, so it's the client's fault"* fails in production—because broken paying customers are still your outage.

So far, we have watched one or two engineers manage a small promo service over time. Now widen the lens: **what happens when the store succeeds and the engineering team grows from 2 engineers and 3 endpoints to 30 engineers and 200 endpoints?**

---

## When Growth Breaks Old Habits—and Why Timing Matters

Software engineering has to survive not only **time**, but also **scale**—growth in codebase size, number of engineers, number of services, user traffic, and external dependencies. Practices that feel effortless at small scale often collapse completely when a team grows.

Imagine a simple team rule when two engineers maintain our promo service: *"Whenever you change our shared discount-validation helper, message the other engineer and update the three endpoints that call it."* At 2 engineers and 3 endpoints, that takes ten minutes—completely reasonable.

Fast-forward to 30 engineers owning 200 endpoints across checkout, billing, subscriptions, and mobile backends. Now that same manual rule requires tracking down ten teams, sending dozens of messages, waiting weeks for replies, and chasing whoever missed the notification. Old and new validation behaviors end up running side by side in production, and every subsequent upgrade becomes slower still.

Look at the math behind why that broke. When the human effort of a recurring policy grows in direct lockstep with codebase size or team headcount—twice the endpoints means twice the manual work (**linear** growth), or worse, every team must coordinate with every other team (**super-linear** growth; we will study growth curves formally in Phase 2)—human capacity eventually saturates. For **high-frequency, broad-scope mechanical policies** like code formatting, running automated tests, checking dependencies for known vulnerabilities, or performing routine deployments, engineers invest in automated tooling so human effort grows much more slowly (**sub-linearly**) as the system expands.

That principle carries a vital boundary: **do not assume every engineering activity can or should be automated to scale sub-linearly.** High-judgment work—framing ambiguous business goals, debating architectural trade-offs, diagnosing unfamiliar production outages, and mentoring engineers—inherently demands human thought. Good engineering automates repetitive mechanical work precisely to preserve scarce human judgment for decisions that genuinely require it.

That raises a sharp follow-up question about **timing**. Once a team invests in design discussions, automated tests, and peer reviews, **when** in the life of a change should flaws be caught?

Every software change moves through stages from left to right:

```text
Problem Framing & Design → Coding → Automated Checks & Tests → Code Review → Staging → Production
```

In plain terms, **problem framing and design** is deciding what the system should do before typing; **coding** is writing the implementation; **automated checks and tests** are programs that run automatically on every change to verify expected behavior; **code review** is another engineer inspecting the change; **staging** is a pre-production copy of the system where changes are exercised safely before real users interact with them (covered further in Phase 24); and **production** is the live system serving actual customers.

Now trace one subtle defect through both ends of that pipeline: suppose an engineer forgets to prevent customers from applying `SAVE10` multiple times to the same shopping cart.

- If someone asks *"Can one cart stack the same promo code twice?"* during **problem framing** or catches it in a local **automated test**, fixing it takes five minutes. The mental model is fresh in the author's head, zero customers have seen it, and no bad order data exists.
- If that same bug reaches **production** on Black Friday, thousands of shoppers check out with stacked discounts. Engineers are paged in an emergency, checkout has to be halted or patched under pressure, finance has to audit corrupted orders, and customer trust takes a hit.

The scope of users, data, money, or downstream systems harmed when a flaw escapes is its **blast radius**. Deliberately moving defect detection, validation, and design critique earlier in this lifecycle—where context is fresh, blast radius is near zero, and emergency coordination never starts—is called **shifting left**.

Just like Hyrum's Law, shifting left is easy to misunderstand if stated as dogma. Many articles claim that fixing a bug in production is *"always exponentially (10× to 100×) more expensive"* than catching it in design. Treat that as a **directional heuristic, not a universal empirical law**. Real repair cost depends heavily on reversibility, blast radius, and deployment speed: fixing a minor wording typo on a checkout button behind fast automated deployments might take three minutes in production, whereas repairing a corrupted billing ledger after a flawed design decision can take months. Just as importantly, shifting left can go too far—demanding a 15-page formal specification and three approval committees for a tiny, easily reversible experiment slows learning and wastes engineering time. **Shift left proportionally to blast radius and irreversibility.**

> **Pause & Check 2** *(Write brief answers in `phases/phase-01/notes/block-01-notes.md` before checking the Self-Check Guidance at the end.)*
> 1. Give one example of a recurring backend policy that breaks down as a team grows from 2 to 30 engineers, and one engineering activity that cannot simply be automated to scale sub-linearly.
> 2. Why is it inaccurate to claim that catching a defect earlier is *always* exponentially cheaper? Contrast one small issue that is cheap to fix after release with one flaw where catching it early avoids massive damage.

Pause right here and notice a tension we have built up across the last three sections. Explicit contracts, steady dependency upgrades, automated tooling, and shifting verification left all prevent real production failures. **Why, then, shouldn't an engineer apply maximum engineering rigor to every script and feature they ever write?**

---

## Why Maximum Rigor Everywhere Is Also Bad Engineering

The answer is straightforward: **every engineering safeguard spends finite resources—and cash on a cloud invoice is only one of them.**

Consider two real dilemmas teams face all the time.

First, imagine a small startup running our store's backend that decides to save $300/month in cloud fees by self-managing PostgreSQL on raw servers instead of paying a cloud provider for managed backups, monitoring, and patching. Six months later, a senior engineer spends 15 hours every month handling alerts, testing backups, and applying server updates—hours pulled directly away from fixing real checkout bugs and shipping customer features.

Second, imagine an engineer building our store's first `SAVE10` code who decides to spend three weeks constructing an elaborate, generic campaign-rules engine *"just in case marketing needs 500 dynamic rule types next year."* Meanwhile, the store misses a seasonal launch window while a competitor captures those shoppers.

In both dilemmas, the team looked at only one surface variable—either the monthly cloud bill or theoretical future elegance—while ignoring the hidden costs actually paid. Winters et al. (2020) organize real engineering decisions across **six distinct cost dimensions**:

| Cost Dimension | What It Actually Measures | Concrete Backend Example |
| :--- | :--- | :--- |
| **Financial** | Direct money spent (cloud hosting, licenses, third-party services). | Paying for a managed PostgreSQL database vs. renting cheaper raw servers. |
| **Resource** | Compute, memory, storage, and network bandwidth consumed. | Returning all 50,000 of a customer's past orders at once instead of a small page at a time increases database work and network traffic. |
| **Personnel** | Engineering time spent building, reviewing, debugging, and operating. | Self-managing a database burns 15 engineering hours/month in backups, patching, and alerts. |
| **Transaction** | Effort and risk required to *perform* a change or migration. | Renaming a live API field used by external apps requires advance warnings, temporary compatibility, and coordinated rollouts. |
| **Opportunity** | Valuable product or reliability work *not* done because capacity was spent here. | Spending three weeks on a speculative promo engine instead of fixing broken checkout edge cases. |
| **Societal / User** | Real-world harm to users (lost money, exposed personal data, downtime). | Skipping dependency security patches or validation exposes customer accounts or causes wrong charges. |

Because the inputs to these six costs—team size, traffic, revenue, and regulatory stakes—change over time, **good engineering decisions have expiration dates.** Running one straightforward Node.js service and one PostgreSQL database for 200 users is usually smart engineering today. Treating that choice as sacred forever—or conversely, adopting complex multi-service infrastructure at 200 users because giant tech companies use it—ignores the real inputs. Sound engineering revisits decisions when measured evidence shows the inputs have shifted.

To see why calibrating rigor matters, step intentionally outside our promo service and compare two backend tasks at opposite extremes:

- **High rigor is essential:** A public business-to-business billing API integrated by 400 paying companies over an expected five-year lifespan. If it breaks, partners suffer duplicate charges, broken invoicing pipelines, and legal liability. Here, explicit versioned contracts, deterministic ordering, structured error codes, automated tests, steady dependency upgrades, and careful reviews are worth every hour invested. Under-engineering this system guarantees serious damage.
- **Heavy rigor is harmful overhead:** A 25-line local Node.js script written once by one engineer to remove stray whitespace from a single historical CSV file before importing it this afternoon. If it misformats a row, the engineer spots it on a test copy, fixes line 12, and re-runs before importing. Spending two days building modular abstractions, automated deployment pipelines, and formal versioning for a script discarded at 5:00 PM wastes personnel and opportunity cost without meaningfully reducing risk. Over-engineering this task is *also* a failure of software engineering.

Engineering judgment is never *"maximize process everywhere."* It is matching rigor proportionally to **expected lifespan**, **scale**, and **blast radius**.

> **Pause & Check 3** *(Write brief answers in `phases/phase-01/notes/block-01-notes.md` before checking the Self-Check Guidance at the end.)*
> 1. In the startup example above (saving $300/month by self-managing its database while spending ~15 senior engineering hours/month on patches, backups, and alerts), which cost dimensions were traded off, and why can this be a net loss?
> 2. Why is over-engineering a 25-line throwaway CSV script a failure of *software engineering*, rather than just "being extra thorough"?

---

## Why Writing Code Is Only One Piece of the Discipline—and Where We Go Next

Pause and look back across everything our single `SAVE10` feature just forced us to think through.

Writing the ten lines of JavaScript that took 10% off an order took minutes. Every real engineering challenge lived *around* those ten lines: understanding how requirements evolve over time, managing dependencies and security patches, guarding implicit API contracts against Hyrum's Law, scaling team coordination without crushing human judgment, shifting checks left proportionally to blast radius, and balancing six competing costs across a system's lifespan. Code never runs in a purely technical vacuum; it operates inside wider **system boundaries** shaped by users, organizations, operations, and economics.

That observation is reflected directly in the IEEE Computer Society's ***Guide to the Software Engineering Body of Knowledge* (SWEBOK Guide V4.0, 2024; updated as V4.0a in 2025)**, which organizes the formal discipline of software engineering into **18 Knowledge Areas** (adding *Software Architecture*, *Software Engineering Operations*, and *Software Security* in its fourth edition), grouped below into five intuitive clusters for learning rather than official SWEBOK categories:

- **Understanding & Designing What to Build:** Software Requirements, Software Architecture *(V4)*, Software Design, Software Engineering Models and Methods.
- **Writing the Code (1 of 18):** **Software Construction**.
- **Verifying & Protecting It:** Software Testing, Software Quality, Software Security *(V4)*.
- **Operating & Sustaining It Over Time:** Software Engineering Operations *(V4)*, Software Maintenance, Software Configuration Management.
- **Coordinating, Measuring & Foundations:** Software Engineering Management, Software Engineering Process, Software Engineering Economics, Software Engineering Professional Practice, Computing Foundations, Mathematical Foundations, Engineering Foundations.

Look at that middle bullet: **writing code (*Software Construction*) is just one of eighteen knowledge areas.**

That fact explains a central reality of modern backend engineering. AI coding assistants can now generate syntax in seconds—dramatically lowering the keystroke cost of *Software Construction*. Yet speeding up code generation does not solve the other seventeen areas; if anything, it makes them *more* decisive. Generating 400 lines of promo-code logic in ten seconds creates risk faster unless an engineer can verify whether it solves the right problem, avoids leaking fragile contracts, stays maintainable next year, and respects real security and cost constraints. An engineer who cannot judge what happens to generated code after Day 1 is still operating only at the programming layer (we will explore AI-augmented engineering deeply in Block 7).

And look closely at where almost every headache in our `SAVE10` journey began. We jumped straight into coding `SAVE10` before asking what happens when a code expires, gets reused twice, encounters a refund, or collides with another discount. Before an engineer designs a contract, picks a trade-off, or writes a single line of code, they have to interrogate the problem itself—exposing hidden assumptions, ambiguities, and missing rules while changing direction is still cheap.

That is why **Block 2: Understanding the Problem Before Coding** comes next.

---

## Study Plan, Learning Objectives & Mandatory Reading

> **Continuity & Ownership Notes:** Phase 1 Diagnostic is already `completed`—do **not** repeat it. Record all inline checks, exercises, and closed-book assessment answers independently in `phases/phase-01/notes/block-01-notes.md`. Reading this lesson does **not** mark Block 1 `completed` or `approved`, and AI preparation time does **not** count toward your study hours.

### Target 3-Hour (180-Minute) Study Budget

| Stage | Activity | Target Time |
| :---: | :--- | :---: |
| **1** | Read the guided lesson above + complete Pause & Checks 1–3 in `notes/block-01-notes.md` | 65 min |
| **2** | Complementary Mandatory Reading (*SWE at Google* Ch. 1 + SWEBOK V4.0 18-KA skim) | 35 min |
| **3** | Complete Practical Reasoning Exercises 1–3 independently in `notes/block-01-notes.md` | 55 min |
| **4** | Answer Mastery Assessment Questions 1–5 closed-book → submit to ChatGPT | 25 min |
| **Total** | — | **180 min (3 h)** |

### Block 1 Learning Objectives *(Primary: Outcome 1; Early Bridge: Outcome 10)*

- **LO 1.1 (`Working Understanding`):** Distinguish **programming**, **software development**, and **software engineering** operationally, recognizing this three-way split as a course working distinction rather than a universal ISO taxonomy.
- **LO 1.2 (`Working Understanding`):** Trace how **time and change**, **scale and efficiency**, and **trade-offs and costs** change engineering decisions across a system's lifespan.
- **LO 1.3 (`Working Understanding`):** Explain the mechanisms, consequences, and limits of **Hyrum's Law** and **shifting left** in long-lived backend systems *(consolidated via Exercise 1)*.
- **LO 1.4 (`Awareness`):** Situate code construction within the **18 Knowledge Areas of IEEE SWEBOK Guide V4.0 (2024; updated as V4.0a in 2025)** and explain why syntax speed alone is not software engineering.

### Complementary Authoritative Reading

- **Mandatory (~30 min — Read After the Guided Lesson):** Titus Winters, Tom Manshreck, Hyrum Wright, *Software Engineering at Google* (O'Reilly, 2020; free HTML edition) — [Chapter 1: What Is Software Engineering?](https://abseil.io/resources/swe-book/html/ch01.html). Read the full chapter (*Time and Change* incl. Hyrum's Law and hash ordering; *Scale and Efficiency* incl. policies that don't/do scale, compiler upgrade, Shifting Left; *Trade-offs and Costs* incl. markers, inputs to decision making, distributed builds, deciding between time and scale, revisiting decisions; *Software Engineering Versus Programming*; *Conclusion*; and *TL;DRs*). Use its Google-scale case studies to stress-test the mental models built above.
- **Reference Skim (~5 min):** IEEE Computer Society, *Guide to the Software Engineering Body of Knowledge (SWEBOK Guide V4.0, 2024; updated as V4.0a in 2025)* — [Official SWEBOK Portal](https://www.computer.org/education/bodies-of-knowledge/software-engineering-swebok) (Introduction and the 18 Knowledge Areas list only; do not read individual chapters in Phase 1).
- **Optional Supplementary (Outside the 3-Hour Budget):** Mary Shaw, *"Prospects for an Engineering Discipline of Software"*, *IEEE Software*, Vol. 7, No. 6, pp. 15–24 (Nov 1990) — [CMU Figshare](https://doi.org/10.1184/R1/6608717) | [IEEE DOI](https://doi.org/10.1109/52.60586).

---

## Practical Reasoning Exercises (~55 Minutes Total)

> **Ownership Rule:** Write all responses independently in `phases/phase-01/notes/block-01-notes.md`. Do **not** ask AI to draft your answers.

### Exercise 1: Hyrum's Law, Implicit Contracts & Proportional Shifting Left (~20 minutes)

**Scenario:**
Your team maintains a Node.js + PostgreSQL backend exposing an HTTP endpoint `GET /api/orders` *(which returns a customer's orders formatted as JSON—standard structured text data)*. The API documentation states only: *"Returns a list of the authenticated user's orders."* Internally, the SQL query retrieves matching orders without an `ORDER BY` clause *(in SQL and PostgreSQL, a `SELECT` query without `ORDER BY` makes no guarantee about the order in which rows are returned)*.

For 18 months, PostgreSQL happened to return rows in creation order, so both your mobile app and two enterprise partners quietly relied on `orders[0]` being the oldest order and `orders[orders.length - 1]` being the newest. Later, to speed up slow lookups as the table grows, an engineer adds a **database index** *(an extra lookup structure that helps the database find matching rows faster—we will study indexing internals in Phase 10)*. After the index is added, PostgreSQL may choose a different query plan and return rows in a different observed order; in this incident, it does. Mobile shoppers now see old cancelled orders at the top of their screen, and a partner's billing script pulls the wrong invoice—not because PostgreSQL violated a promise, but because the query never guaranteed an order in the first place.

**Tasks:**
1. **Cause-and-Effect Diagnosis:** Trace the chain of events connecting the Day-1 omission, Hyrum's Law, and the Day-540 outage.
2. **Programmer vs. Software Engineer Response:** Contrast a point-in-time programmer's reaction (*"We never documented an order; clients must fix their code"*) with a software engineer's immediate mitigation and long-term contract fix.
3. **Proportional Shift Left:** Identify **two** earlier lifecycle checkpoints where this risk could have been caught cheaply—and name **one** heavyweight safeguard that would have been unnecessary overhead for an early minimal product.

---

### Exercise 2: Calibrating Engineering Rigor Across Four Backend Artifacts (~20 minutes)

*(Treat 20 minutes as a rough target rather than a hard limit—prioritize clear engineering reasoning over rushing.)*

Evaluate these four distinct backend artifacts:
- **Artifact A:** A 30-line Node.js script run once locally to clean duplicate whitespace in one historical CSV file before a single import today.
- **Artifact B:** An internal nightly **cron job** *(a scheduled background task run automatically on a timer)* used by three operations staff to email a summary of failed shipments each morning.
- **Artifact C:** A public business-to-business billing **webhook** *(an automated HTTP notification sent to partner systems when an event occurs)* and invoice API integrated by 400 external paying companies.
- **Artifact D:** The company's central authentication and **session-validation** *(checking whether each incoming request belongs to a valid logged-in user)* service called on every backend request.

**Tasks:**
For **each** artifact (A–D):
1. Estimate its expected **lifespan** and **blast radius** if it fails.
2. Specify the appropriate level of engineering rigor (testing, contracts/versioning, upgrade policy, review depth).
3. Give **one concrete failure of under-engineering** AND **one concrete failure of over-engineering** for that specific artifact.

---

### Exercise 3: Multi-Dimensional Cost & Sustainability Analysis (~15 minutes)

**Scenario:**
A 3-engineer backend team runs a production Node.js API on a version reaching End-of-Life (EOL) in 4 months. Upgrading now takes about 1.5 engineering days (checking dependencies, verifying automated tests, and doing a **staged rollout**—releasing the update to a small portion of traffic before everyone else). A manager proposes: *"Skip the upgrade for two years so we ship features faster—nothing is broken right now."*

**Tasks:**
1. Analyze the manager's proposal across **at least four** of Winters et al.'s six cost dimensions (financial, resource, personnel, transaction, opportunity, societal/user).
2. Explain the cause-and-effect impact of a two-year deferral on **software sustainability**.
3. Describe **one realistic scenario** where deferring the upgrade for a short, bounded window (such as 3 weeks) *would* be a sound engineering trade-off—and specify the exact condition or commitment needed so it does not turn into indefinite deferral.

---

## Closed-Book Mastery Assessment (~25 Minutes)

Close this lesson and external tabs. Write clear, original answers to all five questions in `phases/phase-01/notes/block-01-notes.md`, then submit your work to **ChatGPT (Curriculum Owner & Instructor)** for evaluation:

1. **(LO 1.1 / Outcome 1)** Using a concrete backend example, explain the operational difference between **programming**, **software development**, and **software engineering**. Why do we treat this three-way split as a course working distinction rather than a universally standardized ISO taxonomy?
2. **(LO 1.2 / Outcome 1)** Explain how **time and change**, **scale and efficiency**, and **trade-offs and costs** each force decisions that do not arise in a single-use program.
3. **(LO 1.3)** Explain how **Hyrum's Law** works mechanistically, why **shifting left** reduces many (though not universally all) defect costs, and where each concept has realistic limits.
4. **(LO 1.2 / Outcome 10)** Define **software sustainability** as used in this program and explain why *"working code nobody dares to touch"* represents an engineering failure with real consequences.
5. **(LO 1.4)** Referring to SWEBOK V4.0's breadth (18 Knowledge Areas), explain why generating code faster—whether by hand or with AI—does not by itself solve software engineering.

### Block 1 Readiness Checklist (Before Moving to Block 2)
- [ ] Studied the guided lesson and completed Pause & Checks 1–3 (checking Self-Check Guidance afterward).
- [ ] Completed complementary mandatory reading (*Software Engineering at Google* Ch. 1) and 5-minute SWEBOK V4.0 KA skim.
- [ ] Independently wrote answers to Exercises 1–3 in `phases/phase-01/notes/block-01-notes.md`.
- [ ] Independently answered Mastery Assessment Questions 1–5 closed-book.
- [ ] Submitted Block 1 work to ChatGPT for feedback and confirmation before starting Block 2.

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *Check these only **after** writing your own answers in `notes/block-01-notes.md`. No solutions are provided for the Practical Reasoning Exercises or Closed-Book Mastery Assessment—those remain learner-owned for ChatGPT review.*

- **Pause & Check 1:**
  1. *Strong answer includes:* External runtimes, libraries, security vulnerabilities, data volume, and business needs keep evolving even when your repository is untouched. If the team cannot safely modify or upgrade the service when those external changes arrive, the system is unsustainable despite zero current bug tickets.
  2. *Strong answer includes:* Skipping dependency updates on a long-lived login service allows CVEs and breaking changes across versions to pile up until a mandatory upgrade becomes a high-risk emergency—or leaves users exposed. Leaving working code untouched *is* reasonable for a genuinely short-lived, isolated artifact scheduled for retirement before EOL or security exposure, or when avoiding cosmetic rewrites that add risk without lifecycle value.
- **Pause & Check 2:**
  1. *Strong answer includes:* Manual cross-team coordination rules (such as messaging every team individually to update a shared helper or manually running checks on every laptop) saturate human capacity as headcount grows. High-judgment activities—framing ambiguous problems, weighing architectural trade-offs, investigating unfamiliar outages, and mentoring—cannot simply be automated to scale sub-linearly.
  2. *Strong answer includes:* Actual fix cost depends on blast radius, reversibility, and deployment speed rather than a universal exponential formula. Fixing a small wording typo behind fast automated deployments is cheap post-release; repairing corrupted billing records or breaking 400 external integrations from a flawed API or schema design is far costlier after release than during design.
- **Pause & Check 3:**
  1. *Strong answer includes:* The startup lowered direct **financial** cost ($300/month) by significantly increasing **personnel** cost (15 senior hours/month) and **opportunity** cost (checkout bugs and features left undone), while also risking **societal/user** cost if manual patching or backup errors cause data loss or exposure.
  2. *Strong answer includes:* Software engineering matches rigor to lifespan, scale, and blast radius under real constraints. Spending days adding elaborate abstractions, pipelines, and versioning to a script that runs once and is discarded wastes personnel and opportunity cost without reducing real risk.
