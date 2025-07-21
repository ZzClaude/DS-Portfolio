# Empowering User Experience - Optimizing Product Recommendations through AB Testing


<p>
  <img src="https://miro.medium.com/v2/resize:fit:1400/1*VhPfZIYkcnV-vWZYaAvgjg.jpeg" alt="Image" width="600">
</p>



## <big><strong>1. Project Overview</strong></big>

### <strong>1.1 Context and Motivation</strong>

In mobile game design, **progression gates**—levels that temporarily block advancement—are often used to pace user engagement or encourage in-app purchases. While effective for monetization, such design choices can negatively impact user retention if not well-tuned.

This project investigates the impact of shifting the **first gate** in the puzzle game *Cookie Cats* from **level 30 (control group)** to **level 40 (test group)**. The business hypothesis was that delaying the gate might lead to more initial engagement and higher conversion before encountering friction.

To evaluate this, an A/B test was conducted on **90,189 players**, randomly assigned to either the original gate at level 30 (`gate_30`) or the experimental version at level 40 (`gate_40`). We examined how this design change affected:

- **Short-term engagement**: measured by total game rounds played
- **User retention**: at 1-day and 7-day marks

The results from this test ultimately informed a **data-driven product decision** on whether to release the new gate configuration.

### <strong>1.2 Experiment Design</strong>

The A/B test was set up with a **between-subject design**:

- **Control group (`gate_30`)**: Players encounter the first gate at level 30.
- **Test group (`gate_40`)**: The gate is postponed to level 40.

Each group contained approximately **45,000 players**, ensuring statistically comparable sample sizes.

We tracked the following core metrics:

- `sum_gamerounds`: total number of game rounds played in the first 14 days post-install
- `retention_1`: whether the user returned on Day 1
- `retention_7`: whether the user returned on Day 7

Players were randomly assigned to groups at install time. **Randomization quality** and **group balance** were later verified (see Section 4).

The core analysis pipeline includes:

- Summary statistics and visualization
- Hypothesis testing: t-test, chi-square, and Mann-Whitney U
- Proportion testing (z-test for retention rates)
- Bootstrap-based validation

By combining traditional inference with resampling methods, we aim to **robustly assess user behavior changes** and inform monetization strategies.



## <big><strong>2. Data Overview & Preprocessing</strong></big>

### <strong>2.1 Dataset Summary</strong>

The dataset contains **90,189 rows**, each representing a unique player who installed the game during the A/B test period. Key variables include:

| Column           | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| `userid`         | Unique player identifier                                     |
| `version`        | Group assignment: `gate_30` or `gate_40`                     |
| `sum_gamerounds` | Total game rounds played within the first 14 days post-install |
| `retention_1`    | Boolean indicating Day-1 retention                           |
| `retention_7`    | Boolean indicating Day-7 retention                           |

Initial group sizes were balanced:

- `gate_30`: 44,700 players
- `gate_40`: 45,489 players

Basic descriptive statistics revealed a **highly skewed distribution** in `sum_gamerounds`. While the median was around 16–17 rounds, the **maximum value reached over 49,000**, suggesting extreme outliers.

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1CfGqvFGRB3ZthbOmmQFC_WpGLMeAT0HU&sz=s4000" width="800"><br>   <sub>Figure 1. Distribution of `sum_gamerounds` before removing outliers</sub> </div>

> **Insight:** The heavy right skew can distort averages and hypothesis testing, especially since a few hyperactive users disproportionately impact the mean.

------

### <strong>2.2 Outlier Removal</strong>

To mitigate the influence of extreme values, we removed the single maximum data point (`sum_gamerounds` = 49,854). This adjustment reduced the maximum to **2,961 rounds**, with negligible change to the rest of the distribution.

This step notably **stabilized the standard deviation**, making downstream statistical testing more reliable. Visualization before and after filtering confirms that the tail shrinkage affects only an extreme minority.

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1tW-WBBus3I7S8aYV9G8K3rfiFVAD-rKI&sz=s4000" width="800"><br>   <sub>Figure 2. Comparison of game round distributions before and after filtering</sub> </div>

> **Insight:** The cleaned dataset better reflects typical user behavior and avoids bias from a single anomalous player.

## <big><strong>3. Exploratory Analysis</strong></big>

### <strong>3.1 Game Round Distribution</strong>

We first explored whether shifting the gate position had any noticeable effect on user engagement, as measured by the total number of game rounds played in the first 14 days.

Using histograms and boxplots for both groups, we observed that:

- The **median game rounds** were very close: ~17 for `gate_30` and ~16 for `gate_40`
- The **distribution shapes** were similar and remained **right-skewed**, with long tails in both groups
- A visual inspection suggested **no major change in engagement behavior** due to the gate shift

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1FRiIGFgsduCfsOlGP3dytLUPx_iFRf0o&sz=s4000" width="1000"><br>   <sub>Figure 3. Game round distributions for gate_30 and gate_40 groups (after outlier removal)</sub> </div>

> **Insight:** Delaying the first gate did not noticeably change how much users engaged with the game overall. Both groups showed similar playtime patterns.

------

