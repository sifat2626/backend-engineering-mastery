# Phase 1, Block 7 — AI-Augmented Engineering

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `notes/block-07-notes.md`-এ Pause & Checks 1–3 লেখা | **~45–60 min** *(flexible — limit নয়)* |
| **Mandatory 1:** DORA 2024 *Accelerate State of DevOps Report* | কেবল AI findings summary *(adoption, productivity/flow vs. delivery-performance trade-offs, 39.2% low/no trust)* | ~15 min |
| **Mandatory 2:** *2025 DORA Report: State of AI-assisted Software Development* | কেবল executive findings (*"AI is an amplifier"* thesis) | ~15 min |
| **Mandatory 3:** *DORA AI Capabilities Model* (2025 Companion Report) | ৭টি foundational organizational capabilities-এর overview | ~10 min |
| **Mandatory 4:** Perry et al. (Stanford, ACM CCS 2023) | Abstract, key security & overconfidence findings, এবং critical-engagement takeaway *(পুরো paper নয়)* | ~20 min |
| **Mandatory 5:** Spracklen et al. (USENIX Security 2025) | Abstract, quantitative findings, slopsquatting threat model, এবং mitigations summary *(পুরো paper নয়)* | ~20 min |
| **Mandatory 6:** Birgitta Böckeler (MartinFowler.com, 5 Feb 2026) | *Context Engineering for Coding Agents* — curating context, instructions vs. guidance/rules, structured context ও outer harness | ~25 min |
| **Exercise 1 (Mandatory):** AI Output Audit Lab (Node.js / SQL) | ~50-line AI-generated Node.js + SQL code independently audit করা, docs verify করা, failing tests লেখা ও fix করা | ~45–50 min |
| **Exercise 2 (Mandatory):** Delegation Boundary Exercise | ৬টি backend task classify করা (*Delegation*, *Collaborative Drafting*, *Human-Owned*) এবং reasoning লেখা | ~15–20 min |
| **Exercise 3 (Mandatory):** Context Packet Construction | Vague 1-line prompt বনাম structured context packet তুলনা করে objective observations নথিভুক্ত করা | ~20–25 min |
| **Total (Required Core — without Optional Assessment)** | Guided Lesson + ৬টি Mandatory Targeted Reading (~105 min) + ৩টি Mandatory Exercise (~80–95 min) | **~230–260 min (~3.8–4.3 h)** |
| **Optional Closed-Book Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি synthesis প্রশ্নের উত্তর লেখা | ~20–25 min *(optional)* |
| **Total (with Optional Assessment)** | Required Core + Optional Closed-Book Assessment | **~250–285 min (~4.2–4.75 h)** |

> **Planning Note:** Approved curriculum ও dossier-এ Block 7-এর allocation হলো **4 hours (240 minutes)**—যেটা একটি flexible planning estimate, কোনো hard deadline নয়। ৬টি mandatory reading-কে intentionally targeted sections-এ সীমাবদ্ধ রাখা হয়েছে (`~105 min` total), যাতে hands-on audit lab ও practical exercises-এর জন্য পর্যাপ্ত সময় থাকে (`~80–95 min`)। Guided lesson-এ কোনো concept গভীরভাবে বুঝতে বাড়তি সময় লাগলে তাড়াহুড়ো করবেন না; **Optional Closed-Book Assessment** সম্পূর্ণ optional এবং Block 8 শুরু করার mandatory gate নয়।

---

Block 6-এর শেষে আমরা Fred Brooks-এর (1987) একটা সতর্কবার্তা মনে করেছিলাম: কোনো *silver bullet* software-এর **essential complexity** মুছে দিতে পারে না। অথচ আজকের **AI coding assistants** code লেখাটাকে ইতিহাসের যেকোনো সময়ের চেয়ে বেশি **easy** করে তুলেছে।

এখন আমাদের সেই পরিচিত Node.js + PostgreSQL online store backend-এ ফিরে আসুন। Marketing আর Support team একটা নতুন feature চেয়েছে:

> *"কোনো customer paid order cancel করলে system যেন eligible item-গুলোর জন্য automatic store-credit refund ইস্যু করে, কিন্তু একই order-এ দুবার যেন credit না হয়।"*

একজন backend engineer তাঁর AI coding tool খুলে এক লাইনের একটা prompt দিলেন: *"Write a Node.js + PostgreSQL function to issue store credit when an order is cancelled and write tests for it."*

মাত্র কুড়ি সেকেন্ডের মধ্যে AI-টা ঝকঝকে, সুন্দর format করা ৮০ লাইন JavaScript code আর ৫টা unit test তৈরি করে দিল। Engineer test-গুলো চালালেন—৫টার ৫টাই green! Code-টা দেখতেও দারুণ familiar ও clean লাগল। তাঁর মনে হলো: *"দুর্দান্ত! যে কাজে আগে ৩ ঘণ্টা লাগত, AI দিয়ে ১০ মিনিটেই হয়ে গেল!"*

তিনি code-টা merge করে deploy করে দিলেন। এক সপ্তাহ পর production-এ তিনটি নীরব বিপর্যয় একসাথে ধরা পড়ল:
1. AI-টা `pg` library-র একটা nonexistent option বসিয়ে রেখেছিল যেটা runtime-এ চুপিসারে ignored হয়েছিল—ফলে যে database isolation তিনি ভেবেছিলেন চালু আছে, সেটা আদৌ চালু ছিল না।
2. একই customer দুর্ঘটনাবশত mobile app-এ `Cancel` button দুবার tap করায় দুটো concurrent request একই সাথে `"credit আগে দেওয়া হয়নি"` পড়ে নিয়েছিল এবং তাঁর wallet-এ **দ্বিগুণ store credit** জমা হয়ে গিয়েছিল!
3. আর AI-র লেখা ৫টা unit test-ই কেন পাস করেছিল? কারণ **যে ভুল assumption দিয়ে AI code লিখেছিল (single sequential happy path, mock database returning fixed rows), ঠিক সেই একই ভুল assumption দিয়েই সে test-গুলোও লিখেছিল!**

একটু থেমে দেখুন: AI এখানে খারাপ ছিল না—বরং শুরুর boilerplate, function skeleton, আর test structure draft করতে AI সত্যিই অনেক সময় বাঁচিয়েছিল। **তাহলে সমস্যাটা ঠিক কোথায় হলো?**

সমস্যাটা হলো, একজন modern backend engineer-এর সবচেয়ে গুরুত্বপূর্ণ প্রশ্নটা এখানে এড়িয়ে যাওয়া হয়েছিল:

> **AI coding tools যখন কয়েক সেকেন্ডেই fluent, plausible-looking code ও tests generate করতে পারে, তখন আমরা কীভাবে AI-র drafting speed কাজে লাগাব—অথচ hallucination, hidden security flaws, broken invariants, এবং overconfidence-এর ফাঁদে পড়ে production ভাঙা থেকে বাঁচব?**

---

## Delegation বনাম Abdication: কী AI-কে দেবেন, আর কী নিজে Own করবেন?

AI ব্যবহার নিয়ে developer-দের মধ্যে প্রায়ই দুটো extreme reaction দেখা যায়: একদল বলেন *"AI দিয়ে কাজ করা unsafe, আমি সব নিজে hand-type করব"*, আর আরেকদল বলেন *"AI তো সবই লিখে দেয়, আমার প্রতিটা লাইন বোঝার দরকার কী!"*—দুটোই দুর্বল engineering judgment।

একজন disciplined backend engineer যে সীমারেখাটা স্পষ্টভাবে টানেন, তা হলো **Delegation vs. Abdication**:

- **Delegation (সচেতন দায়িত্ববণ্টন):** আপনি AI-কে **drafting speed ও exploration**-এর কাজ দেন, কিন্তু **problem framing, constraints, invariants, architecture boundaries, verification, এবং final production ownership** সম্পূর্ণ নিজের হাতে রাখেন।
- **Abdication (দায়িত্ব ছেড়ে দেওয়া):** আপনি vague একটা request AI-কে দিয়ে তার output না বুঝে বা independently verify না করেই commit করে দেন—আর bug ধরা পড়লে ভাবেন *"AI-টা ভুল করেছে!"*

এই পার্থক্যটা চেনার সবচেয়ে সহজ বাস্তব নিয়ম হলো: **আপনি যে code commit করছেন, তার প্রতিটা লাইন কেন আছে, কোন edge case-এ সেটা ভাঙতে পারে, এবং রাত ৩টায় production incident হলে সেটা কীভাবে debug করবেন—তা যদি আপনি নিজে explain করতে না পারেন, তবে আপনি code-টা own করেন না।**

বাস্তব backend কাজে কোন কাজগুলো AI-কে delegate করা ভালো আর কোনগুলো human-owned থাকা বাধ্যতামূলক, তার পরিষ্কার বিভাজন নিচে দেখুন:

| Category | কী ধরনের Backend কাজ | আপনার Role কী থাকবে |
| :--- | :--- | :--- |
| **Good AI Delegation (with verification)** | Repetitive boilerplate (DTO/schema skeleton, CRUD scaffolding), test-case brainstorming, unfamiliar syntax বা API lookup-এর প্রথম ধারণা, tight spec দেওয়া small local refactor, অপরিচিত error message বা unfamiliar code-এর explanation *(as a starting hypothesis, never final truth!)*। | Output-টা line-by-line review করা, official docs দিয়ে verify করা, এবং independently execute করে test করা। |
| **Collaborative Drafting (human leads constraints)** | Pure business-logic helper draft করা, SQL query draft করা, বা existing design-এর ভেতর ছোট feature add করা—যেখানে আপনি আগে থেকেই exact invariants, types, non-goals, error contract, এবং edge cases define করে দিচ্ছেন। | Problem Space ও constraints আপনি lead করবেন; AI শুধু bounded implementation draft করবে; আপনি adversarial/negative cases verify করবেন। |
| **Human-Owned Decision (কখনোই delegate করবেন না)** | Ambiguous problem framing (Block 2), domain invariants ও data consistency boundaries (Block 3), constraint-driven architecture trade-offs ও ADRs (Block 4), security threat modeling ও authN/authZ boundaries, database transaction/concurrency design, production schema migration approval, এবং final production accountability। | AI-কে brainstorming partner বা second pair of eyes হিসেবে প্রশ্ন করতে পারেন, কিন্তু reasoning, decision ও responsibility শতভাগ আপনার। |

এখান থেকে একটা key distinction মাথায় গেঁথে রাখুন: **Speed of code generation ≠ speed of verified, maintainable delivery।** একটা 800-line unreviewed AI diff ২০ সেকেন্ডে তৈরি করে PR-এ dump করলে code generation খুব fast মনে হয়—কিন্তু reviewer-এর cognitive load (Block 6!), subtle bugs, rework, আর production rollback মিলে আসল delivery speed অনেক কমে যায়। উল্টোদিকে ছোট, verifiable batches + tight review + deterministic tests মিলিয়েই সত্যিকারের fast, reliable delivery হয়।

এখন একটা গভীর প্রশ্ন আসে: **AI-র লেখা code দেখতে এত confident, clean, আর syntactically polished হওয়ার পরও সেটি এত নিঃশব্দে ভুল হয় কেন—আর বাস্তব empirical research এই বিষয়ে কী প্রমাণ দেখিয়েছে?**

---

## AI-র Output দেখতে Convincing হওয়ার পরও কেন ভুল হয়—এবং Empirical Evidence কী বলে

এখানে কোনো ML model architecture বা training math-এ যাওয়ার দরকার নেই (সেগুলো আমাদের post-Phase 29 AI Engineering program-এর বিষয়)। একজন backend engineer হিসেবে আপনার শুধু একটা core behavioral truth বুঝতে হবে:

**Large Language Models (LLMs) হলো probabilistic pattern generators—deterministic formal verifiers বা runtime executors নয়।**

যখন আপনি AI-কে Node.js + PostgreSQL code লিখতে বলেন, তখন model-টা আপনার আসল database schema চালিয়ে দেখছে না, `npm` registry-তে live lookup করছে না (যদি না বাইরে থেকে tool দেওয়া হয়), বা concurrency interleaving prove করছে না। এটি training patterns ও আপনার দেওয়া context-এর ভিত্তিতে সবচেয়ে *plausible-looking next tokens* predict করে। ফলে একটা ভুল API option, একটা subtle SQL injection, বা একটা race condition-ও দেখতে একদম সিনিয়র engineer-এর লেখা clean code-এর মতোই confident লাগে!

বাস্তব backend কাজে এই probabilistic nature থেকে **ছয়টি concrete AI failure mechanism** নিয়মিত দেখা যায়:

1. **Confident Hallucination / Confabulation:** Plausible শোনায় কিন্তু বাস্তবে exists করে না—এমন API options, method flags, CLI arguments, বা fabricated documentation citations তৈরি করা। আমাদের store-credit উদাহরণে `pg`-র nonexistent option বসানো এর সরাসরি উদাহরণ।
2. **Package Hallucination & "Slopsquatting":** AI মাঝে মাঝে এমন `npm` বা `PyPI` package name বানিয়ে দেয় যেটা বাস্তবে কখনো তৈরিই হয়নি (যেমন আমাদের কাল্পনিক উদাহরণ `pg-idempotent-refund-guard`)। কোনো developer যাচাই না করেই `npm install` চালালে সেটা সাধারণত 404 দেয়—কিন্তু এখানে একটা ভয়ংকর supply-chain attack জন্মেছে, যার নাম **Slopsquatting**: attackers দেখে কোন fake package নামগুলো LLMs বারবার hallucinate করে, তারপর তারা নিজেরাই সেই নামগুলো `npm` বা `PyPI`-তে malicious code সহ register করে রাখে! পরের বার কোনো developer blindly AI-suggested `npm install` চালালেই malware তাঁর machine বা production server-এ ঢুকে পড়ে!
3. **Silent Security Vulnerabilities + Automation Bias & Overconfidence Trap:** SQL query-র বেশির ভাগ অংশ `$1, $2` দিয়ে parameterized রেখে কোনো dynamic expression বা identifier সরাসরি string-এ interpolate করে দেওয়া, missing ownership check, বা weak cryptographic defaults—এগুলো চট করে চোখে পড়ে না। আর মানুষের মনস্তাত্ত্বিক প্রবণতা হলো **Automation Bias**: একটি fluent, authoritative-looking automated system যখন উত্তর দেয়, তখন আমরা সেটাকে প্রয়োজনের চেয়ে বেশি trust করে ফেলি এবং আমাদের skepticism কমে যায়।
4. **Plausible-Looking Logic & Edge-Case Bugs:** Normal happy-path input-এ code একদম ঠিক কাজ করে, কিন্তু empty array `[]`, `null`, `0`, negative amount, UTC vs. local timezone boundary, failure চুপচাপ গিলে ফেলে safe-looking fallback value ফেরত দেওয়া, অথবা concurrent duplicate requests এলে চুপিসারে data corrupt করে।
5. **Subtle API / Version Drift:** Training data-তে বহু বছরের পুরোনো ও নতুন library versions একসাথে মিশে থাকে। ফলে AI অনায়াসে Node.js 16-এর deprecated API, পুরোনো framework signature, বা আপনার project-এর PostgreSQL version-এর সাথে অসামঞ্জস্যপূর্ণ syntax মিশিয়ে দিতে পারে।
6. **Systemic Amplification of Weak Engineering Practices:** একটি টিমে যদি ভালো tests, small batches, version control discipline, এবং code review না থাকে, তবে AI সেই দুর্বলতা ঠিক করে না—বরং আরও দ্রুত বড় বড় unreviewed changes ঢুকিয়ে পুরো delivery system-এর instability বাড়িয়ে দেয়!

এবার Block 5-এর **Evidence-Based Engineering** discipline প্রয়োগ করুন। ওপরের claims-গুলো শুধু opinion বা anecdote নয়—এগুলোর পেছনে গুরুত্বপূর্ণ **empirical research** আছে। তবে প্রতিটা study-কে আমরা তার exact experimental context ও limitations সহ দেখব—**study-specific findings-কে কখনোই universal industry rate হিসেবে inflate করব না!**

### ১. DORA 2024 & 2025 + DORA AI Capabilities Model *(Large-Scale Industry Surveys)*
- **DORA 2024 (*Accelerate State of DevOps Report*):** বড় পরিসরের industry survey-তে respondent-রা AI adoption-এর সাথে individual **productivity ও flow benefits** report করেছেন, কিন্তু একই সাথে software delivery performance-এ (throughput ও stability) negative trade-off-ও ধরা পড়েছে—অর্থাৎ *reported personal speed* আর *verified team delivery outcomes* এক জিনিস নয়! আরও উল্লেখযোগ্য finding: **39.2% of respondents reported little or no trust in AI**।
- **2025 DORA Report (*State of AI-assisted Software Development*):** এর মূল thesis হলো **"AI is an amplifier"**—AI ভাঙা engineering process-কে জাদুর মতো ঠিক করে না; বরং একটি organization-এর বিদ্যমান strength (ভালো tests, small batches, clear architecture) অথবা বিদ্যমান chaos—যেটাই থাকুক, সেটাকেই magnify করে!
- **DORA AI Capabilities Model (2025 Companion Report):** এই companion model-টি **৭টি foundational organizational capabilities** চিহ্নিত করে যেগুলো positive AI ROI enable করতে সাহায্য করে:
  1. *Clear and communicated AI stance*
  2. *Healthy data ecosystems*
  3. *AI-accessible internal data*
  4. *Strong version control practices*
  5. *Working in small batches*
  6. *User-centric focus*
  7. *Quality internal platforms*
  *(Evidence discipline: DORA হলো large-scale self-reported survey research—এটি organizational patterns দেখায়, প্রতিটা team বা tool-এর জন্য universal guarantee নয়। আর "Working in small batches"-এর মতো capability-গুলো হলো enabling organizational capabilities—এগুলোকে AI-induced batch-size inflation-এর direct প্রমাণ হিসেবে ভুল ব্যাখ্যা করবেন না।)*

