# Project Overview — Behavior-Driven LTV Modeling

This project was completed as part of the University of Washington’s MSBA Capstone in partnership with Paylocity, a B2B SaaS company that provides cloud-based HR and payroll solutions. Final presentation delivered to Paylocity’s Data Science and Marketing team in June 2025.

Our goal was to **build an actionable Customer Lifetime Value (LTV) model** based on real billing and product data. We aimed not only to estimate LTV at the individual client level but also to enable **strategic client segmentation and ROI measurement** to support sales prioritization, retention efforts, and long-term growth planning.

To do this, we built a closed-form LTV model based on discounted cash flows, refined its key parameters ($r$, $g$) through machine learning and clustering, and conducted sensitivity analysis to highlight business levers.

------

## The Problem We're Facing

### The Blind Spot in SaaS Practices

In B2B SaaS, acquisition is expensive, churn is sticky, and value is uneven.

Yet many companies—Paylocity included—still manage client strategy through backward-looking metrics like past billing or CRM tags. These indicators often fail to reflect the **forward-looking value** of a client, such as their retention potential or upsell opportunity.

Paylocity, like many SaaS companies, traditionally relies on **past-year revenue** to guide decisions. This backward-looking lens fails to reflect true **lifetime value (LTV)**: particularly when LTV is estimated using only broad averages like:

- **LTV = ARPU × Retention Rate**

But not all clients are equal. Even within the same revenue tier, their **future potential** can vary drastically.

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1DyIprnNsvrATVgCnDWAtIi2FD-znb8at&sz=s4000" width="500"> <br> <sub>Avg 1-Year Revenue vs. Estimated LTV: Undervaluation of Future Potential</sub> </div>

This chart shows a stark example: while average one-year revenue is **14.28**, the modeled LTV is **64.39** — over **4x** higher. That difference comes from factoring in:

- Longer tenure
- Retention likelihood
- Expected growth
- Discounting over time

Relying on current revenue leads to **undervaluing long-term clients**: and **overinvesting** in clients unlikely to stay.

### Misallocation Without Forward Value

Same revenue today does **not** mean same value tomorrow. But Paylocity’s current model treats them identically.

#### **What This Misses**

- **No way to distinguish** strategic vs. short-term clients
- **Upsell, retention, and acquisition** strategies are based on guesswork
- **No targeting of expansion signals** like score movement or billing growth
- Customer heterogeneity gets masked

> **All revenue looks equal: until it doesn’t.**

To unlock smarter resource allocation, Paylocity needs a **forward-looking, client-specific value model**: one that reflects **behavioral variation** and **growth trajectories**, not just last year’s numbers.

## Why care about Customer Lifetime Value?

Without a robust LTV model, sales reps are forced to prioritize accounts based on **visible size rather than strategic potential**, and finance teams miss the opportunity to benchmark **CAC payback or ROI** across client segments.

LTV is more than just a financial metric: it’s a decision tool. A well-structured LTV framework helps identify which clients to **retain**, **grow**, or **let go**, and supports **targeted acquisition** that scales efficiently.

> **“You can't manage what you can't measure.”**
>  Our goal was to turn LTV from an abstract concept into a data-driven, operationally usable tool.

### Our Key Questions

- What’s the true economic value of each client, accounting for retention and growth?
- What factors drive LTV most, and how sensitive is it to these drivers?
- How can we use LTV to **inform CRM prioritization** and **sales planning**?

------

## Our Approach: A Behavior-Driven LTV Pipeline

> ***Forecasting Value: Not Just Tracking Revenue***

### **Modeling Philosophy**

We designed a pipeline that learns from just **one year** of client behavior, yet forecasts each customer’s long-term value.

Instead of a static, average-based formula, we use a dynamic flow:
$$
LTV = \sum_{t=0}^{\infty} m \cdot \left[\frac{r(1 + g)}{1 + i}\right]^t \iff LTV = m \cdot \frac{1 + i}{1 + i - r(1 + g)}
$$
Where:

- $r$ is the **client-specific retention rate**
- $g$ is the **growth rate**
- WACC reflects **Paylocity’s discount rate**

### **Pipeline Architecture**

We constructed the entire modeling pipeline from scratch: transforming raw billing records into actionable strategic insights:

- Engineered behavioral features (e.g., product usage patterns, cross-sell signals)
- Predicted **retention** using lifespan modeling on churned clients
- Assigned **growth rates** based on maturity and product expansion indicators
- Used clustering to segment clients into strategic profiles (e.g., Prime, Climbers)
- Conducted ROI analysis through **LTV/CAC** to guide resource allocation

Our pipeline flowed as:

```css
Feature engineering (e.g. contract type, growth trends, lifespan)
        ↓
Behavior Signals
        ↓
Retention Rate Prediction (XGBoost)
        ↓
Clustering (K-Means++)
        ↓
Growth Rate Adjustment (Rule-based)
        ↓
LTV formula & sensitivity analysis
        ↓
CRM score export + segmentation → Strategic insights
```

This design enabled not just LTV prediction, but also **what-if simulations**, **segment-level trends**, and CRM scoring for day-to-day rep actions.

### **Takeaway**

You don’t need five years of data: just **one year of intelligent behavior signals** is enough to uncover:

- High-potential accounts
- Risky churners
- Strategic opportunities hidden in plain sight



---

# 1. Data Source, Validity Checking & Preprocessing

This section provides a comprehensive overview of the input data used in our modeling process, the rigorous validation checks performed to ensure data quality, and the preprocessing steps taken to derive modeling-ready variables. These steps were crucial in enabling robust and interpretable lifetime value modeling.

