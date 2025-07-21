## <big><strong>Project Overview</strong></big>

This project conducts a comprehensive data analysis on the Yelp dataset, with a focus on leveraging **NLP techniques, geospatial mapping, and social network analysis** to uncover insights on business performance, user behavior, and community structure.

Our objective is to go beyond surface-level EDA and explore how user-generated content (UGC)—particularly **review text and reactions (useful, funny, cool)**—can inform **rating optimization, platform governance, and community strategy**.

<div align="center"><img src="https://diablogym.net/wp-content/uploads/2018/01/yelp-logo-22.png" width="500"><br><sub>Yelp as a multi-sided platform connecting users, businesses, and the review ecosystem</sub></div>

------

### <big><strong>Stakeholder Perspectives</strong></big>

This project is grounded in the needs of three key stakeholders on the Yelp platform:

- **Platform operators**
   Analyze review credibility, elite user influence, and geographic trends to inform rating governance, community growth, and potential monetization strategies.
- **Users (reviewers and readers)**
   Understand how user sentiment, review style, and reaction patterns vary across elite and regular users to improve recommendation, trust signals, and user experience.
- **Businesses**
   Identify how reviews reflect product/service issues, when and where customer engagement peaks, and how review feedback aligns with operational decisions (e.g., opening hours, location planning).

------

### <big><strong>Project Goal and Scope</strong></big>

This analysis aims to answer five core questions:

- What drives differences in business ratings?
- How do reaction metrics (useful/funny/cool) relate to sentiment and review quality?
- Can we extract reliable signals from free-text reviews to support fraud detection or risk segmentation?
- What are the spatial and temporal patterns in business activity and user behavior?
- How does the user friendship network inform influence, community, or engagement clusters?

We use Python-based analysis on multiple components of the Yelp dataset (business, user, review, tip, check-in) and apply tools including:

- **TextBlob** for sentiment analysis
- **Folium / Cartopy** for map-based visualizations
- **NetworkX** for graph construction and community detection

The goal is not to train a full recommendation model, but to surface interpretable insights that could inform **product strategy, risk management, and community engagement design**.



### <big><strong>Dataset Description</strong></big>

This project leverages multiple structured datasets derived from Yelp’s public data release. The data reflects real user-business interactions across multiple U.S. and Canadian cities, and is ideal for analyzing consumer behavior, business reputation, and social dynamics on a local platform.

We focus on the following five core datasets:

| File                | Description                                                  |
| ------------------- | ------------------------------------------------------------ |
| `yelp_business.csv` | Metadata of businesses including name, category, location, star rating, and review count |
| `yelp_user.csv`     | User-level attributes such as review count, average stars, elite status, compliment counts, and social links |
| `yelp_review.csv`   | Full-text reviews with associated star ratings and reaction counts (useful, funny, cool) |
| `yelp_tip.csv`      | Short user-generated tips or suggestions for businesses      |
| `yelp_checkin.csv`  | Timestamped business check-in logs indicating temporal activity patterns |

<div align="center"><img src="https://drive.google.com/thumbnail?id=1EE9ms0pOAym20IBah3FSc1sPMQ2JIGgt&sz=s4000" width="800"><br><sub>Entity relationships and schema overview of Yelp datasets</sub></div>

These datasets are joined primarily through `business_id` and `user_id`, enabling both micro-level analysis (e.g., review sentiment trends) and macro-level patterns (e.g., elite user behavior, category distributions, city-level saturation).

A subset of ~170,000 businesses and ~1 million reviews was sampled and cleaned to support faster iteration in exploratory and NLP tasks.



## <big><strong>1. Exploratory Data Analysis</strong></big>

### <big><strong>1.1 Business Rating Distribution</strong></big>

To understand the overall landscape of business ratings on Yelp, we first visualize the distribution of star ratings (1.0 to 5.0) across all businesses. This helps us detect bias, platform inflation, and typical sentiment skew.

Using the `stars` field in `yelp_business.csv`, we calculate frequency and proportion of each rating level, and render a bar chart with annotated counts.

<div align="center"><img src="https://drive.google.com/thumbnail?id=16cHRzuvIc9ISnveizHdv7ylou8n4r29W&sz=s4000" width="800"><br><sub>Distribution of Yelp business ratings (1.0–5.0)</sub></div>

**Insights**

