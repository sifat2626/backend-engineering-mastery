# Phase 1, Block 6 — Simplicity, Complexity and Maintainability

## Study Time Guide

| Activity | Assigned Scope | Estimated Time |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(main priority)* | পুরো guided lesson মনোযোগ দিয়ে পড়া এবং `notes/block-06-notes.md`-এ Pause & Checks 1–3 লেখা | **~65–85 min** *(flexible — limit নয়)* |
| **Mandatory 1:** Rich Hickey — *Simple Made Easy* (2011) | InfoQ-এর পুরো primary talk দেখা অথবা transcript পড়া *(simple vs. easy, complecting, maintainability impact)* | ~60 min |
| **Mandatory 2:** John Ousterhout — *A Philosophy of Software Design* (2018) | YouTube talk-এর কেবল selected sections *(Complexity & 3 symptoms, Tactical vs. Strategic, Deep vs. Shallow Modules — full Q&A বাদ)* | ~35–40 min |
| **Mandatory 3:** Martin Fowler — *Technical Debt Quadrant* (2009) | পুরো short article পড়া *(2×2 matrix: Prudent/Reckless × Deliberate/Inadvertent)* | ~5–10 min |
| **Exercise 1 (Mandatory):** Overengineering Critique | 7-file pass-through Node.js CRUD feature-এর complexity symptoms, shallow modules ও essential vs. accidental complexity বিশ্লেষণ | ~30–35 min |
| **Total (Required Core — without Optional Activities)** | Guided Lesson + ৩টি Mandatory Resource + Mandatory Exercise 1 | **~195–230 min (~3.25–3.8 h)** |
| **Exercise 2 (Optional Extension):** Simple vs. Easy Classification | Small backend service-এ config/validation-এর দুটি approach তুলনা ও ৬ মাস পরের change behavior বিশ্লেষণ | ~10–15 min *(optional)* |
| **Optional Closed-Book Assessment** *(optional)* | Notes/tabs বন্ধ রেখে ৫টি synthesis প্রশ্নের উত্তর লেখা | ~25–35 min *(optional)* |
| **Total (with Optional Exercise 2 & Optional Assessment)** | Required Core + Optional Exercise 2 + Optional Assessment | **~230–280 min (~3.8–4.7 h)** |

> **Planning Note (Honest Scheduling Reference):** Approved curriculum ও dossier-এ Block 6-এর allocation হলো **2.5 hours (150 minutes)**—যেটা unchanged রাখা হয়েছে একটি flexible planning reference হিসেবে, কোনো hard deadline নয়। বাস্তবতা হলো, এই block-এর তিনটি mandatory resource একসাথে প্রায় `100–110 min` নেয় এবং mandatory Exercise 1 নেয় `30–35 min`। Guided lesson-কে artificially ১৫–২০ মিনিটে চেপে না দিয়ে এখানে honest ও realistic estimate দেখানো হয়েছে (`~3.25–3.8 h` required core; `~3.8–4.7 h` optional কাজসহ)। ভালো learning quality সবসময় artificial number মেলানোর চেয়ে বেশি গুরুত্বপূর্ণ—চাইলে Block 6-কে দুটি focused session-এ ভাগ করে পড়তে পারেন, অথবা Rich Hickey-র video-র বদলে InfoQ transcript পড়ে সময় কিছুটা বাঁচাতে পারেন। **Optional Exercise 2** ও **Optional Closed-Book Assessment** কোনোটিই পরের block-এ যাওয়ার জন্য mandatory নয়।

---

Block 5-এ আমাদের online store-এর `deploy #412` incident ও blameless postmortem-এর কথা মনে আছে? একটা ছোট্ট query filter change-এর কারণে repeat shopper-দের checkout `p95` `80 ms` থেকে `920 ms`-এ উঠে গিয়েছিল—অথচ pre-release review-তে কেউ আঁচই করতে পারেননি যে ওই change-টা index break করবে।

এবার ছয় মাস সামনে এগিয়ে যান। আমাদের সেই `LAUNCH50` promo-code backend-টা এখন অনেক বড় হয়েছে। Store-এ এখন percentage discount, flat discount, first-time-buyer code, per-user cap, regional tax rules, minimum order threshold, আর seasonal expiry—সবই চলছে।

সোমবার সকালে Marketing থেকে খুব সাধারণ, এক লাইনের একটা request এল:

> *"VIP customer-দের জন্য promo code apply করার সময় minimum order threshold check-টা skip করতে হবে।"*

নতুন join করা একজন backend engineer ভাবলেন, *"খুব তো ছোট একটা business rule—১৫ মিনিটের কাজ!"* তিনি repo খুললেন। কিন্তু তিন দিন পরও তাঁর PR merge করা যায়নি, আর staging-এ দুটো নতুন bug ধরা পড়েছে। কেন?

কোডটা খুলে দেখতেই তিনটি বাস্তবতা সামনে এল:

1. Promo eligibility-র rule এক জায়গায় নেই। Cart preview, checkout endpoint, mobile quote serializer, order validator, এবং refund worker—মোট **৫টা আলাদা file**-এ একই `minOrderAmount` check একটু একটু ভিন্নভাবে copy-paste করা আছে।
2. Checkout-এর মাঝখানে একটা shared `discountContext` object আছে, যেটাকে তিনটি আলাদা helper function-ই সরাসরি mutate করে—আর একটা function-এর নাম `getPromoDetails()`, কিন্তু ভেতরে ঢুকে দেখা গেল সেটা চুপিসারে database-এ `last_checked_at` update করে আর tax rate-ও override করে দেয়!
3. এর ওপর আবার, মাসখানেক আগে একজন developer codebase-কে "enterprise-ready" করার নামে একটা সাধারণ promo-expiry তারিখ চেক করার জন্য ৪টা আলাদা thin wrapper class (`ExpiryCheckProxy → PromoDateHelper → TimestampAdapter → RawDateReader`) বানিয়ে রেখেছেন। প্রতিটা file খুলে দেখা যায় ওগুলো শুধু call-টাকে পরের file-এ forward করছে—আসল logic কোথায় আছে খুঁজে বের করতেই মাথা ধরে যায়!

একটু থেমে দেখুন: **তিনটা problem-ই শেষ পর্যন্ত একটা জায়গায় এসে মিশেছে—Complexity।** একটা codebase যখন ধীরে ধীরে বোঝা ও change করার জন্য hard আর risky হয়ে ওঠে, তখন সবচেয়ে দক্ষ engineer-ও নিরাপদে একটা ১৫ মিনিটের feature add করতে পারেন না।

এই বাস্তব অভিজ্ঞতা থেকেই আমাদের Block 6-এর central question-টা উঠে আসে:

> **একটা backend codebase সময়ের সাথে কেন understand ও modify করা এত কঠিন হয়ে পড়ে—এর কোন complexity-টা আসলেই unavoidable, কোনটা আমরাই ভুল design choice দিয়ে তৈরি করি, আর কোনো design আসলেই simple ও maintainable কি না তা আমরা কীভাবে objectively বিচার করব?**

---

## সব Complexity কি দূর করা সম্ভব? Essential বনাম Accidental Complexity

আমাদের VIP promo-rule feature-টা আরেকবার খেয়াল করুন। এখানে যে কাজটা করতে বলা হয়েছে, তার সব কঠিনতাই কি খারাপ code-এর দোষ?

একদমই না। একটু ঠান্ডা মাথায় দেখলে দুটি সম্পূর্ণ আলাদা ধরনের কঠিনতা চোখে পড়বে:

- **প্রথম ধরন:** Real-world ব্যবসাটাই সত্যিকার অর্থে জটিল। একটি order-এ VIP customer-এর জন্য minimum threshold মাফ করা, কিন্তু non-VIP-র জন্য বহাল রাখা; tax-এর আগে না পরে discount হিসাব হবে তা regional আইন মেনে ঠিক করা; একই user যেন দুটি browser tab থেকে concurrent checkout করে single-use promo দুবার না নিতে পারে; full cancellation-এ promo ফেরত দেওয়া কিন্তু partial refund-এ না দেওয়া—এই rule-গুলো আমাদের বানানো নয়, এগুলো **real-world problem domain**-এর অংশ। আপনি Node.js ছেড়ে Go বা Rust-এ গেলেও, বা PostgreSQL ছেড়ে অন্য database নিলেও, এই business logic-এর কঠিনতা এক চুলও কমবে না।
- **দ্বিতীয় ধরন:** একই `minOrderAmount` check-টা ৫টা file-এ ছড়িয়ে থাকা; `getPromoDetails()` নাম দিয়ে ভেতরে চুপিসারে database write ও tax mutate করা; আর একটা সাধারণ expiry date check-এর ওপর ৪টা empty pass-through wrapper চাপিয়ে দেওয়া—এগুলোর কোনোটাই business problem-এর অংশ নয়! এগুলো আমরা নিজেরাই আমাদের tools, shortcuts, বা ভুল design decisions দিয়ে তৈরি করেছি।

১৯৮৭ সালে Turing Award বিজয়ী computer scientist **Fred Brooks** তাঁর বিখ্যাত প্রবন্ধ ***"No Silver Bullet — Essence and Accident in Software Engineering"*** (*IEEE Computer*, April 1987)-এ এই বিভাজনটাকেই software engineering-এর চিরন্তন ভিত্তি হিসেবে প্রতিষ্ঠা করেছেন *(খেয়াল রাখুন: এটি Brooks-এর একটি foundational conceptual framework, কোনো formal ISO standard নয়)*:

- **Essential Complexity (Essence):** Problem domain-এর ভেতরেই অন্তর্নিহিত conceptual difficulty—অর্থাৎ আসল business rules, data relationships, state transitions, ও edge cases-এর জটিলতা। যেকোনো correct solution-কেই এই essential complexity handle করতেই হবে; কোনো নতুন framework বা language একে জাদুর মতো মুছে ফেলতে পারে না।
- **Accidental Complexity (Accident):** যে difficulty-টা মূল problem-এর নিজস্ব অংশ নয়, বরং আমাদের বেছে নেওয়া tools, abstractions, unnecessary indirection, tangled code, বা poor design থেকে তৈরি হয়। Engineering skill-এর বড় অংশই হলো **unnecessary accidental complexity ঢুকতে না দেওয়া এবং বিদ্যমান accidental complexity কমানো।**

