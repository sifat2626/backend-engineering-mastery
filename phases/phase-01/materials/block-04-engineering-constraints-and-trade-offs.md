# Phase 1, Block 4 — Engineering Constraints and Trade-offs

## Study Time Guide

| কার্যক্রম (Activity) | নির্ধারিত পরিসর (Scope) | আনুমানিক সময় (Estimated Time) |
| :--- | :--- | :---: |
| **Guided Lesson + Pause & Checks 1–3** *(মূল অগ্রাধিকার)* | পুরো মূল lesson গভীরভাবে পড়া ও `notes/block-04-notes.md`-এ inline check লেখা | **~85–105 min** *(নমনীয় — সীমা নয়)* |
| **Mandatory Reading 1:** Michael Nygard (2011) | *"Documenting Architecture Decisions"* — পুরো সংক্ষিপ্ত প্রবন্ধ (*Context*, *Decision*, *Status*, *Consequences*, *Experience Report*; *More Reading* লিংকগুলো বাদে) | ~10–15 min |
| **Mandatory Reading 2:** Google SRE Book, Chapter 3 | *Embracing Risk* — কেবল *Managing Risk*, *Risk Tolerance of Services*-এর ধারণাগত কাঠামো, এবং *Motivation for Error Budgets* *(গাণিতিক হিসাব ও policy অংশ Phase 21-এর জন্য বাদ)* | ~15–20 min |
| **Mandatory Reading 3:** *SWE at Google*, Chapter 1 | কেবল *Trade-offs and Costs*, *Inputs to Decision Making*, এবং *Revisiting Decisions, Making Mistakes* *(Block 1-এর অংশগুলো বাদ)* | ~15–20 min |
| **Exercise 1:** Constraint-Driven ADR | ২-engineer SaaS startup-এর জন্য দুই বিকল্পের তুলনা ও পূর্ণ ১-পৃষ্ঠার ADR লেখা | ~35–45 min |
| **Exercise 2:** Reliability Cost & Risk Tolerance | একটি internal weekly reporting tool-এর জন্য 99.999% বনাম 99.5% লক্ষ্যমাত্রার trade-off বিশ্লেষণ | ~20–25 min |
| **মোট (Optional Assessment ছাড়া)** | Guided Lesson + ৩টি Mandatory Reading + ২টি Exercise | **~180–230 min (~3–3.8 h)** |
| **Optional Closed-Book Assessment** *(ঐচ্ছিক)* | বই/নোট বন্ধ রেখে ৫টি সংশ্লেষণাত্মক (synthesis) প্রশ্নের উত্তর লেখা | ~30–40 min *(optional)* |
| **মোট (Optional Assessment সহ)** | ওপরের সব কার্যক্রম + ঐচ্ছিক Closed-Book Assessment | **~210–270 min (~3.5–4.5 h)** |

> **পরিকল্পনা-নির্দেশিকা (Planning Note):** ওপরের প্রতিটি সময় কেবল নমনীয় পরিকল্পনা-সহায়িকা—কোনো কার্যক্রমেরই কঠোর সময়সীমা (deadline) নেই। দ্রুত শেষ করার চেয়ে ধারণাগুলো গভীরভাবে বোঝা ও স্বাধীনভাবে যুক্তি দাঁড় করানোই মূল অগ্রাধিকার। **Guided Lesson** এই block-এর প্রধান শেখার কার্যক্রম, তাই প্রয়োজনে এর জন্য আনুমানিক সময়ের চেয়ে বেশি সময় লাগা সম্পূর্ণ স্বাভাবিক। প্রতিটি reading-এর সময় কেবল এই block-এ নির্ধারিত অংশগুলোর জন্য হিসাব করা হয়েছে, পুরো অধ্যায় বা বই পড়ার জন্য নয়। **Optional Closed-Book Assessment** সম্পূর্ণ ঐচ্ছিক এবং পরের block-এ এগোনোর জন্য বাধ্যতামূলক নয়। *(মূল roadmap-এ Block 4-এর ৩–৪ ঘণ্টার—গবেষণা ডসিয়ারে ~৩.৫ ঘণ্টার—বরাদ্দটি একটি পরিকল্পনা-রেফারেন্স হিসেবেই অপরিবর্তিত আছে।)*

---

Block 2-এ আপনি কোড লেখার আগেই **Problem Space** *(ব্যবহারকারী ও ব্যবসার প্রকৃত সমস্যা, অস্পষ্টতা, অনুমান ও সীমানা)* পরিষ্কার করতে শিখেছেন, আর Block 3-এ দেখেছেন যে software-এর গুণমান একক কোনো মাত্রা নয়—**ISO/IEC 25010:2023**-এর নয়টি quality attribute প্রায়ই পরস্পরের বিপরীতে টানে (যেমন Security বাড়ালে Performance Efficiency-তে চাপ পড়ে, কিংবা Reliability বাড়াতে গিয়ে Maintainability জটিল হয়ে ওঠে)।

এবার আমাদের সেই পরিচিত বাড়তে থাকা online store-এ ফিরে যান—ঠিক `LAUNCH50` promo campaign-এর ভাইরাল শুক্রবারের ঘটনার পরের সোমবার সকালে।

দোকানটি দ্রুত বড় হচ্ছে। এখন প্রতিদিন গড়ে ১৫,০০০ অর্ডার আসছে, আর **ঠিক পাঁচ সপ্তাহ পর** আসছে বছরের সবচেয়ে বড় মৌসুমি flash sale—যখন দিনে প্রায় ৫০,০০০ অর্ডার আসতে পারে বলে ব্যবসা টিমের অনুমান। আপনার backend টিমে আছেন মাত্র **৩ জন engineer**, যাঁরা সবাই **Node.js** ও **PostgreSQL**-এ দক্ষ। পরের অর্থনৈতিক মাইলফলকে না পৌঁছানো পর্যন্ত মাসিক cloud infrastructure বাজেট কঠোরভাবে **৮০০ ডলারের (USD 800/month)** মধ্যে আটকে রাখতে হবে। এর পাশাপাশি নতুন একটি payment-partner compliance নিয়ম চালু হয়েছে: প্রতিটি promo redemption এবং order-discount হিসাবের **audit record** *(কে, কখন, কোন অর্ডারে কত ছাড় পেল তার অপরিবর্তনীয় ইতিহাস)* কমপক্ষে **৫ বছর** নিরাপদে সংরক্ষণ করতে হবে—কোনো record নীরবে হারিয়ে যাওয়া চলবে না।

`LAUNCH50`-র ঘটনায় concurrent request-এর race condition আর ধীর checkout-এর অভিজ্ঞতা দেখে প্রতিষ্ঠাতা টিমকে বললেন: *"Flash sale-এর আগে আমাদের promo-redemption ব্যবস্থাটা নির্ভরযোগ্য করে ঢেলে সাজাও, আর Marketing টিমের জন্য campaign-এর ব্যবহার দেখার একটি analytics dashboard-ও দাও।"*

সোমবারের whiteboard বৈঠকে টিমের ভেতর থেকে দুটি প্রস্তাব উঠে এল:

- **প্রস্তাব ১ ("সবচেয়ে আধুনিক ও আকর্ষণীয় স্থাপত্য"):**
  একজন সহকর্মী সম্প্রতি একটি বড় টেক কোম্পানির engineering blog পড়ে উৎসাহিত হয়ে বললেন—*"চলো এবার কাজটা একদম industry best practice মেনে করি!"* তিনি প্রস্তাব দিলেন: আমাদের backend-কে তিনটি আলাদা **microservice**-এ ভাগ করি *(একটি বড় application-কে আলাদাভাবে deploy ও পরিচালনা করা যায় এমন ছোট ছোট স্বাধীন network service-এ ভাগ করা—বিস্তারিত Phase 22-এ)*—Checkout, Promo, এবং Analytics। Promo lookup দ্রুত করতে সামনে বসাই **Redis** *(দ্রুতগতির in-memory key-value data store—বিস্তারিত Phase 16-এ)*, Analytics-এ তাৎক্ষণিক event পাঠাতে বসাই **Apache Kafka** *(বড় পরিসরে event-streaming-এর জন্য distributed messaging platform—বিস্তারিত Phase 17-এ)*, আর checkout যাতে কখনোই বন্ধ না হয় সেজন্য দুটি ভিন্ন ভৌগোলিক অঞ্চলে active-active replication চালু করে লক্ষ্য রাখি **99.999% uptime**!
- **প্রস্তাব ২ ("সংযত ও সীমাবদ্ধতা-সচেতন নকশা"):**
  আরেকজন সহকর্মী বললেন—*"আমাদের বিদ্যমান একটিমাত্র Node.js backend service এবং managed PostgreSQL database-ই রাখি।"* `LAUNCH50`-এর concurrency ও negative-total সমস্যাগুলো দূর করতে PostgreSQL-এর ভেতরেই atomic transaction এবং `CHECK`/`UNIQUE` constraint ব্যবহার করি *(database-স্তরের এমন নিয়ম যা একইসাথে সমান্তরাল request-এর সংঘাত ও অবৈধ state ঠেকায়—বিস্তারিত Phase 10 ও 14-এ)*, দ্রুত lookup-এর জন্য সঠিক database index যোগ করি *(যা পুরো table না ঘেঁটেই দ্রুত row খুঁজে দেয়—Phase 10)*, এবং একই transaction-এ ৫-বছর-মেয়াদি redemption audit row লিখে রাখি। Marketing analytics-এর জন্য Kafka না বসিয়ে প্রতি ২ মিনিটে refresh হওয়া একটি হালকা summary query রাখি (অর্থাৎ sub-second real-time-এর বদলে ২ মিনিটের বিলম্ব মেনে নিই), এবং checkout-এর availability লক্ষ্য রাখি বাস্তবসম্মত **99.9%**।

একটু থেমে দুটি প্রস্তাবের দিকে তাকান।

কাগজে-কলমে **প্রস্তাব ১** দেখতে চমৎকার—সম্মেলনের স্লাইডে দারুণ মানাবে, প্রতিটি জনপ্রিয় প্রযুক্তির নাম তাতে আছে, আর ৯৯.৯৯৯% uptime শুনতেও বেশ গম্ভীর। **তাহলে আমরা চোখ বন্ধ করে প্রস্তাব ১ বেছে নিচ্ছি না কেন?**

কারণ প্রস্তাব ১ বাস্তবতার দেয়ালে ধাক্কা খেয়ে ভেঙে পড়বে। ৩ জন engineer—যাঁদের কেউ কখনো production-এ Kafka বা multi-region database replication চালাননি—তাঁরা ৫ সপ্তাহের মধ্যে ৩টি distributed service, Redis, Kafka এবং multi-region failover নির্ভুলভাবে তৈরি ও নিরাপদে চালু করতে পারবেন না; মাসিক cloud খরচ ৮০০ ডলারের কয়েক গুণ ছাড়িয়ে যাবে; আর flash sale-এর রাতে কোনো network সমস্যা হলে ৩ জন মানুষ রাত ৩টায় ৫টি distributed উপাদান সামলাতে গিয়ে সম্পূর্ণ বিপর্যয়ে পড়বেন।

আবার উল্টোদিকে, **প্রস্তাব ২-ও কি "নিখুঁত"?** মোটেই না! এতে Marketing টিম তাৎক্ষণিক নয়, ২ মিনিট পুরোনো সংখ্যা দেখবে; checkout এবং analytics একই PostgreSQL database ভাগ করে নেওয়ায় অসতর্ক ভারী analytics query checkout-কে ধীর করে দেওয়ার ঝুঁকি তৈরি করবে; আর আগামী দুই বছরে অর্ডার ৫০ গুণ বেড়ে গেলে একক database-টি একসময় সীমায় পৌঁছে যাবে।

এই একটি বাস্তব দ্বিধার ভেতর থেকেই আমাদের পুরো block-টির কেন্দ্রীয় প্রশ্ন জন্ম নেয়:

> **যখন একটি backend সমস্যার একাধিক কারিগরি সমাধান থাকে—আর কোনো সমাধানই নিখুঁত নয়—তখন একজন engineer কীভাবে বাস্তব সীমাবদ্ধতার ভেতর দাঁড়িয়ে যুক্তিসঙ্গত বিকল্প বাছাই করেন, প্রতিটি বিকল্পের প্রকৃত মূল্য ও ত্যাগ হিসাব করেন, এবং ভবিষ্যতের টিমের জন্য সেই সিদ্ধান্তের যুক্তি ধরে রাখেন?**

---

## অদৃশ্য দেয়াল চেনা: Engineering Constraints এবং বহুমাত্রিক খরচ

নতুন engineer-রা প্রায়ই একটি নকশা বিচার করেন কেবল বিশুদ্ধ কারিগরি সৌন্দর্যের মাপকাঠিতে—*"কোন tool-টা সবচেয়ে দ্রুত?", "কোন architecture-টা সবচেয়ে আধুনিক?"* কিন্তু বাস্তব software engineering কখনো শূন্য মহাকাশে হয় না। প্রতিটি প্রকৃত প্রকল্পের চারপাশে কয়েকটি বাস্তব দেয়াল দাঁড়িয়ে থাকে, যেগুলো নির্ধারণ করে কোন সমাধানগুলো আসলেই সম্ভব (**feasible**), আর কোনগুলো কেবল হোয়াইটবোর্ডের কল্পনা। এই বাস্তব সীমানাগুলোকেই বলা হয় **Engineering Constraints**।

আমাদের online store-এর দৃশ্যপটটি আরেকবার খুঁটিয়ে দেখলে টের পাবেন, কারিগরি সীমাবদ্ধতা ওই দেয়ালগুলোর মাত্র একটি অংশ। বাস্তব production পরিবেশে engineering constraints প্রধানত **ছয়টি স্বতন্ত্র মাত্রায় (six dimensions)** কাজ করে:

| Constraint Dimension | এটি আসলে কী সীমাবদ্ধ করে | আমাদের Online Store-এ বাস্তব উদাহরণ | উপেক্ষা করলে কী ব্যর্থতা ঘটে |
| :--- | :--- | :--- | :--- |
| **1. Technical** | বিদ্যমান প্রযুক্তি-স্তর, পুরোনো API contract, data invariant, latency বা পদার্থবিদ্যার সীমা। | Node.js + PostgreSQL stack; existing mobile app-এর API contract (Block 1-এর Hyrum's Law — অর্থাৎ API-র যেকোনো দৃশ্যমান আচরণের ওপর ব্যবহারকারীরা নির্ভর করে বসেন); `used_count <= max` ও `final_total >= 0` invariant। | সমাধানটি বিদ্যমান client ভেঙে দেয়, data corrupt করে, বা বাস্তব latency সীমা অতিক্রম করে। |
| **2. Resource & Cost** | অর্থ (cloud বিল, license), compute/RAM/storage, engineering সময় এবং opportunity cost। | মাসিক cloud বাজেট সর্বোচ্চ USD 800; সীমিত CPU/RAM ও ৩ জন engineer-এর সাপ্তাহিক কর্মঘণ্টা। | বাজেট ফুরিয়ে যায়, কিংবা অপ্রয়োজনীয় অবকাঠামো বানাতে গিয়ে গুরুত্বপূর্ণ customer feature থেমে যায়। |
| **3. Schedule** | নির্দিষ্ট সময়সীমা, মৌসুমি launch তারিখ, ব্যবসায়িক চুক্তি বা বাজার-সুযোগের জানালা। | ঠিক ৫ সপ্তাহ পর মৌসুমি flash sale—তার আগেই নিরাপদে deploy ও verify করতে হবে। | "নিখুঁত" architecture ৭০% অর্ধসমাপ্ত অবস্থায় পড়ে থাকে; flash sale-এ পুরোনো ভাঙা কোডই চালাতে হয়। |
| **4. Team & Organizational** | টিমের আকার, দক্ষতার ধরন, cognitive load (মানসিক বোঝা), সাংগঠনিক কাঠামো ও অন্য টিমের ওপর নির্ভরতা। | মাত্র ৩ জন engineer; Node.js ও PostgreSQL-এ দক্ষ, কিন্তু Kafka বা multi-region ops অভিজ্ঞতা শূন্য। | যে প্রযুক্তি টিম গভীরভাবে বোঝে না, production-এ সেটি ভাঙলে কেউ দ্রুত debug বা recover করতে পারেন না। |
| **5. Regulatory & Compliance** | আইন, আর্থিক নিরীক্ষা, গোপনীয়তা বিধি, data residency বা শিল্পখাতের বাধ্যতামূলক মানদণ্ড। | Payment partner-এর শর্ত অনুযায়ী ৫ বছর অপরিবর্তনীয় redemption audit record সংরক্ষণ (কিংবা card data-র জন্য PCI-DSS বা আঞ্চলিক privacy আইন)। | আইনগত জরিমানা, payment gateway-র সংযোগ বিচ্ছিন্ন হওয়া, বা বাধ্যতামূলক জরুরি পুনর্লিখন। |
| **6. Operational** | চলমান সিস্টেমকে deploy, monitor, backup, recover এবং রাত ৩টায় on-call সামলানোর বাস্তব সক্ষমতা। | ৩ জনের টিমকে deployment, alert, backup ও incident response—সব সামলাতে হবে; বড় কোনো নিবেদিত SRE টিম নেই। | বহু distributed অংশের জটিল ব্যর্থতা সামলাতে গিয়ে alert fatigue, দীর্ঘ outage ও টিমের burnout। |

এখানে একটি সূক্ষ্ম কিন্তু অত্যাবশ্যকীয় পার্থক্য লক্ষ করুন। যেকোনো প্রকল্পের সব constraint সমান অনমনীয় নয়। Engineer-রা এগুলোকে দুই ভাগে ভাগ করেন:

- **Hard Constraint (কঠোর ও অনমনীয় সীমা):** যে সীমা ভাঙলে সমাধানটি সরাসরি অকার্যকর, অবৈধ, বা অগ্রহণযোগ্য বলে বাতিল হয়ে যায়। আমাদের দৃশ্যপটে ৫ বছরের audit retention (regulatory), ৫ সপ্তাহ পরের flash sale তারিখ (schedule—মৌসুমি উৎসব পেছানো যাবে না!), USD 800-এর কঠোর runway সীমা (financial), এবং `final_total >= 0` data invariant (technical)—এগুলো হলো Hard Constraint। কোনো সমাধান যতই আধুনিক হোক, Hard Constraint ভাঙলে তা আর *viable alternative* থাকে না।
- **Soft Constraint (নমনীয় বা আপসযোগ্য অগ্রাধিকার):** এমন লক্ষ্য বা পছন্দ যা পূরণ হলে ভালো হয়, কিন্তু বড় কোনো Hard Constraint রক্ষা করতে গিয়ে stakeholder-দের সম্মতিতে যেখানে যুক্তিসঙ্গত ছাড় (compromise) দেওয়া যায়। যেমন: Marketing analytics dashboard-এ ডেটা ১ সেকেন্ডের বদলে ২ মিনিট পর আপডেট হওয়া, বা প্রথম ধাপে নতুন campaign তৈরির সুন্দর UI না বানিয়ে নিরাপদ internal admin script ব্যবহার করা।

একজন পরিণত engineer-এর একটি বিশেষ অভ্যাস আছে: **তিনি অন্ধভাবে সব constraint মেনে নেন না, আবার অন্ধভাবে কোনোটি উপেক্ষাও করেন না।** প্রথমত, Block 2-এর *Fact vs. Assumption* চর্চা মনে করে তিনি যাচাই করেন—যেটিকে Hard Constraint বলা হচ্ছে, সেটি কি আসলেই Hard, নাকি কেবল কারও অনুমান (*assumed constraint*)? যেমন Marketing টিম হয়তো মুখে বলেছিলেন *"আমাদের real-time analytics চাই"*, কিন্তু কথা বলে জানা গেল তাঁরা প্রতি ঘণ্টায় একবার dashboard খুলে দেখেন—অর্থাৎ "real-time" আসলে Hard নয়, Soft (বা ভুল অনুমানকৃত) constraint ছিল! দ্বিতীয়ত, সত্যিকারের Hard Constraint (যেমন compliance বা ৩ জনের টিমের on-call ক্ষমতা) চিহ্নিত হওয়ার পর তিনি সেটিকে সমাধান-বিকল্পের প্রাথমিক ছাঁকনি (filter) হিসেবে ব্যবহার করেন।

এখানে **"খরচ (Cost)"** শব্দটিকে কেবল cloud বিলের ডলারে আটকে রাখলে সবচেয়ে বড় ফাঁদে পড়বেন। Block 1-এ Titus Winters, Tom Manshreck ও Hyrum Wright (*Software Engineering at Google*, 2020, Chapter 1)-এর **ছয়টি cost dimension**-এর সাথে আপনার প্রাথমিক পরিচয় হয়েছিল; Block 4-এ সিদ্ধান্ত নেওয়ার সময় এই ছয়টি মাত্রা কীভাবে সরাসরি বিকল্প নির্বাচনের হিসাব বদলে দেয় তা দেখুন:

1. **Financial Cost:** সরাসরি অর্থ ব্যয় (cloud server, managed DB, license)।
2. **Resource Cost:** CPU, RAM, disk I/O এবং network bandwidth।
3. **Personnel Cost:** তৈরি, শেখা, review, debug ও পরিচালনায় engineer-দের সময় ও মানসিক শক্তি। Winters et al. একটি অসাধারণ বাস্তব সত্য মনে করিয়ে দেন: *software engineering-এর মতো সৃজনশীল ক্ষেত্রে প্রায়ই টাকার অঙ্ক নয়, বরং **personnel cost**-ই আসল bottleneck!* (তাঁদের *Example: Markers*-এ দেখানো হয়েছে, ১ ডলারের whiteboard marker বাঁচাতে গিয়ে তালাবন্ধ করে রাখলে বৈঠকের মনোযোগ ও engineering সময়ে যে ক্ষতি হয়, তা marker-এর দামের চেয়ে বহু গুণ বেশি।)
4. **Transaction Cost:** এক অবস্থা থেকে আরেক অবস্থায় যেতে—যেমন migration, schema বদলানো, বা টিমের অভ্যাস পরিবর্তন করতে—যে শ্রম ও ঝুঁকি লাগে।
5. **Opportunity Cost:** একটি পথে সীমিত সময় ও শক্তি ব্যয় করায় **অন্য কোন মূল্যবান কাজটি আর করা গেল না** (যেমন Kafka setup-এ চার সপ্তাহ খরচ করায় checkout-এর refund bug ঠিক না হওয়া বা flash-sale load test বাদ পড়ে যাওয়া)।
6. **Societal / User Cost:** আমাদের সিদ্ধান্তের ফলে গ্রাহক বা সমাজের ওপর কী বাস্তব ক্ষতি বা ঝুঁকি তৈরি হতে পারে (ডাবল চার্জ, ব্যক্তিগত তথ্য ফাঁস, অকার্যকর সেবা, বা অন্যায্য পক্ষপাত)।

আর Winters et al. (*Inputs to Decision Making*) আরও দুটি গভীর বাস্তবতার কথা বলেন, যা প্রায় প্রতিটি trade-off-এ আপনার সামনে আসবে:
- **সব ইনপুট সমানভাবে মাপা যায় না:** কিছু জিনিস সরাসরি পরিমাপ বা অনুমান করা যায় (যেমন *২ সপ্তাহ engineering সময় ব্যয় করে মাসে ৪০০ ডলারের CPU বাঁচানো লাভজনক কি না*)। কিন্তু অনেক অত্যন্ত গুরুত্বপূর্ণ খরচ—যেমন একটি অস্পষ্ট API-র দীর্ঘমেয়াদি cognitive load, বা গ্রাহকের আস্থা হারানোর ক্ষতি—সহজে একটি সংখ্যায় বাঁধা যায় না। তাই যেগুলো মাপা যায় না সেগুলোকে উপেক্ষা না করে বরং **অতিরিক্ত যত্ন ও বিচারবুদ্ধি** দিয়ে বিবেচনা করতে হয়।
- **অপ্রত্যাশিত পরোক্ষ খরচ (Unforeseen Downstream Effects):** কোনো একটি কাজ সহজ বা সস্তা হয়ে গেলে মানুষ অনেক সময় তা আরও বেশি করে ব্যবহার করতে শুরু করে—অর্থনীতিতে একে **Jevons Paradox** *(কোনো সম্পদ ব্যবহারে দক্ষতা বাড়লে অনেক সময় তার মোট খরচ কমার বদলে উল্টো বেড়ে যাওয়া)* বলে। *SWE at Google*-এ দেখানো হয়েছে, distributed build system বানিয়ে Google যখন build অনেক দ্রুত করল, তখন engineer-রা build-এর কষ্ট আর গায়ে লাগাতেন না বলে কোডে অপ্রয়োজনীয় dependency জমে build আবার ভারী হয়ে গেল! অর্থাৎ একটি trade-off তাৎক্ষণিক যে খরচ কমায়, সময়ের সাথে সেটি ভিন্ন জায়গায় নতুন খরচ তৈরি করতে পারে।

> **Pause & Check 1** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-04-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. ধরুন, ৪ জন backend engineer-এর একটি স্বাস্থ্যসেবা (healthtech) startup আগামী ৮ সপ্তাহের মধ্যে রোগীদের ল্যাব-রিপোর্ট দেখার API চালু করতে চায়। স্থানীয় আইন অনুযায়ী রোগীর ডেটা দেশের বাইরের কোনো server-এ রাখা যাবে না; টিম কেবল Node.js ও PostgreSQL জানে; আর ডাক্তাররা বলেছেন রিপোর্ট সাধারণত ২ সেকেন্ডের মধ্যে খুললে ভালো হয়। এখান থেকে অন্তত **চারটি আলাদা মাত্রার (dimensions)** constraint চিহ্নিত করুন এবং কোনটি **Hard** আর কোনটি **Soft** constraint হতে পারে তা যুক্তিসহ আলাদা করুন।
> 2. ওই startup-এর একজন পরামর্শক বললেন: *"Managed PostgreSQL না নিয়ে নিজেরাই Linux VM-এ PostgreSQL বসাই—মাসে ২৫০ ডলার বাঁচবে!"* এখানে কেবল **Financial Cost** না দেখে Winters et al.-এর অন্তত **তিনটি** অন্য cost dimension (যেমন Personnel, Opportunity, Transaction, বা Societal/User) দিয়ে দেখান কেন এই "সস্তা" সিদ্ধান্তটি আসলেই অনেক বেশি ব্যয়বহুল হতে পারে।

এবার ধরুন, আপনি সতর্কতার সাথে আপনার প্রকল্পের ছয়টি মাত্রার Hard ও Soft constraint খুঁজে বের করেছেন। কিন্তু সেই দেয়ালগুলোর ভেতর দাঁড়িয়েই আপনি এক অদ্ভুত বাস্তবতার মুখোমুখি হবেন: **দেয়ালগুলোর ভেতর যে কটি বৈধ সমাধান আছে, তাদের একটিও পুরোপুরি নিখুঁত নয়!** কোনোটা দ্রুত কিন্তু পরিচালনায় জটিল, কোনোটা সরল কিন্তু বাড়তি চাহিদায় সীমাবদ্ধ, কোনোটা অত্যন্ত নির্ভরযোগ্য কিন্তু খরচে আকাশছোঁয়া। তাহলে একজন engineer কীভাবে বোঝেন—"ভালো" নকশা বলতে আসলে কী বোঝায়, আর *"industry best practice"* বা *"100% uptime"*-এর মতো চকচকে লক্ষ্যগুলো কেন প্রায়ই ফাঁদ হয়ে দাঁড়ায়?

---

## সবকিছুই Trade-off: "Best Practice" নয়, Context-Dependent "Least Worst"

অনেক জুনিয়র engineer (এবং দুর্ভাগ্যজনকভাবে অনেক অভিজ্ঞ ডেভেলপারও) একটি কাল্পনিক জিনিস খুঁজতে থাকেন: *"এই সমস্যার একমাত্র সঠিক বা best-practice সমাধান কী?"*

সফটওয়্যার স্থাপত্যের দুই প্রবীণ লেখক **Mark Richards এবং Neal Ford** (*Fundamentals of Software Architecture*, 1st ed. 2020 / 2nd ed. 2025) এই ভ্রমটিকে এক বাক্যে চূর্ণ করে দিয়েছেন, যা আজ **First Law of Software Architecture** নামে সুপরিচিত:

> **"Everything in software architecture is a trade-off."**
> *(সফটওয়্যার স্থাপত্যের প্রতিটি সিদ্ধান্তই একটি trade-off।)*

এর অর্থ অত্যন্ত স্পষ্ট ও গভীর: কোনো কারিগরি পছন্দ যদি দেখে মনে হয় *"এতে শুধু লাভই লাভ, কোনো ক্ষতি নেই"*, তাহলে বুঝবেন আপনি এখনো এর লুকানো খরচটা খুঁজে পাননি! আর ঠিক এই প্রথম সূত্র থেকেই বেরিয়ে আসে Richards ও Ford-এর সেই মুক্তিকামী অনুসিদ্ধান্ত (corollary):

> **সফটওয়্যার প্রকৌশলে "সর্বোত্তম (the best)" নকশা বলে সর্বজনীন কিছু নেই; একজন engineer-এর কাজ হলো নির্দিষ্ট প্রেক্ষাপট ও সীমাবদ্ধতার জন্য "সবচেয়ে কম ক্ষতিকর (least worst)" trade-off-এর সমন্বয় খুঁজে বের করা।**

এখান থেকেই একটি ভীষণ ক্ষতিকর ভুল ধারণা ভেঙে ফেলা দরকার:

- **ভুল ধারণা: *"ভালো engineering মানেই সব সময় বড় কোম্পানির 'best practices' (যেমন microservices, 99.999% uptime, zero technical debt) বেছে নেওয়া।"***
  কেন এটি বিপজ্জনক? কারণ প্রতিটি তথাকথিত "best practice" জন্ম নিয়েছে একটি *নির্দিষ্ট* প্রেক্ষাপটের সীমাবদ্ধতা সামলাতে। Netflix বা Google যখন শত শত microservices চালায়, তখন তাদের পেছনে থাকে হাজারো engineer, নিবেদিত platform ও SRE টিম, এবং কোটি কোটি সমান্তরাল request-এর সাংগঠনিক ও প্রযুক্তিগত চাপ। একজন ৩-engineer startup যখন ওই একই প্রেক্ষাপট ছাড়া microservices বা multi-region cluster নকল করে (**cargo-culting**), তখন সে বড় কোম্পানির *সমাধানের জটিলতা ও operational খরচ* ঠিকই আমদানি করে, কিন্তু যে সমস্যার জন্য ওটা তৈরি হয়েছিল সেই সমস্যাটাই তার নেই! ফলে যা ৫০০ জনের টিমে "least worst" ছিল, তা ৩ জনের টিমে পরিণত হয় ধ্বংসাত্মক **overengineering**-এ (যা আমরা Block 6-এ আরও গভীরে ব্যবচ্ছেদ করব)।

একটি ভালো decision-making সংস্কৃতিতে তাই Titus Winters et al. (*SWE at Google*, Ch. 1)-এর কথাটি সব সময় মনে রাখা জরুরি: কোনো সিদ্ধান্তের কারণ কখনোই *"কারণ আমি বলেছি (because I said so)"* বা *"কারণ বড় সব কোম্পানি এটাই করে (because everyone else does it)"* হতে পারে না। প্রকৃত engineering সিদ্ধান্তের পেছনে থাকে মাত্র দুটি বৈধ কারণ:
1. আমরা এটি করছি কারণ আমাদের **করতেই হবে** (আইনি, compliance, বা কঠোর ব্যবসায়িক Hard Constraint), অথবা
2. বর্তমান প্রমাণ ও সীমাবদ্ধতার আলোকে এটিই আমাদের দেখা **সবচেয়ে যুক্তিসঙ্গত trade-off**—যার পেছনে একটি স্পষ্ট *"কারণ (because...)"* আছে।

*(এর পাশাপাশি Winters et al. আমাদের নিজস্ব মানসিক পক্ষপাত—যেমন বর্তমান অবস্থাকে বদলাতে না চাওয়ার **status quo bias** বা সম্ভাব্য লাভের চেয়ে ক্ষতির ভয় বেশি কাজ করার **loss aversion**—সম্পর্কেও সচেতন থাকতে বলেছেন।)*

### কেন "100% Reliability" প্রায় সব সময়ই ভুল লক্ষ্য *(Awareness)*

"সবকিছুই trade-off" এবং "context-dependent fit"—এই দুটি নীতিকে বাস্তব production সংখ্যার সাহায্যে দেখার সেরা উপায় হলো **Reliability (নির্ভরযোগ্যতা)** বনাম **Cost ও Velocity (গতি)**-এর টানাপোড়েন।

শুনতে খুব স্বাভাবিক মনে হয়: *"একজন ভালো backend engineer-এর লক্ষ্য তো হওয়া উচিত 100% uptime—অর্থাৎ সিস্টেম যেন এক সেকেন্ডও বন্ধ না হয়!"* কিন্তু Google-এর ***Site Reliability Engineering* (O'Reilly, 2016)** বইয়ের **Chapter 3 — *Embracing Risk*** চোখে আঙুল দিয়ে দেখিয়েছে: **প্রায় প্রতিটি সিস্টেমের জন্য 100% reliability আসলে ভুল লক্ষ্য!**

এর পেছনে দুটি কঠিন বাস্তব কারণ কাজ করে:

**প্রথম কারণ — প্রতিটি অতিরিক্ত "9"-এর খরচ সরলরৈখিক নয়, বহুগুণ (nonlinearly) বাড়ে:**
সার্ভিসের সচল থাকার হার (**availability**) প্রায়ই "নয়-এর সংখ্যা (nines)" দিয়ে প্রকাশ করা হয়। এক বছরে (৩৬৫ দিন = ৫,২৫,৬০০ মিনিট) বিভিন্ন লক্ষ্যমাত্রায় আনুমানিক অনুমোদিত downtime কতটুকু হয়, তা দেখুন:

| Availability Target | আনুমানিক Downtime / বছর | আনুমানিক Downtime / মাস | এর জন্য সাধারণত কী ধরনের অবকাঠামো ও শ্রম লাগে |
| :---: | :---: | :---: | :--- |
| **99.5%** | ~43.8 ঘণ্টা | ~3.65 ঘণ্টা | একক server + নিয়মিত backup + পরিকল্পিত maintenance window-তে restart। |
| **99.9%** *(three nines)* | ~8.76 ঘণ্টা | ~43.8 মিনিট | Managed DB, স্বয়ংক্রিয় health check ও restart, zero-downtime deploy, মৌলিক monitoring ও alert। |
| **99.99%** *(four nines)* | ~52.6 মিনিট | ~4.38 মিনিট | Multi-zone redundancy, automated failover, কঠোর canary testing, ২৪/৭ দ্রুত on-call response। |
| **99.999%** *(five nines)* | ~5.26 মিনিট | ~26 সেকেন্ড | Multi-region active-active architecture, সম্পূর্ণ automated self-healing, অত্যন্ত কঠোর release gate, বিশাল নিবেদিত SRE/ops টিম। |

Google SRE বইয়ের *Managing Risk* অংশটি এই অরৈখিক (nonlinear) ব্যয়কে দুটি সরাসরি মাত্রায় ভেঙে দেখায়—যেগুলোর সাথে আপনি ইতিমধ্যেই পরিচিত:
1. **Redundant machine/compute resources-এর খরচ:** অতিরিক্ত server, standby database replica, multi-region replication ও নিরবচ্ছিন্ন failover অবকাঠামোর বিপুল আর্থিক ও যান্ত্রিক ব্যয়—এক ধাপ reliability বাড়াতে খরচ আগের ধাপের চেয়ে কখনো ১০ গুণ থেকে ১০০ গুণ পর্যন্ত বাড়তে পারে।
2. **Opportunity Cost:** আপনার সেরা engineer-রা যখন নতুন customer feature, উন্নত product বা জরুরি business সমস্যা সমাধান না করে কেবল 99.9% থেকে 99.999%-এ ওঠার জটিল redundancy ও release-barrier বানাতে থাকেন—এবং প্রতিটি ছোট deploy-এর আগে ভয়ে দীর্ঘ অনুমোদন প্রক্রিয়া বসান—তখন পণ্যের অগ্রগতি ও উদ্ভাবনের গতি (**velocity**) থমকে যায়।

**দ্বিতীয় কারণ — ব্যবহারকারী সেই অতিরিক্ত নির্ভরযোগ্যতা টেরই পান না:**
ভেবে দেখুন, একজন shopper যখন আমাদের online store-এ ঢোকেন, তখন তাঁর মোবাইল নেটওয়ার্ক, Wi-Fi রাউটার, কিংবা ফোন অ্যাপের নিজের নির্ভরযোগ্যতাই হয়তো ৯৯% থেকে ৯৯.৯%-এর আশেপাশে। আপনার backend-কে ৯৯.৯% থেকে ৯৯.৯৯৯%-এ তুলতে বিপুল টাকা ও ৪ জন engineer-এর পুরো বছর খরচ করলেও shopper হয়তো কোনো পার্থক্যই অনুভব করবেন না—কারণ তাঁর মোবাইল সিগন্যালেই তার চেয়ে অনেক বেশি বিচ্ছিন্নতা ঘটে! অথচ ওই সময়টায় প্রতিদ্বন্দ্বী store যদি দ্রুত নতুন feature, ভালো search আর সহজ checkout নিয়ে আসে, তবে আপনার "অতি-নির্ভরযোগ্য" store গ্রাহক হারাবে। তাই Google SRE-র মূল নীতিটি চমৎকার: **কোনো সিস্টেমকে প্রয়োজনের মতো নির্ভরযোগ্য বানান, কিন্তু প্রয়োজনের চেয়ে বেশি নয়** *(reliable enough, but no more reliable than it needs to be)*।

এই প্রয়োজনটা মাপার ধারণাকেই Google SRE বলে **Risk Tolerance (ঝুঁকি সহনশীলতা)**। প্রতিটি সার্ভিসের risk tolerance তার ব্যবসায়িক প্রেক্ষাপটের ওপর নির্ভর করে:
- **Enterprise বনাম নতুন Consumer পণ্যের পার্থক্য:** Google SRE বইয়ে চমৎকার একটি তুলনা আছে—**Google Apps for Work** (Gmail, Docs, Drive)-এর মতো enterprise সেবার ওপর হাজার হাজার কোম্পানির দৈনন্দিন কাজ নির্ভর করে; তাই এর outage মানে বহু ব্যবসার ক্ষতি (ফলে এর ঝুঁকি সহনশীলতা কম এবং কঠোর availability লক্ষ্য দরকার)। অন্যদিকে ২০০৬ সালে Google যখন **YouTube** অধিগ্রহণ করে, তখন দ্রুত নতুন feature ও বৃদ্ধি ছিল বেঁচে থাকার মূল চাবিকাঠি—তাই সেখানে enterprise সেবার তুলনায় অপেক্ষাকৃত কম availability লক্ষ্য বেছে নেওয়া হয়েছিল যাতে feature-এর গতি না থামে।
- **ব্যর্থতার ধরন (Types of Failures) সমান নয়:** Risk tolerance মানে কেবল "কত মিনিট বন্ধ থাকল" তা নয়। ধরুন, একটি contact management অ্যাপে মাঝেমধ্যে profile photo load না হওয়া বনাম একজন user-এর ব্যক্তিগত contact তালিকা অন্য user-এর স্ক্রিনে ফাঁস হয়ে যাওয়া—সংখ্যার হিসাবে দুটোই "error" হতে পারে, কিন্তু প্রথমটি সাময়িক বিরক্তি আর দ্বিতীয়টি গ্রাহকের আস্থার চূড়ান্ত মৃত্যু! একইভাবে আমাদের store-এ campaign analytics dashboard ২ মিনিট না চলা মেনে নেওয়া যায়, কিন্তু promo code-এ ভুল করে গ্রাহকের কার্ড দুবার চার্জ হওয়া বা audit record হারিয়ে যাওয়া কখনোই মেনে নেওয়া যায় না।

এখন দেখুন, বাস্তব কোম্পানিতে এই reliability বনাম velocity-র দ্বন্দ্ব প্রায়ই টিমের ভেতর কলহ তৈরি করে। **Product টিম** মূল্যায়িত হয় দ্রুত নতুন feature ছাড়ার (velocity) ওপর; অন্যদিকে **Operations / SRE টিম** মূল্যায়িত হয় কোনো outage না ঘটার (reliability) ওপর—ফলে তারা প্রতিটি নতুন release আটকাতে চায়। দুই পক্ষের এই চিরন্তন টানাপোড়েনকে অফিস-রাজনীতি, ভয়, বা *"আশা করি ভাঙবে না"* (Google SRE-র অঘোষিত নীতি: ***"Hope is not a strategy"***)-এর বদলে একটি যৌথ, বস্তুনিষ্ঠ নিয়মে বাঁধার ধারণাগত হাতিয়ার হলো **Error Budget** *(Phase 1-এ কেবল **Awareness** স্তর)*।

ধারণাটি খুবই সহজবোধ্য: ব্যবসা ও engineering মিলে যদি স্থির করে যে আমাদের checkout service-এর লক্ষ্যমাত্রা **99.9% availability**, তবে বাকি **0.1%** হলো সেই সময়ের জন্য আমাদের অনুমোদিত ব্যর্থতার সীমা—অর্থাৎ **Error Budget**।
- যতক্ষণ budget-এ যথেষ্ট জায়গা আছে, টিম নির্দ্বিধায় নতুন feature deploy করতে, experiment চালাতে এবং দ্রুত এগোতে পারে।
- যদি কোনো বাজে deploy বা bug-এর কারণে error budget প্রায় ফুরিয়ে যায়, তবে দুই পক্ষের পূর্বসম্মত নিয়ম অনুযায়ী নতুন feature release সাময়িকভাবে থামিয়ে টিম reliability ও stability ঠিক করার কাজে মনোযোগ দেয়।

> **পরিধি-নোট (Phase Boundary):** Phase 1-এ আপনার জন্য কেবল এই *ধারণাগত সত্যটুকু* (**Awareness**) বোঝাই যথেষ্ট—কেন 100% reliability একটি বাজে constraint trade-off, কীভাবে service-ভেদে Risk Tolerance বদলায়, এবং Error Budget কীভাবে reliability ও velocity-র মধ্যে যুক্তিসঙ্গত ভারসাম্য আনে। **SLI/SLO সংজ্ঞা, error budget-এর গাণিতিক হিসাব এবং বাস্তব SRE নীতিমালা** আমরা **Phase 21: Reliability Engineering**-এ গভীরভাবে শিখব।

> **Pause & Check 2** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-04-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. একজন নতুন tech lead বললেন: *"একটা বড় পেমেন্ট কোম্পানি আমাদের মতোই Node.js ব্যবহার করে, আর তারা সবকিছু ১২টি microservice ও Kafka-তে ভেঙেছে। আমরা যেহেতু শুরু থেকেই 'best practice' মানতে চাই, তাই আমাদের ৪ জনের startup-এও ওটাই করব।"* Richards ও Ford-এর **First Law of Software Architecture** এবং **"least worst"** ধারণা ব্যবহার করে এই যুক্তির মূল দুর্বলতা ব্যাখ্যা করুন।
> 2. Google SRE-এর *Embracing Risk* দৃষ্টিভঙ্গি অনুযায়ী, কেন কোনো consumer backend-এর জন্য *"আমরা 100% uptime চাই"* বলাটা সাধারণত ব্যবসার জন্য ক্ষতিকর—এবং **Error Budget**-এর ধারণা কীভাবে product velocity ও reliability-র দ্বন্দ্বকে আবেগ থেকে যুক্তিতে নিয়ে আসে *(কোনো গাণিতিক সূত্র ছাড়া কেবল ধারণাগতভাবে ব্যাখ্যা করুন)*?

আমরা এখন দুটি বড় সত্য বুঝে গেছি: প্রথমত, ছয় মাত্রার constraint আমাদের সম্ভাব্য পথের দেয়াল তৈরি করে; দ্বিতীয়ত, সেই দেয়ালের ভেতর প্রতিটি বিকল্পই একটি trade-off—আমাদের খুঁজতে হয় প্রেক্ষাপট-উপযোগী *least worst* সমাধান।

কিন্তু এবার একটি অত্যন্ত ব্যবহারিক প্রশ্ন সামনে আসে: **বাস্তব কোনো engineering সিদ্ধান্তের মুখে পড়লে আমি কীভাবে দুই বা ততোধিক বিকল্পকে পক্ষপাতহীন ও পদ্ধতিগতভাবে তুলনা করব—আর ছোটখাটো প্রতিটি সিদ্ধান্তেই কি আমাকে দুদিন ধরে বিশ্লেষণ করতে হবে?**

---

## বিকল্পগুলোর সৎ ও পদ্ধতিগত তুলনা: Reversibility, Criteria এবং Decision Quality

প্রথমেই একটি জরুরি সময়-বাঁচানো প্রশ্ন পরিষ্কার করুন: **সব সিদ্ধান্ত কি সমান গভীর বিশ্লেষণ দাবি করে?**

না! যদি প্রতিটি সামান্য library বেছে নেওয়া বা internal helper-এর নামের জন্য আপনি ৩ দিন ধরে তুলনামূলক বিশ্লেষণ ও রিভিউ মিটিং করেন, তবে আপনি নিজেই ভয়াবহ opportunity cost তৈরি করবেন। কোন সিদ্ধান্তে কতটা সময়, প্রমাণ ও সতর্কতা ব্যয় করবেন, তা মাপার সবচেয়ে কার্যকর মাপকাঠি হলো **Reversibility (প্রত্যাবর্তনযোগ্যতা)**—অর্থাৎ সিদ্ধান্তটি ভুল প্রমাণিত হলে পেছনে ফিরতে কতটা কষ্ট ও ক্ষতি হবে।

Amazon-এর ২০১৫ সালের shareholder চিঠিতে Jeff Bezos সিদ্ধান্তকে দুটি বিখ্যাত শ্রেণিতে ভাগ করেছিলেন, যা আজ software engineering-এ বহুল ব্যবহৃত একটি মানসিক মডেল:

| ধরন | মূল বৈশিষ্ট্য | সিদ্ধান্ত নেওয়ার গতি ও কঠোরতা | বাস্তব Backend উদাহরণ |
| :--- | :--- | :--- | :--- |
| **Two-Way Door** *(Type 2 — Reversible)* | দরজা দিয়ে ঢোকার পর পছন্দ না হলে কম খরচে ও ছোট blast radius-এ (ক্ষতির পরিসরে) আবার দরজা খুলে পেছনে ফেরা যায়। | **দ্রুত সিদ্ধান্ত নিন**, অহেতুক কমিটি বা সপ্তাহব্যাপী বিশ্লেষণে আটকে থাকবেন না; প্রয়োজনে বদলে নিন। | একটি internal logging format library নির্বাচন, feature flag-এর পেছনে রাখা নতুন checkout UI পরীক্ষা, বা একটি internal helper function-এর গঠন। |
| **One-Way Door** *(Type 1 — Irreversible বা উচ্চ-খরচে Reversible)* | একবার ঢুকলে পেছনে ফেরা প্রায় অসম্ভব অথবা ভয়াবহ transaction, data-migration ও coordination cost লাগে। | **ধীরে, গভীরভাবে, প্রমাণসহ বিশ্লেষণ করুন**; অভিজ্ঞ সহকর্মীদের সমালোচনামূলক review নিন এবং ঝুঁকি আগে যাচাই করুন। | শত শত external partner-এর ব্যবহৃত public API contract (Hyrum's Law!), core financial ledger-এর database schema, ৩ জনের টিমের ওপর ৪টি microservice ও আলাদা DB চাপিয়ে দেওয়া, বা vendor-specific proprietary database-এ পুরো ব্যবসা আটকে দেওয়া। |

বাস্তব engineering-এ দুটি সূক্ষ্ম সতর্কতা মাথায় রাখবেন। প্রথমত, টিম বড় হলে প্রায়ই **Two-Way Door-কে One-Way Door বানিয়ে ফেলার** (অতিরিক্ত আমলাতন্ত্র) রোগ দেখা দেয়, আবার ছোট দ্রুতগতির টিমে ঠিক উল্টোটা—**One-Way Door-কে হালকা Two-Way Door ভেবে ঝাঁপিয়ে পড়ার** (যেমন না ভেবে public API বা database schema ছাড়া) দুর্ঘটনা ঘটে। দ্বিতীয়ত, Reversibility কোনো কেবল কালো-সাদা সুইচ নয়, এটি একটি **পরিসর (spectrum)**—আর দক্ষ engineer-রা অনেক সময় একটি সম্ভাব্য One-Way Door-কে পরিচ্ছন্ন boundary দিয়ে **আংশিকভাবে Two-Way Door-এ রূপান্তর** করেন। যেমন: আপনার পুরো কোডবেসের ৪০টি ফাইলে সরাসরি একটি নির্দিষ্ট external email provider-এর SDK ঢুকিয়ে দিলে provider বদলানো One-Way Door-এর মতো ব্যয়বহুল হয়ে যায়; কিন্তু সেটিকে একটি ছোট internal `EmailService` interface-এর পেছনে রাখলে ভবিষ্যতে provider বদলানোর transaction cost অনেক কমে আসে।

### সৎ Trade-off তুলনার শৃঙ্খলা (বনাম সাজানো Pros/Cons তালিকা)

এবার ধরুন, আপনার সামনে একটি সত্যিকারের গুরুত্বপূর্ণ (One-Way বা ব্যয়বহুল-to-reverse) সিদ্ধান্ত এসেছে—যেমন আমাদের online store-এর **Promo Redemption এবং Campaign Analytics** ঢেলে সাজানো।

এখানেই দ্বিতীয় বড় ভুল ধারণাটি বহু টিমকে ডুবিয়ে দেয়:

- **ভুল ধারণা: *"Trade-off বিশ্লেষণ মানে একটি Pros/Cons তালিকা, যেখানে আমার পছন্দের বিকল্পটিতে সব Pro লেখা থাকবে আর অন্য বিকল্পগুলোতে সব Con।"***
  সেটা engineering বিশ্লেষণ নয়—সেটা নিজের আগের পছন্দকে সাজিয়ে-গুছিয়ে বৈধতা দেওয়া (**confirmation bias** বা post-hoc rationalization)! একটি **সৎ (honest) trade-off বিশ্লেষণের** আসল পরীক্ষা হলো:
  1. আপনি কি **কমপক্ষে দুটি সত্যিকারের কার্যকর বিকল্প (genuinely viable alternatives)** পাশাপাশি রেখেছেন, নাকি একটি বাস্তব অপশনের পাশে একটি অযৌক্তিক কাকতাড়ুয়া (**strawman**) দাঁড় করিয়েছেন?
  2. আপনি কি আপনার **নিজের নির্বাচিত বিকল্পটির বাস্তব অসুবিধা ও ত্যাগ (what it sacrifices)** স্পষ্ট করে লিখেছেন?
  3. আপনি কি **প্রমাণিত তথ্য (facts)**, **অনুমান (assumptions)**, এবং **আনুমানিক হিসাব (estimates)** আলাদা করেছেন?
  4. আপনি কি স্পষ্ট করে বলেছেন **নির্বাচিত বিকল্পটি ঠিক কোন পরিস্থিতিতে ব্যর্থ হবে (failure modes)** এবং **ভবিষ্যতে কোন পরিমাপযোগ্য শর্তে (revisit triggers)** আপনাকে এই সিদ্ধান্ত বদলাতে হবে?

প্রতিটি গুরুত্বপূর্ণ বিকল্প তুলনার আগে নিজেকে এই আটটি প্রশ্ন দিয়ে যাচাই করুন:
1. আমরা ঠিক **কোন সমস্যাটি** সমাধান করছি?
2. এখানে প্রকৃতপক্ষে **কোন কোন Hard ও Soft constraint** প্রযোজ্য?
3. সেই constraint-গুলোর মধ্যে **কোন বিকল্পগুলো বাস্তবে কার্যকর**?
4. প্রতিটি বিকল্প **কী উন্নত করে**?
5. প্রতিটি বিকল্প **কী ত্যাগ বা বিসর্জন দেয়**?
6. আমরা **কোন অনুমানগুলোর** ওপর দাঁড়িয়ে আছি?
7. এই পছন্দের ফলে **ভবিষ্যতে কোন failure mode বা পরোক্ষ পরিণতি** দেখা দিতে পারে?
8. বর্তমান প্রেক্ষাপটে **কেন একটি নির্দিষ্ট বিকল্পই সবচেয়ে কম ক্ষতিকর (*least worst*)**—এবং কোন শর্তে তা পুনর্বিবেচনা করব?

### হাতে-কলমে বিশ্লেষণ: আমাদের Online Store-এর Promo Redemption ও Analytics সিদ্ধান্ত

এবার এই পুরো কাঠামোটি আমাদের শুরুর online store সমস্যাটিতে প্রয়োগ করে দেখুন। লক্ষ করুন, আমরা কেবল "খুব সরল" বনাম "খুব জটিল" দুটি চরম পথ রাখিনি—বরং মাঝামাঝি একটি আকর্ষণীয় বিকল্পও (**Option B: PostgreSQL + Redis**) রেখেছি, যাতে তুলনাটি বাস্তবসম্মত ও শিক্ষণীয় হয়!

- **সমস্যা:** ৫ সপ্তাহ পরের flash sale-এ (~50,000 orders/day অনুমান; ব্যস্ত সময়ে ধরে নেওয়া হচ্ছে ~50–100 checkout RPS—*estimate, এখনো load test দিয়ে যাচাইকৃত নয়*) `LAUNCH50`-এর মতো promo redemption-এ concurrency race ও negative-total bug দূর করা, ৫ বছরের জন্য অপরিবর্তনীয় redemption audit record রাখা, এবং Marketing-কে campaign-এর ব্যবহার দেখানো।
- **প্রযোজ্য Constraints:**
  - *Hard:* ৫ সপ্তাহের launch সময়সীমা (Schedule); ৩ জন Node.js/PostgreSQL engineer (Team); USD 800/month cloud বাজেট (Resource/Cost); ৫-বছরের zero-loss audit record (Regulatory/Compliance); `used_count <= max` ও `final_total >= 0` (Technical invariant)।
  - *Soft:* Campaign analytics-এর freshness (Marketing-এর সাথে কথা বলে নিশ্চিত হওয়া গেছে যে ২–৫ মিনিট পর্যন্ত বিলম্ব গ্রহণযোগ্য); checkout availability লক্ষ্য ~99.9% (99.999% নয়)।

এখন তিনটি বিকল্পকে একই সুনির্দিষ্ট মানদণ্ডের (explicit criteria) ওপর দাঁড় করিয়ে তুলনা করুন:

| মূল্যায়নের মানদণ্ড (Criteria) | Option A: একক Node.js + Managed PostgreSQL (Atomic Txn + Indexed Lookup + 2-min Summary Refresh) | Option B: একক Node.js + Managed PostgreSQL + Managed Redis (Promo Cache ও Counter-এর জন্য) | Option C: ৩টি Microservice + আলাদা DB + Redis + Kafka + Multi-Region (99.999%) |
| :--- | :--- | :--- | :--- |
| **৫ সপ্তাহে নিরাপদ Delivery ও টিম-দক্ষতা** | **উচ্চ:** টিমের পরিচিত stack; নতুন distributed অবকাঠামো শেখার খরচ নেই। | **মাঝারি:** Redis চালু করা সহজ, কিন্তু PostgreSQL ও Redis-এর মধ্যে state sync ও failure recovery ঠিক করতে বাড়তি সময় ও testing লাগবে। | **অসম্ভব (Hard Constraint ভঙ্গ):** ৩ জন অনভিজ্ঞ engineer-এর পক্ষে ৫ সপ্তাহে নিরাপদে তৈরি ও test করা অবাস্তব। |
| **অর্থ, Operational ও Cognitive Load** | **সর্বনিম্ন:** Managed PostgreSQL (~USD 150–250/mo) ও Node.js instances বাজেটের অনেক নিচে; ৩ জনের on-call সামলানো সহজ। | **মাঝারি:** বাড়তি Redis খরচ + একটি অতিরিক্ত stateful উপাদান monitor, fail-over ও debug করার ভার। | **অত্যধিক (Hard Constraint ভঙ্গ):** Cloud খরচ বাজেটের বহুগুণ; ৩ জনের টিমে ভয়াবহ operational বোঝা। |
| **Correctness ও `LAUNCH50` Invariant** | **শক্তিশালী:** একটিমাত্র PostgreSQL transaction-এ promo count update, order discount ও ৫-বছরের audit row একসাথে atomic-ভাবে সংরক্ষিত হয়। | **জটিলতা ও ঝুঁকি বাড়ে:** দুটি source of truth (Redis বনাম PostgreSQL) তৈরি হয়—Redis crash বা stale cache হলে over-redemption বা অসঙ্গত count হওয়ার ঝুঁকি থাকে। | **সবচেয়ে জটিল:** আলাদা DB-র মধ্যে distributed transaction ও eventual consistency সামলাতে গিয়ে সূক্ষ্ম data-drift bug-এর ঝুঁকি অনেক বেশি। |
| **Performance ও Analytics Freshness** | **যথেষ্ট (অনুমানসাপেক্ষ):** সঠিক index-সহ PostgreSQL 50–100 RPS সামলাতে পারে (load test দিয়ে যাচাই করতে হবে); analytics-এ ২ মিনিট বিলম্ব। | **দ্রুত Read:** প্রচুর repeated promo read-এ latency কমতে পারে; কিন্তু আমাদের বর্তমান load-এ PostgreSQL আসলেই bottleneck কি না তার কোনো প্রমাণ এখনো নেই! | **উচ্চ Theoretical Scale:** বড় scale-এ স্বাধীন scaling ও sub-second streaming সম্ভব—কিন্তু বর্তমান 50K orders/day-তে অপ্রয়োজনীয়। |
| **Reversibility (Door Type)** | **উচ্চ (অনেকটাই Two-Way):** পরে প্রমাণিত bottleneck এলে পরিচ্ছন্ন module boundary রেখে Redis বা read replica যোগ করা সহজ। | **মাঝারি:** Application কোড Redis-এর counter semantics-এ জড়িয়ে পড়লে পরে সরাতে মাঝারি transaction cost। | **অত্যন্ত নিম্ন (One-Way Door):** ৩টি service ও আলাদা DB একত্র করা বা ভুল boundary ঠিক করা ভয়াবহ ব্যয়বহুল। |
| **বাস্তব Failure Modes** | (১) অসতর্ক ভারী analytics query primary DB-র CPU/I/O দখল করে checkout ধীর করে দিতে পারে; (২) traffic ১০–২০× বাড়লে একক DB-র write limit পৌঁছাবে; (৩) ২ মিনিটের মধ্যে Marketing একদম তাৎক্ষণিক সংখ্যা পাবে না। | (১) Cache invalidation bug-এ expired promo চালু দেখাতে পারে; (২) Redis ও PostgreSQL-এর count অমিল হয়ে over-redemption বা false rejection; (৩) Redis outage-এ fallback path ঠিক না থাকলে checkout crash। | (১) Distributed partial failure ও network timeout; (২) Kafka consumer lag বা duplicate event; (৩) রাত ৩টায় অনভিজ্ঞ টিমের দীর্ঘ outage। |

একটু থেমে এই টেবিলটির দিকে নিবিড়ভাবে তাকান—বিশেষ করে **Option A বনাম Option B**-এর তুলনায়!

Option C যে Hard Constraint ভেঙে বাতিল হচ্ছে, তা সহজেই বোঝা যায়। কিন্তু প্রকৃত engineering বিচারবুদ্ধি ধরা পড়ে **Option B প্রত্যাখ্যান করে Option A বেছে নেওয়ার যুক্তিতে**। অনেকেই ভাবেন *"একটা Redis বসালে ক্ষতি কী, আরও fast হবে!"*—কিন্তু সৎ trade-off বিশ্লেষণ দেখিয়ে দিল:
- **Option B** দুটি জায়গায় state রাখার (dual-store synchronization) নতুন failure mode ও operational বোঝা আজকেই যোগ করছে—অথচ আমাদের হাতে এখনো কোনো প্রমাণ নেই যে সঠিক index-যুক্ত PostgreSQL আমাদের প্রত্যাশিত load সামলাতে পারবে না!
- **Option A** আমাদের বর্তমান ছয় মাত্রার constraint-এর ভেতর **সবচেয়ে কম ক্ষতিকর (*least worst*)** পথ—আর এর সবচেয়ে বড় সুবিধা হলো এটি **reversible**: ভবিষ্যতে প্রমাণিত চাপ এলে একটি read replica বা targeted cache যোগ করার দরজা খোলা থাকছে।
- কিন্তু খেয়াল করুন, আমরা **Option A-কেও "নিখুঁত" বলিনি!** আমরা সৎভাবে স্বীকার করেছি Option A কী ত্যাগ করছে (sub-second analytics), এর কোন failure mode ঠেকাতে হবে (analytics query-তে কঠোর `statement_timeout` ও pre-aggregated summary table ব্যবহার করে primary DB রক্ষা করা), কোন অনুমান যাচাই বাকি (flash-sale-এর আগে PostgreSQL-এ realistic load test চালানো), এবং **কোন Revisit Trigger** ঘটলে সিদ্ধান্ত বদলাতে হবে (যেমন load test বা production-এ promo validation-এর `p95` latency—অর্থাৎ ৯৫% request যে সময়ের মধ্যে শেষ হয়—150 ms ছাড়ালে, DB CPU ধারাবাহিকভাবে ৭০%-এর ওপরে থাকলে, বা ব্যবসা সত্যিকারের sub-second analytics-কে Hard Constraint ঘোষণা করলে)।

*(Winters et al., Ch. 1-এ এই একই trade-off যুক্তি আরেকটি পরিচিত backend প্রশ্নেও দেখা যায়—"Example: Deciding Between Time and Scale"—অর্থাৎ কোনো external library বা tool সরাসরি ব্যবহার করবেন, নাকি নিজের মতো করে **fork / reimplement** করবেন? নিজের তৈরি custom সংস্করণ আপনার সংকীর্ণ প্রয়োজনে দ্রুত ও নিয়ন্ত্রণযোগ্য হতে পারে, কিন্তু প্রতিটি টিম যদি নিজস্ব fork বানায় তবে security vulnerability ধরা পড়লে শত শত fork খুঁজে patch করা অসম্ভব হয়ে পড়ে। এখানেও কোনো এক-মাপের উত্তর নেই—প্রকল্পের lifespan, scope ও maintenance cost মিলিয়েই "least worst" পথ বাছতে হয়।)*

### Decision Quality বনাম Outcome Luck — এবং ভুল স্বীকারের সাহস

সবশেষে একটি মানসিক পরিপক্বতা আয়ত্ত করুন, যা একজন জুনিয়র কোডারকে পরিণত engineer থেকে আলাদা করে: **Decision Quality (সিদ্ধান্তের মান)** এবং **Outcome Luck (ফলাফলের ভাগ্য)** কখনোই এক জিনিস নয়।

ধরুন, আপনি ওপরের মতো করে সব constraint যাচাই করলেন, load test করে দেখলেন Option A অনায়াসে 150 RPS সামলাচ্ছে, সৎভাবে downsides নথিভুক্ত করলেন এবং Option A বেছে নিলেন। Flash sale দারুণ সফল হলো। কিন্তু চার মাস পর কোনো অপ্রত্যাশিত আন্তর্জাতিক তারকা হঠাৎ store-এর ভিডিও শেয়ার করায় এক রাতে traffic ৩০ গুণ হয়ে PostgreSQL চাপে পড়ে গেল। কেউ যদি তখন বলে *"দেখেছ! চার মাস আগে Option A বেছে নেওয়াটা ভুল সিদ্ধান্ত ছিল!"*—সেটি হলো **hindsight bias**। চার মাস আগে যে তথ্য, বাজেট ও সময়সীমা ছিল, তার ভিত্তিতে Option A-ই ছিল উচ্চ-মানের সিদ্ধান্ত (**high Decision Quality**)।

উল্টোদিকে, কেউ যদি কোনো constraint না দেখে, কোনো load test না করে কেবল *"বড় কোম্পানি করে"* বলে ৩ জন engineer দিয়ে ৫ সপ্তাহে Kafka ও microservices-এ ঝাঁপ দেয়, আর দৈবক্রমে flash sale-এ অর্ডার কম আসায় সিস্টেমটি কোনোমতে টিকে যায়—তবে ফলাফল ভালো দেখালেও ওটা ছিল খারাপ সিদ্ধান্ত ও নিছক **Outcome Luck**!

আর Titus Winters et al. (*Revisiting Decisions, Making Mistakes*) এখানে একটি অমূল্য শিক্ষা যোগ করেন: **সময়ের সাথে আপনার সিদ্ধান্তের ইনপুট (traffic, টিমের আকার, বাজেট, ব্যবসার লক্ষ্য) বদলাবেই।** তাই দীর্ঘমেয়াদি সিস্টেমে পুরোনো কোনো সিদ্ধান্ত আজ আর খাটছে না বা কোনো অনুমান ভুল ছিল—তা নতুন প্রমাণ দেখে স্বীকার করে সিদ্ধান্ত **পুনর্বিবেচনা (revisit)** করা ব্যর্থতা নয়; বরং তা একজন দায়িত্বশীল engineer-এর সবচেয়ে সম্মানজনক গুণ।

> **Pause & Check 3** *(শেষের Self-Check Guidance দেখার আগে `phases/phase-01/notes/block-04-notes.md` ফাইলে সংক্ষেপে নিজের উত্তর লিখুন।)*
> 1. নিচের দুটি backend সিদ্ধান্তের কোনটি মূলত **One-Way Door** এবং কোনটি **Two-Way Door**? প্রতিটি ক্ষেত্রে পেছনে ফিরতে কী ধরনের খরচ (বা কেন কম খরচ) হবে তা এক-দুই বাক্যে ব্যাখ্যা করুন:
>    - **(ক)** ২০০টি external partner কোম্পানির ব্যবহৃত public order-webhook API-র JSON payload কাঠামো ও authentication নিয়ম ঠিক করা।
>    - **(খ)** আপনার টিমের ভেতরে কেবল internal debugging-এর জন্য ব্যবহৃত Node.js logger package-এর output format বদলানো।
> 2. একজন সহকর্মী একটি বিকল্প তুলনায় লিখলেন: *"Option A (PostgreSQL): সহজ, সস্তা, নির্ভরযোগ্য, কোনো বড় অসুবিধা নেই। Option B (MongoDB): জটিল, অস্থির, টিম জানে না।"* — এই তুলনাটি কেন সৎ engineering trade-off বিশ্লেষণ হিসেবে ব্যর্থ? একে সৎ করতে আপনি কমপক্ষে কোন তিনটি জিনিস যোগ করতে বলবেন?
> 3. **Decision Quality** এবং **Outcome Luck**-এর পার্থক্য নিজের ভাষায় একটি ছোট উদাহরণ দিয়ে বুঝিয়ে বলুন।

এবার ১৮ মাস সামনে এগিয়ে যান।

ধরুন, আমাদের online store দারুণ সফল হয়েছে। টিম এখন ৩ জন থেকে বেড়ে ১২ জন engineer-এ দাঁড়িয়েছে। নতুন যোগ দেওয়া একজন সিনিয়র engineer codebase ঘুরে দেখে অবাক হয়ে গেলেন: *"আরে, এরা checkout আর analytics একই PostgreSQL-এ রেখেছে, আর analytics-এ ২ মিনিট বিলম্ব! নিশ্চয়ই আগের ডেভেলপাররা event streaming জানতেন না!"*

তাঁর সামনে তখন মাত্র দুটি পথ—আর দুটিই সমান ভয়ংকর: হয় তিনি অজানা ভয়ে পুরোনো সবকিছু অন্ধভাবে মেনে নেবেন (যদিও এখন টিম বড় হয়েছে এবং traffic আসলেই Revisit Trigger ছাড়িয়ে গেছে!), নয়তো তিনি না জেনেই সেটি ভেঙে নতুন কিছু বসিয়ে দেবেন (এবং অজান্তেই ৫-বছরের atomic compliance audit invariant ভেঙে ফেলবেন!)।

**আপনার আজকের চমৎকার constraint বিশ্লেষণ আর trade-off যুক্তি যেন ছয় মাস পর Slack-এর হাজার মেসেজের নিচে চাপা পড়ে হারিয়ে না যায়—ভবিষ্যতের engineer-দের জন্য সেই সিদ্ধান্তের স্মৃতি ও জবাবদিহি ধরে রাখার উপায় কী?**

---

## ভবিষ্যতের টিমের সাথে সংলাপ: Architecture Decision Records (ADRs)

২০১১ সালের ১৫ নভেম্বর সফটওয়্যার স্থপতি **Michael Nygard** তাঁর বিখ্যাত প্রবন্ধ ***"Documenting Architecture Decisions"*** (Cognitect Blog)-এ ঠিক এই সমস্যাটিরই একটি অসাধারণ, হালকা ও স্থায়ী সমাধান তুলে ধরেন।

Nygard দুটি তিক্ত বাস্তবতা থেকে শুরু করেছিলেন:
1. **বিশাল architecture নথি কেউ পড়ে না, কেউ হালনাগাদও রাখে না।** অনেক প্রকল্পে কোডের চেয়ে বড় specification ফাইল জমে থাকে, যা ছয় মাসেই বাস্তবতা থেকে বিচ্ছিন্ন হয়ে অকেজো হয়ে পড়ে।
2. **অথচ কোনো রেকর্ড না থাকলে নতুন engineer দুটি ফাঁদে পড়েন:** **Blindly accept it** *(প্রেক্ষাপট বদলে গেলেও না বুঝে পুরোনো সিদ্ধান্ত মেনে নেওয়া—যাতে টিম পরিবর্তন করতে ভয় পায় এবং প্রকল্প নিজের ভারে ভেঙে পড়ে)*, অথবা **Blindly change it** *(পুরোনো সিদ্ধান্তটি কোন লুকানো constraint বা untested non-functional requirement রক্ষা করছিল তা না জেনেই সেটি উল্টে দিয়ে সিস্টেমের ক্ষতি করা)*।

এই দুই চরম ফাঁদ এড়াতেই Nygard প্রস্তাব করেন **Architecture Decision Record (ADR)**—একটি সংক্ষিপ্ত (১ থেকে ২ পৃষ্ঠার), সরল text বা Markdown নথি, যা কোডের সাথেই Git repository-তে (যেমন `doc/arch/adr-NNN.md` বা `docs/adr/`) version-controlled অবস্থায় থাকে।

### কখন ADR লিখবেন, আর কীভাবে তা ইতিহাস রক্ষা করে?

প্রতিটি তুচ্ছ কোড পরিবর্তনের জন্য ADR লিখতে বসলে তা আবার সেই অপ্রয়োজনীয় কাগজপত্রের বোঝায় পরিণত হবে। Nygard-এর সুস্পষ্ট সীমারেখা হলো: **শুধুমাত্র "architecturally significant" সিদ্ধান্তগুলোর জন্য ADR লিখুন**—অর্থাৎ যেগুলো সিস্টেমের structure, non-functional characteristics (reliability, security, performance, compliance), মূল dependencies, public interfaces, অথবা প্রধান construction technique-কে প্রভাবিত করে (বিশেষত One-Way Door বা ব্যয়বহুল-to-reverse সিদ্ধান্ত)।

এর সাথে দুটি কঠোর নিয়ম ADR-কে জীবন্ত ও বিশ্বাসযোগ্য রাখে:
- **ক্রমিক ও অপরিবর্তনীয় সংখ্যা (Sequential, monotonic numbering):** `ADR-001`, `ADR-002`, `ADR-003`... নম্বর কখনো পুনর্ব্যবহার (reuse) করা হয় না।
- **অপরিবর্তনীয় ইতিহাস (Immutable history):** এক বছর পর প্রেক্ষাপট বদলে যদি কোনো পুরোনো সিদ্ধান্ত উল্টে দিতে হয়, তবে পুরোনো ADR ফাইলটি **মুছে ফেলা (delete) বা ইতিহাস বদলে নতুন করে লেখা হয় না!** বরং একটি নতুন ADR (ধরুন `ADR-014`) লিখে নতুন সিদ্ধান্ত ব্যাখ্যা করা হয়, আর পুরোনো `ADR-001`-এর Status বদলে লেখা হয় `Superseded by ADR-014`। কেন? কারণ ভবিষ্যতের টিমের জানা দরকার যে একসময় এটিই সিদ্ধান্ত ছিল এবং ঠিক কোন প্রেক্ষাপটের পরিবর্তনে আমরা সেখান থেকে সরে এসেছি।

### একটি কার্যকর ADR-এর গঠন (Structure)

Nygard-এর মূল ২০১১ প্রবন্ধে পাঁচটি মৌলিক অংশ ছিল: **Title**, **Context**, **Decision**, **Status**, এবং **Consequences** *(মজার ব্যাপার হলো, Nygard তাঁর পুরো ব্লগপোস্টটিই নিজে একটি ADR-এর ফরম্যাটে লিখেছিলেন!)*। আধুনিক engineering চর্চায়—এবং আমাদের এই কোর্সের অনুমোদিত কাঠামোয়—এর সাথে একটি স্বতন্ত্র **Alternatives Considered** অংশ যুক্ত করা হয় (এবং Consequences-এর ভেতর বা শেষে **Revisit Triggers** রাখা হয়), যাতে এক বছর পর কেউ ইতিমধ্যে বাতিল করা কোনো বিকল্প নতুন করে তুললে তার কারণ হাতের কাছেই পাওয়া যায়:

1. **Title:** সংক্ষিপ্ত noun phrase যা সিদ্ধান্তটিকে সরাসরি প্রকাশ করে—যেমন `ADR-001: Keep Promo Redemption and Campaign Analytics in Single Node.js Service and PostgreSQL` *(কেবল অস্পষ্ট `"Database choice"` বা `"Analytics"` নয়)*।
2. **Status:** সিদ্ধান্তের বর্তমান জীবনচক্রের অবস্থা—`Proposed` (আলোচনাধীন), `Accepted` (টিম কর্তৃক গৃহীত), অথবা পরে বদলালে `Deprecated` / `Superseded by ADR-NNN`।
3. **Context:** সেই মুহূর্তে কাজ করা সমস্ত বাস্তব শক্তি ও সীমাবদ্ধতা (technological, business, schedule, team, regulatory, operational forces এবং তাদের মধ্যকার টানাপোড়েন)। Nygard-এর একটি সোনালি নিয়ম: **Context অংশের ভাষা হতে হবে value-neutral (নিরপেক্ষ ও তথ্যভিত্তিক)**—এখানে আগেভাগে নিজের পছন্দের পক্ষে ওকালতি না করে কেবল বাস্তব তথ্য ও সীমাবদ্ধতাগুলো তুলে ধরুন।
4. **Decision:** সেই শক্তিগুলোর জবাবে টিম ঠিক কী করছে—**পূর্ণ বাক্যে এবং active voice-এ** (`"We will ..."`), যাতে কোনো ধোঁয়াশা না থাকে।
5. **Alternatives Considered:** কমপক্ষে দুটি বাস্তব বিকল্পের সংক্ষিপ্ত ও সৎ মূল্যায়ন—প্রতিটি বিকল্প কী দিত এবং কোন constraint বা trade-off-এর কারণে সেটি নেওয়া হলো না।
6. **Consequences:** সিদ্ধান্তটি নেওয়ার ফলে নতুন বাস্তবতা কী দাঁড়াবে। এখানে কেবল সুবিধা (positive) নয়, বরং **positive, negative, এবং neutral—সব পরিণতিই সততার সাথে লিখতে হবে**, যার মধ্যে থাকবে পরিচিত failure modes, মূল assumptions এবং কোন পরিমাপযোগ্য **Revisit Triggers** ঘটলে এই ADR-টিকে supersede করতে হবে। Nygard সুন্দর একটি সূত্র দেখিয়েছেন: ***"একটি ADR-এর Consequences প্রায়ই পরবর্তী ADR-এর Context হয়ে দাঁড়ায়।"***

*(নোট: Block 2-এর **Design Doc** আর Block 4-এর **ADR**-কে গুলিয়ে ফেলবেন না। Design Doc হলো কোনো মাঝারি বা বড় প্রকল্প শুরু করার আগে পুরো problem framing, Goals, Non-Goals এবং বিস্তারিত নকশা নিয়ে টিমের আলোচনার কর্মপত্র; আর ADR হলো একটিমাত্র architecturally significant সিদ্ধান্ত, তার trade-off এবং ঐতিহাসিক প্রেক্ষাপটের সংক্ষিপ্ত, স্থায়ী রেকর্ড। একটি বড় Design Doc থেকে সহজেই দুই-তিনটি ADR তৈরি হতে পারে।)*

### একটি বাস্তব ১-পৃষ্ঠার ADR নমুনা (English Technical Writing Model)

পেশাদার backend দলগুলোতে ADR সাধারণত সংক্ষিপ্ত, স্পষ্ট ইংরেজি গদ্যে লেখা হয় (*Nygard মনে করিয়ে দেন: এটি ভবিষ্যতের একজন developer-এর সাথে সংলাপ, তাই অস্পষ্ট এক-শব্দের bullet fragment নয়, স্পষ্ট পূর্ণ বাক্য ব্যবহার করুন*)। আমাদের online store-এর সিদ্ধান্তটির জন্য একটি পূর্ণাঙ্গ নমুনা ADR দেখুন:

```markdown
# ADR-001: Keep Promo Redemption and Campaign Analytics in Single Node.js Service and PostgreSQL

- **Date:** 2026-11-25
- **Status:** Accepted

## Context

Our online store currently processes ~15,000 orders/day on a single Node.js backend service backed by managed PostgreSQL. A major seasonal flash sale is scheduled in 5 weeks, with business estimates projecting up to ~50,000 orders/day (estimated peak 50–100 checkout RPS; not yet verified by load testing).

A recent viral campaign (`LAUNCH50`) exposed concurrency over-redemptions and negative order-total bugs. In addition, a new payment-partner compliance rule requires durable, zero-loss retention of every promo redemption and discount audit record for 5 years. Marketing also requested a campaign analytics dashboard to monitor promo usage.

Key active constraints:
- **Schedule (Hard):** Changes must be verified and live before the flash sale in 5 weeks.
- **Team (Hard):** 3 backend engineers experienced in Node.js and PostgreSQL, with no production experience operating Kafka, Redis clusters, or multi-region replication.
- **Budget (Hard):** Cloud infrastructure spend is capped at USD 800/month until the next revenue milestone.
- **Compliance (Hard):** 5-year durable audit log with zero silent drops of redemption records.
- **Analytics Freshness & Availability (Soft):** Marketing confirmed 2–5 minute dashboard lag is acceptable; checkout targets ~99.9% availability (~43.8 min/month downtime tolerance).

## Decision

We will keep promo redemption, 5-year audit recording, and campaign analytics within our existing single Node.js service and managed PostgreSQL database. We will enforce promo invariants (`final_total >= 0`, `used_count <= max_total_redemptions`) and write redemption audit records inside the same atomic PostgreSQL transaction, add targeted database indexes for promo validation, and serve Marketing analytics from a scheduled summary table refreshed every 2 minutes with strict query timeouts.

## Alternatives Considered

1. **Add Managed Redis for Promo Lookups and Usage Counters:**
   Redis would reduce repeated promo read latency, but introducing a second stateful store creates cache-invalidation and dual-source-of-truth failure modes that directly threaten our redemption-count invariant and compliance audit accuracy. We currently have no empirical evidence that indexed PostgreSQL fails to meet checkout latency targets, so taking on Redis operational overhead before load testing is premature.
2. **Split into 3 Microservices (Checkout, Promo, Analytics) + Kafka + Multi-Region 99.999% Setup:**
   Independent services and event streaming would allow sub-second analytics and independent scaling at much higher traffic. However, this option violates our 5-week schedule, USD 800/month cloud budget, and 3-engineer operational capacity, while introducing distributed partial-failure modes and a hard-to-reverse (one-way door) service boundary split.

## Consequences

### Positive
- Meets the 5-week deadline and USD 800/month budget while staying within the team's Node.js and PostgreSQL expertise.
- Guarantees atomic consistency between order discounts, promo usage limits, and 5-year compliance audit rows without distributed transactions.
- Keeps the architecture largely reversible (two-way door): clean internal module boundaries allow adding a read replica, cache, or async pipeline later if measured bottlenecks emerge.

### Negative
- Marketing analytics reflects up to 2 minutes of staleness rather than sub-second real-time counts.
- Checkout and analytics share the same PostgreSQL primary instance; an unguarded or poorly indexed analytics query could contend with checkout CPU/I/O if query timeouts and summary-table discipline are not enforced.
- Single-database write throughput imposes an eventual vertical scaling ceiling if order volume grows by an order of magnitude.

### Neutral
- Requires running a realistic pre-launch load test against PostgreSQL (currently an unverified estimate) before the flash sale.

## Assumptions & Revisit Triggers

- **Key Assumptions:** Peak checkout traffic stays near the estimated 50–100 RPS; promo validation with indexing meets `p95 <= 150 ms` under load testing.
- **Revisit Triggers:** We will supersede this ADR if (a) pre-launch load tests or production telemetry show promo validation `p95 > 150 ms` after proper indexing, (b) analytics refresh queries measurably degrade checkout latency or push primary DB CPU above 70% sustained, or (c) business requirements change to legally or contractually mandate sub-second analytics or availability above 99.95%.
```

খেয়াল করুন, এই এক পৃষ্ঠার নথিটি কেবল তথ্য লিখে রাখার কাজ করেনি—এটি সরাসরি **engineering accountability (Outcome 10)**-কেও রূপ দিয়েছে। যখন আপনি সচেতনভাবে লেখেন *কোন constraint আমরা মেনেছি, কোন ঝুঁকিগুলো জেনে-শুনে গ্রহণ করেছি, কোন অনুমান এখনো যাচাই বাকি, এবং কোন সংকেত পেলে আমরা নিজেরাই এই সিদ্ধান্ত বদলাব*—তখন আপনি আর *"সবাই এটাই করে"* বা *"AI এটা সাজেস্ট করেছিল"* অজুহাতের আড়ালে লুকান না; আপনি একজন প্রকৃত engineer-এর মতো আপনার সিদ্ধান্ত ও তার পরিণতির মালিকানা নেন।

---

## সব সূত্র একসাথে বাঁধা — এবং Block 5-এর দিকে যাত্রা

পেছনে তাকিয়ে দেখুন, শুরুর সেই দ্বিধা (*"আকর্ষণীয় আধুনিক stack বনাম সীমিত সম্পদের বাস্তবতা"*) থেকে আমরা একটি পূর্ণাঙ্গ, যুক্তিনির্ভর engineering সিদ্ধান্ত-কাঠামোতে পৌঁছে গেছি:

1. **Constraints প্রথমে সমাধান-ক্ষেত্রের দেয়াল আঁকে:** Technical, Resource/Cost (Winters et al.-এর ছয়টি cost dimension-সহ), Schedule, Team/Organizational, Regulatory/Compliance, এবং Operational—এই ছয় মাত্রায় **Hard বনাম Soft constraint** আলাদা না করে কোনো প্রযুক্তি বাছাই করলে সমাধানটি বাস্তবে টেকে না।
2. **সবকিছুই Trade-off (First Law):** সর্বজনীন "best practice" বা "100% reliability" বলে কিছু নেই। নির্দিষ্ট প্রেক্ষাপটের জন্য **least worst** সমন্বয় খুঁজতে হয়—যেখানে Google SRE-র nonlinear reliability cost, **Risk Tolerance**, এবং ধারণাগত **Error Budget** (`Awareness`) দেখিয়ে দেয় কেন অতিরিক্ত নির্ভরযোগ্যতা কখনো কখনো ব্যবসারই ক্ষতি করে।
3. **সৎ তুলনা ও Reversibility:** **One-Way Door** বনাম **Two-Way Door** অনুযায়ী বিশ্লেষণের গভীরতা ঠিক করতে হয়; প্রতিটি তুলনায় নির্বাচিত বিকল্পের বাস্তব ত্যাগ, failure modes, assumptions এবং **Revisit Triggers** স্পষ্টভাবে লিখতে হয়; এবং **Decision Quality**-কে কখনো **Outcome Luck**-এর সাথে গুলিয়ে ফেলা যাবে না।
4. **ADR ভবিষ্যতের স্মৃতি ও দায়িত্ব ধরে রাখে:** Michael Nygard-এর সংক্ষিপ্ত, version-controlled, অপরিবর্তনীয় ইতিহাসযুক্ত **ADR** ভবিষ্যতের টিমকে blind acceptance ও blind reversal—দুই ফাঁদ থেকেই বাঁচায়।

কিন্তু এখন ওপরের `ADR-001`-এর **Assumptions & Revisit Triggers** অংশটির দিকে আরেকবার তাকান—ওখানেই লুকিয়ে আছে আমাদের পরবর্তী অনিবার্য প্রশ্ন!

আমরা `ADR-001`-এ লিখেছি: *"অনুমান করা হচ্ছে indexed PostgreSQL 50–100 RPS-এ 150 ms-এর নিচে উত্তর দেবে—যদি load test বা production telemetry দেখায় যে তা পারছে না, তবে আমরা সিদ্ধান্ত পুনর্বিবেচনা করব।"* এবার ভাবুন, টিমের একজন বললেন *"আমার অভিজ্ঞতা বলছে PostgreSQL-এ এটা ধীর হবেই, Redis লাগবে!"*, আরেকজন বললেন *"আমি একটা ব্লগ পোস্টে দেখেছি Redis ৩ গুণ দ্রুত!"*, আর তৃতীয়জন production ধীর হওয়া মাত্রই কোনো পরিমাপ না করে server restart করে connection pool দ্বিগুণ করে দিলেন।

**একজন backend engineer কীভাবে মতামত (opinion), জনপ্রিয় দাবি (hype), ও অন্ধ অনুমান থেকে প্রকৃত প্রমাণ (empirical evidence) আলাদা করবেন—এবং বৈজ্ঞানিক পদ্ধতিতে (scientific method) পরিমাপ ও পরীক্ষা চালিয়ে সিস্টেমকে বুঝবেন?**

ঠিক এই প্রশ্নটিই আমাদের নিয়ে যাচ্ছে **Block 5: Evidence-Based Engineering**-এ।

---

## Study Plan, Learning Objectives & Mandatory Reading

> **Continuity & Ownership Notes:** আপনার সব inline check, practical exercise এবং (যদি করতে চান) optional closed-book assessment-এর উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-04-notes.md` ফাইলে লিখুন। AI-এর প্রস্তুতির সময় আপনার study hours-এ গণনা হয় না।
>
> **Progression & Verification Policy:** আনুষ্ঠানিক assessment **ঐচ্ছিক (optional)** এবং পরের block-এ এগোনোর নিয়ন্ত্রণ আপনার হাতে। আপনি চাইলে assessment সম্পন্ন করে মূল্যায়নের জন্য ChatGPT-র কাছে শেয়ার করতে পারেন (`Mastery verified`-এর প্রমাণ হিসেবে), নিজে যথেষ্ট শিখেছেন মনে করলে assessment বাদ দিয়ে এগিয়ে যেতে পারেন (`Studied — mastery not formally verified`), অথবা assessment ছাড়াই যেকোনো অস্পষ্ট ধারণা নিয়ে প্রশ্ন ও আলোচনা করতে পারেন। কোনো গুরুত্বপূর্ণ ফাঁক থাকলে ChatGPT বাড়তি যাচাইয়ের পরামর্শ দিতে পারেন, তবে assessment সম্পন্ন করা বা ChatGPT-র অনুমোদন পরের block শুরু করার বাধ্যতামূলক শর্ত নয়। Assessment বাদ দিলে তা স্বয়ংক্রিয়ভাবে `Mastery verified` বলে গণ্য হয় না।

### Study Time Reference

*কার্যক্রমভিত্তিক বাস্তবসম্মত সময়-বিভাজনের জন্য এই নথির শুরুর **Study Time Guide** দেখুন। মূল roadmap-এ Block 4-এর ৩–৪ ঘণ্টার (গবেষণা ডসিয়ারে ~৩.৫ ঘণ্টার) বরাদ্দটি একটি আনুমানিক পরিকল্পনা-রেফারেন্স; কোনো কার্যক্রমেরই কঠোর সময়সীমা নেই।*

### Block 4 Learning Objectives *(Primary: Outcomes 5, 6; Supporting: Outcome 10)*

- **LO 4.1 (`Working Understanding`):** বাস্তব **engineering constraints** চিহ্নিত ও শ্রেণিবদ্ধ করা—**technical**, **resource/cost**, **schedule**, **team/organizational**, **regulatory/compliance**, এবং **operational** মাত্রাগুলোতে (Hard বনাম Soft constraint ও বহুমাত্রিক cost-সহ)।
- **LO 4.2 (`Working Understanding`):** Mark Richards ও Neal Ford-এর **First Law of Software Architecture** (*"Everything in software architecture is a trade-off"*) এবং এর অনুসিদ্ধান্ত (সর্বজনীন "best practices"-এর বদলে নির্দিষ্ট প্রেক্ষাপটের জন্য *"least worst"* নকশা খোঁজা) প্রয়োগ করা—পাশাপাশি Google SRE-এর *Embracing Risk* কাঠামো থেকে **Risk Tolerance** ও **Error Budget**-এর ধারণাগত ভিত্তি (`Awareness`) বোঝা।
- **LO 4.3 (`Practical Competence`):** স্পষ্ট মানদণ্ডের বিপরীতে কমপক্ষে দুটি কার্যকর কারিগরি বিকল্পকে পদ্ধতিগতভাবে তুলনা করা—প্রতিটি বিকল্প কী দেয়, কী ত্যাগ করে, **reversibility** (one-way বনাম two-way door), assumptions, failure modes, এবং revisit triggers উন্মোচন করে।
- **LO 4.4 (`Practical Competence`):** **Context**, **Decision**, **Alternatives Considered**, **Consequences**, এবং **Status** অন্তর্ভুক্ত করে একটি সুগঠিত **Architecture Decision Record (ADR)** রচনা করা।

### Complementary Authoritative Reading

- **Mandatory 1 (~10–15 min):** Michael Nygard, *"Documenting Architecture Decisions"* (Cognitect Blog, Nov 15, 2011) — [cognitect.com/blog/2011/11/15/documenting-architecture-decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)।
  - **কী পড়বেন:** পুরো সংক্ষিপ্ত প্রবন্ধটি (~১,০০০ শব্দ) পড়ুন—*Context*, *Decision*, *Status*, *Consequences*, এবং *Experience Report*। বিশেষভাবে লক্ষ করুন blind acceptance বনাম blind reversal-এর যুক্তি, value-neutral Context, active-voice Decision, positive/negative/neutral Consequences, এবং immutable history (superseded ADR)।
  - **কী বাদ দেবেন:** নিচের *More Reading* অংশের বাহ্যিক লিংকগুলো Phase 1-এ বাদ দিতে পারেন।
  - **কোন Objective-কে সমর্থন করে:** `LO 4.4` (এবং সৎ trade-off নথিভুক্তির মাধ্যমে `LO 4.3`)।
- **Mandatory 2 (~15–20 min — Awareness scope):** Betsy Beyer, Chris Jones, Jennifer Petoff, Niall Richard Murphy (eds.), *Site Reliability Engineering* (O'Reilly, 2016) — [Chapter 3 — Embracing Risk](https://sre.google/sre-book/embracing-risk/)।
  - **কী পড়বেন:** (১) *Managing Risk* (সম্পূর্ণ), (২) *Risk Tolerance of Services*-এর ধারণাগত কাঠামো (বিশেষত *Identifying the Risk Tolerance of Consumer Services* — availability target, types of failures, cost — এবং consumer বনাম infrastructure risk tolerance-এর মূল পার্থক্য), এবং (৩) *Motivation for Error Budgets* (সম্পূর্ণ)।
  - **কী বাদ দেবেন:** *Measuring Service Risk* (availability সূত্র ও গণনা), *Risk Tolerance of Infrastructure Services*-এর ভেতরের গভীর Bigtable/Spanner অবকাঠামো বিবরণ, *Forming Your Error Budget*, এবং *Benefits* অংশগুলো—বিস্তারিত SLI/SLO ও error-budget engineering আমরা **Phase 21 (Reliability Engineering)**-এ শিখব।
  - **কোন Objective-কে সমর্থন করে:** `LO 4.1`, `LO 4.2` (Error Budget ও Risk Tolerance-এর জন্য কেবল `Awareness` স্তর)।
- **Mandatory 3 (~15–20 min — নির্বাচিত অংশের গভীর পুনর্পাঠ):** Titus Winters, Tom Manshreck, Hyrum Wright, *Software Engineering at Google* (O'Reilly, 2020; free HTML edition) — [Chapter 1 — Trade-offs and Costs](https://abseil.io/resources/swe-book/html/ch01.html)।
  - **কী পড়বেন:** কেবল তিনটি উপ-অধ্যায় পড়ুন (Block 1-এ ইতিমধ্যে পড়ে থাকলে এবার trade-off সিদ্ধান্তের লেন্স দিয়ে মনোযোগসহ আবার দেখুন): *Trade-offs and Costs* (*Example: Markers*-সহ), *Inputs to Decision Making* (*Example: Distributed Builds* ও *Example: Deciding Between Time and Scale*-সহ), এবং *Revisiting Decisions, Making Mistakes*।
  - **কী বাদ দেবেন:** *Time and Change*, *Scale and Efficiency*, *Software Engineering Versus Programming*, *Conclusion*, এবং *TL;DRs* (এগুলো Block 1-এর আওতায় ছিল)।
  - **কোন Objective-কে সমর্থন করে:** `LO 4.1`, `LO 4.2`, `LO 4.3`।
- **Optional Supplementary (মূল ৩–৪ ঘণ্টার বাজেটের বাইরে, বাণিজ্যিক বই):** Mark Richards & Neal Ford, *Fundamentals of Software Architecture* (2nd ed., O'Reilly, 2025; 1st ed., 2020) — **First Law of Software Architecture** (*"Everything in software architecture is a trade-off"*) এবং *"least worst architecture"* ধারণার মূল উৎস *(বাণিজ্যিক বই হওয়ায় এর ভেতরের সুনির্দিষ্ট ২য় সংস্করণের অধ্যায়/পৃষ্ঠা নম্বর এখানে যাচাইকৃত নয়; কেনার কোনো বাধ্যবাধকতা নেই)*।

---

## Practical Reasoning Exercises (~55–70 min)

> **Ownership Rule:** সব উত্তর স্বাধীনভাবে `phases/phase-01/notes/block-04-notes.md` ফাইলে লিখুন। AI-কে দিয়ে উত্তর বা ADR খসড়া করাবেন না। লক্ষ্য হলো ওপরের শেখা mental model-গুলোকে দুটি নতুন backend দৃশ্যপটে স্বাধীনভাবে প্রয়োগ করা।

### Exercise 1: Constraint-Driven ADR — Startup SaaS Backend (~35–45 min)

**দৃশ্যপট (Scenario):**
মাত্র **২ জন backend engineer**-এর একটি নতুন startup একটি B2B **SaaS** *(Software as a Service — ইন্টারনেটের মাধ্যমে ব্যবহৃত subscription-ভিত্তিক সফটওয়্যার)* backend (যেমন ছোট ব্যবসাগুলোর জন্য invoice ও client-management প্ল্যাটফর্ম) চালু করতে যাচ্ছে। পরবর্তী ১২ মাসের জন্য প্রত্যাশিত ব্যবহারকারীর সংখ্যা প্রায় **৫০০ daily active users (DAU)**। বিনিয়োগকারী ও প্রাথমিক গ্রাহকদের সাথে চুক্তির কারণে **৬ সপ্তাহের মধ্যে** পণ্যের প্রথম সংস্করণ চালু করার কঠোর সময়সীমা (**launch constraint**) রয়েছে। দুইজন engineer-ই Node.js এবং PostgreSQL-এ স্বাচ্ছন্দ্য, কিন্তু তাঁদের কেউ আগে production-এ distributed message broker বা multi-service orchestration চালাননি। প্রাথমিক পর্যায়ে cloud বাজেট সীমিত, এবং গ্রাহকদের আর্থিক invoice ডেটা হারালে বা ভুল হিসাব হলে ব্যবসায়িক আস্থা মারাত্মকভাবে ক্ষতিগ্রস্ত হবে।

ডিজাইন বৈঠকে দুটি বিকল্প সামনে এসেছে:
- **Option A:** একটিমাত্র **Node.js service** (ভেতরে পরিচ্ছন্ন modular কাঠামোসহ) + একটিমাত্র **managed PostgreSQL database**।
- **Option B:** **৪টি আলাদা microservice** (Auth, Clients, Invoices, Notifications) + প্রতিটি service-এর জন্য আলাদা database + service-গুলোর মধ্যে যোগাযোগের জন্য একটি **message broker** *(যেমন RabbitMQ বা Kafka-র মতো asynchronous বার্তা পরিবহন ব্যবস্থা—বিস্তারিত Phase 17-এ)*।

**কাজ (Tasks):**
1. **Constraint Inventory (`LO 4.1`):** এই দৃশ্যপটের প্রকাশ্য ও লুকানো constraint-গুলোকে ছয়টি মাত্রায় (Technical, Resource/Cost, Schedule, Team/Organizational, Regulatory/Compliance বা Data-Integrity প্রত্যাশা, এবং Operational) শ্রেণিবদ্ধ করুন—এবং কোনটি **Hard** আর কোনটি **Soft** constraint তা চিহ্নিত করুন।
2. **Structured Trade-off & Reversibility Comparison (`LO 4.3`):** Option A এবং Option B-কে স্পষ্ট মানদণ্ডের বিপরীতে তুলনা করুন। প্রতিটি বিকল্পের ক্ষেত্রে (ক) কী লাভ হচ্ছে, (খ) কী ত্যাগ করতে হচ্ছে (Winters et al.-এর cost dimension উল্লেখ করে), (গ) এটি কতটা **One-Way** বনাম **Two-Way Door**, এবং (ঘ) কমপক্ষে **দুটি বাস্তব failure mode** উল্লেখ করুন। *(মনোযোগ দিন: আপনার পছন্দের বিকল্পটির অসুবিধা ও failure mode-ও সততার সাথে লিখতেই হবে!)*
3. **Complete 1-Page ADR (`LO 4.4`):** ওপরের বিশ্লেষণের ভিত্তিতে **ইংরেজিতে** (পেশাদার engineering writing অনুশীলনের জন্য) একটি পরিপূর্ণ, সংক্ষিপ্ত ১-পৃষ্ঠার **ADR** লিখুন, যাতে অবশ্যই থাকবে:
   - `Title` (সংক্ষিপ্ত noun phrase)
   - `Status`
   - `Context` *(value-neutral ও তথ্যভিত্তিক)*
   - `Decision` *(পূর্ণ বাক্যে active voice-এ `"We will ..."`)*
   - `Alternatives Considered` *(উভয় বিকল্পের সৎ মূল্যায়ন)*
   - `Consequences` *(নির্বাচিত বিকল্পের **Positive**, **Negative**, এবং **Neutral** পরিণতি)*
   - `Assumptions & Revisit Triggers` *(কোন পরিমাপযোগ্য শর্ত ঘটলে আপনি এই সিদ্ধান্ত বদলে নতুন ADR লিখবেন)*

---

### Exercise 2: Reliability Cost & Risk Tolerance Analysis (~20–25 min)

**দৃশ্যপট (Scenario):**
একটি মাঝারি ই-কমার্স কোম্পানির **internal weekly reporting tool** আছে, যেটি প্রতি সোমবার সকাল ৯টা থেকে ১১টার মধ্যে কোম্পানির ১৫ জন finance ও operations কর্মী খুলে আগের সপ্তাহের বিক্রির সারসংক্ষেপ ও spreadsheet export দেখেন। সপ্তাহের বাকি সময় tool-টি খুব কমই ব্যবহৃত হয়।

নতুন একজন manager একটি সম্মেলন থেকে ফিরে এসে ticket খুললেন: *"আমাদের সব internal tool-কে এখন enterprise মানদণ্ডে নিতে হবে—এই reporting tool-টির availability লক্ষ্যমাত্রা **99.999% (five nines)** করতে হবে।"*

আপনার টিমে backend engineer সীমিত, এবং ওই একই টিম customer-facing checkout API ও payment reconciliation-ও সামলায়।

**কাজ (Tasks):**
1. **Downtime Budget-এর বাস্তব অনুবাদ:** এক বছরে **99.999%** (~5.26 মিনিট/বছর) বনাম **99.5%** (~43.8 ঘণ্টা/বছর বা ~3.65 ঘণ্টা/মাস) availability-র পার্থক্যটি এই reporting tool-এর প্রকৃত ব্যবহারের সময়ের (প্রতি সোমবার সকালের ২ ঘণ্টা) আলোকে ব্যাখ্যা করুন।
2. **Google SRE *Embracing Risk* বিশ্লেষণ (`LO 4.1`, `LO 4.2` — Awareness scope):** কেন 99.999% লক্ষ্যমাত্রা চাপিয়ে দিলে কোম্পানির লাভের বদলে **বাস্তব ক্ষতি** হবে তা ব্যাখ্যা করুন—অবশ্যই নিচের তিনটি দিক যুক্ত করবেন:
   - অতিরিক্ত "nines"-এর **nonlinear redundancy/infrastructure খরচ**;
   - **Opportunity Cost** (একই টিম customer-facing checkout ও payment-এ কোন কাজ হারাবে);
   - **Risk Tolerance** ও ধারণাগত **Error Budget** (কেন 99.5% বা সমমানের লক্ষ্য এই সার্ভিসের প্রকৃত ঝুঁকি সহনশীলতার সাথে বেশি মানানসই—কোনো SLI/SLO সূত্র বানানোর দরকার নেই)।
3. **Failure Type-এর সূক্ষ্ম পার্থক্য:** Availability লক্ষ্যমাত্রা 99.5% রাখা মানেই কি *সব ধরনের ব্যর্থতা* মেনে নেওয়া? এমন একটি **গ্রহণযোগ্য failure** (যেমন রোববার রাত ২টায় ২০ মিনিটের maintenance restart) এবং এমন একটি **অগ্রহণযোগ্য failure** (যেমন নীরবে ভুল revenue সংখ্যা দেখানো বা অননুমোদিত কর্মীর কাছে গোপন বেতন/মার্জিন ডেটা ফাঁস হওয়া)-এর তুলনা করে দেখান কেন ব্যর্থতার ধরন গুরুত্বপূর্ণ।
4. **Revisit Trigger:** এমন একটি বাস্তব ব্যবসায়িক পরিবর্তনের উদাহরণ দিন, যা ঘটলে এই reporting tool-টির availability লক্ষ্যমাত্রা 99.5% থেকে বাড়িয়ে পুনর্বিবেচনা করা আসলেই যুক্তিসঙ্গত হবে।

---

## Closed-Book Mastery Assessment (Optional — ~30–40 min)

এই assessment-টি **ঐচ্ছিক (optional)**—Block 5 শুরু করার জন্য এটি সম্পন্ন করা বা ChatGPT-র অনুমোদন নেওয়া বাধ্যতামূলক নয়। আপনি চাইলে:
- এই lesson ও বাইরের সব tab বন্ধ করে নিচের পাঁচটি প্রশ্নের স্পষ্ট, মৌলিক ও cause-and-effect সমৃদ্ধ উত্তর `phases/phase-01/notes/block-04-notes.md` ফাইলে লিখে আনুষ্ঠানিক মূল্যায়নের জন্য **ChatGPT**-র কাছে শেয়ার করতে পারেন (`Mastery verified`-এর প্রমাণ হিসেবে);
- অথবা আপনি যথেষ্ট শিখেছেন মনে করলে assessment বাদ দিয়েই পরের block-এ এগিয়ে যেতে পারেন (`Studied — mastery not formally verified`);
- অথবা assessment না করেই যেকোনো অস্পষ্ট ধারণা নিয়ে প্রশ্ন ও আলোচনা করতে পারেন।

*(নোট: ~৩০–৪০ মিনিট কেবল যাঁরা এটি করতে চান তাঁদের জন্য একটি আনুমানিক লক্ষ্যমাত্রা—কঠোর সময়সীমা নয়। পাঁচটি বিশ্লেষণাত্মক প্রশ্ন গভীরভাবে লিখতে এর চেয়ে বেশি সময় লাগতেই পারে।)*

1. **(LO 4.1 / Outcome 5)** বাস্তব software engineering-এর **ছয়টি constraint মাত্রা** কী কী? **Hard Constraint** ও **Soft Constraint**-এর পার্থক্য ব্যাখ্যা করুন, এবং একটি উদাহরণ দিয়ে দেখান কীভাবে কোনো non-technical constraint (যেমন team, schedule, operational, বা regulatory) উপেক্ষা করলে কারিগরিভাবে "উন্নত" একটি সমাধানও ব্যর্থ হয়।
2. **(LO 4.2 / Outcome 6)** Mark Richards ও Neal Ford-এর **First Law of Software Architecture** এবং *"least worst"* নকশার ধারণা ব্যাখ্যা করুন। বড় কোম্পানির "industry best practice" বা **"100% reliability"** লক্ষ্য অন্ধভাবে অনুসরণ করা কেন প্রায়ই ক্ষতিকর—তা Google SRE-এর nonlinear cost, **Risk Tolerance**, এবং ধারণাগত **Error Budget** (`Awareness`) ব্যবহার করে যুক্তিসহ বলুন।
3. **(LO 4.3 / Outcome 6)** **One-Way Door** বনাম **Two-Way Door** সিদ্ধান্তের পার্থক্য (প্রতিটির একটি করে নিজস্ব backend উদাহরণসহ) ব্যাখ্যা করুন। একটি সাজানো pros/cons তালিকার সাথে একটি সৎ engineering trade-off তুলনার মূল পার্থক্য কোথায়?
4. **(LO 4.3 / Outcome 10)** **Decision Quality** এবং **Outcome Luck**-এর পার্থক্য ব্যাখ্যা করুন। একটি উচ্চ-মানের engineering সিদ্ধান্তও কেন ছয় মাস পর আর উপযুক্ত না থাকতে পারে—এবং Winters et al. (*Revisiting Decisions, Making Mistakes*) অনুযায়ী তখন একজন দায়িত্বশীল engineer-এর কী করা উচিত?
5. **(LO 4.4 / Outcomes 6, 10)** Michael Nygard-এর **Architecture Decision Record (ADR)** কোন দুটি বিপরীত বিপদ (blind acceptance বনাম blind reversal) ঠেকায়? একটি ADR-এর মূল অংশগুলো (Title, Status, Context, Decision, Alternatives Considered, Consequences) সংক্ষেপে ব্যাখ্যা করুন—বিশেষত Context কেন value-neutral হতে হয়, Consequences-এ নেতিবাচক দিক লেখা কেন বাধ্যতামূলক, এবং পুরোনো সিদ্ধান্ত বদলালে আগের ADR ফাইলটি কেন মুছে না ফেলে `Superseded` করা হয়?

### Block 4 Readiness & Progression Checklist
- [ ] Guided lesson গভীরভাবে পড়েছেন এবং Pause & Checks 1–3 সম্পন্ন করেছেন (পরে Self-Check Guidance মিলিয়ে দেখেছেন)।
- [ ] Complementary mandatory reading (**Nygard 2011**, **Google SRE Ch. 3** নির্বাচিত অংশ, এবং **SWE at Google Ch. 1** নির্বাচিত অংশ) সম্পন্ন করেছেন।
- [ ] `phases/phase-01/notes/block-04-notes.md` ফাইলে স্বাধীনভাবে Exercise 1 (Constraint-Driven ADR) এবং Exercise 2 (Reliability Cost & Risk Tolerance Analysis) লিখেছেন।
- [ ] *(Optional)* বই/নোট বন্ধ রেখে Mastery Assessment Questions 1–5 লিখেছেন এবং আনুষ্ঠানিক মূল্যায়নের জন্য ChatGPT-র কাছে শেয়ার করেছেন।
- [ ] *(Optional)* কোনো ধারণা অস্পষ্ট মনে হলে assessment ছাড়াই সেগুলো নিয়ে ChatGPT-র সাথে আলোচনা করে পরিষ্কার হয়েছেন।

> **অগ্রগতির অবস্থা (Progression Status Distinction):** Block 5 কখন শুরু করবেন তা আপনার সিদ্ধান্ত। পড়া ও চর্চা শেষ করে assessment ছাড়া এগোলে এই block-টির অবস্থা হবে **Studied — mastery not formally verified**। যথেষ্ট প্রমাণ (exercise ও/অথবা assessment) শেয়ার করে মূল্যায়ন সম্পন্ন করলেই তা **Mastery verified** হিসেবে গণ্য হবে; assessment বাদ দিলে তা স্বয়ংক্রিয়ভাবে `Mastery verified` হয় না।

---

## Self-Check Guidance for Inline Pause & Checks 1–3

> *`notes/block-04-notes.md` ফাইলে নিজের উত্তর লেখার **পরেই কেবল** নিচের নির্দেশিকা দেখুন। Practical Reasoning Exercises বা Optional Closed-Book Mastery Assessment-এর কোনো উত্তর এখানে দেওয়া হয়নি—সেগুলো আপনার স্বাধীন চর্চা ও (চাইলে) ঐচ্ছিক ChatGPT মূল্যায়নের জন্য learner-owned থাকবে।*

- **Pause & Check 1:**
  1. *শক্তিশালী উত্তরে থাকবে:* চারটি বা তার বেশি স্পষ্ট মাত্রা ও Hard/Soft বিভাজন—যেমন: (ক) **Regulatory/Compliance (Hard):** স্থানীয় আইন অনুযায়ী রোগীর ডেটা দেশের বাইরের server-এ রাখা যাবে না; (খ) **Schedule (Hard):** ৮ সপ্তাহের মধ্যে launch; (গ) **Team/Organizational (Hard, স্বল্পমেয়াদে):** ৪ জন engineer কেবল Node.js ও PostgreSQL জানেন; (ঘ) **Technical / Performance (সম্ভবত Soft — যাচাইসাপেক্ষ):** *"সাধারণত ২ সেকেন্ডের মধ্যে খুললে ভালো হয়"* একটি নমনীয় লক্ষ্য, কোনো কঠোর আইনি বা নিরাপত্তা-সীমা নয়; (ঙ) **Operational/Resource (Hard/Soft):** ৪ জনের ছোট টিমের পক্ষে রোগীর সংবেদনশীল ডেটাসহ জটিল অবকাঠামো ২৪/৭ সামলানো।
  2. *শক্তিশালী উত্তরে থাকবে:* মাসে ২৫০ ডলার **Financial Cost** বাঁচাতে গিয়ে startup-টি বিপুল **Personnel Cost** (৪ জনের মধ্যে engineer-দের backup, OS security patching, failover ও monitoring সামলানোর সময়), **Opportunity Cost** (৮ সপ্তাহের কঠিন সময়সীমায় আসল ল্যাব-রিপোর্ট feature, security audit ও test না করে DB প্রশাসন করা), **Transaction Cost** (ভবিষ্যতে স্কেল বা সমস্যা হলে unmanaged VM থেকে migrate করার ঝুঁকি), এবং **Societal/User Cost** (স্বাস্থ্যসেবার সংবেদনশীল ডেটায় manual config ভুলে breach বা backup ব্যর্থতায় রোগীর রিপোর্ট হারানো)-এর ঝুঁকি নিচ্ছে—যা ২৫০ ডলারের চেয়ে বহু গুণ বড় ক্ষতি।
- **Pause & Check 2:**
  1. *শক্তিশালী উত্তরে থাকবে:* Richards ও Ford-এর **First Law** অনুযায়ী প্রতিটি স্থাপত্যগত পছন্দই একটি trade-off—সর্বজনীন "best practice" বলে কিছু নেই। বড় পেমেন্ট কোম্পানির constraint (শত শত engineer, বহু টিম, বিপুল transaction ও আলাদা SRE টিম) আর ৪ জনের startup-এর constraint সম্পূর্ণ ভিন্ন। ওই নকশা নকল করলে startup-টি বড় কোম্পানির distributed operational ও cognitive cost ঠিকই বহন করবে, কিন্তু তার প্রয়োজনীয় সমস্যাই নেই; ফলে এটি প্রেক্ষাপটের জন্য *"least worst"* না হয়ে ধ্বংসাত্মক overengineering হবে।
  2. *শক্তিশালী উত্তরে থাকবে:* ১০০% uptime-এর কাছাকাছি যেতে redundancy ও engineering শ্রমের খরচ অরৈখিকভাবে (nonlinearly) আকাশছোঁয়া হয়ে যায়, নতুন feature ও উদ্ভাবন থমকে যায় (opportunity cost), আর অনেক সময় ব্যবহারকারীর নিজস্ব নেটওয়ার্কের অস্থিরতার কারণে তিনি সেই অতিরিক্ত পার্থক্য টেরই পান না। **Error Budget** (যেমন 99.9% লক্ষ্য থাকলে বাকি 0.1% অনুমোদিত ব্যর্থতার সীমা) Product টিম ও Ops/SRE টিমকে একটি যৌথ বস্তুনিষ্ঠ মাপকাঠি দেয়—বাজেট থাকলে দ্রুত release ও পরীক্ষা চলবে, বাজেট ফুরোলে সাময়িকভাবে stability-তে মন দেওয়া হবে।
- **Pause & Check 3:**
  1. *শক্তিশালী উত্তরে থাকবে:* **(ক) One-Way Door (বা অত্যন্ত ব্যয়বহুল-to-reverse):** ২০০টি external partner একবার payload ও auth contract-এর ওপর নিজেরা কোড লিখে ফেললে (Hyrum's Law) পরে তা বদলাতে দীর্ঘ deprecation, multi-version support ও partner coordination (বিশাল transaction cost) লাগে। **(খ) Two-Way Door:** কেবল internal টিমের debugging logger format নিজেদের repository-তে অল্প সময়ে কম ঝুঁকিতে বদলে ফেলা বা আগের অবস্থায় ফেরত নেওয়া যায়।
  2. *শক্তিশালী উত্তরে থাকবে:* এটি একটি পক্ষপাতদুষ্ট (rigged) pros/cons তালিকা, কারণ নির্বাচিত Option A-র কোনো বাস্তব ত্যাগ বা সীমাবদ্ধতা দেখানো হয়নি এবং Option B-র কোনো প্রকৃত শক্তি স্বীকার করা হয়নি। একে সৎ করতে যোগ করতে হবে: (১) **Option A-র বাস্তব downsides ও ত্যাগ** (কী হারাচ্ছি), (২) **Option A ও B উভয়ের বাস্তব failure modes**, (৩) **প্রমাণিত তথ্য বনাম অনুমান (assumptions)** আলাদা করা, এবং (৪) সুনির্দিষ্ট **Revisit Triggers** (কোন শর্তে সিদ্ধান্তটি বদলাতে হবে)।
  3. *শক্তিশালী উত্তরে থাকবে:* **Decision Quality** বিচার করা হয় সিদ্ধান্ত নেওয়ার সময় যে constraint, প্রমাণ ও যুক্তি সহজলভ্য ছিল তার মান দিয়ে; আর **Outcome Luck** হলো নিয়ন্ত্রণের বাইরের ভাগ্যের কারণে ভালো বা খারাপ ফল হওয়া। যেমন: সব constraint ও load test যাচাই করে সরল PostgreSQL নেওয়ার পর অপ্রত্যাশিত হঠাৎ ৫০× ভাইরাল স্পাইকে চাপ পড়া মানে আগের সিদ্ধান্ত খারাপ ছিল না (ভালো Decision Quality, অপ্রত্যাশিত নতুন প্রেক্ষাপট); আবার কোনো testing বা constraint না ভেবে জটিল distributed stack নিয়ে কাকতালীয়ভাবে বড় ট্রাফিক না আসায় টিকে যাওয়া ভালো সিদ্ধান্ত নয়, বরং Outcome Luck।
