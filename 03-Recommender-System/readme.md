# <big><strong>Project Overview</strong></big>

This project tackles a real-world **news recommendation problem**:
 Given a user's historical reading behavior, our goal is to **predict the final news article that the user will click**, and build a recommendation system accordingly.

Instead of traditional tabular data with explicit features and labels, we are provided with **click logs** from a real application environment. Therefore, the first key step is to **reformulate the task into a supervised learning problem** so that we can apply modeling techniques.

------

## <big><strong>Problem Understanding</strong></big>

Unlike standard regression or classification problems, this project asks:

> "Which of the 360,000 news articles is the one the user will click at the end?"

This poses two immediate challenges:

- The **prediction target** is not a label or score, but a specific article ID.
- The **input format** is not structured features, but a click log recording all user-article interactions.

To make this task solvable with machine learning, we **transform it into a classification problem**:

> Given a (user, article) pair, predict the probability that the user will click the article.

This allows us to reframe the goal as **click-through rate (CTR) prediction**, a familiar setting for supervised learning models.

------

## <big><strong>How We Reformulated the Problem</strong></big>

Instead of directly classifying one out of 360,000 categories (a massive multi-class problem), we:

1. Generate candidate articles for each user (via recall algorithms).
2. Predict the click probability for each (user, article) pair.
3. Rank these candidates based on the predicted probability.

The article with the highest predicted probability becomes the top recommendation.
 Thus, the original task becomes a binary classification over (user, article) samples:

> **Label = 1** if the article is the final clicked one, **0 otherwise.**

This enables us to:

- Apply standard ML classifiers (e.g., logistic regression, GBDT).
- Engineer features across user profiles, article metadata, and interaction history.
- Evaluate predictions using position-sensitive hit scores.

------

### <big><strong>Evaluation Metric</strong></big>

We adopt a **position-decayed hit score** to evaluate the top-5 recommendations.

#### Step-by-step:

- For each user, we recall 50 articles from the entire corpus (~360k).
- Then, we select the **top 5 articles** ranked by predicted click probability.
- The **ground truth** is the actual last-clicked article for that user.

We define the score as:

$$\text{Score}(user) = \sum_{k=1}^5 \frac{s(user, k)}{k}$$

Where:

- $s(user, k) = 1$ if the $k$-th article matches the ground truth; otherwise $0$.
- $\frac{1}{k}$ acts as a **position decay** factor — earlier hits earn higher scores.

#### Example:

If the true article appears in:

- Position 1 → score = 1.0
- Position 2 → score = 0.5
- Not in top 5 → score = 0

This metric **rewards both hit accuracy and ranking quality**.


## <big><strong>Recommendation System Overview</strong></big>

### <strong>What is a Recommendation System Actually Doing?</strong>

At its core, a recommendation system predicts what a user may be **interested in**, based on their previous interactions. But “interest” is not directly observable — it must be **inferred from behavior**.

Common **signals of interest** include:

- **Explicit feedback**: likes, favorites, follows, shares, comments.
- **Implicit feedback**: dwell time, pauses, completion rate, exit behavior.
- **Action traces**: click logs, item views, purchases.

For example:

- In a video platform, even a **pause action** might indicate engagement — maybe the user is watching closely or planning to screenshot.
- In a news app, **clicks and time-on-article** are usually the strongest signals of interest.

<img src="https://towardsdatascience.com/wp-content/uploads/2022/11/1yPeDvQjUoFdLKb9CkxaFPA-768x430.png" width="800">

------

### <strong>Why Is Click the Core Signal?</strong>

Because it’s:

- **Ubiquitous**: Every user leaves click traces.
- **Immediate**: Happens early in the engagement funnel.
- **Quantifiable**: Easy to collect, robust volume.

> Even if a user later likes or comments, **they must have clicked first**.

Thus, most systems — especially in cold-start or large-scale settings — rely on **click behavior as the primary learning signal**.

------

### <strong>Two-Stage Architecture: Recall → Rank</strong>

Modern large-scale recommender systems typically follow a **two-stage pipeline**:

| Stage       | Example Models                 | Purpose                             | Predicts Click Probability? | Output        |
| ----------- | ------------------------------ | ----------------------------------- | --------------------------- | ------------- |
| **Recall**  | ItemCF, UserCF, YouTubeDNN     | Select a few hundred relevant items | ❌ Not necessarily accurate  | Candidate set |
| **Ranking** | Logistic Regression, GBDT, DIN | Score and sort candidates           | ✅ Yes (CTR prediction)      | Ranked list   |

