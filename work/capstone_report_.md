Capstone Report — Clustering

Author: Ahmed Mansour
Lane: Clustering
Repository: "fly-rank-ml-internship-starter"
Date: September 7, 2026

0. Abstract

This project investigates whether content items can be grouped into distinct performance archetypes using observed search visibility, engagement, and content characteristics.

The analysis applies K-Means clustering to eight standardized features representing observed content performance and characteristics. The number of clusters was selected by comparing silhouette scores for candidate values of k from 2 through 6. The selected five-cluster solution achieved a final silhouette score of 0.4136 on a reproducible 10,000-item evaluation sample.

The resulting clusters are used to support directional content-prioritization decisions, including protecting relatively strong performers, improving moderate performers, reviewing weaker performers, and monitoring unusual small groups.

The analysis is intended as decision-support rather than a causal study. The findings describe observed patterns in the available data and do not claim to prove changes in or causal effects from any search-engine algorithm.

1. Problem Framing

The decision supported by this analysis is how to prioritize content for further review and optimization.

The unit of analysis is a content item within a client, represented as a client-content pair. The output is a cluster assignment and a cluster profile describing observed performance and content characteristics.

A human editor can use the output to identify groups of content that may deserve different actions, such as:

- Protecting relatively strong performers
- Improving moderate performers
- Rewriting weaker performers
- Monitoring unusual or low-volume groups

The cost of a wrong decision is that useful content could be unnecessarily changed, while weak or underperforming content could receive insufficient attention.

Machine learning helps by identifying multidimensional patterns that may be difficult to capture with a single rule based only on CTR or search position.

2. Data Safety

The analysis uses an internship-provided search-performance dataset.

The main data sources consist of content metadata and daily content-performance records for the March 2026 analysis period.

The modeling features were:

- "ctr_pct"
- "total_impressions"
- "avg_position"
- "word_count"
- "char_count"
- "backlinks"
- "search_volume"
- "competition"

Pseudonymous identifiers such as client and content hash identifiers were used only for joining and grouping and were deliberately excluded from the model features.

Other identifier and hash fields were also excluded from the modeling features.

Outcome-derived fields such as "trend_direction" and "trend_pct" were not used as model features.

Publication-status fields were used only to restrict the analysis to relevant published, non-deleted content.

No client names, private search queries, private credentials, raw data exports, or other client-identifying information are included in the public analysis artifacts.

3. Baseline

The baseline is a transparent rule-based segmentation using CTR and average search position.

The rules were:

- Strong visibility: CTR >= 0.3% and average position <= 15
- Needs improvement: CTR < 0.3% and average position <= 15
- Low visibility: average position > 15

On the same 86,661 usable content items, the baseline produced:

- Needs improvement: 44,201 items
- Low visibility: 26,185 items
- Strong visibility: 16,275 items

This baseline is useful because it is simple, transparent, and easy to reproduce.

It provides a reference point for determining whether a multidimensional clustering approach reveals additional structure beyond two basic performance indicators.

Because this is an unsupervised task with no ground-truth target label, the baseline is not evaluated using classification accuracy.

Instead, the clustering output is compared descriptively with the rule-based segments on the same analysis data.

4. Model and Analysis

The analysis uses K-Means clustering because the objective is to group content items according to similarity across multiple observed numerical characteristics.

The eight features were standardized before clustering:

- "ctr_pct"
- "total_impressions"
- "avg_position"
- "word_count"
- "char_count"
- "backlinks"
- "search_volume"
- "competition"

Rows with missing feature values were removed before scaling.

Average position values of zero were treated as missing.

There is no target label because this is an unsupervised clustering problem. The model assigns each content item to one of the discovered clusters.

K-Means solutions with k=2 through k=6 were compared using a reproducible random sample of 10,000 items.

The silhouette scores were:

Number of clusters| Silhouette score
2| 0.4007
3| 0.4070
4| 0.4089
5| 0.4300
6| 0.4272

The highest observed score occurred at k=5, so five clusters were selected for the final model.

The final K-Means model used:

- "random_state = 42"
- "n_init = 10"
- "k = 5"

The final model was fitted to 86,661 usable content items.

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

The evaluation is directional rather than a predictive holdout evaluation.

The analysis uses the March 2026 performance period and does not claim a client-grouped or time-separated test set.

Error and Stability Considerations

The main interpretation concern is that Clusters 3 and 4 are extremely small and contain unusual values.

Cluster 3 has very high observed CTR but only about 2.23 average impressions.

Cluster 4 has an unusually high average backlink count.

These groups should therefore be treated as special cases rather than broad population segments.

Their small sizes mean that conclusions about them are less reliable than conclusions about the three major clusters.

6. Cluster Interpretation

Cluster 2 — Dominant Performance Group

Cluster 2 is the dominant group, containing 65,382 items.

It has:

- Average CTR: 0.314%
- Average impressions: approximately 2,063
- Average position: 13.15

Its large size and relatively strong observed performance make it the main group to protect and monitor.

The recommendation is not that every item in this cluster is inherently successful, but that the group represents a large observed performance pattern that should be preserved while being monitored for changes.

Cluster 0 — Moderate Performance Group

Cluster 0 contains 10,619 items.

Its profile includes:

- Average CTR: 0.307%
- Average position: 14.05

This suggests a group with reasonable observed performance but potential room for targeted improvement.

