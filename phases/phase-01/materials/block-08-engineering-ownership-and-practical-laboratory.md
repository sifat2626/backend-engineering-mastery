# Phase 1, Block 8 — Engineering Ownership and Practical Laboratory

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `notes/block-08-notes.md`-এ Pause & Checks 1–3 লেখা | **~45–60 min** *(flexible — limit নয়)* |
| **Mandatory 1:** Google Engineering Practices — *Code Review Developer Guide* | কেবল *The Standard of Code Review*, *What to look for in a code review*, *Writing good CL descriptions*, এবং *Small CLs* | ~25–30 min |
| **Mandatory 2:** ACM — *ACM Code of Ethics and Professional Conduct (2018)* | কেবল Section 1 (`1.1`, `1.2 Avoid harm`, `1.3 Be honest and trustworthy`) এবং Section 2 (`2.1`, `2.2`, `2.5`) | ~15–20 min |
| **Pre-Lab Exercise (Mandatory):** Structured Code Review Audit (`LO 8.2`) | Sample Node.js PR diff ও CL description-এর ওপর Google-এর code review dimensions প্রয়োগ ও CL description rewrite | ~20–25 min |
| **Capstone Lab — Stage 1 (Mandatory):** Problem & Requirements Brief | Ambiguous ticket বিশ্লেষণ: Facts/Assumptions/Questions/Decisions, ≥6 ambiguities, Non-Goals, RFC 2119/8174 criteria, invariants | ~35–40 min |
| **Capstone Lab — Stage 2 (Mandatory):** Constraint & Trade-Off ADR + Mid-Lab Complication | `ADR-001` (২+ alternatives, downsides, overengineering rejection) + Day-2 changed requirement reassessment | ~35–40 min |
| **Capstone Lab — Stage 3 (Mandatory):** Controlled AI-Assisted Implementation, Audit & Tests | Context packet, pure Node.js 22 `promoRedemption.mjs` (in-memory), line-by-line audit log, ও ≥10 adversarial `node --test` cases | ~50–60 min |
| **Capstone Lab — Stage 4 (Mandatory):** Engineering Ownership & Defense Statement | Production-grade CL description, decision defense, Verified vs. Unverified matrix, production failure modes, revisit triggers & sign-off | ~25–30 min |
| **Total (Required Core — without Optional Assessment)** | Guided Lesson + ২টি Mandatory Reading + Pre-Lab Exercise + 4-Stage Capstone Lab | **~250–305 min (~4.2–5.1 h)** |
| **Optional Closed-Book Synthesis Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি Phase-1 synthesis প্রশ্নের উত্তর লেখা | ~25–30 min *(optional)* |
| **Total (with Optional Assessment)** | Required Core + Optional Closed-Book Synthesis Assessment | **~275–335 min (~4.6–5.6 h)** |

> **Planning Note (Honest Scheduling Reference):** Approved Research Dossier-এ Block 8-এর target estimate হলো **4 hours (240 minutes)** এবং `PROGRESS.md` / `phases/phase-01/README.md`-এ approximate range হলো **4–6 hours**—উভয়ই flexible planning reference, কোনো strict deadline নয়। বাস্তবে একটি genuine end-to-end capstone lab (problem framing, ADR + mid-lab reassessment, pure Node.js verification + adversarial tests, এবং ownership defense) artificially চেপে দিলে শেখার মান কমে যায়; তাই এখানে সৎ, বাস্তবসম্মত estimate (`~4.2–5.1 h` required core; `~4.6–5.6 h` optional assessment সহ) দেওয়া হয়েছে—যা 4–6 ঘণ্টার range-এর ভেতরেই পড়ে। দ্রুত শেষ করার চেয়ে ধারণাগুলো গভীরভাবে বোঝা ও নিজের engineering judgment যাচাই করাই মূল অগ্রাধিকার; বিশেষ করে **Guided Lesson**-এর জন্য প্রয়োজনে আনুমানিক সময়ের চেয়ে বেশি সময় লাগা সম্পূর্ণ স্বাভাবিক। প্রতিটি reading-এর সময় কেবল নির্ধারিত sections-এর জন্য, পুরো documentation site বা সম্পূর্ণ ethics code পড়ার জন্য নয়। চাইলে Block 8-কে দুটি focused session-এ ভাগ করে করতে পারেন (Session 1: Guided Lesson + ২টি Mandatory Reading + Pre-Lab Code Review; Session 2: 4-Stage Capstone Lab)। **Optional Closed-Book Synthesis Assessment** সম্পূর্ণ optional এবং কোনো mandatory gate নয়।

---

Block 1 থেকে Block 7 পর্যন্ত আমাদের সেই online store-এর যাত্রাটা একবার মনে করুন। `SAVE10` দিয়ে শুরু হয়ে payment receipt specification, `LAUNCH50`-এর invariant failure, flash-sale-এর `ADR-001`, `deploy #412`-এর evidence-based postmortem, VIP promo-র accidental complexity, এবং AI-generated store-credit code audit—সবই আপনি পেরিয়ে এসেছেন।

এবার flash sale-এর পরের সপ্তাহের একটি বাস্তব incident দিয়ে Phase 1-এর চূড়ান্ত অধ্যায়ে ঢুকুন।

শুক্রবার বিকেল ৪টায় একটি promo-redemption update production-এ deploy হলো। মাত্র দুই ঘণ্টার মধ্যে customer support-এর queue উপচে পড়ল: নির্দিষ্ট কিছু VIP customer checkout-এ বৈধ promo code বসিয়েও discount পাচ্ছেন না, আর কয়েকজন customer-এর card payment প্রথমবার decline হওয়ার পর তাঁদের single-use promo code স্থায়ীভাবে "already used" হয়ে লক হয়ে গেছে!

শনিবার সকালে জরুরি review call-এ যখন জানতে চাওয়া হলো—*"এই defective change-টা আমাদের পুরো pipeline পেরিয়ে live customer-দের কাছে পৌঁছাল কীভাবে?"*—তখন টিমের ভেতর থেকে তিনটি অত্যন্ত পরিচিত, ভীষণ মানবিক উত্তর এল:

1. **Author** বললেন: *"Product Manager-এর ticket-এ তো failed payment-এর কথা লেখাই ছিল না! আমি ticket-এ যা লেখা ছিল তাই করেছি। আর validation-এর বড় অংশটা AI coding assistant generate করেছিল—AI-র বানানো সব unit tests green ছিল!"*
2. **Reviewer** বললেন: *"PR-টা ছিল ৯০০ লাইনের, শুক্রবার বিকেলে release-এর তাড়া ছিল। দেখলাম CI-তে সব tests pass করেছে আর Author অভিজ্ঞ মানুষ—তাই বিস্তারিত প্রতিটি লাইন না পড়েই `LGTM` দিয়ে approve করে দিয়েছি।"*
3. **Author আবার যোগ করলেন:** *"আর production-এ দেড় ঘণ্টা ধরে যে customer-রা fail করছিল সেটা আমি জানব কীভাবে? Deploy তো automated pipeline করেছে, আর runtime alert দেখা তো Ops টিমের কাজ—আমি শুধু feature code লিখেছি!"*

একটু থেমে পুরো দৃশ্যটার দিকে তাকান।

লক্ষ করুন—একটা আশ্চর্যজনক ঘটনা ঘটেছে: **কেউই নিজের কোনো দায় দেখতে পাচ্ছেন না!** Author দায় চাপালেন PM-এর vague ticket, AI tool, এবং Ops-এর ঘাড়ে; Reviewer দায় চাপালেন বড় PR আর শুক্রবারের তাড়াহুড়োর ঘাড়ে। অথচ real customer-রা ক্ষুব্ধ হলেন, sales হারাল, support team চাপে পড়ল, আর company-র ওপর মানুষের আস্থা ক্ষতিগ্রস্ত হলো।

এই মুহূর্তেই আপনি বুঝতে পারছেন কেন Block 1 থেকে Block 7-এর প্রতিটি skill শেখার পরও **Block 8** ছাড়া Phase 1 অসম্পূর্ণ। আপনি specification লিখতে জানেন, invariants চেনেন, ADR বানাতে পারেন, evidence দিয়ে diagnose করতে জানেন, complexity বুঝতে পারেন, আর AI-কে verify করতে পারেন—**কিন্তু শেষ পর্যন্ত যখন আপনাকে একটি বাস্তব backend decision নিতে হয়, অন্যের code review করতে হয়, আর live system-এ কিছু ship করতে হয়, তখন একজন accountable engineer হিসেবে "Engineering Ownership" আসলে দেখতে কেমন হয়?**

এই একটি প্রশ্নই আমাদের পুরো Block 8—এবং শেষে আপনার Phase 1 Capstone Practical Laboratory—এক সুতোয় বেঁধে রাখবে।

---

## Engineering Ownership, Professional Accountability, এবং Ethics

দৈনন্দিন software টিমে "ownership" শব্দটা নিয়ে দুটি সম্পূর্ণ বিপরীত ভুল ধারণা প্রচলিত আছে। প্রথম ভুল ধারণাটা আমরা ওপরেই দেখলাম: **দায়িত্ব এড়ানো (Abdication / Blame Deflection)**—যেখানে engineer নিজেকে কেবল "ticket-to-code translator" ভাবেন এবং কোনো সমস্যা হলে vague ticket, AI tool, reviewer, বা operations-কে দোষ দেন।

আবার ঠিক এর উল্টোদিকে আরেকটি সূক্ষ্ম ফাঁদ আছে: **Territorial Ownership**—যেখানে কোনো engineer মনে করেন, *"এই service বা module-টা আমার ব্যক্তিগত এলাকা; এখানে অন্য কেউ হাত দিতে পারবে না, review-তে আমার code নিয়ে প্রশ্ন তোলা মানে আমাকে অপমান করা!"*

বাস্তব software engineering-এ এর কোনোটিই প্রকৃত ownership নয়। প্রকৃত **Engineering Ownership** হলো **Accountable Stewardship**:

| দৃষ্টিভঙ্গি | এটি আসলে কেমন আচরণ করে | এর ফলাফল কী হয় |
| :--- | :--- | :--- |
| **Abdication / Blame Deflection** | *"Ticket-এ লেখা ছিল না"*, *"AI লিখে দিয়েছে"*, *"Reviewer ধরেনি"*, *"Ops দেখবে"*। | Ambiguity ও bugs অবাধে production-এ যায়; team trust ভেঙে পড়ে; বারবার একই outage ঘটে। |
| **Territorial Ownership** | *"এটা আমার code, কেউ touch করবে না"*; defensive review attitude; knowledge hoarding। | **Bus factor = 1** *(একজন engineer চলে গেলেই বা ছুটিতে গেলেই সিস্টেম অচল)*; team velocity থেমে যায়; code review অকার্যকর হয়। |
| **Accountable Stewardship (True Ownership)** | *"আমি যা author, approve বা ship করি তার correctness, clarity, verification ও real-world impact-এর দায়িত্ব আমি নিই—এবং সিস্টেমটা এমনভাবে রাখি যেন পুরো টিম নিরাপদে এটি বুঝতে, review করতে ও maintain করতে পারে।"* | High trust, উচ্চ code health, দ্রুত কিন্তু নিরাপদ delivery, এবং honest learning from failure। |

এবার একটি গভীর ও সূক্ষ্ম প্রশ্নের মুখোমুখি হোন—যেটা না বুঝলে Block 5-এর **Blameless Postmortem** আর Block 8-এর **Engineering Ownership**-এর মধ্যে আপাতদৃষ্টিতে একটা দ্বন্দ্ব মনে হতে পারে!