------

## 1.1 Data Sources

We utilized four primary data sources, each contributing a different lens into client behavior and value:

| **Data Type**      | **Description**                                              | **Source**          | **Notes**                                          |
| ------------------ | ------------------------------------------------------------ | ------------------- | -------------------------------------------------- |
| **Billing**        | Invoice-level billing records across FY21–FY25               | NetSuite            | Reflects actual **cash received**, not bookings    |
| **Client Profile** | Account-level segment, headcount, industry, lifecycle status | Salesforce          | Two variants: `profile.csv` and `profile_full.csv` |
| **Account Health** | Yearly account status (Green, Yellow, Red, etc.) + time spent in each | Paylocity Health DB | Used to model behavioral churn risk                |
| **Estimated CAC**  | Segment-based customer acquisition cost estimates            | Provided by client  | Based on conversion benchmarks, not actual spend   |

All data was provided at the **individual account level**, with no parent-child rollups or partner mapping.

------

## 1.2 Validity Checking

To ensure modeling integrity, we performed extensive validation on the source data:

### 1.2.1 Cross-Table Coverage Analysis

We evaluated the row and unique `company_id` coverage across six datasets:

```sql
select
  t_name,
  row_cnt,
  id_cnt
from (
    select 'profile' t_name, count(*) row_cnt from company_profile
    union all
    select 'profile_full', count(*) from company_profile_no_filter
    union all
    select 'billing', count(*) from company_billing
    union all
    select '5_year_billing', count(*) from historical
    union all
    select '5_year_billing_line', count(*) from historical_by_line_type
    union all
    select '5_year_billing_product', count(*) from historical_by_product_group
) t1
join (
    select 'profile' t_name, count(distinct company_code) id_cnt from company_profile
    union all
    select 'profile_full', count(distinct company_code) from company_profile_no_filter
    union all
    select 'billing', count(distinct company_id) from company_billing
    union all
    select '5_year_billing', count(distinct company_id) from historical
    union all
    select '5_year_billing_line', count(distinct company_id) from historical_by_line_type
    union all
    select '5_year_billing_product', count(distinct company_id) from historical_by_product_group
) t2 using (t_name)
```

| Table                     | Row Count | Unique Company IDs |
| ------------------------- | --------- | ------------------ |
| `profile`                 | 92,738    | 92,738             |
| `profile_full`            | 167,660   | 167,660            |
| `billing`                 | 213,978   | 110,068            |
| `billing_history`         | 239,360   | 123,688            |
| `billing_history_line`    | 568,257   | 123,688            |
| `billing_history_product` | 3,230,728 | 123,688            |

This confirms structural alignment of billing and product-level records by `company_id`.

### 1.2.2 Profile vs Billing: Active Client Discrepancy

Among “active clients” (defined as `account_stage == "Active Client"` and `company_enddate == NULL`):

| Match Status    | % of Active IDs |
| --------------- | --------------- |
| Found in both   | 54.72%          |
| Only in profile | 11.23%          |
| Only in billing | 34.05%          |

We removed the 11% from profile-only set (likely false positives), and assumed churn for the 34% billing-only group.

### 1.2.3 Profile vs Billing (All IDs)

Comparing billing history against the full unfiltered profile:

| Match Status    | % of All IDs |
| --------------- | ------------ |
| Found in both   | 68.16%       |
| Only in profile | 28.61%       |
| Only in billing | 3.23%        |

We removed the 3.23% of billing-only IDs with no profile match.

### 1.2.4 End Date on Active Accounts

Among “active clients”, 0.8% had a non-null `company_enddate`. Client clarified this was due to billing lag (e.g., future-dated termination), and negligible in impact.

### 1.2.5 Account Health Coverage

Joining account health data with profile revealed:

| Join Status      | Account IDs |
| ---------------- | ----------- |
| Matched          | 77,229      |
| Missing Health   | 46,459      |
| Health w/o match | 0           |

Health data covers ~62% of billing population; missing values handled in downstream modeling via flagging and weighted scoring.

------

## 1.3 Preprocessing & Feature Engineering

We engineered multiple modeling variables from raw inputs, categorized as follows:

### 1.3.1 Billing Aggregation

- Aggregated FY21–FY25 billing values to client level
- Calculated:
  - `total_billing`, `avg_billing`
  - `tenure_billing` (count of fiscal years with non-zero billing)

### 1.3.2 Tenure Construction

- Merged in `company_startdate` and `company_enddate`
- Calculated `tenure = end - start` in years
- Where invalid or missing, fallback to `tenure_billing`

### 1.3.3 Retention Modeling

- **Portfolio Retention Rate**:
   $r_t = \frac{\text{Active clients in year } t \cap t-1}{\text{Active clients in year } t-1}$
   → Average: **89.22%**
- **Individual Client Retention Estimate**:
  - If churned: $1 - \frac{1}{\text{tenure}}$
  - If active: impute using 75th percentile of historic $r$
  - → `r_client` average: **70.04%**

### 1.3.4 Growth Rate (CAGR)

- CAGR from FY21 → FY25 using:

  ```python
  def safe_cagr(start, end, years):
      if start <= 0 or end <= 0:
          return 0
      return (end / start) ** (1 / years) - 1
  ```

- Stored as `growth`

### 1.3.5 New Client & Revenue Rates

- Marked new clients each year: appeared in FY(n), not in FY(n-1)
- Computed:
  - `new_client_rate` per FY
  - `net_new_client_rate = new - (1 - r)`
  - `new_revenue_rate = new_revenue / total_revenue`

