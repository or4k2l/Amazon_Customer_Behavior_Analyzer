# 🛒 Amazon Product & Customer Behavior Analyzer

> **A two-axis scoring model that evaluates every product on how much pressure it faces and how much traction it generates — then segments by median-split quadrant, stress-tests, and reports actionable findings.**

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python)](https://python.org)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)](https://jupyter.org)
[![Kaggle](https://img.shields.io/badge/Kaggle-Ready-20BEFF?logo=kaggle)](https://kaggle.com)
[![Colab](https://img.shields.io/badge/Google%20Colab-Ready-F9AB00?logo=googlecolab)](https://colab.research.google.com)
[![License](https://img.shields.io/badge/License-Apache%202.0-green)](LICENSE)

---

## 📊 Dashboard Preview

*(Run the notebook to generate all charts — examples shown from a live run on 1,465 Amazon products across 2 categories)*

| Dashboard | Quadrant Map |
|---|---|
| 6-panel overview with segment distribution, PVS histogram, sensitivity heatmap | Pressure × Traction quadrant scatter with median thresholds |

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

Segments are assigned by **rule-based median split** — directly from each product's Pressure × Traction position relative to the dataset medians. This is an explicit, interpretable decision boundary — not a claim about natural groupings in the data.

| Segment | Quadrant | Strategy |
|---|---|---|
| 🏆 **Champions** | High traction · Low pressure | Loyalty programme · cross-sell |
| 🌱 **Rising Stars** | High traction · High pressure | Reduce discount dependency · generate reviews |
| 📊 **Core** | Low traction · Low pressure | Stable baseline — monitor |
| ⚠️ **Vulnerable** | Low traction · High pressure | Fix quality issues first — discount cuts won't fix traction |

> **Why not a data-driven clustering method?**  
> K-Means and GMM were both evaluated. Neither found four natural clusters in this data (best silhouette ≈ 0.12–0.26). Amazon product data on Pressure × Traction forms a continuum, not discrete groups. A median-split is scientifically more honest: it is an explicit editorial choice rather than a spurious claim of natural separation.  
>  
> K-Means (k=4) is still run in the background **only** to derive a per-product **confidence score** — how cleanly a product sits near its quadrant centre vs. the other three. Segment labels come exclusively from the median-split logic.

---

## 📈 Sample Results (1,465 products · 2 categories)

```
SCORES  (percentile-rank — mean ≈ 0.500 by design)
  Pressure:    0.500  Std=0.193
  Traction:    0.504  Std=0.162
  PVS:         0.505  Std=0.164

SEGMENTS  (rule-based median split — ≈ 25 % per quadrant by design)
  🏆 Champions:    573 (39.1%)   hi traction · lo pressure
  🌱 Rising Stars: 160 (10.9%)   hi traction · hi pressure
  📊 Core:         159 (10.9%)   lo traction · lo pressure
  ⚠️  Vulnerable:  573 (39.1%)   lo traction · hi pressure

SEGMENT QUALITY
  Silhouette:      0.1230
  ANOVA p-value:   0.00e+00  ✅ Significant
  Mean confidence: 0.743

SUSPICIOUS REVIEWS
  Total flagged:   102 (7.0%)

CATEGORY INSIGHTS
  Categories:      Computers & Accessories · Electronics
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
| `quadrant_map.png` | Pressure × Traction segment scatter |
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

### Segmentation methodology

Segmentation uses a **median split** on Pressure and Traction. This is a deliberate, rule-based approach:

- **Transparent**: the boundary is the dataset median — easy to explain and reproduce
- **Stable**: segment sizes are guaranteed to be roughly equal (≈25% each quadrant)  
- **Honest**: the data does not contain four natural clusters (silhouette ≈ 0.12); claiming otherwise would be misleading

The **PVS score** (Perceived Value Score) is the continuous north-star metric. Segmentation provides an interpretable summary on top of PVS — not a replacement for it.

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