- **Recall stage** = cast a wide net, filtering the universe (e.g., from 360k to 50 articles).
- **Ranking stage** = precision targeting, predicting $P(\text{click})$ for each user-item pair.

<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/0*w2yvcYmMCMffCa_o" width="800">

------

## <big><strong>Recommendation System Principles</strong></big>

### <strong>Core Idea: Predict Interaction Probability</strong>

A recommendation system doesn’t “decide” what to recommend.
 It **estimates probabilities** and **lets ranking do the work**.

### <strong>Scoring Formula (Multi-Behavior Fusion)</strong>

When using multi-task or multi-signal models:

$$\text{Score} = P_{\text{click}} \cdot W_{\text{click}} + P_{\text{comment}} \cdot W_{\text{comment}} + \cdots$$

Where:

- $P$ are predicted probabilities for different behaviors.
- $W$ are weights reflecting the business value of each behavior.

> These weights are typically tuned **via A/B testing**, and may change over time.

#### Examples:

- In early-stage platforms → **Follow** may be valued more ($W_{follow}$ ↑).
- In later-stage platforms → **Click-through** or **completion** may matter more.

<img src="https://drive.google.com/thumbnail?id=1wg7Lk9SLnV0iYjxCu8Iu3qZZXBqbxHpj&sz=s4000" width="500">

------

### <strong>Practical Trade-offs by Platform Stage</strong>

- **Early-stage platforms**:
  - Use **clicks** as the primary target.
  - High volume makes them statistically significant even in small datasets.
- **Mid-stage platforms**:
  - Begin integrating **likes, follows, and shares** as **auxiliary objectives** or in **multi-task setups**.
  - These actions reflect stronger engagement, though they’re less frequent.
- **Mature platforms**:
  - Experiment with **richer signals** like dwell time, scroll depth, rewatch rate.
  - May implement custom event tracking (e.g. pause, zoom, repeat), through **event logging design**.

------

### <strong>Challenge: User Group Generalization</strong>

Not all users behave the same.

Some users **share preferences** with common groups, while others have **atypical interests** not represented well in the training set.	
 This leads to the **user group generalization problem** — the model cannot learn good representations for “cold” or “mixed-preference” users.

> 🔍 Example: Tech content isn't only watched by stereotypical users (e.g., developers). Some fans may look totally unrelated by profile.

This results in:

- Low recall for certain users.
- Overfitting to dominant behavior patterns.
- Underperforming on niche or cold-start segments.

------



# <big><strong>1. Data Overview</strong></big>

------

## <big><strong>1.1 Dataset Overview</strong></big>

We work with user interaction logs from a news application. The dataset includes approximately:

- **300K users**
- **2.9M clicks**
- **360K unique articles**

Each article has a corresponding **embedding vector** representation. The data is partitioned as follows:

- **Training set**: 200K users’ click history
- **Validation set A**: 50K users’ click history
- **Prediction set B**: 50K users for final submission

The main data files:

| File Name             | Description                         |
| --------------------- | ----------------------------------- |
| `train_click_log.csv` | Training set: user click logs       |
| `val_click_log.csv`   | Validation set A: user click logs   |
| `articles.csv`        | Article metadata                    |
| `articles_emb.csv`    | Article embeddings (249 dimensions) |



### Column Dictionary

| Column                | Description                                |
| --------------------- | ------------------------------------------ |
| `user_id`             | User ID                                    |
| `click_article_id`    | Clicked article ID                         |
| `click_timestamp`     | Click time (UNIX timestamp)                |
| `click_environment`   | Click environment                          |
| `click_deviceGroup`   | Device group                               |
| `click_os`            | Operating system                           |
| `click_country`       | Country                                    |
| `click_region`        | Region                                     |
| `click_referrer_type` | Type of referrer (e.g., direct, ad, etc.)  |
| `article_id`          | Article ID (aligned with click_article_id) |
| `category_id`         | Article category ID                        |
| `created_at_ts`       | Article creation timestamp                 |
| `words_count`         | Number of words in the article             |
| `emb_1` to `emb_249`  | Embedding vector components                |



------

## <big><strong>1.2 Click Log Preprocessing</strong></big>

We began by sorting the click logs by timestamp and assigning reverse rank numbers for each user’s click sequence (i.e., most recent = 1).