প্রশ্নটা হলো: *"Block 5-এ তো আমরা শিখলাম production outage হলে একজন মানুষের ঘাড়ে দোষ না চাপিয়ে blameless postmortem করতে হয়! তাহলে এখন আবার কেন বলছি যে একজন engineer নিজের কাজের পূর্ণ accountability নেবেন?"*

এই আপাত দ্বন্দ্বটার সমাধান লুকিয়ে আছে **Individual Responsibility** বনাম **Distributed Organizational Accountability**-র স্পষ্ট পার্থক্যের ভেতর:

- **Individual Responsibility (ব্যক্তিগত পেশাদার দায়িত্ব — যা কখনো abdicate করা যায় না):** আপনি যখন কোনো code author করেন, কোনো PR approve করেন, বা কোনো technical decision sign-off করেন, তখন আপনি নিজের judgment-এর দায়িত্ব কোনো AI tool, vague ticket, বা অন্য কারও ওপর ছেড়ে দিতে পারেন না। আপনার কাজটি কি আপনি নিজে বুঝেছেন? Ambiguities কি প্রশ্ন করে পরিষ্কার করেছেন? Edge cases ও dependencies কি independently verify করেছেন? Risks কি সততার সাথে disclose করেছেন? এই প্রশ্নের উত্তর আপনাকেই দিতে হবে।
- **Distributed Organizational Accountability (বিস্তৃত সাংগঠনিক দায়বদ্ধতা):** একই সাথে, আধুনিক software systems একক কোনো মানুষের বীরত্বে নিরাপদ থাকে না। প্রকৃত production reliability-র দায়বদ্ধতা একটি পুরো organization জুড়ে **distributed** থাকে—author-এর verification, reviewer-এর careful second look, team-এর automated tests ও CI guardrails, engineering leadership-এর realistic deadline ও psychological safety তৈরি, এবং operational monitoring ও rollback ব্যবস্থা।

তাই **Blameless Postmortem ≠ Responsibility-free culture!**
একটি healthy engineering team-এ কোনো incident ঘটলে সংগঠন একজন মানুষকে scapegoat করে শাস্তি দেয় না—বরং খোঁজে *"আমাদের কোন প্রক্রিয়া, missing test, বা deadline চাপ এই ভুলটাকে production পর্যন্ত যেতে দিল?"* আর ঠিক সেই একই সময়ে, প্রতিটি engineer ব্যক্তিগতভাবে নিজের professional ownership ধরে রাখেন: ভুল হলে লুকান না, AI-র ঘাড়ে দোষ চাপান না, কী verify হয়নি তা সৎভাবে জানান, এবং ভবিষ্যতে যেন সেই শ্রেণির ভুল আর না ঘটে তার জন্য system শক্তিশালী করেন।

### কেন এটি কেবল "ভালো অভ্যাস" নয়, বরং Professional Ethics

এখানে অনেক developer ভাবেন, *"আচ্ছা, এসব তো বড় কোম্পানির process কথা—ছোট একটা promo code বা API bug-এ ethics-এর কী আছে?"*

একটু থেমে বাস্তব জগতের দিকে তাকান। একজন backend engineer-এর লেখা code কখনো শুধু শূন্যের ভেতর চলা puzzle নয়। একটি payment bug কোনো পরিবারকে মাসের শেষে double-charge করে বিপদে ফেলতে পারে; একটি authorization ভুল হাজারো user-এর personal data ও ঠিকানা ফাঁস করে দিতে পারে; একটি healthcare বা fintech backend-এর silent error-swallowing কারও জরুরি সেবা বা বেতন আটকে দিতে পারে; আর যাচাই না করে AI-suggested fake `npm` package install করলে (Block 7-এর slopsquatting!) পুরো customer database attacker-দের হাতে চলে যেতে পারে।

ঠিক এই কারণেই computing পেশার সবচেয়ে স্বীকৃত আন্তর্জাতিক নৈতিক মানদণ্ড—**ACM Code of Ethics and Professional Conduct (2018)**—software engineers-দের ওপর ছয়টি সরাসরি ব্যবহারিক দায়িত্ব আরোপ করে *(এবং এর সফটওয়্যার-নির্দিষ্ট সহচর হলো **ACM/IEEE-CS Software Engineering Code of Ethics and Professional Practice, 1999**, যার ৮টি principle—Public, Client and Employer, Product, Judgment, Management, Profession, Colleagues, Self—একই ভিত্তি জোরদার করে)*:

1. **1.1 Contribute to society and to human well-being:** স্বীকার করুন যে আপনার system যাঁদের জীবন, অর্থ, সময় বা ডেটা স্পর্শ করে, সেই **সব মানুষই stakeholder**—শুধু যে manager ticket খুলেছেন তিনি নন।
2. **1.2 Avoid harm:** "Harm" বলতে বোঝায় উল্লেখযোগ্য ও অন্যায্য নেতিবাচক পরিণতি—আর্থিক ক্ষতি, গোপনীয়তা লঙ্ঘন, নিরাপত্তা ঝুঁকি, বা অসতর্কতার ফলে তৈরি unintended side effects। Harm এড়াতে হলে ঝুঁকিগুলো আগে থেকেই চিহ্নিত ও mitigate করতে হয়।
3. **1.3 Be honest and trustworthy:** Engineering-এ সততা মানে কেবল মিথ্যা না বলা নয়—এর অর্থ হলো **system-এর সক্ষমতা, সীমাবদ্ধতা, এবং সম্ভাব্য ঝুঁকি সম্পূর্ণভাবে disclose করা**। যদি কোনো edge case বা production concurrency এখনো verify করা না হয়ে থাকে, তবে *"সব ১০০% tested এবং production-ready"* দাবি করা সরাসরি পেশাদার অসততা!
4. **2.1 Strive to achieve high quality in both the processes and products of professional work:** কেবল চটজলদি output নয়—specification, design, verification ও documentation-এর process-এও উচ্চ মান বজায় রাখা।
5. **2.2 Maintain high standards of professional competence, conduct, and ethical practice:** যে technology বা AI-generated code আপনি পুরোপুরি বোঝেন না, তা অন্ধভাবে production-এ না চালিয়ে নিজের competence যাচাই ও উন্নত করা, এবং সীমা জানলে স্পষ্ট করে বলা।
6. **2.5 Give comprehensive and thorough evaluations of computer systems and their impacts, including analysis of possible risks:** যখন আপনি কোনো design, ADR, বা code review মূল্যায়ন করেন, তখন সুবিধাজনক অংশগুলো দেখিয়ে অস্বস্তিকর ঝুঁকি লুকানো যাবে না—বস্তুনিষ্ঠ, সৎ risk analysis দিতে হবে।

লক্ষ করুন, এই ছয়টি নীতি কোনো দার্শনিক বক্তৃতা নয়—এগুলো সরাসরি আপনার **PR description**, **ADR-এর downsides section**, **code review**, এবং **ownership sign-off**-এর ব্যবহারিক মানদণ্ড!

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-08-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন engineer বললেন: *"আমাদের টিমে যেহেতু blameless postmortem সংস্কৃতি আছে, তাই আমার PR-এ AI-generated bug থেকে production-এ double-charge হলেও ওটা আসলে process-এর সমস্যা—আমার ব্যক্তিগত accountability বলে কিছু নেই।"* **Individual Responsibility** বনাম **Distributed Organizational Accountability** এবং **Accountable Stewardship** ব্যবহার করে এই যুক্তিটির ভুল কোথায় তা ২–৪ বাক্যে ব্যাখ্যা করুন।
> 2. শুক্রবার বিকেলে আপনার ম্যানেজার বললেন: *"নতুন refund API-টা আজই live করতে হবে—concurrent duplicate-request tests পরে লিখো, আপাতত staging-এ একটা happy-path test চালিয়ে deploy করে দাও আর client-কে বলো সব verified!"* **ACM Code of Ethics (2018)**-এর নির্দিষ্ট clauses (`1.2`, `1.3`, `2.5`) উদ্ধৃত করে আপনি কীভাবে পেশাদারভাবে সাড়া দেবেন?

এবার আমাদের opening incident-এর দ্বিতীয় ভাঙা জায়গায় আসুন। Author যখন ৯০০ লাইনের unreviewed, AI-assisted PR জমা দিলেন আর Reviewer Friday বিকেলে `LGTM` দিয়ে approve করে দিলেন—তখন সমস্যাটা শুধু দুজন মানুষের অবহেলা ছিল না, বরং **Code Review-র প্রকৃত মানদণ্ড ও দায়িত্ববণ্টন** নিয়েই মৌলিক ভুল বোঝাবুঝি ছিল। একজন accountable engineer হিসেবে **Author** ও **Reviewer**—দুই ভূমিকাতেই disciplined code review কীভাবে কাজ করে?

---

## Disciplined Code Review: Author Responsibility এবং Reviewer Rigor

প্রথমেই software industry-র সবচেয়ে ক্ষতিকর একটি misconception চিরতরে ভেঙে দিন:

> **Misconception:** *"Code review হলো reviewer-এর কাজ আমার কোডের bug খুঁজে বের করা। আমি দ্রুত PR খুলে দিই—কোনো ভুল থাকলে reviewer-ই তো ধরে দেবে!"*

কেন এটি ভয়ংকর ভুল?
কারণ **Author হলেন change-টির correctness, clarity, এবং verification-এর প্রথম ও প্রধান মালিক।** *Software Engineering at Google* (Chapter 9 — *Code Review*) এবং Google-এর *Code Review Developer Guide* পরিষ্কারভাবে দেখায়: code review-র উদ্দেশ্য author-এর প্রাথমিক testing ও thinking-এর বিকল্প হওয়া নয়। বরং code review হলো একটি **second layer of defense**, shared codebase comprehension তৈরি করার মাধ্যম, এবং সময়ের সাথে পুরো codebase-এর **code health** বজায় রাখার ব্যবস্থা।

আপনি যখন AI দিয়ে generate করা বা তাড়াহুড়ো করে লেখা ৯০০ লাইনের unverified PR আরেকজন teammate-এর review queue-তে ছুঁড়ে দেন, তখন দুটি ক্ষতি একসাথে ঘটে:
1. আপনি নিজের verification-এর শ্রম কোনো সম্মতি ছাড়াই অন্য engineer-এর ঘাড়ে চাপিয়ে দেন (**uncompensated review burden**)।
2. বিশাল ও অস্পষ্ট PR দেখে reviewer-এর মনোযোগ ও ধৈর্য ভেঙে পড়ে (**review fatigue**)—ফলে আসল design flaw, security vulnerability, বা concurrency bug চোখ এড়িয়ে যায় এবং reviewer কেবল formatting বা variable naming-এ মন্তব্য করে `LGTM` দিয়ে দেন!

তাই disciplined code review দাঁড়িয়ে থাকে **দুটি সমান গুরুত্বপূর্ণ স্তম্ভের** ওপর: **Author-এর দায়িত্ব** এবং **Reviewer-এর মানদণ্ড**।

### ১. Author Responsibility in Code Review (Google-এর *CL Author's Guide*)

কোনো teammate-এর কাছে review চাওয়ার *আগেই* একজন accountable author চারটি কাজ নিশ্চিত করেন *(Google-এর guide-এ PR-কে "CL" বা Changelist বলা হয়—ধারণাটি GitHub Pull Request-এর ক্ষেত্রে হুবহু একই)*:

