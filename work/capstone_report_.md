Capstone Report — Clustering

- Author: Ahmed Mansour
- Lane: Clustering
- Repo: fly-rank-ml-internship-starter
- Date: 2026-09-07

0. Abstract

This project asks whether content items can be grouped into distinct performance archetypes using observed search visibility, engagement, and content characteristics. The analysis uses the FlyRank internship warehouse, focusing on March 2026 content performance joined with published, non-deleted content metadata. K-Means clustering was applied to eight standardized features, with the number of clusters selected by comparing silhouette scores for k=2 through k=6. The selected five-cluster solution achieved a final silhouette score of 0.4136 on a reproducible 10,000-item evaluation sample. The resulting clusters are intended to support directional content prioritization decisions such as protecting, improving, rewriting, or monitoring groups of content.

1. Problem framing

The decision supported by this analysis is how to prioritize content for further review and optimization.

The unit of analysis is a content item within a client, represented by a client-content pair. The output is a cluster assignment and a cluster profile describing observed performance and content characteristics.

A human editor can use the output to identify groups of content that may deserve different actions, such as protecting relatively strong performers, improving moderate performers, rewriting weaker performers, or monitoring unusual small groups.

The cost of a wrong call is that useful content could be unnecessarily changed, while weak or underperforming content could receive insufficient attention. ML helps by identifying multidimensional patterns that are difficult to capture with a single rule based only on CTR or search position.

2. Data safety

The analysis uses the FlyRank internship warehouse, build "v20260703".

The main sources are "dim_content" and the March 2026 partition of "fact_content_daily_performance".

The modeling features were:

- "ctr_pct"
- "total_impressions"
- "avg_position"
- "word_count"
- "char_count"
- "backlinks"
- "search_volume"
- "competition"

Pseudonymous identifiers such as "client_hash_id" and "content_hash_id" were used for joining and grouping only and were deliberately excluded from model features. Other identifier/hash fields were also excluded.

Outcome-derived fields such as "trend_direction" and "trend_pct" were not used as features. "is_published" and "is_deleted" were used only to filter the analysis to published, non-deleted content.

No client names, private queries, URLs, or other client-identifying information are included in the analysis or working artifacts.

3. Baseline

The baseline is a transparent rule-based segmentation using CTR and average search position.

The rules were:

- "Strong visibility": CTR >= 0.3% and average position <= 15
- "Needs improvement": CTR < 0.3% and average position <= 15
- "Low visibility": average position > 15

On the same 86,661 usable content items:

- Needs improvement: 44,201 items
- Low visibility: 26,185 items
- Strong visibility: 16,275 items

This baseline is useful because it is simple, transparent, and easy to reproduce. It provides a reference point for determining whether a multidimensional clustering approach reveals additional structure beyond two basic performance indicators.

Because this is an unsupervised task with no ground-truth target label, the baseline is not evaluated using classification accuracy. Instead, the clustering output is compared descriptively with the rule-based segments on the same analysis data.

4. Model / analysis

The analysis uses K-Means clustering because the objective is to group content items according to similarity across multiple observed numerical characteristics.

The eight features were standardized before clustering:

"ctr_pct", "total_impressions", "avg_position", "word_count", "char_count", "backlinks", "search_volume", and "competition".

Rows with missing feature values were removed before scaling. Average position values of zero were treated as missing.

There is no target label because this is an unsupervised clustering problem. The model assigns each content item to one of the discovered clusters.

K-Means solutions with k=2 through k=6 were compared using a reproducible random sample of 10,000 items. The silhouette scores were:

- k=2: 0.4007
- k=3: 0.4070
- k=4: 0.4089
- k=5: 0.4300
- k=6: 0.4272

The highest observed score was at k=5, so five clusters were selected for the final model. The final K-Means model used "random_state=42" and "n_init=10" and was fitted to 86,661 usable content items.

5. Evaluation