### ২. Perry et al. (Stanford, ACM CCS 2023) — *Overconfidence Trap*
- **Study setup:** Neil Perry, Megha Srivastava, Deepak Kumar ও Dan Boneh একটি controlled user study পরিচালনা করেন, যেখানে participants-দের specific security-sensitive coding tasks দেওয়া হয়েছিল—এক দল OpenAI-এর **`codex-davinci-002`**-ভিত্তিক AI assistant ব্যবহার করেছে, অন্য দল করেনি।
- **Key empirical findings:** AI assistant ব্যবহার করা participants-রা control group-এর তুলনায় **significantly less secure code** লিখেছেন—এবং আরও ভয়ংকরভাবে, **তাঁরা অনেক বেশি confident ছিলেন যে তাঁদের code secure!** তবে একটা আশাব্যঞ্জক pattern-ও পাওয়া গেছে: যেসব participant AI-কে *কম trust* করেছেন এবং critically prompts ও output ঘেঁটে দেখেছেন, তাঁদের code-এ vulnerabilities কম ছিল।
- **Strict limitation:** এই controlled study-টি `codex-davinci-002` এবং নির্দিষ্ট কিছু security tasks evaluate করেছে। এটি **behavioral automation-bias ও overconfidence mechanism** প্রমাণ করে—একে আজকের প্রতিটি আধুনিক AI model-এর জন্য fixed universal vulnerability rate হিসেবে generalize করা যাবে না।

### ৩. Spracklen et al. (USENIX Security 2025 Distinguished Paper) — *Package Hallucinations & Slopsquatting*
- **Study setup & exact numbers:** Joseph Spracklen et al. ১৬টি code-generating LLM evaluate করে **576,000** generated Python ও JavaScript code samples তৈরি করেন, যাতে মোট **approximately 2.23 million package recommendations** ছিল, এবং সেগুলোকে **January 10, 2024**-এর package-repository snapshot-এর বিপরীতে verify করেন।
- **Key quantitative findings (exact statistical distinction):**
  - মোট **440,445টি package recommendation** hallucinated (nonexistent) ছিল—যা মোট **package recommendations-এর 19.7%** *(খুব সতর্ক থাকুন: এটি **19.7% of package recommendations**, কখনোই 19.7% of code samples নয়!)*।
  - Commercial models-এর average hallucination rate ছিল **5.2%**, আর open-source models-এর average ছিল **21.7%**।
  - এতে **205,000+ unique fake package names** ধরা পড়ে, যেগুলোর অনেকগুলো বারবার repeat হয়—ফলে **slopsquatting** একটি বাস্তব supply-chain threat হিসেবে প্রমাণিত হয়।
  - গবেষকেরা mitigation হিসেবে RAG (verified package lists/docs context-এ দেওয়া), self-detected feedback, এবং fine-tuning-ও evaluate করেছেন—কিন্তু একজন backend developer-এর সবচেয়ে নির্ভুল প্রতিরক্ষা হলো **কোনো AI-suggested package install করার আগে official registry ও docs independently verify করা।**
- **Strict limitation:** এই ফলাফলগুলো paper-এর evaluated 16 models, নির্দিষ্ট prompts, এবং **January 10, 2024** snapshot-এর প্রতিফলন (paper-টি নিজেই উল্লেখ করে যে evaluation-এর পর আরও নতুন models এসেছে)—তাই এই সংখ্যাগুলোকে আজকের সব model-এর universal current error rate ভাবা যাবে না।

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-07-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন teammate বললেন: *"Spracklen et al. প্রমাণ করেছে যে আজকের সব AI assistant-এর লেখা ১৯.৭% code sample-এ fake package থাকে, আর Perry et al. দেখিয়েছে AI ব্যবহার করলে সবসময় insecure code হয়—তাই আমাদের টিমে AI পুরোপুরি নিষিদ্ধ করা উচিত।"* Evidence-based reasoning (Block 5) ও Block 7-এর empirical limitations ব্যবহার করে এই বক্তব্যের **তিনটি** নির্দিষ্ট তথ্যগত বা যৌক্তিক ত্রুটি ধরিয়ে দিন।
> 2. **Delegation** বনাম **Abdication**-এর পার্থক্য একটি বাস্তব Node.js + PostgreSQL উদাহরণ দিয়ে দেখান, এবং ব্যাখ্যা করুন DORA 2025 কেন বলে **"AI is an amplifier"**।

এখন আপনার মাথায় একটা খুব practical প্রশ্ন আসার কথা। আমরা বুঝলাম LLM probabilistic এবং vague prompt পেলেই নিজের মতো hidden assumptions, outdated APIs, আর complected code বসিয়ে দেয়। **তাহলে code generate করার *আগেই* আমরা কীভাবে model-টার সামনে এমন স্পষ্ট boundaries তৈরি করব যাতে প্রথম চেষ্টাতেই error-এর সম্ভাবনা নাটকীয়ভাবে কমে আসে?**

---

## Vague Prompts থেকে Context Engineering-এ উত্তরণ

অনেকে মনে করেন AI থেকে ভালো code পাওয়া মানে হলো কিছু গোপন "magic prompt template" মুখস্থ করা (*"Act as a 10x senior engineer and write bug-free code!"*)। বাস্তবে এ ধরনের magic phrases প্রায় কোনো কাজেই আসে না।

কেন? কারণ আমাদের store-credit refund উদাহরণে AI-টা কোনো অদক্ষতার কারণে ভুল করেনি—সে ভুল করেছে কারণ সে **আপনার system-এর আসল context জানতই না!** সে জানত না আপনি Node.js 22 ও PostgreSQL 16 ব্যবহার করছেন, আপনার schema-তে কোনো unique constraint আছে কি না, money integer cents-এ রাখা হয় নাকি decimal-এ, কোন dependencies forbidden, এবং concurrent duplicate clicks হলে কী error code ফেরত দিতে হবে।

ঠিক এই জায়গাতেই আধুনিক AI-assisted engineering-এ **Prompt Engineering**-এর বদলে এসেছে অনেক বেশি বাস্তবসম্মত একটি শৃঙ্খলা: **Context Engineering**।

**Birgitta Böckeler** তাঁর প্রবন্ধ ***"Context Engineering for Coding Agents"*** (MartinFowler.com, **5 February 2026**)-এ (Bharani Subramaniam-এর সংজ্ঞা উদ্ধৃত করে) Context Engineering-কে চমৎকার এক বাক্যে সংজ্ঞায়িত করেছেন:

> *"Curating what the model sees so that you get a better result."*

লক্ষ করুন verb-টা: **Curating (যত্ন করে বাছাই করে দেওয়া)**—*dumping* নয়! Anthropic Engineering-এর *Effective Context Engineering for AI Agents* এবং Böckeler উভয়ই একটি crucial failure pattern তুলে ধরেন: **Context Bloat / "Context Rot"** (বা *"lost-in-the-middle"* degradation)। আপনি যদি model-কে ৫০টা অপ্রাসঙ্গিক file, বিশাল noisy log, আর ২০ পৃষ্ঠার পুরোনো chat history গাদাগাদি করে ঢেলে দেন, তবে signal-to-noise ratio পড়ে যায় এবং model গুরুত্বপূর্ণ constraints মিস করে বসে।

বাস্তব backend workflow-এ effective **Context Engineering**-এর চারটি মূল স্তম্ভ আছে—যার অনেকগুলো এই repository-তেই আমরা প্রতিদিন ব্যবহার করছি:

1. **Task-Specific Context Packet (প্রতিটা bounded কাজের জন্য):**
   Vague ১-লাইন prompt-এর বদলে একটি compact, structured brief দিন যাতে স্পষ্টভাবে থাকে:
   - **Goal:** ঠিক কী behavior বানাতে হবে (Block 2-এর verifiable spec!)।
   - **Non-Goals:** কী touch করা যাবে না বা কী এখন scope-এর বাইরে।
   - **Exact Types, Interfaces & Constraints:** Input/output shapes, integer cents vs. floats, UTC timestamps, valid ranges।
   - **Runtime & Dependency Versions:** যেমন `Node.js 22 LTS`, `PostgreSQL 16`, `pg ^8.13.0`—যাতে version drift কমে।
   - **Domain Invariants:** যেমন *"একই `order_id`-র জন্য একবারের বেশি return credit দেওয়া যাবে না; `balance_cents >= 0`"* *(Block 3!)।*
   - **Error-Handling Contract:** Silent catch নিষিদ্ধ; structured error codes ও expected HTTP status।
   - **Forbidden Dependencies / Approaches:** কোনো নতুন `npm` package যোগ করা যাবে না; SQL string interpolation নিষিদ্ধ।
   - **Repository Conventions:** ESM imports, naming style, pure functions separated from DB I/O (Block 6!)।
