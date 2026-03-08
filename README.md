# 🛒 Amazon Product & Customer Behavior Analyzer

![Python 3.x](https://img.shields.io/badge/Python-3.x-blue?logo=python&logoColor=white)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Platform](https://img.shields.io/badge/Platform-Colab%20%2B%20Kaggle-orange?logo=googlecolab&logoColor=white)

A two-axis scoring model (Pressure Index × Traction Index) that processes 1,465 raw Amazon products (1,344 after deduplication and category filter), segments them into four strategic quadrants, detects suspicious reviews, and produces an interactive stakeholder dashboard.

Runs on **Google Colab** and **Kaggle** without any changes.

---

## Visuals

| Output | Description |
|---|---|
| `02_segments_scatter.png` / `quadrant_map.png` | Segment scatter plot — products plotted in Pressure × Traction space |
| `dashboard.png` | 6-panel static dashboard overview |
| `dashboard.html` / `interactive.html` | Hover/zoom Plotly interactive dashboard |
| `08_wordcloud.png` | Per-segment word clouds |
| `07_pca.png` | PCA feature space + loadings |

### Price Elasticity & Category Analysis
<img width="1482" height="880" alt="06_price_elasticity" src="https://github.com/user-attachments/assets/2c5e98af-d389-40ee-b202-ff11ec458cbe" />

### PCA — Feature Space
<img width="1482" height="1030" alt="07_pca" src="https://github.com/user-attachments/assets/9a59e7ac-0a74-4fac-aa82-9d25ed2fa4ee" />

### Word Clouds by Segment
<img width="1939" height="887" alt="08_wordcloud" src="https://github.com/user-attachments/assets/c0ff2510-42fc-4c93-92bf-4f3b4e8ae96c" />

---

## Scoring Framework

| Axis | Name | Measures |
|---|---|---|
| Axis 1 | **Pressure Index** | Discount dependency · rating weakness · low social proof |
| Axis 2 | **Traction Index** | Engagement depth · quality signals · sentiment |
| Combined | **Product Vitality Score (PVS)** | Weighted blend — the north-star metric |

---

## Segments

| Segment | Quadrant | Count | Share |
|---|---|---|---|
| 🏆 Champions | High traction · Low pressure | 259 | 19.3% |
| 🌱 Rising Stars | High traction · High pressure | 411 | 30.6% |
| 📊 Core | Low traction · Low pressure | 376 | 28.0% |
| ⚠️ Vulnerable | Low traction · High pressure | 298 | 22.2% |

---

## Key Results

Results from the confirmed run on the Amazon Sales Dataset:

- **Dataset:** 1,465 raw rows → 1,351 after deduplication → 1,344 after category filter
- **Categories (≥20 products):** Electronics (490), Home&Kitchen (448), Computers&Accessories (375), OfficeProducts (31)
- **Scoring indices:** Pressure mean=0.395 (σ=0.172) · Traction mean=0.521 (σ=0.150) · PVS mean=0.457 (σ=0.147)
- **Model quality:** Silhouette=0.2437 · ANOVA p=3.88e-247 · Mean confidence=0.629
- **Suspicious reviews:** 95 products flagged (7.1%)
- **RFM distribution (product-level RFM segmentation):** Champions 252 · Loyal 341 · Potential 390 · At Risk 236 · Inactive 125
  *(Note: RFM Champions and K-Means Champions are independent segmentation schemes with different criteria.)*
- **PCA variance explained:** PC1=49.4% · PC2=27.8% → 77.2% cumulative
- **Key sensitivity finding:** Discount −10% has the strongest lever effect on PVS
- **Output files generated:** 11 PNG charts · 2 interactive HTML dashboards · results.csv · suspicious_products.csv

---

## Quick Start

```
# Option A — Google Colab (recommended)
# Open the notebook in Colab, run all cells. kagglehub downloads the dataset automatically.

# Option B — Kaggle
# Fork the notebook, add the dataset "karkavelrajaj/amazon-sales-dataset" via "+ Add Data"
```

---

## Requirements

```
pandas
numpy
matplotlib
seaborn
plotly
scikit-learn
scipy
wordcloud
vaderSentiment
textblob
kagglehub
```

---

## Output Files

| File | Description |
|---|---|
| `results.csv` | Full scored dataset (14 columns) |
| `suspicious_products.csv` | 95 flagged products |
| `dashboard.html` | Interactive Plotly dashboard |
| `interactive_dark.html` | Dark-theme variant |
| `01_index_distributions.png` | Score calibration check |
| `02_segments_scatter.png` | Segment overview scatter |
| `03_suspicious_reviews.png` | Suspicious review analysis |
| `04_rfm_segments.png` | RFM segments |
| `05_silhouette.png` | Silhouette quality plot |
| `05b_sensitivity_heatmap.png` | Sensitivity heatmap |
| `06_price_elasticity.png` | Price elasticity |
| `07_pca.png` | PCA feature space |
| `08_wordcloud.png` | Per-segment word clouds |
| `09_category_heatmap.png` | Category comparison |
| `10_rising_stars.png` | Rising stars discovery |

---

## Category Insights

| Category | Products | Mean Rating | Mean Discount | Mean PVS |
|---|---|---|---|---|
| OfficeProducts | 31 | 4.31 | 12.4% | 0.898 |
| Computers&Accessories | 375 | 4.15 | 53.2% | 0.590 |
| Electronics | 490 | 4.08 | 49.9% | 0.439 |
| Home&Kitchen | 448 | 4.04 | 40.1% | 0.334 |

*Only categories with ≥20 products are included in segment analysis. Smaller categories are excluded from scoring.*

---

## Strategic Recommendations

1. **Champions (259):** Launch loyalty programme · cross-sell into lower-PVS categories
2. **Rising Stars (411):** Reduce discount dependency · invest in review generation
3. **Vulnerable (298):** Address root quality issues · target average discount below 50%
4. **Suspicious reviews (95):** Manual audit required before using PVS scores in decisions
5. **Category priorities:** Invest in OfficeProducts expansion · restructure Electronics discount strategy

---

## Model Architecture

The model is structured in three layers:

1. **Feature Engineering** — raw fields (rating, discount, review count, price, sentiment) are cleaned and normalised into dimensionless signals (`rating_norm`, `cat_rating_norm`, `engagement`, `discount_pct`).
2. **Percentile Ranking** — each signal is ranked percentile-wise across all products, making scores relative rather than absolute and robust to outliers.
3. **Weighted Combination** — percentile ranks are blended with tuned weights into the Pressure Index, Traction Index, and the composite Product Vitality Score (PVS).

Segmentation uses **K-Means (k=4)** on the two-dimensional Pressure × Traction space. A **confidence score** is derived from the normalised inverse distance to each product's cluster centroid, indicating how cleanly the product sits within its segment.

---

## Methodology Notes

- **Sentiment:** VADER (60%) + TextBlob (40%) ensemble, Z-score calibrated (mean ≈ 0)
- **Suspicious review detection:** Isolation Forest (contamination=0.05) + 3 rule-based heuristics (perfect rating + tiny review volume, volume/sentiment mismatch, extreme discount + top rating)
- **RFM adapted for product data:** R = inverse discount · F = log review count · M = price × rating_norm
- **Model health:** Shapiro-Wilk normality test · silhouette score · ANOVA + Tukey HSD · segment size check

---

## Dataset

[Amazon Sales Dataset by karkavelrajaj](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset) — hosted on Kaggle.

---

## License

Apache License 2.0