```python
# Assign rank: latest click = 1
trn_click['rank'] = trn_click.groupby('user_id')['click_timestamp'].rank(ascending=False).astype(int)
```

We also counted the number of articles each user clicked:

```python
# Count number of clicks per user
trn_click['click_cnts'] = trn_click.groupby('user_id')['click_timestamp'].transform('count')
```

- Every user in the training set clicked **at least 2 articles**.
- There is **no user overlap** between the training and validation sets.

------

## <big><strong>1.3 Preliminary Exploration</strong></big>

### Feature Distributions

We plotted the distribution of ten key columns to get an intuitive sense of the data:

- **Click environment, device group, OS, country, region, referrer type**
- **Article popularity (`click_article_id`), user activity (`click_cnts`), and click rank (`rank`)**

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1aQGKei3DK2ewNdQPWVt2Jw2icf7qpmcq&sz=s4000" width="800"><br />   <sub>Feature Distributions</sub> </div>

Key takeaway from `click_deviceGroup`:

```python
trn_click['click_deviceGroup'].value_counts(normalize=True)
```

| Device Group | Proportion |
| ------------ | ---------- |
| 1            | 60.95%     |
| 3            | 35.55%     |
| 4, 5, 2      | <4% total  |

Device type `1` dominates, suggesting mobile traffic is the majority.

------

## <big><strong>1.4 Article Metadata</strong></big>

We explored the article-level metadata to understand content trends.

### Word Count Distribution

```python
sns.histplot(item_df['words_count'], bins=60, kde=True)
```

<div align="center">   <img src="https://drive.google.com/thumbnail?id=18PzeNAMvlYP32Yb5cvQJzzoi7ac2wYSX&sz=s4000" width="800"><br />   </div>

- The word count distribution is **strongly right-skewed**, with a sharp peak around 200 words.
- While most articles are short-form (under 300 words), a small number of outliers exceed **6,000 words**, indicating the presence of long-read or in-depth content on the platform.
- Such a distribution suggests that users primarily consume **quick-read news**, though longer-form content does exist and may serve niche or in-depth interest areas.

```python
# Violin / Box / KDE
plot_distribution(item_df, 'words_count')
```

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1lhkGjaf-92VMnTN3FDaMrBGSkeIj4nrp&sz=s4000" width="800"><br />   <sub></sub> </div>

The **boxplot and violin plot** confirm the extreme skewness of the distribution, with most articles tightly clustered around the median.

Outliers on the far right suggest content heterogeneity, which can be leveraged for personalization (e.g., recommending long-form content to heavy readers).

### Article Length Over Time

Average article length over years:

```python
item_df['created_at'] = pd.to_datetime(item_df['created_at_ts'], unit='ms')
item_df['year'] = item_df['created_at'].dt.year
sns.lineplot(x='year', y='words_count', data=item_df, estimator='mean', ci=95)
```

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1Q6uODPo5hYVb2pc-odalGoud5bqN-SSR&sz=s4000" width="800"><br />   <sub></sub> </div>

- The average word count **fluctuated significantly between 2005–2010**, suggesting changes in content strategies or editorial guidelines.
- After 2011, article length gradually stabilized and declined, possibly indicating a **shift toward mobile-friendly or bite-sized news** formats.
- These changes could inform **time-aware models** or longitudinal content recommendation strategies.

## <big><strong>1.5 Category Distribution</strong></big>

We also analyzed the frequency of different `category_id` values to identify dominant content themes:

```python
plot_distribution(item_df, 'category_id')
```

<div align="center">     <img src="https://drive.google.com/thumbnail?id=1OeXs7yqP_Cnx9f9RMfGqDOlkkofwCHEJ&sz=s4000" width="800"><br />     <sub></sub>   </div>

- The distribution is **highly imbalanced**, with a few category IDs dominating the dataset.
- This may indicate that the platform leans heavily toward certain content types (e.g., sports, politics), which can bias recommendations unless explicitly corrected.
- In future iterations, we could explore **category diversity** in user history as a feature to balance novelty and familiarity.

------

# <big><strong>2. Data Analysis</strong></big>

## <big><strong>2.1 User Behavior Exploration</strong></big>

### Repeated Clicks

We first analyzed whether users revisit the same article multiple times. After merging the training and validation datasets:

```python
user_click_merge = pd.concat([trn_click, val_click])
user_click_count = user_click_merge.groupby(['user_id', 'click_article_id'])['click_timestamp'].agg(['count']).reset_index()
user_click_count['count'].value_counts(1).map(lambda x: f'{x:.2%}')
```

