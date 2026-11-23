# Market Analysis and Stock Segmentation Using RFM

Individual coursework for **6BUIS017W – Customer Relationship Management and Customer Management with Business Intelligence**, University of Westminster.
Student ID: **w2052143**

The project segments S&P 500 shares in two ways and compares the results:

1. **Risk and return** – mean weekly return and beta against the S&P 500 index.
2. **Trading activity (adapted RFM)** – Recency, Frequency and Monetary value built from daily price and volume data.

Both are clustered with **Ward hierarchical clustering** and **K-means**, and the segments are then compared against GICS sectors, against each other, and across two halves of the period.

## Run the notebook

[Open in Google Colab](https://colab.research.google.com/github/aaisha-aamina/crm-sp500-rfm-segmentation/blob/main/Coursework_w2052143.ipynb)

1. Open the link above and choose **Runtime → Run all**.
2. The first cell clones this repository, so the saved data snapshot in `data/` is used.
3. Runs are reproducible: `USE_SNAPSHOT = True` reads the stored snapshot instead of downloading prices again, and the random seed is fixed at 42.

To re-download fresh data instead, set `USE_SNAPSHOT = False`. Results will then differ from those in the report, because Yahoo Finance and Wikipedia change over time.

## Repository contents

| Path | Contents |
|---|---|
| `Coursework_w2052143.ipynb` | Full analysis: Task 1 (data and features), Task 2 (clustering), Task 3 evidence |
| `data/` | Frozen data snapshot: raw daily prices (long format, compressed), S&P 500 constituents, Wikipedia page snapshot, download log and metadata |
| `outputs/` | Feature table, weekly returns, exclusion log, validation summary, split-event check, standardised and raw feature matrices, cluster assignments, run metadata |
| `outputs/figures/` | The 12 figures used in the report |

## Data

| Item | Detail |
|---|---|
| Constituents | S&P 500 list from Wikipedia (503 tickers at retrieval) |
| Prices | Daily OHLCV from Yahoo Finance via `yfinance` (`auto_adjust=False`) |
| Benchmark | S&P 500 index (`^GSPC`) |
| Period | 1 Jan 2021 to 31 Dec 2024 |
| Snapshot retrieved | 14 Nov 2026 |
| Shares retained | 486 of 503 (exclusions are listed in `outputs/exclusion_log.csv`) |
| Weekly observations | 207 (Friday-ending weeks, incomplete first and last weeks dropped) |

## Method summary

**Task 1 – features**
- Beta = covariance of the share's weekly returns with the index divided by the index variance. Mean weekly return is the average of weekly adjusted-close returns.
- Observation date for RFM: 31 Dec 2024.
- A high-volume day is a day when volume is more than twice the average of the previous 20 sessions.
- **R** = calendar days since the latest high-volume day. **F** = number of high-volume days in the last 60 sessions. **M** = sum of Close × Volume over the last 60 sessions (a liquidity measure in USD, not market capitalisation).

**Task 2 – clustering**
- Features are standardised separately for each matrix. M is log-transformed because of extreme skew.
- Ward linkage (Euclidean) and K-means (k-means++, 50 starts) are run for k = 2 to 8.
- k is chosen by the highest silhouette among k ≥ 3 where the smallest cluster has at least 3% of shares (minimum 5). Merge-height gaps, inertia and cophenetic correlation are reported as supporting evidence.
- Extra checks: correlations, VIF and PCA of R, F and M, and a repeat of the clustering without M.

**Task 3 – evidence**
- Sector composition of clusters (Cramér's V, chi-square), agreement between the risk-return and RFM segmentations (ARI, NMI), within-cluster return correlation, and stability of beta and mean return between the two halves of the period.

## Selected solutions

| Feature set | Ward | K-means |
|---|---|---|
| Risk and return | k = 5 | k = 5 |
| Trading activity (R, F, M) | k = 3 | k = 4 |

## Limitations

- The constituent list is the current index membership, so shares that left the index during 2021–2024 are not included (survivorship bias).
- Monetary value is a trading-liquidity proxy, not a customer spend measure.
- On 20 Dec 2024 (a market-wide index-rebalance and options-expiry day) most shares have the same latest high-volume date, so R has many ties and discriminates weakly.
- Segment profiles describe 2021–2024. Beta was stable across the two halves of the period, but mean return was not, so the segments are descriptive rather than predictive.

## Reproducibility files

- `outputs/run_metadata.json` records the settings and versions used for the run.
- `data/metadata.json` records when and how the data snapshot was retrieved.

This repository is for coursework and educational use. Price data is from Yahoo Finance and is subject to its terms.
