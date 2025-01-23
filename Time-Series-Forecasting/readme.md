# Revolutionizing Cash Flow Forecasts - A Paradigm Shift in Purchase and Redemption Predictions

<p align="center">
  <img src="https://miro.medium.com/v2/resize:fit:1101/1*Vd4TbG2Ocj4cZN2xOcMM8A.jpeg" alt="Image" width="600">
</p>

## Background

> Accurate cash flow forecasting is a critical component for manufacturing companies preparing for IPOs. Predicting financial trends, such as production, procurement, and sales-related cash flows, is essential to showcase financial stability and ensure alignment with investor expectations. By leveraging historical data and advanced forecasting models, this project supports IPO disclosures and enhances confidence among regulators and investors.

### Project Overview

This project was conducted for a manufacturing company undergoing IPO preparation. The objective was to forecast future cash flows related to production, procurement, and sales using historical financial data. The insights gained provided the company with evidence of robust financial management, supporting their IPO filing with transparent and reliable cash flow projections.

### Project Objectives

1. **Cash Flow Forecasting**: Develop machine learning models to predict daily cash inflows (sales), outflows (procurement), and operational expenses based on historical transaction data.
2. **Support IPO Disclosure**: Provide actionable insights for IPO documents, demonstrating revenue stability and effective capital allocation to regulators and potential investors.
3. **Operational Efficiency**: Optimize the company’s working capital by aligning cash flow forecasts with operational planning, reducing short-term capital requirements.

### Impact & Expected Results

#### Potential Benefits

1. **Strengthening IPO Confidence**:
   - Accurate forecasts help demonstrate financial stability and sustainable growth to regulators and investors.
   - Highlights a robust capital management strategy, showcasing the company's ability to manage operational challenges.

2. **Optimizing Operational Decisions**:
   - Predictions enable better planning for procurement and production, reducing risks of overstock or supply chain disruptions.
   - Aligns operational activities with sales projections, ensuring efficient resource allocation.

3. **Risk Mitigation**:
   - Prevents risks associated with cash flow volatility, such as delayed production or financial shortfalls.
   - Strengthens the company’s financial positioning by showcasing controlled cash flow management.

4. **Enhancing Competitive Edge**:
   - Demonstrates the company’s ability to adapt to market changes and operational demands, fostering investor confidence.

### Data Inspection

#### 1. Financial Transaction Table

A total of 2.8 million records covering cash flow behaviors related to production, procurement, and sales from July 2013 to August 2014 were analyzed. Data attributes include daily transaction details for different cash flow categories.

| Attribute         | Type    | Description                                | Example  |
| ----------------- | ------- | ------------------------------------------ | -------- |
| report_date       | string  | Transaction Date                          | 20140831 |
| sales_amount      | bigint  | Daily sales revenue                       | 500000   |
| procurement_cost  | bigint  | Daily procurement expense                 | 300000   |
| operational_cost  | bigint  | Daily operational expense                 | 200000   |

#### 2. Production Metrics Table

Detailed records of production output, inventory levels, and raw material usage were included to model the impact of production activity on cash flow.

| Attribute       | Type    | Description                                | Example  |
| --------------- | ------- | ------------------------------------------ | -------- |
| production_date | string  | Date of production activity                | 20140831 |
| output_units    | bigint  | Units produced                             | 15000    |
| inventory_level | bigint  | Remaining inventory after production       | 12000    |

#### 3. Historical Cost Data

The data includes historical operational costs, such as raw material prices, utility costs, and labor expenses, to capture trends affecting cash flow.

| Attribute         | Type    | Description                                | Example  |
| ----------------- | ------- | ------------------------------------------ | -------- |
| cost_date         | string  | Date of cost record                       | 20140831 |
| material_cost     | bigint  | Cost of raw materials                     | 250000   |
| utility_cost      | bigint  | Utility costs for production              | 50000    |
| labor_cost        | bigint  | Labor costs for the day                   | 100000   |

#### 4. External Economic Indicators

Macro-level indicators such as market demand, interest rates, and commodity prices are included to provide additional context for cash flow forecasting.

