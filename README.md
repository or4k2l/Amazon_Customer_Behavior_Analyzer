# 🛒 Amazon Product & Customer Behavior Analyzer

A two-axis scoring model that evaluates every product on **how much pressure it faces** and **how much traction it generates** — then segments, stress-tests, and reports actionable findings.

Runs on **Google Colab** and **Kaggle** without any changes.

---

## Scoring Framework

| Axis | Name | Measures |
|---|---|---|
| Axis 1 | **Pressure Index** | Discount dependency · rating weakness · low social proof |
| Axis 2 | **Traction Index** | Engagement depth · quality signals · sentiment |
| Combined | **Product Vitality Score (PVS)** | Weighted blend — the north-star metric |

## Segments

| Segment | Quadrant |
|---|---|
| 🏆 **Champions** | High traction · low pressure |
| 🌱 **Rising Stars** | High traction · high pressure |
| 📊 **Core** | Low traction · low pressure |
| ⚠️ **Vulnerable** | Low traction · high pressure |

---

## Sample Results (Amazon Sales Dataset, 1,465 products)

| Metric | Value |
|---|---|
| Products | 1,465 |
| Categories | 9 |
| PVS Mean | 0.505 |
| Silhouette | 0.1230 |
| Suspicious reviews | 102 (7.0%) |
| Best category (PVS) | OfficeProducts |
| Most pressure | Electronics |
| Strongest lever | Rating improvement (+10% PVS) |

### Segment Distribution
- 🏆 Champions: 39.1%
- ⚠️ Vulnerable: 39.1%
- 🌱 Rising Stars: 10.9%
- 📊 Core: 10.9%

---

## Output Files

| File | Description |
|---|---|
| `results.csv` | Full scored dataset |
| `summary.txt` | Executive summary |
| `dashboard.png` | 6-panel overview |
| `quadrant_map.png` | Segment scatter |
| `price_analysis.png` | Price elasticity & categories |
| `pca.png` | Feature space + loadings |
| `wordclouds.png` | Per-segment word clouds |
| `rfm.png` | RFM segments |
| `segment_quality.png` | Silhouette + sensitivity |
| `interactive.html` | Interactive Plotly dashboard |

---

## How to Run

### Google Colab
1. Open the notebook in Colab
2. Run all cells — dataset downloads automatically via `kagglehub`

### Kaggle
1. Add dataset: **Amazon Sales Dataset** by karkavelrajaj
2. Run all cells — output appears in the output panel

### Local
```bash
pip install -r requirements.txt
jupyter notebook Amazon_Customer_Behavior_Analyzer.ipynb
```

---

## Key Findings

1. **Discount barely hurts PVS** (β=−0.0014) — quality signals matter more than price
2. **OfficeProducts** has the highest average PVS and lowest pressure rate (<5%)
3. **Electronics** has ~27% high-pressure products — needs discount restructuring
4. **7% suspicious reviews** detected via Isolation Forest + 3 rule-based signals
5. **Rating improvement** is the strongest single lever (+10.0% PVS)

---

## License
Apache License 2.0