Brooks কেন বলেছিলেন যে software engineering-এ কোনো *"silver bullet"* (এমন কোনো একক প্রযুক্তি যা রাতারাতি productivity বা simplicity ১০ গুণ বাড়িয়ে দেবে) নেই? কারণ তাঁর বিশ্লেষণ অনুযায়ী software-এর **essence**-এর ভেতরেই চারটি intrinsic property লুকিয়ে আছে:

1. **Complexity:** Software-এর ভিন্ন ভিন্ন অংশের মধ্যে non-linear interaction থাকে; system যত বড় হয়, সম্ভাব্য state-এর সংখ্যা exponentially বাড়ে।
2. **Conformity:** Software-কে শূন্যে চলতে হয় না—একে মানুষের তৈরি পুরোনো আইন, tax rules, legacy database, আর external partner APIs-এর অদ্ভুত নিয়মগুলোর সাথে মানিয়ে (conform করে) চলতে হয়।
3. **Changeability:** সফল software কখনো স্থির থাকে না (Block 1-এর *Time & Change*)—ব্যবসা, user, আর platform বদলানোর সাথে সাথে তাকে ক্রমাগত বদলাতেই হয়।
4. **Invisibility:** একটি bridge বা building-এর মতো software-এর কোনো physical geometry নেই যেটা এক নজরে দেখে পুরো structure-টা visualize করা যায়।

এই একটি framework দিয়েই আধুনিক backend hype-গুলোর আসল চেহারা পরিষ্কার হয়ে যায়। High-level languages, managed PostgreSQL, type systems, কিংবা আজকের **AI coding tools**—এগুলো syntax লেখা, boilerplate তৈরি বা low-level setup-এর মতো **accidental** কাজ অনেক সহজ করে দিতে পারে। কিন্তু কোনো framework বা AI tool আপনার ব্যবসার **essential complexity** (কোন refund-এ promo ফেরত যাবে, কোন concurrent edge case-এ কী invariant থাকবে) মুছে দিতে পারে না। বরং careful design ছাড়া AI দিয়ে দ্রুত code generate করলে codebase-এ আরও দ্রুত **accidental complexity** জমে ওঠে! *(AI-augmented workflow আমরা Block 7-এ আলাদাভাবে পড়ব।)*

এখন একটা স্বাভাবিক প্রশ্ন আসে। কোনো engineer-ই তো সকালে অফিস খুলে পরিকল্পনা করে না যে *"আজ আমি codebase-এ অনেক accidental complexity ঢোকাব!"* **তাহলে বুদ্ধিমান, ভালো engineer-দের হাত দিয়েই প্রতিদিন এত accidental complexity তৈরি হয় কীভাবে?**

---

## "লিখতে Easy" বনাম "গঠনে Simple": Simple Made Easy

আমাদের online store-এর ওই `discountContext` bug-টা কীভাবে জন্ম নিয়েছিল, তা একটু rewind করে দেখুন।

ছয় মাস আগে একজন developer-কে দুপুরের মধ্যে একটা নতুন shipping-discount rule ship করতে বলা হয়েছিল। তাঁর হাতের কাছেই ছিল পুরোনো একটা `checkoutState` object এবং একটা 180-line `processCheckout()` function। তিনি ভাবলেন: *"আলাদা clean boundary বানাতে গেলে আরও এক ঘণ্টা লাগবে—এর চেয়ে `checkoutState`-এর ভেতরেই দুটো extra flag ঢুকিয়ে `processCheckout()`-এ একটা `if` বসাই, আর একটা পরিচিত ORM lifecycle hook দিয়ে save করার সময় auto-update করে দিই!"*

মাত্র ২০ মিনিটে feature-টা কাজ করতে শুরু করল। Developer খুশি, manager খুশি।

কিন্তু ৬ মাস পর কী হলো? এখন `processCheckout()`-এর ভেতরে HTTP input parsing, promo eligibility, tax calculation, shared mutable state, database write, আর email trigger—ছয়টা সম্পূর্ণ আলাদা concern একসাথে দড়ির বিনুনির মতো পেঁচিয়ে গেছে! আপনি এখন VIP threshold rule-টা বদলাতে গেলেই অজান্তে tax বা refund-এর ভেতর side effect তৈরি হচ্ছে।

ঠিক এই জায়গাতেই Clojure-এর স্রষ্টা **Rich Hickey** তাঁর বিখ্যাত ২০১১ সালের Strange Loop keynote ***"Simple Made Easy"***-তে আমাদের প্রতিদিনের ভাষার একটা ভয়ংকর গোলমাল ধরিয়ে দিয়েছেন *(আবারও মনে রাখবেন, এটি Hickey-র অত্যন্ত প্রভাবশালী conceptual lens—সর্বজনীন কোনো mathematical standard নয়)*। দৈনন্দিন কথায় আমরা `simple` আর `easy`-কে একার্থক ভাবি—অথচ এ দুটি সম্পূর্ণ ভিন্ন concept!

| Dimension | **Simple** (সরল গঠন) | **Easy** (হাতের নাগালে / পরিচিত) |
| :--- | :--- | :--- |
| **মূল উৎস ও অর্থ** | Latin *simplex* ("one fold / one braid") — একটা স্পষ্ট role বা concern, অন্য জিনিসের সাথে পেঁচানো নয় (**unbraided**)। | Latin roots meaning **"near at hand"** — যা আপনার বর্তমান familiarity, skill set, বা tooling-এর কাছাকাছি এবং শুরুতে দ্রুত ব্যবহার করা যায়। |
| **বৈশিষ্ট্য কেমন?** | **Objective structural property** — code-টার structure দেখেই বিচার করা যায় কয়টা concern একসাথে জড়ানো আছে। | **Relative / Subjective** — ব্যক্তিভেদে বদলায়। বাংলা পড়া আপনার কাছে *easy*, কিন্তু একজন German engineer-এর কাছে কঠিন; যে library আপনি ৩ বছর ধরে ব্যবহার করছেন সেটা আপনার জন্য *easy*, নতুন teammate-এর জন্য নয়। |
| **বিপরীত শব্দ** | **Complex** (*complected* — একসাথে পেঁচানো বা braided together)। | **Hard** (অপরিচিত, শুরুতে শিখতে বা setup করতে বেশি effort লাগে)। |
| **Backend উদাহরণ** | Pure pricing function যা শুধু cart ও promo values নেয় এবং calculated discount ফেরত দেয়—কোনো DB, HTTP বা global state স্পর্শ করে না। | Global mutable `checkoutState`, framework magic যা implicitly DB query চালায়, বা AI দিয়ে ৩০ সেকেন্ডে copy-paste করা 200-line tangled handler। |

Hickey একটা অসাধারণ verb জনপ্রিয় করেছেন: **Complecting** *(একসাথে বিনুনি করে পেঁচিয়ে ফেলা — to braid/interleave independent concerns together)*।

Backend engineering-এ আমরা কীভাবে অজান্তে জিনিসপত্র **complect** করে ফেলি, তার তিনটি খুব বাস্তব উদাহরণ দেখুন:
1. **Shared mutable state (value আর time-কে complect করা):** আমাদের `discountContext` object-টি যদি পাঁচটা function মিলে সরাসরি mutate করে, তাহলে কোনো মুহূর্তে `discountContext.total`-এর মান কী হবে তা নির্ভর করে ঠিক *কোন ক্রমে* (order/time) function-গুলো চলেছে তার ওপর! একটি pure value-কে সময় ও execution order-এর সাথে জড়িয়ে ফেলা হলো classic complecting।
2. **HTTP transport, business logic আর SQL একসাথে পেঁচানো:** একটি Express route handler-এর ভেতরই `req.body` থেকে field তোলা, promo eligibility হিসাব করা, SQL `UPDATE` চালানো, আর response format বানানো—শুরুতে এটা লিখতে ভীষণ *easy* (একটা file-এই সব আছে!)। কিন্তু পরের মাসে ওই একই promo logic যখন একটা background worker বা admin API থেকে call করতে হবে, তখন আপনি সেটা আলাদা করে reuse বা unit-test করতে পারবেন না।
3. **Quick AI-generated glue code:** কোনো prompt দিয়ে ৩০ সেকেন্ডে working code বের করে নেওয়া খুবই *easy*। কিন্তু সেই code যদি আপনার বিদ্যমান domain rules-এর সাথে duplicated logic, hidden side effects, আর implicit assumptions জড়িয়ে ফেলে, তবে Day 1-এর *ease* Day 90-এ মারাত্মক *complexity* হয়ে ফিরে আসে।