1. **Change-টি ছোট ও স্বয়ংসম্পূর্ণ রাখুন (*Small CLs*):**
   একটি PR সাধারণত **একটিমাত্র cohesive কাজ** করবে (যেমন একটি bug fix বা একটি bounded feature slice)। Google-এর *Small CLs* guide দেখায় কেন small changes এত শক্তিশালী:
   - Reviewer-রা ছোট PR **অনেক দ্রুত** এবং **অনেক বেশি গভীরভাবে** review করতে পারেন (একবারে ৩০–৬০ মিনিট বের করার চেয়ে ৫–১০ মিনিটে সূক্ষ্ম bug ধরা সহজ)।
   - Bug ঢোকার সম্ভাবনা কমে, merge conflict কমে, design polish করা সহজ হয়, আর production-এ সমস্যা হলে **rollback** করা অনেক নিরাপদ হয়।
   - **গুরুত্বপূর্ণ নিয়ম:** Behavior-changing feature code আর pure refactoring আলাদা PR-এ রাখুন—কিন্তু **সংশ্লিষ্ট automated tests সবসময় code-এর সাথে একই PR-এ রাখুন** (যাতে reviewer behavior এবং তার proof একসাথে দেখতে পারেন)। এবং মনে রাখবেন: কোনো PR অযথা বিশাল হলে reviewer-এর পুরো অধিকার আছে কেবল আকারের কারণেই সেটি reject করে ছোট অংশে ভাগ করে আনতে বলার! *(Block 7-এর DORA "Working in small batches"-এর সরাসরি বাস্তব প্রয়োগ!)*
2. **স্পষ্ট ও তথ্যপূর্ণ PR / CL Description লিখুন (*Writing good CL descriptions*):**
   PR description কোনো সাময়িক chat message নয়—এটি version control history-র একটি **permanent public record**, যা ছয় মাস বা দুই বছর পর অন্য engineer-রা (কিংবা আপনি নিজেই!) পড়বেন। Google-এর guide অনুযায়ী প্রতিটি ভালো description দুটি প্রশ্নের পরিষ্কার উত্তর দেয়:
   - **What changed?** (প্রথম লাইনে এক বাক্যের সংক্ষিপ্ত imperative summary—যেমন `Enforce per-user cap and idempotent retries in promo redemption`—না যে `Update promo stuff`!)
   - **Why was this change made?** কোড পড়লে বোঝা যায় সিস্টেম *কী করছে*, কিন্তু অনেক সময় বোঝা যায় না *কেন* এটি এভাবে করা হলো, কোন constraints ছিল, কোন alternatives বাদ দেওয়া হয়েছে, এবং কীভাবে এটি verify করা হয়েছে। এটি না লিখলে ভবিষ্যতের engineer **Chesterton's fence** সমস্যায় পড়েন—অর্থাৎ পুরোনো কোনো অদ্ভুত check বা guard কেন রাখা হয়েছিল তা না জেনে সেটি মুছে ফেলে পুরোনো bug ফিরিয়ে আনেন!
3. **নিজে আগে Line-by-Line Self-Review ও Independent Verification করুন:**
   Review request পাঠানোর আগে নিজের diff-এর প্রতিটি লাইন নিজে পড়ুন (বিশেষত AI-assisted অংশগুলো—Block 7!), সব automated tests ও static checks independently run করুন, এবং reviewer-এর জন্য কোনো subtle design trade-off বা tricky edge case থাকলে PR-এ upfront note লিখে দিন।
4. **Review Feedback-কে ব্যক্তিগত আক্রমণ নয়, shared engineering rigor হিসেবে নিন:**
   Reviewer যখন design বা edge case নিয়ে প্রশ্ন তোলেন, তখন defensive না হয়ে evidence ও trade-offs দিয়ে আলোচনা করুন।

### ২. Reviewer Rigor (Google-এর *How to do a code review*)

এবার মুদ্রার অন্য পিঠে আসুন—যখন আপনি নিজে **Reviewer**। অনেক নতুন reviewer দুটি চরম প্রান্তের একটিতে পড়ে যান: হয় তাঁরা কেবল `LGTM` দিয়ে রাবার-স্ট্যাম্প মারেন, নয়তো নিজের ব্যক্তিগত style পছন্দ চাপিয়ে দিতে গিয়ে অপ্রয়োজনীয় perfectionism-এ পুরো টিমের কাজ আটকে দেন!

Google-এর ***The Standard of Code Review*** এই দুটি চরম প্রান্তের মাঝখানে একটি অসাধারণ senior rule দাঁড় করিয়েছে:

> **The Standard of Code Review:**
> *"In general, reviewers should **favor approving a CL** once it is in a state where it **definitely improves the overall code health** of the system being worked on, even if the CL isn't perfect."*

এই নীতিটির পেছনের ভারসাম্যটা ভালো করে বুঝুন:
- **"Perfect" code বলে কিছু নেই—আছে কেবল *better* code।** Reviewer যদি প্রতিটি ছোট stylistic পছন্দ না মেলা পর্যন্ত PR আটকে রাখেন, তবে developer-দের অগ্রগতি (velocity) থেমে যায় এবং ভবিষ্যতে তাঁরা codebase improve করতে নিরুৎসাহিত হন।
- **আবার অন্যদিকে**, codebase এক দিনে নষ্ট হয় না—নষ্ট হয় তাড়াহুড়োর অজুহাতে জমতে থাকা ছোট ছোট shortcut-এ (Block 6-এর tactical programming!)। তাই কোনো PR যদি correctness ঝুঁকি, security flaw, দুর্বল tests, বা অকারণ complexity ঢুকিয়ে **overall code health কমিয়ে দেয়**, তবে তা approval পাওয়ার যোগ্য নয়।

তাহলে একজন disciplined reviewer একটি PR খুলে ঠিক কোন কোন মাত্রা (dimensions) পদ্ধতিগতভাবে পরীক্ষা করেন? Google-এর ***What to look for in a code review***-এ এর একটি পরিষ্কার ক্রম আছে:

| Review Dimension | Reviewer নিজেকে ঠিক কী প্রশ্ন করবেন | Phase 1-এর কোন Block-এর সাথে যুক্ত |
| :--- | :--- | :--- |
| **1. Design** *(সবচেয়ে গুরুত্বপূর্ণ)* | কোডের বিভিন্ন অংশের interaction কি যৌক্তিক? এই logic কি এখানেই থাকা উচিত? এটি কি বিদ্যমান system ও future changes-এর সাথে পরিষ্কারভাবে fit করে? | Blocks 1, 4, 6 |
| **2. Functionality** | এটি কি শুধু happy path নয়, বরং end-users ও future developers উভয়ের জন্যই ঠিক কাজ করে? Edge cases, invalid inputs, error handling, বা concurrency/idempotency bug কি লুকিয়ে আছে? | Blocks 2, 3, 7 |
| **3. Complexity** | কোডটা কি প্রয়োজনের চেয়ে বেশি জটিল (overengineered, shallow pass-through wrappers, hidden side effects, complected concerns)? আরেকজন engineer কি দ্রুত এটি বুঝে safely modify করতে পারবেন? | Block 6 |
| **4. Tests** | Automated tests কি সঠিক, বোধগম্য, এবং useful? **সবচেয়ে বড় পরীক্ষা:** আসল production code ভেঙে দিলে কি এই test-গুলো সত্যিই fail করবে (নাকি শুধু AI-র happy path বা mock behavior confirm করছে)? | Blocks 3, 5, 7 |
| **5. Naming, Comments & Clarity** | Variable/function নামগুলো কি স্পষ্টভাবে তাদের আসল উদ্দেশ্য ও side effects বোঝায়? Comments কি কোড *কী করছে* তা পুনরাবৃত্তি না করে *কেন* করছে তা ব্যাখ্যা করে? | Blocks 1, 6 |
| **6. Every Line & Context** | আমি কি শুধু বদলানো ৩ লাইন নয়, আশেপাশের context-সহ **প্রতিটি লাইন** বুঝে পড়েছি? যে লাইন আমি বুঝতে পারছি না, তার মানে পরবর্তী engineer-ও বুঝতে পারবেন না—অতএব clarification বা simplification চাইতে হবে! | Blocks 6, 7, 8 |

এবং review comment লেখার সময় একটি দারুণ ব্যবহারিক অভ্যাস হলো **severity স্পষ্ট করা**:
- **Blocking issues** (correctness bug, missing invariant, security flaw, broken error contract, severe complexity/test gap) স্পষ্ট reason ও evidence দিয়ে চিহ্নিত করুন।
- **Optional minor suggestions** (ছোট stylistic বা readability উন্নয়ন যা overall code health আটকাচ্ছে না) আলাদা করে **`Nit:`** (বা `Optional:`) label দিয়ে লিখুন, যাতে author পরিষ্কার বোঝেন কোনটা merge-এর জন্য mandatory আর কোনটা ঐচ্ছিক। আর কোডে ভালো কিছু দেখলে তার প্রশংসাও করুন—কোডের সমালোচনা করুন, মানুষের নয়।

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-08-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন teammate আপনার কাছে একটি 60-line PR review করতে দিলেন। PR-টি একটি বাস্তব rounding bug ঠিক করে এবং তার জন্য ৩টি ভালো regression test যোগ করেছে; তবে দুই জায়গায় আপনি হলে `for` loop-এর বদলে `.reduce()` লিখতেন, আর একটি helper variable-এর নাম একটু আরও সুন্দর করা যেত। Google-এর **The Standard of Code Review** অনুযায়ী আপনার সিদ্ধান্ত কী হওয়া উচিত—এবং কেন?
> 2. এবার উল্টো দৃশ্য ভাবুন: আরেকজন teammate AI-র সাহায্যে একটি 550-line PR পাঠালেন যার description-এ শুধু লেখা *"Refactor checkout and add loyalty points"*, যেখানে পুরোনো code-এর formatting বদল, নতুন feature, আর একটি নতুন `npm` package একসাথে মেশানো, এবং unit test মাত্র ১টি। Reviewer হিসেবে Google-এর **Small CLs**, **CL Descriptions**, এবং **What to look for** অনুযায়ী আপনি কোন ৩টি concrete কারণে এখনই PR-টি approve করবেন না?

এবার আমাদের opening incident-এর তৃতীয় ও শেষ অজুহাতে আসুন। Author বলেছিলেন: *"Deploy তো pipeline করেছে, production-এ কী ঘটল দেখা তো Ops-এর কাজ—আমি শুধু code লিখেছি!"* Code review পাস হয়ে PR merge হয়ে যাওয়ার পর একজন accountable engineer-এর দায়িত্ব ঠিক কোথায় গিয়ে থামে—আর নিজের technical সিদ্ধান্তগুলোকে তিনি কীভাবে defend করেন?

---

## Defense of Decisions এবং Operational Ownership ("You Build It, You Run It")

একটি feature যখন আপনার হাত দিয়ে merge হয়, তখন আপনি কেবল কিছু JavaScript বা SQL লাইন commit করছেন না—আপনি আপনার টিম ও গ্রাহকদের ওপর একটি **দীর্ঘমেয়াদি operational ও maintenance প্রতিশ্রুতি** গ্রহণ করছেন।

এই প্রতিশ্রুতি দুটি স্পষ্ট অভ্যাসে প্রকাশ পায়: **Defense of Decisions** *(Practical Competence)* এবং **Operational Ownership** *(Awareness)*।

### ১. Defense of Decisions (নিজের সিদ্ধান্ত যুক্তি ও প্রমাণ দিয়ে দাঁড় করানো)

একজন senior-trajectory backend engineer-কে যখন তাঁর লেখা design বা code নিয়ে কঠিন প্রশ্ন করা হয়—হোক তা PR review-তে, design review-তে, কিংবা production incident-এর পর—তখন তিনি কখনোই বলেন না *"AI এটা suggest করেছিল"*, *"আগের ফাইলটায় এভাবেই ছিল"*, বা *"আমার মনে হয়েছিল এটাই best practice"*।

