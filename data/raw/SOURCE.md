# Data source

**Dataset:** Coursera Courses Dataset 2021 (Kaggle)
**Content:** ~3,500 Coursera courses scraped from the public Coursera catalog in 2021.
**Expected columns:** `Course Name`, `University`, `Difficulty Level`, `Course Rating`,
`Course URL`, `Course Description`, `Skills`

## How to add it

1. Download the CSV from Kaggle (requires a free Kaggle login).
2. Put the file in this folder (`data/raw/`), e.g. `data/raw/Coursera.csv`.
   If there is more than one CSV here, set `RAW_CSV_NAME` in the notebook.
3. Run the notebook's data section: it writes the cleaned catalog to
   `data/courses_clean.csv`, which is committed so graders don't need a Kaggle account.

## Fill in after downloading

| Field | Value |
|---|---|
| Kaggle URL | _TODO_ |
| Author / uploader | _TODO_ |
| License shown on Kaggle | _TODO_ |
| Download date | _TODO_ |
| File name and row count | _TODO_ |

## Notes for the report (limitations)

- This is a **public, scraped snapshot (2021)**. Courses may have changed or been retired since,
  so ratings and descriptions are not current.
- It contains no personal data. Learner personas in `eval/test_cases.json` are **simulated**.
- We use it only for non-commercial coursework. Cite the Kaggle page in the summary and slides.