2. **Instructions vs. Guidance / Rules (Böckeler-এর কাঠামো):**
   - **Instructions:** এই নির্দিষ্ট task-এ এখন কী করতে হবে (আপনার Context Packet)।
   - **Guidance / Rules:** পুরো repository-র durable conventions ও guardrails যা সব session-এ বা নির্দিষ্ট file pattern-এ apply হয়—ঠিক যেমন আমাদের এই repo-তে `AGENTS.md` (always-on rules), `.agents/rules/*.md`, এবং task-অনুযায়ী on-demand `.agents/skills/*/SKILL.md`!
3. **Small, Verifiable Batches:**
   একবারে *"পুরো refund subsystem বানিয়ে দাও"* বললে AI ৮০০ লাইনের unreviewable diff তৈরি করে। পরিবর্তে কাজটিকে ছোট, independently verifiable batch-এ ভাগ করুন: প্রথম batch-এ pure eligibility calculation function + tests; দ্বিতীয় batch-এ PostgreSQL atomic credit transaction; তৃতীয় batch-এ HTTP endpoint wiring। *(DORA AI Capabilities Model-এর "Working in small batches"-এর সরাসরি প্রয়োগ!)*
4. **The Outer Harness (Deterministic Feedback Loops):**
   Böckeler একটা অসাধারণ mental model দেন: probabilistic coding model-এর চারপাশে একটি deterministic **Outer Harness** তৈরি করা—automated test runner, linter, type checker, schema constraints, এবং build checks। যাতে কেবল text instruction-এর ওপর ভরসা না করে, automated checks নিজেরাই model-এর ভুল ধরে feedback দেয়।

**একটি ভয়ংকর ভুল ধারণা এখানেই ভেঙে দিন (Böckeler-এর "Illusion of Control"):**
> **Misconception:** *"খুব ভালো structured context packet ও rules file দিলে AI আর hallucinate করবে না বা security bug তৈরি করবে না।"*
> **Reality:** ভালো context error rate অনেক কমায়—কিন্তু rules file বা context packet একটি probabilistic LLM-কে deterministic machine বানিয়ে দেয় না! যেকোনো মুহূর্তে model একটা rule ignore করতে পারে বা subtle bug ঢুকিয়ে দিতে পারে। তাই **deterministic, independent verification সবসময় non-negotiable।**

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-07-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন developer ভাবলেন *"AI-কে যত বেশি context দেব তত ভালো—চলো পুরো repo-র সব ৫০টা file আর আগের ৩ সপ্তাহের chat history একসাথে দিয়ে দিই!"* Context engineering-এর আলোকে এই approach-এ কী problem তৈরি হয়, এবং এর বদলে কীভাবে context curate করা উচিত?
> 2. কেন ভালো **Context Engineering** (strict rules, versions, invariants) ব্যবহার করার পরও independent verification বাদ দেওয়া যায় না? *"Illusion of Control"* ধারণাটি দিয়ে ২–৩ বাক্যে ব্যাখ্যা করুন।

এবার সবচেয়ে practical ধাপে আসুন। ধরুন আপনি AI-কে একটি ছোট, well-scoped task দিয়েছেন, tight context packet দিয়েছেন, এবং AI আপনাকে ৫০ লাইন Node.js + SQL code দিয়েছে। **Commit বা PR merge করার আগে ঠিক কোন disciplined workflow অনুসরণ করলে আপনি আত্মবিশ্বাসের সাথে code-টার দায়িত্ব নিতে পারবেন?**

---

## Independent AI Verification Workflow এবং Verification Harness

আগেই একটা সবচেয়ে সাধারণ ও বিপজ্জনক misconception পরিষ্কার করে নিন:

> **Misconception:** *"AI code লিখেছে, AI-ই তার জন্য unit tests লিখেছে, আর সব test পাস করেছে—তাহলে তো code verified হয়ে গেল!"*

কেন এটা ভয়ংকর ফাঁদ? ধরুন AI-টা ভুল করে ধরে নিয়েছে যে কোনো order-এ `discountCents` কখনো `orderTotalCents`-এর চেয়ে বড় হবে না, অথবা `created_at` local server time-এ আছে। আপনি যদি সেই একই AI-কে বলেন *"এর জন্য tests লিখে দাও"*, তবে AI **সেই একই ভুল assumption দিয়ে test cases বানাবে!** ফলে test-গুলো কেবল bug-টাকেই confirm করবে—এবং Block 3-এ Dijkstra-র শেখানো সত্যটা এখানে আরও তীব্রভাবে সত্য: passing tests হলো *evidence*, কখনোই bug-এর অনুপস্থিতির *proof* নয়।

তাই probabilistic AI output-কে production-grade code-এ রূপান্তর করতে আমাদের দরকার একটি **Verification Harness** *(deterministic checks—independent tests, linters, type checks, database constraints, ও empirical execution)* এবং নিচের **পুনর্ব্যবহারযোগ্য Independent AI Verification Workflow**:

1. **আগে নিজে Requirement ও Invariants Frame করুন:** AI-র code পড়ার *আগেই* নিজে ঠিক করুন এই operation-এর preconditions, postconditions, invariants, এবং failure modes কী হওয়া উচিত (Blocks 2 & 3)। নাহলে AI-র fluent code দেখে আপনি তার ভুল framing-এই anchored হয়ে যাবেন!
2. **Line-by-Line Inspection (প্রতিটা লাইনের কারণ বুঝুন):** Generated diff-এর প্রতিটি লাইন পড়ে নিজেকে প্রশ্ন করুন: *"এই লাইনটা কেন আছে? এটি বাদ দিলে বা বদলালে কী হবে?"* যে লাইন আপনি explain করতে পারেন না, সেটি commit করবেন না।
3. **Dependencies, Versions & APIs Independently Verify করুন:** কোডে কোনো `import` / `require`, config option, বা method flag থাকলে **কখনোই blindly `npm install` চালাবেন না**। প্রথমে `package.json`, official Node.js/PostgreSQL/library documentation, এবং `npm` registry-তে package-টির অস্তিত্ব, maintainer ও version compatibility যাচাই করুন (slopsquatting ও API/version drift প্রতিরোধ!)।
4. **Error Handling ও Boundary / Negative Cases Audit করুন:** খুঁজুন—কোথাও কি silent `catch` block error গিলে ফেলছে? `null`, `undefined`, empty array `[]`, `0`, negative numbers, huge payloads, অথবা UTC/timezone boundary এলে কী ঘটবে?
5. **Security-Sensitive Boundaries ও Data Access যাচাই করুন:** SQL queries সত্যিই parameterized কি না (বিশেষত dynamic column/table names বা sorting/filtering clauses-এ string interpolation লুকিয়ে আছে কি না!), authentication ও authorization (ownership check: user A যেন user B-র data না দেখে!) ঠিক আছে কি না, এবং error response-এ secrets বা raw database stack trace leak হচ্ছে কি না। *(Awareness note: application code secure করার সাধারণ baseline হিসেবে **NIST SP 800-218 SSDF** নীতিগুলো প্রযোজ্য; **NIST SP 800-218A** মূলত যাঁরা foundation models বা AI systems তৈরি/acquire করেন তাঁদের জন্য companion—সাধারণ backend developer-দের third-party model API বা coding assistant ব্যবহারের primary checklist নয়। আর **OWASP Top 10 for LLM Applications (2025)** আমাদের supply-chain, improper output handling, sensitive info disclosure ও hallucination vocabulary দেয়; গভীর backend security exploits আমরা Phase 19-এ শিখব।)*
6. **Concurrency, Transactions ও Idempotency Assumptions Stress-Test করুন:** নিজেকে প্রশ্ন করুন—একই request দুটো tab থেকে একই মিলিসেকেন্ডে এলে কী হবে? Client timeout-এর পর একই request retry করলে কি দুবার charge বা দুবার credit হবে? একাধিক SQL write কি atomic transaction-এর ভেতর আছে? *(Concurrency ও DB isolation-এর গভীর internals Phases 6, 10 ও 14-এ আসবে—এখানে আপনার কাজ হলো unstated assumptions ধরা!)*
7. **Tests Independently লিখুন বা Adversarially Audit করুন:** AI-র লেখা happy-path tests-এ সন্তুষ্ট থাকবেন না। আপনার step 1-এ ঠিক করা invariants ও negative/edge cases ভেঙে দেখার জন্য নিজে failing tests লিখুন (বা AI-র brainstorm করা cases নিজে যাচাই করে adversarial cases যোগ করুন)।
8. **Code, Tests ও Static Checks Independently Run করুন:** AI *"এটা কাজ করবে"* বলেছে বলে বিশ্বাস করবেন না—নিজের environment-এ code, tests, linter, এবং schema/type checks নিজে চালিয়ে empirical output দেখুন (Block 5!)।
9. **Maintainability ও Accidental Complexity Review করুন (Block 6 Connection!):** Code-টা কি কাজ করার পাশাপাশি *simple* ও *maintainable*? নাকি AI দ্রুত কাজ সারতে গিয়ে concerns **complect** করে ফেলেছে, অপ্রয়োজনীয় **shallow wrappers** বা copy-paste duplication ঢুকিয়ে দিয়েছে, বা ভবিষ্যতে change করা hard বানিয়ে দিয়েছে?
10. **Explicit Human Accountability Retain করুন:** শেষ commit ও deploy decision সম্পূর্ণ আপনার। Production-এ কোনো flaw বেরোলে *"AI-টা এভাবে suggest করেছিল"* কোনো engineering defense নয়।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-07-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. AI একটি refund function লিখল এবং তার জন্য ৬টা unit test-ও লিখে দিল; ৬টা test-ই পাস করল। ঠিক কোন mechanism-এর কারণে এই ৬টা green test থাকার পরও function-টিতে মারাত্মক production bug লুকিয়ে থাকতে পারে?
> 2. আপনার টিমের একজন junior developer AI agent ব্যবহার করে একটিমাত্র PR-এ ১,২০০ লাইন code ও ৩টি নতুন `npm` package যোগ করে review চেয়েছেন এবং লিখেছেন *"সব AI tests pass করেছে, merge করা যাবে?"*। Block 7-এর workflow এবং DORA/Spracklen findings-এর ভিত্তিতে আপনি reviewer হিসেবে ঠিক কী করবেন এবং কেন?