| Metric              | FY22   | FY23   | FY24   | FY25   | Average |
| ------------------- | ------ | ------ | ------ | ------ | ------- |
| New Client Rate     | 22.77% | 22.48% | 17.98% | 15.51% | 19.69%  |
| Net New Client Rate | 11.87% | 11.47% | 6.82%  | 5.48%  | 8.91%   |
| New Revenue Rate    | 13.97% | 15.00% | 9.11%  | 6.89%  | 11.24%  |

Also calculated average new client rate by segment (Growth, Majors, Enterprise).

### 1.3.6 CAC Assignment

- CAC values provided by client:

| Segment    | CAC Value |
| ---------- | --------- |
| Growth     | $6.00     |
| Majors     | $5.01     |
| Enterprise | $14.28    |

- Used for future LTV / CAC ratio analysis

### 1.3.7 Headcount Mapping

Mapped `calculated_segment` to estimated headcount:

| Segment    | Proxy Headcount |
| ---------- | --------------- |
| Growth     | 50              |
| Majors     | 250             |
| Enterprise | 500             |
| Missing    | Default to 50   |

### 1.3.8 Health Risk Score Construction

- Normalized days in Green, Yellow, Red, etc. (per fiscal year)

- Assigned ordinal weights:

  - Green=0, Yellow=1, Red=2, Grey=3, Black=4, Blue=N/A

- Computed weighted average risk score using:

  ```python
  def compute_risk_score(row):
      score = 0
      total_weight = 0
      for status, weight in status_weights.items():
          proportion = row.get(status, 0)
          if weight is not None:
              score += weight * proportion
              total_weight += proportion
      return score if total_weight > 0 else None
  ```

- Produced:

  - `risk_2024`, `risk_2025`
  - `risk_avg`, `risk_wvg`, `risk_diff`, `risk_pct_change`
  - `is_new_2025`: flag for new clients with missing 2024 risk

### 1.3.9 Cross-Sell Signal

Constructed from product group adoption sets per year:

- `cross_sell_flag`: adopted new product group after initial year
- `cross_sell_yr_count`: number of YoY new product additions
- `cross_sell_count`: number of unique product groups added

Used product group set logic to identify net new groupings year over year.

### 1.3.10 Upsell Signal

Defined as significant billing growth in same product group:

- Computed CAGR for each `(company_id, product_group_1)`
- Marked as upsell if CAGR > 10%
- Aggregated to client-level:

| Variable            | Description                           |
| ------------------- | ------------------------------------- |
| `upsell_flag`       | Any product group upsold (Boolean)    |
| `upsell_count`      | Count of upsold product groups        |
| `avg_upsell_growth` | Average CAGR across all product lines |



------

# 2. Retention Modeling

Retention modeling serves as a core element in accurately forecasting customer lifetime value (LTV). Our goal was clear: **predict the expected lifespan of each client** to derive a realistic retention rate, directly impacting the LTV calculation. This section outlines how we approached the problem—starting with baseline modeling, identifying feature limitations, and ultimately improving predictive performance through behavioral feature engineering.

## **2.1 Baseline Modeling with Profile Features**

We began by training two baseline models to predict client tenure using only readily available **profile-level features**:

- **Features Used**: Segment, industry, headcount, region
- **Models Tested**: Ridge Regression (regularized linear model), XGBoost (non-linear benchmark)
- **Evaluation Metric**: Coefficient of Determination ($R^2$)

To improve data quality and reduce leakage:

- We applied **target encoding with 5-fold cross-validation** to all categorical features.
- We conducted a **stepwise VIF check** to eliminate multicollinear predictors (threshold = 10).
- This process left us with very few usable features, confirming the **limited predictive power of profile-only data**.

**Baseline Results**:

- Ridge Regression: $R^2$ ≈ 0.06
- XGBoost: $R^2$ ≈ 0.23

> These results revealed that traditional profile attributes are useful for **segmentation**, but insufficient for **predicting outcomes** like retention or tenure.

------

## **2.2 Behavioral Feature Engineering**

To improve retention prediction, we engineered features that serve as **behavioral proxies**, leveraging billing and product purchase history in the absence of direct usage logs (e.g., login events, NPS scores).

### 🔧 Feature Design Strategy

We focused on three dimensions of client behavior:

| Behavioral Aspect          | Proxy Features                                               |
| -------------------------- | ------------------------------------------------------------ |
| **Engagement Depth**       | `cross_sell_flag`, `cross_sell_count`, `upsell_count`, `avg_upsell_growth` |
| **Product Diversity**      | `bundle_count`, `product_count`                              |
| **Stability & Stickiness** | `billing_variability`, `recurring_ratio`                     |

**Examples**:

- `cross_sell_flag`: Indicates whether a client adopted new product lines over time.
- `avg_upsell_growth`: Measures average billing expansion within existing product categories.
- `billing_variability`: Captures fluctuations in yearly billing over FY21–FY25.
- `recurring_ratio`: Proportion of revenue tied to recurring subscription-based charges.

> These features were carefully selected to **approximate engagement behavior**, enabling the model to detect signs of growth, diversification, and retention even without direct interaction data.

------

## **2.3 Model Performance After Feature Engineering**

We retrained the models using the expanded feature set. The impact was substantial:

| Model   | R² Before <br />Feature Engineering | R² After <br />Feature Engineering | Impact    |
| ------- | ----------------------------------- | ---------------------------------- | --------- |
| Ridge   | 0.0618                              | 0.4711                             | 8× Better |
| XGBoost | 0.2344                              | 0.6812                             | 3× Better |

**Key Insights**:

- Behavioral proxy features drove significant lift in predictive power.
- `cross_sell_flag` and `avg_upsell_growth` alone explained a large portion of tenure variance.
- The feature importance rankings and SHAP analysis (see next section) confirm that **engagement-driven features outperform static attributes** like industry or region.

> This modeling milestone validated our hypothesis: **retention is a behavioral outcome**, and can be effectively predicted with well-engineered behavioral signals.

---

## **2.4 Feature Attribution via XGBoost Gain & SHAP**

To validate the impact of our engineered features and ensure the model remains interpretable, we performed **global feature attribution** using both **XGBoost's gain-based importance** and **SHAP values**.

<div align="center">     <img src="https://drive.google.com/thumbnail?id=1ukisDFjNKSnMooebYkH2EiwCmDvJaZAj&sz=s8000" width="800"><br>   <sub>Feature importance by gain (XGBoost)</sub>   </div>

The top contributors reflect engineered behavioral proxies rather than raw demographic inputs. Features like `billing_variability`, `cross_sell_flag`, and `avg_billing` dominate the model’s gain-based learning process—indicating that retention behavior is best captured by **inferred engagement patterns**.

------

### SHAP Analysis for Model Explainability

SHAP (SHapley Additive exPlanations) further quantifies each feature’s impact on individual predictions.

<div align="center">     <img src="https://drive.google.com/thumbnail?id=1bKyIt-6Sq_H39vb68RUZlZ2g6QGcoeye&sz=s8000" width="600"><br>   <sub>Average SHAP value for each feature (global importance)</sub>   </div>

- **`billing_variability`** contributes the highest average SHAP value (**+0.40**), confirming that **billing fluctuation** serves as a strong proxy for long-term engagement.
- **`cross_sell_flag`** and **`avg_billing`** also emerged as key signals—indicating that **product expansion** and **contract size** closely relate to retention outcomes.
- Meanwhile, profile-based variables such as `sector`, `product_count`, and `risk_avg` showed weaker contributions.

<div align="center">     <img src="https://drive.google.com/thumbnail?id=1USbjVRCeE7yWXbyRMDGQEUkr7vMTU84e&sz=s8000" width="600"><br>   <sub>SHAP beeswarm plot — distribution of each feature's impact</sub>   </div>

The beeswarm visualization highlights the spread of SHAP values across samples, revealing that:

- Higher `billing_variability` and active `cross_sell_flag` status robustly push tenure predictions upward.
- Some clients with high `avg_billing` show shorter expected tenure, possibly reflecting large but short-lived contracts.

<div align="center">     <img src="https://drive.google.com/thumbnail?id=1pV3JRBYhghLG2J-OBwSEYvkst71zFIap&sz=s8000" width="600"><br>   <sub>SHAP scatter plot for top feature: `billing_variability`</sub>   </div>

The SHAP scatter plot for `billing_variability` further confirms its non-linear effect—small to moderate variability boosts retention predictions the most, while extremely erratic patterns may signal account instability.

------

**Conclusion**:

This dual-layer analysis reinforces two conclusions:

- Our **behavioral feature engineering unlocked latent retention signals** absent from static demographics.
- The model remains **interpretable and aligned with business logic**, providing confidence for downstream deployment and stakeholder trust.

---

## **2.5 Distributional Patterns and Business Relevance**

To understand how retention patterns differ by customer size, we visualized the **distribution of predicted client lifespans** across the three key segments and its connection to financial return.

- **Inside Sales/Growth Markets**: <100 employees (blue)
- **Majors**: 100–499 employees (orange)
- **Enterprise**: 500+ employees (green)

<div align="center">  
  <img src="https://drive.google.com/thumbnail?id=1no1fplYy_ckZ6t9xeWr7b-4er95Sh4Pu&sz=s8000" width="800"><br>
  <sub>Predicted Tenure Distribution by Segment and Billing Relationship</sub>  
</div>

The **left panel** shows predicted tenure (in years) across different client segments. Most clients cluster tightly around the 5-year mark, but **Growth** clients display a more spread-out distribution—reflecting both potential and volatility.

The **right panel** plots tenure against **average billing**, revealing a positive trend: **longer-lived clients tend to spend more**, particularly in the **Enterprise** tier.

- The **Enterprise segment** tends to have a broader spread of predicted tenure and higher average billing, suggesting stronger long-term value.
- **Growth clients** (<100 employees) are tightly clustered around 3–5 years, with fewer long-lived accounts.
- The **Majors segment** lies in between, forming a bridge between smaller and larger accounts.

These patterns reinforce our modeling objective: by accurately forecasting client tenure, we lay the groundwork for **quantifiable LTV estimation**.



# 3. Cluster-Based Segmentation

While predicting **customer tenure** allows us to estimate retention, the second core component of LTV—**growth rate (g)**—is inherently more ambiguous.

Unlike retention, growth reflects **future potential** rather than historical behavior. Although billing expansion can be measured historically, it's often non-linear and constrained. For instance, a client that has already upsold heavily may have limited room left to grow.

> This makes growth **difficult to predict using traditional supervised models**, as past expansion may not be indicative of future opportunity.

------

## 3.1 Clustering Setup: Scaling + Initialization

Clustering requires well-behaved numeric inputs. We first diagnosed the distributional properties of selected behavioral and transactional features using skewness and kurtosis.

- **Highly skewed features** (e.g., `avg_billing`, `upsell_count`, `billing_variability`) were log-transformed using `log1p`.
- All features were standardized using **`StandardScaler`** to align magnitudes and improve distance-based clustering.

