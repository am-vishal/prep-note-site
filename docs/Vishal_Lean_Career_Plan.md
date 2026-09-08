# Vishal — Lean Career Plan (Realistic, 30 min/day, Tired-After-Work Version)

## 1. Exact first 30 days

**Week 1 — Resume metrics + rewrite (no new technical learning yet)**
- Day 1: Message your manager or check internal dashboards for one real number — order volume or customer count for your platform. That's the whole session.
- Day 2: Rewrite bullet 1 (VTEX platform work) using that number.
- Day 3: Rewrite bullet 2 (React/Node integration work) — no new number needed, just stronger verbs (owned, built, integrated — not "contributed to").
- Day 4: Rewrite bullet 3 (order sync / retry logic — you already told me this story, just write it down).
- Day 5: Read the whole resume out loud once. Fix anything that sounds passive.
- Day 6 (30 min): Update LinkedIn headline + first two lines of About section using the rewritten resume language.
- Day 7: Rest.

**Week 2 — Resume done, applications start**
- Day 1: Finalize resume as a PDF. Done — stop editing it after this.
- Day 2: Apply to 2 jobs from the target list (Section 5 below).
- Day 3: Apply to 2 more.
- Day 4: SQL — first lesson only (SELECT + WHERE), one free resource, no more.
- Day 5: Apply to 1–2 more jobs.
- Day 6 (30 min): SQL — JOIN basics.
- Day 7: Rest.

**Week 3–4 — SQL continues, applications continue, no other new topic introduced**
- 3 days/week: SQL practice (JOIN, GROUP BY, one subquery) — that's the entire technical curriculum for this month.
- 2 days/week: applications (2 each session)
- 1 day/week (Saturday): review — what got applied to, any replies, adjust next week

**By day 30 you should have**: a rewritten resume, an updated LinkedIn, ~10–14 applications sent, and basic working SQL. Nothing else. That's deliberate.

---

## 2. The 5 skills to focus on