---

## Block 8-এর দিকে সেতুবন্ধন: Engineering Ownership

একটু থেমে পুরো Phase 1-এর যাত্রাটা দেখুন। Block 1 থেকে Block 7 পর্যন্ত আপনি শিখেছেন:
- Software engineering মানে শুধু Day 1-এর syntax নয়, সময়ের সাথে sustainability ও proportional rigor (Block 1);
- কোডের আগে Problem Space, ambiguities ও verifiable BCP 14 specs পরিষ্কার করা (Block 2);
- Correctness, invariants, Fault → Error → Failure, ও multidimensional ISO/IEC 25010:2023 quality (Block 3);
- Constraints, honest trade-offs, ও ADRs (Block 4);
- Empirical evidence, Scientific Method, ও blameless postmortems (Block 5);
- Essential vs. accidental complexity, simple vs. easy, deep modules, ও Technical Debt Quadrant (Block 6);
- এবং এখন AI-কে deliberate partner হিসেবে ব্যবহার করে context engineering ও independent verification চালানো (Block 7)।

কিন্তু এই সমস্ত দক্ষতা এক জায়গায় এসে একটা শেষ প্রশ্নের মুখোমুখি দাঁড়ায়: **যখন আপনি নিজে (অথবা AI-র সহায়তায়) একটা backend component তৈরি করে, review করিয়ে, production-এ ship করেন—তখন real *engineering ownership*, professional ethics, এবং disciplined code review-র মানদণ্ড ঠিক কেমন হয়?**

এই প্রশ্নের উত্তর এবং পুরো Phase 1-এর integrative hands-on demonstration নিয়েই আসছে **Block 8: Engineering Ownership and Practical Laboratory**।

---

## Study Plan, Learning Objectives, Terminology & Mandatory Reading