| Attribute           | Type    | Description                                | Example  |
| ------------------- | ------- | ------------------------------------------ | -------- |
| indicator_date      | string  | Date of economic indicator                | 20140831 |
| market_demand_index | double  | Index of market demand                    | 1.25     |
| interest_rate       | double  | Daily interest rate (%)                   | 3.5      |
| commodity_price     | double  | Price of key commodities ($/unit)         | 50.75    |


### Evaluation

The objective of this project is to forecast daily cash inflows (sales) and outflows (procurement and operational expenses) for the next 30 days with high accuracy. Accurate predictions are crucial for IPO preparation, as they provide insights into financial stability and operational efficiency, which are key factors for investor confidence.

#### Scoring Function

1. **Relative Error Calculation**:
   - Compute the relative error between actual and forecasted values for key metrics (sales, procurement, and operational costs).

   $$\text{Relative Error} = \frac{\lvert \text{Actual Value} - \text{Predicted Value} \rvert}{\text{Actual Value}}$$

2. **Weighted Scoring**:
   - Errors are penalized based on business criticality (e.g., procurement errors during high-demand periods receive higher weights).
   - A scoring function $f(*)$ is applied to evaluate daily predictions, with higher penalties for larger errors.

3. **Final Score Calculation**:
   - Aggregate the weighted daily scores for each cash flow category (sales, procurement, and operational costs).
   - Apply business-specific weights to prioritize metrics critical for IPO reporting.

   $$\text{Total Score} = w_1 \cdot f(\text{Sales Error}) + w_2 \cdot f(\text{Procurement Error}) + w_3 \cdot f(\text{Operational Cost Error})$$

---

## Project Phases

### 1. Data Processing & Exploratory Data Analysis (EDA)

- **Time Series Analysis and Visualization**  
  Analyzed cyclical patterns in cash flows related to sales, procurement, and production activities to identify seasonality and trends.

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/bd3aaba0-135c-475a-ba3b-c0681e29400f)

- **Difference in Sales and Procurement Total Amounts from Monday to Sunday**  
  Explored weekly patterns to detect variations in cash flow distribution.

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/a66e9deb-b701-44c5-a5a7-fbd51769e5fb)

- **Month Feature Analysis**  
  Assessed monthly trends to identify cash flow seasonality and its alignment with operational cycles.

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/d991c6ba-78e7-4f0e-abfc-6d296969d241)

- **Date Feature Analysis**  
  Investigated daily cash flow fluctuations to capture high and low activity periods.

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/31d287e2-0b14-4e17-8114-98201596aa96)

- **Holiday Analysis**  
  Examined the impact of public holidays on sales and procurement activities.

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/2d5e37a9-8404-4afd-92f4-469ceb44dbe1)

- **Analysis of Large Anomalous Transactions**  
  Investigated outlier transactions to mitigate their impact on forecast accuracy.

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/a25f4db3-8f24-418b-b38e-bd642786573a)

- **Correlation Analysis**  
  Evaluated correlations between key cash flow metrics and external factors (e.g., interest rates, market demand).

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/f9fba23b-b652-4a7e-97d2-8359b32cdfee)

- **Analysis of Bank and Operational Interest Rates**  
  Assessed the relationship between cash flow trends and borrowing costs.

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/511e17d5-c143-439d-818d-4b7fcd883324)

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/5f89cb95-74c6-4425-b352-328eae65851b)



### 2. Model Baseline

Given the pronounced cyclical patterns in the cash flow data of the manufacturing company, this project adopted a **Cyclical Factor Model** as the baseline for forecasting.

#### Key Steps in the Model:

1. **Calculation of Cyclical Factors (factors):**
   - Captures recurring patterns (e.g., weekly sales peaks, monthly procurement spikes) observed in the cash flow data.
   
2. **Calculation of Baseline (base):**
   - Determines the underlying trend of cash flow behavior independent of seasonal or cyclical effects.
   
3. **Forecasting Process:**
   - Future cash flows are predicted by multiplying the baseline values with corresponding cyclical factors:
     $$\text{Forecast} = \text{Base} \times \text{Factors}$$

#### Example Scenario:

Below is a simplified example of calculating cyclical factors for weekly cash flow forecasting:

| Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday | Weekly Average |
| ------ | ------- | --------- | -------- | ------ | -------- | ------ | -------------- |
| Week 1 | 20      | 10        | 70       | 50     | 250      | 200    | 100            |
| Week 2 | 26      | 18        | 66       | 50     | 180      | 140    | 80             |
| Week 3 | 15      | 8         | 67       | 60     | 270      | 160    | 120            |

#### Steps:

1. **Calculate Proportional Factors:**
   - Divide each day’s cash flow by the weekly average to compute proportions.

2. **Determine Cyclical Factors:**
   - Use the median of daily proportions across weeks to calculate robust cyclical factors.

3. **Generate Forecast:**
   - Multiply the cyclical factors by a base value (e.g., average cash flow for the most recent week).

#### Optimization:

- **Weighted Blending:** Blend mean and median values based on performance during testing to improve accuracy.
- **Dynamic Baseline Adjustment:** Optimize the baseline using a subset of recent data to account for operational changes.


### 3. Feature Engineering

#### **Static Feature Extraction**

Derived static features from time-related attributes to capture meaningful patterns influencing cash flow, including:

- **Date-Based Features**:
  - `is_weekend`: Indicates whether the day falls on a weekend (Saturday or Sunday).
  - `is_holiday`: Indicates whether the day is a public holiday.
  - `is_firstday_of_month`: Indicates whether the day is the first day of the month.
  - `is_lastday_of_workday`: Indicates whether the day is the last workday before a holiday.

- **Month Period Features**:
  - `is_premonth`, `is_midmonth`, `is_tailmonth`: Identifies whether the day belongs to the beginning, middle, or end of the month.
  - `is_first_week`, `is_second_week`, `is_third_week`, `is_fourth_week`: Indicates which week of the month the day falls into.

![image](https://ice.frostsky.com/2023/08/19/5b8b7f979b2bae65fd5c44c1f10016cd.png)

---

#### **Distance-Based Features**

Enhanced the dataset by calculating temporal distances to key reference points, such as holidays and working days, to contextualize cash flow behaviors:

- `dis_to_holiday` and `dis_from_holiday`: Distance to the next and previous public holidays.
- `dis_to_work` and `dis_from_work`: Distance to the next and previous working days.
- `dis_from_startofmonth` and `dis_from_endofweek`: Distance from the start of the month and the end of the week.

![image](https://ice.frostsky.com/2023/08/19/017de87ec8dd269f5913c38143e0f7f4.png)

---

#### **Cycle-Related Features**

Captured patterns in user behavior by calculating average purchase and redemption rates for different time intervals (e.g., weekdays, days of the month). Weighted rates were used to uncover specific cyclical patterns affecting cash flow.

---

#### **Dynamic Behavior Features**

Extracted dynamic features reflecting the variability of purchase behaviors over time:

- Computed statistics (mean, median, max, min, standard deviation) for purchase and redemption behaviors across rolling windows.
- Included skewness to capture asymmetry in cash flow distributions.

---

#### **Feature Selection**

Removed multicollinear and non-informative features to improve model interpretability and predictive power. The winning features were selected based on their ability to enhance forecasting accuracy.

![image](https://ice.frostsky.com/2023/08/19/5d82c469e0cbf7c618e62377df748ada.png)



### 4. Model Development & Comparison

Developed a diverse set of over 10 systematic forecasting models, including univariate and multivariate approaches, that delivered unparalleled predictive insights into future cash flows.

Utilized innovative techniques, such as weighted averaging of purchase and redemption errors, leading to the identification of the best-performing model—LSTM—exhibiting an impressive uplift in predictive accuracy.

![image](https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/bdaebde3-8af0-4c43-a9b1-05ee6a01887d)


<p align="center">
  <img src="https://ice.frostsky.com/2023/08/19/ebc04d85bdb7e8fb8da494e39a6c9b9d.png" alt="Image" width="600" height="310">
</p>


<p align="center">
  <img src="https://github.com/datoujinggzj/DS_Project_Portfolio/assets/99417740/ff60b0cf-40bf-46bb-a821-f570545e002d" alt="Image" width="600" height="310">
</p>