```css
>>> 
count
1     99.25%
2      0.72%
3      0.03%
4      0.00%
5      0.00%
6      0.00%
10     0.00%
7      0.00%
13     0.00%
Name: proportion, dtype: object
```

- Over **99% of user-article interactions** are unique, indicating that **most users do not revisit the same article**.
- Repeated clicks (≥2) occur in only ~0.7% of cases.
- This can be used to construct a binary feature: *whether the user has re-clicked this article*.

------

### User Activity Level

We calculated the **total number of articles clicked per user** to analyze user engagement:

```python
user_click_item_count = user_click_merge.groupby('user_id')['click_article_id'].count()\
                                        .to_frame('click_count').sort_values(by='click_count', ascending=False).reset_index()
```

Based on click count quantiles, we classified users into three activity levels:

- **Low Activity**: ≤ 2 clicks
- **Medium Activity**: 3–9 clicks
- **High Activity**: ≥ 10 clicks

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1iq_97RNmjkezhJgHmYbrjuzhj5jTF4JS&sz=s4000" width="700"> </div>

- Users with **≤2 clicks** account for a **large proportion** and are considered **inactive users**.
- A **small minority of users** contributed disproportionately large numbers of clicks. For example, the top 50 users all have **more than 100 clicks**.
- These patterns suggest the platform has a **long-tail user engagement** distribution.

## <big><strong>2.2 Article Click Distribution</strong></big>

We analyzed how frequently different articles were clicked by users. The distribution showed a classic **long-tail pattern**, where:

- A **small number of articles** received extremely high click counts (e.g., 15,000+).
- A **majority of articles** were clicked only **once or twice**, indicating cold content.

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1ggWjDXwjlFMZpL0pSMEyQ7W54yUZBIR4&sz=s4000" width="800"> <sub>Distribution of total article click counts (long-tail)</sub> </div>

To illustrate the **tail concentration**, we also plotted the top 100 most-clicked articles:

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1EG3QCIJ6lCdKbJi-1wDgsP7UnJc8zdTR&sz=s4000" width="800"> <sub>Click count distribution among top 100 articles</sub> </div>

- Over 20 articles had more than **2,500** clicks.
- The top 10 articles received between **11,000 and 15,000** clicks.

Such articles can be labeled as **hot articles**, and may influence model features around popularity and exposure.

To summarize:

- **Hot articles**: Top 100 articles with >2500 clicks.
- **Cold articles**: Articles with ≤2 clicks.
- Article popularity is **extremely skewed**, which is a key characteristic in recommendation data.



## <big><strong>2.3 Article Co-Occurrence Analysis</strong></big>

### <big>**What is “Co-Occurrence”?** </big>

Co-occurrence frequency refers to the **number of times two articles are read consecutively** by the same user.		
 For example, if user A reads article 101 and then article 205, the pair (101, 205) gets a +1 count.

High co-occurrence may indicate **semantic or temporal relatedness**, such as articles covering the same news event or follow-up stories.

------

### Application Scenarios

- **Recommendation System**: Build a graph of co-clicked articles to support jump-to recommendations (e.g., “Users who read A often continue with B”)
- **User Behavior Analysis**: Understand user browsing habits and patterns
- **Topic Clustering**: Frequently co-clicked articles may indicate topic proximity and help enhance unsupervised content grouping

------

### Observations from Co-Occurrence Statistics

We sorted all article pairs by co-occurrence frequency and found:

```python
# After sorting by timestamp, generate the next clicked article for each user.
# This creates (current article, next article) pairs for downstream analysis.
tmp = user_click_merge.sort_values('click_timestamp')
tmp['next_item'] = tmp.groupby('user_id')[['click_article_id']].shift(-1)

# Count how often each article pair co-occurs in sequence.
union_item = tmp.groupby(['click_article_id','next_item'])['click_timestamp'].agg({'count'}).reset_index().sort_values('count', ascending=False)

# Get a quick statistical overview of the co-occurrence counts for all article pairs.
union_item[['count']].describe().map(lambda x: f'{x:.1f}')
```

| Statistic    | Value |
| ------------ | ----- |
| Mean         | 3.2   |
| 50% Quantile | 1.0   |
| 75% Quantile | 2.0   |
| Max          | 2202  |