### <strong>3.2 Retention Overview</strong>

Next, we compared **Day-1** and **Day-7** retention rates between the two groups.

Bar charts and grouped statistics revealed:

- **Day-1 retention** was approximately **44.8%** for `gate_30` vs **44.2%** for `gate_40`
- **Day-7 retention** dropped more visibly: **19.0%** vs **18.2%**
- A combined metric (users retained on both Day 1 & 7) showed a subtle decrease in the test group

| Version | Users Count | Day-1 Retention Rate | Day-7 Retention Rate | Total Game Rounds |
| ------- | ----------- | -------------------- | -------------------- | ----------------- |
| gate_30 | 44,700      | 0.4482               | 0.1902               | 2,344,795         |
| gate_40 | 45,489      | 0.4423               | 0.1820               | 2,333,530         |

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1OlYg9vMbeuj2Bdw1-eF8I3_ed83X0BWu&sz=s4000" width="1000"><br>    <sub>Figure 4. Comparison of 1-day and 7-day retention rates between groups</sub> </div>

> **Insight:** Although the Day-1 retention difference is marginal, the **Day-7 gap is more pronounced**, hinting that the gate relocation may negatively affect longer-term engagement.

## <big><strong>4. Randomization Check (AA Test)</strong></big>

### <strong>4.1 Validating Group Balance</strong>

Before evaluating treatment effects, we simulate an **AA-style test** to verify that the two groups (`gate_30` and `gate_40`) were statistically comparable at baseline. While our dataset contains post-treatment metrics, we assume no immediate impact from the gate relocation in the initial moments of gameplay.

This approach serves as a **randomization validity check**, ensuring that:

- Any observed differences in later retention are attributable to the gate change
- The experiment meets the internal validity condition of **pre-treatment comparability**

We tested the following:

| Metric           | Test Used       | p-value | Conclusion                  |
| ---------------- | --------------- | ------- | --------------------------- |
| `sum_gamerounds` | Welch's t-test  | 0.949   | ✅ Groups are comparable     |
| `retention_1`    | Chi-square test | 0.075   | ✅ No significant difference |
| `retention_7`    | Chi-square test | 0.064   | ✅ No significant difference |

> **Note:** All tests returned non-significant results (p > 0.05), confirming the assumption of balanced random assignment. This validates the integrity of the experimental design under an AA-style framework.

------

### <strong>4.2 Visual Comparison of Distributions</strong>

To further confirm the comparability of group behavior, we plotted distribution curves and empirical cumulative distribution functions (ECDF) for `sum_gamerounds`.

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1P8rW5HQT6wQUGnH3I9cITqkK72JfnV6O&sz=s4000" width="800"><br>   <sub>Figure 5. Distribution and ECDF plots of total game rounds</sub> </div>

| Version | Users Count | Median Game Rounds | Mean Game Rounds | Std Dev | Max Game Rounds |
| ------- | ----------- | ------------------ | ---------------- | ------- | --------------- |
| gate_30 | 44,699      | 17.0               | 51.34            | 102.06  | 2,961           |
| gate_40 | 45,489      | 16.0               | 51.30            | 103.29  | 2,640           |

Both groups exhibit highly similar:

- Median engagement (~16–17 rounds)
- Right-skewed tail behavior
- Cumulative progression curves

> **Insight:** The similarity across all engagement metrics confirms that the experimental design achieved proper randomization, simulating the logic of an AA test despite post-treatment data. 

These pre-checks lay the statistical groundwork for attributing any downstream behavioral differences to the gate change itself.

## <big><strong>5. Hypothesis Testing</strong></big>

### <strong>5.1 Game Rounds Played (Numerical Metric)</strong>

To assess whether delaying the first gate changed user engagement, we tested for differences in `sum_gamerounds` between the two groups.

### <strong>Modular A/B Testing Function</strong>

To streamline the statistical testing process, we implemented a reusable function `AB_Test()` that dynamically chooses the appropriate test strategy based on the data’s characteristics:

```python
# Simplified logic of AB_Test()
if normality_passed:
    if variance_homogeneous:
        # Use standard t-test
    else:
        # Use Welch's t-test with unequal variances
else:
    # Use Mann-Whitney U test (non-parametric)
```

The function returns effect size, test type, and power estimates, and can also plot group distributions for quick visual diagnostics.

This design ensures that statistical methods are **data-driven**, not hardcoded, and aligns with best practices in experimental inference.

#### Why not use t-test?

Initial distribution analysis showed that `sum_gamerounds` is **non-normally distributed and highly right-skewed**, violating the assumptions of a parametric t-test.

We therefore applied the **non-parametric Mann-Whitney U test**, which compares medians without assuming normality.

#### Test Result

| Test                | p-value | Result                                     |
| ------------------- | ------- | ------------------------------------------ |
| Mann-Whitney U Test | 0.0509  | ❌ Not statistically significant (α = 0.05) |

> Although the p-value is close to the threshold, it does not pass the standard 0.05 significance level.

#### <strong>Interpretation</strong>

> **Insight:** Moving the gate from level 30 to level 40 **did not significantly impact how much users played overall**.
>  Both groups had similar distributions and central tendencies in game rounds.