We adopted the **KMeans++** initialization method, which improves convergence and stability by selecting better initial centroids.

To determine the optimal number of clusters, we tested $k$ from **2 to 11** and evaluated both:

- **Elbow Method**: Measures within-cluster sum of squares (WCSS)
- **Silhouette Score**: Evaluates consistency within clusters

**Key Code Example**:

```python
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

# Feature preparation (billing and product usage patterns)
features = df[['annual_billing', 'product_count', 'cross_sell_count', 'recurring_ratio']]

# KMeans++ clustering
kmeans = KMeans(n_clusters=7, init='k-means++', random_state=42)
clusters = kmeans.fit_predict(features)
df['cluster_label'] = clusters

# Cluster validation
silhouette_avg = silhouette_score(features, clusters)
print(f"Silhouette Score: {silhouette_avg}")
```

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1nM6vMzRAZtKgeQYGIHzSCmU0Um62oRnf&sz=s4000" width="700"><br> <sub>Cluster validation metrics (Elbow and Silhouette analysis)</sub> </div>

Both methods converged around **7 clusters**, balancing cohesion and separation. The silhouette score peaked around **0.266**, which, while modest, indicated reasonable structure given the business context and numeric noise.

## 3.2 Cluster Profiles and Strategic Archetypes

We summarized the behavioral properties of each cluster to extract business-relevant insights:

<div align="center">     <img src="https://drive.google.com/thumbnail?id=1QXPOCTc_NIq67UFdXCoPiqvUChiHk04N&sz=s8000" width="600"><br>   <sub>Behavioral summary statistics by cluster (scaled)</sub>   </div>

<div align="center"><img src="https://drive.google.com/thumbnail?id=1teP1vgjNjeLemUe1IdtU2Dh-yBfA9idJ&sz=s4000" width="600"><br><sub></sub></div>

🟥 **Cluster 0 – All-Around Flagship Clients**

- High spend, broad and deep product usage
- High upsell + cross-sell activity
- Ideal for retention, referrals, and advocacy

🟧 **Cluster 1 – Breadth-Driven Stable Clients**

- Broad product usage, high billing
- Very low upsell behavior
- May benefit from upgrade prompts or education

🟨 **Cluster 2 – Non-Recurring Loyal Clients**

- Long tenure with non-subscription spending
- Fully active, medium billing
- Potential for conversion to recurring contracts

🟪 **Cluster 3 – Shallow Users with High Risk**

- Large product count but near-zero activity
- High churn score (`risk_2025`)
- Should be flagged for behavioral outreach

🟩 **Cluster 4 – Emerging Growth Clients**

- New, active users with high engagement ratio
- Low billing but broad product exploration
- High potential for upsell and education

🟦 **Cluster 5 – Silent Passive Clients**

- Moderate tenure, no growth indicators
- Minimal upsell/cross-sell
- Suitable for no-touch retention strategy

⛔ **Cluster 6 – Activation Drop-off Clients**

- Touched many features but never activated
- Lowest activity and highest churn risk
- Ideal for churn diagnosis and exit tracking

In the next section, we show how clustering and tree-based insights guide our **growth rate adjustment framework**, enabling LTV estimates that are grounded in both data and business reality.

------

## 3.3 Why Clustering Matters

Clustering was not used as a predictive model—but rather as a **strategic layer** to support:

1️⃣ **Growth Rate Adjustment**

In later stages of LTV modeling, we used clusters to calibrate growth rates:

- Clients in “high-potential” clusters were uplifted toward their peers’ growth trajectories
- Only if they had not already saturated product adoption

**This avoided overfitting to past growth trends**, which are often misleading for long-term forecasting.

2️⃣ **Behavioral Segmentation for Strategy**

Clustering helped us uncover strategic personas:

- Retainable but under-monetized groups
- At-risk clients with early disengagement
- Emerging users with growth readiness

These insights inform GTM strategies beyond just modeling:

- Prioritize onboarding
- Trigger proactive outreach
- Tailor pricing or packaging

------

> We conclude clustering as a supporting but crucial module: **linking behavioral patterns to future value**, and ensuring our LTV estimates are not only mathematically sound, but strategically aligned.

------

## 3.4 Interpretability and Strategic Regrouping

While our 7-cluster KMeans solution captured fine-grained behavioral differences, we regrouped them into **4 strategic segments** for presentation clarity and stakeholder alignment:

| Strategic Segment              | Clusters Included | Strategic Focus                                 |
| ------------------------------ | ----------------- | ----------------------------------------------- |
| **Prime** (High-Value Core)    | C0 + C1           | Flagship programs, referral, retention showcase |
| **Loyal** (Stable Base)        | C2 + C5           | Low-touch renewal, ARPU maintenance             |
| **Climbers** (Growth-Priority) | C3 + C4           | Onboarding, upsell nudges, reactivation alerts  |
| **Churning** (At-Risk)         | C6                | Exit tracking, disengagement diagnostics        |

> ⚠️ Note: For real-world operations, the original 7-cluster segmentation should be used, as it provides richer granularity for targeting and personalization.

------

## 3.5 Tree-Based Explanation of Strategic Segments

To enhance explainability, we trained a **decision tree classifier** using behavioral features to mimic the regrouped 4-segment classification.

<div align="center">     <img src="https://drive.google.com/thumbnail?id=16rDS5_0jLDhj82H2wCQoepKXAYoL7NJS&sz=s8000" width="800"><br>   <sub>Decision Tree for Strategic Segmentation (Prime, Loyal, Climbers, Churning)</sub>   </div>

The tree acts as an **interpretable surrogate model**, mapping client behavior to strategic groups in a human-readable format.

