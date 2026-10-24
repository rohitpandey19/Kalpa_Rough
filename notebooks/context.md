# Project context: Build 1, Data to Insight in Kalpa Health

Source: `Build1_KalpaHealth_ProjectHandbook_STUDENT_v3.pdf` (30 pages, IIT Gandhinagar, PG Diploma in AI-ML and Agentic AI Engineering, Cohort 2, Mini Project 1, Build Week 1, Module 1). This file condenses the handbook so the notebooks can be read against it. When the two disagree, the handbook wins.

Kalpa Group and Kalpa Health are fictional. Dr Lal PathLabs, Metropolis, Thyrocare and Apollo Diagnostics are real, and the handbook quotes their published figures as checked on 19 October 2026.

---

## 1. The problem in one paragraph

Kalpa Health is a diagnostics chain: six Indian cities, two channels (clinic and home collection), one central lab per city. The board approved a plan for **+18% test volume in H1** of this year, set city by city on last year's mix. The half closed at **+5%**. The group board meets in Singapore **in two weeks** and will ask: *"Where did the 13 points go, and what are you doing about it?"* The COO, **Dr Priya Menon**, needs **one sentence per pain point that she can defend to the board, backed by numbers her finance team can audit.**

## 2. The industry frame

A chain sells tests: a booking, then a sample is collected, then the lab runs one or more tests on it, then an invoice is raised.

**The tree:** Revenue = Patients × Tests per patient × Revenue per test.
- Worked example, Metropolis, quarter to June 2026: Rs 450.2 cr ≈ 3.7 M patients × ~2.1 tests × Rs 575 per test.
- For Kalpa, fill all three boxes for **both years** by the end of profiling.

**Peers, quarter to June 2026 (Q1 FY27):**

| | Revenue | Patients | Tests or samples | Note |
|---|---|---|---|---|
| Dr Lal PathLabs | Rs 797.7 cr, +19.1% | 8.2 M, +8.2% | 25.9 M samples, +10.7% | revenue grew more than twice as fast as patients |
| Metropolis | Rs 450.2 cr, +16.6% | 3.7 M, +10% | 7.8 M tests, +11% | Rs 1,219 per patient, Rs 575 per test, no price rise, so the test mix moved |
| **Kalpa** | not public | | **test volume +5% vs plan +18%** | the board won't accept "the market was soft" |

**Warm-up questions:**
1. Work out Dr Lal's revenue per patient and per sample, and state your assumptions.
2. Why is Rs 575 per test not comparable with a per-sample figure?
3. Both chains grew money faster than people. Which branch of the tree moved, and what in Kalpa's data would show the same thing?

**Vocabulary:**
- **Patient**: one person who booked, counted once per visit.
- **Sample**: what's collected (a tube, a swab). One visit can give several.
- **Test**: one analysis on a sample. A package is many tests on one sample.
- **Home collection**: costs more per visit, but wins patients who wouldn't travel.
- **No-show**: a booking where no sample was ever collected.
- **Realisation**: revenue per test rising with no price rise, because the mix shifted to dearer tests.

## 3. The client's systems, mapped to the data pack

| System | What it records | Table in `data/raw/` |
|---|---|---|
| Booking system | bookings from app, phone and front desk. **Two cities moved to a new vendor in Q2.** | `bookings/legacy`, `bookings/nova` |
| Collection app | a phlebotomist marks a sample collected, or the patient absent | `collections` |
| Lab system | each test, its result and timestamp | `results` |
| Billing | one invoice per visit, from the tests performed | `invoices` |
| Payment gateway | card, UPI and cash deposits. **Its ids look nothing like billing's.** | `payments` |
| Marketing tool | which bookings came under an offer | `campaigns` |
| Also included | patient master, city plan, reference tables | `patients`, `plan`, `reference` |

Nothing in the pack has been cleaned. *"A count is only as true as the system that recorded it."*

## 4. The five briefs (a group answers one, end to end)