1. **Resume/interview storytelling with real numbers** — not a technical skill, but the highest-ROI thing you have, because it's the only thing that's pure upside with zero new learning curve.
2. **SQL (joins, aggregation only)** — the one technical gap that's both fast to close and a real screening filter across almost every JD you'll see.
3. **VTEX/e-commerce interview fluency** — you already have the raw material from your prep sessions with me; the "skill" here is just repetition until it's automatic, not new learning.
4. **Behavioral/STAR storytelling** — 4 stories, reused across almost every behavioral question. Low effort, high leverage, and you already have real material (checkout customization, order sync/retry, a production bug you've debugged).
5. **Basic cloud vocabulary (conceptual only, not hands-on)** — being able to say what S3/EC2/Lambda are and when you'd use each. Skip actually deploying anything — the ROI on hands-on AWS practice at your stage isn't worth 30-minute-session friction of setting up an AWS account, billing alerts, IAM, etc.

---

## 3. The 3 skills to NOT spend time on

1. **DSA / LeetCode-style coding practice.** Your resume doesn't support a product-company DSA-round path right now, and building that up at 30 min/day realistically takes the better part of a year. Skip it entirely for this cycle — don't half-do it.
2. **Hands-on AWS deployment / any cloud certification.** Conceptual knowledge (skill #5 above) is enough for your target roles. Actually provisioning infrastructure costs setup time you don't have and isn't what Path A/VTEX-adjacent interviews test deeply.
3. **System design depth.** You don't need to be able to design a distributed system from scratch. You need one or two "here's how I'd approach this" answers ready, not a study program. Cut it from the curriculum — fold it into interview-prep week only, not months of prep.

---

## 4. The single best project for your resume

**One project only — not two.** Build the **VTEX-style Catalog + Order API**, using **only your existing stack** (Node, Express, MongoDB) — zero new tools, zero new setup. This is the single highest-signal, lowest-friction project because:
- You already know the domain model cold (Product/SKU/Trade Policy/Order) from real work
- No new technology to learn just to start typing code
- Directly supports your actual interview story ("I generalized the commerce model I work with professionally")

Skip PostgreSQL, skip AWS, skip a frontend. Backend API only. If you have leftover energy/momentum after it's genuinely done, only then consider a thin React admin UI on top — as a bonus, not a requirement.

**Resume bullet**: *"Built a RESTful product/order API modeling e-commerce catalog and pricing relationships, generalized from production VTEX platform experience (Node.js, Express, MongoDB)."*

---

## 5. Exact job titles to target

- VTEX Developer
- Senior VTEX Developer
- E-commerce Developer (React/Node)
- Full Stack Developer — E-commerce / Digital Commerce
- Software Engineer — E-commerce Platforms

**Do not target**: SDE-2/product-company roles requiring DSA rounds, Tech Lead titles requiring 6+ years or formal reports (you're close but not quite there — wait one more cycle), anything requiring cloud/DevOps as a primary skill.

---

## 6. Minimum interview preparation needed

- Your existing VTEX Q&A prep (already built — just re-read it aloud twice before any interview)
- 5 SQL questions (joins, group by — nothing exotic)
- 4 behavioral STAR stories (ownership, conflict/disagreement, a bug you debugged, a challenging feature you shipped — you already have real material for all four from our earlier conversations)
- 1 project walkthrough (the Catalog/Order API) rehearsed until it's under 90 seconds spoken
- That's it. No system design deep-dive, no DSA, no cloud deep-dive — just enough cloud vocabulary to not go blank if asked.

---

## 7. Minimum applications/networking per week

- **Applications**: 3–4 per week is the realistic sustainable minimum for a tired full-time employee — not 5–7. Quality over volume; each one should take under 10 minutes since your resume is now finalized and reusable.
- **Networking**: Optional, not required. If you do it, one LinkedIn comment or light engagement per week, Saturday, 5–10 minutes. Skip it entirely in weeks you're too tired — it's the lowest-leverage item on this whole list and won't meaningfully change your outcome if dropped.

---

## 8. When to start applying

**Week 2.** Not week 20, not after SQL, not after the project. Your existing resume + VTEX experience already qualifies you for the target titles in Section 5. The project and SQL improve your odds and your interview performance, but they are not gates on applying — they run in parallel, starting immediately.

---

## 9. Skills that actually move you toward ₹15–20+ LPA

Ranked by real impact, not by how "impressive" they sound:

1. **A resume that states real scale and ownership instead of vague task descriptions** — this alone can be the difference between getting screened out and getting a call, at zero new learning cost.
2. **Positioning as a VTEX/e-commerce specialist, not a generic full-stack dev** — niche demand commands a premium; VTEX partner companies and e-commerce brands are specifically short on this combination.
3. **SQL** — clears a screening filter that would otherwise silently cost you interviews you never even hear about.
4. **A demonstrable project that mirrors real production complexity** (retry/idempotency, catalog modeling) rather than a toy CRUD app — this is what separates a 5-year engineer's portfolio from a fresher's.
5. **Interview fluency under real conversational pressure** — not more knowledge, just smoother delivery of what you already know. This is often the actual gap between an offer and a "we went with someone else" at your experience level.

Notably absent from this list: cloud, system design, DSA. They're not zero-value, but for your specific resume and timeline, they are not what moves the needle from ₹12 to ₹18 LPA. Positioning and screening-filter basics do that; deep new technical breadth doesn't, not on this timeline.

---

## 10. Revised dashboard — 3 sheets, not 10

Realistically maintainable for a tired person. Everything else from the original 10-sheet version is cut.

**Sheet 1 — Applications** (the only tracker that directly drives outcomes)
Columns: Company | Role | Date Applied | Source | Status (dropdown: Applied/Interview/Rejected/Offer) | Next Action | Notes

**Sheet 2 — Daily Streak** (just enough to build the habit, nothing more)
Columns: Date | Did 30 min? (checkbox) | What I did (one line) | Streak (formula: `=IF(B2=TRUE, C1+1, 0)`)

**Sheet 3 — Skills & Project Checklist** (static list, checked off once, not a heavy tracker)
Columns: Item (SQL joins / SQL group-by / Resume rewrite / LinkedIn updated / Catalog API — auth done / Catalog API — CRUD done / Catalog API — README done) | Done? (checkbox) | Date completed

**One formula worth having**: Application conversion rate — `=COUNTIF(Sheet1_Status,"Interview")/COUNTA(Sheet1_Applied)` — check this monthly, not daily, to see if your resume/targeting is working or needs another pass.

No skill matrix, no company target list as a separate sheet (fold good companies directly into Sheet 1 when you apply), no separate interview question bank sheet (use your existing prep docs instead), no weekly review sheet (Saturday's 30-min session on Sheet 1/2 review IS the weekly review).

---

## Bottom line

If you genuinely only have 30 minutes and you're tired most days, the entire plan compresses to three sentences: **Fix your resume with real numbers in week 1, start applying in week 2, and spend every other spare session on SQL and rehearsing your VTEX interview answers until they're automatic.** Everything else in the original 16-section plan was optional insurance — useful if you have energy to spare, not required for the ₹15–20 LPA outcome.