#### How the Tree Works (Simplified Explanation)

We **start with `recurring_ratio`**:

```markdown
- Low`recurring_ratio`(≤ 27%) → check `tenure`
  - If short → check `cross_sell_count`
    - None → **Churning**
    - Some → **Climber**
  - If long → check `risk_2025`
    - Low → **Loyal**, High → **Climber**

- High `recurring_ratio` → check `cross_sell_count`
  - None → check `active`
    - Inactive → **Churning**
    - Active → **Climber**
  - Some cross-sell → check `risk_2025`
    - Low → **Prime**, High → **Climber**
```

This shows our segmentation is not just descriptive — it’s **explainable and replicable**.
 We can apply this tree in real time to classify any new client quickly and consistently.

------



# 4. Growth Modeling

To simulate the long-term cash flow trajectory of each client, we modeled individual growth rates (`g_ltv`) using a tiered rule-based strategy. While every client starts from a baseline assumption aligned with macroeconomic trends, we selectively applied upward adjustments based on behavioral patterns and cluster-specific potential.

------

## 4.1 Baseline Growth Assumption

To begin our growth modeling, we established a conservative baseline assumption: every client was initially assigned a long-term annual growth rate of **4.4%**, roughly aligned with the historical U.S. inflation rate. This provided a neutral, risk-averse starting point for lifetime value (LTV) estimation: one that assumed no particular expansion behavior, but still captured general economic trends.

**Limitations**:

- A single growth assumption, while simple, doesn't reflect the varied potential across diverse client segments.
- A flat rate risked undervaluing strategically important, high-potential clients and misguiding investment strategies.

------

## 4.2 Adjusting Growth Rates Based on Cluster Insights

Assign customized growth rates based on cluster-specific behaviors and strategic considerations.

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1cv9VThgTV60GMQdw5jLaEShD84qEzw1G&sz=s5000" width="900"><br> <sub>Assignment Logic</sub> </div>

- **Condition for adjustment**: Only clients with product coverage below the 75th percentile (i.e., with room to grow).
- **Clients in Cluster 0 & 1**: Assigned each cluster’s historical average growth rate, reflecting proven expansion behavior.
- **Clients in Cluster 4**: Treated as cautious versions of Cluster 0, assigned conservative growth at the 25th percentile of Cluster 0’s historical growth.
- **All other clusters**: Maintained baseline growth rate (4.4%).

**Key Code Example**:

```python
# Define growth adjustment logic based on cluster and product coverage
def assign_growth_rate(row, cluster_growth_stats):
    coverage_threshold = row['product_coverage_percentile'] < 0.75
    if coverage_threshold:
        if row['cluster_label'] in [0, 1]:
            return cluster_growth_stats[row['cluster_label']]['mean_growth']
        elif row['cluster_label'] == 4:
            return cluster_growth_stats[0]['25th_percentile_growth']
    return 0.044  # baseline growth

# Applying growth assignment
df['adjusted_growth_rate'] = df.apply(lambda row: assign_growth_rate(row, cluster_growth_stats), axis=1)
```

### **Why This Matters**

This approach ensures growth expectations are grounded in behavior, not just history.

- **No overfitting**: We don’t extrapolate based on past billing alone.
- **Behavior-driven**: Only unsaturated and active clients are eligible for growth boosts.
- **Actionable segmentation**: Clusters with similar retention may still diverge in growth, enabling more granular strategy.
- **Conservative but dynamic**: Cluster 4 clients, though promising, are handled cautiously: highlighting our preference for long-term realism over short-term optimism.

---

# 5. Sensitivity Analysis & Strategic Insights

Customer lifetime value (LTV) is a nonlinear function shaped by three key parameters: **retention (r)**, **growth (g)**, and **discount rate (i)**. To understand which levers matter most, we conducted a sensitivity analysis using the final modeled values:
$$
LTV = m \cdot \frac{1 + i}{1 + i - r(1 + g)}
$$

------

## 5.1 Retention: The Exponential Lever

**Logic Flow**:

- Average retention in our model: **73.96%**, unweighted.
- Compared to Paylocity’s internal benchmark (~92%, revenue-weighted), our estimate offers a more realistic view of the broader customer base.
- Sensitivity coefficients were calculated as the percent change in LTV per 1% change in each variable.

<div align="left"> <img src="https://drive.google.com/thumbnail?id=1xb3KsewFh1oMMdw4woHlKCrNn7VQB7mE&sz=s4000" width="800">
  <br> <div align="center"><sub>LTV sensitivity to Retention Rate, with model baseline at 73.96%</sub> </div>

**Key Insights**:

- Even at a conservative baseline, **retention yields a sensitivity coefficient of 3.51**, meaning a 1% increase in retention yields a 3.51% gain in LTV.

- This impact becomes exponentially stronger as retention increases:

  | Retention (%)                          | Average_LTV                           | Sensitivity                          | Note           |
  | -------------------------------------- | ------------------------------------- | ------------------------------------ | -------------- |
  | <font color='darkorange'>73.96%</font> | <font color='darkorange'>64.40</font> | <font color='darkorange'>3.51</font> | *LTV estimate* |
  | 85.0%                                  | 135.20                                | 8.45                                 |                |
  | 88.0%                                  | 192.81                                | 12.48                                |                |
  | 89.0%                                  | 224.72                                | 14.71                                |                |
  | 92.0%                                  | 446.37                                | 30.22                                |                |

- The curve steepens dramatically beyond the 85% mark, indicating **nonlinear compounding effects**.
- This confirms: **Retention is the most powerful lever in LTV modeling**.