বরং **Defense of Decisions** মানে হলো নিজের তৈরি প্রতিটি কাজের জন্য চারটি প্রশ্নের সৎ, স্পষ্ট ও evidence-backed উত্তর দিতে পারা:

1. **Problem Framing Defense:** *"কেন আমরা এই requirement-কে ঠিক এইভাবে interpret করলাম, কোনগুলো verified facts আর কোনগুলো explicit assumptions, এবং কী কী সচেতনভাবে Non-Goal রাখা হয়েছে?"* *(Block 2)*
2. **Design & Trade-Off Defense:** *"অন্য কোন viable alternatives আমরা বিবেচনা করেছিলাম, কোন constraints-এর কারণে এই নকশাটা বেছে নেওয়া হলো, এই chosen option-টির নিজস্ব downsides কী, এবং কেন আরও complex/overengineered বিকল্পগুলো বাদ দেওয়া হলো?"* *(Blocks 4 & 6)*
3. **Verification & Honesty Defense:** *"কোন invariants ও edge cases আমি empirically verify করেছি (কোন tests/measurements দিয়ে), আর এই মুহূর্তে কী কী এখনো unverified রয়ে গেছে?"* *(Blocks 3, 5, 7 + ACM 1.3)*
4. **Failure & Evolution Defense:** *"Production-এ এই component-টা সবার আগে কোন অবস্থায় কীভাবে ভাঙতে পারে, এর blast radius কতটা, আর ভবিষ্যতে ঠিক কোন measurable evidence দেখলে আমরা এই decision-টি revisit করব?"* *(Blocks 1, 3, 4, 5)*

লক্ষ করুন: আপনি যদি নিজের তৈরি code বা AI-assisted solution-এর জন্য এই চারটি প্রশ্নের উত্তর দিতে না পারেন, তবে সেই কাজটি এখনো ship করার জন্য প্রস্তুত নয়!

### ২. Operational Ownership — "You Build It, You Run It" *(Awareness)*

