# <big><strong>🔍 Claude's Data Science Portfolio</strong></big>

Welcome! I'm Claude, a data scientist with a business lens. 

It's a curated selection of end-to-end projects that span machine learning, behavioral modeling, A/B testing, forecasting, and recommendation systems. Each project reflects my focus on building **interpretable, business-driven solutions** through deep data exploration and thoughtful modeling.

- 💼 **Business Use Cases**: Customer value modeling, product iteration, growth forecasting  
- 🧠 **Machine Learning**: Classification, regression, unsupervised clustering  
- ✨ **End-to-End Analysis**: From data preprocessing and feature engineering to insight generation  
- 📈 **Visualization & Communication**: Clear storytelling for stakeholders

---

## <big><strong>📬 Let's Connect</strong></big>

If you're a recruiter, teammate, or fellow data enthusiast, feel free to explore these projects and reach out if you'd like to collaborate or chat about any of these ideas!.  

I’m passionate about solving real-world problems through thoughtful analysis and scalable models.

👉 [Connect with me on LinkedIn](https://www.linkedin.com/in/claude-zz-wang/)

---

| Project                                                      | Topic                                  | Skills & Tools                                               | Data Link                                                    |
| ------------------------------------------------------------ | -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Full-Stack LTV Modeling for SaaS](#-behavior-driven-ltv-modeling-for-b2b-saas) | B2B Lifetime Value, Retention & Growth | Feature engineering, clustering, XGBoost, rule-based modeling | [Link](https://drive.google.com/drive/folders/1U5BdXPyVNrz6sj4dK6-ZIvIhofZrK2Ms?usp=drive_link) |
| [Game Design Experiment](#-ab-testing-in-mobile-game-design) | A/B Testing, Retention, Engagement     | Hypothesis testing, bootstrap, visualization                 | [Link](https://drive.google.com/drive/folders/1C8HnbdoFgt83fhCzPXYCp938QK_uKAv0?usp=drive_link) |
| [Recommender System](#-news-recommender-system-with-click-prediction) | Recommendation System, Click Behavior  | ItemCF/UserCF, CTR modeling, ranking, Faiss                  | [Link](https://drive.google.com/drive/folders/1Zkwe0_W8h86adkzXer42t2e-3nPvuuQU?usp=drive_link) |
| [UGC Analysis w/ NLP](#%EF%B8%8F-nlp--network-analysis-on-yelp-ugc) | NLP, Social Graph, Geospatial          | TextBlob, NetworkX, Folium                                   | [Link](https://drive.google.com/drive/folders/1YL5eo2ko3elyS5r0fC879d74XDuvJV40?usp=drive_link) |
| [Cash Flow Forecast](#-forecasting-purchase--redemption-for-cash-flow-management) | Time Series, Purchase/Redemption       | Time features, anomaly detection, seasonal patterns          | [Link](https://drive.google.com/drive/folders/1W9GH6-MUSChpsbXFIKSXnK5KmKkWIVhb?usp=drive_link) |


---

## <big><strong>🛠️ Tools & Techniques</strong></big>

`Python` `SQL` `XGBoost` `LightGBM` `EDA` `K-Means` `Logistic Regression`  
`Hypothesis Testing` `NLP` `Time Series Forecasting` `CTR Modeling` `Network Analysis`  
`Folium` `Seaborn` `Matplotlib` `Pandas` `Scikit-Learn`

---

## <big><strong>📁 Project Highlights</strong></big>

### 📊 Behavior-Driven LTV Modeling for B2B SaaS

> *Built a forward-looking Customer Lifetime Value model for Paylocity based on billing, retention, and product adoption signals.*

- Designed a closed-form DCF model with **personalized retention** and growth rates derived from **XGBoost** and business rules.
- **Engineered behavior features** from billing data to model upsell, cross-sell, and tenure with **strong interpretability**.
- Segmented clients using **k-means++** clustering to support sales planning and customer success prioritization.
- Conducted **sensitivity analysis** and calculated segment-level LTV/CAC ratios for **ROI optimization**.
- Delivered insights directly integrated into Salesforce for **client scoring** and resource allocation.

📂 [Read More](https://github.com/ZzClaude/DS-Portfolio/blob/main/01-Full-Stack-LTV-Pipeline/readme.md)

---

### <big><strong>🧪 A/B Testing in Mobile Game Design</strong></big>

> *Evaluated user retention and engagement impact of gate relocation in a puzzle game via controlled A/B testing.*

- Ran hypothesis tests (t-test, z-test, bootstrap) on gameplay and retention across 90,000+ players.
- Identified a 5% drop in Day-7 retention after gate delay, advising against rollout despite positive Day-1 metrics.
- Designed AA test to validate randomization and eliminate selection bias.
- Visualized skewed engagement distribution and used statistical correction for outliers.
- Supported product decision-making through rigorous, interpretable experiment results.

📂 [Read More](https://github.com/ZzClaude/DS-Portfolio/blob/main/02-AB-Testing-For-Game-Design/readme.md)

---

### <big><strong>📰 News Recommender System with Click Prediction</strong></big>

> *Predicted user-article CTR through recall-ranking framework and rich user behavior features.*

- Reformulated click logs into CTR classification task and engineered session-based features.
- Implemented recall via ItemCF and UserCF; ranked using LightGBM with custom features.
- Addressed cold start via embedding similarity and re-click patterns.
- Evaluated with position-decayed hit score, optimizing for top-5 ranking accuracy.
- Achieved over 20% uplift in predicted CTR versus random baseline.

📂 [Read More](https://github.com/ZzClaude/DS-Portfolio/blob/main/03-Recommender-System/readme.md)

---

### <big><strong>🗺️ NLP & Network Analysis on Yelp UGC</strong></big>

> *Explored review sentiment, elite user influence, and geospatial trends across Yelp businesses and users.*

- Applied sentiment analysis on 1M+ reviews to adjust business star ratings, revealing rating inflation.
- Built elite user subnetwork using NetworkX to identify community clusters and influence hubs.
- Mapped business categories and check-in behavior via geospatial visualizations (Folium, Cartopy).
- Investigated “useful/funny/cool” reactions as proxies for credibility and social signals.
- Delivered multi-sided insights for platform governance, business marketing, and user experience design.

📂 [Read More](https://github.com/ZzClaude/DS-Portfolio/blob/main/04-NLP-Review-Analysis/readme.md)

---

### <big><strong>📉 Forecasting Purchase & Redemption for Cash Flow Management</strong></big>

> *Built forecasting models for financial inflows and outflows in a money-market fund system using time series signals.*

- Analyzed cyclical, holiday, and anomaly patterns from 2.8M+ transaction logs.
- Engineered time-aware features and implemented multivariate models including XGBoost and Prophet.
- Tuned forecasting to handle asymmetric penalty metrics based on prediction confidence.
- Enabled more stable fund allocation planning and optimized redemption risk control.
- Enhanced prediction accuracy over baseline by 15% in a real-world business simulation.

📂 [Read More](https://github.com/ZzClaude/DS-Portfolio/blob/main/05-Time-Series-Forecasting/readme.md)
