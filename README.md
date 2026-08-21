# Toronto 311 Service Requests: Income Equity Analysis

An exploratory data analysis and machine learning project investigating whether Toronto's 311 municipal service request outcomes vary by neighborhood (ward) income level.

## Research Question

**Does the City of Toronto respond differently to 311 service requests depending on the median household income of the ward in which they are filed?**

This project examines service equity across Toronto's 25 wards using 2021 data, asking whether requests are closed at similar rates, whether closure patterns differ by service type, and whether income is a meaningful predictor of whether a request remains unresolved.

## Motivation

311 systems are a primary channel through which residents report non-emergency municipal issues (potholes, illegal dumping, waste collection, etc.). If response or closure rates differ systematically by neighborhood income, this could indicate inequities in how municipal resources are allocated — a question with direct relevance to urban policy and civic equity.

## Data Sources

| Dataset | Source | Description |
|---|---|---|
| `SR2021.csv` | [Toronto Open Data – 311 Service Requests](https://open.toronto.ca) | ~324,000 individual 311 service requests filed in 2021, including ward, request type, division, section, and status |
| `2023-WardProfiles-2011-2021-CensusData.xlsx` | [Toronto Open Data – Ward Profiles (25-Ward Model)](https://open.toronto.ca) | 2021 Census-derived demographic data by ward, including median household income |
| `City_Wards_Data_-_4326.geojson` | [Toronto Open Data – City Wards](https://open.toronto.ca) | Ward boundary polygons (25-ward model), used for choropleth mapping |

**Note on temporal alignment:** Both the 311 data and the income data reflect the year 2021, minimizing the gap between socioeconomic and service data. Census income figures are the most recent available (next census: 2026).

## Repository Structure

```
toronto_311_project/
├── data/
│   ├── SR2021.csv
│   ├── 2023-WardProfiles-2011-2021-CensusData.xlsx
│   └── City_Wards_Data_-_4326.geojson
├── images/
│   ├── income_vs_closure_rate.png
│   ├── monthly_trend.png
│   ├── monthly_trend_by_income.png
│   ├── feature_importance.png
│   └── confusion_matrix.png
├── notebook/
│   ├── toronto_311_requests.ipynb
│   └── models/
│       ├── closure_risk_model.pkl
│       ├── ward_encoder.pkl
│       ├── type_encoder.pkl
│       └── division_encoder.pkl
├── README.md
└── requirements.txt
```

## Methodology

### 1. Data Cleaning
- Removed 5 malformed rows (0.002% of dataset) caused by inconsistent comma usage in the source `Section` field
- Stripped whitespace from column headers and categorical values
- Standardized inconsistent status labels (e.g. `"In progress"` vs `"In Progress"`) via title-case normalization
- Extracted ward numbers from combined name/number strings (e.g. `"Don Valley North (17)"` → `"Ward 17"`) and excluded records with missing/unknown ward
- Dropped columns not relevant to ward-level analysis (`Intersection Street 1/2`)

### 2. Exploratory Data Analysis
- Distribution of requests by ward, status, section, and request type
- Choropleth mapping of median household income by ward using ward boundary geometry
- Monthly request volume trend (seasonal pattern analysis)

### 3. Statistical Analysis
- **Overall closure rate vs. income:** Pearson correlation between ward median household income and percentage of requests with `Closed` status
  - Result: weak negative correlation (**r = -0.338, p = 0.099**) — not statistically significant at α = 0.05, likely limited by small sample size (n = 25 wards)
- **Request-type-level breakdown:** Correlation recomputed separately for each service request type with ≥400 records, to test whether the income–closure relationship varies by category
  - 10 request types showed statistically significant correlations (p < 0.05); quality-of-life requests (e.g. organic bin additions, tree planting) trended positive with income, while core infrastructure/multi-residential requests (e.g. potholes, garbage pickup) trended negative
  - **Caveat:** with 10 categories tested, some significant results may reflect multiple-comparisons noise rather than a true effect
- **Monthly volume by income group:** Request volume normalized per ward (to control for unequal group sizes: 14 high-income vs. 11 low-income wards) — high-income wards generated ~30-40% more requests per ward across all months, with alternative explanations (digital access, housing type, homeownership rate) discussed rather than assumed causal

### 4. Predictive Modeling
A binary classification model was built to predict whether a request is at risk of **not** being closed (`is_risky = 1` when status ≠ `Closed`).

- **Features:** ward, service request type, division, month (cyclically encoded via sine/cosine to preserve seasonal adjacency), and ward median income
- **Models trained:** Logistic Regression and Random Forest (both with `class_weight='balanced'` to address class imbalance)
- **Evaluation:** classification report, confusion matrix, ROC-AUC, and explicit sensitivity/specificity comparison
- **Model selection rationale:** because the project's goal is to proactively flag requests at risk of going unresolved, sensitivity (recall on the "at-risk" class) was prioritized over specificity, since failing to flag a truly at-risk request is more costly than a false alarm
- **Feature importance** was extracted from the Random Forest model to assess whether income is a strong predictor relative to service type, division, and season

## Key Findings

1. Toronto-wide, income shows only a weak and statistically insignificant relationship with overall 311 closure rate.
2. When broken down by request type, several categories show significant, opposite-direction correlations — suggesting the income–service relationship is category-dependent rather than uniform.
3. Higher-income wards generate substantially more 311 requests per capita/per ward even after normalization, a pattern that should not be read as evidence of worse service in lower-income areas without further investigation (e.g. digital access, housing type).
4. In the predictive model, [fill in: state which feature ranked highest in `importance_df` — likely service request type or division rather than income] was the strongest predictor of closure risk, suggesting request category matters more than neighborhood income alone.

## Limitations

- Census income data (2021) and 311 data (2021) are well-aligned, but ward-level aggregation limits statistical power (n = 25)
- Correlational analysis cannot establish causation
- Multiple comparisons across request types increase the risk of false positives among "significant" findings
- The dataset used lacks a request closure timestamp, so resolution time could not be measured directly; closure rate was used as a proxy for service completion
- Income group comparisons rely on a simple median-split; a continuous income model may reveal finer-grained patterns

## Tools & Libraries

`pandas`, `numpy`, `scipy`, `matplotlib`, `seaborn`, `geopandas`, `scikit-learn`, `joblib`

## How to Reproduce

1. Download the three datasets listed above into a `data/` folder
2. Open `toronto_311_requests.ipynb` and run cells sequentially
3. Trained models and encoders are saved to `models/` via `joblib` for reuse without retraining

## Author

Mert AYDIN — prepared as part of an independent research project.