- Over **75%** of article pairs occurred **≤ 2 times**, suggesting most sequences are **incidental**.
- A few article pairs showed **very high co-occurrence (up to 2202)**, indicating **highly frequent reading paths**.

To remove noise, a **frequency threshold** (e.g., ≥10) can be set to filter meaningful transitions.

------

### Co-Click Graph Visualization

We visualized the **top 10 most frequent article pairs** using a co-occurrence graph:

```python
import networkx as nx

# Get the top 20 most frequently co-occurring article pairs
top_20_pairs = union_item.nlargest(20, 'count')

# Use NetworkX to build a graph where:
# - Each node represents an article
# - Each edge represents a sequential co-occurrence between articles
# - Edge weight indicates the frequency of that co-occurrence
G = nx.from_pandas_edgelist(top_20_pairs, 'click_article_id', 'next_item', ['count'])

# Set up the plot
plt.figure(figsize=(8, 6))
pos = nx.spring_layout(G, seed=42)  # Use spring layout for better spacing

# Draw the network graph
nx.draw(
    G, pos, with_labels=True, node_size=2000, node_color='skyblue',
    font_size=10, font_color='black', edge_color='gray', alpha=0.7
)

# Add edge labels to display co-occurrence counts
labels = nx.get_edge_attributes(G, 'count')
nx.draw_networkx_edge_labels(G, pos, edge_labels=labels)

```

- **Node** = article ID
- **Edge** = co-occurrence between two articles
- **Edge label** = frequency count

<div align="center"> <img src="https://drive.google.com/thumbnail?id=1luxr5R5-5qbmmnG-zftIoPk_pwfNTQgH&sz=s4000" width="800"> <sub></sub> </div>

This graph highlights high-frequency transitions between articles, which could be leveraged in:

- Graph-based recommendation modules
- Session-based models
- Content exploration tools

------

### Further Exploration

We extracted the **article metadata** for these co-clicked pairs to examine:

- Do the articles belong to the **same category**?
- Are they **long-form or short-form**?
- Do their **titles** reflect the same event or theme?

These insights can validate whether co-occurrence reflects **semantic relatedness** and support further content modeling efforts.



## <big>2.4 Diversity of User Interests</big>

To assess how broad each user's reading scope is, we counted the number of **distinct article categories** they’ve engaged with. This serves as a proxy for **content diversity preference**.

```css
>>>
cate_count
2–3 categories      110395
4–5 categories       56904
6–10 categories      55850
11–20 categories     22894
More than 20          3957
1 category               0
Name: count, dtype: int64
```



<div align="center">   <img src="https://drive.google.com/thumbnail?id=1vkCmipdZhMZrLDWVE4ZmVsBRrEoDKMpj&sz=s4000" width="600"><br></div>

Key observations:

- Nearly **45%** of users focus on just 2–3 categories, indicating relatively **narrow preferences**;
- Only a small fraction (~4%) explore **20+ categories**, reflecting **broad and exploratory behavior**;
- The absence of users with exactly one category suggests at least minimal cross-category browsing for all users.

This indicator is valuable for user profiling. For example:

- In **CF modeling**, users with similar interest breadth may have similar exposure bias;
- In **content recommendation**, users with wide interests may benefit more from diverse feed strategies, while focused users may prefer tighter topic clusters.

------

## <big>2.5 Temporal Patterns of User Engagement</big>

We analyzed users’ **average time gap between consecutive clicks** to uncover latent behavioral rhythms.

<div align="center">   <img src="https://drive.google.com/thumbnail?id=1asMydYFzAeg5vsD1FlD_7SxOUTVmP4a7&sz=s4000" width="800"><br> </div>

Findings:

- The distribution is long-tailed, ranging from **intensive sessions** (clicks within seconds or minutes) to **sporadic behaviors** (intervals of several days);
- Most users fall below the **1-day range**, with a clustering in the 10–60 minute band;
- Even the **maximum observed gap** stays under **10,000 minutes (~7 days)**, suggesting that even low-frequency users return within a week — a useful assumption for session segmentation;
- This temporal trait implies **session-based behaviors**, supporting the idea of grouping clicks into short activity bursts.

This time-gap metric can be incorporated into:

- **Retention models** (as a proxy for habitual engagement);
- **Sequential models** (to weigh time-aware transitions between articles);
- **User clustering**, distinguishing casual scrollers from dedicated readers.



# <big><strong>3. Multi-Channel Recall</strong></big>

## <big><strong>3.1 Overview and Design Logic</strong></big>

