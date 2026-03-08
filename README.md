# 🛒 Amazon Product & Customer Behavior Analyzer

A two-axis scoring model that evaluates every product on **how much pressure it
faces** and **how much traction it generates** — then segments, stress-tests,
and reports actionable findings.

Runs on **Google Colab** and **Kaggle** without any changes.

---

## Scoring Framework

| Axis | Name | Measures |
|---|---|---|
| Axis 1 | **Pressure Index** | Discount dependency · rating weakness · low social proof |
| Axis 2 | **Traction Index** | Engagement depth · quality signals · sentiment |
| Combined | **Product Vitality Score (PVS)** | Weighted blend — the north-star metric |

## Segments

| Segment | Quadrant | Description |
|---|---|---|
| 🏆 Champions | Hi traction · Lo pressure | Healthy, self-sustaining products |
| 🌱 Rising Stars | Hi traction · Hi pressure | Strong fundamentals, needs optimisation |
| 📊 Core | Lo traction · Lo pressure | Stable but unspectacular |
| ⚠️ Vulnerable | Lo traction · Hi pressure | At-risk, needs intervention |

---

## Quick Start

### Google Colab
1. Open `Amazon_Customer_Behavior_Analyzer.ipynb` in Colab.
2. Run all cells — `kagglehub` downloads the dataset automatically.
3. Run the last cell to download all output files.

### Kaggle
1. Add the [Amazon Sales Dataset](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset) via **+ Add Data**.
2. Run all cells — output files appear in the right-hand output panel.

---

## Output Files

| File | Description |
|---|---|
| `results.csv` | Full scored dataset with all indices |
| `summary.txt` | Executive summary report |
| `dashboard.png` | 6-panel static overview |
| `interactive.html` | Hover/zoom Plotly dashboard |
| `interactive_dark.html` | Dark-theme version of the interactive dashboard |
| `quadrant_map.png` | Pressure × Traction segment map |
| `score_distributions.png` | Score calibration check |
| `segment_quality.png` | Silhouette + sensitivity heatmap |
| `price_analysis.png` | Price elasticity & category breakdown |
| `pca.png` | PCA feature space + loadings |
| `wordclouds.png` | Per-segment word clouds |
| `rfm.png` | RFM segments + overlap heatmap |
| `suspicious_products.csv` | Flagged products for manual audit |

---

## Sample Results

The outputs below were generated on the [Amazon Sales Dataset](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset) (1,465 products, 9 categories).

### Dashboard
![image2](image2)

### Price Elasticity & Category Analysis
![image1](image1)

### PCA — Feature Space
![image3](image3)

### Word Clouds by Segment
![image4](image4)

---

## Methodology

- **Scoring**: Percentile-rank model — all scores are relative, not absolute.
- **Segmentation**: Quadrant-based (split at Pressure × Traction medians). All four segments are always populated.
- **Confidence**: K-Means distance score — how cleanly each product sits in its quadrant.
- **Sentiment**: VADER (60%) + TextBlob (40%) ensemble, Z-score calibrated so mean ≈ 0.
- **Fake detection**: Isolation Forest + 3 rule-based signals (perfect+tiny, volume/sentiment mismatch, extreme discount+top rating).
- **RFM**: Adapted for product data — Recency = inverse discount, Frequency = review count, Monetary = original price.

---

## Dataset

[Amazon Sales Dataset by karkavelrajaj](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset) — hosted on Kaggle.

---

## License

Apache License 2.0