এখান থেকে দুটি প্রচলিত ভুল ধারণা সরাসরি ভেঙে যায়:
- **Short ≠ Simple:** একটা clever, dense 1-line nested ternary বা cryptic chaining দেখলে মনে হতে পারে *"বাহ, মাত্র এক লাইন—কত simple!"* আসলে সেটা *short*, *simple* নয়—কারণ এক লাইনের ভেতর চারটা condition ও side effect complect করা থাকলে তা বুঝতে ও modify করতে অনেক বেশি ঝুঁকি তৈরি হয়।
- **Easy ≠ Maintainable:** যে code আজ লিখতে *easy* (familiar shortcut, magic framework hook, quick copy-paste), তা প্রায়ই দীর্ঘমেয়াদে maintain করা সবচেয়ে *hard* হয়। যেকোনো design-এ আপনি যদি কেবল *easy* বেছে নেন, তবে শুরুতে speed পাবেন ঠিকই, কিন্তু দ্রুত একটা complexity wall-এ ধাক্কা খাবেন—যেখানে আপনার brain পুরো system-টা আর একসাথে ধরে রাখতে পারবে না।

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-06-notes.md` ফাইলে নিজের উত্তর সংক্ষেপে লিখুন।)*
> 1. আমাদের online store-এ একটি invoice generator feature-এর দুটি অংশ দেখুন: **(ক)** বাংলাদেশের VAT rules ও আন্তর্জাতিক customer-দের জন্য ভিন্ন tax exemption নিয়ম মেনে tax হিসাব করা; **(খ)** সেই tax calculation logic-টা `invoiceController.js` আর `emailWorker.js`-এ দুইভাবে copy-paste করে রাখা এবং global `process.env.DEFAULT_TAX`-এর ওপর নির্ভর করা। এর কোনটি **essential complexity** আর কোনটি **accidental complexity**—এবং কেন?
> 2. একজন সহকর্মী বললেন: *"এই ৩-লাইনের compact nested ternary আর ORM auto-hook ব্যবহার করাটাই সবচেয়ে simple, কারণ এতে কোড কম আর আমি ৫ মিনিটেই লিখে ফেলতে পারি।"* Rich Hickey-র **simple vs. easy** এবং **complecting** ধারণা দিয়ে এই যুক্তিটির দুর্বলতা ব্যাখ্যা করুন।

এখন একটা বাস্তব team-level সমস্যার দিকে তাকান। ধরুন, আপনি একটা PR review-তে বললেন *"এই implementation-টা খুব complex হয়ে গেছে"*—আর লেখক পাল্টা বললেন *"আরে না, আমার কাছে তো বেশ clean আর familiar লাগছে!"* **তখন ব্যক্তিগত taste বা opinion-এর ঝগড়া না করে, আমরা কীভাবে concrete, observable symptoms দেখে complexity-কে diagnose করব?**

---

## Complexity-কে চোখে দেখা: Three Symptoms এবং Two Root Causes

Stanford-এর professor ও দীর্ঘদিনের systems builder **John Ousterhout** তাঁর বই ***A Philosophy of Software Design*** (2018; 2nd ed. 2021) এবং Google-এ দেওয়া একই নামের talk-এ complexity-কে কোনো ধোঁয়াটে aesthetics হিসেবে নয়, বরং একটি অত্যন্ত practical, operational সংজ্ঞায় বেঁধেছেন *(এটি Ousterhout-এর engineering design framework—সব context-এ অন্ধ নিয়ম নয়, বরং অত্যন্ত কার্যকর diagnostic lens)*:

> *"Complexity is anything related to the structure of a software system that makes it hard to understand and modify the system."*

এই সংজ্ঞার দুটি সূক্ষ্ম দিক খেয়াল করুন:
- **Reader-এর চোখ দিয়ে বিচার:** যে engineer কোডটা এইমাত্র লিখেছেন, তাঁর মাথায় পুরো context তাজা থাকে—তাই তাঁর কাছে কোডটা simple মনে হতে পারে। কিন্তু অন্য একজন engineer যখন তিন মাস পর সেই code বুঝতে বা change করতে গিয়ে হোঁচট খান, তখনই আসল complexity ধরা পড়ে।
- **Complexity is incremental:** একটা ভালো codebase রাতারাতি ভেঙে পড়ে না। এটি নষ্ট হয় শত শত ছোট ছোট compromise ও shortcut জমতে জমতে—যার প্রতিটিকে আলাদাভাবে দেখে মনে হয় *"এটুকু আর কী এমন ক্ষতি করবে!"*

তাহলে কোনো design বা codebase-এ complexity বেড়েছে কি না, তা আমরা কোন **তিনটি observable symptoms** দেখে চিনব? আমাদের VIP promo-code story-র সাথে মিলিয়ে দেখুন:

### Symptom 1: Change Amplification
একটা দেখতে খুব ছোট, localized change করতে গিয়ে codebase-এর অনেকগুলো আলাদা file বা জায়গায় edit করতে হওয়াকে বলে **Change Amplification**।
- আমাদের store-এ `minOrderAmount` rule-টা VIP customer-দের জন্য বদলাতে গিয়ে `cartPreview.js`, `checkoutRoute.js`, `orderValidator.js`, `mobileQuote.js`, এবং `refundWorker.js`—৫ জায়গায় হাত দিতে হলো। একটা business decision, অথচ ৫টা edit! যত বেশি জায়গা বদলাতে হবে, কোনো একটা জায়গা ভুলে বাদ পড়ে যাওয়ার ঝুঁকি তত বেশি।

### Symptom 2: Cognitive Load
একটা নির্দিষ্ট task (bug fix বা ছোট feature) শেষ করার আগে একজন developer-কে মাথার ভেতর কতটা তথ্য, নিয়ম, convention, আর call chain একসাথে ধরে রাখতে হয়, তা হলো **Cognitive Load**।
- আমাদের store-এ একটা simple promo expiry check বুঝতে গিয়ে যদি আপনাকে ৬টা file খুলতে হয়, মনে রাখতে হয় কোন function-এ timestamp UTC আর কোনটায় local string, আর কোন helper-টা global `discountContext` mutate করে—তাহলে আপনার **cognitive load** ভয়াবহ বেশি।
- Ousterhout এখানে দারুণ একটা point করেন: **কম line code মানেই কম cognitive load নয়!** কখনো কখনো কয়েক লাইন বেশি, কিন্তু একদম সোজা ও self-explanatory code পড়তে অনেক কম মানসিক চাপ লাগে, আর clever shorthand বা ৫টা indirection layer-এর মধ্যে লাফালাফি করলে cognitive load আকাশে উঠে যায়।

### Symptom 3: Unknown Unknowns (সবচেয়ে বিপজ্জনক Symptom!)
যখন এটা স্পষ্টই বোঝা যায় না যে একটা change করতে হলে codebase-এর *কোন কোন জায়গা* modify করতে হবে, অথবা আপনার change-এর কারণে সিস্টেমের কোন দূরবর্তী অংশ চুপিসারে ভেঙে যাবে—তখন তাকে বলে **Unknown Unknowns**।
- Change amplification বিরক্তিকর, কিন্তু অন্তত আপনি জানেন কোন ৫টা file বদলাতে হবে। Cognitive load ক্লান্তিকর, কিন্তু সময় নিয়ে পড়ে বোঝা যায়। কিন্তু **unknown unknown** সরাসরি production incident ঘটায়! যেমন: আপনি `promo_codes.status`-এ `"VIP_ONLY"` নামে নতুন একটা মান যোগ করলেন, সব unit test পাস করল—অথচ কেউ জানত না যে দুই বছর আগে লেখা একটা nightly finance export script-এ ধরে নেওয়া আছে `status !== "ACTIVE"` মানেই `"EXPIRED"`, ফলে ওই script-টা production-এ নীরবে সব VIP order-এর হিসাব গড়বড় করে দিল!
- ভালো design-এর অন্যতম বড় লক্ষ্য হলো **obviousness**—অর্থাৎ একজন engineer code-টা দেখেই দ্রুত ও নির্ভরযোগ্যভাবে বুঝতে পারবেন কোথায় কী বদলাতে হবে এবং এর প্রভাব কোথায় পড়বে।

এবার পরের ধাপের কারণটা খুঁজুন: **এই তিনটি symptom তৈরি হয় কোন দুটি root cause থেকে?** Ousterhout দেখিয়েছেন, প্রায় সমস্ত software complexity-র মূলে থাকে মাত্র **দুটি root cause**:

| Root Cause | এটি আসলে কী | কোন Symptoms বাড়ায় | আমাদের Online Store উদাহরণ |
| :--- | :--- | :--- | :--- |
| **1. Dependencies** | যখন কোনো code-এর অংশ অন্য কোনো অংশের detail না জেনে একা বোঝা বা modify করা যায় না। | মূলত **Change Amplification** ও **Cognitive Load** | `checkoutRoute`, `mobileQuote`, এবং `refundWorker`—সবাই promo database column ও `minOrderAmount` হিসাবের internal detail-এর ওপর সরাসরি নির্ভরশীল। |
| **2. Obscurity** | যখন কোনো গুরুত্বপূর্ণ তথ্য code দেখে সহজে স্পষ্ট (obvious) হয় না—vague naming, inconsistent conventions, hidden side effects, বা missing context। | মূলত **Unknown Unknowns** ও **Cognitive Load** | `getPromoDetails()` নাম শুনে মনে হয় এটা read-only, অথচ ভেতরে চুপিসারে database write করে আর shared `discountContext`-এর tax field বদলে দেয়! |

খেয়াল রাখবেন: বাস্তব software-এ **dependencies পুরোপুরি zero করা যায় না**—কারণ বিভিন্ন component একসাথে কাজ করেই তো system বানায়! Engineering-এর লক্ষ্য হলো *unnecessary* dependencies বাদ দেওয়া, আর যে dependencies আসলেই প্রয়োজন, সেগুলোকে যতটা সম্ভব **simple, minimal ও obvious** রাখা।

এখান থেকে একটা গভীর প্রশ্ন উঠে আসে। আমরা symptoms চিনলাম, root causes চিনলাম। **তাহলে বাস্তব engineering team-গুলোতে প্রতিদিন এত unnecessary dependencies আর obscurity জমে ওঠে কীভাবে?**

---

## প্রতিদিন Complexity জমার কারখানা: Tactical বনাম Strategic Programming

Ousterhout-এর মতে, codebase ধীরে ধীরে অচল হয়ে যাওয়ার মূল কারণ মেধার অভাব নয়—বরং কোড লেখার সময় আমাদের daily **mindset**। তিনি দুটি বিপরীত mindset-এর তুলনা করেছেন:

### Tactical Programming ("কোনোমতে আজকের কাজটা চালিয়ে দাও")
**Tactical programming**-এ একমাত্র লক্ষ্য থাকে: *আজকের feature বা bug fix-টা যত দ্রুত সম্ভব working অবস্থায় শেষ করা।*
- একজন developer একটা নতুন VIP exception যোগ করতে গিয়ে ভাবলেন: *"Promo module-টা clean করে restructure করতে ২ ঘণ্টা লাগবে; তার চেয়ে `checkoutRoute.js`-এ একটা `if (user.isVip)` বসিয়ে দিই, আর `refundWorker.js`-এ আরেকটা quick hack দিয়ে দিই—২০ মিনিটে done! পরে সময় পেলে clean করব।"*
- সমস্যাটা হলো, **"পরে সময়" প্রায় কখনোই আসে না**—পরের সপ্তাহেই নতুন আরেকটা urgent ticket আসে! প্রতিটা tactical shortcut codebase-এ সামান্য করে dependency আর obscurity যোগ করে।
- Ousterhout এর চরম রূপকে মজার কিন্তু নির্ভুল একটা নাম দিয়েছেন: **Tactical Tornado**। প্রায় প্রতিটা বড় টিমেই এমন একজন prolific developer থাকেন যিনি অন্য সবার চেয়ে দ্বিগুণ দ্রুত feature ship করেন, কিন্তু প্রতিটা PR-এর পেছনে রেখে যান অস্পষ্ট logic, duplicate code, আর hidden edge-case bug-এর স্তূপ। Short-term-এ তিনি "star performer" মনে হলেও, বাস্তবে বাকি পুরো টিমকে মাসের পর মাস তাঁর তৈরি complexity পরিষ্কার করতে গিয়ে পিছিয়ে পড়তে হয়।

### Strategic Programming ("Working code যথেষ্ট নয়—Great design-ই আসল product")
**Strategic programming**-এ একটা feature কাজ করলেই কাজ শেষ হয় না। আপনার প্রধান লক্ষ্য হলো **দীর্ঘমেয়াদে এমন একটা clean structure তৈরি ও maintain করা, যাতে future changes সহজ ও নিরাপদ থাকে।**
- নতুন কোনো feature বা bug fix করার সময় আপনি শুধু *"এটা কীভাবে দ্রুত কাজ করাব?"* ভাবেন না—বরং ভাবেন *"এই change-টা আমাদের বিদ্যমান design-এ সবচেয়ে clean উপায়ে কোথায় বসবে? কোনো duplicate logic বা obscurity কি এখনই সরিয়ে নেওয়া যায়?"*
- Ousterhout-এর recommendation অনুযায়ী, পুরো **total development time**-এর প্রায় **10–20%** ধারাবাহিকভাবে continuous design improvements-এ invest করা উচিত (যেমন clean abstraction ভাবা, ভালো নাম দেওয়া, design alternatives তুলনা করা, ছোট incremental cleanup ও clear tests লেখা)। খেয়াল রাখবেন: এটি প্রতিটি individual task-এর ওপর চাপানো কোনো fixed per-task surcharge নয়, বরং পুরো engineering workflow-জুড়ে একটি ongoing investment allocation।

এই দুই mindset-এর ফলাফল সময়ের সাথে কীভাবে বদলায়, নিচের qualitative conceptual curve-টা দেখুন:

```text
Cumulative
Progress
   ▲                                         / Strategic Programming
   │                                       /   (শুরুতে ongoing design
   │                                     /      investment-এ কিছুটা ধীর,
   │                                   /        কিন্তু speed ধরে রাখে)
   │                             ────/────── Tactical Programming
   │                       ─────/────          (শুরুতে fast, কিন্তু
   │                  ────/───                 complexity জমে flat হয়ে যায়)
   │              ───/──
   │          ───/──
   │       ──/──
   │    ──/──
   │ ──/──
   └─────────────────────────────────────────────────────────▶ Time
                  ▲
           Crossover Point
     (Ousterhout-এর personal estimate: ~6–18 months;
      empirically measured universal rule নয়)
