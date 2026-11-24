# Phase 1, Block 1 — Learner Notes

## Pause & Check 1

### Question 1
> এই মাসে কোনো open bug ticket না থাকা এবং 100% uptime থাকার পরও একটি backend service কীভাবে *unsustainable* হতে পারে?

**Learner Answer:**
- **Initial response:** The most common reasons can be security vulnerability, dependent libraries are deprecated, increase of traffic, if the code is not good for horizontal scaling, but to handle the load it needs to be horizontally scaled, etc.
- **Refined addition:** Along with security vulnerability, deprecated libraries, etc., factors like lack of proper automated tests and change in business rules are also necessary.

### Question 2
> একজন সহকর্মী বললেন: *"আমাদের login service-এর dependencies-এ হাত দিয়ো না—এখন তো ঠিকই চলছে।"* Cause-and-effect দেখিয়ে এই যুক্তির দুর্বলতা ব্যাখ্যা করুন, এবং এমন একটি বাস্তব পরিস্থিতি বলুন যেখানে কার্যকর কোডে হাত না দেওয়াটা আসলেই যুক্তিসঙ্গত।

**Learner Answer:**
- **Part 1:** If it's an important API, many users can get affected by this, so we should obviously touch it. Like last year, Next.js had a CVE issue, and we had to change the Next.js version of those projects.
- **Part 2:** If the code is already being deprecated, or we are closer to an event where we shouldn't change the stable setup, or it doesn't have good test cases written which may result in failure in some cases, then it's better not to touch the code.
- **Refinement on missing tests:** We can write the test cases first. If the test is not necessary, then we can update the code.

## Pause & Check 2

### Question 1
> টিম ২ জন থেকে ৩০ জন engineer-এ বড় হলে ভেঙে পড়ে এমন একটি recurring backend policy-র উদাহরণ দিন, এবং এমন একটি engineering কাজের কথা বলুন যা কেবল automate করে sub-linearly scale করা যায় না।

**Learner Answer:**
- **Part 1 (Scaling from 2 to 30 engineers):** Handling a team of 2 engineers is easy compared to a team of 30 engineers. A team of 2 may already be familiar with each other's coding style, the review is easy, and communication is easy. However, if the team is large, handling each one's problems, communicating with each other, making everyone up to date, and following the same pattern is tough as everyone has different opinions. Also, PR review becomes tougher.
- **Part 2 (Cannot be scaled sub-linearly by automation alone):** Software architecture needs engineering judgment. As we have more engineers, it is not possible to get a sub-linear solution; aligning the engineers is not sub-linear.

### Question 2
> কোনো ত্রুটি আগে ধরলে তা *সব সময়* exponentially কম খরুচে হয়—এই দাবিটি কেন পুরোপুরি সঠিক নয়? Release-এর পর ঠিক করা সস্তা এমন একটি ছোট সমস্যার সাথে, আগে না ধরলে বিশাল ক্ষতি হয় এমন একটি ত্রুটির তুলনা করুন।

**Learner Answer:**
We should not do over-engineering; it has opportunity cost. Today we have many automated systems like CI/CD pipelines, so updating cheap issues is fine rather than investing all our time into finding such cases, although it depends on the importance of the API we are building.
- **Comparison addition:** Small bugs like a spelling mistake or a date format mistake are negotiable after release, compared to making a mistake in payment APIs.

## Pause & Check 3

### Question 1
> ওপরের startup উদাহরণে (নিজে database সামলে মাসে $300 বাঁচানো, কিন্তু patch, backup ও alert-এ মাসে ~১৫ ঘণ্টা senior engineering সময় ব্যয় করা) কোন কোন cost dimension-এর মধ্যে trade-off হয়েছে, এবং এটি কেন সামগ্রিকভাবে লোকসান হতে পারে?

**Learner Answer:**
Personnel cost: we are saving $300 but giving a senior engineer more than that. It also has opportunity cost, because we could have used the engineer for some great feature work. Also, if the engineer does something wrong, it can cost the user too.

### Question 2
> ২৫ লাইনের একটি একবার-ব্যবহারযোগ্য (throwaway) CSV script-কে over-engineer করা কেবল "বাড়তি সতর্কতা" নয়, বরং *Software Engineering*-এর ব্যর্থতা কেন?

**Learner Answer:**
It has the same personnel cost and opportunity cost. We have to understand the blast radius and lifespan of the work as well.