### <big>**What is multi-channel recall?**</big>

 Multi-channel recall refers to the **integration of multiple recall strategies**, each retrieving candidate items from a different angle. These channels are **independent and complementary**, and their outputs are merged to form a richer and more comprehensive candidate set for downstream ranking models.

Typical recall channels include:

- **User-based collaborative filtering (UserCF)**
   Recommends items that similar users have interacted with.
- **Item-based collaborative filtering (ItemCF)**
   Recommends items similar to those the user has interacted with.
- **Embedding-based similarity recall**
   Recommends articles with high semantic similarity (e.g., via Word2Vec or other vector encodings).
- **Popularity-based recall**
   Supplements the recall set with hot articles to increase hit rate.
- **Cold-start heuristics**
   Introduces fresh or unexplored content for new users or items.

<div align="center">   <img src="https://raw.githubusercontent.com/xei/recommender-system-tutorial/main/assets/retrieval_ranking.png" width="800"><br>   <sub>Multi-channel recall design: each channel retrieves independently and is merged for ranking</sub> </div>

------

### <big><strong>Why do we need multiple recall channels?</strong></big>

Because **each method captures a different perspective**:

- **UserCF** captures *user similarity*;
- **ItemCF** captures *item-item associations*;
- **Embedding-based** recall captures *semantic proximity*;
- **Hot article / cold-start recall** improves *coverage and freshness*.

> No single method can achieve optimal coverage and diversity — combining them helps mitigate individual weaknesses.

For example:

- Relying solely on UserCF may fail for new users (cold-start);
- Embedding-based recall may struggle with sparse content or noisy representations;
- Hot recall ensures recent or trending content is not missed.

By merging all sources, we **increase the likelihood of including relevant items**, thereby improving **overall recall rate** while maintaining **computational efficiency**. Each strategy can be executed **in parallel**, allowing for scalable implementation in practice.



## <big><strong>3.2 UserCF-based Recall</strong></big>

### Objective

To recommend articles that were clicked by users who are similar to the current user.
 User similarity is calculated based on overlapping reading behavior and time-based weights.

------

### Approach

UserCF (User-based Collaborative Filtering) assumes that users with similar behaviors tend to like similar items. The basic steps include:

1. **Build user-item inverted table** to track which users clicked which articles.

2. **Compute similarity between users**:

   - Penalize popular articles using
   
     $$\frac{1}{\log(1 + \text{user}_\text{count})}$$.

   - Apply time-decay weight to clicks:
     
     $$\text{sim}_\text{weight} = \exp\left(0.8^{|\text{click}_i - \text{click}_j|}\right)$$
     

3. **Aggregate neighbors' preferences** to recommend items that the similar users clicked but the current user has not.

4. **Score each candidate item** with a combination of similarity and weight.

<img src="https://drive.google.com/thumbnail?id=1Z6k-fB4WQgh8HWaxHiBTgatsvdCf0gyA&sz=s4000" width="500">

------

### Core Code Snippet

```python
# Time-decayed user similarity
sim_user_time = np.exp(0.8 ** np.abs(click_time_user_i - click_time_user_j))

# Build user-user similarity matrix
user_sim_matrix = defaultdict(lambda: defaultdict(float))

for article, users in item_user_clicks.items():
    for u in users:
        for v in users:
            if u == v:
                continue
            user_sim_matrix[u][v] += 1 / math.log(1 + len(users)) * sim_user_time
```

------

### Interpretation

- Articles read by other users with similar click histories are selected as candidates.
- Time decay gives higher weight to more recent interactions.
- Popular items are down-weighted to avoid bias from frequently clicked articles.

This strategy helps surface articles that “people like you also read,” especially when user-item interactions are rich.

## <big><strong>3.3 ItemCF-based Recall</strong></big>

### Objective

To recommend articles similar to those that a user has previously clicked.		
 Item similarity is derived from co-click frequency, adjusted by time decay and position bias.

------

### Approach

ItemCF (Item-based Collaborative Filtering) leverages the idea that items often clicked together are likely similar. The steps:

1. **Construct item co-occurrence matrix** based on users’ click sequences.

2. **Apply time-decay and position weightings**:

   - Time-based weight:

     $$w_{\text{time}} = \exp\left(0.8^{|\text{click}_i - \text{click}_j|}\right)$$

   - Position-based weight:

     $$w_{\text{loc}} = \alpha \cdot 0.9^{|\text{loc}_i - \text{loc}_j| - 1}$$

     where $\alpha = 1.0$ if $j$ follows $i$ (forward), and $\alpha = 0.7$ if $j$ precedes $i$ (backward).

