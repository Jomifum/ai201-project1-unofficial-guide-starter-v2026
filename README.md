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

### Stretch feature: metadata filtering

I implemented a source filter so retrieval can be narrowed to one document at a time. This is useful when a question is about a single campus policy or housing file and I want to see whether the matching evidence is confined to that source instead of mixing in unrelated documents.

Example without the filter:

```bash
python app.py retrieve "is the housing lottery random?"
```

Example with the filter:

```bash
python app.py retrieve "is the housing lottery random?" --source admin_housing_lottery.txt
```

The filtered run only returns chunks from the housing lottery document, which makes the source boundary explicit; the unfiltered run pulls in nearby but less specific results from other campus-life files. The filter narrows the search space without changing the question itself, which is the observable effect I wanted.

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

The project ran against the five questions in [questions.py](questions.py) and the gate against the five out-of-scope prompts. The evidence file is in [results/run_2026-09-20_2327_before.md](results/run_2026-09-20_2327_before.md). The before run showed that all five in-scope questions passed the retrieval gate and all five out-of-scope questions were refused.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Sampled chunks stand alone as complete thoughts | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 5. Answers include the specific fact asked for | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |

The run produced real output in the results file. For example, the first question was answered from the housing-lottery document, and the gate refused all five out-of-scope questions. The evidence file names the source and the function that produced the chunks (`chunker.py::split_documents`). Because `run_eval.py` has no `scorer.py` in this project, the pass counts below are manual judgments from the recorded answers, not automated verdicts.

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | All five in-scope questions retrieved a chunk containing the answer, and the best distance was always under the cutoff. |
| 2 | Every answer names a source | MET | Each recorded answer identified a real source, either with a `Source:` line or an inline parenthetical such as `admin_housing_lottery.txt`. |
| 3 | Gate stops out-of-corpus questions | MET | All five out-of-scope topics were refused by the gate, with best distances between 0.825 and 0.934. |
| 4 | Sampled chunks stand alone as complete thoughts | MET | The five sample chunks printed by `python app.py chunks -n 5` each read as a complete thought and could be answered from without surrounding context. |
| 5 | Answers include the specific fact asked for | MET | Four answers supplied the requested fact directly. The east-lot answer supplied the 12-minute walk and correctly said the documents did not support the question's claim about why students prefer it, rather than inventing a reason. |

## Diagnoses

There were no misses. The system met all five of the targets in the before run, so the main issue was not a pipeline failure; it was usability and observability. I still added a metadata filter as a stretch feature so retrieval could be narrowed to a single source document when the user wanted a more focused search.

## The Improvement

**What I changed:** I added a `--source` filter to retrieval and asking, and I documented it as a stretch feature in the README. The implementation is in [store.py](store.py) and the CLI is in [app.py](app.py).

**Why I picked it:** This was a source-level narrowing feature rather than a fix for a failed criterion. It makes the search more precise when a user wants to restrict the answer to one document, and it is measurable in a direct before/after check with the same question and a different source filter.

### Run Log — After

The after log uses the same questions, cutoff, corpus, and three-run format because the improvement was a source-narrowing usability feature rather than a change to the core retrieval threshold. I created the after evidence file in [results/run_2026-09-20_2331_after.md](results/run_2026-09-20_2331_after.md). The script reports blank automated verdict columns because this project does not include `scorer.py`; the criterion verdicts below are manual reviews of the actual answers in that file.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 2. Every answer names a source | 5 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 4. Sampled chunks stand alone as complete thoughts | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |
| 5. Answers include the specific fact asked for | 4 of 5 | 5 of 5 | 5 of 5 | 5 of 5 | MET |

**Did it help?** Yes, for source-specific exploration: the filtered housing-lottery query returned only `admin_housing_lottery.txt`, while the unfiltered query returned five nearby matches. It did not materially change the core quality metrics because it was a usability feature, not a change to chunking, retrieval cutoff, or generation. The after run still retrieved all five in-scope answers under the 0.7 cutoff and refused all five out-of-scope questions.

## What's Still Broken

No criterion is still missed in the measured before/after evaluation. The remaining limitation is that the project does not include a custom `scorer.py`, so the run logs contain raw answers rather than automated pass/fail verdicts; the criterion judgments here are manual and traceable to those recorded answers.

## What I'd Do Differently

If I were writing this again, I would keep the same five criteria and the 0.7 cutoff because they reflect the actual campus_life corpus and the measured distance gap. I would add `scorer.py` earlier so the next evaluation could record automated verdicts as well as raw answers, while still reviewing grounded refusals such as the east-lot answer manually.
