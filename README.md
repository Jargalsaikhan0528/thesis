# Identifying Downside Market Reactions to Financial Text

Replication code for the undergraduate thesis:

> Gansuld, J. (2026). *Identifying downside market reactions to financial text: A comparison of sentiment methods and large language models.* Central European University, Bachelor of Science in Data Science and Society.

This repository contains all code used to construct the sample, retrieve and clean SEC 8-K filings, compute abnormal returns, run three sentiment scoring methods (Loughran-McDonald dictionary, FinBERT, GPT-4o), and produce the empirical results and robustness checks reported in the thesis.

## Overview

The pipeline compares three sentiment extraction methods on 5,280 earnings press releases filed as SEC 8-K exhibits by S&P 500 companies between November 2023 and April 2026. Each method produces a sentiment score that is evaluated against a binary downside label, defined as a cumulative abnormal return below −3.90% over the two-day event window CAR[0,+1], following the standard market model of MacKinlay (1997).

## Repository structure

```
.
├── README.md                  # this file
├── requirements.txt           # pinned Python dependencies
├── .env.example               # template for API keys (copy to .env)
├── .gitignore                 # files git ignores (data, secrets, etc.)
├── LICENSE                    # MIT
├── notebooks/                 # full pipeline, run in numerical order
│   ├── 01_edgar_pull.ipynb           # S&P 500 membership + ticker-CIK matching + 8-K retrieval
│   ├── 02_exhibit_fetcher.ipynb      # Exhibit 99.1 HTML extraction
│   ├── 03_yahoo_finance_merge.ipynb  # Price data, market-model CAR computation, downside labels
│   ├── 04_text_preprocessing.ipynb   # 7-layer text cleaning
│   ├── 05a_lm_scoring.ipynb          # Loughran-McDonald dictionary
│   ├── 05b_finbert_scoring.ipynb     # FinBERT (yiyanghkust/finbert-tone)
│   ├── 05c_gpt4o_scoring.ipynb       # GPT-4o (gpt-4o-2024-05-13)
│   ├── 06_results.ipynb              # AUC, DeLong test, Spearman, bottom-decile CAR
│   └── 07_robustness_check.ipynb     # Alternative event windows, market-adj returns, thresholds
├── prompts/
│   └── gpt4o_system_prompt.txt       # full GPT-4o prompt (matches thesis Appendix A)
└── data/
    └── README.md                     # data is not in repo; this explains how to regenerate
```

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/Jargalsaikhan0528/thesis.git
cd thesis
```

### 2. Create a Python environment

The pipeline was developed with Python 3.14 but should work with 3.10+.

```bash
python -m venv .venv
source .venv/bin/activate     # on macOS / Linux
# OR
.venv\Scripts\activate        # on Windows
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure secrets

Copy `.env.example` to `.env` and fill in your own values:

```bash
cp .env.example .env
```

Then edit `.env` and add:

- `OPENAI_API_KEY` — required only for notebook `05c_gpt4o_scoring.ipynb`. Get one at https://platform.openai.com/api-keys
- `SEC_USER_AGENT` — required by SEC EDGAR. Use the format `Your Name your.email@example.com`

The `.env` file is excluded from git via `.gitignore` and must never be committed.

### 5. Download the Loughran-McDonald dictionary

Place the 2025 Master Dictionary CSV at:

```
data/Loughran-McDonald_MasterDictionary_1993-2025.csv
```

Download from: https://sraf.nd.edu/loughranmcdonald-master-dictionary

## Running the pipeline

Notebooks are designed to run in numerical order. Each builds on output produced by earlier stages:

1. **`01_edgar_pull.ipynb`** — Reconstructs point-in-time S&P 500 membership and retrieves all 8-K filings for those companies between November 2023 and April 2026.
2. **`02_exhibit_fetcher.ipynb`** — Downloads and extracts Exhibit 99.1 text from each filing.
3. **`03_yahoo_finance_merge.ipynb`** — Retrieves price data via `yfinance`, estimates the market model, and computes CAR[0,+1] and the downside label.
4. **`04_text_preprocessing.ipynb`** — Applies the seven-layer text cleaning procedure described in the thesis (zero-width chars, SGML headers, address blocks, contact boilerplate, About sections, whitespace).
5. **`05a` / `05b` / `05c`** — The three sentiment scoring methods. These can be run independently in any order.
6. **`06_results.ipynb`** — Reproduces all tables and figures from the Results chapter.
7. **`07_robustness_check.ipynb`** — Reproduces the robustness checks (alternative event windows, market-adjusted returns, threshold sensitivity).

## Cost note for `05c_gpt4o_scoring.ipynb`

This notebook calls the OpenAI API once per filing. The thesis evaluates GPT-4o on a stratified subsample of 2,000 filings; at `gpt-4o-2024-05-13` pricing, the full subsample run cost approximately USD 10–15.

## Data availability

Raw and processed data are not redistributed in this repository, both for size reasons and because the underlying sources (SEC EDGAR, Yahoo Finance, Loughran-McDonald dictionary) cannot be republished. See `data/README.md` for details on each expected file and how to regenerate the dataset from scratch.

## Reproducibility notes

- **GPT-4o version is pinned** to `gpt-4o-2024-05-13` in `05c_gpt4o_scoring.ipynb`. Other GPT-4o versions will produce different scores.
- **Yahoo Finance price data** is retrieved live and may drift over time as Yahoo backfills or revises historical data. The CARs in the thesis reflect data retrieved during the sample-construction window.
- **Random seeds** are fixed for the stratified GPT-4o subsample (`random_state=42`) and for bootstrap confidence intervals.
- **The exact prompt** used for GPT-4o is provided at `prompts/gpt4o_system_prompt.txt` and matches Appendix A of the thesis.

## License

MIT — see `LICENSE`.

## Contact

For questions about the thesis or replication, open an issue on this repository.