> 📌 Even modest retention gains unlock exponential LTV growth: no other variable comes close.

------

## 5.2 Growth: A Conditional Amplifier

**Logic Flow**:

- Average modeled growth rate: **15.41%**, based on cluster behavior and upsell indicators.
- Despite being higher than industry average (13.2%), **growth sensitivity remains below 1**, due to relatively low retention.

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1zmg2vZsP30YbP8bpJ7s3r07T0wGd3xXZ&sz=s4000" width="800"><br> <sub>Growth Sensitivity vs. Retention Interaction</sub> </div>

**Key Insight**:

- Growth influences LTV through the term $r(1 + g)$.
- When retention is low (e.g., ~74%), even large increases in g do not significantly move the needle.
- Only when retention exceeds ~88% does growth start behaving as a meaningful LTV amplifier.

| Growth (%)                             | Average_LTV                           | Sensitivity                          | Note               |
| -------------------------------------- | ------------------------------------- | ------------------------------------ | ------------------ |
| <font color='crimson'>4.4%</font>      | <font color='crimson'>48.24</font>    | <font color='crimson'>0.10</font>    | *inflation*        |
| 8.0%                                   | 52.55                                 | 0.20                                 |                    |
| 11.0%                                  | 56.78                                 | 0.29                                 |                    |
| <font color='crimson'>13.2%</font>     | <font color='crimson'>60.33</font>    | <font color='crimson'>0.38</font>    | *industry average* |
| <font color='darkorange'>15.41%</font> | <font color='darkorange'>64.37</font> | <font color='darkorange'>0.46</font> | *LTV estimate*     |
| 16.0%                                  | 65.56                                 | 0.49                                 |                    |

> Growth ≠ Value unless customers stick around long enough to realize it.

------

## 5.3 Discount Rate (WACC): Predictable but Passive

**Logic Flow**:

