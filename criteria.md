# Acceptance criteria - The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1 **before** any results existed.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that contains the answer.

**Why this target:** I expect this to be achievable because campus_life documents are short and single-topic. I'm allowing for 1 miss because the CS 340 exam detail is one specific fact within a short document, which could get diluted if retrieval pulls a less relevant chunk instead.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:** I'm targeting all 5, not 4, because every chunk carries its source filename as metadata from ingest.py through store.py, and generate.py's grounding instruction requires the model to name the file. Since this is enforced structurally rather than left to model judgment, I expect it to hold every time.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate stops it and the system returns "I don't have enough information about that" in at least 4 of 5 tries.

**Why this target:** I haven't measured my actual distance gap yet, that happens in Milestone 4. I expect a clean separation because my OUT_OF_SCOPE questions share no vocabulary with campus_life's content. I will replace this reasoning with my actual measured distances after Milestone 4.

---

## 4. At least 4 of 5 sampled chunks read as a complete thought

At least 4 of 5 sampled chunks read as a complete, self-contained thought, answerable without reading the paragraph before or after it. No chunk is shorter than 50 characters.

**Why this target:** I picked 4 of 5 because a few documents, like housing_fenwick_court.txt, repeat information already covered in dedicated companion files, so a chunk from the overview might read as slightly redundant rather than incomplete.

---

## 5. Correct source attribution, not just any source

When a question is answerable from more than one document, for example laundry info appearing in both housing_fenwick_court.txt and housing_fenwick_court_laundry.txt, the system cites at least one genuinely relevant source in at least 4 of 5 such cases.

**Why this target:** I care about this because duplicate content across companion files makes it easy for retrieval to look successful by accident, pulling back a source, but not necessarily the right one.