> **Continuity & Ownership Notes:** আপনার সব inline check, practical exercise এবং (যদি করতে চান) optional closed-book assessment-এর উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-07-notes.md` ফাইলে লিখুন। AI-এর প্রস্তুতির সময় আপনার study hours-এ গণনা হয় না।
>
> **Progression & Verification Policy:** Formal assessment সম্পূর্ণ **optional** এবং পরের block-এ এগোনোর নিয়ন্ত্রণ আপনার হাতে। আপনি চাইলে assessment সম্পন্ন করে formal evaluation-এর জন্য ChatGPT-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে), নিজে যথেষ্ট শিখেছেন মনে করলে assessment skip করে এগিয়ে যেতে পারেন (`Studied — mastery not formally verified`), অথবা assessment ছাড়াই যেকোনো unclear concept নিয়ে প্রশ্ন ও আলোচনা করতে পারেন। কোনো গুরুত্বপূর্ণ gap থাকলে ChatGPT বাড়তি verification-এর পরামর্শ দিতে পারেন, তবে assessment সম্পন্ন করা বা ChatGPT-র approval পরের block শুরু করার mandatory gate নয়। Assessment skip করলে তা automatically `Mastery verified` বলে গণ্য হয় না।

### Study Time Reference

*প্রতিটা activity-র বাস্তবসম্মত time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। Approved dossier-এ Block 7-এর `4 hours` allocation একটি flexible planning estimate; কোনো activity-রই strict deadline নেই।*

### Block 7 Learning Objectives *(Primary: Outcome 9; Supporting: Outcomes 7, 10)*

- **LO 7.1 (`Practical Competence`):** Decide **what to delegate** to AI (boilerplate, test-case brainstorming, unfamiliar syntax exploration, initial refactors with tight specs, code explanation as a starting hypothesis) versus **what requires independent human engineering reasoning** (problem framing, invariant definition, trade-off decisions, threat modeling, architecture boundaries, final verification, production accountability).
- **LO 7.2 (`Practical Competence`):** Apply **context engineering** — AI-কে vague prompts না দিয়ে explicit constraints, invariants, interfaces, non-goals, versions, এবং repository conventions দেওয়া, এবং কাজগুলোকে small, verifiable batches-এ রাখা।
- **LO 7.3 (`Working Understanding`):** Empirical evidence দিয়ে concrete **AI failure mechanisms** ব্যাখ্যা করা: confident hallucination/confabulation, **package hallucination / slopsquatting**, silent security vulnerabilities paired with developer overconfidence, plausible-looking logic/edge-case bugs, subtle API/version drift, এবং weak engineering practices-এর systemic amplification (DORA 2024/2025)। *(Note: এই failure modes-এর practical verification `LO 7.4`-এর অধীনে `Practical Competence`-এ চর্চা করা হয়।)*
- **LO 7.4 (`Practical Competence`):** AI-assisted code accept বা commit করার আগে একটি disciplined **independent verification workflow** execute করা (line-by-line review, dependency/version verification, negative/edge-case testing, security sanity checks, এবং independent execution)।

**Expected Outcomes:** Block 7 শেষ করে আপনি empirically (DORA 2024/2025, Perry et al., ও Spracklen et al. উদ্ধৃত করে) ব্যাখ্যা করতে পারবেন কেন দুর্বল verification, uncritical trust, বা দুর্বল engineering system-এর সাথে AI risk amplify করে; একটি structured context packet তৈরি করতে পারবেন; এবং AI-generated backend code-এর ওপর পূর্ণাঙ্গ independent verification checklist execute করতে পারবেন।

### Terminology Inventory

| Term | Official Target Level | Why It Matters in Block 7 |
| :--- | :---: | :--- |
| **Delegation vs. Abdication** | `Practical Competence` | Drafting speed delegate করে specification, verification ও ownership নিজের হাতে রাখা বনাম blindly output trust করা। |
| **Context Engineering** | `Practical Competence` | Probabilistic AI output constrain করতে exact specs, constraints, rules, files ও feedback loops curate করা। |
| **Hallucination / Confabulation** | `Working Understanding` | Plausible শোনায় কিন্তু অসত্য তথ্য, nonexistent API/options, wrong flags, বা fabricated citations তৈরি হওয়া। |
| **Package Hallucination ("Slopsquatting")** | `Working Understanding` | LLM-এর তৈরি nonexistent `npm`/`PyPI` package names attackers register করে supply-chain compromise ঘটানোর ঝুঁকি। |
| **Automation Bias & Overconfidence Trap** | `Working Understanding` | AI ব্যবহারের সময় insecure বা subtly buggy code-কে প্রয়োজনের চেয়ে বেশি নিরাপদ ভেবে বসার empirically documented প্রবণতা। |
| **AI as an Amplifier (DORA 2025)** | `Working Understanding` | AI বিদ্যমান engineering discipline (small batches, version control, tests, reviews) অথবা বিদ্যমান chaos—উভয়কেই magnify করে। |
| **Verification Harness** | `Practical Competence` | Non-deterministic AI output constrain করতে ব্যবহৃত deterministic checks (tests, linters, type checks, schema constraints, empirical execution)। |

### Mandatory Authoritative Resources *(Targeted Reading — ~105 min total)*

1. **Mandatory 1 (~15 min — AI findings summary only):** DORA (DevOps Research and Assessment) — [2024 Accelerate State of DevOps Report](https://dora.dev/dora-report-2024) (`LO 7.1`, `LO 7.3`)।
   - **Assigned scope:** AI adoption, developer attitudes, reported productivity/flow benefits alongside reductions in software delivery performance, এবং exact trust finding (**39.2% of respondents reported little or no trust in AI**)। *Scope reminder: Large-scale self-reported survey patterns, কোনো team বা tool-এর universal outcome নয়।*
2. **Mandatory 2 (~15 min — executive findings only):** DORA (DevOps Research and Assessment) — [2025 DORA Report: State of AI-assisted Software Development](https://dora.dev/research/2025/dora-report/) (`LO 7.1`, `LO 7.3`)।
   - **Assigned scope:** Executive findings ও *"AI is an amplifier"* thesis—AI কীভাবে বিদ্যমান organizational strengths ও weaknesses magnify করে।
3. **Mandatory 3 (~10 min — 7 capabilities overview):** DORA (DevOps Research and Assessment) — [DORA AI Capabilities Model (2025 Companion Report)](https://dora.dev/ai/capabilities-model/report/) *(overview: [dora.dev/ai](https://dora.dev/ai/))* (`LO 7.1`, `LO 7.3`)।
   - **Assigned scope:** Positive AI ROI enable করে এমন ৭টি foundational organizational capabilities skim করুন (*Clear and communicated AI stance*, *Healthy data ecosystems*, *AI-accessible internal data*, *Strong version control practices*, *Working in small batches*, *User-centric focus*, *Quality internal platforms*)।
4. **Mandatory 4 (~20 min — targeted sections only):** Neil Perry, Megha Srivastava, Deepak Kumar, Dan Boneh (Stanford, ACM CCS 2023) — *"Do Users Write More Insecure Code with AI Assistants?"*: [arXiv:2211.03622](https://arxiv.org/abs/2211.03622) / [DOI 10.48550/arXiv.2211.03622](https://doi.org/10.48550/arXiv.2211.03622) (`LO 7.3`, `LO 7.4`)।
   - **Assigned scope:** Abstract, key empirical security & overconfidence findings, এবং prompt/critical-engagement takeaway। *(Experimental context: controlled study evaluating `codex-davinci-002` on specific security tasks; পুরো paper পড়া বা universal current defect rate হিসেবে generalize করা নিষিদ্ধ।)*
5. **Mandatory 5 (~20 min — targeted sections only):** Joseph Spracklen et al. (USENIX Security 2025 Distinguished Paper) — *"We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code Generating LLMs"*: [USENIX Security '25](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen) / [arXiv:2406.10279](https://arxiv.org/abs/2406.10279) (`LO 7.3`, `LO 7.4`)।
   - **Assigned scope:** Abstract, quantitative findings (**576,000** code samples, **~2.23M** package recommendations, **440,445 / 19.7%** hallucinated *package recommendations* [not code samples] — **5.2%** commercial avg, **21.7%** open-source avg, **205,000+** unique fake names), slopsquatting threat model, এবং mitigations summary। *(Context: 16 models, specific prompts, Jan 10, 2024 snapshot — not a universal current error rate।)*
6. **Mandatory 6 (~25 min — core sections):** Birgitta Böckeler (MartinFowler.com, **5 February 2026**) — [Context Engineering for Coding Agents](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html) (`LO 7.2`, `LO 7.4`)।
   - **Assigned scope:** Curating what the model sees, *Instructions vs. Guidance/Rules*, structured context, *outer harness*, এবং *illusion of control / context bloat*।

### Optional Supplementary & Reference Resources *(Outside Mandatory Budget)*

- **Optional Supplementary:** Anthropic Engineering — [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) *(context curation, signal-to-noise degradation / "lost-in-the-middle", ও structured boundaries; exact publication date ও section headings এখানে unverified)*।
- **Reference (`Awareness` — General Application Security Baseline):** NIST SP 800-218 (Version 1.1, Feb 2022) — [Secure Software Development Framework (SSDF)](https://csrc.nist.gov/pubs/sp/800/218/final) *(general software verification & security baseline — Prepare the Organization, Protect the Software, Produce Well-Secured Software, Respond to Vulnerabilities — যা AI-assisted application code-এও প্রযোজ্য)*।
- **Optional Supplementary (`Awareness` — AI-System Producers/Acquirers Companion):** NIST SP 800-218A (2024) — [Secure Software Development Practices for Generative AI and Dual-Use Foundation Models](https://csrc.nist.gov/pubs/sp/800/218/a/final) *(Executive Summary skim — এটি generative AI / foundation models তৈরি বা AI systems acquire করা organizations-এর জন্য AI-specific SSDF companion; সাধারণ backend developers যাঁরা third-party model APIs ব্যবহার করেন তাঁদের primary application checklist নয়)*।
- **Reference (`Awareness` only):** OWASP GenAI Security Project (2025) — [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/) *(Supply Chain, Improper Output Handling, Sensitive Information Disclosure, Misinformation/Hallucination — কেবল shared vocabulary-র জন্য; বিস্তারিত LLM application security Phase 19-এ)*।

---

## Practical Exercises (~80–95 min total)

> **Ownership Rule:** সব উত্তর, audit log ও observations স্বাধীনভাবে `phases/phase-01/notes/block-07-notes.md` ফাইলে লিখুন। AI-কে দিয়ে আপনার audit বা classification answer draft করাবেন না (Exercise 3-এ AI ব্যবহার করবেন শুধুমাত্র experimental subject হিসেবে output compare করার জন্য)।

### Exercise 1: AI Output Audit Lab — Node.js / SQL *(Mandatory — ~45–50 min)*

**Environment & Version Context (for official documentation verification):**
- **Runtime:** Node.js 22 LTS (ES Modules, built-in `node:test` + `node:assert/strict`)
- **Database:** PostgreSQL 16
- **Driver:** `pg` (`node-postgres`) `^8.13.0`
- **Relevant Tables:**
  ```sql
  CREATE TABLE customer_wallets (
    user_id       INTEGER PRIMARY KEY,
    balance_cents INTEGER NOT NULL CHECK (balance_cents >= 0)
  );

  CREATE TABLE store_credits (
    id            SERIAL PRIMARY KEY,
    user_id       INTEGER NOT NULL REFERENCES customer_wallets(user_id),
    order_id      INTEGER NOT NULL,
    reason        TEXT NOT NULL,
    amount_cents  INTEGER NOT NULL,
    expires_at    TIMESTAMPTZ NOT NULL,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
  );
  ```

> **CRITICAL SAFETY RULE:** নিচের code-এ থাকা কোনো package নাম official `npm` registry ও documentation-এ independently verify করার আগে **কখনোই `npm install` চালাবেন না**! এখানে কোনো real malicious package ব্যবহার করা হয়নি, কিন্তু real-world slopsquatting প্রতিরক্ষার অভ্যাস হিসেবে unverified package install করা কঠোরভাবে নিষিদ্ধ।

**Scenario & Code Under Audit (`issueReturnCredits.mjs`):**
ধরুন, একটি AI coding assistant নিচের ~55-line Node.js + PostgreSQL ES module-টি generate করেছে। এর কাজ হলো: কোনো customer-এর approved return items-এর জন্য eligible store credit ইস্যু করা এবং সেই customer-এর credit history-র সাম্প্রতিক তালিকা (caller-এর পছন্দমতো sort order-এ) return করা। Specification অনুযায়ী: (1) প্রতিটি `amountCents` অবশ্যই positive integer হতে হবে; (2) একই `(orderId, reason)`-এর জন্য কখনো দুবার credit দেওয়া যাবে না; (3) `store_credits` insert ও `customer_wallets` balance update atomic হতে হবে; এবং (4) কোনো database failure চুপিসারে গোপন না করে explicit error হিসেবে caller-কে জানাতে হবে।

```javascript
// issueReturnCredits.mjs — AI-generated draft for independent audit
export async function issueReturnCredits(pool, payload, deps = {}) {
  const { userId, orderId, reason, items, expiresAt, sortBy = "created_at" } = payload;

  if (
    !Array.isArray(items) ||
    items.length === 0 ||
    items.some((item) => !Number.isInteger(item?.amountCents) || item.amountCents <= 0)
  ) {
    throw new Error("INVALID_ITEMS");
  }

  const expiresAtMs = new Date(expiresAt).getTime();
  if (!Number.isFinite(expiresAtMs) || expiresAtMs <= Date.now()) {
    throw new Error("INVALID_EXPIRY");
  }

  const withSafeTxGuard =
    deps.withSafeTxGuard ?? (await import("pg-tx-guard-helper")).withSafeTxGuard;

  try {
    const existing = await pool.query({
      text: "SELECT id FROM store_credits WHERE order_id = $1 AND reason = $2",
      values: [orderId, reason],
      isolationLevel: "serializable",
      autoRetryOnDeadlock: true,
    });

    if (existing.rows.length > 0) {
      return { creditedCents: 0, duplicate: true, history: [] };
    }

    const totalCents = items.reduce((sum, item) => sum + item.amountCents, 0);

    await withSafeTxGuard(pool, async () => {
      await pool.query(
        `INSERT INTO store_credits (user_id, order_id, reason, amount_cents, expires_at)
         VALUES ($1, $2, $3, $4, $5)`,
        [userId, orderId, reason, totalCents, expiresAt]
      );

      await pool.query(
        `UPDATE customer_wallets
         SET balance_cents = balance_cents + $1
         WHERE user_id = $2`,
        [totalCents, userId]
      );
    });

    const historyRes = await pool.query(
      `SELECT id, order_id, amount_cents, created_at
       FROM store_credits
       WHERE user_id = $1
       ORDER BY ${sortBy} DESC
       LIMIT 20`,
      [userId]
    );

    return { creditedCents: totalCents, duplicate: false, history: historyRes.rows };
  } catch (err) {
    return { creditedCents: 0, duplicate: false, history: [] };
  }
}
```

**Safe Local Audit & Test Setup (কোনো external package install না করেই):**
1. একটি local scratch folder-এ ওপরের code-টি `issueReturnCredits.mjs` নামে save করুন—কোনো `npm install` চালাবেন না।
2. প্রথমে code-এ থাকা external package ও `pg` driver options-গুলো official `npm` registry (`https://www.npmjs.com` বা read-only `npm view <package-name>`) এবং official `node-postgres` documentation-এ independently verify করুন।
3. Built-in `node:test` + `node:assert/strict` ব্যবহার করে `issueReturnCredits.test.mjs` লিখুন এবং `node --test issueReturnCredits.test.mjs` দিয়ে চালান:
   - **Default runtime path যাচাই করতে:** তৃতীয় argument (`deps`) ছাড়াই `issueReturnCredits(mockPool, payload)` call করে দেখুন কী ঘটে।
   - **বাকি behavior ও edge cases safely test করতে:** কোনো unverified package install বা trust না করেই তৃতীয় argument হিসেবে একটি minimal pass-through stub `{ withSafeTxGuard: (_pool, fn) => fn() }` এবং একটি minimal in-memory mock `pool` (`{ query: async (...) => ... }`) অথবা local PostgreSQL `pg.Pool` pass করুন।
   - **Corrected implementation verify করতে:** সব defects fix করার পর extra কোনো third-party dependency ছাড়াই (শুধু Node.js 22 built-ins ও `pg`-র documented API ব্যবহার করে) একই independent tests আবার চালিয়ে দেখান যে আগে fail করা tests এখন pass করছে।

