# Data

Raw and processed data files are **not included** in this repository. They are excluded via `.gitignore` because (a) they are large, and (b) the underlying sources (SEC EDGAR, Yahoo Finance) cannot be redistributed.

To regenerate the data, run the pipeline notebooks in order from the `notebooks/` directory. They will populate this folder with the intermediate and final CSV files used by the analysis.

## Expected files (after running the full pipeline)

| File | Produced by | Description |
|---|---|---|
| `sp500_membership.csv` | `01_edgar_pull.ipynb` | Point-in-time S&P 500 membership list |
| `filings_metadata.csv` | `01_edgar_pull.ipynb` | 8-K filing metadata from EDGAR |
| `filings_with_text.csv` | `02_exhibit_fetcher.ipynb` | Exhibit 99.1 text extracted from HTML |
| `filings_labeled.csv` | `03_yahoo_finance_merge.ipynb` | Filings merged with prices, CARs, downside labels |
| `filings_clean.csv` | `04_text_preprocessing.ipynb` | Cleaned text after 7-layer preprocessing |
| `lm_checkpoint.csv` | `05a_lm_scoring.ipynb` | Loughran-McDonald sentiment scores |
| `finbert_checkpoint.csv` | `05b_finbert_scoring.ipynb` | FinBERT sentiment scores |
| `gpt_checkpoint.csv` | `05c_gpt4o_scoring.ipynb` | GPT-4o sentiment scores (2,000-filing subsample) |

## External data dependencies

The pipeline expects the **Loughran-McDonald Master Dictionary** (2025 version) to be placed at:

```
data/Loughran-McDonald_MasterDictionary_1993-2025.csv
```

Download it from: https://sraf.nd.edu/loughranmcdonald-master-dictionary

This file is not redistributed in the repository per the dictionary's terms of use.

## Notes on reproducibility

- **SEC EDGAR** requests require a `User-Agent` header identifying the requester. Set `SEC_USER_AGENT` in your `.env` file.
- **Yahoo Finance** prices are retrieved via the `yfinance` library and may change over time as Yahoo updates its data. Exact replication of CAR values requires snapshotting the price series.
- **GPT-4o** uses the pinned model version `gpt-4o-2024-05-13`. With `temperature=0`, outputs are largely deterministic but small variation across runs is possible.