The clustering model was evaluated using silhouette score and cluster-profile analysis.

A reproducible sample of 10,000 items was used for model selection and final silhouette evaluation, with random seed 42.

The final five-cluster solution achieved a silhouette score of 0.4136 on the evaluation sample.

The final clusters contained:

- Cluster 0: 10,619 items
- Cluster 1: 10,587 items
- Cluster 2: 65,382 items
- Cluster 3: 60 items
- Cluster 4: 13 items

The evaluation is directional rather than a predictive holdout evaluation. The analysis uses the March 2026 performance partition and does not claim a client-grouped or time-separated test set.

The main error-analysis concern is that Clusters 3 and 4 are extremely small and contain unusual values. Cluster 3 has very high observed CTR but only about 2.23 average impressions, while Cluster 4 has an unusually high average backlink count. These groups should therefore be treated as special cases rather than broad population segments.

6. Interpretation

Cluster 2 is the dominant group, containing 65,382 items. It has an average CTR of 0.314%, average impressions of about 2,063, and an average position of 13.15. Its size and relatively strong observed performance make it the main group to protect and monitor.

Cluster 0 contains 10,619 items with an average CTR of 0.307% and average position of 14.05. Its profile suggests a group with reasonable observed performance but room for targeted improvement.

Cluster 1 contains 10,587 items and has the weakest profile among the three major clusters, with an average CTR of 0.192% and average position of 19.34. This group is therefore a candidate for deeper review and possible rewriting.

Clusters 3 and 4 are very small. Cluster 3 contains only 60 items and has an unusually high average CTR of 69.290%, but only about 2.23 average impressions. Cluster 4 contains only 13 items and has an unusually high average backlink count of approximately 689,074. These extreme values are a negative result for broad interpretation: they indicate that some clusters may represent rare or unusual observations rather than useful general archetypes.

Compared with the baseline, the clustering approach provides more multidimensional segmentation because it considers content characteristics and several performance-related features rather than only CTR and average position.

7. Recommendation

1. Protect Cluster 2 — This is the largest cluster and shows relatively strong observed CTR and search position. Content in this group should be protected and monitored to preserve observed performance.

2. Improve Cluster 0 — This group shows moderate observed performance and may benefit from targeted optimization while maintaining existing visibility.

3. Rewrite Cluster 1 — This group has lower observed CTR and weaker average search position than the other major clusters, making it a priority for deeper content and metadata review.

4. Monitor Cluster 3 — The cluster is very small and has an unusually high CTR combined with very low impressions. It should be validated and monitored rather than used as a broad recommendation category.

5. Monitor Cluster 4 — This cluster contains only 13 items and has an extreme backlink profile. It should be investigated separately before applying broad content actions.

Confidence is highest for the three major clusters and lower for Clusters 3 and 4 because of their very small sizes and unusual values. These recommendations are directional and decision-support guidance, not causal conclusions.

8. Reproducibility

The analysis was developed in Google Colab using the committed notebook:

"work/notebooks/capstone.ipynb"

The notebook can be rerun from a fresh clone/opened Colab environment after providing the required Hugging Face access token through the Colab secret named "HF_TOKEN". The token is not hard-coded in the notebook.

The main setup command is:

"%pip -q install duckdb"

The analysis uses DuckDB, pandas, NumPy, scikit-learn, and Matplotlib.

The key reproducibility settings are:

- Random seed: "42"
- K-Means "n_init": "10"
- Candidate k values: "2, 3, 4, 5, 6"
- Model-selection/evaluation sample size: "10,000"
- Final number of clusters: "5"

The notebook performs the data loading, March 2026 performance aggregation, join with content metadata, preprocessing, feature scaling, K-Means model selection, final clustering, baseline comparison, evaluation, cluster profiling, recommendations, and PCA visualization.

No sealed or blind holdout evaluation is claimed in this report.

9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset. Data credit: https://flyrank.ai
