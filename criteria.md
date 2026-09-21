# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
This corpus contains short, direct student posts, so the useful fact is often a
single sentence. If the retrieval is working, at least four of the five
questions should land on a chunk with the answer in it without needing a broad
search.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
The project asks for source attribution as a core behavior, and this corpus has
many short documents that can be traced back to one file. I want the answer to
be tied to a real document in every case, because a citation-free answer would
not be reliable enough for a student-facing guide.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

**Why this target:**
My corpus is about campus life and university logistics, so a question about
Mongolia, car maintenance, or Rust is far outside the scope of the documents.
A cutoff that blocks at least four of those five should be strict enough to
keep the system from making up an answer from unrelated material.

---

## 4. Something about your chunks

At least 4 of 5 sampled chunks should read as a complete thought, with no
sentence broken in half at either end.

**Why this target:**
The campus_life documents are short posts where the useful information is often a
single sentence. If a chunk cuts through a sentence or leaves a half-finished
thought, the answer becomes harder to trust and the retrieval system is less
reliable.

---

## 5. Your choice

At least 4 of 5 answers should include the specific fact the question asks for,
not just a vague paraphrase of the topic.

**Why this target:**
The questions are deliberately precise, and the answers in this corpus are often
fact-based rather than open-ended. I care about correctness of the actual detail
— like a week number, a time, or a percentage — more than about giving a broad
but unhelpful summary.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