3. **Downweight high-frequency items** to reduce popularity bias.

4. **Aggregate scores** of similar items and return the top-N.

------

### Core Code Snippet

```python
# Compute item-item similarity
item_sim_matrix = defaultdict(lambda: defaultdict(float))

for user, item_seq in user_click_dict.items():
    for i, item_i in enumerate(item_seq):
        for j, item_j in enumerate(item_seq):
            if item_i == item_j:
                continue
            time_weight = np.exp(0.8 ** abs(time_i - time_j))
            loc_weight = (1.0 if j > i else 0.7) * 0.9 ** (abs(i - j) - 1)
            item_sim_matrix[item_i][item_j] += time_weight * loc_weight / math.log(1 + len(item_seq))
```

------

### Interpretation

- This method focuses on **item-to-item transitions**, suitable for modeling article similarity.
- Incorporating **time gap** and **reading order** captures user reading patterns better than plain co-occurrence.
- The resulting item similarity matrix supports recommending "related articles" even for new users with sparse history.

This module is especially useful when article metadata (like embedding or topic tags) is unavailable or unreliable.



## <big><strong>3.4 Content-based Recall</strong></big>

When a new article is published, we often lack user interaction history. To solve this **cold-start problem** and provide semantic-level matching, we use **article embeddings** to measure content similarity directly.

By comparing embeddings of recently clicked articles with all articles, we can recommend semantically related ones, even if they have never been clicked.

------

### Cosine Similarity for Article Matching

We use **cosine similarity** to measure how similar two articles are, based on their embedding vectors:

```python
def cos_sim(x, y):
    return np.dot(x, y) / (np.linalg.norm(x) * np.linalg.norm(y))
```

To ensure efficient access and avoid repeated computation:

```python
# Construct a dictionary: {article_id: embedding vector}
article_emb_dict = dict(zip(articles_emb_df['article_id'], articles_emb_df['emb'].apply(lambda x: np.array(x.split(' ')).astype(float))))
```

Then for each article, we compute top-k similar articles:

```python
def emb_sim_topk(article_id, top_k=10):
    sim_dict = {}
    a_vec = article_emb_dict[article_id]
    for i in article_emb_dict.keys():
        if i == article_id:
            continue
        sim = cos_sim(a_vec, article_emb_dict[i])
        sim_dict[i] = sim
    # Return top-k articles with highest similarity
    return sorted(sim_dict.items(), key=lambda x: x[1], reverse=True)[:top_k]
```

------

### Embedding-based User Recall

To recall items for a given user:

1. Select the **latest N clicked articles** of that user.
2. For each article, retrieve top-K similar articles.
3. Aggregate all retrieved articles as the candidate recall set.

```python
def emb_recall(user_item_dict, item_emb_dict, sim_item_topk=10, recall_item_num=100):
    user_recall_dict = dict()
    for user_id, item_list in tqdm(user_item_dict.items()):
        rank = {}
        for loc, i in enumerate(item_list):
            if i not in item_emb_dict:
                continue
            sim_item_list = emb_sim_topk(i, sim_item_topk)
            for j, sim in sim_item_list:
                if j not in rank:
                    rank[j] = sim * (0.9 ** loc)  # decay weight by click order
                else:
                    rank[j] += sim * (0.9 ** loc)
        user_recall_dict[user_id] = sorted(rank.items(), key=lambda x: x[1], reverse=True)[:recall_item_num]
    return user_recall_dict
```

------

### FAISS for Efficient Similarity Search

To handle large-scale article pools (e.g. 300k+ articles), we can replace brute-force cosine similarity with **FAISS**:

- Developed by Facebook AI Research
- Supports approximate nearest neighbor search
- Enables real-time retrieval in embedding spaces

```python
import faiss

# Normalize embeddings
emb_matrix = np.array(list(article_emb_dict.values())).astype('float32')
faiss.normalize_L2(emb_matrix)

# Build index
index = faiss.IndexFlatIP(emb_matrix.shape[1])  # inner product ≈ cosine similarity if normalized
index.add(emb_matrix)

# Query: top 10 most similar articles
D, I = index.search(np.expand_dims(emb_matrix[0], axis=0), k=10)
```

We map back index `I` to `article_id` for interpretation.