```

*(নোট: এটি একটি qualitative conceptual curve—কোনো empirically established industry result নয়। Ousterhout নিজেই স্পষ্টভাবে স্বীকার করেন যে কোনো universal crossover point প্রমাণ করার মতো empirical data নেই; বইয়ে তাঁর দেওয়া **6–18 months** estimate-টি কেবল তাঁর personal opinion, কোনো measured industry rule নয়। একদিনের throwaway script-এ এই crossover point কখনোই আসবে না—যা আমরা Block 1-এর proportional rigor-এ দেখেছি—কিন্তু long-lived backend service-এ continuous design investment-এর payoff সময়ের সাথে স্পষ্ট হয়ে ওঠে।)*

কিন্তু এখানেই একটা মারাত্মক ফাঁদ তৈরি হয়! অনেক engineer *Strategic Programming*-এর নাম শুনে, বা "clean architecture" বানানোর উৎসাহে, একেবারে উল্টো প্রান্তে চলে যান—তাঁরা ছোট্ট একটা feature-কে ডজনখানেক ছোট ছোট class, interface আর wrapper layer-এ ভেঙে ফেলেন। **তাহলে আমরা কীভাবে চিনব কোন abstraction-টা সত্যিই complexity কমাচ্ছে, আর কোনটা উল্টো শুধু extra layer যোগ করছে?**

---

## ভালো Abstraction বনাম অপ্রয়োজনীয় Layer: Deep বনাম Shallow Modules

একটা অত্যন্ত ক্ষতিকর industry myth হলো: *"বেশি abstraction layer আর ছোট ছোট class মানেই বেশি maintainable code!"*

Ousterhout এই myth-টিকে চমৎকার একটা জ্যামিতিক ছবি দিয়ে ভেঙে দিয়েছেন। যেকোনো **module**—হোক সেটা একটা function, class, file, package, বা পুরো একটা service—তার দুটি অংশ থাকে:
1. **Interface (চওড়া প্রান্ত / Cost):** Module-টা ব্যবহার করতে caller-কে যা জানতে হয়—function signature, parameters, return types, exceptions, এবং এর আচরণগত contract বা side effects। **Interface হলো সেই complexity-র অংশ যা caller-এর ঘাড়ে চাপে।**
2. **Implementation (গভীরতা / Benefit):** ওই interface-এর পেছনে module-টা আসলে যতটুকু কঠিন কাজ ও detail লুকিয়ে (hide করে) সম্পন্ন করে।

ভালো module-এর সূত্রটা একেবারে সহজ: **ছোট ও simple interface-এর পেছনে বড় ও meaningful functionality লুকিয়ে রাখা (Information Hiding)।**

```text
     Deep Module                              Shallow Module
  ┌───────────────┐                ┌──────────────────────────────────┐
  │ Small, Simple │  ◀── Interface │  Large / Boilerplate Interface   │
  │   Interface   │                ├──────────────────────────────────┤
  ├───────────────┤                │ Thin Pass-Through Implementation │
  │               │                └──────────────────────────────────┘
  │  Substantial  │                   (প্রায় কিছুই hide করে না, কিন্তু
  │    Hidden     │                    caller-কে পুরো interface বুঝতে হয়!)
  │ Functionality │
  │  & Edge-Case  │
  │   Handling    │
  └───────────────┘
```

দুটোর বাস্তব backend উদাহরণ পাশাপাশি দেখুন:

- **Deep Modules (Complexity সত্যিই hide করে):**
  - **PostgreSQL SQL & Transactions:** একজন backend developer মাত্র কয়েক লাইনের SQL query আর `BEGIN ... COMMIT` interface ব্যবহার করেন। এর পেছনে লুকানো থাকে query optimizer, index traversal, disk page buffering, crash recovery, আর concurrent transaction isolation (বিস্তারিত Phase 10-এ)। Tiny interface, massive hidden complexity!
  - **Unix/POSIX File I/O (`open`, `read`, `write`, `close`):** মাত্র কয়েকটা সাধারণ operation-এর পেছনে disk blocks, OS page cache, permissions, আর filesystem drivers লুকানো থাকে (Phase 3-এ দেখবেন)।
  - **আমাদের Store-এ একটি Deep Promo Module:** ধরুন `evaluatePromoCode({ cart, customer, code, nowUtc })` নামে একটি coherent module বানালাম। Caller শুধু এই একটি clear interface জানে এবং একটি structured result (`{ eligible, discountAmount, errorCode }`) পায়। এর ভেতরে লুকানো থাকে expiry boundary check, VIP minimum-order exemption, flat vs. percentage clamping (`>= 0`), এবং consistent error mapping। এখন যদি VIP rule বদলায়, আপনার কেবল এই একটি module-এর ভেতরে change করলেই চলে!
- **Shallow Modules (Complexity না কমিয়ে উল্টো বাড়ায়):**
  - ধরুন একজন developer একটা 1-line date comparison (`now > expiresAt`) wrap করে বানালেন `ExpiryCheckProxy` → `PromoDateHelper` → `TimestampAdapter` → `RawDateReader`, যেখানে প্রতিটা class শুধু পরের class-এর একই নামের method-কে call forward করছে (`return this.helper.isExpired(code, now)`), কোনো meaningful validation, timezone normalization, বা invariant enforce করছে না!
  - Ousterhout এই প্রবণতাকে বলেন **Classitis** — অনেক ছোট ছোট class বা layer বানালেই design ভালো হবে, এই ভ্রান্ত ধারণা থেকে সিস্টেমকে ডজনখানেক shallow wrapper-এ টুকরো টুকরো করে ফেলা। প্রতিটা shallow module নিজে কোনো complexity hide করে না, কিন্তু একটা নতুন interface, extra file, extra type mapping, এবং extra navigation যোগ করে—ফলে **cognitive load** আর **change amplification** দুটোই বেড়ে যায়!

**গুরুত্বপূর্ণ সূক্ষ্মতা (Ousterhout-কে oversimplify করবেন না):**
"Deep module ভালো" মানে কখনোই এটা নয় যে পুরো backend-কে একটা 4,000-line giant file-এ গুঁজে দেবেন! একটা function বা module-কে তখনই আলাদা করুন যখন সেই split-টা আসলেই **meaningful complexity hide করে**, unrelated concerns-কে আলাদা (unbraid) রাখে, এবং caller-এর cognitive load কমায়। আর যদি দেখেন দুটো module-কে বুঝতে গেলে সবসময় দুটোকেই পাশাপাশি খুলে পড়তে হচ্ছে (tightly coupled pass-through layers), তবে সেটা ভালো modularity নয়—সেটা shallow fragmentation। *(OOP, SOLID ও modular design patterns-এর গভীর চর্চা আমরা Phase 12-এ করব।)*

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-06-notes.md` ফাইলে নিজের উত্তর সংক্ষেপে লিখুন।)*
> 1. আমাদের store-এ একজন developer `shipping_fee` হিসাব করার জন্য নতুন একটা helper লিখলেন: নাম দিলেন `formatOrderSummary(order)`, কিন্তু ভেতরে ওটা `order.total`-কে সরাসরি mutate করে shipping fee যোগ করে দেয় এবং database-এ একটা log row লিখে ফেলে। কয়েক সপ্তাহ পর আরেকজন developer invoice PDF বানাতে গিয়ে `formatOrderSummary(order)` call করলেন, আর production-এ customer-এর invoice-এ দুবার shipping fee যোগ হয়ে গেল! Ousterhout-এর **three symptoms** এবং **two root causes** দিয়ে এই bug-টাকে diagnose করুন।
> 2. কেন একটা 50-line self-contained function কখনো কখনো ৫টা আলাদা 10-line pass-through helper class-এর চেয়ে বেশি **simple** ও **deep** হতে পারে?