- The distribution is **center-skewed**, peaking between 3.5 and 4.5 stars.
- A **notable proportion (~15%)** of businesses received perfect 5.0 ratings, which may reflect early-stage reviews or sampling bias.
- Low-rated businesses (1.0 to 2.0) are underrepresented, possibly due to Yelp's moderation policy or survivorship bias (closed businesses).

**Implication**

This non-uniform distribution suggests that downstream models (e.g., for rating prediction or fraud detection) must account for this skew, rather than assuming normality or uniform class balance.

### <big><strong>1.2 Top Business Categories</strong></big>

To identify the dominant business types on Yelp and understand the service landscape users interact with, we analyze the frequency of business categories.

Each business on Yelp can belong to multiple categories (e.g., "Restaurants; Bars; Nightlife"). We split the `categories` field and compute the most common tags across all businesses.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1LJQATHCrUnc3gZhN4VNMEJ-QslJMuDH9&sz=s4000" width="800"><br><sub>Top 20 most common business categories on Yelp</sub></div>

**Insights**

- **Restaurants**, **Shopping**, and **Food** top the list, jointly accounting for over **60%** of all business tags.
- Service industries like **Beauty & Spas**, **Health & Medical**, and **Home Services** form the second tier.
- Nightlife-related categories (Bars, Clubs) are present but not dominant in volume, indicating Yelp’s primary focus on daily services over leisure sectors.

**Implication**

This distribution shapes the nature of user reviews, with most NLP sentiment patterns likely stemming from food and shopping experiences. It also explains why review density and emotional intensity tend to be high in these verticals.

### <big><strong>1.3 Geographic Analysis</strong></big>

#### <big><strong>1.3.1 Global and Regional Distribution</strong></big>

To understand where Yelp activity is geographically concentrated, we map the locations of all businesses by latitude and longitude. This helps identify major markets and supports further regional or city-level segmentation.

Using `latitude` and `longitude` from `yelp_business.csv`, we plotted all business locations on a global projection using Cartopy. We then zoomed in on high-density zones in **North America** and **Europe** for more granular visualization.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1MCROYKAmi0JHXq33IlPa1kD1sOBduZSs&sz=s4000" width="600"><br><sub>Global distribution of Yelp businesses</sub></div> 

<div align="center"><img src="https://drive.google.com/thumbnail?id=1yV0rAfZRs2smG5mLHuLd6g4T9lJRctq9&sz=s4000" width="800"><br><sub>Zoomed-in view: North America</sub></div> 

<div align="center"><img src="https://drive.google.com/thumbnail?id=1BQZrfR1mjLQj-Fwose9vk0utEGEcxiRK&sz=s4000" width="800"><br><sub>Zoomed-in view: Europe</sub></div>

**Insights**

- **North America dominates** the business distribution, with dense clusters in cities like Las Vegas, Phoenix, and Toronto.
- **European cities**, though present, show lower density and coverage, likely reflecting limited market penetration.
- Business geographies appear **strongly urban-centric**, suggesting Yelp is most active in metropolitan consumer zones.

**Implication**

This geographic context helps prioritize downstream analyses (e.g., sentiment, check-in volume) by region. It also guides which cities to highlight in heatmaps, community networks, and elite behavior studies. 

#### <big><strong>1.3.2 City-Level Density Maps</strong></big>

To explore intra-city business clustering and urban layout patterns, we zoom into four cities—**Las Vegas**, **Phoenix**, **Stuttgart**, and **Edinburgh**—and visualize business density using scatter plots on geographic maps.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1JWyWxDyKDZInXjaLlUkUwA58SlMHbuws&sz=s4000" width="800"><br></div> 

We isolate each city using bounding boxes based on latitude/longitude, then plot all businesses with varying point density on dark background maps for visual clarity.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1ggAIBQbMeR8Sl7D-DJp2n6-kX-mEa-kj&sz=s4000" width="800"><br></div> 

<div align="center"><img src="https://drive.google.com/thumbnail?id=1z2FTyqTc8sDndmz9ht1g8J2OgK-d6vre&sz=s4000" width="800"><br><sub>Business density map</sub></div> 

**Insights**

- **Las Vegas** and **Phoenix** exhibit **grid-like clustering**, typical of U.S. cities with planned urban layouts.
- **Stuttgart** and **Edinburgh** show **less regular patterns**, consistent with older European city structures and walkability.
- Yelp business distribution correlates strongly with commercial districts and downtown cores.