ঐতিহ্যবাহী বহু প্রতিষ্ঠানে একসময় development আর operations-এর মাঝখানে একটি অদৃশ্য দেয়াল ছিল: developer-রা code লিখে আলাদা Ops টিমের হাতে ছেড়ে দিতেন এবং তারপর আর পেছনে ফিরে তাকাতেন না। ২০০৬ সালের *ACM Queue* সাক্ষাৎকার [*"A Conversation with Werner Vogels"*](https://queue.acm.org/detail.cfm?id=1142065)-এ Amazon-এর CTO **Werner Vogels** এই পুরোনো handoff মডেলের বদলে **"You build it, you run it"** নীতিটি সামনে আনেন—তাঁর মূল বক্তব্য ছিল, যে টিম একটি service তৈরি করে, তার দৈনন্দিন operation-এর দায়িত্বও সেই টিমেরই থাকা উচিত, যাতে developer-রা সরাসরি নিজেদের software-এর live আচরণ এবং বাস্তব customer অভিজ্ঞতার সংস্পর্শে থাকেন।

এই ধারণার পেছনের **mechanism** খুব সহজ কিন্তু অত্যন্ত শক্তিশালী: **Feedback Loop ও Incentive Alignment**।
- যখন code লেখার মানুষ আর production-এ রাত ৩টায় incident সামলানোর মানুষ আলাদা হয়, তখন developer-এর সামনে silent error-swallowing, cryptic error message, missing timeout, বা risky shortcut নেওয়ার তাৎক্ষণিক কোনো কষ্ট নেই—কষ্টটা ভোগ করে অন্য একটি টিম!
- কিন্তু যখন আপনার টিম নিজেরাই নিজের service-এর operational health-এর মালিক হয়, তখন coding-এর সময়ই আপনার ভাবনা বদলে যায়: *"রাত ৩টায় এই promo operation fail করলে আমি log দেখে ২ মিনিটে বুঝতে পারব তো কেন fail করল? এই error-টা কি স্পষ্ট? Rollback করা কি নিরাপদ?"*

*(Scope guardrail: Phase 1-এ Operational Ownership আমাদের **Awareness** স্তরের mindset—যাতে আপনি প্রথম দিন থেকেই "code merge = কাজ শেষ" ভ্রম থেকে বেরিয়ে আসেন। বাস্তব production telemetry, logs/metrics/traces, SLOs, retries/circuit breakers, Linux hosts, Docker ও CI/CD operational mechanics আমরা ধাপে ধাপে **Phases 20–24**-এ গভীরভাবে শিখব।)*

---

## পুরো Phase 1 কীভাবে একটি অখণ্ড Engineering Loop তৈরি করে

এখন আপনি প্রথমবারের মতো Phase 1-এর পুরো আটটি block-কে বিচ্ছিন্ন পাঠ হিসেবে নয়, বরং একটি **একক, ধারাবাহিক engineering loop** হিসেবে দেখতে পাচ্ছেন—যার প্রতিটি ধাপ পরের ধাপকে অনিবার্য করে তোলে:

| ধাপ | Block | একজন Accountable Engineer-এর মূল প্রশ্ন | পরের ধাপে এর Output কী দেয় |
| :---: | :--- | :--- | :--- |
| **1** | **Block 1** *(Software Engineering)* | এই কাজটির expected lifespan, scale, Hyrum's Law ঝুঁকি ও proportional rigor কতটা? | ঠিক কতটা engineering depth ও contract stability দরকার তা নির্ধারণ করে। |
| **2** | **Block 2** *(Problem Before Coding)* | আসল problem কী, কোনগুলো facts বনাম assumptions, কী Non-Goal, আর verifiable BCP 14 spec কী? | Unambiguous requirements তৈরি করে—যার ছাড়া correctness মাপা অসম্ভব। |
| **3** | **Block 3** *(Correctness & Quality)* | কোন invariants সর্বদা অটুট থাকতে হবে, কী Fault → Error → Failure ঘটতে পারে, কোন ISO 25010 quality attributes টানাপোড়েনে আছে? | Design ও test suite-এর জন্য non-negotiable correctness boundary দেয়। |
| **4** | **Block 4** *(Constraints & Trade-offs)* | ৬ মাত্রার constraints-এর ভেতর কোন viable alternatives আছে, chosen option-এর downsides কী, আর revisit triggers কী? | একটি documented ADR তৈরি করে যা implementation-কে constraint-fit রাখে। |
| **5** | **Block 5** *(Evidence-Based Engineering)* | আমার কোন claims empirical evidence, আর কোনগুলো কেবল hypothesis বা opinion? কীভাবে falsify করব? | Measurement ও falsifiable verification plan দেয়—anti-methods আটকায়। |
| **6** | **Block 6** *(Simplicity & Maintainability)* | কোনটা essential complexity আর কোনটা accidental? Solution-টা কি simple, deep, এবং overengineering-মুক্ত? | Code-কে দীর্ঘমেয়াদে readable, testable ও modifiable রাখে। |
| **7** | **Block 7** *(AI-Augmented Engineering)* | কোন bounded sub-task AI-কে delegate করব, কী context packet দেব, আর কোন independent verification harness চালাব? | Probabilistic AI output-কে deterministic checks ও audit দিয়ে নিয়ন্ত্রণ করে। |
| **8** | **Block 8** *(Ownership & Capstone Lab)* | এই change কি small, reviewable, ethically honest, empirically defended, এবং production impact-সহ আমার নিজের owned? | সম্পূর্ণ end-to-end engineering accountability ও sign-off নিশ্চিত করে। |

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-08-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. আপনার তৈরি একটি নতুন discount calculation module-এ সব local unit test pass করেছে, কিন্তু আপনি এখনো real concurrent database load-এ এটি test করেননি। **Defense of Decisions** এবং **ACM 1.3 / 2.5** মেনে PR description-এ আপনি এই অবস্থানটি কীভাবে উপস্থাপন করবেন—এবং কোন ধরনের phrasing এড়িয়ে চলবেন?
> 2. *"You build it, you run it"* (Werner Vogels, 2006) মানসিকতাটি ঠিক কোন **incentive / feedback-loop mechanism**-এর মাধ্যমে developer-দের day-to-day coding ও error-handling সিদ্ধান্ত বদলে দেয়? (কোনো Phase 20–24 tooling নাম মুখস্থ না বলে mechanism-টা ২–৩ বাক্যে ব্যাখ্যা করুন।)

এবার সময় এসেছে শুধু নীতি পড়া থামিয়ে, **নিজে একজন accountable backend engineer হয়ে কাজ করে দেখানোর!** নিচে আগে Block 8-এর official learning objectives ও mandatory reading scope গুছিয়ে দেওয়া হলো, তারপর রয়েছে আপনার **Pre-Lab Code Review Exercise** এবং পুরো Phase 1-এর চূড়ান্ত **4-Stage Capstone Practical Laboratory**।

---

## Study Plan, Learning Objectives, Terminology & Resources

> **Continuity & Ownership Notes:** আপনার সব Pause & Checks, Pre-Lab Code Review, Capstone Lab-এর লিখিত artifacts (`ADR`, briefs, audit log, sign-off), এবং (যদি করতে চান) optional closed-book synthesis assessment স্বাধীনভাবে `phases/phase-01/notes/block-08-notes.md` ফাইলে লিখুন; আর Capstone Lab Stage 3-এর runnable Pure Node.js code ও tests রাখুন `phases/phase-01/lab/promo-redemption-capstone/` ফোল্ডারে। AI-এর প্রস্তুতির সময় বা agent-generated material কখনো learner study hours বা completed learning হিসেবে গণ্য হয় না।
>
> **Progression & Phase 1 Checkpoint Policy:** Block 8-এর Closed-Book Synthesis Assessment সম্পূর্ণ **optional**। আপনি পড়া ও lab শেষ করে assessment ছাড়া নিজের study সম্পন্ন করলে এই block-এর status হবে **`Studied — mastery not formally verified`**। তবে পুরো **Phase 1-এর `approved` sign-off** (এবং Phase 2 শুরু করার আগে formal mastery verification) সম্পূর্ণভাবে **ChatGPT (Curriculum Owner & Instructor)**-এর এখতিয়ারাধীন—আপনি যখন প্রস্তুত হবেন, তখন আপনার learner-authored notes, Block 8 Capstone Lab artifacts, এবং (চাইলে) optional assessments `phases/phase-01/checkpoint.md`-এ সংকলন করে ChatGPT-র কাছে review-এর জন্য share করবেন।

### Study Time Reference

*প্রতিটি activity-র বিস্তারিত ও সৎ time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। Approved Research Dossier-এ Block 8-এর `4 hours` target estimate (`PROGRESS.md` / `README.md`: `4–6 hours`) একটি flexible planning reference হিসেবে অপরিবর্তিত আছে; কোনো activity-রই strict deadline নেই।*

### Block 8 Learning Objectives *(Primary: Outcome 10; Integrative Supporting: Outcomes 1–9)*

- **LO 8.1 (`Practical Competence`):** Demonstrate **engineering ownership and professional accountability**: individuals **cannot abdicate responsibility for work they author, approve, or ship** (never deflecting blame to an AI tool or vague ticket), while recognizing that **organizational accountability is distributed** across teams, reviewers, leadership, and operational systems. *(Cognitive focus: Evaluate / Demonstrate — conceptual ethics ও accountability নীতিগুলো প্রথমে বুঝে Block 8-এর code review এবং capstone ownership sign-off-এ প্রয়োগ করা।)*
- **LO 8.2 (`Working Understanding`):** Apply disciplined **code review** standards (both as author preparing reviewable, small, well-explained changes and as reviewer verifying design, functionality, complexity, tests, and readability). *(Cognitive focus: Apply / Evaluate।)*
- **LO 8.3 (`Practical Competence`):** Synthesize **all eight Phase 1 blocks** in an end-to-end Practical Laboratory: framing an ambiguous backend problem, surfacing assumptions/constraints, evaluating trade-offs in an ADR, using AI deliberately under strict context and verification discipline, conducting an independent audit, and taking explicit sign-off accountability. *(Cognitive focus: Synthesize / Create / Defend।)*

**Expected Outcomes:** Block 8 সম্পন্ন করে আপনি নিজের তৈরি lab artifact-এর প্রতিটি requirement interpretation, trade-off, verification step, ও code line স্বাধীনভাবে defend করতে পারবেন (কখনোই *"the AI suggested it"* না বলে), এবং Google-এর review dimensions (*design, functionality, complexity, tests, naming/clarity*) ব্যবহার করে একটি sample change-এর ওপর structured code review চালাতে পারবেন।

### Terminology Inventory

| Term | Official Target Level | Why It Matters in Block 8 |
| :--- | :---: | :--- |
| **Engineering Ownership & Accountability** | `Practical Competence` | Problem framing থেকে verification, deployment impact ও maintenance পর্যন্ত ফলাফলের সৎ দায়িত্ব নেওয়া। |
| **Author Responsibility in Code Review** | `Practical Competence` | Unverified বা AI-generated বড় PR জমা দিলে teammate-দের ওপর অযাচিত review burden চাপে এবং team trust নষ্ট হয়। |
| **Professional Ethics & Harm Avoidance** | `Working Understanding` | Technical সিদ্ধান্ত real users-এর অর্থ, privacy, নিরাপত্তা ও আস্থাকে প্রভাবিত করে—শুধু abstract puzzle নয়। |
| **Defense of Decisions** | `Practical Competence` | কোড বা design কেন এমন, কোন alternatives বাদ পড়েছে, কী verified/unverified, আর সবার আগে কী ভাঙতে পারে তা স্পষ্টভাবে ব্যাখ্যা করা। |
| **Operational Ownership ("You Build It, You Run It")** | `Awareness` | Code লেখা শেষ ধাপ নয়—তা operate ও debug করার দায়িত্বও engineering ownership-এর অংশ (বিস্তারিত Phases 20–24-এ)। |

### Mandatory Authoritative Resources *(Targeted Reading — ~40–50 min total)*

1. **Mandatory 1 (~25–30 min — targeted sections only):** Google Engineering Practices — [Code Review Developer Guide](https://google.github.io/eng-practices/) (`LO 8.1`, `LO 8.2`)।
   - **Assigned scope (৪টি নির্দিষ্ট page পড়ুন):**
     - [The Standard of Code Review](https://google.github.io/eng-practices/review/reviewer/standard.html) — *overall code health* নীতি, perfection বনাম improvement ভারসাম্য।
     - [What to look for in a code review](https://google.github.io/eng-practices/review/reviewer/looking-for.html) — *Design, Functionality, Complexity, Tests, Naming, Comments, Every Line, Context*।
     - [Writing good CL descriptions](https://google.github.io/eng-practices/review/developer/cl-descriptions.html) — *First Line*, *What* ও *Why*, এবং *Chesterton's fence*।
     - [Small CLs](https://google.github.io/eng-practices/review/developer/small-cls.html) — *Why Write Small CLs?*, *What is Small?*, *Separate Out Refactorings*, এবং *Keep related test code in the same CL*।
   - **What to extract:** Author-এর প্রাথমিক দায়িত্ব কী, কোন condition-এ reviewer approve বা block করবেন, এবং কীভাবে ছোট, well-explained change team velocity ও quality উভয়ই রক্ষা করে।
2. **Mandatory 2 (~15–20 min — targeted clauses only):** ACM — [ACM Code of Ethics and Professional Conduct (2018)](https://www.acm.org/code-of-ethics) (`LO 8.1`, Outcome 10)।
   - **Assigned scope:** কেবল **Section 1 (General Ethical Principles):** `1.1` (*Contribute to society and to human well-being*), `1.2` (*Avoid harm*), `1.3` (*Be honest and trustworthy*) এবং **Section 2 (Professional Responsibilities):** `2.1` (*Strive to achieve high quality*), `2.2` (*Maintain high standards of professional competence*), `2.5` (*Give comprehensive and thorough evaluations of computer systems and their impacts, including analysis of possible risks*)।
   - **What to extract:** Harm avoidance, system limitations/risks-এর সৎ disclosure, এবং comprehensive technical evaluation কীভাবে daily backend engineering-এ প্রয়োগ হয়।

### Optional Supplementary & Reference Resources *(Outside Mandatory Budget)*

- **Supplementary (Free):** Tom Manshreck & Caitlin Sadowski, *Software Engineering at Google* (O'Reilly, 2020) — [Chapter 9: Code Review](https://abseil.io/resources/swe-book/html/ch09.html) — *Code Review Benefits* ও *Code Review Best Practices* (`LO 8.1`, `LO 8.2`)।
- **Reference (Free):** ACM / IEEE-CS — [Software Engineering Code of Ethics and Professional Practice (1999)](https://ethics.acm.org/code-of-ethics/software-engineering-code/) — ৮টি Principle (*Public, Client and Employer, Product, Judgment, Management, Profession, Colleagues, Self*), বিশেষত *Product* ও *Judgment* skim করুন (`LO 8.1`)।

---

## Pre-Lab Exercise: Structured Code Review Audit *(Mandatory — ~20–25 min)*

> **Purpose (`LO 8.2`):** Capstone Lab শুরু করার আগে Google-এর Code Review Developer Guide-এর মানদণ্ডগুলো একটি বাস্তবসম্মত ছোট Pull Request-এ প্রয়োগ করে দেখুন। উত্তর লিখুন `phases/phase-01/notes/block-08-notes.md` ফাইলে (AI দিয়ে review লিখাবেন না)।

**Context:** আমাদের online store-এ money সবসময় **integer minor units (`cents`)**-এ হিসাব হয় (Block 3)। একজন developer নিচের Pull Request-টি review-এর জন্য পাঠিয়েছেন:

- **PR Title:** `Update discount stuff`
- **PR Description:** `Fixed discount helper for VIP tiers as requested. AI helped draft this and tests pass.`

```javascript
// applyTierDiscount.mjs — Sample PR diff under review
class DiscountStrategyBase {
  compute(amt) { return amt; }
}
class PercentageStrategy extends DiscountStrategyBase {
  constructor(r) { super(); this.r = r; }
  compute(amt) { return amt - amt * this.r; }
}

// Computes final price and marks promo as used on the cart
export function check(d, u) {
  let x = d.total;
  if (u.tier == "VIP") {
    const s = new PercentageStrategy(0.15);
    x = s.compute(x);
  } else if (u.tier == "GOLD") {
    const s = new PercentageStrategy(0.1);
    x = s.compute(x);
  }
  d.usedPromo = true;
  d.finalTotal = x;
  return x;
}
```

```javascript
// applyTierDiscount.test.mjs — Submitted test file in the same PR
import assert from "node:assert/strict";
import { check } from "./applyTierDiscount.mjs";

const cart = { total: 1000 };
const result = check(cart, { tier: "VIP" });
assert.equal(result, 850);
```

**Your Tasks (`phases/phase-01/notes/block-08-notes.md`):**
1. **Overall Review Verdict (`LO 8.2`):** Google-এর *The Standard of Code Review* ব্যবহার করে বলুন—আপনি কি এই PR-টি **Approve** করবেন নাকি **Request Changes** দেবেন, এবং ২ বাক্যে এর মূল যুক্তি কী?
2. **Dimension-by-Dimension Review Comments (`LO 8.2`):** Google-এর review dimensions (**Design**, **Functionality / Edge Cases**, **Complexity**, **Tests**, **Naming & Comments**) এবং **CL Description** ধরে constructive, polite, actionable review comments লিখুন। প্রতিটি comment-এ স্পষ্টভাবে label করুন কোনটি **Blocking** (code health কমাচ্ছে বা bug/risk তৈরি করছে) এবং কোনটি non-blocking **`Nit:`**।
3. **Author Accountability Rewrite (`LO 8.1`, `LO 8.2`):** আপনি এই PR-টির Author হলে—সংশোধনের পর Google-এর *CL Author's Guide* মেনে এই change-টির জন্য একটি পরিষ্কার **PR Title + PR Description** (What, Why, Edge-case policy, Verification evidence) লিখে দেখান।

---

## Phase 1 Capstone Practical Laboratory: Accountable Promo-Code Redemption *(Mandatory — ~145–170 min across 4 Stages)*

> **Capstone Goal (`LO 8.1`, `LO 8.3`, Integrative Outcomes 1–10):** এটি কোনো বড় production service বা framework project নয়—এটি একটি **engineering-mindset capstone**। এখানে আপনার লক্ষ্য হলো একটি অস্পষ্ট, বাস্তবসম্মত backend feature request-কে প্রথম প্রশ্ন থেকে শুরু করে ADR, mid-lab requirement change reassessment, controlled AI-assisted pure Node.js verification, এবং সৎ engineering ownership sign-off পর্যন্ত end-to-end পরিচালনা ও defend করা।

### Strict Capstone Scope Boundary *(Approved Dossier Guardrail — No Scope Expansion)*

- **Business & Architectural Continuity:** এই feature-টি আমাদের সেই বাড়তে থাকা **Node.js + PostgreSQL online store** (Blocks 1–7)-এর core promo-redemption domain logic হিসেবে তৈরি হচ্ছে, যা পরের sprint-এ store-এর PostgreSQL-backed checkout service-এ integrate হবে। তাই Stage 2-এর ADR ও Stage 4-এর production-failure analysis-এ আপনি বাস্তব Node.js + PostgreSQL backend প্রেক্ষাপট মাথায় রেখে যুক্তি দেবেন।
- **Strictly Allowed for Stage 3 Implementation:**
  - **Pure Node.js 22 only** (ES Modules `.mjs`, **zero external `npm` dependencies**) — continuity ও reproducibility-র জন্য Phase 1-এ Node.js 22 pinned রাখা হয়েছে *(উল্লেখ্য: November 2026 অনুযায়ী Node.js 22 হলো **Maintenance LTS** release, বর্তমান Active LTS নয়)*।
  - একটি ছোট **in-memory model** (`Map` / plain JS objects)।
  - **একটিমাত্র narrowly defined operation:** `redeemPromoCode` (এবং প্রয়োজন হলে এর pure helper function)।
  - একটি minimal test harness — শুধু Node.js 22 built-in **`node:test` + `node:assert/strict`**।
- **Explicitly Prohibited (Do NOT Build These in Phase 1):**
  - কোনো HTTP web framework বা server (Express, Fastify, `http.createServer` routing),
  - কোনো PostgreSQL driver setup, live database, ORM, বা SQL migration,
  - Authentication / authorization system,
  - Distributed locking, Redis, Kafka, বা message queues,
  - Docker, CI/CD, বা production deployment। *(এসব দক্ষতা Phases 9, 10, 14, 16, 17, 18, ও 24-এ গভীরভাবে আসবে।)*

### The Initial Business Request & Background Context

> **Ticket #PROMO-408 (Submitted by Marketing & Product Lead):**
> *"আসন্ন উৎসবের campaign-এর জন্য আমাদের e-commerce backend-এ একটি promo-code redemption operation যোগ করুন। Customer-রা checkout-এ promo code দিলে দ্রুত valid discount apply হতে হবে (percentage বা flat amount), প্রতিটা code যেন নির্দিষ্ট limit-এর বেশি ব্যবহার না হয়, আর একই customer যেন ভুল করে বা ইচ্ছা করে দুবার benefit না নিতে পারেন। ৩ দিনের মধ্যে core logic ready চাই, যেন পরের sprint-এ checkout API-তে লাগানো যায়।"*

**Confirmed Background Facts (যা ইতিমধ্যেই বিদ্যমান codebase ও team context থেকে verified):**
1. **Team & Timeline:** Backend টিমে আছেন **৩ জন engineer** (Node.js + PostgreSQL stack); এই core redemption module-টির জন্য **৩ দিনের** sprint window বরাদ্দ আছে।
2. **Currency Convention:** Store-এর সব monetary amount **integer minor units (`cents`)**-এ রাখা হয় (`subtotalCents`, `discountCents`, `finalTotalCents`)—কখনো floating-point dollars নয়।
3. **Promo Definition Shape (existing catalog fields):** প্রতিটি promo object-এ আছে: `code` (string), `type` (`"PERCENT"` বা `"FIXED"`), `value` (integer — `"PERCENT"` হলে যেমন `10` মানে 10%, `"FIXED"` হলে cents-এ flat amount), `maxTotalRedemptions` (positive integer), `perUserLimit` (positive integer), `expiresAt` (ISO-8601 UTC timestamp string), এবং `isActive` (boolean)।
4. **Caller Request Shape (checkout caller provides):** `orderId` (string), `userId` (string), `subtotalCents` (integer), `promoCode` (string), `requestedAt` (ISO-8601 UTC timestamp string), এবং optional `idempotencyKey` (string)।
5. **Runtime & Dependency Rule:** Node.js 22 (Maintenance LTS as of November 2026, pinned for Phase 1 continuity; ESM `.mjs`); Stage 3-এর lab code-এ কোনো external `npm` package install করা যাবে না।

*(বাস্তব backend ticket-এর মতোই এখানে calculation rules, boundary conditions, duplicate/retry semantics, এবং error contract-এর অনেক গুরুত্বপূর্ণ সিদ্ধান্ত ইচ্ছাকৃতভাবেই অস্পষ্ট রাখা হয়েছে—Stage 1-এ আপনার প্রথম কাজ হলো সেগুলো নিজেই খুঁজে বের করা!)*

### Where to Put Your Lab Deliverables

- **All written engineering artifacts (Stages 1, 2, 3 audit log, and Stage 4 sign-off):** `phases/phase-01/notes/block-08-notes.md` (স্পষ্ট Stage headings দিয়ে সাজান)।
- **Stage 3 runnable code & independent test suite:** `phases/phase-01/lab/promo-redemption-capstone/` ফোল্ডারে দুটি ফাইল তৈরি করুন:
  - `phases/phase-01/lab/promo-redemption-capstone/promoRedemption.mjs`
  - `phases/phase-01/lab/promo-redemption-capstone/promoRedemption.test.mjs`
  - Test command: `node --test phases/phase-01/lab/promo-redemption-capstone/promoRedemption.test.mjs`

---

### Stage 1 — Problem & Requirements Brief *(~35–40 min | Blocks 2 & 3)*

**Why this stage is necessary & how it informs Stage 2:** অস্পষ্ট ticket থেকে সরাসরি architecture বা code-এ ঝাঁপ দিলে ভুল assumptions আপনার data model ও API contract-এ গেঁথে যাবে (Hyrum's Law + expensive rework)। Stage 1-এর লিখিত specification ও invariants-ই Stage 2-এর trade-off comparison এবং Stage 3-এর context packet ও test suite-এর একমাত্র ভিত্তি (source of truth) হবে।

**Required Deliverables (in `phases/phase-01/notes/block-08-notes.md`):**
1. **Epistemic Separation Table:** Ticket ও context থেকে পরিষ্কার **পাঁচটি আলাদা বিভাগে** নথিভুক্ত করুন:
   - **Confirmed Facts** (ওপরের verified facts থেকে),
   - **Assumptions** (কমপক্ষে ৪টি — যা নিশ্চিত করা হয়নি কিন্তু না মানলে এগোনো যাবে না),
   - **Hypotheses** (কমপক্ষে ২টি — testable technical বা failure-mode অনুমান যা Stage 3-এর tests বা ভবিষ্যৎ production telemetry দিয়ে যাচাই/falsify করতে হবে),
   - **Unresolved Stakeholder Questions** (কমপক্ষে ৪টি — যা আসল team-এ Product/Finance/Checkout-কে জিজ্ঞাসা করতে হবে),
   - **Working Decisions (Provisional)** (৩ দিনের sprint এগিয়ে নিতে আপনি আপাতত কোন working rules adopt করছেন, stakeholder confirmation-এর আগে provisional হিসেবে mark করে)।
2. **Ambiguity & Edge-Case Inventory:** কমপক্ষে **৬টি concrete ambiguity বা edge case** (যেমন calculation/rounding, boundary conditions, invalid inputs, বা duplicate-usage semantics) নিজে চিহ্নিত করুন।
3. **Explicit Non-Goals:** কমপক্ষে **৩টি বাস্তবসম্মত Non-Goal** (কোন যুক্তিসঙ্গত feature-গুলো এখন সচেতনভাবে বাদ দিচ্ছেন—যেমন promo stacking, cross-account fraud detection, live DB persistence)।
4. **Verifiable Requirements (RFC 2119 / RFC 8174):** কমপক্ষে **৬টি** verifiable functional ও non-functional requirement লিখুন—uppercase normative keywords (`MUST`, `MUST NOT`, `SHOULD`, `MAY`) সঠিকভাবে ব্যবহার করে, কোনো vague adjective (*"fast"*, *"secure"*, *"normally"*) ছাড়া।
5. **Domain Invariants, Preconditions/Postconditions & Failure Paths:**
   - অন্তত **৪টি core domain invariant** (যেমন `finalTotalCents >= 0`, `0 <= usedCount <= maxTotalRedemptions`, ইত্যাদি) এবং `redeemPromoCode`-এর মূল Preconditions/Postconditions।
   - ২টি সম্ভাব্য **`Fault → Error → Failure`** causal path (Avižienis et al., Block 3)।

**Stage 1 Acceptance Criteria (Self-Review Check):**
- [ ] Facts, Assumptions, Hypotheses, Unresolved Questions, এবং Provisional Working Decisions—এই পাঁচটি শ্রেণি একে অপরের সাথে গুলিয়ে যায়নি।
- [ ] অন্তত ৬টি non-trivial ambiguity/edge case এবং ৩টি genuine Non-Goal স্পষ্টভাবে লেখা আছে।
- [ ] ৬টি requirement-এর প্রতিটিই objectively verifiable (test দিয়ে pass/fail প্রমাণ করা যায়) এবং uppercase BCP 14 keywords মেনে লেখা।
- [ ] অন্তত ৪টি invariant ও ২টি `Fault → Error → Failure` trace স্পষ্টভাবে সংযুক্ত।

---

### Stage 2 — Constraint & Trade-Off ADR + Mid-Lab Complication *(~35–40 min | Blocks 4 & 6)*

**Why this stage is necessary & how it informs Stage 3:** Stage 1-এ আপনি স্থির করেছেন system *কী করবে*; এবার স্থির করতে হবে আমাদের বাস্তব constraints-এর ভেতর *কোন নকশাটি সবচেয়ে কম ক্ষতিকর ("least worst")* এবং কীভাবে আমরা unnecessary accidental complexity এড়াব। এই ADR-ই নির্ধারণ করবে Stage 3-এ আপনি ঠিক কোন module boundaries তৈরি করবেন।

#### Part A: Initial Constraint & Trade-Off ADR (`ADR-001`)
1. **Explicit Constraints Inventory:** Technical, Schedule (3 days), Team (3 Node.js/PostgreSQL engineers), Zero-Dependency Lab Boundary, এবং Future PostgreSQL Integration constraint-গুলো **Hard vs. Soft** হিসেবে আলাদা করুন।
2. **Alternative Comparison (কমপক্ষে ২টি viable alternative + ১টি rejected overengineered alternative):**
   - **Option A (Pure Domain Core + Explicit Thin Coordinator):** Discount calculation ও eligibility validation একটি **pure function**-এ রাখা (`evaluatePromoEligibility(promo, usageStats, request)` — কোনো state mutation নেই) এবং state update ও usage tracking একটি ছোট, স্পষ্ট coordinator/in-memory store wrapper-এ (`createPromoRedemptionService(store)`) আলাদা রাখা।
   - **Option B (Single Stateful Monolithic Service / Class):** Validation, calculation, mutation, ও logging সব একসাথে একটিমাত্র stateful class বা single mutable function-এ রাখা *(অথবা আপনার নিজের তৈরি কোনো ভিন্ন viable Option B)*।
   - **Rejected Overengineered Option (Explicitly Critique):** কেন আপনি এখনই একটি generic multi-rule plugin engine, event-bus architecture, বা 6-layer Repository/Mapper/Facade hierarchy (Block 6: shallow modules, speculative generality) বানাচ্ছেন না তা essential vs. accidental complexity দিয়ে ব্যাখ্যা করুন।
3. **Structured ADR (`Context`, `Decision`, `Alternatives Considered`, `Consequences`, `Revisit Triggers`):**
   - **Honesty rule (Block 4 + ACM 2.5):** আপনার **chosen option-টির কমপক্ষে ২টি বাস্তব downside/risk** স্পষ্টভাবে লিখুন।

#### Part B: Mid-Lab Complication — Changed Information *(Reassessment Required!)*

> **Day 2 Morning Update (নতুন Stakeholder তথ্য — আপনার Stage 1 ও Stage 2 সিদ্ধান্ত পুনর্মূল্যায়ন করুন):**
> আপনি `ADR-001` শেষ করার পরের দিন সকালে দুটি নতুন বাস্তব তথ্য এল:
> 1. **Finance & Checkout Lead:** *"Network timeout-এর কারণে Checkout service একই `orderId` এবং একই `idempotencyKey` দিয়ে `redeemPromoCode` দুইবার call করতে পারে। দ্বিতীয়বার call করলে error দেখানো যাবে না এবং `usedCount` দ্বিতীয়বার বাড়ানো যাবে না—বরং প্রথম সফল redemption-এর **হুবহু একই result** ফেরত দিতে হবে! কিন্তু যদি একই `idempotencyKey` দিয়ে **ভিন্ন** `orderId`, ভিন্ন `userId`, ভিন্ন `subtotalCents`, বা ভিন্ন `promoCode` পাঠানো হয়, তবে সেটি অবশ্যই একটি explicit conflict error দিয়ে reject করতে হবে (যাতে silent bug বা accidental key reuse ধরা পড়ে)!"*
> 2. **Product Lead:** *"অর্ডার cancel হলে পরের sprint-এ আমাদের promo redemption release করার একটি operation (`releasePromoRedemption`) লাগতে পারে—কিন্তু এই ৩ দিনের sprint-এ cancellation feature বানানোর সময় নেই।"*

**Required Reassessment (`ADR-001 Addendum — Mid-Lab Revision`):**
1. **Impact on Stage 1:** এই নতুন তথ্যের ফলে Stage 1-এর কোন **working decisions, requirements, বা invariants** বদলাতে বা যোগ করতে হলো তা স্পষ্টভাবে update করুন (এবং নতুন কোনো **Hypothesis** বা **Unresolved Question** তৈরি হলে—যেমন *"failed validation attempt-এর `idempotencyKey` কি store হবে নাকি শুধু accepted redemption?"*—তা আলাদাভাবে চিহ্নিত করুন)।
2. **Impact on Stage 2 Design (`ADR-001 Addendum`):**
   - Idempotent replay + payload-mismatch conflict detect করতে আপনার chosen design-এ ন্যূনতম কী পরিবর্তন লাগবে (যেমন stored idempotency record-এ request fingerprint ও saved outcome রাখা)?
   - **Overengineering Check (Block 6):** Product Lead-এর future cancellation কথাটার কারণে আজকেই পুরো `releasePromoRedemption` বা complex state machine বানাবেন না—বরং দেখান কীভাবে আজ কোনো speculative bloat ছাড়াই কেবল redemption record-টিকে clean ও evolvable রাখা যায় (Prudent design vs. Speculative generality)!

**Stage 2 Acceptance Criteria (Self-Review Check):**
- [ ] Hard vs. Soft constraints স্পষ্টভাবে চিহ্নিত এবং viable alternatives বস্তুনিষ্ঠভাবে তুলনা করা হয়েছে।
- [ ] Chosen option-এর কমপক্ষে ২টি সৎ downside এবং overengineered alternative reject করার স্পষ্ট Block-6 যুক্তি আছে।
- [ ] Mid-Lab Complication-এর দুটি অংশই (idempotent replay + mismatch conflict, এবং future cancellation-এ overengineering না করা) আপনার Stage 1 ও Stage 2 সিদ্ধান্তে বাস্তবভাবে প্রতিফলিত হয়েছে—কোনো mechanical copy-paste নয়।

---

### Stage 3 — Controlled AI-Assisted Implementation, Verification & Audit Log *(~50–60 min | Blocks 5 & 7)*

**Why this stage is necessary & how it informs Stage 4:** কাগজের specification ও ADR যত ভালোই হোক, আসল engineering পরীক্ষা হয় implementation ও empirical verification-এ। এই ধাপে আপনি Block 7-এর context engineering ও independent verification workflow এবং Block 5-এর empirical testing প্রয়োগ করে এমন একটি runnable module ও test evidence তৈরি করবেন যার প্রতিটি লাইনের দায়িত্ব আপনি Stage 4-এ নিতে পারবেন।

**Required Deliverables:**
1. **Written Context Packet (in `notes/block-08-notes.md`):**
   - আপনার Stage 1+2 (revised) design থেকে একটি **bounded sub-task** বেছে নিন (যেমন pure `evaluatePromoEligibility` function অথবা পুরো `redeemPromoCode` module-এর initial draft) এবং AI coding assistant-কে দেওয়ার জন্য একটি compact, structured **Context Packet** লিখুন যাতে থাকবে: Goal, Non-Goals, exact input/output contract, Node.js 22 LTS ESM + **zero external dependencies**, সব revised invariants ও Mid-Lab idempotency rules, error contract, এবং simplicity conventions (no shallow class hierarchies, no hidden mutations)।
   - *(Transparency rule: আপনি যদি এই ধাপে কোনো external AI coding assistant ব্যবহার না করে নিজেই প্রথম draft লেখেন, তবে স্পষ্টভাবে তা note করুন—কখনোই AI ব্যবহারের ভুয়া log বানাবেন না। AI ব্যবহার করলে আসল Context Packet ও raw output-এ পাওয়া প্রতিটি flaw সৎভাবে নথিভুক্ত করুন।)*
2. **Pure Node.js Implementation (`phases/phase-01/lab/promo-redemption-capstone/promoRedemption.mjs`):**
   - Zero `npm` dependencies, pure ES module (`.mjs`), revised Stage 1+2 specification ও Mid-Lab idempotency contract মেনে পরিষ্কার, simple, readable implementation।
3. **Independent Line-by-Line Audit Log (in `notes/block-08-notes.md`):**
   - AI-generated draft (বা আপনার প্রথম draft) accept করার আগে line-by-line inspect করে লিখুন: কী কী flaw, subtle assumption, edge-case gap, rounding issue, বা accidental complexity ধরা পড়েছিল, exact line/behavior কী ছিল, এবং কীভাবে fix করেছেন। (যদি কোনো অংশে flaw না মিলে থাকে, তবে ঠিক কোন checks ও adversarial cases দিয়ে সেটা confirm করেছেন তা লিখুন।)
4. **Independent Adversarial Test Suite (`phases/phase-01/lab/promo-redemption-capstone/promoRedemption.test.mjs`):**
   - Built-in `node:test` + `node:assert/strict` দিয়ে **কমপক্ষে ১০টি independent test case** লিখুন (AI-র happy-path tests-এ থেমে যাবেন না!), যার মধ্যে নিচের ১০টি মূল scenario অবশ্যই cover করতে হবে:
     1. Valid `"PERCENT"` এবং `"FIXED"` redemption (exact integer calculation সহ),
     2. Percentage fractional-cent rounding edge case (যেমন 15% of 999 cents — আপনার Stage 1-এ নির্ধারিত rounding rule অনুযায়ী exact check),
     3. `"FIXED"` discount > `subtotalCents` boundary (আপনার নির্ধারিত `finalTotalCents >= 0` বা rejection policy check),
     4. Exact `expiresAt` timestamp boundary (`requestedAt === expiresAt` এবং ১ সেকেন্ড পরে),
     5. Inactive বা unknown promo code rejection (এবং usage count অপরিবর্তিত থাকা),
     6. `maxTotalRedemptions` limit পূরণ হয়ে গেলে পরবর্তী redemption reject হওয়া,
     7. `perUserLimit` limit পূরণ হয়ে গেলে একই user-এর নতুন order reject হওয়া,
     8. **Mid-Lab Idempotent Replay:** একই `idempotencyKey` + একই payload দিয়ে দ্বিতীয়বার call করলে হুবহু একই successful result ফেরা এবং `usedCount` মাত্র `1` থাকা,
     9. **Mid-Lab Idempotency Conflict:** একই `idempotencyKey` কিন্তু ভিন্ন `orderId` বা ভিন্ন `promoCode`/`subtotalCents` দিয়ে call করলে explicit conflict error পাওয়া (এবং state mutate না হওয়া),
     10. Invalid inputs (যেমন `subtotalCents <= 0`, non-integer cents, empty promo code) explicit error-এ reject হওয়া।
5. **Empirical Execution Evidence (in `notes/block-08-notes.md`):**
   - `node --test phases/phase-01/lab/promo-redemption-capstone/promoRedemption.test.mjs` চালিয়ে raw test output (এবং যদি fix-এর আগে কোনো failing test থাকে, তবে তার before → after evidence) সংযুক্ত করুন।

**Stage 3 Acceptance Criteria (Self-Review Check):**
- [ ] `promoRedemption.mjs` এবং `promoRedemption.test.mjs`-এ কোনো external `npm` dependency, HTTP framework, বা DB driver নেই।
- [ ] `node --test phases/phase-01/lab/promo-redemption-capstone/promoRedemption.test.mjs` zero errors-এ runs ও passes।
- [ ] কমপক্ষে ১০টি adversarial/edge-case test (Mid-Lab idempotent replay ও payload-mismatch conflict সহ) independently verify করা আছে।
- [ ] Context Packet, সৎ Audit Log, এবং empirical `node --test` output `notes/block-08-notes.md`-এ recorded আছে।

---

### Stage 4 — Engineering Ownership, Review & Defense Statement *(~25–30 min | Blocks 1 & 8)*

**Why this stage is necessary:** Code লেখা ও tests pass করানো মানেই engineering ownership সম্পন্ন নয়। এই শেষ ধাপে আপনি ACM `1.3` (*Be honest and trustworthy*), ACM `2.5` (*thorough evaluation including risks*), এবং Google-এর *CL Author's Guide* প্রয়োগ করে একজন accountable engineer হিসেবে নিজের কাজের সম্পূর্ণ দায়িত্ব ও সীমাবদ্ধতা ঘোষণা করবেন।

**Required Deliverables (in `phases/phase-01/notes/block-08-notes.md` — concise ~1 page):**
1. **Production-Grade CL / PR Description:** Google-এর *Writing good CL descriptions* অনুযায়ী আপনার Stage 3 module-টির জন্য একটি স্পষ্ট PR description লিখুন (1-line imperative summary + What changed + Why + Non-Goals + how it was verified)।
2. **Defense of Key Decisions:** আপনার solution-এর অন্তত **৪টি মূল সিদ্ধান্ত** (যেমন rounding rule, error contract, pure core vs. store boundary, idempotency conflict strategy) নিজের ভাষায় defend করুন—কখনোই *"AI suggest করেছিল"* বা *"এটাই সহজ ছিল"* না বলে।
3. **Verified vs. Unverified Honesty Matrix (ACM 1.3 & 2.5):** দুই কলামে সৎভাবে আলাদা করুন:
   - **Empirically Verified Now** (Stage 3-এর `node --test` কী প্রমাণ করেছে), বনাম
   - **Explicitly Unverified / Out of Lab Scope** (এই pure in-memory lab কী প্রমাণ করেনি—যেমন multi-process concurrency, PostgreSQL transaction isolation/row locking, server crash durability, network timeouts, API authentication, real production load)।
4. **Production Failure Modes & Future PostgreSQL Integration Risks:** এই pure Node.js logic আগামী sprint-এ যখন store-এর live Node.js + PostgreSQL checkout API-তে যাবে, তখন কোন **৩টি concrete failure mode** সবার আগে ঘটতে পারে (যেমন multiple Node.js instances-এর মধ্যে in-memory `Map` share না হওয়া, PostgreSQL-এ `UNIQUE` constraint/atomic transaction ছাড়া concurrent race condition, অথবা API response format-এ Hyrum's Law lock-in)—এবং প্রতিটি মোকাবিলার উচ্চ-স্তরের কৌশল কী?
5. **Revisit Triggers & Operational Ownership Reflection:**
   - ঠিক কোন **measurable evidence** দেখলে (Blocks 4 & 5) আপনি `ADR-001`-এর সিদ্ধান্ত revisit করবেন (অন্তত ২টি concrete trigger)?
   - *"You build it, you run it"* মানসিকতায়—এই promo operation live যাওয়ার পর রাত ৩টায় customer complaints এলে দ্রুত root cause ধরতে আপনার ন্যূনতম কোন **observable signals / structured error information** দরকার হবে?
6. **Personal Engineering Sign-Off:** ২–৩ বাক্যের একটি explicit accountability statement যেখানে আপনি Author হিসেবে এই design, code, tests, এবং documented limitations-এর পূর্ণ দায়িত্ব গ্রহণ করছেন।

**Stage 4 Acceptance Criteria (Self-Review Check):**
- [ ] CL description-এ স্পষ্ট *What*, *Why*, এবং *Verification* আছে।
- [ ] প্রতিটি decision-এর পেছনে আপনার নিজের যুক্তি আছে (AI-র ঘাড়ে কোনো দায় চাপানো নেই)।
- [ ] Verified বনাম Unverified সীমানা সম্পূর্ণ সৎভাবে আলাদা করা হয়েছে (কোনো false certainty নেই)।
- [ ] অন্তত ৩টি বাস্তব production failure mode, ২টি measurable revisit trigger, এবং একটি explicit accountability sign-off আছে।

---

## Closed-Book Synthesis Assessment *(Optional — ~25–30 min)*

এই assessment-টি সম্পূর্ণ **optional**—এটি কোনো mandatory progression gate নয়। আপনি চাইলে:
- সব notes ও browser tabs বন্ধ রেখে নিচের ৫টি Phase-1 synthesis প্রশ্নের নিজের ভাষায় উত্তর `phases/phase-01/notes/block-08-notes.md`-এ লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা আপনি যথেষ্ট বুঝেছেন ও Capstone Lab দিয়ে দক্ষতা প্রমাণ করেছেন মনে করলে এই closed-book assessment skip করতে পারেন (মনে রাখবেন: assessment skip করলে তা automatically `Mastery verified` হয় না);
- অথবা assessment ছাড়াই যেকোনো unclear concept নিয়ে ChatGPT-র সাথে আলোচনা করতে পারেন।

*(Note: `~25–30 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 8.1 / Outcome 10)** **Territorial ownership** বনাম **Accountable stewardship**, এবং **Individual responsibility** বনাম **Distributed organizational accountability**-র পার্থক্য ব্যাখ্যা করুন। কীভাবে একটি টিম একইসাথে **Blameless Postmortem** (Block 5) চর্চা করতে পারে *এবং* প্রতিটি engineer-এর কাছ থেকে কঠোর ব্যক্তিগত ownership প্রত্যাশা করতে পারে?
2. **(LO 8.1 / Outcome 10)** **ACM Code of Ethics and Professional Conduct (2018)**-এর clauses `1.2` (*Avoid harm*), `1.3` (*Be honest and trustworthy*), এবং `2.5` (*thorough evaluation including risks*) কীভাবে একটি backend PR, ADR, এবং AI-assisted feature-এর দৈনন্দিন সিদ্ধান্তে সরাসরি প্রয়োগ হয়—একটি concrete উদাহরণসহ ব্যাখ্যা করুন।
3. **(LO 8.2 / Outcomes 8, 10)** Google-এর **Code Review Developer Guide** অনুযায়ী **The Standard of Code Review**, **Small CLs**, **Good CL Descriptions**, এবং **What to look for** (*Design, Functionality, Complexity, Tests, Naming*) ব্যাখ্যা করুন। কেন ৯০০-লাইনের একটি AI-generated PR জমা দেওয়া Author responsibility-র ব্যর্থতা?
4. **(LO 8.1, LO 8.3 / Outcomes 1, 9, 10)** **Defense of Decisions**-এর চারটি মূল প্রশ্ন কী কী, এবং Werner Vogels-এর **"You build it, you run it"** নীতিটি কোন feedback-loop mechanism-এর মাধ্যমে software quality ও error handling-এর মান বদলে দেয়?
5. **(LO 8.3 / Integrative Outcomes 1–10)** ধরুন একটি নতুন backend feature-এর ticket এল: *"Checkout-এ loyalty points redeem করার সুবিধা দাও।"* এক প্যারায় না লিখে একটি সুসংহত **৮-ধাপের engineering workflow** লিখুন—যেখানে Phase 1-এর প্রতিটি Block (Blocks 1–8)-এর একটি করে সুনির্দিষ্ট mechanism কীভাবে পরের ধাপকে চালিত করে তা স্পষ্টভাবে দেখাবেন।

**Self-Review Criteria for the Optional Synthesis Assessment:**
- প্রতিটি উত্তর শুধু definition মুখস্থ না করে cause-and-effect mechanism, concrete backend উদাহরণ, এবং সীমাবদ্ধতা/trade-off দেখায়।
- Question 5-এ Blocks 1–8 কেবল তালিকার মতো নাম বলা নয়, বরং আগের ধাপের output কীভাবে পরের ধাপের input হয় তা স্পষ্টভাবে যুক্ত।

---

## Phase 1-এর সমাপ্তি ও Phase 2-এর দিকে সেতুবন্ধন

আপনার Capstone Lab-এর Stage 3-এর কথা একটু ভাবুন। সেখানে আপনি কয়েক ডজন promo code ও redemption record একটি in-memory `Map` বা array-তে রেখে test করেছেন—সব কটি test হয়তো **১ মিলিসেকেন্ডেরও কম** সময়ে শেষ হয়ে গেছে!

কিন্তু এবার একটি অস্বস্তিকর বাস্তব প্রশ্ন নিজেকে করুন:
> *আজ আমাদের in-memory store-এ ১০০টি redemption record থাকায় সবকিছু মুহূর্তেই চলছে। কিন্তু এক বছর পর যখন আমাদের online store-এ **১০,০০,০০০ (১ মিলিয়ন)** redemption record জমা হবে, তখন কোন design choice-টা হঠাৎ করে ১ মিলিসেকেন্ড থেকে ৫ সেকেন্ডে ভেঙে পড়বে—আর production ভেঙে পড়ার আগেই আমরা কীভাবে গাণিতিকভাবে growth rate ও resource bottleneck অনুমান করব?*

ঠিক এই প্রশ্নের উত্তর খুঁজতেই আসছে **Phase 2: Complexity and Performance Reasoning**!

### Block 8 Readiness & Phase 1 Checkpoint Handoff Checklist

- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 `notes/block-08-notes.md`-এ লিখে (পরে Self-Check Guidance মিলিয়ে) সম্পন্ন করেছেন।
- [ ] ২টি Mandatory Resource (**Google Code Review Developer Guide**-এর ৪টি নির্দিষ্ট page এবং **ACM Code of Ethics 2018**-এর নির্ধারিত clauses `1.1–1.3`, `2.1–2.2`, `2.5`) পড়েছেন।
- [ ] `phases/phase-01/notes/block-08-notes.md`-এ **Pre-Lab Exercise (Structured Code Review Audit)** সম্পন্ন করেছেন।
- [ ] **Capstone Practical Laboratory Stages 1–4** সম্পন্ন করেছেন (`notes/block-08-notes.md`-এ Brief, `ADR-001` + Mid-Lab Addendum, Context Packet + Audit Log, ও Ownership Defense Sign-Off; এবং `phases/phase-01/lab/promo-redemption-capstone/`-এ runnable `promoRedemption.mjs` + `promoRedemption.test.mjs`, `node --test` passing)।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে **Optional Closed-Book Synthesis Assessment** লিখেছেন।
- [ ] **Phase 1 Checkpoint Handoff (ChatGPT Final Approval):** পুরো Phase 1-এর study শেষ হলে আপনার learner-authored notes ও verified lab evidence `phases/phase-01/checkpoint.md`-এ সংকলন করে **ChatGPT (Curriculum Owner & Instructor)**-এর কাছে formal Phase 1 mastery evaluation ও `approved` sign-off-এর জন্য জমা দিন। *(মনে রাখবেন: শুধুমাত্র ChatGPT-ই Phase 1-কে `approved` ঘোষণা করতে পারেন।)*

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`notes/block-08-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Pre-Lab Code Review Exercise, 4-Stage Capstone Practical Laboratory, বা Optional Closed-Book Synthesis Assessment-এর কোনো solution এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* Blameless postmortem-এর অর্থ দায়িত্বহীনতা নয়—এর অর্থ হলো incident-এর পর কোনো একজন মানুষকে শাস্তি দেওয়ার বদলে systemic weaknesses (missing CI tests, large PRs, deadline pressure) ঠিক করা (**Distributed Organizational Accountability**)। কিন্তু সেটি কখনো একজন author-এর **Individual Responsibility** মুছে দেয় না: AI-generated code না বুঝে ও independently verify না করে merge করা ছিল একটি ব্যক্তিগত ownership failure (**Accountable Stewardship**-এর বিপরীত)। একজন accountable engineer স্বীকার করবেন যে তাঁর verification-এ ঘাটতি ছিল, *এবং* একই সাথে টিমের সাথে মিলে process/test guardrails শক্ত করবেন।
  2. *শক্তিশালী উত্তরে থাকবে:* **ACM 1.2 (Avoid harm):** Concurrent duplicate refund-এ গ্রাহক বা কোম্পানির সরাসরি আর্থিক ক্ষতি/data corruption-এর ঝুঁকি আছে—তাই এটি অবহেলা করা যাবে না। **ACM 1.3 (Be honest and trustworthy):** Concurrent cases test না করে client-কে *"সব verified"* বলা সরাসরি অসততা; ঠিক কী verify হয়েছে আর কী হয়নি তা সম্পূর্ণ সৎভাবে জানাতে হবে। **ACM 2.5 (Thorough evaluation including risks):** Manager-কে বস্তুনিষ্ঠ risk assessment দিতে হবে—যেমন (ক) আজ deploy করলে duplicate refund-এর concrete risk ও blast radius কী, (খ) কতটুকু বাড়তি সময় বা ন্যূনতম guard (যেমন DB uniqueness/idempotency check বা feature flag/limited rollout) লাগলে ঝুঁকি গ্রহণযোগ্য হবে, এবং (গ) business যদি তবুও ঝুঁকি নিয়ে এগোতে চায়, তবে সেই risk ও unverified limitation স্পষ্টভাবে নথিভুক্ত থাকবে—মিথ্যা "verified" দাবি করে নয়।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* Google-এর **The Standard of Code Review** অনুযায়ী আপনি PR-টি **Approve** করবেন (এবং `.reduce()` বা variable naming-এর পরামর্শ দিলে সেগুলো স্পষ্টভাবে non-blocking **`Nit:`** হিসেবে উল্লেখ করবেন)। কারণ PR-টি একটি বাস্তব rounding bug ঠিক করে এবং ভালো regression tests যোগ করে **overall code health স্পষ্টভাবে improve করছে**—perfection-এর নামে ব্যক্তিগত style preference চাপিয়ে team velocity আটকানো reviewer-এর কাজ নয়।
  2. *শক্তিশালী উত্তরে থাকবে:* অন্তত তিনটি blocking কারণ: (ক) **Small CLs লঙ্ঘন:** 550-line PR-এ unrelated formatting, refactoring, ও নতুন feature একসাথে মেশানো আছে, যা review fatigue তৈরি করে এবং rollback ঝুঁকিপূর্ণ করে—এটি ছোট, logically separate PR-এ ভাগ করতে হবে। (খ) **Weak CL Description:** *"Refactor checkout and add loyalty points"* কিছুই explain করে না—কেন পরিবর্তন দরকার, কী invariants ও trade-offs আছে, এবং নতুন `npm` package কেন যোগ করা হয়েছে তা missing। (গ) **Tests & Supply-Chain / Functionality Risk (`What to look for`):** 550 লাইন ও নতুন financial/loyalty logic-এর জন্য মাত্র ১টি unit test মারাত্মকভাবে অপ্রতুল, আর AI-suggested নতুন `npm` package-টি real, maintained, ও প্রয়োজনীয় কি না তা যাচাই করা হয়নি।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* সৎ phrasing হবে: *"Pure discount calculation এবং edge-case unit tests (rounding, expiry, negative totals) locally `node --test`-এ verified হয়েছে। **Unverified / Remaining risk:** multi-instance concurrency ও PostgreSQL transaction isolation এই PR-এ এখনো load-test করা হয়নি; staging DB integration ও concurrency test পরের ধাপে/before full rollout সম্পন্ন করতে হবে।"* এড়িয়ে চলতে হবে: *"All tests green, 100% production-ready"* বা unverified বিষয় গোপন করা।
  2. *শক্তিশালী উত্তরে থাকবে:* মূল mechanism হলো **closing the feedback loop / incentive alignment**। Developer নিজেই যখন নিজের service-এর production operation ও customer impact-এর মুখোমুখি হন (কোনো আলাদা Ops টিমের ঘাড়ে "throw it over the wall" করতে পারেন না), তখন সিস্টেম ভেঙে পড়ার খরচ সরাসরি builder-এর কাছেই ফিরে আসে। ফলে code লেখার সময়ই engineer-রা defensive error handling, স্পষ্ট diagnostic signals, safe defaults, এবং simple maintainable design বেছে নিতে স্বতঃস্ফূর্তভাবে অনুপ্রাণিত হন।
