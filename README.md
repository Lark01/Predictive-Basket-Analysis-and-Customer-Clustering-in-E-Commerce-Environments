# Market Basket Analysis and Segmented Recommendation System for Steam Users
This project applies market basket analysis and clustering techniques to Steam user data in order to build a segmented recommendation system. The system combines user behavioral clustering with association rule mining to generate personalized game recommendations for different types of players.

The project uses two major data mining approaches:
- Market Basket Analysis using Apriori and FP-Growth
- User Segmentation using K-Means and DBSCAN clustering
The final system generates recommendations based on both:
- The games a user already owns or plays
- The behavioral segment the user belongs to

## Project Objectives
The goals of this project are:
- Discover relationships between games frequently played together
- Segment Steam users into meaningful behavioral groups
- Build a recommendation engine tailored to different gamer profiles
- Compare the performance of Apriori and FP-Growth algorithms
- Evaluate clustering approaches for behavioral analysis

## Dataset
The project integrates multiple Steam datasets into a unified data warehouse.
### Game Dataset (gameDim.csv)
Contains:
- Game ID
- Game Name
- Genres
- Tags
- Price
- Release information
### User Transaction Dataset (steamUserFact.csv)
Contains:
- Steam User IDs
- Owned Games
- Playtime Hours
- Purchase Activity

## Data Preprocessing
Beyond the original preprocessing steps done in PowerBI to create the Data Warehouse tables, several preprocessing steps were applied in code before analysis:
### Cleaning
- Removed duplicate transactions
- Removed games with very low engagement
- Filtered users with less than 2 hours of playtime
```
fact_table = fact_table.drop_duplicates()
fact_table = fact_table[fact_table["playtime_hours"] > 2]
```
### Feature Reduction
Unused columns were removed:
```
games_df = games_df.drop(columns=[
    "specs",
    "release_date",
    "price",
    "genres",
    "tags"
])
```
### Transaction Generation
Each Steam user was converted into a transaction containing their played games.
Example:
```
[
    ["Half-Life 2", "Portal", "Terraria"],
    ["Rust", "DayZ", "Arma 3"]
]
```

## Market Basket Analysis
### Transaction Encoding
Transactions were transformed into a one-hot encoded basket format using TransactionEncoder.
```
transaction_enc = TransactionEncoder()
encoded = transaction_enc.fit_transform(transactions)
```
The resulting basket matrix contained:
- 5101 game columns
- Boolean ownership/play indicators
### Apriori Algorithm
Apriori was used to generate frequent itemsets and association rules.
```
freq_item_set = apriori(
    bask,
    min_support=0.02,
    use_colnames=True
)
```
Association rules were generated using lift:
```
rules_apriori = association_rules(
    freq_item_set,
    metric='lift',
    min_threshold=1
)
```
### Apriori Results
The algorithm successfully identified meaningful relationships between games.
Example Rules
| Antecedent               | Consequent                    | Confidence |  Lift |
| ------------------------ | ----------------------------- | ---------: | ----: |
| Half-Life 2: Episode One | Half-Life 2: Episode Two      |      66.8% | 21.43 |
| Half-Life 2: Episode Two | Half-Life 2: Episode One      |      64.6% | 21.43 |
| The Binding of Isaac     | The Binding of Isaac: Rebirth |      35.3% |  5.73 |
| DayZ + Rust              | Arma 3                        |      50.6% |  5.42 |
These results demonstrate that the algorithm captured highly related games and franchise relationships correctly.
### FP-Growth Algorithm
FP-Growth was also applied using the same support threshold.
```
fp_itemset = fpgrowth(
    bask,
    min_support=0.02,
    use_colnames=True
)
```
The generated rules were nearly identical to Apriori.
### FP-Growth Results
FP-Growth produced:
- The same top association rules
- The same number of generated rules
- Faster execution time
This validates both the correctness and efficiency of the FP-Growth approach.
### Algorithm Comparison
| Metric             | Apriori   | FP-Growth |
| ------------------ | --------- | --------- |
| Rules Generated    | 1730      | 1730      |
| Time Taken         | 0.01158 s | 0.01072 s |
| Average Support    | 0.030842  | 0.030842  |
| Average Confidence | 0.265218  | 0.265218  |
| Average Lift       | 1.745984  | 1.745984  |
### Observation
FP-Growth performed slightly faster while generating identical rule quality and quantity. This aligns with the theoretical advantage of FP-Growth, which avoids expensive candidate generation used by Apriori.

## User Segmentation
To improve recommendation quality, users were segmented according to behavioral patterns.
### Feature Engineering
User-level features included:
- Total games owned
- Total playtime
- Average game price
- Genre distribution percentages
Example:
```
user_stats = df_merged.groupby('steam_id').agg(
    total_games=('game_id', 'count'),
    total_playtime=('playtime_hours', 'sum'),
    avg_price=('price_clean', 'mean')
)
```
### K-Means Clustering
K-Means clustering was applied after feature scaling.
```
kmeans = KMeans(
    n_clusters=4,
    random_state=42,
    n_init=10
)
```
The elbow method was used to determine the optimal cluster count.
### Gamer Segments
The final K-Means model produced four major gamer profiles:
| Cluster | Segment                 |
| ------- | ----------------------- |
| 0       | Mainstream Action Gamer |
| 1       | Free-to-Play Casual     |
| 2       | Broad / Power User      |
| 3       | Independent Enthusiast  |
### Segment Characteristics
#### Mainstream Action Gamer
- High playtime
- Focus on action/adventure titles
- Higher average spending
#### Free-to-Play Casual
- Lowest spending
- Lower game counts
- Focus on free multiplayer and casual titles
#### Broad / Power User
- Highest game diversity
- Includes software and non-gaming tools
- Large libraries and high engagement
#### Independent Enthusiast
- Strong interest in indie games
- Moderate-to-high engagement
- Preference for niche titles
### DBSCAN Clustering
DBSCAN was used to identify density-based behavioral groups and outliers.
```
dbscan = DBSCAN(
    eps=3,
    min_samples=50
)
```
### DBSCAN Findings
DBSCAN revealed:
- Micro-clusters missed by K-Means
- Small groups focused entirely on sports or casual games
- Approximately 1900 noise users with irregular behavior
This demonstrated that not all users fit cleanly into broad behavioral categories.

## Segmented Recommendation Engine
The final recommendation system combines:
1. Behavioral clustering
2. FP-Growth association rules
Separate association rules were generated for each user segment.
Example:
- An indie gamer receives indie-focused recommendations
- A mainstream action player receives AAA franchise recommendations
### Interactive Recommendation Widget
The notebook includes an interactive widget built using ipywidgets.
Users can:
- Select a gamer profile
- Enter a target game
- Receive segment-specific recommendations
Example output:
```
Other Mainstream Action Gamers who played 'Rust' also loved:
Arma 3
DayZ
```
## Technologies Used
- Python
- Pandas
- NumPy
- Scikit-learn
- MLXtend
- Matplotlib
- Seaborn
- IPyWidgets
## Key Findings
- FP-Growth is more computationally efficient than Apriori while producing equivalent rules.
- Association rules successfully captured logical relationships between games.
- User segmentation significantly improves personalization quality.
- K-Means provides clean high-level segmentation.
- DBSCAN identifies niche groups and outlier users missed by centroid-based clustering.
