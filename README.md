<!-- Nelly Nyauntu — corpus: campus_life -->


# Unit 1

## What This Does
This system answers questions about student life in college using the campus_life corpus — 88 short posts covering housing, dining, courses, and admin topics. It answers specific questions like whether the housing lottery is random or how loud a particular dorm is, always citing the source file it pulled from.

## Chunking Strategy
Paragraph-based, with an 800-character safety cap and 100-character overlap for any paragraph exceeding it.

I chose this because campus_life's documents are short, self-contained posts, but several (like housing_fenwick_court.txt) contain multiple distinct sub-topics separated by paragraph breaks. Splitting on paragraph breaks keeps each thought intact without cutting mid-sentence, and doesn't over-split posts that are already a single paragraph. In practice, campus_life's longest document is 549 characters, so the 800-character cap never activates — every chunk here is a natural paragraph.

Re-indexing with this chunker turned 88 documents into 271 chunks (up from the fallback's 88), confirming most documents do contain multiple paragraphs.

## Sample Chunks

**Chunk 1** — source: `course_biol_160.txt#1` — produced by: `chunker.py::split_documents`
I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

**Chunk 2** — source: `dining_halden_hall.txt#1` — produced by: `chunker.py::split_documents`
I lived here my sophomore year. Wait times: rarely more than 8 minutes, even at noon. The thing worth going for is soup rotation, and the bread is baked on site. The thing to know is that closes at 7:00pm, which catches people out.


**Chunk 3** — source: `housing_fenwick_court_laundry.txt#1` — produced by: `chunker.py::split_documents`

Machines take $2.00 wash, $1.75 dry, app-based. There are eight washers and six dryers for the building, which is the wrong ratio and means the dryers back up on Sunday evenings.


**Chunk 4** — source: `housing_morrow_house.txt#2` — produced by: `chunker.py::split_documents`

The good: cheapest housing tier by about $900 a year, and the singles are real singles.


**Chunk 5** — source: `money_textbooks.txt#2` — produced by: `chunker.py::split_documents`

The campus store price-matches, which is not advertised anywhere and you have to ask at the counter with the other listing on your phone.


**Known limitation:** a few documents follow a Title/Body structure where the title (e.g. "On the add/drop deadline") is short enough to end up as its own paragraph and its own chunk, with no content. In a 15-chunk sample, 3 of 15 (20%) were heading-only fragments. A fix — merging any paragraph under 60 characters into the one that follows — was identified but not yet implemented.

## Sample Answer

**Question:**
Is the housing lottery based on random chance for everyone?

**Answer:**
No, the housing lottery is not random for everyone. While rising sophomores have a random number drawn, juniors and seniors are ordered by accumulated credit hours first, with random selection used only as a tie-breaker (admin_housing_lottery.txt). Source: admin_housing_lottery.txt

Note: this prompt's retrieved chunks included "On the housing lottery" — the heading-only fragment described in the Known Limitation above — alongside 3 other irrelevant chunks pulled in by top-k=5. Despite that noise, the model correctly grounded its answer only in the relevant document and ignored the rest, confirming the grounding instruction works even when retrieval returns some off-topic material.'

**My relevance cutoff: 0.6** (the starter default)

I measured the best retrieval distance for all 5 of my test questions and all 5 OUT_OF_SCOPE questions. My worst in-scope distance was 0.4066 (CS 340 exam format) and my best out-of-scope distance was 0.7803 (1994 World Cup) — a clean gap of about 0.37 with no overlap between the two groups. 0.6 sits comfortably in the middle of that gap, so I kept the starter's default rather than adjusting it.

| Question | In corpus? | Best distance |
|---|---|---|
| Is the housing lottery based on random chance for everyone? | Yes | 0.2476 |
| When is the best time to do laundry in Fenwick Court? | Yes | 0.2304 |
| How is CS 340 graded - how many exams and are they open-book? | Yes | 0.4066 |
| What's the price for a wash and dry cycle in Fenwick Court's laundry room? | Yes | 0.2723 |
| How often does the campus shuttle run on weekdays? | Yes | 0.3851 |
| What is the capital of Mongolia? | No | 0.7986 |
| How do I change the oil in a diesel engine? | No | 0.8502 |
| Who won the 1994 World Cup? | No | 0.7803 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8243 |
| How do I write a for loop in Rust? | No | 0.8313 |

## How I Used AI

1. I asked Claude to help me measure my relevance cutoff for Milestone 4 by running python app.py retrieve on all 5 of my test questions and all 5 OUT_OF_SCOPE questions. It reported my worst in-scope distance as 0.4066 and my best out-of-scope distance as 0.7803. I checked this myself against the actual terminal output rather than just trusting the summary, and confirmed the starter's default of 0.6 genuinely sits in the middle of that 0.37-wide gap, so I kept it rather than changing it.

2. I asked Claude to write a custom chunker for Milestone 3, since campus_life's documents are short, self-contained posts. It produced a paragraph-splitting function with an 800-character safety cap and 100-character overlap for any oversized paragraph. When I sampled 15 chunks to check quality, I found 3 were bare headings ("On the add/drop deadline", "On the parking permits") with no real content  because those documents put the title on its own line, separated from the body by a blank line, and my chunker splits on exactly that blank-line boundary. Claude proposed a fix (merging any paragraph under 60 characters into the next one), but I chose to document it as a known limitation in my README instead of implementing it immediately, since it wasn't blocking my five test questions from getting correct answers.

     Milestone 5. -->

**1.**

**2.**

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2


## Run Log — Before


| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunk quality (complete thought, no chunk under 50 chars) | 4 of 5 + no chunk under 50 chars | 4/5, one chunk at 25 chars | 4/5, one chunk at 25 chars | 4/5, one chunk at 25 chars | MISSED |
| 5. Correct source attribution on duplicate content | 4 of 5 such cases | 2/2 such cases found | 2/2 such cases found | 2/2 such cases found | Unmeasurable as written |

**Criterion 1 & 2 example** (source: `results/run_2026-09-30_1347.md`, produced by `generate.py::answer_from_chunks`):

Question: Is the housing lottery based on random chance for everyone?

Answer: No, the housing lottery is not random for everyone. While rising sophomores have a random number drawn, juniors and seniors are ordered by accumulated credit hours first, with random selection used only as a tie-breaker (admin_housing_lottery.txt).

**Criterion 3** (produced by `run_eval.py::check_out_of_scope`): gate refused 5 of 5 out-of-scope questions, best distances ranging 0.780-0.850, all above the 0.6 cutoff.

**Criterion 4** (produced by `chunker.py::split_documents`, via `app.py chunks -n 5`): Chunk 1, source admin_add_drop_deadline.txt#0: "On the add/drop deadline" (25 characters — fails the 50-char floor).

**Criterion 5**: Q2 retrieved laundry files from 4 different buildings but correctly cited housing_fenwick_court_laundry.txt. Q5 retrieved 3 buildings with different prices ($2.00/$1.75, $1.75/$1.75, $1.50/$1.50) but correctly cited housing_fenwick_court.txt's $2.00/$1.75.

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | All 5 questions had their correct source among the top-5 retrieved chunks in all 3 runs. Retrieval is deterministic, so this doesn't vary by run. |
| 2 | Every answer names a source | MET | All 15 generated answers (5 questions x 3 runs) name their source file, either inline or as a separate line. This is structurally guaranteed by the grounding instruction and chunk metadata. |
| 3 | Gate stops out-of-corpus questions | MET | All 5 OUT_OF_SCOPE questions were refused, with best distances (0.780-0.850) well above the 0.6 cutoff, in the single deterministic gate pass. |
| 4 | Chunk quality (complete thought, no chunk under 50 chars) | MISSED | 4 of 5 sampled chunks read as complete thoughts, but Chunk 1 ("On the add/drop deadline") is 25 characters, breaking my own 50-character floor. My criterion joined both conditions with "and," so one violation misses the whole target. |
| 5 | Correct source attribution on duplicate content | Unmeasurable as written | Only 2 of my 5 test questions actually involve duplicate-content ambiguity, not 5. Both of those 2 cases correctly attributed the right building-specific source. My target assumed a denominator of 5 that doesn't exist in my test set. |

 
## Diagnoses

Criterion 4 (Chunk quality) — MISSED

Stage: chunking.

Mechanism: `chunker.py::split_documents` splits on blank-line paragraph breaks. Several documents in campus_life (e.g. admin_add_drop_deadline.txt) are written as a short title line, then a blank line, then the body paragraph. Because the title line sits above a blank line just like a real paragraph break, my chunker treats it as its own separate paragraph and therefore its own chunk — even though a bare title like "On the add/drop deadline" (25 characters) can't answer any question on its own.

This is a chunking-stage problem, not a retrieval or generation problem: the fragment chunk gets created before retrieval ever runs, and it doesn't cause wrong answers (Criterion 1 and 2 both still passed 5/5) because retrieval still finds the real content chunk alongside the fragment. But it does violate my own quality bar, and it wastes one of the 5 chunk slots retrieval could otherwise use.

Pattern check: I sampled a wider set (15 chunks) back in Unit 1 and found this same issue in 3 of 15 (20%), so this isn't a one-off. It's a systematic property of any document following the Title/Body structure.

Criterion 5 (Correct source attribution on duplicate content) — not a miss but measurement problem

This isn't a pipeline failure. The system actually performed correctly on both relevant test cases (2 of 2). The issue is with how I wrote the criterion: I set a target of "4 of 5 such cases" without checking how many of my actual 5 test questions would even qualify as "such cases." Only 2 do. 

## The Improvement

What I changed: 
In `chunker.py::split_documents`, I added logic to merge any paragraph under 60 characters into the paragraph that follows it, before chunking. This catches heading-only paragraphs (like "On the add/drop deadline") and attaches them to their body text instead of leaving them as standalone fragment chunks.

Why I picked it: 
This directly targets my Criterion 4 diagnosis: the chunker was splitting on blank lines, and several documents have a short title line separated from their body by a blank line, so the title became its own useless chunk. This was confirmed in 3 of 15 sampled chunks (20%) before the fix.

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunk quality (complete thought, no chunk under 50 chars) | 4 of 5 + no chunk under 50 chars | 5/5, shortest 61 chars | 5/5, shortest 61 chars | 5/5, shortest 61 chars | MET |
| 5. Correct source attribution on duplicate content | 4 of 5 such cases | 2/2 such cases found | 2/2 such cases found | 2/2 such cases found | Still unmeasurable as written |

Did it help?
Yes, measurably. Criterion 4 flipped from MISSED to MET: re-sampling 5 chunks after the fix showed all 5 reading as complete thoughts, with the shortest chunk now 61 characters (above my 50-character floor), compared to the 25-character fragment before. Chunk count dropped from 271 to 177, since fragment paragraphs merged into their neighbors rather than standing alone. Criteria 1-3 stayed MET with no regression, and Criterion 5 is unchanged (expected, since this fix targeted chunking, not the criterion's wording problem).

## What's Still Broken

Criterion 5 still can't really be measured. My fix was about chunking, so it didn't touch this problem at all. Only 2 of my 5 test questions actually involve duplicate content across buildings, so I can't fairly say "4 of 5" when there are only 2 cases to check. I didn't fix this now because the assignment's rule for this unit is one change only, and this isn't a code problem anyway — it's a problem with how I wrote the criterion. If I kept going, I'd add 2-3 more test questions that involve duplicate content, so I'd actually have 5 real cases to check.

## What I'd Do Differently

I'd write Criterion 5 differently from the start. The idea behind it is still good: check that the system picks the right source when several documents look similar. But I picked the number "4 of 5" before checking whether my test questions would even give me 5 cases like that. Next time I'd write it more like "for every question in my test set that has this duplicate-content problem, the system should pick the right source" instead of locking in a number before I knew how many such questions I'd actually have.