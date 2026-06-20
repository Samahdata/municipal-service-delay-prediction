# 🏙️ Municipal Service Delay Diagnostics

> **End-to-end ML diagnostic system** predicting SLA breaches in city service requests — before they happen.

---

## 📋 Executive Summary

Machine learning pipeline designed to identify municipal service requests at risk of missing SLA targets, built for the City of Saint-Florent's operations team.

| | |
|---|---|
| **Dataset** | 101K+ municipal service requests |
| **Database** | 5 relational SQLite tables |
| **Models Evaluated** | 6 (including AutoML) |
| **Best Model** | Tuned Logistic Regression |
| **F1 Score** | 0.41 (late class) |
| **Recall** | 54% |
| **Business Outcome** | Early identification of high-risk requests for proactive intervention |
| **Key Risk Flag** | Saint-Laurent borough: do-not-deploy recommendation issued |

---

## 🗃️ Database Schema

Five relational SQLite tables joined to construct the modelling dataset:

```
service_requests     — request_id, request_type, created_date, priority, status, department, channel
locations            — request_id, borough, neighborhood, latitude, longitude, postal_code_area
service_targets      — request_type, service_category, expected_days (SLA definition)
weather_daily        — date, temperature, snow_cm, rain_mm, weather_condition
borough_profiles     — borough, population, area_km2, service_staff_count, avg_income_level
```

---

## 🔧 14-Phase Pipeline

| Phase | Description |
|---|---|
| 1 | Exploratory Data Analysis & Data Quality Audit |
| 2 | SQL Joins across 5 relational tables |
| 3 | Data Cleaning & Normalization |
| 4 | Feature Engineering |
| 5 | **Leakage Audit** — 4 features identified & removed |
| 6 | Train/Test Split (time-ordered 83/17) |
| 7 | Baseline Model |
| 8 | Supervised Model Comparison (6 models) |
| 9 | Hyperparameter Tuning |
| 10 | Overfitting / Underfitting Analysis |
| 11 | **Fairness Analysis** across 6 boroughs |
| 12 | AutoML Comparison (FLAML) |
| 13 | Cluster Analysis |
| 14 | Final Summary & Deployment Recommendation |

---

## 🚨 Leakage Audit (Phase 5)

Four features were identified and **removed before modeling**:

| Feature | Why It Leaks |
|---|---|
| `status` | Post-creation field — unavailable at request time |
| `assigned_team` | Assigned after submission |
| `completion_days` | Computed from closed_date (target-derived) |
| `avg_completion_days` | Historical average derived from target |

Without this audit, inflated F1 scores would have produced an **undeployable model**.

---

## 📊 Model Comparison

![Model Comparison Chart](https://private-user-images.githubusercontent.com/279774191/610791039-b57042b3-a381-4649-91a9-f7491d71a4ea.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3ODE5Njk3NDQsIm5iZiI6MTc4MTk2OTQ0NCwicGF0aCI6Ii8yNzk3NzQxOTEvNjEwNzkxMDM5LWI1NzA0MmIzLWEzODEtNDY0OS05MWE5LWY3NDkxZDcxYTRlYS5wbmciLCJjb25kaXRpb25zIjpbeyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZGF0ZSI6IjIwMjYtMDYtMjAifV19.kkr91fOAeyGKYIq64L7-7Nh2R2-Jz-QsToGxeVIpc_w)

| Model | F1 (Late Class) | Recall | Notes |
|---|---|---|---|
| **Tuned Logistic Regression** | **0.41** | **0.54** | ✅ Selected — interpretable, no leakage |
| Tuned Random Forest | 0.399 | — | Overfitting gap ~0.07 |
| Default Random Forest | 0.121 | — | Severe overfitting (Train F1: 0.98) |
| AutoML (FLAML / XGBoost) | 0.275 | — | Cannot perform leakage audit |
| Rule-Based Baseline | 0.00 | — | Predicted no requests as late |

**Why Logistic Regression over AutoML:** AutoML optimizes hyperparameters but cannot perform domain-aware leakage detection. Our manual pipeline outperformed FLAML on F1 while remaining interpretable to City operations staff.

---

## ✅ Best Model — Confusion Matrix (Phase 9)

![Confusion Matrix](https://private-user-images.githubusercontent.com/279774191/610790122-a153fc2b-18b6-4334-ba7c-d537e72320bc.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3ODE5Njk3MzgsIm5iZiI6MTc4MTk2OTQzOCwicGF0aCI6Ii8yNzk3NzQxOTEvNjEwNzkwMTIyLWExNTNmYzJiLTE4YjYtNDMzNC1iYTdjLWQ1MzdlNzIzMjBiYy5wbmciLCJjb25kaXRpb25zIjpbeyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZGF0ZSI6IjIwMjYtMDYtMjAifV19.g7_pvVvKPePoqipo_UCesHquHvagMaDaHHA0EVtcRmM)

---

## ⚖️ Fairness Analysis — Borough Level (Phase 11)

![Borough Fairness Analysis](https://private-user-images.githubusercontent.com/279774191/610790232-f733ddab-386c-4d4c-9260-6d3a6be2aa96.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3ODE5Njk3NDEsIm5iZiI6MTc4MTk2OTQ0MSwicGF0aCI6Ii8yNzk3NzQxOTEvNjEwNzkwMjMyLWY3MzNkZGFiLTM4NmMtNGQ0Yy05MjYwLTZkM2E2YmUyYWE5Ni5wbmciLCJjb25kaXRpb25zIjpbeyJ0eXBlIjoiYWNjZXNzX3Rva2VuIiwiZGF0ZSI6IjIwMjYtMDYtMjAifV19.0Xv3RT9Wn66OrPzP8tLoS8b528ijRqW3ym6IU6EQS-M)

| Borough | Recall | F1 | Flag |
|---|---|---|---|
| Saint-Laurent | 0.129 | 0.183 | 🚨 DO NOT DEPLOY |
| Other boroughs | 0.54 avg | 0.41 avg | ✅ Acceptable |

**Saint-Laurent finding:** The model misses ~87% of true delays in this borough. Root cause: Saint-Laurent has the highest staffing and income levels in the dataset, shifting its feature distributions relative to other boroughs.

**Recommendation:** Do not use this model to deprioritize or delay responses in Saint-Laurent. A borough-specific model or dedicated monitoring system is required before operational deployment.

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Language | Python |
| Database | SQLite (5 relational tables) |
| ML Libraries | Scikit-learn, FLAML (AutoML) |
| Models | Logistic Regression, Random Forest, MLP, XGBoost |
| Pipeline | sklearn Pipeline + ColumnTransformer |
| Validation | TimeSeriesSplit, cross-validation |
| Visualization | Matplotlib, Seaborn |

---

## 👤 Author

**Samah Sayed** — Junior Data Scientist  
15+ years domain expertise in aviation, finance & public services  
📍 Quebec, Canada | [LinkedIn](https://www.linkedin.com/in/samah-sayed55) | [GitHub Portfolio](https://github.com/Samahdata)