**Implication**
 These maps help us understand Yelp’s footprint in different urban contexts and support further spatial analysis such as:

- Mapping elite user activity zones
- Connecting check-in patterns with service areas
- Identifying underserved regions for potential expansion

#### <big><strong>1.3.3 Heatmap by Star Rating (Folium Animation)</strong></big>

To assess whether high-rated and low-rated businesses cluster spatially, we use a **temporal heatmap** to visualize business locations in Las Vegas by star rating tiers.

We group Las Vegas businesses by their `stars` rating (1.0 to 5.0), extract their latitude and longitude, and create a time-lapse heatmap using `folium.plugins.HeatMapWithTime`.

<div align="center"><img src="https://github.com/ZzClaude/DS-Portfolio/blob/main/NLP-Yelp-Analysis/yelp_10.gif?raw=true" width="800"><br><sub>Animated heatmap of business ratings by location (Las Vegas)</sub></div>

**Insights**

- **High-rated and low-rated businesses are spatially mixed**, with no clear evidence of rating-based clustering.
- Central Las Vegas shows **dense mixed ratings**, possibly reflecting tourist traffic and business volatility.
- Outer areas are sparser but not necessarily lower-rated, suggesting **rating is not location-driven**.

**Implication**

This challenges common assumptions that “bad areas = bad ratings.” Instead, review scores seem to be shaped more by **service quality or customer expectation** than pure geography.

The animation format also provides an intuitive tool for **internal review**, e.g., city managers or category managers monitoring business health across time or quality segments.

### <big><strong>1.4 User Behavior Analysis</strong></big>

#### <big><strong>1.4.1 Review Volume Distribution</strong></big>

To evaluate community engagement and the structure of Yelp’s user base, we analyze the distribution of review counts per user. This reveals whether content contribution is evenly distributed or concentrated among a few power users.

Using the `yelp_review.csv` and `yelp_user.csv`, we group reviews by `user_id`, calculate review counts, and visualize the results with:

- A **KDE plot** showing review count distribution;
- A **cumulative distribution plot** to understand what percent of users write how many reviews.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1SkI38tGdKtqZ288BP9e01FqQPJBhOyfi&sz=s4000" width="800"><br><sub>Distribution of number of reviews per user (capped at 30)<br>Cumulative distribution of user review volume</sub> </div>

**Insights**

- The distribution is **extremely long-tailed**: over **80% of users post fewer than 3 reviews**.
- A small set of users are highly prolific, suggesting a potential for elite promotion or influencer leverage.
- The community is highly **contributor-imbalanced**, typical of UGC platforms.

**Implication**

This long-tail structure reinforces the value of elite programs and reviewer incentives. Activating the "silent middle" (users who posted once or twice) may yield stronger platform engagement than focusing solely on either end of the spectrum.

We explore this further by profiling top reviewers in the next section.

#### <big><strong>1.4.2 Top Reviewers & Behavior Profiling</strong></big>

To understand the traits of Yelp’s most active contributors, we analyze the top reviewers by volume and explore their review behavior across time and geography.

We extract the top users by `review_count` from `yelp_review.csv`, then merge with `yelp_user.csv` and `yelp_business.csv` to trace their check-in locations. Using `folium.plugins.HeatMapWithTime`, we animate each user’s review trail by date.

<div align="center"><img src="https://github.com/ZzClaude/DS-Portfolio/blob/main/NLP-Yelp-Analysis/yelp_12.gif?raw=true" width="800"><br><sub>Heatmap animation of top user's review locations across time</sub></div>

**Insights**

- The top reviewer (70+ reviews in our sample) is geographically concentrated in **Toronto suburbs**, suggesting localized lifestyle or neighborhood influence.
- This user’s reviews span over 8 years, indicating long-term platform loyalty.
- High-activity users generate significantly more reactions (useful, funny, cool), reinforcing their role as **signal amplifiers** within the community.

**Implication**

Understanding the location and time span of power users enables better **targeting for elite onboarding, local events, or reviewer campaigns**. These users often anchor local credibility and can influence nearby business reputation.



## <big><strong>2. Review Analysis</strong></big>

### <big><strong>2.1 Usefulness and Rating</strong></big>

