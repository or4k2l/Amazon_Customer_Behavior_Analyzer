# Amazon Product & Customer Behavior Analyzer

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python)
![License](https://img.shields.io/badge/License-Apache%202.0-green)
![Platform](https://img.shields.io/badge/Platform-Kaggle%20%7C%20Colab-orange?logo=google-colab)

## Abstract

This project analyzes **1,465 Amazon products across 9 categories** from the
[`karkavelrajaj/amazon-sales-dataset`](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset)
Kaggle dataset. It builds three custom scoring indices — **Pressure Index**,
**Traction Index**, and **Perceived Value Score (PVS)** — then clusters products
into four actionable business segments, detects suspicious reviews using Isolation
Forest, performs RFM analysis, runs price-elasticity regression, and produces an
interactive Plotly dashboard. All results are reproducible with `RANDOM_STATE = 42`.

---

## Screenshots

> The notebook saves the following plots to the output directory:

| File | Description |
|------|-------------|
| `01_index_distributions.png` | Histogram of Pressure / Traction / PVS |
| `02_segments_scatter.png` | Traction vs PVS coloured by segment |
| `03_suspicious_reviews.png` | Risk-level scatter (discount & rating) |
| `04_rfm_segments.png` | Bar chart of RFM segment counts |
| `05_silhouette.png` | Silhouette coefficient per segment |
| `06_price_elasticity.png` | Discount % vs PVS regression |
| `07_pca.png` | 2D PCA projection by segment |
| `08_wordcloud.png` | Most frequent product terms |
| `09_category_heatmap.png` | Category × metric heatmap |
| `10_rising_stars.png` | Rising Stars count by category |
| `dashboard.html` | Interactive 7-panel Plotly dashboard |
| `results.csv` | Full per-product results table |

---

## Scoring Framework

| Index | Formula (weights) | Interpretation |
|-------|-------------------|----------------|
| **Pressure Index** | 0.40 × Discount + 0.40 × Rating-Risk + 0.20 × Low-Engagement | How much market/competitive stress the product faces |
| **Traction Index** | 0.35 × Engagement + 0.35 × Cat-Rating + 0.30 × Sentiment | How strongly customers are pulled toward the product |
| **PVS** | 0.50 × Traction + 0.30 × Cat-Rating + 0.20 × Sentiment | Overall perceived value in the eyes of the customer |

All indices are min-max normalised to **[0, 1]**.

---

## Segments

| # | Emoji | Segment | Quadrant | Description |
|---|-------|---------|----------|-------------|
| 1 | 🏆 | **Champions** | High PVS · Low Pressure | Top performers — protect & invest |
| 2 | 🚀 | **Rising Stars** | High PVS · High Pressure | Growth potential — nurture & de-risk |
| 3 | 🔵 | **Core** | Mid PVS · Low Pressure | Stable base — maintain margins |
| 4 | ⚠️ | **Vulnerable** | Low PVS · High Pressure | At-risk products — needs intervention |

**Example counts** (from a reference run on `karkavelrajaj/amazon-sales-dataset` with `RANDOM_STATE = 42`):
Champions 573 (39.1%) · Rising Stars 160 (10.9%) · Core 159 (10.9%) · Vulnerable 573 (39.1%)

---

## Quick Start

### Option A — Kaggle

1. Open [Kaggle Notebooks](https://www.kaggle.com/code) → **New Notebook**
2. Upload `Amazon_Customer_Behavior_Analyzer.ipynb`
3. Enable **Internet** in the notebook settings (to download `karkavelrajaj/amazon-sales-dataset`)
4. **Run All** — outputs are saved to `/kaggle/working/`

### Option B — Google Colab

1. Go to [colab.research.google.com](https://colab.research.google.com) → **File → Upload notebook**
2. Upload `Amazon_Customer_Behavior_Analyzer.ipynb`
3. Add your Kaggle credentials:
   ```python
   import os
   os.environ['KAGGLE_USERNAME'] = 'your_username'
   os.environ['KAGGLE_KEY']      = 'your_api_key'
   ```
4. **Runtime → Run all** — outputs are saved to `/content/output/` and auto-downloaded as a ZIP.

---

## Output Files

| File | Format | Description |
|------|--------|-------------|
| `results.csv` | CSV | Product-level scores, segments, RFM labels, risk flags |
| `dashboard.html` | HTML | Interactive 7-panel Plotly dashboard |
| `01_index_distributions.png` | PNG (150 dpi) | Index distribution histograms |
| `02_segments_scatter.png` | PNG | Segment scatter plot |
| `03_suspicious_reviews.png` | PNG | Suspicious review scatter |
| `04_rfm_segments.png` | PNG | RFM bar chart |
| `05_silhouette.png` | PNG | Silhouette analysis |
| `06_price_elasticity.png` | PNG | Price elasticity regression |
| `07_pca.png` | PNG | PCA 2D projection |
| `08_wordcloud.png` | PNG | Product term word cloud |
| `09_category_heatmap.png` | PNG | Category metric heatmap |
| `10_rising_stars.png` | PNG | Rising Stars by category |

---

## Key Findings

> Results from a reference run on `karkavelrajaj/amazon-sales-dataset` with `RANDOM_STATE = 42`.
> Re-running the notebook will reproduce these values exactly given the same dataset version.

| Finding | Value |
|---------|-------|
| Products analysed | **1,465** across **9 categories** |
| Champion / Vulnerable symmetry | Both **39.1%** — same size, opposite health |
| Best category by PVS | **OfficeProducts** |
| Highest-pressure category | **Electronics** |
| Suspicious products | **102 (7.0%)** — detected via Isolation Forest |
| Strongest improvement lever | **Rating quality (+10% PVS)** via `PVS_R+` scenario |
| Largest RFM group | **"At Risk"** (~400 products) |
| Silhouette score | **0.1230** |
| ANOVA p-value | **< 0.001** (segments are statistically distinct) |
| Mean segment confidence | **0.743** |

---

## Methodology

### Sentiment Ensemble
Each product's sentiment is computed as the average of:
- **VADER** compound score (rule-based, good for short reviews)
- **TextBlob** polarity score (pattern-based)

The ensemble is clipped to `[-1, 1]` before z-scoring to prevent extreme outliers
from inflating the standard deviation. When fewer than 100 review texts are
available, a **rating-based proxy** `(rating − 3) / 2` is used transparently.

### Fake / Suspicious Review Detection
Four signals are fed into an **Isolation Forest** (`contamination=0.07`):
- Review count (unusual volume)
- Discount percentage (extreme discounts to inflate reviews)
- Rating (suspiciously perfect or uniformly low)
- Engagement score (mismatch between traffic and reviews)

Products are classified into **Low / Medium / High** risk tiers.

### RFM Mapping to Product Data
Since this is a product-level (not transaction-level) dataset, RFM dimensions
are proxied as:

| Dimension | Proxy |
|-----------|-------|
| **R** (Recency) | Review count rank — more reviews ≈ more recently active |
| **F** (Frequency) | log(review_count) — normalised repeat-interaction proxy |
| **M** (Monetary) | actual_price × rating_norm — value-weighted price contribution |

---

## Improvements Applied

All 14 improvements from the design specification are implemented:

| # | Section | Improvement |
|---|---------|-------------|
| 3.1 | Setup | `subprocess`-based `_pip()` instead of `!pip`; `print('Dependencies OK')`; `RANDOM_STATE = 42` constant |
| 3.2 | Clean & Detect | Data quality report: % missing per column + rows dropped before imputation |
| 3.3 | Sentiment | Fallback message explaining rating-proxy; `clip(-1, 1)` before z-scoring |
| 3.4 | Pressure / Traction / PVS | Named weight dicts `W_PRESSURE`, `W_TRACTION`, `W_PVS` |
| 3.5 | Segmentation | % share printed per segment; `LOW_CONFIDENCE_THRESHOLD = 0.30` with per-segment counts |
| 3.6 | Suspicious Reviews | `risk_summary` DataFrame: mean Rating, ReviewCount, Discount per Risk_Level |
| 3.7 | RFM | `print()` block explaining R/F/M mapping above the chart code |
| 3.8 | Sensitivity & Validation | Tukey HSD pairwise tests via `statsmodels`; `statsmodels` added to `_pip()` |
| 3.9 | Price Elasticity | Pearson r annotated on the scatter plot alongside β |
| 3.10 | PCA | Variance explained table for all components printed before the 2D plot |
| 3.11 | Dashboard | 7th panel (2×4 layout): horizontal bar chart of Mean PVS per RFM segment |
| 3.12 | Rising Stars Discovery | Cross-sell gap table: Champion coverage vs Rising Stars per category |
| 3.13 | Executive Summary | `TIMESTAMP` line; file size in KB printed after saving `results.csv` |
| 3.14 | Download | File manifest table (name · size KB · exists ✅/⚠️) before downloading |

---

## License

This project is licensed under the **Apache License 2.0** — see the [LICENSE](LICENSE) file for details.

Dataset: [`karkavelrajaj/amazon-sales-dataset`](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset)
on Kaggle (originally scraped from Amazon.in).