## <big><strong>3.5 Popularity-based Recall</strong></big>

This is also a cold start problem. When there is no interaction history for a user (new user) or an article (new item), conventional collaborative filtering methods like UserCF or ItemCF fail to produce meaningful recommendations. In these cases, we need a fallback mechanism — one of the most effective and interpretable strategies is **recalling popular articles**.

This popularity-based recall approach retrieves the most frequently clicked articles in the training data. The underlying assumption is simple yet robust: **if an article is widely read, it is more likely to interest a new user too.**

The use cases could be:

- **New users** with no click history (cold-start users)
- **Sparse users** with only 1 or 2 clicks (insufficient signal for collaborative methods)
- **Fallback component** in multi-channel recall, ensuring basic recommendation coverage for all users

### Core Code Snippet

```python
# Calculate click counts for each article
item_popularity = trn_click.groupby('click_article_id')['click_timestamp'].count() \
                            .reset_index().sort_values(by='click_timestamp', ascending=False)

# Rename columns
item_popularity.columns = ['article_id', 'click_count']

# Get top-N popular articles
top_popular_articles = item_popularity['article_id'].tolist()[:topk]  # e.g., topk = 100
```

We can choose to recall a fixed number of top-N articles for every cold-start user or filter based on certain metadata (e.g., same region, same category) to increase precision.

### Notes and Extensions

- This method does **not personalize** recommendations, but it serves as a **low-risk, high-coverage default**.
- We can further **customize** the popularity metric:
  - Recent click counts (e.g., in the past 7 days)
  - Weighted clicks (e.g., by dwell time or engagement)
- In production systems, it's often **combined** with other cold-start strategies:
  - **Content-based fallback** using embeddings or metadata similarity
  - **Category-based recall** for new users with partial interest signals (e.g., onboarding tags)



# <big><strong>4. Ranking Strategy</strong></big>

## <strong>4.1 Rule-based Ranking</strong>

After generating candidate articles from multiple recall channels — including UserCF, ItemCF, embedding similarity, and cold-start strategies — we need to **combine these results into a unified candidate pool** for downstream ranking.

In this project, we used a **rule-based score aggregation strategy** to simulate ranking by combining outputs from multiple recall channels. Each channel (e.g., UserCF, ItemCF, Embedding similarity) returned a ranked list of candidate articles with a recall score.

If the same article appears in multiple recall lists, this likely signals higher relevance — **aggregating scores leverages this signal**.

### Merging Strategy

For each user, we obtain multiple candidate lists:

- `recall_usercf_dict`: user-based collaborative filtering results
- `recall_itemcf_dict`: item-based collaborative filtering results
- `recall_emb_dict`: content embedding similarity results
- `recall_hot_dict`: popularity-based fallback results

We use a loop to **merge these dictionaries** and consolidate article IDs.

```python
from collections import defaultdict

# Initialize final recall pool
final_recall_dict = defaultdict(dict)

# Combine all strategies
recall_dicts = [recall_usercf_dict, recall_itemcf_dict, recall_emb_dict, recall_hot_dict]

for recall_channel in recall_dicts:
    for user_id, item_dict in recall_channel.items():
        for item_id, score in item_dict.items():
            if item_id not in final_recall_dict[user_id]:
                final_recall_dict[user_id][item_id] = score
            else:
                final_recall_dict[user_id][item_id] += score  # Optional: additive score
```

------

### Output Structure

Each user in `final_recall_dict` is associated with a dictionary:

```python
final_recall_dict[user_id] = {
    item_id_1: aggregated_score_1,
    item_id_2: aggregated_score_2,
    ...
}
```

This candidate pool is then passed to the **ranking module**, which will reorder items based on more complex user-item interaction signals and ranking models.

------

## <strong>4.2 Future Extensions</strong>

To improve the recommendation accuracy, we can replace rule-based ranking with a **learning-based ranking model**, using training data and supervised labels (e.g., clicks).

Common candidates include:

- **Logistic Regression** (as a simple and interpretable baseline)
- **LightGBM / XGBoost** (tree-based ranking models with strong performance)
- **Deep Interest Network (DIN)**: A neural model that captures user interest evolution via attention on historical clicks

These models can incorporate rich features such as:

- User and article embeddings
- Contextual features (e.g., device, time)
- Interaction history
- Article-level metadata

Training such models requires labeled data and negative sampling strategies. This is a natural next step beyond multi-channel recall.
