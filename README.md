# Team E – LLM-supported Course Recommender

Group project for **Operation of Large Language Models (2026/27 fall)**.

- **Project 1:** build an LLM-supported recommendation system that suggests online courses from a learner's
  free-text goals and preferences, with a short explanation for each pick.
- **Project 2 (later):** analyse what it would cost to deploy and run this system.

The full plan, roles and timeline are in [`docs/PLAN.md`](docs/PLAN.md).

## How it works (v0)

```
learner request (free text) + optional profile
  → [LLM 1] extract preferences as validated JSON (goal, topics, level, min rating, exclusions, search query)
  → [retrieval] hard filters + semantic search over the course catalog → top-20 candidates
  → [LLM 2] pick the top 5 *only from the candidates* and explain each one
  → [guardrails] schema check, drop invented course IDs, retry → fallback, refuse off-topic requests
```

It is compared against three baselines (B1 rating-only, B2 retrieval without LLM, B3 LLM without catalog)
on 20 predefined test cases in [`eval/test_cases.json`](eval/test_cases.json).

## Repository layout

| Path | What it is |
|---|---|
| `notebooks/project1_course_recommender.ipynb` | **The submission notebook**, self-contained and runs top to bottom |
| `notebooks/dev/` | Personal scratch notebooks (merge finished work into the main notebook) |
| `data/raw/` | The original Kaggle CSV (see [`data/raw/SOURCE.md`](data/raw/SOURCE.md)) |
| `data/courses_clean.csv` | Cleaned catalog written by the notebook (committed) |
| `eval/test_cases.json` | 20 evaluation scenarios |
| `eval/labeling_pool.csv` / `eval/labels.csv` | Relevance-labelling sheet written by the notebook, and the filled-in copy |
| `cache/llm_cache.json` | Saved LLM responses, so the notebook re-runs without an API key |
| `results/` | Metrics tables, per-case outputs, `usage_log.csv` (tokens/latency, input for Project 2) |

## Quick start

### 1. Get a free Gemini API key
Create one at [Google AI Studio](https://aistudio.google.com/apikey). It is free and needs no credit card.
**Never commit the key or paste it into a notebook cell.**

### 2a. Run on Google Colab (recommended for the final run)
1. Open the notebook in Colab (File → Open notebook → GitHub, or upload it).
2. Add the key under **🔑 Secrets** in the left sidebar as `GEMINI_API_KEY`, with notebook access enabled.
3. Runtime → Run all. The first cell clones this repo if the data folder isn't present.
   That only works while the repo is public. Otherwise upload the repo folder to `/content/`.

### 2b. Run locally
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # then put your key in .env (git-ignored)
jupyter notebook notebooks/project1_course_recommender.ipynb
```

### Running without a key
If no key is found, the notebook uses only `cache/llm_cache.json`. Every LLM call that was run before (and
committed) is replayed. A call that isn't in the cache stops with a clear error.

## Free-tier notes
- Defaults: `LLM_MODEL=gemini-2.5-flash` with temperature 0 and thinking disabled. Check the models and live
  quota shown in AI Studio, because free-tier limits change. Override the model via `.env` or Colab secrets.
- The notebook throttles to about 9 requests/minute (`MIN_SECONDS_BETWEEN_CALLS`) and retries on HTTP 429.
- One full evaluation run is about 100–120 LLM calls. Commit `cache/llm_cache.json` afterwards so teammates and
  graders can re-run it for free.
- Data sent to free-tier APIs may be used by the provider, so **only send simulated personas, never real student data.**

## Team & roles

| Member | Role | Owns |
|---|---|---|
| M1 – _name_ | Product & data | business framing, data cleaning/EDA, `SOURCE.md` |
| M2 – _name_ | LLM pipeline | client, prompts, structured output, guardrails |
| M3 – _name_ | Retrieval, baselines & demo | embeddings, filters, B1–B3, live demo |
| M4 – _name_ | Evaluation | test cases, labelling, metrics, judge, error analysis |

## AI tool usage log (required in the summary)

Keep this up to date as we go. The project summary must say which AI tools were used and for what.

| Date | Tool | Used for | By |
|---|---|---|---|
| 2026-10-01 | Claude Code | Project plan, repo scaffold, v0 notebook code, draft test cases | _name_ |
