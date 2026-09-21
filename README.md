# The Unofficial Guide

Khanh Do — Corpus: `campus_life`

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
The Unofficial Guide is a RAG system built using the `campus_life` corpus, which contains short posts about courses, dining, housing, and other parts of student life. It searches these documents to answer specific questions such as dining wait times, course exams, housing rules, and campus policies. The system retrieves relevant chunks and uses them to generate an answer that names its source. If the retrieved information is not relevant enough, the relevance gate refuses to answer instead of using unrelated information.

## Chunking Strategy

**Chunk size:** 400 characters maximum

**Overlap:** 0 characters

I chose this strategy because the `campus_life` corpus contains short posts that average about 317 characters, and most useful information is contained in one or two sentences. Instead of cutting text at an arbitrary character position, my chunker keeps paragraph boundaries intact and combines neighboring paragraphs until adding another paragraph would exceed 400 characters.

My first attempt made every paragraph its own chunk. This produced 271 chunks with an average length of only 101 characters and some chunks as short as 10 characters, including headings with no useful information. I changed the strategy so short headings stay together with the paragraphs that follow them. The final version produced 100 chunks averaging 278 characters, with the shortest chunk at 94 characters.


## Sample Chunks

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```text
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_cs_210.txt#0` — produced by: `chunker.py::split_documents`

```CS 210 Data Structures
I'm a junior and I've done this twice now. Format is lecture with weekly labs; slides go up after class, not before. Assessment: two midterms and a final, all drawn from lecture material rather than the textbook. Midterms are curved, the final is not.

Expect 8 to 10 hours a week outside class.
```

**Chunk 3** — source: `course_math_220_workload.txt#0` — produced by: `chunker.py::split_documents`

```Workload for MATH 220 Linear Algebra
People keep asking so: 6 to 8 hours a week, almost all of it on problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_the_ridgeway_cafe_followup.txt#0` — produced by: `chunker.py::split_documents`

```Re: The Ridgeway Café
Adding to what people have said about The Ridgeway Café. The wait figure of 10 to 15 minutes at 12:30 matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: seating is tight; about 40 seats for a building of 900. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_morrow_house.txt#0` — produced by: `chunker.py::split_documents`

```Morrow House — what it's actually like
Just finished a year in this building. Built 1954, partially renovated 2008. Rooms are singles and doubles, hall bathrooms.

The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** How are juniors and seniors ordered in the housing lottery?

**Answer:**

```Juniors and seniors are ordered by accumulated credit hours first, with a random tie-break used if needed.

Source: admin_housing_lottery.txt
```

**My relevance cutoff:** 0.6

I kept the relevance cutoff at 0.6 because the five questions covered by my corpus had best distances between 0.204 and 0.356, while the five out-of-scope questions had distances between 0.825 and 0.934. There was a large gap between the two groups, so 0.6 separates them without rejecting any of my five test questions.


| Question | In corpus? | Best distance |
|---|---|---:|
| How are juniors and seniors ordered in the housing lottery? | Yes | 0.225 |
| How long are the wait times at Kestrel Commons between 12:15 and 1:00? | Yes | 0.223 |
| What material are the CS 210 exams based on? | Yes | 0.300 |
| Which floors in Aldridge Hall are quiet floors? | Yes | 0.356 |
| Do dining dollars roll over from spring to the following autumn? | Yes | 0.204 |
| What is the capital of Mongolia? | No | 0.825 |
| How do I change the oil in a diesel engine? | No | 0.934 |
| Who won the 1994 World Cup? | No | 0.886 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.844 |
| How do I write a for loop in Rust? | No | 0.891 |
|  |  |  |

## How I Used AI

**1.** 
I asked AI to help me understand and design the chunking strategy. My first version made every paragraph a separate chunk, which produced 271 chunks averaging only 101 characters and created very short heading-only chunks. After inspecting the output, I changed the approach to combine neighboring paragraphs up to 400 characters, which reduced the result to 100 more complete chunks.


**2.**
I asked AI to explain semantic search, top-k, distance, and the relevance cutoff because I did not understand how they related to retrieval. I then tested five in-corpus and five out-of-corpus questions myself. The in-corpus distances were between 0.204 and 0.356, while the out-of-corpus distances were between 0.825 and 0.934, so I kept the existing cutoff of 0.6 instead of changing it unnecessarily.

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
