# 🛒 Amazon Product & Customer Behavior Analyzer

> **A two-axis scoring model that evaluates every product on how much pressure it faces and how much traction it generates — then segments, stress-tests, and reports actionable findings.**

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python)](https://python.org)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)](https://jupyter.org)
[![Kaggle](https://img.shields.io/badge/Kaggle-Ready-20BEFF?logo=kaggle)](https://kaggle.com)
[![Colab](https://img.shields.io/badge/Google%20Colab-Ready-F9AB00?logo=googlecolab)](https://colab.research.google.com)
[![License](https://img.shields.io/badge/License-Apache%202.0-green)](LICENSE)

---

## 📊 Dashboard Preview

*(Run the notebook to generate all charts — examples shown from a live run on 1,465 Amazon products)*

| Dashboard | Quadrant Map |
|---|---|
| 6-panel overview with segment distribution, PVS histogram, sensitivity heatmap | Pressure × Traction quadrant scatter with median thresholds |

| Price Elasticity | Word Clouds |
|---|---|
| Discount tiers, PVS regression, category PVS & pressure rates | Per-segment word clouds (Champions/Rising Stars/Core/Vulnerable) |

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

Segments are assigned by **quadrant** — directly from each product's Pressure × Traction position, split at dataset medians. This guarantees all four segments are meaningfully populated.

| Segment | Quadrant | Strategy |
|---|---|---|
| 🏆 **Champions** | High traction · Low pressure | Loyalty programme · cross-sell |
| 🌱 **Rising Stars** | High traction · High pressure | Reduce discount dependency · generate reviews |
| 📊 **Core** | Low traction · Low pressure | Stable baseline — monitor |
| ⚠️ **Vulnerable** | Low traction · High pressure | Fix quality issues first — discount cuts won't fix traction |

> K-Means (k=4) is still run in the background, but **only** to derive a per-product Confidence score (how cleanly a product sits in its quadrant vs. the other three centres). Segment labels come exclusively from the median-split quadrant logic.

---

## 📈 Sample Results (1,465 products · 4 categories)

```
SCORES  (percentile-rank — mean ≈ 0.500 by design)
  Pressure:    0.500  Std=0.193
  Traction:    0.504  Std=0.162
  PVS:         0.505  Std=0.164

SEGMENTS
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
4. Run Section 17 to download all output files

### Option B — Kaggle
1. Create a new Kaggle Notebook
2. Add dataset: **[Amazon Sales Dataset](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset)** via `+ Add Data`
3. Upload `Amazon_Customer_Behavior_Analyzer.ipynb`
4. Run All — output files appear in the right-side panel

### Option C — Local
```bash
pip install vaderSentiment kagglehub wordcloud plotly scikit-learn textblob pandas numpy matplotlib seaborn scipy
jupyter notebook Amazon_Customer_Behavior_Analyzer.ipynb
```

---

## 📦 Output Files

| File | Description |
|---|---|
| `results.csv` | Full scored dataset with all metrics |
| `summary.txt` | Executive summary text report |
| `dashboard.png` | 6-panel static overview |
| `quadrant_map.png` | Pressure × Traction segment scatter |
| `score_distributions.png` | Score calibration check |
| `segment_quality.png` | Silhouette + sensitivity heatmap |
| `price_analysis.png` | Price elasticity & category analysis |
| `pca.png` | Feature space PCA + loadings |
| `wordclouds.png` | Per-segment word clouds |
| `rfm.png` | RFM segments + overlap heatmap |
| `interactive.html` | Plotly interactive dashboard (open in browser) |

---

## 🔬 Methodology

### Why percentile ranks?
Absolute scores are dataset-dependent — a "70% discount" means something different in electronics vs. office products. Percentile ranks normalize across the entire catalogue so every score is relative to peers. The mathematical consequence is `mean ≈ 0.500` for all axes — a useful sanity check.

### Sentiment pipeline
```
VADER (60%) + TextBlob (40%) → raw ensemble → Z-score calibration → clip(-3,3)/3.0
```
Z-score calibration ensures `mean ≈ 0` regardless of how uniformly positive Amazon reviews are.

### Text cleaning
Removes: CDN image URLs, HTML tags, `dp`/`ref`/`qid`/`sr`/`utm` tokens, file extensions. Average noise reduction: ~60-80% of raw text length.

### Suspicious review detection
Four independent signals (each contributes 1 point to Suspicion score):
1. **Isolation Forest** — statistical anomaly in full feature space (contamination=5%)
2. **Perfect + tiny** — rating ≥ 4.8 with < 10 reviews
3. **Volume/sentiment gap** — top-10% review count + bottom-20% sentiment
4. **Extreme discount trap** — discount > 80% + rating ≥ 4.5

Products with Suspicion ≥ 1 are flagged. PVS scores for flagged products should not be used as-is without manual audit.

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
kagglehub >= 0.1
```

---

## 📄 License

Apache License 2.0 — see [LICENSE](LICENSE) for details.
