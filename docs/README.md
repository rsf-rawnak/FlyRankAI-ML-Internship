# FlyRank Content Refresh Priority Model

A capstone project for the **Google Search Ranking & Discoverability** track, focused on **Lane 2 — Refresh / Content Opportunity Scoring**.

## Research question

> Given a content item's available search and engagement signals, which pages should a content reviewer look at first this week, and what should they do with each one?

The project treats this as a **ranking problem** because reviewer capacity is limited to roughly 20–50 pages per week while the modeling snapshot contains 30,000 pages.

## Capstone paper

The deployed research paper is the `index.html` file in `docs/`.

It presents:

- Abstract
- Introduction and problem statement
- Data
- Methodology
- Results
- Limitations and honest framing
- Ranked recommendations
- Reproducibility
- Acknowledgments and data credit

## Notebook

The reproducible capstone notebook is:

`work/notebooks/capstone.ipynb`

It consolidates the W01–W07 progression:

`research question → data contract → baseline → model → validation audit → action playbook`

The notebook is designed to work in Google Colab and includes an **Open in Colab** link.

## Dataset and target

The final modeling snapshot contains:

- **30,000 content rows**
- **32 clients**
- One row per pseudonymized content item
- Search, engagement, content-age, freshness, and content metadata

The target is:

`is_declining_label = (trend_direction == "down")`

This is explicitly treated as a **current-window proxy**, not a genuine future outcome.

The model excludes:

- `trend_direction`
- `trend_pct`
- `is_declining_label`
- `*_last_30d`
- `*_prev_30d`

`client_id` is used for grouping/splitting only and is not a model feature.

## Model and validation

The primary model is **Logistic Regression**, with **Random Forest** used as a comparison model.

Validation uses:

- `GroupShuffleSplit`
- 80/20 split
- `random_state=42`
- groups = `client_id`
- zero client overlap between train and test

The final split contains:

- Train: **23,837 rows / 25 clients**
- Test: **6,163 rows / 7 clients**
- Test base rate: **0.511**

## Results

All methods are evaluated on the same held-out test rows using Precision@50 as the primary operational metric.

| Method | Precision@50 | ROC-AUC | Average Precision |
|---|---:|---:|---:|
| Week-4 rule baseline | 0.340 | 0.510 | 0.511 |
| Logistic Regression | **0.860** | 0.626 | 0.625 |
| Random Forest | 0.520 | 0.595 | 0.581 |

The Logistic Regression result is approximately **2.53× the rule-baseline Precision@50** on this split.

## Leakage audit

The notebook deliberately tests the validation setup by re-injecting `trend_pct`, a direct source of the proxy label.

Result:

- Honest Precision@50: **0.860**
- Deliberately leaky Precision@50: **1.000**
- Leak-induced jump: **+0.140**

The leaky feature is not included in the production feature set.

The notebook also records that a naive row-level split reached Precision@50 = 0.92 but allowed 31 clients to appear on both sides of the split. The client-grouped result of 0.86 with zero overlap is therefore the reported result.

## Action playbook

The model is intended as **human-in-the-loop decision support**, not autonomous decision-making.

The paper describes four action lanes:

- `refresh`
- `refresh_and_review_ctr`
- `monitor`
- `technical_check`

Unranked pages (`avg_position == 0`) receive separate technical-review treatment rather than being interpreted as healthy pages.

## Honest framing

This project reports an observed ranking result on the stated anonymized snapshot and validation split.

It does **not** claim:

- that refreshing a page causes recovery;
- that the model predicts Google's ranking algorithm;
- that any individual page is guaranteed to improve;
- that the current-window proxy is equivalent to a future outcome.

The stronger future-prediction design documented in the notebook is:

`prior-window features → later-window outcome`

## Repository structure

Use this structure when adding the uploaded files:

```text
FlyRankAI-ML-Internship/
├── docs/
│   └── index.html
├── work/
│   └── notebooks/
│       └── capstone.ipynb
├── .github/
│   └── workflows/
│       └── pages.yml
└── README.md
```

The GitHub Pages workflow publishes `docs/` as the static site.

## GitHub Pages

The included `.github/workflows/pages.yml` deploys `docs/index.html` automatically whenever changes are pushed to `main`.

In GitHub:

1. Open **Settings → Pages**.
2. Set the build/deployment source to **GitHub Actions**.
3. Push the files to `main`.
4. The workflow deploys the paper from `docs/`.

## Data credit

Built on the FlyRank ML Internship dataset.

[FlyRank AI](https://flyrank.ai)

## Reproducibility

The primary reproducibility artifact is:

`work/notebooks/capstone.ipynb`

The notebook contains the data loading, data-quality checks, feature policy, baseline, Logistic Regression and Random Forest comparison, grouped validation, leakage audit, model interpretation, and capstone conclusions.