এখন পুরো ছবিটা প্রায় সম্পূর্ণ। একপাশে আমরা দেখলাম **Tactical Programming**-এর কারণে জমে ওঠা tangled mess; অন্যপাশে দেখলাম **Shallow Modules**-এর নামে বানানো অপ্রয়োজনীয় overengineering। **তাহলে বাস্তব engineering আলোচনায় আমরা maintainability, overengineering, এবং প্রায় প্রতিদিন শোনা শব্দ "technical debt"-কে কীভাবে পরিষ্কার, সৎ মানদণ্ডে বিচার করব?**

---

## Maintainability, Overengineering এবং Martin Fowler-এর Technical Debt Quadrant

প্রথমে **Maintainability** কথাটাকে এতক্ষণের সব ধারণার সাথে যুক্ত করে দেখুন। Block 1-এ আমরা *software sustainability* দেখেছিলাম, আর Block 3-এ **ISO/IEC 25010:2023**-এ *Maintainability* দেখেছিলাম (*modularity, analysability, modifiability, testability*)। এখন আপনি দেখতে পাচ্ছেন **maintainability আসলে কী দিয়ে তৈরি হয়**:
- Essential complexity-কে স্পষ্টভাবে handle করা,
- Unnecessary accidental complexity ও complecting এড়ানো,
- Change amplification, cognitive load, ও unknown unknowns কম রাখা, এবং
- Deep, obvious modules তৈরি করা।

কিন্তু maintainability-র দুটি বিপরীত শত্রু আছে—একপাশে **Overengineering**, অন্যপাশে **Reckless Mess**।

### শত্রু ১: Overengineering এবং Speculative Generality
অনেক developer ভবিষ্যতের ভয়ে আগেভাগেই বিশাল, জটিল কাঠামো বানিয়ে ফেলেন:
- *"আজ আমাদের ৩টা promo rule আছে, কিন্তু আগামী বছর যদি ৫০০টা dynamic rule লাগে—তাই এখনই একটা custom rule-engine DSL বানাই!"* (Block 1-এ দেখেছেন!)
- *"আজ আমাদের শুধু PostgreSQL আছে, কিন্তু ভবিষ্যতে যদি MongoDB বা DynamoDB-তে move করতে হয়—তাই এখনই প্রতিটা query-র ওপর তিনটা abstract repository interface আর generic mapper layer বসাই!"*
- *"আজ ৫০০ daily user, কিন্তু ভবিষ্যতে যদি ১০ মিলিয়ন হয়—তাই আজই ৪টা microservice আর Kafka বসাই!"* (Block 4-এ দেখেছেন!)

একেই বলে **Speculative Generality** বা **Overengineering** — *hypothetical future requirements*-এর জন্য আজই complex abstraction বা infrastructure বানানো। এর বাস্তব failure mode-টা খুব নির্মম:
1. ভবিষ্যতের যে requirement কল্পনা করে abstraction বানানো হয়, তার বেশিরভাগই **কখনোই আসে না**—আর এলেও দেখা যায় আসল requirement-টা আপনার অনুমানের চেয়ে সম্পূর্ণ আলাদা, ফলে পুরোনো abstraction-টা কাজে তো লাগেই না, উল্টো বাধা হয়ে দাঁড়ায়!
2. অথচ ওই extra layers ও abstractions-এর **accidental complexity, cognitive load, এবং change amplification-এর cost আপনাকে আজ থেকেই প্রতিটা PR-এ দিতে হয়** (Block 1-এর *Opportunity Cost* এবং Block 4-এর *First Law of Software Architecture*)।

ভালো engineering-এর নীতি হলো: **যে complexity আজ প্রমাণিতভাবে বিদ্যমান, শুধু সেটাকেই clean, deep modules দিয়ে handle করুন; hypothetical ভবিষ্যতের জন্য নয়।** *(Evidence-based decision — Block 5!)*

### শত্রু ২: "Technical Debt" শব্দের অপব্যবহার — Martin Fowler-এর Technical Debt Quadrant
এবার অন্য প্রান্তটা দেখুন। বাস্তব টিমে প্রায়ই শোনা যায়: *"কোডটা একটু messy হয়ে গেছে, test লেখা হয়নি, পরে দেখব—ওটা technical debt!"*

এই অজুহাতটা বন্ধ করার জন্যই **Martin Fowler** (Oct 14, 2009 — Ward Cunningham-এর মূল debt metaphor-এর ওপর ভিত্তি করে) তাঁর বিখ্যাত **Technical Debt Quadrant** তৈরি করেন *(আবারও note: এটি একটি conceptual communication framework, কোনো formal standard নয়)*।

Fowler দেখিয়েছেন, **সব খারাপ code-ই "technical debt" নয়**, এবং সব technical debt-ই সমান ক্ষতিকর নয়। যেকোনো design compromise-কে দুটি axis-এ বিচার করতে হয়:
- **Reckless বনাম Prudent:** সিদ্ধান্তটা কি পরিণতি, cost ও repayment ভেবে বিচক্ষণভাবে নেওয়া হয়েছে, নাকি অসতর্কতা/অজ্ঞতা থেকে?
- **Deliberate বনাম Inadvertent:** টিম কি জেনেশুনে সচেতনভাবে shortcut নিয়েছে, নাকি অজান্তেই তা তৈরি হয়ে গেছে?

| | **Reckless** *(বেপরোয়া / অপরিণামদর্শী)* | **Prudent** *(বিচক্ষণ / সচেতন)* |
| :--- | :--- | :--- |
| **Deliberate** *(ইচ্ছাকৃত)* | ***"We don't have time for design."***<br>কোনো strategic justification বা repayment plan ছাড়াই impatience বা laziness থেকে design ও verification বাদ দেওয়া। এটি *Tactical Tornado*-র তৈরি করা **reckless mess**—এতে দ্রুত heavy interest (bugs, slow delivery) জমে ওঠে। | ***"We must ship now and deal with consequences."***<br>একটি বাস্তব, high-value business deadline ধরার জন্য team সচেতনভাবে একটা bounded shortcut নেয়, এর risk ও cost স্পষ্টভাবে document করে (যেমন Block 4-এর ADR-এ), এবং শোধ করার plan রাখে। এটাই financial debt-এর মতো প্রকৃত calculated trade-off। |
| **Inadvertent** *(অনিচ্ছাকৃত)* | ***"What's layering?"***<br>মৌলিক software design principles না জানার কারণে অজান্তেই tangled, coupled, unmaintainable code তৈরি করা। সবচেয়ে বিপজ্জনক দিক হলো—টিম নিজেই বুঝতে পারে না যে debt জমছে! | ***"Now we know how we should have done it."***<br>টিম শুরুতে তাদের সেরা জ্ঞান ও care দিয়ে ভালো design-ই করেছিল; কিন্তু ৬ মাস system চালানোর পর domain-টা গভীরভাবে বুঝে টের পেল আসল boundary বা model-টা ভিন্ন হওয়া উচিত ছিল। Complex software-এ এই ধরনের debt অনেকটাই unavoidable—ভালো দলগুলো এটি চিনে ধীরে ধীরে refactor করে শোধ করে। |

আমাদের online store দিয়ে চারটা quadrant-এর পার্থক্য এক নজরে মিলিয়ে নিন:
- **Prudent & Deliberate:** ৫ সপ্তাহ পরের flash sale ধরার জন্য marketing analytics-এ real-time pipeline না বানিয়ে 2-minute summary query ship করা, ADR-এ এর limitation ও revisit trigger লিখে রাখা।
- **Reckless & Deliberate:** কোনো urgent deadline না থাকা সত্ত্বেও *"test আর validation লিখতে সময় লাগে, বাদ দাও"* বলে `minOrderAmount` logic ৫ জায়গায় copy-paste করে deploy করে দেওয়া।
- **Prudent & Inadvertent:** শুরুতে single-country BDT pricing ধরে clean promo module বানানো হয়েছিল; এক বছর পর ৩টা দেশে multi-currency launch করতে গিয়ে বোঝা গেল currency ও tax rules-কে আলাদা module-এ ভাগ করলে অনেক বেশি clean হতো।
- **Reckless & Inadvertent:** একজন developer separation of concerns বা SQL parameterization না জেনেই সব route handler-এ raw SQL, global mutable state, আর business logic একসাথে মিশিয়ে দিলেন।

