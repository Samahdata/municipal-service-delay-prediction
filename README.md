# 🏙️ Municipal Service Delay Diagnostics

> **Can we predict — at the moment a citizen submits a request — whether it will be completed late?**

A full end-to-end machine learning diagnostic system built for a city operations team, covering data leakage auditing, multi-model comparison, fairness analysis across boroughs, and a deployable scoring pipeline.

---

## 📌 Business Problem

The City of Saint-Florent processes ~101,000 municipal service requests per year (pothole repairs, snow removal, garbage collection, building permits, and more). Approximately **28% of requests are completed past their promised SLA deadline**, generating citizen dissatisfaction and operational inefficiency.

**Stakeholder:** Director of Operations  
**Decision supported:** Flag high-risk requests *at creation time* for proactive staff intervention

| Error Type | Business Impact | Severity |
|---|---|---|
| False Negative (miss a late request) | SLA breach, citizen complaint, no early intervention | HIGH |
| False Positive (flag an on-time request) | Staff time wasted on unnecessary check-in | MEDIUM |

---

## 🔍 Key Findings

### ⚠️ Data Leakage Audit (Critical Step)
Four features were identified and removed as they encode future information unavailable at intake time:

| Feature | Why It Leaks |
|---|---|
| `status` | Updated after triage — always 'closed' in training data |
| `assigned_team` | Assigned post-submission, not at creation |
| `completion_days` | Derived from `closed_date` — pure future data |
| `avg_completion_days` | Borough aggregate of outcomes — target-derived |

### 🏆 Model Comparison

| Model | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| **LR Tuned ★** | 0.562 | 0.330 | **0.540** | **0.410** |
| Logistic Regression | 0.570 | 0.332 | 0.520 | 0.405 |
| RF Tuned | 0.579 | 0.334 | 0.495 | 0.399 |
| AutoML (FLAML) | 0.631 | 0.310 | 0.252 | 0.278 |
| MLP | 0.688 | 0.330 | 0.103 | 0.157 |
| Random Forest | 0.699 | 0.340 | 0.074 | 0.121 |
| Baseline (rule-based) | 0.718 | 0.000 | 0.000 | 0.000 |

**Winner: Logistic Regression (tuned, C=0.1, L2)** — highest F1 and recall on the late class, and fully explainable to non-technical city staff.

### ⚖️ Fairness Analysis — Per Borough

| Borough | Precision | Recall | F1 | Flag |
|---|---|---|---|---|
| Lasalle | 0.42 | 0.62 | 0.50 | ✅ |
| Cote-Des-Neiges | 0.36 | 0.58 | 0.44 | ✅ |
| Rosemont | 0.33 | 0.55 | 0.41 | ✅ |
| Plateau | 0.31 | 0.52 | 0.39 | ✅ |
| Verdun | 0.29 | 0.48 | 0.36 | ✅ |
| **Saint-Laurent** | 0.25 | **0.13** | **0.18** | 🔴 HIGH RISK |

> **Do-not-use recommendation:** The City should NOT use this model to deprioritise responses in Saint-Laurent. With recall = 0.129, the model misses ~87% of true delays there. A borough-specific model or separate threshold is required before deployment in that area.

---

## 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3+-orange)
![SQLite](https://img.shields.io/badge/SQLite-database-lightgrey)
![FLAML](https://img.shields.io/badge/FLAML-AutoML-green)
![Pandas](https://img.shields.io/badge/Pandas-latest-lightgrey)

- **Language:** Python 3.10+
- **Database:** SQLite (5 relational tables, ~101K rows)
- **Libraries:** pandas, numpy, scikit-learn, matplotlib, seaborn, FLAML
- **Modeling:** sklearn Pipeline with ColumnTransformer, TimeSeriesSplit, RandomizedSearchCV

---

## 📁 Project Structure

```
municipal-service-delay-prediction/
│
├── student_starter_notebook.ipynb   # Full 14-phase ML pipeline
├── municipal_service_diagnostics.db # SQLite database (5 tables)
├── Municipal_Service_Report.pdf     # Full diagnostic report
├── requirements.txt                 # Dependencies
└── README.md                        # This file
```

---

## 🚀 How to Run

```bash
# 1. Clone the repo
git clone https://github.com/Samahdata/municipal-service-delay-prediction.git
cd municipal-service-delay-prediction

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run the notebook
jupyter notebook student_starter_notebook.ipynb
```

> ⚠️ The database file `municipal_service_diagnostics.db` must be in the same directory as the notebook.  
> Headline F1 should reproduce within ±0.01.

---

## ⚠️ Limitations & Deployment Risks

- **Concept drift:** Trained on 18 months of data. Recommend retraining every 6 months.
- **Weather leakage:** Daily weather totals are a pragmatic approximation — replace with morning readings in production.
- **Precision constraint:** ~33% precision means 66% of flagged requests are false alarms. Human review required.
- **Saint-Laurent blind spot:** Borough requires dedicated monitoring or a separate model.

---

## 👤 Author

**Samah Sayed** — Data Scientist | AI & Automation  
[LinkedIn](https://www.linkedin.com/in/samahsayed55) · [GitHub](https://github.com/Samahdata) · samah.sayed055@gmail.com