**Deliverables (`phases/phase-01/notes/block-07-notes.md`):**
1. **Independent Line-by-Line Audit Log (`LO 7.3`, `LO 7.4`):** AI-র code-এ থাকা **চারটি ভিন্ন category-র defects** independently খুঁজে বের করুন এবং প্রতিটির জন্য লিখুন: (ক) exact line(s), (খ) কী defect এবং কেন এটি dangerous, এবং (গ) কোন official documentation বা empirical check দিয়ে আপনি এটি confirm করেছেন (`node-postgres` / `npm` / PostgreSQL 16 / Node.js 22 docs দেখে)।
2. **Failing Test Design & Execution (`LO 7.4`):** প্রতিটা defect category expose করার জন্য একটি করে **independent failing test case** লিখুন এবং ওপরের safe setup অনুসরণ করে `node --test` দিয়ে চালান—যাতে fix করার আগে test-গুলো fail করে এবং fix করার পর pass করে।
3. **Corrected Implementation & Schema Note (`LO 7.4`):** সব কটি defect fix করে পরিষ্কার, runnable, extra-dependency-free (শুধু `pg` ও Node.js 22 built-ins ব্যবহার করে) revised implementation লিখুন—এবং schema-তে কোনো constraint যোগ করা প্রয়োজন হলে সেটিও উল্লেখ করুন।

**Evaluation Criteria:** আপনার audit কেবল "এটা দেখতে খারাপ" বলবে না—প্রতিটা defect-এর exact mechanism, official docs verification, reproduction/failing test, এবং clean fix (নতুন accidental complexity না বাড়িয়ে) স্পষ্টভাবে প্রমাণ করবে।

---

### Exercise 2: Delegation Boundary Exercise *(Mandatory — ~15–20 min)*

নিচের **৬টি concrete backend task** মূল্যায়ন করুন:

1. **Drafting a schema validation helper** *(কোনো API payload-এর নির্দিষ্ট field types ও ranges validate করার helper)*
2. **Choosing an authentication architecture** *(stateful sessions vs. signed tokens vs. OAuth/OIDC বেছে নেওয়া)*
3. **Brainstorming edge-case tests** *(একটি বিদ্যমান pure discount function-এর জন্য extra boundary test ideas খোঁজা)*
4. **Designing transaction boundaries for money transfer** *(wallet-to-wallet transfer-এ atomicity, locking, ও idempotency design)*
5. **Explaining an unfamiliar error message** *(PostgreSQL বা Node.js-এর একটি অচেনা error stack trace প্রাথমিকভাবে বোঝা)*
6. **Approving a production migration** *(live multi-million-row PostgreSQL table-এ schema migration PR approve ও deploy করা)*

**Deliverables (`phases/phase-01/notes/block-07-notes.md`):**
প্রতিটি task-কে নিচের তিনটি category-র যেকোনো একটিতে classify করুন:
- **`Good AI Delegation (with verification)`**
- **`Collaborative Drafting (human leads constraints)`**
- **`Human-Owned Decision`**

শুধু label দিলে চলবে না—প্রতিটি task-এর পাশে ২–৩ বাক্যে **justification** লিখুন: (ক) কেন এই category বেছে নিলেন, (খ) এটি ভুলভাবে over-delegate (abdicate) করলে production-এ কী risk তৈরি হবে, এবং (গ) AI ব্যবহার করলে ঠিক কোন human constraints বা verification steps বাধ্যতামূলক।

---

### Exercise 3: Context Packet Construction *(Mandatory — ~20–25 min)*

**Pure-Logic Backend Task:**
আমাদের online store-এর জন্য একটি ছোট pure Node.js 22 function তৈরি করতে হবে: `calculateEligibleDiscount({ subtotalCents, promo, userUsageCount, nowUtcIso })` — যা promo code-এর expiry (UTC), minimum order threshold, per-user usage limit, এবং flat/percentage discount (integer cents, never negative, floor rounding) যাচাই করে deterministic structured result ফেরত দেয়।

**Deliverables (`phases/phase-01/notes/block-07-notes.md`):**
1. **Prompt A (Vague 1-Line Prompt):** একটি সাধারণ, অস্পষ্ট ১-লাইনের prompt লিখুন (যেমন বাস্তবে তাড়াহুড়োয় developer-রা লেখেন)।
2. **Prompt B (Structured Context Packet):** একই task-এর জন্য একটি সম্পূর্ণ, compact **Structured Context Packet** তৈরি করুন, যাতে স্পষ্টভাবে নিচের **৮টি অংশ** থাকবে:
   - **Goal**
   - **Non-Goals**
   - **Exact Types & Input/Output Constraints**
   - **Runtime / Environment Versions** (Node.js 22 LTS; PostgreSQL version যেখানে প্রাসঙ্গিক)
   - **Important Invariants**
   - **Error-Handling Expectations**
   - **Forbidden Dependencies or Approaches**
   - **Repository Conventions**
3. **Objective Comparison (`LO 7.2`, `LO 7.4`):** দুটি prompt (আপনার পছন্দের কোনো AI tool-এ চালিয়ে অথবা side-by-side specification gap বিশ্লেষণ করে) নিচের explicit criteria-র ওপর তুলনা করুন এবং আপনার **observations** নথিভুক্ত করুন:
   - কয়টি hidden assumption নিজে থেকেই বসিয়ে দিয়েছে?
   - Boundary/edge cases (`0`, negative, exact UTC expiry second, floats vs. cents) কীভাবে handle হয়েছে?
   - Error contract ও forbidden dependencies মেনে চলেছে কি না?
   - Accidental complexity (Block 6) বেড়েছে না কমেছে?
   - **Verification Note:** Structured Context Packet দেওয়ার পরও output-টিতে কী কী জিনিস আপনাকে এখনো independently verify ও execute করতে হয়েছে (অর্থাৎ কেন better context-ও correctness guarantee করে না)?