To explore whether **“useful” votes** serve as a proxy for review quality or reliability, we analyze how rating, review length, and volume change as the threshold for `useful` votes increases.

We filter reviews with `useful > 0` and group them by thresholds from 100 to 1000. For each subset, we compute:

- Maximum star rating
- Average star rating
- Average review length
- Number of qualifying reviews

Each is plotted as a line chart using `sns.regplot`. These are plotted to show how increasing the useful vote threshold relates to review quality and sentiment.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1MvOulJ7gmhADCi8kXe2Xgqvj5mzDNJTF&sz=s4000" width="800"><br><sub>Trend of max/avg rating, review length, and review count vs. useful threshold</sub></div>

**Insights**

- Both **maximum and average star ratings decline** as the `useful` vote threshold increases.
   → Highly upvoted reviews tend to be **low-rating**, likely highlighting critical issues or negative experiences.
- **Review length increases linearly** with `useful` votes, suggesting detailed reviews are perceived as more helpful.
- **The number of such reviews declines sharply**, showing high-usefulness reviews are rare and not mass-produced.

**Implication**

Contrary to popular belief, it’s **negative, detailed, and rare** reviews that earn the most "useful" votes—not positive feedback.
 This has two implications:

- Platforms should **prioritize these reviews** in ranking/surfacing to protect consumers and surface risk.
- Businesses should **pay attention to long-form, low-rating reviews**, as they carry disproportionate influence.

------

### <big><strong>2.2 Sentiment Distribution: Elite vs. Regular Users</strong></big>

To assess whether Yelp’s elite users express opinions differently than regular users, we apply sentiment analysis to their review texts and compare the distribution of polarity scores.

Using `TextBlob`, we compute sentiment polarity for each review written, split by elite vs. regular users (based on the `elite` field in `yelp_user.csv`). The polarity score ranges from **−1 (very negative)** to **+1 (very positive)**.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1GRB1ZGp9eKXwe_DH4Z1KMgUgct1dZd2u&sz=s4000" width="800"><br><sub>Sentiment score distribution of reviews in 2012 by elite vs. regular users</sub></div>

**Insights**

- Both groups skew slightly positive, but **elite users have a tighter, more centered distribution**.
- Regular users show a wider spread of sentiment, including more extreme polarity scores on both ends.
- Elite reviews tend to be **more moderate and consistent**, possibly due to community norms or editorial standards.

**Implication**

Yelp’s elite users serve as **emotionally calibrated content anchors**, providing balanced perspectives that the platform can prioritize or weight more heavily in decision-making (e.g., for rating calculation or fraud detection).
 This also reinforces the importance of **elite recruiting** and **content quality standards**.

In the next section, we further explore **extreme sentiment outliers** and generate word clouds for interpretability.

------

### <big><strong>2.3 Word Clouds of Extremes</strong></big>

To interpret what drives extreme positive or negative sentiment in reviews, we extract reviews with **very high or very low polarity** and visualize common keywords using word clouds.

Using `TextBlob` polarity scores, we define:

- **Positive reviews**: polarity > 0.25
- **Negative reviews**: polarity < −0.5

We sample ~2,000 reviews and generate separate word clouds for each polarity segment. Common non-informative words (`place`, `service`, `food`) are excluded to enhance signal.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1qI_9PFE-LcBi5cqNtIC8-_rVpGTyLStN&sz=s4000" width="800"><br><sub>Word cloud of reviews</sub></div>

**Insights**

- Negative clouds surface pain points like `"rude"`, `"never"`, `"horrible"`, `"cashier"`—highlighting service breakdowns and poor experience.
- Positive clouds emphasize `"great"`, `"amazing"`, `"delicious"`, `"love"`—highlighting staff and food quality.
- Neutral words like `"car"`, `"said"` appearing in the negative cloud may indicate **hidden structural issues** (e.g. parking complaints, staff disputes).

**Implication**

This method supports **root cause analysis** for low ratings and informs **business strategy** (e.g., service training, queue management).
 It also provides platforms with a lightweight way to **monitor complaints** and detect emergent issues **without training heavy NLP models**.

------



## <big><strong>3. Social Network Analysis</strong></big>

### <big><strong>3.1 Network Construction</strong></big>

