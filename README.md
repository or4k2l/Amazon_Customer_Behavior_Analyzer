# 🛒 Amazon Product & Customer Behavior Analyzer

> **A two-axis scoring model that evaluates every product on how much pressure it faces and how much traction it generates — then segments via GMM, stress-tests, and reports actionable findings.**

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python)](https://python.org)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)](https://jupyter.org)
[![Kaggle](https://img.shields.io/badge/Kaggle-Ready-20BEFF?logo=kaggle)](https://kaggle.com)
[![Colab](https://img.shields.io/badge/Google%20Colab-Ready-F9AB00?logo=googlecolab)](https://colab.research.google.com)
[![License](https://img.shields.io/badge/License-Apache%202.0-green)](LICENSE)

---

## 🔬 Feature Space — PCA Projection

*The PCA plot below shows how cleanly the four segments separate in the full 6-dimensional feature space. Each dot is a product; colours are GMM-assigned segment labels.*

<!-- Replace with your own generated pca.png after running the notebook -->
![PCA — 2D projection of product features by segment](pca.png)

---

## 📊 Dashboard Preview

*(Run the notebook to generate all charts — examples shown from a live run on 1,465 Amazon products)*

| Dashboard | Quadrant Map |
|---|---|
| 6-panel overview with segment distribution, PVS histogram, sensitivity heatmap | Pressure × Traction quadrant scatter with GMM segment boundaries |

| Discount–PVS Relationship | Word Clouds |
|---|---|
| OLS regression of discount % vs Perceived Value Score by segment | Per-segment word clouds (Champions / Rising Stars / Core / Vulnerable) |

---

## 🔍 Scoring Framework

| Axis | Name | Measures |
|---|---|---|
| Axis 1 | **Pressure Index** | Discount dependency · rating weakness · low social proof |
| Axis 2 | **Traction Index** | Engagement depth · quality signals · sentiment |
| Combined | **Product Vitality Score (PVS)** | Weighted blend — the north-star metric |

All components are **percentile ranks** — the model grades products relative to each other so output is meaningful regardless of whether the dataset skews premium or budget overall. This guarantees `mean ≈ 0.500` by mathematical construction.

### Pressure Index (PI)
```
PI = 0.40 × discount_rank + 0.40 × rating_risk_rank + 0.20 × low_engagement_rank
```

### Traction Index (TI)
```
TI = 0.35 × engagement_rank + 0.35 × category_adj_rating_rank + 0.30 × positive_sentiment
```

### Product Vitality Score (PVS)
```
PVS = 0.50 × Traction + 0.30 × category_adj_rating + 0.20 × sentiment
```

---

## 🗺️ Segmentation

Segments are assigned by **Gaussian Mixture Model (GMM)** — a probabilistic clustering method that finds natural density clusters in the 6-dimensional feature space. Unlike a hard median-split, GMM lets segment boundaries follow the actual structure of the data.

| Segment | Quadrant | Strategy |
|---|---|---|
| 🏆 **Champions** | High traction · Low pressure | Loyalty programme · cross-sell |
| 🌱 **Rising Stars** | High traction · High pressure | Reduce discount dependency · generate reviews |
| 📊 **Core** | Low traction · Low pressure | Stable baseline — monitor |
| ⚠️ **Vulnerable** | Low traction · High pressure | Fix quality issues first — discount cuts won't fix traction |

After GMM clustering, each cluster centre is mapped to a quadrant label based on its mean Pressure and Traction values. A per-product **confidence score** is derived from the GMM posterior probabilities: `confidence = P(assigned cluster) − P(second-best cluster)`.

> Unlike a median-split (which forces exactly 50/50 per axis), GMM segment sizes reflect the actual data distribution. A high confidence score indicates the product sits clearly inside one cluster; a low score suggests it sits near a boundary.

---

## 📈 Sample Results (1,465 products · 4 categories)

```
SCORES  (percentile-rank — mean ≈ 0.500 by design)
  Pressure:    0.500  Std=0.193
  Traction:    0.504  Std=0.162
  PVS:         0.505  Std=0.164

SEGMENTS  (GMM — data-driven sizes)
  🏆 Champions:    573 (39.1%)   hi traction · lo pressure
  🌱 Rising Stars: 160 (10.9%)   hi traction · hi pressure
  📊 Core:         159 (10.9%)   lo traction · lo pressure
  ⚠️  Vulnerable:  573 (39.1%)   lo traction · hi pressure

SEGMENT QUALITY
  Silhouette:      0.1230
  ANOVA p-value:   0.00e+00  ✅ Significant
  Mean confidence: 0.743

ANOMALY-FLAGGED PRODUCTS
  Total flagged:   102 (7.0%)
  Isolation Forest anomalies:       73
  Perfect rating + <10 reviews:     38
  High volume + low sentiment:      12
  Extreme discount + top rating:    15

CATEGORY INSIGHTS
  Strongest PVS:    OfficeProducts
  Highest pressure: Electronics

SENSITIVITY
  Strongest lever:  Rating quality push  (+10.0% PVS)
  Riskiest action:  Discount +10%  →  PVS -2.6%
```

---

## 🚀 Quick Start

### Option A — Google Colab (recommended, zero setup)
1. Open `Amazon_Customer_Behavior_Analyzer.ipynb` in Google Colab
2. Run All (`Runtime → Run all`)
3. The notebook auto-downloads the dataset via `kagglehub`
4. Run Section 18 to download all output files

### Option B — Kaggle
1. Create a new Kaggle Notebook
2. Add dataset: **[Amazon Sales Dataset](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset)** via `+ Add Data`
3. Upload `Amazon_Customer_Behavior_Analyzer.ipynb`
4. Run All — output files appear in the right-side panel

### Option C — Local
```bash
pip install vaderSentiment kagglehub wordcloud plotly scikit-learn textblob pandas numpy matplotlib seaborn scipy statsmodels
jupyter notebook Amazon_Customer_Behavior_Analyzer.ipynb
```

---

## 📦 Output Files

| File | Description |
|---|---|
| `results.csv` | Full scored dataset with all metrics |
| `quadrant_map.png` | Pressure × Traction segment scatter (GMM) |
| `score_distributions.png` | Score calibration check |
| `segment_quality.png` | Silhouette plot per segment |
| `price_analysis.png` | Discount % vs PVS OLS regression |
| `pca.png` | 2D PCA projection of all features by segment |
| `wordclouds.png` | Most frequent product terms |
| `rfm.png` | Engagement quintile tier distribution |
| `05b_sensitivity_heatmap.png` | Δ Mean PVS per segment under stress-test scenarios |
| `09_category_heatmap.png` | Category × metric heatmap |
| `10_rising_stars.png` | Rising Stars by category |
| `interactive.html` | Plotly interactive dashboard (open in browser) |
| `interactive_dark.html` | Dark-theme version of the dashboard |
| `suspicious_products.csv` | Anomaly-flagged products |

---

## 🔬 Methodology

### Why percentile ranks?
Absolute scores are dataset-dependent — a "70% discount" means something different in electronics vs. office products. Percentile ranks normalize across the entire catalogue so every score is relative to peers. The mathematical consequence is `mean ≈ 0.500` for all axes — a useful sanity check.

### GMM clustering
The Gaussian Mixture Model fits a mixture of 4 multivariate Gaussians to the 6-dimensional standardised feature space (`Pressure`, `Traction`, `PVS`, `discount_pct`, `rating_norm`, `engagement`). The best model is selected by BIC over 5 random seeds. Posterior probabilities give a calibrated confidence score per product — something a hard threshold cannot provide.

### Sentiment pipeline
```
VADER (50%) + TextBlob (50%) → raw ensemble → Z-score calibration → clip(-3,3)/3.0
```
Z-score calibration ensures `mean ≈ 0` regardless of how uniformly positive Amazon reviews are. Falls back to a rating-based proxy `(rating − 3) / 2` when fewer than 100 products have usable review text.

### Anomaly-flagged products
Isolation Forest (contamination=7%) detects statistical anomalies in the review-count × discount × rating × engagement feature space. Flagged products are saved to `suspicious_products.csv`. PVS scores for flagged products should not be used as-is without manual audit.

### Engagement Quintile Tiers (RFM-proxy)
The notebook segments products into five engagement tiers using quintile scoring on:
- **R** — inverse discount dependency (low discount = organically demanded)
- **F** — log-normalised review count (interaction volume)
- **M** — `actual_price × rating_norm` (perceived revenue contribution)

These are product-level proxies, not true customer-level RFM — the dataset does not contain transaction timestamps.

### Discount–PVS relationship
OLS regression of `discount_pct` on `PVS` quantifies how much each percentage point of discount correlates with perceived value. The β coefficient and Pearson r are annotated directly on the scatter plot.

---

## 📁 Repository Structure

```
Amazon_Customer_Behavior_Analyzer/
├── Amazon_Customer_Behavior_Analyzer.ipynb   # Main analysis notebook
├── README.md                                  # This file
├── LICENSE                                    # Apache 2.0
└── .gitignore
```

---

## 🗂️ Dataset

**[Amazon Sales Dataset](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset)** by Karavel Raja (Kaggle)

The notebook auto-detects column roles by name pattern — it will work with any CSV that has columns containing the words `rating`, `discount`, `category`, `price`. No manual configuration required.

---

## 📋 Requirements

```
pandas >= 1.3
numpy >= 1.21
matplotlib >= 3.4
seaborn >= 0.11
plotly >= 5.0
scikit-learn >= 0.24
scipy >= 1.7
wordcloud >= 1.8
vaderSentiment >= 3.3
textblob >= 0.17
statsmodels >= 0.13
kagglehub >= 0.1
```

---

## 📄 License

Apache License 2.0 — see [LICENSE](LICENSE) for details.
