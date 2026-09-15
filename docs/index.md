# Predicting Organic Traffic Decay: A Decision-Support Model for Content Refresh Prioritization

## 1. Abstract
Search visibility is highly volatile, yet editorial teams often only notice content decay after the traffic has completely collapsed. This project investigates whether pre-period search signals can reliably identify which published articles are at the highest risk of losing organic clicks. Using a dataset of 129,990 published URLs from the FlyRank warehouse, we engineered historical features including click-through rate, average position, and AI overview exposure. We trained a Random Forest model validated under a strict grouped split (holding out entire client domains), achieving a Precision@20 of 85.00%—a 4.9× lift over the natural decay base rate. The resulting output is not an automated fix, but rather a ranked decision-support queue that helps SEO and content teams prioritize their refresh pipelines before visibility drops further.

## 2. Introduction & Problem Statement
For large publishers and e-commerce sites, maintaining historical content is just as critical as publishing new pages. However, editorial resources are finite. When teams must manually sift through thousands of URLs to decide what to update, they often rely on lagging indicators (e.g., waiting for a 50% year-over-year traffic drop). 

This work supports a proactive decision: **"Which pages should our editors review and update this week?"** By identifying pages that exhibit patterns associated with impending traffic loss—such as slipping from striking-distance rankings or underperforming expected CTR baselines—we can allocate human editorial effort where it has the highest potential impact to protect existing search visibility.

## 3. Data
This analysis relies on the FlyRank ML Internship dataset, queried directly via DuckDB. 
* **Scope:** 129,990 rows of published, non-deleted content pages across 25 active client domains.
* **Time Windows:** Features (signals) were aggregated from the pre-period (January 1, 2026, to April 30, 2026). The target outcome was evaluated on post-period data (May 1, 2026, onward).
* **Exclusions:** We strictly excluded unpublished drafts, deleted pages, and zero-impression anomalies. To protect privacy and adhere to public-safe guidelines, no client names, raw URLs, or private queries are included in this dataset or analysis.

## 4. Methodology
We framed this as a binary classification problem to rank decay risk.
* **Label Definition:** A page was flagged as `is_decaying = 1` if its total clicks in the post-period were lower than its total clicks in the pre-period.
* **Features:** We utilized strictly pre-period signals, including average position, CTR, search volume, word count, and AI traffic percentage (`sessions_ai` / total sessions).
* **Leakage Checks:** We ran a rigorous leakage audit, ensuring zero future-window or outcome-derived columns existed in the feature matrix.
* **Validation Design (Grouped Split):** Random splits in SEO data often allow models to artificially memorize domain-level characteristics. To test true generalization, we validated the model using 5-fold `GroupKFold`, grouping by `client_hash_id` so the model was always tested on client websites it had never seen during training.

## 5. Results
The natural base rate of traffic decay in the dataset was **17.29%**. 

Our validation audit exposed a massive memorization gap when comparing naive validation to honest validation:

| Evaluation Metric | Precision@20 | Note |
| :--- | :--- | :--- |
| **Observed Base Rate** | 17.29% | Naive random floor |
| **Random Forest (Naive Random Split)** | 100.00% | Artificially inflated by domain memorization |
| **Random Forest (Honest Grouped Split)** | **85.00%** | **4.9× lift; true out-of-domain generalization** |

The 15.00% memorization gap confirms that grouped splits are necessary for honest SEO modeling. Feature importance analysis revealed that pre-period click-through rate (77.69% relative importance) and average position (10.52%) were the strongest indicators of subsequent decay.

## 6. Limitations & Honest Framing
This model and its recommendations are designed strictly for **decision-support**. 
* **Directional, Not Causal:** We measured an observed association between pre-period metrics (like AI exposure and lagging CTR) and future traffic drops. This does not prove that AI overviews *caused* the drop, nor does it guarantee that executing a content refresh will successfully restore traffic.
* **Blind to Technical SEO:** The model relies on on-page characteristics and historical click signals. It cannot diagnose technical drops (e.g., accidental `noindex` tags, server downtime, or manual algorithmic penalties).

## 7. Ranked Recommendations (Action Playbook)
The model outputs a ranked queue of at-risk pages, which we map to specific human-reviewed actions based on feature archetypes:

1. **Optimize format for AI Summaries:** Triggered for pages in striking distance (Positions 4–10) with high AI traffic exposure. *Action:* Restructure headers and lists to directly answer AI summary queries.
2. **Rewrite Title/Meta:** Triggered for pages with high search volume but historically below-median CTR. *Action:* A/B test a more compelling title tag to capture lost clicks.
3. **Comprehensive Update:** Triggered for general decay risk. *Action:* Audit for outdated information and expand topic depth.

**The No-Go List (Mandatory Human Review):**
Automated updates should *never* be applied to:
* **Legal/Compliance Pages:** Terms of Service or Privacy Policies.
* **Navigational Queries:** Pages driving traffic for strict brand names.
* **Highly Seasonal Content:** (e.g., "2025 Holiday Guide" decaying in February is a natural, unpreventable cycle).

## 8. Reproducibility
The code used to generate this analysis, audit the validation splits, and export the decision-support queue is fully available in the project repository. 
* **Repository Link:** [Insert your GitHub Repo URL here]
* **Notebooks:** See the `work/notebooks/` directory for data ingestion, model training, and playbook generation scripts.

## 9. Acknowledgments
Built on the FlyRank ML Internship dataset [https://flyrank.ai](https://flyrank.ai).