------

### <strong>5.2 Retention Rates (Binary Metrics)</strong>

Next, we examined user retention at **Day 1** and **Day 7** using **two-proportion z-tests**.
 These tests evaluate whether the proportion of retained users differs significantly between groups.

#### Z-Test Results

| Metric        | Retention (gate_30) | Retention (gate_40) | p-value | Conclusion               |
| ------------- | ------------------- | ------------------- | ------- | ------------------------ |
| `retention_1` | 44.8%               | 44.2%               | 0.0739  | ❌ Not significant        |
| `retention_7` | 19.0%               | 18.2%               | 0.0016  | ✅ Significant difference |

> The Day-7 retention difference is statistically significant, with the `gate_40` group performing worse.

#### <strong>Interpretation</strong>

Although short-term retention (Day-1) remains stable, the gate change **negatively impacts long-term retention**.

This suggests that pushing the first gate further out may reduce user stickiness over time.

## <big><strong>6. Bootstrapping Analysis</strong></big>

### <strong>6.1 Why Bootstrapping?</strong>

While hypothesis testing provides p-values and binary decision-making, it often fails to reveal the **distributional uncertainty** or **effect magnitude variability**.
 To address this, we apply **bootstrapping**—a resampling technique that generates empirical distributions of the metric differences.

> Bootstrapping allows us to estimate confidence intervals (CI) around treatment effects, helping answer questions like:
>
> - “What is the likely range of retention differences?”
> - “How confident are we that one variant is worse than the other?”

------

### <strong>6.2 Methodology</strong>

- We resampled the dataset **5000 times with replacement**
- For each sample, we calculated:
  - **Day-1 retention rate**
  - **Day-7 retention rate**
  - Their difference between `gate_40` and `gate_30`
- We then visualized the resulting **distribution of differences** and calculated **95% confidence intervals**

<div align="center">   
  <img src="https://drive.google.com/thumbnail?id=1_USMyKEybkIRq8Lrvesp4fT2GLpj-i-a&sz=s4000" width="800"><br> 
  <img src="https://drive.google.com/thumbnail?id=1kRFXtMtppDYT50FPcW-KvJoAmhNmdYJ9&sz=s4000" width="800"><br>  
  <sub>Figure 6. Bootstrapped differences in retention_1 and retention_7 with confidence intervals</sub> </div>

------

### <strong>6.3 Results</strong>

| Metric        | 95% Confidence Interval | Excludes 0? | Conclusion                    |
| ------------- | ----------------------- | ----------- | ----------------------------- |
| `retention_1` | [-0.0123, +0.0005]      | ❌ No        | Not statistically significant |
| `retention_7` | [-0.0134, -0.0032]      | ✅ Yes       | Significant negative effect   |

> These findings align with the z-test results, but also reveal the **magnitude and direction** of the difference.

In addition, we computed the **probability that gate_40 performs worse than gate_30**:

```python
P(retention_1_diff < 0) = 96.60%
P(retention_7_diff < 0) = 99.98%
```

> **Insight:** There is near certainty that the test group performs worse in 7-day retention. Even though the Day-1 effect is less clear, the long-term negative impact is statistically and practically meaningful.

## <big><strong>7. Insights & Recommendation</strong></big>

### <strong>7.1 Key Findings</strong>

Through a combination of traditional hypothesis testing and robust bootstrapping, the experiment yielded the following takeaways:

- **Delaying the first gate from level 30 to 40 did not significantly improve user engagement**, as measured by game rounds played.
- **Day-1 retention remained statistically unchanged**, indicating short-term user behavior was not affected.
- **Day-7 retention dropped by approximately 0.8 percentage points**, which translates to a **~5% relative decrease** in long-term retention. This result was confirmed by both traditional testing and bootstrapping.
  - z-test p = 0.0016
  - 95% CI = [-0.0134, -0.0032]
- **99.8% of bootstrap samples** showed lower 7-day retention in the test group.

> These findings collectively suggest that the gate relocation not only failed to create measurable benefits, but also introduced **a subtle yet statistically robust risk of long-term churn**.

------

### <strong>7.2 Business Impact</strong>

Although the goal of this experiment was to delay friction and potentially improve monetization by moving the first gate from level 30 to 40, the analysis showed that this change **harmed long-term retention** without improving short-term engagement.

By **identifying this negative retention trend before rollout**, the team was able to **prevent a potentially harmful product change**. While A/B testing is often framed around “improvements”, this case highlights its equally important role in **risk mitigation** and **design validation**.

The analysis helped the product team:

- Avoid revenue loss tied to lower user retention
- Reconsider assumptions around gate placement and its motivational impact
- Reinforce the need to balance monetization tactics with long-term user experience

> **Final Recommendation:** Do **not** roll out the gate_40 configuration.

By identifying this negative effect early, the team was able to **avoid releasing a change that could have reduced 7-day retention by ~5% at scale**.

This project demonstrates how data-driven decision-making—supported by rigorous testing and distributional analysis—can not only guide optimizations but also **prevent harmful changes from going live.**