---

## Closed-Book Mastery Assessment *(Optional — ~20–25 min)*

এই assessment-টি সম্পূর্ণ **optional**—Block 8 শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র approval নেওয়া mandatory নয়। আপনি চাইলে:
- এই lesson ও বাইরের সব tab বন্ধ করে নিচের পাঁচটি প্রশ্নের clear, original উত্তর `phases/phase-01/notes/block-07-notes.md` ফাইলে লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা আপনি যথেষ্ট বুঝেছেন মনে করলে assessment skip করে পরের block-এ এগোতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment না করেই যেকোনো unclear concept নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(Note: `~20–25 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 7.1 / Outcomes 9, 10)** **Delegation** এবং **Abdication**-এর পার্থক্য ব্যাখ্যা করুন। *Speed of code generation* কেন *speed of verified, maintainable delivery*-র সমান নয়, এবং কোন কোন backend সিদ্ধান্ত কখনোই AI-কে delegate করা উচিত নয়?
2. **(LO 7.2 / Outcome 9)** **Context Engineering** কী, এবং vague prompting বা massive file-dumping (*context bloat*) থেকে এটি কীভাবে আলাদা? একটি high-signal context packet-এর মূল উপাদানগুলো এবং Böckeler-এর *outer harness* ও *illusion of control* ধারণা ব্যাখ্যা করুন।
3. **(LO 7.3 / Outcomes 7, 9)** **DORA 2024 & 2025** (*39.2% low/no trust*, *"AI is an amplifier"*, এবং ৭টি organizational capabilities), **Perry et al. (ACM CCS 2023)**, এবং **Spracklen et al. (USENIX Security 2025)**-এর মূল findings ও **exact experimental/statistical limitations** ব্যাখ্যা করুন—কেন এদের কোনোটিকেই সব বর্তমান AI tool-এর universal defect rate বলা যাবে না?
4. **(LO 7.3 / Outcome 9)** **Package hallucination ও slopsquatting** কীভাবে কাজ করে? পাশাপাশি explain করুন কেন AI-generated code যখন AI-র নিজের generated tests pass করে, তখনও সেটি verified হয় না।
5. **(LO 7.4 / Outcomes 9, 10)** AI-assisted Node.js + PostgreSQL code commit করার আগে আপনার পূর্ণাঙ্গ **independent verification workflow** ধাপে ধাপে ব্যাখ্যা করুন (requirements/invariants framing, line-by-line review, dependency/version check, negative/edge cases, security, concurrency/idempotency, independent execution, ও maintainability checkসহ)।

### Block 7 Readiness & Progression Checklist
- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 সম্পন্ন করেছেন (পরে Self-Check Guidance মিলিয়ে দেখেছেন)।
- [ ] ৬টি Mandatory Targeted Reading (DORA 2024, DORA 2025, DORA AI Capabilities Model, Perry et al., Spracklen et al., এবং Birgitta Böckeler) সম্পন্ন করেছেন।
- [ ] `phases/phase-01/notes/block-07-notes.md` ফাইলে স্বাধীনভাবে Exercise 1 (AI Output Audit Lab), Exercise 2 (Delegation Boundary), এবং Exercise 3 (Context Packet Construction) সম্পন্ন করেছেন।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে Mastery Assessment Questions 1–5 লিখেছেন এবং formal evaluation-এর জন্য ChatGPT-র কাছে share করেছেন।
- [ ] *(Optional)* কোনো concept unclear মনে হলে assessment ছাড়াই সেগুলো নিয়ে ChatGPT-র সাথে আলোচনা করে পরিষ্কার হয়েছেন।

> **Progression Status Distinction:** Block 8 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও চর্চা শেষ করে assessment ছাড়া এগোলে এই block-টির status হবে **`Studied — mastery not formally verified`**। যথেষ্ট evidence (exercises ও/অথবা assessment) share করে evaluation সম্পন্ন করলেই তা **`Mastery verified`** হিসেবে গণ্য হবে; assessment skip করলে তা automatically `Mastery verified` হয় না।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`notes/block-07-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Exercises 1–3 বা Optional Closed-Book Mastery Assessment-এর কোনো solution এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) optional ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* তিনটি স্পষ্ট ত্রুটি আছে: (ক) **Statistical misreading (Spracklen et al.):** `19.7%` হলো মোট **package recommendations**-এর অনুপাত—*code samples*-এর `19.7%` নয়; এবং এটি নির্দিষ্ট ১৬টি model ও **January 10, 2024** snapshot-এর ফলাফল (যার মধ্যে commercial avg ছিল `5.2%` ও open-source `21.7%`), আজকের সব AI tool-এর universal error rate নয়। (খ) **Over-generalization (Perry et al.):** এই controlled study-টি `codex-davinci-002` এবং নির্দিষ্ট কিছু security tasks-এ চালানো হয়েছিল; এটি behavioral overconfidence trap দেখায় এবং এ-ও দেখায় যে critical engagement করলে vulnerabilities কমে—এটি প্রমাণ করে না যে AI ব্যবহার করলেই *সবসময়* insecure code হয়। (গ) **False dichotomy:** ঝুঁকি আছে বলে AI পুরোপুরি নিষিদ্ধ করার বদলে disciplined delegation, context engineering ও deterministic verification দিয়ে risk manage করাই প্রকৃত engineering।
  2. *শক্তিশালী উত্তরে থাকবে:* **Delegation:** আপনি নিজে PostgreSQL schema constraints, invariants, ও error contract define করে AI-কে একটা bounded validation helper draft করতে দিলেন, তারপর প্রতিটা লাইন review করে independent tests চালালেন। **Abdication:** Vague prompt দিয়ে AI-র লেখা database migration ও refund logic না বুঝেই merge করে দিলেন। **AI is an amplifier (DORA 2025):** AI ভাঙা delivery process ঠিক করে না—যে team-এ small batches, version control, tests ও code review শক্তিশালী সেখানে AI productivity বাড়াতে সাহায্য করে; আর যে team-এ review ও tests দুর্বল, সেখানে AI আরও দ্রুত বড় buggy changes ঢুকিয়ে instability বাড়ায়।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* ৫০টা অপ্রাসঙ্গিক file আর ৩ সপ্তাহের chat history ঢেলে দিলে **context bloat / context rot** (signal-to-noise degradation / "lost-in-the-middle") ঘটে—model গুরুত্বপূর্ণ constraints ও invariants miss করে এবং অপ্রাসঙ্গিক patterns অনুকরণ করে বসে। এর বদলে task-relevant minimal files, explicit goal/non-goals, versions, invariants, forbidden dependencies, ও repository rules যত্ন করে **curate** করে দেওয়া উচিত।
  2. *শক্তিশালী উত্তরে থাকবে:* **Illusion of Control** (Böckeler) বলতে বোঝায়—rules files, hooks, বা structured context দেখে মনে হতে পারে model-কে পুরোপুরি control করা গেছে। কিন্তু LLMs মূলত probabilistic; ভালো context error probability কমালেও deterministic compliance guarantee করে না। তাই deterministic verification harness (tests, linters, docs verification, independent execution ও human review) সবসময় বাধ্যতামূলক।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* যখন একই AI code এবং tests দুটোই generate করে, তখন উভয়ের পেছনে **একই flawed assumption** (যেমন happy-path-only input, mocked DB behavior, missing concurrency/timezone edge cases) কাজ করে। ফলে tests আসলে bug-টাকেই confirm করে; তাছাড়া Dijkstra-র নীতি অনুযায়ী finite passing tests কখনোই bug-এর অনুপস্থিতি প্রমাণ করে না।
  2. *শক্তিশালী উত্তরে থাকবে:* Reviewer হিসেবে ১,২০০ লাইনের batch এবং ৩টি unverified package blindly merge করা যাবে না। (ক) **Small batches (DORA):** এত বড় AI diff-এ human review কার্যকারিতা হারায়—তাই change-টিকে ছোট, logically coherent, reviewable PR-এ ভাঙতে বলবেন। (খ) **Supply-chain & API verification (Spracklen et al.):** ৩টি নতুন `npm` package আসলেই real, maintained, ও প্রয়োজনীয় কি না (slopsquatting risk ও accidental complexity check) official registry/docs থেকে verify করতে হবে। (গ) **Independent verification:** Author-কে প্রতিটা লাইন explain করতে হবে এবং AI-generated happy-path tests-এর বাইরে independent negative, security, ও edge-case tests চালিয়ে প্রমাণ দেখাতে হবে।