- Discount rate used: **9.69%**, based on NYU Stern’s [WACC benchmarks](https://pages.stern.nyu.edu/~adamodar/New_Home_Page/datafile/wacc.html).
- Modeled sensitivity ranges from **–0.20 to –0.41**, reflecting a **linear inverse effect**.

<div  align="center"><img  src="https://drive.google.com/thumbnail?id=1G6wai8V1OdYLdlUgCn-NmJFeuFooLYYB&sz=s4000"  width="800"><br><sub></sub></div>

**Key Insight**:

- Lowering WACC improves LTV predictably, but its effect is limited and not under direct operational control.
- As a market variable, WACC represents financing conditions rather than customer dynamics.

------

## 5.4 Strategic Takeaways

**Comparative Sensitivity Summary**:

| Variable  | Sensitivity Range | Turning Point | Interpretation         |
| --------- | ----------------- | ------------- | ---------------------- |
| Retention | 3.5 → 30.2        | Always > 1    | **Exponential driver** |
| Growth    | 0.1 → 0.49        | Never         | Conditional impact     |
| WACC      | –0.2 → –0.41      | Never         | Modest linear drag     |

**Business Implications**:

- **Prioritize Retention**:
  - Invest in engagement, loyalty programs, and churn early-warning systems.
- **Use Growth Strategically**:
  - Target it only in high-retention segments.
- **Treat WACC as a constraint**:
  - Plan around it, don’t bet on changing it.

> 💡 Bottom line: If you can only move one lever — move **retention**.



# 6. Business Strategy

This section links our LTV model back to **strategic and operational decisions**. Using the outputs from modeling and segmentation, we estimate **ROI by client type** and outline how these insights can be embedded directly into Paylocity’s systems — especially **CRM tooling like Salesforce** — to influence day-to-day action.

------

## 6.1 Strategic Playbook by Customer Segment

We translated the customer segments from clustering into a **clear strategic playbook** for Paylocity's go-to-market and retention strategies. This segmentation forms the basis for downstream targeting, product design, and LTV forecasting by **client group**. 

| Segment      | Strategy              | Tactics                                                      |
| ------------ | --------------------- | ------------------------------------------------------------ |
| **Prime**    | White-glove retention | CS prioritization, executive visibility, renewals automation |
| **Climbers** | Guided growth         | Onboarding boosts, targeted upsell, product usage nudges     |
| **Loyal**    | Low-touch maintenance | Regular check-ins, passive success monitoring                |
| **Churning** | Risk triage or exit   | Flag via diagnostics, retention playbooks, cost-effective offboarding |

<div  align="center"><img  src="https://drive.google.com/thumbnail?id=15Jr0lMs6coDoF59j-T94tLuC_Og-dK5C&sz=s4000"  width="600"><br><sub>Segment-level strategic recommendations based on LTV tiers</sub></div>

By comparing average LTV across segments, it becomes clear why differentiated strategies are essential: Prime clients deliver an average LTV of **88.4**, while Churning clients average only **11.1** — a nearly **8× difference** in value. Each group calls for tailored actions to maximize return or minimize loss.

The **Prime** segment, combining Clusters 0 and 1, represents high-billing clients with broad product adoption, strong upsell behavior, and long tenures. These are Paylocity’s flagship customers: active, embedded, and strategically aligned. They merit high-touch customer success, personalized expansion paths, and inclusion in referral or co-marketing programs. Retention here is not just about revenue protection, but also about amplification through advocacy.

The **Loyal** segment (Clusters 2 and 5) consists of stable but passive users. They don’t actively expand usage, yet they remain for the long term with consistent billing. These clients require minimal intervention but should not be overlooked. Low-effort engagement strategies such as auto-renew workflows, drip campaigns, and periodic check-ins can sustain their value. The key is to maintain their steady contribution while avoiding accidental churn.

The **Climbers** segment, derived from Clusters 3 and 4, shows a mix of signals. Cluster 4 indicates newly engaged, active clients exploring the platform, while Cluster 3 contains clients with shallow engagement and elevated risk. As a group, Climbers represent high variance and untapped opportunity. Strategic action here includes optimizing onboarding flows, providing educational nudges, setting churn risk watchlists, and triggering upsell campaigns at the right moments. Pilot programs and test offers can also help identify sub-segments worth promoting to Prime.

Finally, the **Churning** segment (Cluster 6) includes clients who disengaged shortly after onboarding. They show low activity, short tenures, and high churn risk. Rather than expending full customer success resources, the strategy for this group should prioritize efficient offboarding or feedback collection. Where possible, automated reactivation campaigns or last-touch offers may rescue value, but in most cases, this group is best handled through cost-controlled exit processes.

------

## 6.2 From Score to Action: CRM Embedding

To activate intelligence, we propose integrating model outputs into Paylocity’s CRM, such as Salesforce:

> **LTV Prediction + Segment Label**
>  → **Score Tag in CRM**
>  → **Action Playbooks & Prioritization**

This empowers frontline teams with actionable insights — helping them identify which accounts warrant more attention, which ones are at risk and need quick intervention, and how to allocate budgets and customer success resources for maximum return.

**Why this matters**:

- The CRM (e.g., Salesforce) is where reps make day-to-day decisions
- Embedding LTV + Segment scores = smarter resource allocation
- Let sales/CS see: *who to focus on, who’s at risk, who to grow*

**What we recommend**:

- Integrate LTV scores and segment labels into Salesforce client records
- Create workflows that:
  - Flag high-potential **climbers** for onboarding push
  - Set CS priorities by segment (e.g., Prime = high touch)
  - Trigger retention playbooks for Churning clients

------

## 6.3 Cumulative LTV vs. Time: Retention Builds the Base

We examined **LTV accumulation over time** using modeled parameters:

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1kdnnyzBEAC5Dx473rTFV_EB-jWo8QWWu&sz=s4000" width="800"><br> <sub>Cumulative LTV by Year (Sample Client)</sub> </div>

**Key Insights**:

- By year 5, **over 80% of lifetime value is already realized**.
- After year 8, marginal gains taper off.
- Retention drives value early, while **acquisition is needed to sustain momentum** beyond that point.

> Retention builds the core. Acquisition extends the frontier. 
>
> We need both to grow sustainably! 📈 

## 6.4 ROI by Segment: Prioritizing Where to Win

To assess which customer segments offer the best return on investment, we compared segment-level LTV to rough CAC (Customer Acquisition Cost) estimates provided by Paylocity. While not built from full attribution logic, these figures are directionally useful for investment prioritization.

- Used client-provided CAC estimates by segment (not modeled, treated as rough benchmarks).

- Calculated ROI as:
  $$
  \text{ROI} = \frac{\text{Avg LTV}}{\text{CAC}}
  $$

| Segment         | Avg LTV | CAC   | LTV/CAC   | Revenue % |
| --------------- | ------- | ----- | --------- | --------- |
| Growth (S1)     | 30.31   | 6.00  | 5.05      | 40%       |
| Major (S2)      | 172.69  | 5.01  | 34.47     | 43%       |
| Enterprise (S3) | 590.67  | 14.28 | 41.36     | 17%       |
| **Total**       | 64.39   | 6.04  | **10.66** | 100%      |

**Insights**

- **Enterprise clients** offer the highest ROI (41.4x), despite representing just **17% of total revenue**. This highlights them as high-efficiency but lower-scale opportunities — ideal for targeted investment.
- **Major clients** generate **43% of revenue** with strong returns (34.5x), striking a balance between scale and efficiency.
- **Growth clients**, while only returning 5x on CAC, make up **40% of revenue** — they are essential for sustaining volume but less efficient in isolation.

**Takeaway**
 This breakdown suggests a dual-track strategy:

- **Pursue efficiency** by deepening penetration in Enterprise and Major segments.
- **Protect scale** by sustaining the Growth segment, while improving onboarding or pricing efficiency.
   In short, not all revenue is created equal — and this view helps target the most strategic dollar.



# 7. Final Takeaways: From Insight to Action

As we close, we consolidate the key insights from our modeling journey into three strategic imperatives — turning analysis into action.

------

## 7.1 Retention Protects the Core

Customer value is **front-loaded** — especially for **Prime** and **Loyal** segments.
 That means:

- **Retaining** existing high-fit clients is the surest path to ROI.
- Invest in **proactive support**, not just reactive churn prevention.

------

## 7.2 Growth Multiplies Potential — If Retention Holds

**Climbers** represent the growth engine:

- Modest current billing, but strong signals of expansion.
- A few targeted **nudges** (onboarding help, usage milestones, upsell automation) can shift them toward **Prime** status.

Growth is not just about acquiring more clients, it’s about **growing the right ones**.

------

## 7.3 Acquisition Expands the Frontier — With Focus

Our clustering analysis and segment forecasting reveal that:

- Acquisition drives aggregate gains **only when qualified well**
- Not every new lead is worth the same, **targeted prospecting** and **real-time LTV scoring** ensure scalability doesn’t come at the cost of efficiency.

------

#### Final Thought

> Retention secures the base.
>  Growth scales potential.
>  Acquisition expands the frontier — if it's smart.

That’s how we turn customer behavior into business outcomes.

**Thank you for reading.**
