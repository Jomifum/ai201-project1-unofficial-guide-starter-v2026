# The Unofficial Guide

<!-- Replace this line with your name and which corpus you picked. -->

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This project builds a small retrieval-augmented QA system for the campus_life corpus, which contains student-written advice and logistics posts about classes, housing, dining, and campus policy. The app ingests text files, splits them into chunks, embeds them, retrieves the closest matches to a question, and then answers only from those retrieved documents. The goal is to give grounded, source-backed answers to concrete student-life questions while refusing questions that clearly fall outside the corpus.

## Chunking Strategy

**Chunk size:** 550 characters
**Overlap:** 80 characters

The campus_life corpus is made of short student posts, and the useful fact is often a single sentence rather than a long passage. I kept the chunks large enough to retain the full thought but small enough that a single answer does not get buried in unrelated text. The shorter documents mostly stay intact, while longer ones still split on sentence boundaries instead of cutting randomly through a paragraph.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_biol_160.txt` — produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved. Expect 9 to 11 hours a week, the heaviest first-year course by reputation. The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_hist_118_workload.txt` — produced by: `chunker.py::split_documents`

```
Workload for HIST 118 Modern World History People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time. It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_pellew_dining_hall_followup.txt` — produced by: `chunker.py::split_documents`

```
Re: Pellew Dining Hall Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely. Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_innisfree_hall.txt` — produced by: `chunker.py::split_documents`

```
Innisfree Hall — what it's actually like Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms. The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus. The bad: no air conditioning, which matters for the first three weeks of September. Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** is the housing lottery random?

**Answer:**

```
The housing lottery is not entirely random in the way most people assume. While rising sophomores get a number drawn at random, juniors and seniors are ordered by accumulated credit hours first, and random tie-breaking is only used for ties.

Source: admin_housing_lottery.txt
```

**My relevance cutoff:** 0.7

I used the actual retrieval distances from my five in-corpus questions and my five out-of-scope questions to place the cutoff in the gap. In-scope distances ranged from about 0.15 to 0.50, while out-of-scope questions stayed between about 0.82 and 0.93, so 0.7 sits safely between the two groups and rejects unrelated questions without blocking the answers this corpus actually contains.

| Question | In corpus? | Best distance |
|---|---|---|
| Is the housing lottery random for juniors and seniors, or are they sorted by credit hours first? | Yes | 0.1508 |
| Through what week can a student add a course, and when does a drop after week two show up on the transcript? | Yes | 0.1788 |
| When do student parking permits for the west lots go on sale, and how quickly do they sell out? | Yes | 0.1790 |
| In CS 210, what percentage of the grade comes from labs, and what advice do students give about doing them? | Yes | 0.3852 |
| How far is the east parking lot from campus, and why do some students prefer it even though it is farther away? | Yes | 0.4995 |
| What is the capital of Mongolia? | No | 0.8246 |
| How do I change the oil in a diesel engine? | No | 0.9340 |
| Who won the 1994 World Cup? | No | 0.8859 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8442 |
| How do I write a for loop in Rust? | No | 0.8960 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** I asked an AI model to help me design a chunker for the campus_life corpus after I noticed the default fixed-size splitter was leaving entire posts unsplit. The first version it suggested used sentence breaks but still kept a fixed 800-character window, which meant it could cut through a sentence and still leave unrelated details in the same chunk. I changed the strategy to keep short posts intact, split longer content on sentence boundaries, and add a modest overlap so neighboring chunks still share enough context. The final code in chunker.py reflects the result of that adjustment.

**2.** I asked an AI assistant to help me choose a relevance cutoff by comparing the distances of in-corpus and out-of-scope questions. It proposed a number that looked reasonable but was too close to the in-corpus cluster, so I checked the actual retrieval distances from my five test questions and five out-of-scope questions and placed the cutoff in the gap between the two groups. That led me to set the threshold to 0.7 instead of leaving the starter's 0.6 in place, and it improved the gate behavior without rejecting valid campus-life questions.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