| | Brief | Dr Menon's words | Who pushes back | Method | Tables |
|---|---|---|---|---|---|
| **A** | Growth | "Show me the part of my business that is short, by how much, and where." | finance (costs were sized for 18%) | **the tree** | bookings, results, patients, plan |
| **B** | Two cities | "Before I cut targets or people, I need to know whether the fall is real." | city heads (they blame Thyrocare's new centres and lower prices) | **the ladder** | bookings (old), bookings (new), patients |
| **C** | Money | "Which invoices were never paid, which payments match no invoice, and how much of the gap is only timing?" | finance controller | **the bridge** | invoices, payments |
| **D** | One clinic | "Tell me whether the difference is real, and if it is, whom it affects." | regional head (wants the manager replaced). The manager says it's bad luck. | **the chance reference** | bookings, collections |
| **E** | The offer | "Before I sign, tell me whether our offer caused our rise." | marketing lead (defends the 9%, wants the budget doubled) | **the fair comparison** | bookings, campaigns, patients |

Every brief has the same layout: a pipeline, then four steps with less help each time (1 fully guided, 2 with a hint, 3 the question only, 4 open). Each step comes with a "why this, and not that?" prompt, followed by "Done looks like", "Push further" and three viva questions.

### A · Growth
- **Target:** split the 13-point gap across the branches of the tree and across cities and channels, multiplying back to the total, with one sentence for the board.
- **Tree to complete:** Test volume = Patients served (= New patients + **?**) × **?** × Tests per visit. Revenue = Test volume × **?**. Expected fills: returning patients, visits per patient, revenue per test.
- **Step 1:** rebuild test volume for both halves and check it against the 5%. If it differs, find out why, and log it. *Which table did you count from, and why not the other two?*
- **Step 2:** split growth into branches, and check that they multiply back to the total. *Why patients × visits × tests rather than patients × tests? What does the middle branch reveal?*
- **Step 3:** which cities, and which channel, own most of the gap? *Why city before channel?*
- **Step 4:** what would make you wrong? If the gap sits in one or two cities, is it a business problem or a recording problem? *Which single check would you run with ten minutes left?*
- **Industry prompt:** Metropolis earned about 5% more per test with no price rise. What in your data separates a price change from a mix change?
- **Done looks like:** every branch numbered for both years; the branch and cities owning most of the gap, with their share; an approach-log row per join and split; one sentence.
- **Push further:** if the short branch recovered to plan, how many more tests would run, and on what assumption?
- **Viva:** (1) Why these branches for a lab, and which retail branch did you drop? (2) Show me now that the branches multiply back to the total. (3) What changes when an error is a missed diagnosis rather than a missed sale?

### B · Two cities
- **Situation:** two cities' bookings fell in Q2 while four held. Both migrated to a new booking vendor that quarter, in a rushed migration. Nobody documented how the new system counts **a visit with two sample tubes** or **a rescheduled appointment**. Dr Menon is weighing whether to cut those cities' targets and headcount.
- **Target:** split each city's fall into a **counting part** and a **real part**, in bookings and in percent, with the real part traced to a branch of the tree.
- **The ladder:** (1) confirm the drop is real, (2) compare like with like, (3) **?**, (4) **?**, (5) hypothesise, and say what evidence would settle it. Rungs 3 and 4 are for you to fill, e.g. decompose and isolate.
- **Steps:** count bookings by city and quarter with every count labelled by its system (*why bookings, not patients or tests?*); check whether the two systems count a booking the same way by tracing one patient's visit as rows in each; remove the counting difference, then ask whether the rest sits in patients, visits per patient or tests per visit; name two explanations outside the data you can't rule out, and what you'd ask the city heads.
- **Industry prompt:** moving from Universal Analytics to GA4 changed session counts because of the midnight and campaign restart rules. *What is Kalpa's "midnight rule"?*
- **Done looks like:** each city's fall split into counting and real; the system mapping and rejected alternatives in the approach log; two sentences the city heads would accept.
- **Push further:** write the rule Kalpa should follow the next time a city changes systems.
- **Viva:** (1) How did you reconcile different identifiers, and what did you throw away? (2) What share was counting, and how did you prove it? (3) Sales dropped 15% last month. Walk me through it.

### C · Money
- **Situation:** for Q2, billing shows more invoiced than the payment feed shows collected. Nobody has reconciled them since the gateway switch. The difference could be invoices never paid, payments matching no invoice, duplicate settlement rows, or money arriving after the quarter closed.
- **Target:** a bridge from invoiced to collected where every bar is **a count and an amount**, both sides reconcile, and finance has a list to act on.
- **Bridge:** Invoiced in the quarter, then "Invoices with no payment" (named for you), then **?** **?** **?** (e.g. timing after cutoff, payments matching no invoice, amount differences or fees, duplicates), then Collected in the payment feed.
- **Steps:** profile both exports before joining (counts, totals, id formats, date ranges, duplicates) and write the id mapping rule; join and explain the row count before and after (*more rows than invoices means the payment side repeats. Why LEFT, not INNER?*); classify never paid, unmatched payments and timing (*why is timing its own bar?*); name one way the bridge could balance and still be wrong.
- **Industry prompt:** Razorpay keeps separate `order_` / `pay_` / `setl_` id families, a Rs 1,000 payment carries a Rs 29 fee, and one settlement bundles many payments. Which of these explains a gap that isn't a leak?
- **Done looks like:** input = matched + unmatched on both sides; the mapping rule and its alternatives logged; a list finance can act on tomorrow.
- **Push further:** turn the bridge into a weekly check. Which numbers do you compare, and what triggers an alert?
- **Viva:** (1) Walk me through the join. What did the row count do? (2) How do you know your unmatched list is complete? (3) The dashboard and Finance disagree. What do you do first? Plus: if finance says the total is off by Rs 2 lakh, which bar do you re-check first?

### D · One clinic
- **Situation:** one clinic's no-show rate looks far worse than the rest. The manager says "bad luck, small numbers". The regional head wants the manager replaced. Dr Menon has seen clinic rankings flip from one quarter to the next.
- **Target:** a verdict of **real, noise, or not yet knowable**, with the definition, the denominators, a like-with-like comparison and a chance reference.
- **Illustration:** Clinic P has 8 no-shows in 32 bookings and Clinic Q has 250 in 1,000. Both are 25%. Which would you trust?
- **Steps:** define a no-show in one sentence and report every rate with its numerator and denominator (*name a booking your definition counts that someone else wouldn't*); shuffle clinic labels to see how often a clinic this size looks this bad by chance (*why shuffle rather than use a formula? What does the shuffle assume?*); compare like with like on the same slots, tests and channel (*which would you drop if you could control for only two?*); weigh the cost of a false flag against the cost of staying quiet.
- **Industry prompt:** Tu & Gilthorpe (2007) found small hospital trusts at both the top and bottom of league tables, and Wainer found the same pattern behind the small-schools push. *How large is your clinic?*
- **Done looks like:** a verdict with definition, denominators, like-with-like and the chance reference; the shuffle choice logged; if "not yet", the number of bookings that would settle it.
- **Push further:** plot every clinic's rate against its bookings, with the band chance alone would produce (a funnel plot). Who sits outside it?
- **Viva:** (1) How do you know a difference is significant, and what does your p-value not mean? (2) How many bookings would settle it? (3) What does a no-show cost a patient, compared with a cancelled order in retail?

### E · The offer
- **Situation:** marketing ran free home collection for a quarter and reports **bookings +9% against the previous quarter**. They want the budget doubled. Each offered booking costs a phlebotomist visit. Nobody has asked *who* used the offer.
- **Target:** a verdict on cause, backed by a fair comparison and the lift in every segment, with a budget recommendation that states the cost of being wrong.
- **Causal diagram:** Offer → **?** → Bookings, with two blank confounders to name before testing anything (e.g. **seasonality** and **selection**, meaning patients who'd have booked anyway). Then fill the grid *segment × (offer before, offer after, no offer before, no offer after)*.
- **Steps:** reproduce marketing's 9%, stating which bookings, periods and cities it compares; profile who got the offer by comparing **booking behaviour before the offer**; split by city, patient type and channel (*why can every part fall while the total rises?*, i.e. Simpson's paradox and mix); name the comparison that would convince marketing, and how you'd choose who doesn't get the offer next quarter.
- **Industry prompt:** Blake, Nosko & Tadelis (eBay, 2014): brand search ads brought no measurable short-term benefit, and most spend reached buyers who would have bought anyway.
- **Done looks like:** a verdict on cause, the lift per segment, rejected comparisons logged, and a budget recommendation with its cost of being wrong.
- **Push further:** design the next campaign so it can be answered cleanly: who gets it, who doesn't, for how long, measured how (a holdout).
- **Viva:** (1) Correlation or causation: which did marketing show, and which did you? (2) One sentence for the board. (3) Would you double the budget, and what would change your mind?

## 5. Objective, targets, scope

**Objective:** answer one pain point end to end: a claim Dr Menon can repeat, the evidence with every denominator shown, **the one caveat** that would change it, and **the action with its cost**.

| Target | Meaning |
|---|---|
| **T1 Reproduce the 5%** | your test-volume growth matches, or you can say exactly why not |
| **T2 One sentence** | a headline with a number, a denominator and a period, that a COO can say aloud |
| **T3 Auditable** | every number traces to rows, and every cleaning decision is logged with its effect on the headline |
| **T4 Reproducible** | notebooks run top to bottom from a fresh Codespace in **under ten minutes** |
| **T5 Defended** | every member can explain every number and every rejected approach |

- **In scope:** the data pack as released; **H1 this year vs H1 last year**; your brief; Python notebooks, SQL for joins, Excel for the one board number.
- **Out of scope:** forecasting; fixing Kalpa's systems; data you don't have (say so in the caveat); dashboards and web apps unless everything else is done.
- **Deliverables:** six slides; numbered notebooks with outputs cleared that run cold; three logs (approach, decisions, challenges); write-ups (one section per member); a video.

**Requirements (handbook rows, add your own):**
- *Functional:* reproduce the 5% or explain the gap; answer with a number and its denominator; produce the list or split someone can act on.
- *Non-functional:* cold run under 10 minutes with no manual edits; a finance analyst can audit every number to rows; every cleaning decision logged before the next step.
- *Constraints:* only the free tools covered so far; **no assistants on analysis, notebooks or write-ups**; the data as released; 3–4 days; a group of 3–4.

## 6. The pipeline and gates

**Sources → Profile → Clean and reconcile → Analyse → Answer**
- *Profile:* row counts, null rates, id formats, date ranges, duplicates.
- *Clean and reconcile:* decisions log, input = clean + rejected, revenue bridge, join row counts.
- *Analyse:* tree / ladder / bridge / chance reference / fair comparison.
- *Answer:* claim, evidence, caveat, action.

| Gate | Condition |
|---|---|
| **Gate 1** | profile done and requirements written, **before any join** |
| **Gate 2** | headline claim stated, **before it's polished** |
| **Gate 3** | cold run passes, **before the video** |

Kept beside every layer: the **approach log** (two options per part, the one chosen, why), the **challenges log** (what broke, what you tried, what changed) and the **assistant log** (every use, one line).

## 7. Working plan: 3–4 days in four phases

| Phase | Share of hours | Work | Out |
|---|---|---|---|
| 1 Scope | 10% | translate the brief, write requirements, predict the answer, open the repo and logs | translation note, requirements table, prediction, approach-log row 1 |
| 2 Profile and clean | 30% | profile every table before touching it; decide, log, reconcile; join with row counts | decisions log, passing reconciliation checks, first honest totals |
| 3 Analyse | 35% | run the brief's method; state the claim early and spend the rest of the time testing it | headline with denominator and caveat; approach log complete |
| 4 Answer and defend | 25% | six slides, cold run, rehearsal, write-ups, video | repo frozen, video linked, every member can present any slide |

**Checkpoint questions at every phase boundary:**
1. What is the headline number right now, with its denominator?
2. What did we reject since the last checkpoint, and why?
3. What would make us wrong, and what are we doing about it?

## 8. How to think

**First principles:**
1. Test volume is a count of tests. A test needs a booking, a collected sample and a result. Which one does your table record?
2. A growth rate is two numbers and a period.
3. Nothing is true because the header says so. It's true when two tables agree.

**Ask yourself:**
1. What am I taking as true without checking?
2. What is the denominator?
3. What would I see if I were wrong?
4. Who loses if I'm right?
5. Which number am I surest of, and why?
6. What's the cheapest way to find out?
7. Why would another group get a different answer?
8. What did Dr Menon not ask?

**Five whys (an illustration of the method):** 13 points short → volume grew little → nobody split it into patients, visits and tests → the dashboard reports one number on last year's mix → the board asked for one number → the question only hurts when the number is bad. Write your group's own chain before opening the data.

**Approach log** (`docs/approach-log.md`): one row per part (profiling, cleaning, each join, each comparison, each chart) with the columns *Part, Approaches considered, Chosen, Why this and not the others, What would change the choice, What happened*. Write it when the choice is made: the commit time is the proof. A rejected approach you tried for 20 minutes is the best kind of entry, so keep the dead cell. The panel will cover the "chosen" column and ask you to choose again aloud. Worked example: invoice id to payment reference, (a) strip the prefix, (b) match amount and date within 2 days, (c) both, flagging disagreements. They chose (c), which matched 96%, and the rest became finance's list.

## 9. The defence

**Answer shape:** **Claim** (one sentence) → **Evidence** (numbers, each with its denominator) → **Caveat** (the one thing that would change the claim) → **Action** (what to do, what it costs). "Not yet, and here is what would tell us" is a complete answer when it's true.

**Maturity ladder:** (1) you report a number, and get asked "compared with what?"; (2) you explain what moved it ("how sure are you?"); (3) you say how sure you are, and why ("so what should I do?"); (4) you recommend, and price being wrong. **Open at step 3, and reach step 4 when pressed.**

**Presentation slot (25–30 minutes):** about 10 minutes for six slides (any member may present any slide); about 5 minutes for a **live demo from a fresh Codespace running the notebook that produces the headline**; 10–15 minutes of questions to the group, then to individuals, then to the quietest person.

**Six slides:**
1. The question: Dr Menon's words and your translation.
2. The picture: your completed diagram, numbered.
3. The claim: one sentence, nothing else.
4. The evidence: two or three numbers with denominators, one chart.
5. Caveat and alternative: what would change the claim, and the approach you rejected.
6. The action: what to do, what it costs, what being wrong costs.

**Video:** one per group, linked from the README before the slot. 4 members: about 20 minutes (5 each). 3 members: 20–30 minutes (7–10 each). Solo: about 20 minutes. If the group can't record together, each member records 15 minutes on their own part.
- *Spend minutes on:* the question and your prediction; one decision, its rejected alternative and the log row; one trusted number, its denominator and what would break it; one thing that broke (the challenges entry); what you'd do with two more days.
- *Don't spend minutes on:* reading slides aloud, scrolling code, syntax, installs, recapping the brief, or what a teammate covered.
- Screen and face, with name and part on the first frame. Record after the freeze. Say numbers slowly, with denominators.

**Individual write-up** (`docs/writeups.md`): 10 questions, **600–900 words, your own words, no assistant**. The file opens with a two-paragraph group summary, then one section per member, each committed by its author.
1. What was Dr Menon asking for, and what did she actually need?
2. Which part was yours, and what did you produce that the group depends on?
3. Which approaches did you consider, which did you choose, and why not the others?
4. Which number do you trust most, what's its denominator, and what would make you stop trusting it?
5. What broke, what did you try, what did you change?
6. What do you understand about a diagnostics business now that you didn't before?
7. Where did the team work well, where not, and what changes for Build 2?
8. Which Week 1 or 2 idea do you understand better after using it on this data?
9. Which viva question would you least like, and what's your answer to it?
10. With two more days, what would you do first, and why?

## 10. Rules and grading

1. **Group:** 3–4 people, one brief, one public repo. Any member may be questioned alone about any number.
2. **No assistants on the work:** not the notebooks, not the write-ups, not the video script. Every assistant use is logged.
3. **If generated work is found:** you're questioned on every line. A line you can't explain is a reprimand for the group, and the mark follows the handbook.
4. Expect "why this and not that" at any point.
5. **Graded:** Mini Project 1 is **40 marks** (implementation plus your ability to explain it, including the presentation and video). Mock 30, group discussion 30, out of 1000.
6. **Hand in:** notebooks, data as received, the three logs, write-ups, six slides, the video link.
7. **Freeze:** the evening before presentations. Video link and write-ups go in the README before your slot. Nothing merged after the freeze counts.
8. **Tools:** VS Code notebooks on Codespaces, SQL for joins, Excel for the board number, GitHub for everything.

**Assistant use (Appendix C):**
- *Allowed, and logged:* explaining an error you've already tried to read; explaining a concept you then rewrite in your own words; checking English in the write-up without changing its meaning.
- *Not allowed:* generating analysis, a notebook, a cell, a chart, a slide, a write-up or a script; asking what the answer or approach should be; **pasting Kalpa Health data into it**.
- Log every use in `docs/ai-log.md`, one line each (date, who, what, what you did with it). "Twenty honest lines is fine. An empty log next to tutorial-shaped notebooks is not."

## 11. Appendices in brief

- **A · GitHub:** repo name `c2-build1-brief-<letter>-group-<n>`, public. Commit one is the README (brief, members and parts, how to run, Links for the video). Create the folders docs/notebooks/data/src/tests/slides in the first hour. **Protect main**, so everything arrives by an approved PR. data/raw goes in as received with a provenance README, and files over 100 MB are linked, not committed. Workflow: pick a task → branch `feature/...` → micro-commits → PR (what, why, how to check) → review (run it, ask one viva question) → merge commit (not squash), then delete the branch. Commits: one change describable in one line, in the imperative ("Reject 14 duplicate booking rows; log the rule in decisions-log.md"), never "update" or "FINAL". Commit when something runs that didn't, or every 30 minutes. Clear outputs before committing, and keep one owner per notebook.
- **B · Team:** rotate four roles: Driver (writes), Navigator (reads every cell and predicts its output), Scribe (keeps the three logs), Challenger (asks why five times). Work in three modes, one at a time: Plan (15 minutes, no code), Build (one branch, one task, one owner), Review (15 minutes, checkpoint answers). Task board: Backlog (written as results) → Doing → Review (PR open) → Done (merged, logged, run by someone else). On disagreements, log both approaches, try the cheaper one for 30 minutes, and let the result decide.
- **C · Testing:** three tests are worth writing. **Reconciliation** (`len(raw) == len(clean) + len(rejected)`, and revenue before = after + removed), **joins** (row count before and after, and keys matched more than once), and the **daily cold run** (Restart and Run All, record the headline, and if it moved, find out why before going home). Spend about a tenth of your time on tests: five pytest cases per shared helper, and don't test pandas or charts. Research order: your notes → documentation → industry investor decks → each other → the internet (exact error in quotes, note the link).
- **D · Transfer:** the same five questions elsewhere: A, a gym (members × visits × spend); B, food delivery after a payment-provider switch; C, school fees demanded vs received, with an instalment timing bar; D, the smallest courier hub with the worst failure rate; E, a streaming free month that "lifted sign-ups 11%". Add your own example per brief.

**Before you record, check that:**
- a teammate ran everything from a fresh Codespace and got the slide-3 number;
- the task board matches reality;
- every PR was approved by someone else;
- every member has commits on all three logs;
- each write-up section is in its author's commits;
- every approach-log row has two approaches;
- no trainer, mark or outside student is named anywhere.

---

## 12. How this repository maps to the handbook

| Handbook item | Where it lives here |
|---|---|
| Profile (Gate 1) | `notebooks/01_profile.ipynb` |
| Load and validate, input = kept + rejected | `notebooks/02_load_parallel_validate.ipynb` |
| Clean and reconcile, join row counts | `notebooks/03_clean_reconcile.ipynb` |
| SQL for joins, the query for finance | `notebooks/04_sql_transformations.ipynb`, `sql/` |
| The five methods (A tree, B ladder, C bridge, D chance reference, E fair comparison) | `notebooks/05_methods.ipynb` |
| Excel board number, cold run (Gate 3), headline in the last cell | `notebooks/06_answer_cold_run.ipynb` |
| Approach, decisions, challenges, assistant logs | `docs/approach-log.md`, `docs/decisions-log.md`, `docs/challenges-log.md`, `docs/ai-log.md` |
| Project statement and requirements (Scope) | `docs/project-statement.md`, `docs/project-charter.md` |
| Write-ups | `docs/writeups.md` |
| Slides and board number | `slides/` |

**Data-pack facts that bear directly on the briefs**, from the profiling and cleaning runs already in this repo:
- The new booking vendor is **nova**, used in **Chennai and Jaipur from April 2026**. Legacy covers every city before that.
- **The rescheduled-appointment rule** (brief B's "midnight rule"): nova writes a new booking and marks the old one `rescheduled`, while legacy overwrote the slot. The phlebotomy app logs the collection against the **original** id.
- The payment gateway's references match the invoice format exactly in only about 70% of rows, and in 100% after normalisation.
- The campaign in the pack is `HC-FREE-2026Q1`, free home collection from January to March 2026, with a Rs 150 waiver per booking.
- The plan file is `plan/plan_2026_H1.csv`, wide by month, and uses the spelling "Bangalore".

Still to fill for your group: the **brief letter** and **group number** in every notebook's CONFIG, and in `docs/` and the README.
