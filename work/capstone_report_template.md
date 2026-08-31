# Capstone Report — Machine Learning

- **Author:** Mohamed Kroush
- **Lane:** Machine Learning
- **Repo:** https://github.com/MohamedKroush/Flyrank-ML1
- **Date:** August 31, 2026

> Copy this file to `work/capstone_report.md` and fill it in as you build. Sections 1–8
> mirror the Pass / Needs-Work rubric axes, so nothing here is optional. Sections 0 and 9
> are **paper sections**: your deployed research paper must carry both, and they're here so you
> never rebuild them from memory at ship time.

## 0. Abstract

This research asks whether a data-driven priority score can help editors identify content pages that should be reviewed first for potential search-performance decline. Using pseudonymized FlyRank internship warehouse data, the analysis defines an observed decline outcome from changes in Google Search impressions and compares a transparent rule-based ranking with a Random Forest classifier. Across 94,559 evaluated content-level observations, the observed decline base rate was 37.3%. The transparent ranking reached a Precision@50 of 0.480, while the Random Forest reached a ROC-AUC of 0.640 on a stratified evaluation split. The output is intended as a ranked, human-reviewed queue for editorial decision support rather than an automated content-change system.

## 1. Problem framing

The decision this supports is which content pages an editor should review first when editorial resources are limited. The unit of analysis is a content page. The output is a ranked content-review queue based on search-performance signals, with a priority score, reason code, and recommended action. A human editor uses the output to decide which pages deserve review first. A wrong call can waste limited editorial time by prioritizing a page that does not need attention or by missing a page that may deserve review. Data and ML help make the prioritization process repeatable and allow comparison between a simple transparent rule and a more flexible model.

The analysis is intended as decision support for content review, not as a prediction of Google's ranking algorithm or as evidence that any particular editorial change causes a performance change.

## 2. Data safety

The analysis uses the FlyRank ML Internship warehouse dataset:

https://huggingface.co/datasets/FlyRank/internship-warehouse

The working analysis uses a March 2026 mid-panel window. The unit of analysis is one content page, aggregated to content-level search-performance measurements including impressions, clicks, and average search position.

The following fields were deliberately excluded:

- `health_score`
- `priority_score`
- `action_type`
- `trend_direction`
- `trend_pct`

The product-decision fields were excluded because they encode decisions that should not be used as predictive inputs. The trend fields were excluded because they are label-derived and could leak information about the outcome.

Pseudonymous content and client IDs are used only for grouping and joining historical observations, never as predictive model features.

No client names, domains, private queries, credentials, raw exports, or other client-identifying information appear anywhere in `work/`.

## 3. Baseline

The transparent rule or score built first prioritizes pages with meaningful search visibility and weaker average search position.

The broader binary rule flags pages with at least 250 monthly impressions and an average search position above 20 (that is, worse than position 20).

The observed decline base rate was 37.3%. The broader rule achieved 0.409 precision and 15.8% coverage.

The ranked version of the baseline achieved Precision@50 = 0.480. This means that 48.0% of the 50 highest-ranked pages were observed to meet the decline definition.

The baseline is a fair comparison because it uses the same underlying data and outcome definition while remaining simple and interpretable.

## 4. Model / analysis

The comparison model is a Random Forest classifier. It fits the lane because it provides a simple non-linear model that can compare multiple observed search-performance signals while still allowing feature importance to be inspected.

The exact predictive feature list is:

- `impressions_month`
- `clicks_month`
- `avg_position_month`

Client and content identifiers were left out because they are grouping identifiers rather than meaningful predictive features. Product-decision fields and label-derived fields were also left out to avoid leakage.

The target is an observed decline label: a page is labeled as declining when impressions in the later portion of the analysis window are more than 20% below the preceding period, subject to a minimum prior-period impression threshold.

## 5. Evaluation

The Random Forest used a 75/25 stratified random train/test split with `random_state=42`.

The results were:

| Approach | Precision | ROC-AUC |
|---|---:|---:|
| Overall base rate | 0.373 | — |
| Rule baseline | 0.409 | — |
| Random Forest | 0.753 | 0.640 |

At the default 0.5 classification threshold, the Random Forest achieved 0.753 precision, 0.132 recall, and 0.660 accuracy.

The model was selective rather than comprehensive: when it flagged a page, it was often a declining page, but it missed many observed declines.

The evaluation uses a stratified random split rather than a grouped or time-aware split. Because pages can share client and content structure and the data has temporal ordering, this is an initial evaluation rather than strong evidence of production-level generalization.

## 6. Interpretation

The model found that monthly impressions had the highest feature importance at 0.572, followed by clicks at 0.244 and average search position at 0.185.

The lowest visibility quartile had a 48.7% observed decline rate compared with 29.4% in the highest visibility quartile.

Search position showed a weaker but directionally useful relationship: decline rates were 34.8% for positions 1–10, 39.1% for positions 11–20, 43.0% for positions 21–30, and 40.4% for positions above 30.

A notable negative result is that neither visibility nor position perfectly identifies future observed decline. The Random Forest also had low recall despite its higher precision, meaning it was selective rather than comprehensive.

These are model associations and observed relationships. They do not establish causal effects.

## 7. Recommendation

The ranked actions or decisions supported by the output are:

1. **Review highly visible pages with weak search position first.** These pages have enough search visibility to make review potentially valuable while showing weaker average positions.
2. **Prioritize pages with meaningful impressions and positions above 20.** The baseline analysis found higher observed decline rates around positions 21–30 than among positions 1–10.
3. **Treat low-visibility pages separately.** The lowest visibility group had the highest observed decline rate, but low visibility alone does not establish that a page needs an immediate content change.
4. **Require human review before taking action.** Editors should inspect search intent, content quality, freshness, relevance, and the page's historical context before changing content.
5. **Monitor the ranked queue over time.** The score should be reconsidered if the underlying traffic mix, measurement availability, or observed decline rate changes substantially.

These recommendations are prioritization guidance, not automatic content-change instructions.

## 8. Reproducibility

The main analysis is contained in `work/notebooks/capstone.ipynb`.

The notebook uses Python, pandas, DuckDB, scikit-learn, and matplotlib.

The Random Forest uses:

- `n_estimators=100`
- `max_depth=6`
- `min_samples_leaf=50`
- `random_state=42`

The train/test evaluation uses a 75/25 stratified random split with `random_state=42`.

The capstone notebook generates the charts under `work/outputs/`.

To reproduce the analysis from a fresh clone, open `work/notebooks/capstone.ipynb` in Google Colab or a compatible Jupyter environment, run the notebook from top to bottom, and provide the required Hugging Face read token when prompted.

The deployed research paper is:

https://mohamedkroush.github.io/Flyrank-ML1/

The project repository is:

https://github.com/MohamedKroush/Flyrank-ML1

No sealed or blind holdout evaluation is claimed in this report.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset, with the dataset available at:

https://huggingface.co/datasets/FlyRank/internship-warehouse

Dataset provider: https://flyrank.ai

---

> **Claims checklist before submitting:** observed / measured / directional / decision-support
> 
> **Metrics vs. base rate:** report your task's base rate (majority-class %) next to any
> precision@K or accuracy — a high score can just be a high base rate. AUC / lift over
> baseline are the honest discrimination numbers.
> 
> language everywhere · no causal claims without an experiment or causal design · no
> "predicted Google's algorithm" · no client-identifying details · numbers in this report
> match a fresh re-run.