এই quadrant-টা একটা অত্যন্ত জরুরি engineering distinction পরিষ্কার করে: **Prudent, deliberate technical debt** একটা বৈধ, সচেতন engineering trade-off হতে পারে—কিন্তু কেবল তখনই যখন আপনি এর **interest** (ভবিষ্যতের বাড়তি cost) জানেন এবং তা শোধ করার দায়িত্ব নেন। অন্যদিকে **reckless mess**-কে "technical debt" নাম দিয়ে ঢেকে রাখা হলো engineering discipline-এর ব্যর্থতা।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-06-notes.md` ফাইলে নিজের উত্তর সংক্ষেপে লিখুন।)*
> 1. নিচের তিনটি বাস্তব ঘটনাকে Martin Fowler-এর **Technical Debt Quadrant**-এর সঠিক ঘরে ফেলুন এবং ১–২ বাক্যে কারণ ব্যাখ্যা করুন:
>    - **(ক)** টিম ৬ মাস ধরে clean design মেনেই subscription billing চালিয়েছে; নতুন annual-proration নিয়ম আসার পর তারা বুঝল invoice আর subscription state আলাদা করলে design অনেক simpler হতো।
>    - **(খ)** একজন developer বললেন *"বাইরে থেকে তো API কাজ করছেই—unit test আর clean boundary বানানোর সময় নেই, এখনই merge করে দাও"*—যদিও কোনো hard launch deadline ছিল না।
>    - **(গ)** আগামী সপ্তাহে বড় একটি partner demo ধরার জন্য টিম CSV export-এ synchronous generation রাখল (সর্বোচ্চ ৫০০ row limit দিয়ে), limitation-টা ADR-এ লিখল এবং পরের sprint-এ async job-এ নেওয়ার ticket তৈরি করল।
> 2. একজন engineer প্রস্তাব দিলেন: *"আমাদের মাত্র ১টা payment gateway (Stripe) আছে, কিন্তু ভবিষ্যতে হয়তো আরও ৫টা gateway লাগবে—তাই আজ থেকেই আমরা একটা generic Multi-Gateway Orchestration Framework, Plugin Registry, আর ৪টা abstract adapter layer বানিয়ে রাখি।"* এখানে **Speculative Generality / Overengineering**-এর বাস্তব cost আজ কী হবে, এবং এর বদলে একটা prudent, simpler approach কী হতে পারে?

---

## সব সূত্র একসাথে বাঁধা — এবং Block 7-এর দিকে যাত্রা

পেছনে তাকিয়ে দেখুন, এখন কোনো backend design, PR, বা architecture proposal দেখলে আপনার হাতে আর শুধু *"আমার কাছে clean লাগছে"* বা *"খুব messy লাগছে"*—এই ধরনের subjective মতামত নেই। আপনার হাতে আছে একটা পরিষ্কার, পাঁচ-ধাপের **Simplicity & Maintainability Engineering Filter**:

1. **Essential বনাম Accidental (Brooks):** এই complexity-টা কি আসল business domain থেকে আসছে (unavoidable), নাকি আমাদের নিজের tool, abstraction, বা messy code থেকে তৈরি হয়েছে (reducible)?
2. **Simple বনাম Easy (Hickey):** এই solution-টা কি independent concerns-কে আলাদা (**unbraided**) রেখেছে, নাকি শুরুতে quick ও familiar হওয়ার লোভে state, business logic, ও I/O একসাথে **complect** করে ফেলেছে?
3. **Symptoms & Root Causes (Ousterhout):** একটা ছোট change করলে কি অনেক file বদলাতে হচ্ছে (**change amplification**)? বুঝতে কি অতিরিক্ত context মাথায় রাখতে হচ্ছে (**cognitive load**)? কোনো hidden coupling বা side effect কি **unknown unknowns** তৈরি করছে—আর এর পেছনে কোন **dependencies** বা **obscurity** কাজ করছে?
4. **Strategic & Deep vs. Tactical & Shallow (Ousterhout):** আমরা কি continuous strategic design investment-এর মাধ্যমে এমন **deep modules** বানাচ্ছি যা ছোট interface-এর পেছনে আসল complexity hide করে—নাকি **tactical shortcuts** জমাচ্ছি, অথবা উল্টোদিকে **shallow wrappers / speculative generality** দিয়ে overengineering করছি?
5. **Honest Debt Classification (Fowler):** এখানে কোনো shortcut নেওয়া হলে তা কি সত্যিই **prudent & deliberate** (documented, bounded, with a repayment plan), নাকি **reckless mess**?

এখন এই পুরো উপলব্ধিটাকে নিয়ে সরাসরি সামনে তাকান—আধুনিক software engineering-এর সবচেয়ে বড় পরিবর্তনের দিকে।

Fred Brooks ১৯৮৭ সালেই সতর্ক করেছিলেন যে কোনো প্রযুক্তিগত *silver bullet* software-এর **essential complexity** মুছে দিতে পারে না। আজকের **AI coding assistants** কোড তৈরি করাকে ইতিহাসের যেকোনো সময়ের চেয়ে বেশি **easy** করে দিয়েছে—কয়েক সেকেন্ডেই শ’খানেক লাইন syntactically valid Node.js ও SQL code আপনার সামনে চলে আসে! কিন্তু ঠিক এই কারণেই ঝুঁকিও বেড়েছে: AI অনায়াসে এমন code generate করতে পারে যা দেখতে পরিচিত ও দ্রুত, অথচ ভেতরে ভেতরে **complected**, **shallow**, subtle edge-case bug-এ ভরা, বা অপ্রয়োজনীয় **accidental complexity**-তে বোঝাই—এক ধরনের automated *Tactical Tornado*!

তাহলে আধুনিক backend engineer হিসেবে আমরা কীভাবে AI-কে অন্ধভাবে trust না করে (এবং পুরোপুরি বাদও না দিয়ে) একজন disciplined engineering partner হিসেবে ব্যবহার করব—কোন কাজ AI-কে delegate করব, কোনটা নিজে own করব, কীভাবে AI-কে tight context দেব, আর কীভাবে empirically AI output verify করব?

ঠিক এই প্রশ্নগুলো নিয়েই শুরু হবে **Block 7: AI-Augmented Engineering**।

---

## Study Plan, Learning Objectives & Mandatory Reading

> **Continuity & Ownership Notes:** আপনার সব inline check, practical exercise এবং (যদি করতে চান) optional closed-book assessment-এর উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-06-notes.md` ফাইলে লিখুন। AI-এর প্রস্তুতির সময় আপনার study hours-এ গণনা হয় না।
>
> **Progression & Verification Policy:** Formal assessment সম্পূর্ণ **optional** এবং পরের block-এ এগোনোর নিয়ন্ত্রণ আপনার হাতে। আপনি চাইলে assessment সম্পন্ন করে formal evaluation-এর জন্য ChatGPT-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে), নিজে যথেষ্ট শিখেছেন মনে করলে assessment skip করে এগিয়ে যেতে পারেন (`Studied — mastery not formally verified`), অথবা assessment ছাড়াই যেকোনো unclear concept নিয়ে প্রশ্ন ও আলোচনা করতে পারেন। কোনো গুরুত্বপূর্ণ gap থাকলে ChatGPT বাড়তি verification-এর পরামর্শ দিতে পারেন, তবে assessment সম্পন্ন করা বা ChatGPT-র approval পরের block শুরু করার mandatory gate নয়। Assessment skip করলে তা automatically `Mastery verified` বলে গণ্য হয় না।

### Study Time Reference

*প্রতিটা activity-র বাস্তবসম্মত time breakdown দেখতে এই নথির শুরুর **Study Time Guide** দেখুন। মূল roadmap-এ Block 6-এর `2–3 hours` (dossier-এ `2.5 hours`) allocation একটি planning reference হিসেবে অপরিবর্তিত আছে; কোনো activity-রই hard deadline নেই।*

### Block 6 Learning Objectives *(Supports Mastery Outcomes: 4, 6, 8)*

- **LO 6.1 (`Working Understanding`):** **Essential complexity** (real-world problem domain-এর অন্তর্নিহিত জটিলতা) এবং **accidental complexity** (আমাদের tools, abstractions ও design choices থেকে তৈরি জটিলতা — Brooks 1987)-এর মধ্যে স্পষ্ট পার্থক্য নির্ণয় করা।
- **LO 6.2 (`Working Understanding`):** **Simple** (unbraided, কম interleaved concerns — একটি objective structural property) এবং **easy** (familiar, শুরুতে দ্রুত ব্যবহারযোগ্য — developer-এর background সাপেক্ষ relative property — Hickey 2011)-এর পার্থক্য ব্যাখ্যা করা।
- **LO 6.3 (`Working Understanding`):** Complexity-র তিনটি observable symptom (*change amplification*, *cognitive load*, *unknown unknowns*) এবং দুটি root cause (*dependencies* ও *obscurity*) শনাক্ত করা, এবং **tactical vs. strategic programming** ও **deep vs. shallow modules** (Ousterhout) তুলনা করা।
- **LO 6.4 (`Working Understanding`):** **Maintainability**, **overengineering**, এবং **technical debt** নিয়ে সঠিকভাবে reasoning করা — Martin Fowler-এর **Technical Debt Quadrant** (prudent/reckless × deliberate/inadvertent) ব্যবহার করে।

### Complementary Authoritative Reading & Viewing

