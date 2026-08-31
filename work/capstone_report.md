# Capstone Report — Content Opportunity Scoring

- **Author:** Saurabh Nishad
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** Saurabh07-Nishad/flyrank-ml-internship
- **Date:** August 2026

> Copy this file to `work/capstone_report.md` and fill it in as you build. Sections 1–8
> mirror the Pass / Needs-Work rubric axes, so nothing here is optional. Sections 0 and 9
> are **paper sections**: your deployed research paper must carry both, and they're here so
> you never rebuild them from memory at ship time.

## 0. Abstract

This project asks whether observed search performance signals can be used to identify and prioritize content pages that are strong candidates for refresh or further review. The analysis uses the FlyRank ML Internship Search Intelligence warehouse, focusing on observed Google Search Console impressions, clicks, and average position. A rule-based baseline and a machine-learning approach are evaluated using a holdout validation design while excluding future-derived and label-derived fields from the predictive features. The machine-learning model achieved 71.2% accuracy, 63.0% precision, 43.7% recall, and a 51.6% F1 score on the evaluation data. The resulting output is intended as a directional prioritization and decision-support signal for human content review, not as proof that refreshing a page will cause improved search performance.

## 1. Problem framing

The goal of this project is to support the decision of which content pages should be prioritized for refresh or further review.

The unit of analysis is a content page identified by a pseudonymous content hash ID. The output is a ranked content-opportunity score that can be used to prioritize pages for human review.

A FlyRank editor can use the ranking to start with the highest-priority pages, inspect their underlying search-performance signals, and decide whether a refresh or further investigation is appropriate.

A wrong call has a practical cost: prioritizing a page that does not need attention can waste editorial effort, while overlooking a page with a meaningful opportunity can delay useful content improvements.

Data and machine learning are useful because the warehouse contains many content pages and observed search-performance signals. A repeatable scoring approach can help organize these pages consistently instead of relying only on manual selection.

The output is intended as decision support. A high opportunity score does not prove that refreshing a page will improve its future search performance.

## 2. Data safety

The analysis uses the FlyRank ML Internship Search Intelligence warehouse dataset. The main observation data used for the analysis comes from the content daily performance table, using the March 2026 observation window and April 2026 evaluation window.

The predictive features used by the model are:

- `gsc_impressions`
- `gsc_clicks`
- `gsc_avg_position`

The analysis deliberately excludes future-derived and label-derived fields from the predictive features. In particular, fields such as `trend_direction`, `trend_pct`, and `is_declining_label` are not used as model inputs because they can contain information about the outcome being predicted.

The `client_hash_id` and `content_hash_id` fields are used only to match the same content items across observation and evaluation periods. They are pseudonymous identifiers and are not used as predictive features.

The target is constructed separately from the prediction features using the evaluation-period search position. A content item is assigned a positive target when its April average position is better than its March average position.

No client names, domains, private search queries, credentials, or other client-identifying information are included in the report.

The analysis is designed to avoid using future information as a predictive feature. The resulting model should therefore be interpreted as a directional decision-support tool rather than a causal model of search performance.

## 3. Baseline

The baseline is the transparent rule-based scoring approach developed during Week 4. It uses observed search-performance signals to rank content pages by their potential opportunity for refresh or further review.

The baseline uses observed signals including Google Search Console impressions, clicks, and average position. The approach is intentionally simple and interpretable so that the machine-learning model can be compared against a clear starting point.

The Week 4 baseline produced a ranked population of 176,738 unique content items.

This is a fair comparison because the machine-learning approach addresses the same content-prioritization problem and is evaluated using the same general prediction population. The baseline provides a transparent reference point for judging whether the additional complexity of machine learning provides useful decision-support value.

The baseline should be interpreted as a prioritization rule rather than a prediction of guaranteed future performance.

## 4. Model / analysis

The machine-learning approach is used to identify whether a content page shows an improvement in search position between the observation and evaluation periods.

The prediction features are:

- `march_impressions`
- `march_clicks`
- `march_position`

The target is `position_improved`, where a value of 1 means that the April average search position was better than the March average search position, and 0 means it did not improve.

The model uses only March search-performance signals as predictive inputs. April information is used only to construct the evaluation target and is not included as a feature.

The approach is intended to provide an additional prioritization signal beyond the transparent Week 4 baseline. It is not intended to predict Google's ranking algorithm or establish that a content refresh will cause improved performance.


## 5. Evaluation

The model is evaluated using a holdout validation design so that model performance can be measured on data that was not used to fit the model.

The evaluation uses the same content population used to construct the March-April comparison. The target is based on whether average search position improved between March and April.

The model achieved the following evaluation results:

| Metric | Result |
|---|---:|
| Accuracy | 71.2% |
| Precision | 63.0% |
| Recall | 43.7% |
| F1 score | 51.6% |

The majority-class baseline should also be considered when interpreting accuracy because the target classes are not perfectly balanced. Accuracy alone therefore does not establish that the model provides useful ranking value.

The main errors are false positives, where the model identifies a page as likely to improve even though it does not show an improvement in the evaluation period, and false negatives, where an improvement occurs but the model does not identify it.

These results should be interpreted as directional evidence about the usefulness of observed search-performance signals rather than as evidence of causal impact.


## 6. Interpretation

The model uses three observed March search-performance signals: impressions, clicks, and average position.

These features represent the amount of search visibility and engagement observed before the evaluation period. They provide useful information for identifying pages whose search position may change, but they do not explain why the change occurs.

The model should therefore be interpreted as finding patterns in observed search-performance data rather than discovering causal relationships.

A key limitation of the interpretation is that the current analysis does not establish that any individual feature causes improvement in search position. The signals are useful for prioritization, but human review remains necessary before taking editorial action.


## 7. Recommendation

The output should be used as a prioritization aid for content review.

A FlyRank editor can begin with pages receiving stronger opportunity signals, inspect their impressions, clicks, and average position, and then decide whether a refresh or further investigation is appropriate.

Recommended workflow:

1. Start with the highest-priority pages.
2. Review the underlying search-performance signals.
3. Check the page's content and relevance manually.
4. Decide whether a refresh is justified.
5. Monitor subsequent performance rather than assuming that a refresh will improve results.

The model provides directional decision support rather than a guaranteed outcome. Confidence should therefore be highest when the model signal agrees with the transparent baseline and the underlying search signals are meaningful.

## 8. Reproducibility

The analysis was developed in `work/notebooks/capstone.ipynb`.

The notebook uses DuckDB to query the FlyRank ML Internship Search Intelligence warehouse and uses the March 2026 observation data with April 2026 evaluation data.

The main analysis steps are:

1. Authenticate to the Hugging Face dataset.
2. Load the March and April content-performance data through DuckDB.
3. Match content items across the two periods using the pseudonymous client and content identifiers.
4. Construct the `position_improved` target from the change in average search position.
5. Use March impressions, clicks, and average position as model features.
6. Train and evaluate the machine-learning model using a holdout validation design.
7. Compare the model results with the Week 4 rule-based baseline.

Randomness should be controlled using the random seed specified in the notebook wherever applicable.

The notebook should be run from top to bottom before submission to confirm that the reported results can be reproduced without errors.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset. [FlyRank](https://flyrank.ai).

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
