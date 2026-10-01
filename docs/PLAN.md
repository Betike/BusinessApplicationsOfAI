# Team E – Project 1 plan: LLM-supported Course Recommender

## What we have to deliver
- **Project 1 (30% of the grade):** an LLM-supported recommender for a chosen domain (ours: **online courses**). It
  recommends items from user preferences **and explains each recommendation**.
- **Project 2 (30%):** deployment economics of the *same* system (usage volume, token/API cost, model choice,
  business value). That's why we log tokens and latency from day 1 and keep the model swappable.
- **Submission:**
  - a reproducible `.ipynb` that runs top to bottom, with no API keys in any file
  - a ≤2-page summary PDF covering problem, data, method, evaluation, results, limitations and **which AI tools we used**
  - a ≤6-slide deck and an 8–10 min demo, plus 3–5 min Q&A. **Everyone must be able to explain every part.**
- **Grading (100 pts):** problem definition 15, technical implementation 20, LLM methodology 15,
  **evaluation & metrics 20**, business relevance 10, risks & responsible use 10, presentation 10 (+5 bonus).
- Rules: free models only; public or simulated data only; no PII sent to the LLM.

## Business framing
- **Client:** an online learning platform (or a university learning/career centre) with thousands of courses.
- **Problem:** learners describe goals in natural language. Keyword search and "most popular" lists serve them
  poorly, so they drop off or enrol in courses at the wrong level.
- **Solution:** a recommender that returns the top 5 catalog courses, each with a 1–2 sentence explanation tied to
  the learner's needs.
- **KPIs:** enrolment conversion, completion rate, advisor workload. These need an A/B test, which is future work.
  Here we use offline proxy metrics.

## System design
```
request + optional profile
 ├─ [LLM 1] preference extraction → JSON {in_scope, goal, topics, skills, level, min_rating, exclude_providers, search_query}
 ├─ [Retrieval] hard filters (level, rating, providers, completed courses) + semantic similarity
 │              (all-MiniLM-L6-v2, local & free; TF-IDF fallback) → top-20 candidates, filters relaxed if too few
 ├─ [LLM 2] rerank & explain: top-5 ONLY from candidates, JSON with explanations, or "no good match"
 └─ [Guardrails] schema validation, invented IDs dropped and counted, retry → fallback, prompt-injection
                 delimiting + canary check, polite refusal for out-of-scope requests
```
- **LLM access:** Gemini free tier through its OpenAI-compatible endpoint, so another provider is a one-line change
  for Project 2. Temperature 0. Every response is cached in `cache/llm_cache.json`, and every call is logged
  (tokens, latency) to `results/usage_log.csv`.
- **Baselines:**
  - B1: rating only, within the hard filters
  - B2: retrieval only, no LLM
  - B3: LLM only, no catalog. It recommends from memory and is fuzzy-matched to the catalog.

## Evaluation
- 20 test cases in `eval/test_cases.json`:
  - 12 standard
  - 3 multi-constraint
  - 2 vague
  - 1 impossible
  - 1 out-of-domain
  - 1 prompt injection
- **Labelling:** the notebook writes `eval/labeling_pool.csv` (all systems' results per case). Two of us label each
  row 1/0 independently, we resolve disagreements in a `final_label` column, save the result as `eval/labels.csv`,
  and report Cohen's kappa.
- **Metrics:**
  - P@5, Hit@5 and nDCG@5 (once the labels exist)
  - hard-constraint satisfaction
  - topical match (keyword proxy)
  - in-catalog rate (hallucinations)
  - JSON validity
  - expected-behaviour pass rate
  - diversity and coverage
  - LLM-as-judge explanation quality (faithfulness / relevance / clarity), checked by humans on about 10 samples
  - tokens and latency per request
- **Consistency:** 3 repeated runs on 5 cases → top-5 Jaccard overlap.
- **Error analysis:** categorise the failures with examples, and document one improvement iteration (before/after).

## Roles
- **M1 – Product & data:** framing, data cleaning/EDA, `data/raw/SOURCE.md`; summary sections Problem/Data; slides 1–2.
- **M2 – LLM pipeline:** client, prompts, structured output, guardrails; summary Method; slide 3.
- **M3 – Retrieval, baselines & demo:** embeddings, filters, B1–B3, live demo.
- **M4 – Evaluation:** test cases, labelling protocol, metrics, judge, plots, error analysis; summary Evaluation/Results; slide 4.
- **Everyone:** risks & limitations (slide 5), AI-tool log in README, conclusion + Project 2 teaser (slide 6).
  In rehearsal each person presents someone else's part.

## Working with the notebook in git
- The main notebook is the graded artifact. Only **one person edits it at a time**; announce it in the team chat.
- Experiment in `notebooks/dev/<name>_*.ipynb`, then move the finished code into the main notebook.
- Before committing an LLM run, commit `cache/llm_cache.json` too, so everyone can replay it without a key.

## Task board (2 weeks)

### Days 1–2: kickoff
- [x] Plan, repo scaffold, v0 notebook, draft test cases (Claude Code)
- [ ] Download the Kaggle CSV → `data/raw/`, fill in `data/raw/SOURCE.md` (M1)
- [ ] Everyone: create a Gemini key, run the notebook once in Colab
- [ ] Agree on the business framing (client, users, KPIs); write it in notebook section 2 (M1)
- [ ] Fill in names in README (all)

### Days 2–3: test cases (before any prompt tuning!)
- [ ] Review and adjust the 20 draft cases in `eval/test_cases.json`; check that every requested topic exists in the catalog (M4 + all, 5 each)
- [ ] Freeze the test cases (commit with the message "freeze test cases")

### Days 3–5: build & iterate
- [ ] Prompt iterations on preference extraction and rerank/explain (M2)
- [ ] Check embedding backend = MiniLM on Colab; tune `TOP_K_CANDIDATES` and the filter-relaxation rules (M3)
- [ ] Baselines B1–B3 reviewed (M3)
- [ ] Export the labelling pool; two annotators label it → `eval/labels.csv` (M4 + one more)

### Days 6–7: evaluation
- [ ] Full evaluation run on Colab; commit cache + results (M4)
- [ ] LLM-as-judge plus a human check of 10 explanations (M4 + M2)
- [ ] Consistency run (M4)

### Day 8: analysis
- [ ] Error analysis: categorise the failures with examples (all)
- [ ] One documented improvement iteration with before/after numbers (M2/M3)
- [ ] Business value + operational preview (tokens/latency per request) (M1)
- [ ] Risks & responsible use section (all)

### Day 9: freeze & write
- [ ] Fresh Colab "Restart & Run All" with a key, and once without (cache only)
- [ ] `grep` for API keys before the final push
- [ ] 2-page summary PDF (incl. AI tool usage)
- [ ] ≤6 slides (.pdf/.pptx)

### Day 10: rehearsal
- [ ] 8–10 min demo run-through, Q&A drill: everyone explains every section

## Risks to discuss in the report
- **Hallucinated courses or attributes:** handled by grounding, ID validation and the in-catalog metric.
- **Unfaithful or over-persuasive explanations:** checked by the judge plus a human check.
- **Popularity/provider bias and filter bubbles:** measured with the diversity metric.
- **Stale 2021 snapshot and scraped-data terms:** stated as a limitation.
- **Privacy:** simulated personas only. Free-tier prompts may be used by the provider.
- **Prompt injection:** delimiting, a canary token and a dedicated test case.
- **Free-tier rate limits and nondeterminism:** handled by the cache, retries and temperature 0.