To explore Yelp’s social layer, we construct a user friendship graph based on the `friends` field in `yelp_user.csv`. The goal is to uncover **user clusters**, **network sparsity**, and potential influencer structures.

We transform each `user_id` and their comma-separated `friends` list into edge pairs and build an undirected graph using `NetworkX`.
 We first construct a global sample (~6,000 users) and visualize the network layout using the `spring_layout` algorithm.

<div align="center"><img src="https://drive.google.com/thumbnail?id=1PhjraSLe1tW5Z-5yrrtFjuGkS3EZniX1&sz=s4000" width="500"><br><sub>Sampled Yelp user friendship network (Spring layout)</sub></div>

**Insights**

- The global network is **extremely sparse**: many users are **isolated or have ≤1 connection**.
- Some central nodes act as **hubs**, likely elite or socially active users.
- Yelp’s user graph is not deeply connected—interactions may be **driven more by reviews than by social ties**.

**Implication**

While the Yelp friend system exists, it plays a **limited role** in interaction and discovery. This insight is important when considering **collaborative filtering algorithms** or **social-based recommendations**, which may not be effective without stronger graph connectivity.

To explore real communities, we zoom into a specific region in the next section.

### <big><strong>3.2 Stuttgart Subgraph + Community Detection</strong></big>

To observe meaningful user communities in a real region, we extract the user network for **Stuttgart**, Germany, and apply community detection to uncover **local clusters and influence structures**.

We filter users who have reviewed Stuttgart-based businesses and construct their friend network.
 Using `NetworkX`, we:

- Build the subgraph
- Apply **degree centrality** to identify key users
- Use **Louvain algorithm** for community detection
- Visualize communities via `spring_layout`, `circular_layout`, and `kamada_kawai_layout`

<div align="center"><img src="https://drive.google.com/thumbnail?id=1PMqGoAHXXUM3W5hraY-pcrs1o7PQFZVh&sz=s4000" width="500"><br><sub>Stuttgart user network - Spring layout with Louvain communities</sub></div> 

**Insights**

- The Stuttgart subgraph is **denser and more meaningful** than the global network, with identifiable clusters.
- Louvain detected **5 major communities**, likely reflecting real-world social groups or shared interest zones.
- Key influencer nodes can be identified using degree centrality, which could help **promote content or local campaigns**.

**Implication**

Region-specific network analysis reveals **micro-community structure** not visible in global views.
 For recommendation systems or marketing, this suggests a **localized approach to social features**—e.g., promoting elite content within clusters or using community-aware ranking.



## <big><strong>4. Strategic Takeaways</strong></big>

This project uncovers key behavioral and structural patterns from Yelp’s review ecosystem, offering valuable insights for platform operators, data scientists, and business strategists.

------

### <big><strong>4.1 Platform-Level Recommendations</strong></big>

- **Surface rare but high-impact reviews**
   → Reviews with low ratings and high `useful` scores often carry essential service critiques. Prioritize them in dashboards and feeds.
- **Prioritize elite user onboarding & retention**
   → Elite users write more balanced, high-quality reviews and influence community tone. Track their sentiment profiles and promote them in local clusters.
- **Leverage review time patterns for engagement**
   → Most user activity clusters around late afternoons and weekends. Schedule nudges, reward prompts, or ad delivery in those windows.

------

### <big><strong>4.2 Business-Oriented Strategies</strong></big>

- **Monitor NLP sentiment trends for risk signals**
   → Extreme negative sentiment often co-occurs with `useful` tags. Alert business accounts when sentiment drops below thresholds.
- **Optimize staffing and open hours with check-in curves**
   → Weekday vs. weekend demand patterns differ significantly. Help merchants match resource allocation with user flow.
- **Use location-based clusters for regional strategy**
   → Cities like Las Vegas and Stuttgart show distinct behavioral geography. Offer customized toolkits by metro area.

------

### <big><strong>4.3 Data Product Implications</strong></big>

- **Avoid over-relying on friend networks for recommendation**
   → The global user graph is sparse. Social-based CF may not work unless constrained to local subgraphs or elite nodes.
- **Use sentiment + interaction signals for fraud/risk modeling**
   → Rather than binary fraud detection, define review “risk tiers” based on polarity, outlier behavior, and textual features.
- **Establish useful-vote-based quality metrics**
   → Internal QA systems can incorporate `useful` density as a proxy for informative review rate across regions or categories.