Content in this group may benefit from focused optimization while maintaining existing search visibility.

Cluster 1 — Weaker Major Group

Cluster 1 contains 10,587 items.

It has:

- Average CTR: 0.192%
- Average position: 19.34

Among the three major clusters, this group has the weakest observed performance profile.

It is therefore a candidate for deeper content and metadata review and possible rewriting.

Cluster 3 — Rare High-CTR Group

Cluster 3 contains only 60 items.

Its observed average CTR is approximately 69.290%, but its average impressions are only about 2.23.

This combination makes the cluster unsuitable as a broad content recommendation category.

The unusually high CTR should be validated in context before any broad action is taken.

Cluster 4 — Rare High-Backlink Group

Cluster 4 contains only 13 items.

Its average backlink count is approximately 689,074, making it an extreme outlier relative to the larger groups.

Because of its very small size and unusual profile, this cluster should be investigated separately rather than used as a general content-action category.

7. Comparison With the Baseline

The baseline segments content using only CTR and average search position.

The clustering approach adds additional dimensions by incorporating:

- Content length
- Character count
- Impressions
- Backlinks
- Search volume
- Competition
- CTR
- Average search position

This allows the analysis to identify groups that may share similar combinations of performance and content characteristics even when they would fall into the same simple rule-based segment.

The clustering output therefore provides a more multidimensional segmentation than the baseline.

However, this should be interpreted as an analytical advantage rather than proof that clustering will produce better business outcomes.

8. Recommendations

Based on the observed cluster profiles, the recommended prioritization is:

1. Protect Cluster 2

This is the largest cluster and shows relatively strong observed CTR and search position.

Content in this group should be protected and monitored to preserve observed performance.

2. Improve Cluster 0

This group shows moderate observed performance and may benefit from targeted optimization while maintaining existing visibility.

3. Rewrite Cluster 1

This group has lower observed CTR and weaker average search position than the other major clusters.

It should therefore receive deeper content and metadata review, with rewriting considered where appropriate.

4. Monitor Cluster 3

This cluster is very small and has an unusually high CTR combined with very low impressions.

It should be validated and monitored rather than used as a broad recommendation category.

5. Monitor Cluster 4

This cluster contains only 13 items and has an extreme backlink profile.

It should be investigated separately before applying broad content actions.

Confidence

Confidence is highest for the three major clusters because they contain the overwhelming majority of observations.

Confidence is lower for Clusters 3 and 4 because of their very small sizes and unusual values.

These recommendations are directional decision-support guidance and should not be interpreted as causal conclusions.

9. Reproducibility

The analysis was developed in Google Colab using the committed notebook:

"work/notebooks/capstone.ipynb"

The notebook can be rerun from a fresh environment after providing the required data-access credential through the appropriate private environment mechanism.

No access token or credential is hard-coded in the notebook or included in the public repository.

The main setup dependency is DuckDB, installed with:

"%pip -q install duckdb"

The analysis uses:

- DuckDB
- pandas
- NumPy
- scikit-learn
- Matplotlib

Key reproducibility settings are:

- Random seed: 42
- K-Means "n_init": 10
- Candidate k values: 2, 3, 4, 5, 6
- Model-selection/evaluation sample size: 10,000
- Final number of clusters: 5

The notebook performs:

1. Data loading
2. March 2026 performance aggregation
3. Content-metadata integration
4. Data preprocessing
5. Feature scaling
6. K-Means model selection
7. Final clustering
8. Baseline comparison
9. Silhouette evaluation
10. Cluster profiling
11. Recommendation generation
12. PCA visualization

No sealed or blind holdout evaluation is claimed in this report.

10. Limitations

Several limitations should be considered when interpreting the findings.

First, the analysis is based on observed data from a defined analysis period rather than a controlled experiment.

Second, clustering is an unsupervised technique, so there is no ground-truth label against which cluster assignments can be evaluated for predictive accuracy.

Third, the silhouette score measures cluster separation and cohesion; it does not directly measure business impact.

Fourth, the smallest clusters contain unusual observations and should not be generalized to the broader content population.

Finally, the analysis identifies associations and patterns in the available data. It does not establish that any observed content characteristic caused a change in search performance or that the findings prove how a search-engine algorithm operates.

11. Acknowledgments and Data Credit

This project was developed as part of the FlyRank ML Internship using the internship-provided search-performance dataset.

The analysis follows the internship's public-data and privacy requirements and intentionally excludes client-identifying information, private queries, credentials, URLs, and raw data exports from the public artifacts.

12. Final Conclusion

The analysis demonstrates that K-Means clustering can provide a useful multidimensional view of content performance beyond a simple CTR-and-position rule.

The five-cluster solution achieved a silhouette score of 0.4136, with three major clusters representing the majority of observations and two very small clusters containing unusual profiles.

The resulting segmentation supports a practical prioritization framework:

- Cluster 2: Protect and monitor
- Cluster 0: Improve
- Cluster 1: Rewrite
- Cluster 3: Monitor and validate
- Cluster 4: Monitor and investigate

The strongest conclusions concern the three major clusters. The two small clusters should be treated as exceptional cases rather than generalized content archetypes.

Overall, the model provides a reproducible, multidimensional decision-support framework for prioritizing content review while maintaining appropriate caution about causality, generalization, and unusual observations.