- **Mandatory 1 — Primary Talk (~60 min):** Rich Hickey, *"Simple Made Easy"* (Strange Loop Keynote, 2011) — [InfoQ Video & Transcript](https://www.infoq.com/presentations/Simple-Made-Easy/) *(link verified accessible; video এবং full transcript দুটোই পাওয়া যায়)*।
  - **কী দেখবেন/পড়বেন:** পুরো talk-টি দেখুন অথবা transcript পড়ুন — মূল focus দিন *simple vs. easy*-র সংজ্ঞা, *complecting* (concerns পেঁচিয়ে ফেলা), এবং কেন *easy*-কে *simple*-এর ওপর প্রাধান্য দিলে maintainability নষ্ট হয়।
  - **কোন Objective-কে সমর্থন করে:** `LO 6.2`।
- **Mandatory 2 — Selected Sections Only (~35–40 min):** John Ousterhout, *"A Philosophy of Software Design"* (*Talks at Google*, 2018) — [YouTube Video](https://www.youtube.com/watch?v=bmSAYlu0NcY)।
  - **কী দেখবেন (কেবল selected sections — Block 6-এ full Q&A দেখার দরকার নেই):**
    1. Complexity & its 3 symptoms *(approximate: ~00:00–12:00)*
    2. Tactical vs. Strategic Programming *(approximate: ~12:00–22:00)*
    3. Deep vs. Shallow Modules *(approximate: ~22:00–38:00)*
  - **গুরুত্বপূর্ণ Timestamp নোট:** এই YouTube video-তে official chapter markers নেই, তাই ওপরের timestamp-গুলো **approximate / unverified**। মূল video-তে সময় কিছুটা এদিক-ওদিক হলে ঘড়ির বদলে speaker-এর **topic transitions** অনুসরণ করুন এবং Q&A শুরু হওয়ার আগে থেমে যান।
  - **কোন Objective-কে সমর্থন করে:** `LO 6.3`, `LO 6.4`।
- **Mandatory 3 — Short Article (~5–10 min):** Martin Fowler, *"Technical Debt Quadrant"* (Oct 14, 2009) — [martinfowler.com/bliki/TechnicalDebtQuadrant.html](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html)।
  - **কী পড়বেন:** পুরো সংক্ষিপ্ত প্রবন্ধটি পড়ুন (2×2 matrix: Reckless/Prudent × Deliberate/Inadvertent)।
  - **কোন Objective-কে সমর্থন করে:** `LO 6.4`।
- **Optional Supplementary (মূল study budget-এর বাইরে — mandatory নয়):**
  - John Ousterhout, *A Philosophy of Software Design* (2nd ed., Yaknyam Press, 2021 — commercial book): **Ch. 2: The Nature of Complexity** (2.1–2.5: change amplification, cognitive load, unknown unknowns; dependencies & obscurity), **Ch. 3: Working Code Isn't Enough** (3.1–3.5: tactical vs. strategic programming), এবং **Ch. 4: Modules Should Be Deep** (4.1–4.8: deep vs. shallow modules, classitis)। ওপরের বিনামূল্যের talk-টির গভীরতর বই সংস্করণ।
  - Fred Brooks, *"No Silver Bullet — Essence and Accident in Software Engineering"*, *IEEE Computer*, Vol. 20, No. 4, pp. 10–19 (April 1987) — [DOI 10.1109/MC.1987.1663532](https://doi.org/10.1109/MC.1987.1663532) *(IEEE paywall-এর কারণে supplementary হিসেবে রাখা হয়েছে; এর মূল conceptual distinctions ওপরের guided lesson-এ সরাসরি শেখানো হয়েছে)*।

---

## Practical Reasoning Exercises

> **Ownership Rule:** সব উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-06-notes.md` ফাইলে লিখুন। AI-কে দিয়ে উত্তর draft করাবেন না। লক্ষ্য হলো ওপরের শেখা framework-গুলো নতুন বাস্তব backend code ও design-এ স্বাধীনভাবে apply করা।

### Exercise 1: Overengineering Critique *(Mandatory — ~30–35 min)*

**Scenario:**
একটি ছোট Node.js + PostgreSQL backend service-এ একটি খুবই সাধারণ 3-column table আছে:

```sql
CREATE TABLE promo_tags (
  id         SERIAL PRIMARY KEY,
  code       VARCHAR(32) UNIQUE NOT NULL,
  label      VARCHAR(80) NOT NULL
);
```

এটি শুধুমাত্র marketing team-এর internal tagging-এর জন্য tag তৈরি ও read করার একটা simple CRUD feature। কিন্তু একজন developer "future-proof enterprise architecture" বানানোর নামে এই একটিমাত্র feature-কে **৭টি আলাদা pass-through file**-এ ভাগ করেছেন:

`Controller → Facade → ServiceInterface → ServiceImpl → RepositoryInterface → RepositoryImpl → Mapper`

নিচে `POST /api/promo-tags` (নতুন tag তৈরি করার) flow-টার সংক্ষিপ্ত বাস্তব কাঠামো দেখুন:

```javascript
// 1. promoTag.controller.js
export class PromoTagController {
  constructor(facade) { this.facade = facade; }
  async create(req, res) {
    const result = await this.facade.createTag(req.body.code, req.body.label);
    return res.status(201).json(result);
  }
}

// 2. promoTag.facade.js
export class PromoTagFacade {
  constructor(service) { this.service = service; }
  async createTag(code, label) {
    return this.service.create(code, label);
  }
}

// 3. promoTag.service.interface.js
export class IPromoTagService {
  async create(code, label) { throw new Error("Not implemented"); }
}

// 4. promoTag.service.impl.js
export class PromoTagServiceImpl extends IPromoTagService {
  constructor(repo, mapper) { super(); this.repo = repo; this.mapper = mapper; }
  async create(code, label) {
    const entity = this.mapper.toEntity({ code, label });
    const saved = await this.repo.insert(entity);
    return this.mapper.toDto(saved);
  }
}

// 5. promoTag.repository.interface.js
export class IPromoTagRepository {
  async insert(entity) { throw new Error("Not implemented"); }
}

// 6. promoTag.repository.impl.js
export class PromoTagRepositoryImpl extends IPromoTagRepository {
  constructor(db) { super(); this.db = db; }
  async insert(entity) {
    const { rows } = await this.db.query(
      "INSERT INTO promo_tags (code, label) VALUES ($1, $2) RETURNING id, code, label",
      [entity.code, entity.label]
    );
    return rows[0];
  }
}

// 7. promoTag.mapper.js
export class PromoTagMapper {
  toEntity(input) { return { code: input.code, label: input.label }; }
  toDto(row) { return { id: row.id, code: row.code, label: row.label }; }
}
```

**Tasks:**
`phases/phase-01/notes/block-06-notes.md` ফাইলে নিচের পাঁচটি অংশ স্বাধীনভাবে বিশ্লেষণ করুন:

1. **Change Amplification (`LO 6.3`):** ধরুন Product টিম এই `promo_tags` table-এ মাত্র একটি নতুন column (`is_active BOOLEAN DEFAULT true`) যোগ করতে চাইল যাতে API-তে সেটি create ও return করা যায়। ওপরের ৭টি file-এর **ঠিক কোন কোন file ও method** edit করতে হবে, এবং কেন এটি **change amplification**-এর স্পষ্ট উদাহরণ?
2. **Cognitive Load (`LO 6.3`):** একজন নতুন engineer যখন বুঝতে চান *"একটি invalid বা duplicate `code` পাঠালে এখন ঠিক কী ঘটে?"*, তখন এই ৭-file structure তাঁর **cognitive load** কীভাবে বাড়িয়ে দিচ্ছে (এবং কোথায় *obscurity* বা missing real validation লুকিয়ে আছে)?
3. **Shallow Modules & Hidden Complexity (`LO 6.3`, `LO 6.4`):** ওপরের ৭টি layer-এর প্রতিটিকে যাচাই করে বলুন কোনগুলো **shallow modules** (*classitis*), এবং প্রতিটি extra layer আসলেই কোনো **meaningful complexity hide** করছে কি না? যদি এই flow-এর কোনো boundary (যেমন HTTP request/response handling বনাম SQL database query) আসলেই useful হয়, তা স্পষ্টভাবে আলাদা করুন।
4. **Essential vs. Accidental Complexity (`LO 6.1`, `LO 6.4`):** এই 3-column `promo_tags` feature-টির আসল **essential complexity** কী কী (যেমন uniqueness constraint, input validation, HTTP status contract), আর ওপরের design-এ কোনগুলো **accidental complexity** ও **speculative generality / overengineering**?
5. **Simpler Deep-Module Refactoring (`LO 6.3`, `LO 6.4`):** আপনি যদি এই feature-টাকে শুরু থেকে clean ও maintainableভাবে design করতেন, তবে কয়টি module/file রাখতেন এবং প্রতিটার responsibility কী হতো? পাশাপাশি উল্লেখ করুন—ভবিষ্যতে ঠিক কোন **concrete, real requirement** এলে (hypothetical অনুমান নয়) একটি আলাদা service বা mapping layer যোগ করা আসলেই justified হবে?

---

### Exercise 2: Simple vs. Easy Classification *(Optional Extension — ~10–15 min)*

**Scenario:**
একটি ছোট Node.js backend service-এ **configuration ও incoming order-request validation** handle করার দুটি ভিন্ন approach নিচে দেওয়া হলো:

- **Approach A:**
  - যেকোনো function-এ config দরকার হলে সরাসরি `const taxRate = Number(process.env.TAX_RATE || 0.10);` লিখে ব্যবহার করা হয়।
  - Route handler-গুলোর ভেতর scattered `if (!req.body.amount || req.body.amount < 0)` check রাখা হয়েছে, আর যেখানে check মনে পড়েনি সেখানে database error-এর ওপর ছেড়ে দেওয়া হয়েছে।
  - Test চালানোর সময় কয়েকটা file সরাসরি `process.env.TAX_RATE = "0.20"` mutate করে test চালায়।
- **Approach B:**
  - Service boot হওয়ার সময়ই একটা dedicated `loadConfig()` module সমস্ত required environment variable একবার validate ও parse করে একটি **immutable config object** তৈরি করে (জরুরি config না থাকলে শুরুতেই clear error দিয়ে fail-fast করে)।
  - Request validation-কে একটা pure `validateOrderInput(rawPayload)` function-এ আলাদা রাখা হয়েছে, যা valid structured data অথবা explicit validation errors ফেরত দেয়—এবং core pricing function-টি `config` ও validated `order` সরাসরি parameter হিসেবে নেয় (global `process.env` পড়ে না)।

**Tasks:**
1. **Simple vs. Easy Classification (`LO 6.2`):** Rich Hickey-র framework অনুযায়ী Approach A এবং Approach B-কে **simple** বনাম **easy** হিসেবে classify করুন। Approach A কেন Day 1-এ *easy* মনে হয়, আর এটি ঠিক কোন কোন concern-কে একসাথে **complect** করে ফেলছে?
2. **Six Months Later — Change Behavior (`LO 6.2`, `LO 6.3`, `LO 6.4`):** ৬ মাস পর দুটি নতুন requirement এল: (i) এখন একটাই global `TAX_RATE` নয়, customer-এর region অনুযায়ী ভিন্ন tax rate লাগবে; এবং (ii) একই order validation একটি background CSV-import worker থেকেও চালাতে হবে (যেখানে কোনো HTTP `req.body` নেই)। তুলনা করে দেখান—এই দুই change-এ Approach A বনাম Approach B-তে **change amplification**, **cognitive load**, এবং **unknown unknowns** কীভাবে ভিন্নভাবে প্রকাশ পাবে?

---

## Closed-Book Mastery Assessment *(Optional — ~25–35 min)*

এই assessment-টি সম্পূর্ণ **optional**—Block 7 শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র approval নেওয়া mandatory নয়। আপনি চাইলে:
- এই lesson ও বাইরের সব tab বন্ধ করে নিচের পাঁচটি প্রশ্নের clear, original উত্তর `phases/phase-01/notes/block-06-notes.md` ফাইলে লিখে formal evaluation-এর জন্য **ChatGPT**-র কাছে share করতে পারেন (`Mastery verified`-এর evidence হিসেবে);
- অথবা আপনি যথেষ্ট বুঝেছেন মনে করলে assessment skip করে পরের block-এ এগোতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment না করেই যেকোনো unclear concept নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(Note: `~25–35 min` কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি approximate estimate—কোনো strict time limit নয়।)*

1. **(LO 6.1 / Outcomes 6, 8)** Fred Brooks (1987)-এর **essential complexity** এবং **accidental complexity**-র পার্থক্য একটি বাস্তব backend উদাহরণসহ ব্যাখ্যা করুন। Brooks-এর চারটি intrinsic property (*complexity, conformity, changeability, invisibility*)-র সূত্র ধরে বলুন—কেন কোনো নতুন framework বা AI coding tool একাই essential complexity দূর করতে পারে না?
2. **(LO 6.2 / Outcomes 4, 8)** Rich Hickey (2011)-এর **simple** বনাম **easy** পার্থক্য এবং **complecting** ধারণা ব্যাখ্যা করুন। কেন *"কম line code"* বা *"familiar/quick to write"* (যেমন global mutable state বা দ্রুত copy-paste করা AI snippet) automatically simple বা maintainable হয় না?
3. **(LO 6.3 / Outcomes 4, 8)** John Ousterhout-এর **three symptoms of complexity** (*change amplification*, *cognitive load*, *unknown unknowns*) এবং **two root causes** (*dependencies*, *obscurity*) একটি backend উদাহরণ দিয়ে ব্যাখ্যা করুন। এই তিনটির মধ্যে *unknown unknowns* কেন সবচেয়ে বিপজ্জনক?
4. **(LO 6.3 / Outcomes 6, 8)** **Tactical programming** (এবং *Tactical Tornado*) বনাম **Strategic programming**-এর দীর্ঘমেয়াদি ফলাফল তুলনা করুন। এরপর **deep modules** বনাম **shallow modules** (*classitis*) ব্যাখ্যা করে দেখান—কেন বেশি abstraction layer থাকলেই codebase বেশি maintainable হয় না?
5. **(LO 6.4 / Outcomes 4, 6, 8)** **Overengineering (speculative generality)** কীভাবে আজকের maintainability ক্ষতিগ্রস্ত করে তা ব্যাখ্যা করুন। এরপর Martin Fowler-এর **Technical Debt Quadrant**-এর দুটি axis ও চারটি quadrant উদাহরণসহ ব্যাখ্যা করে দেখান—**prudent, deliberate technical debt** কীভাবে **reckless mess** থেকে আলাদা।

### Block 6 Readiness & Progression Checklist
- [ ] Guided lesson মনোযোগ দিয়ে পড়েছেন এবং Pause & Checks 1–3 সম্পন্ন করেছেন (পরে Self-Check Guidance মিলিয়ে দেখেছেন)।
- [ ] তিনটি Mandatory Resource (**Rich Hickey** *Simple Made Easy*, **John Ousterhout** *Talks at Google* selected sections, এবং **Martin Fowler** *Technical Debt Quadrant*) সম্পন্ন করেছেন।
- [ ] `phases/phase-01/notes/block-06-notes.md` ফাইলে স্বাধীনভাবে **Exercise 1 (Mandatory — Overengineering Critique)** লিখেছেন।
- [ ] *(Optional)* `notes/block-06-notes.md`-এ **Exercise 2 (Simple vs. Easy Classification)** সম্পন্ন করেছেন।
- [ ] *(Optional)* Notes/tabs বন্ধ রেখে Mastery Assessment Questions 1–5 লিখেছেন এবং formal evaluation-এর জন্য ChatGPT-র কাছে share করেছেন।
- [ ] *(Optional)* কোনো concept unclear মনে হলে assessment ছাড়াই সেগুলো নিয়ে ChatGPT-র সাথে আলোচনা করে পরিষ্কার হয়েছেন।

> **Progression Status Distinction:** Block 7 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও চর্চা শেষ করে assessment ছাড়া এগোলে এই block-টির status হবে **`Studied — mastery not formally verified`**। যথেষ্ট evidence (exercise ও/অথবা assessment) share করে evaluation সম্পন্ন করলেই তা **`Mastery verified`** হিসেবে গণ্য হবে; assessment skip করলে তা automatically `Mastery verified` হয় না।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`notes/block-06-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের guidance দেখুন। Mandatory Exercise 1, Optional Exercise 2, বা Optional Closed-Book Mastery Assessment-এর কোনো solution এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) optional ChatGPT review-এর জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* **(ক)** বাংলাদেশের VAT ও আন্তর্জাতিক tax exemption rules হলো **essential complexity**—কারণ এগুলো বাস্তব আইন ও business domain-এর অংশ, যা কোনো tool বা language বদলে বাদ দেওয়া যাবে না। **(খ)** ওই logic দুটি file-এ copy-paste করে রাখা এবং global `process.env.DEFAULT_TAX`-এর ওপর নির্ভর করা হলো **accidental complexity**—কারণ এটি problem-এর চাহিদা নয়, বরং দুর্বল design choice, যা একটি shared pure tax-calculation module ও explicit config parameter দিয়ে সহজেই দূর করা যায়।
  2. *শক্তিশালী উত্তরে থাকবে:* সহকর্মী *short* এবং *easy* (familiar, ৫ মিনিটে লেখা যায়)-কে **simple**-এর সাথে গুলিয়ে ফেলেছেন। Dense nested ternary অনেক condition একসাথে পেঁচিয়ে (complect করে) readability কমায়, আর ORM auto-hook persistence-এর সাথে hidden business side effects complect করে ফেলে—ফলে পরে কেউ database save করলে অজান্তে unexpected behavior ঘটবে এবং maintainability নষ্ট হবে।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* **Root causes:** (১) **Obscurity** — function-এর নাম `formatOrderSummary` শুনে বোঝা যায় না যে এটি input mutate করে এবং database write করে (hidden side effects); (২) **Dependencies** — caller-এর invoice calculation এখন গোপনে `formatOrderSummary` আগে call হয়েছে কি না তার ওপর নির্ভরশীল। **Symptoms:** সবচেয়ে তীব্রভাবে **Unknown Unknowns** (নতুন developer জানতেই পারেননি summary format করলে shipping fee দুবার add হয়ে যাবে, ফলে production bug ঘটল) এবং উচ্চ **Cognitive Load** (function-এর নাম বিশ্বাস না করে ভেতরের প্রতিটা লাইন পড়ে side effect খুঁজতে হয়)।
  2. *শক্তিশালী উত্তরে থাকবে:* একটা 50-line self-contained function যদি একটি ছোট, clear interface-এর পেছনে পুরো coherent logic-টা সরাসরি ও readable উপায়ে handle করে, তবে সেটি **deep** এবং কম cognitive load দেয়। বিপরীতে ৫টা 10-line pass-through class কোনো real complexity hide না করে শুধু extra interfaces, boilerplate, আর file-jumping বাড়ায় (shallow modules / classitis)—ফলে code ছোট ছোট টুকরো হওয়া সত্ত্বেও overall complexity বেড়ে যায়।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* **(ক) Prudent & Inadvertent** (*"Now we know how we should have done it"* — শুরুতে care নিয়ে ভালো design করেছিল, পরে real domain experience থেকে আরও ভালো boundary আবিষ্কার হয়েছে); **(খ) Reckless & Deliberate** (*"We don't have time for design"* — কোনো strategic justification ছাড়াই ইচ্ছাকৃতভাবে design ও test বাদ দিয়ে reckless mess তৈরি); **(গ) Prudent & Deliberate** (*"We must ship now and deal with consequences"* — hard partner demo ধরতে সচেতন, bounded shortcut নিয়ে limitation document করা ও repayment plan রাখা)।
  2. *শক্তিশালী উত্তরে থাকবে:* **আজকের cost:** যে ৫টা gateway এখনো exists-ই করে না, তার জন্য generic plugin registry ও ৪টা abstract layer বানালে আজ থেকেই accidental complexity, shallow pass-through code, cognitive load, আর opportunity cost দিতে হবে—আর ভবিষ্যতে দ্বিতীয় gateway এলে তার আসল API contract (যেমন webhook/refund flow) আপনার আজকের অনুমানের সাথে নাও মিলতে পারে! **Simpler prudent approach:** আজ শুধু Stripe-কে একটি clean, **deep** internal payment module-এর পেছনে wrap করে রাখুন (যাতে Stripe-specific details বাকি codebase-এ leak না করে), কিন্তু speculative multi-gateway framework বানাবেন না যতক্ষণ না দ্বিতীয় বাস্তব gateway-র প্রমাণিত requirement আসে।
