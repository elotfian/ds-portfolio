# Bank Marketing: Predicting Term Deposit Subscription

This project compares different classification models for predicting whether a bank client will subscribe to a term deposit following a marketing campaign.

The dataset contains 41,188 client records from direct marketing campaigns conducted by a Portuguese banking institution, released to the UCI Machine Learning Repository by Moro, Cortez & Rita (2014).

Note: the paper's own results were run on a larger, separate dataset (52,944 records, 2008–2013) with a different feature set - this project uses the smaller public release, not the exact data analyzed in the original paper.

The objective is to identify clients with a higher probability of subscription, using only information that would genuinely be available before deciding to make a call: the client's demographic and financial profile, their history with previous campaigns, and the broader economic conditions at that point in time.

## Data Leakage

A key modeling decision is the removal of the `duration` feature, which records the length of the current marketing call.

Although `duration` is known to be strongly associated with subscription outcome — per the dataset documentation, calls with very short duration essentially never result in a subscription — it is only known after the call has taken place. Since the goal is to identify promising clients before contacting them, using this feature would introduce data leakage and make the reported performance unrealistically high.

For that reason, `duration` is excluded from every model in the comparison.

## Class Imbalance and Metric Choice

The target variable is highly imbalanced: approximately 88.7% of clients did not subscribe, and approximately 11.3% subscribed.

This means accuracy alone is not an appropriate measure of model quality. A classifier predicting every client as a non-subscriber would already achieve about 89% accuracy while providing no practical value — it would never identify a single subscriber.

Beyond accuracy, the choice of evaluation metric also has to reflect the *cost* of different errors, and those costs are not symmetric here:

* **A false negative** — a client who would have subscribed, but the model doesn't flag them — is a **missed sale**. That client is never called, and the opportunity is lost entirely.
* **A false positive** — a client who wouldn't subscribe, but the model flags them anyway — costs a few minutes of a call center agent's time. Low cost, easily absorbed.

Because a missed subscriber is far more costly to the business than a wasted call, this project treats **recall as more important than precision**, and uses **F2-score** — which weights recall twice as heavily as precision — as the primary model selection metric, rather than the more commonly used F1-score, which weights them equally.

F1 is still reported throughout the comparison, but only as a point of contrast to F2 — a reminder that optimizing for the wrong metric can select the wrong model for this specific business problem.

Model evaluation reports, for every model:

* Accuracy
* Precision
* Recall
* F1-score
* **F2-score**

## Models

Model families are compared, with class-imbalance handling tested as an *independent, separately-evaluated dimension* for the models that support it natively in scikit-learn:

* Logistic Regression
* Logistic Regression (class-weighted via `class_weight='balanced'`)
* Linear Discriminant Analysis (LDA)
* Quadratic Discriminant Analysis (QDA)
* Naive Bayes
* Support Vector Machine with an RBF kernel
* XGBoost (standard, tuned via RandomizedSearchCV)
* XGBoost (class-weighted via `scale_pos_weight`)

The models were selected to compare different assumptions about the relationship between the predictors and the target.

Logistic Regression and LDA provide strong linear baselines, while QDA allows class-specific covariance structures. Naive Bayes provides a probabilistic model based on conditional independence assumptions.

The RBF-kernel SVM introduces a flexible nonlinear decision boundary, allowing the analysis to test whether nonlinear relationships improve classification performance. This flexibility comes at a real computational cost, however — SVM's training time scales poorly as dataset size grows, which becomes a practical concern on a dataset of this size.

XGBoost provides a complementary tree-based approach that can capture nonlinear effects and feature interactions without explicitly specifying their form, while typically training much faster than SVM at this scale — a real practical advantage on top of any accuracy difference.

### Why some models get class-weighted variants and others don't

Class weighting is tested only where scikit-learn provides a native mechanism for it:

* **Logistic Regression and XGBoost** both support native class weighting (`class_weight='balanced'` and `scale_pos_weight` respectively) and are cheap to retrain, so both are evaluated in weighted and unweighted form — an additional, independent way to account for class imbalance during training itself, alongside the F2 metric and threshold selection below.
* **LDA, QDA, and Naive Bayes have no comparable mechanism.** As generative models, they estimate class-conditional distributions directly rather than optimizing a loss function that a `class_weight`- or `scale_pos_weight`-style parameter can reweight. (scikit-learn's `priors` parameter on LDA/QDA adjusts prior class probabilities — a related but distinct mechanism from loss reweighting — and is not used here.) Class imbalance is instead addressed for these models through the F2 metric and threshold optimization.
* **SVM's inclusion in class-weighting and full threshold optimization is a cost/benefit call**, given its high training time relative to the other models.

## Threshold Optimization

Rather than relying on the default 0.5 probability threshold, this project searches for the decision threshold that maximizes F2 for each model, using data the model wasn't trained on so the threshold reflects genuine performance rather than overfitting. This applies the same recall-over-precision priority that motivated choosing F2 as the primary metric in the first place.

## Model Interpretation

Predictive performance alone is not sufficient for a marketing application. The project therefore uses SHAP values to examine how individual features influence model predictions, providing both global insight into which variables contribute most to the model and local explanations for individual client predictions. The goal is to understand not only which model performs best, but what patterns it relies on — and whether those patterns are actionable by the business.

## Business Perspective

The final stage translates model performance into a practical marketing decision. Rather than simply predicting subscribers and non-subscribers, predicted probabilities and a threshold tuned specifically for this business problem are used together to prioritize clients for a marketing campaign, reflecting the same cost asymmetry that motivated the choice of F2 in the first place.

## Workflow Summary

data preparation -> leakage prevention -> collinearity checks -> model comparison (F1 vs. F2) -> class-weighted variants where applicable -> threshold optimization -> final model selection -> SHAP interpretation -> feature importance -> business recommendation

with the goal of building a classification workflow that is both statistically sound — grounded in the actual cost structure of the business problem — and practically useful.

## Practical, Business-Oriented by Design

**This is a practical, business-oriented workflow, not a model-scoring exercise.** Every choice made here traces back to a real business question — who should this bank call next, and why — rather than to chasing the best number on a leaderboard:

* **The metric was chosen for the cost of being wrong**, not picked by convention. F2 was selected because a missed subscriber costs the bank a real sale, while a wasted call costs a few minutes — F1 or accuracy would have picked a worse model for this specific business.
* **The winning model was chosen for what it can defend, not just what it scored.** Once several models performed comparably, the deciding factor was which one's assumptions actually held for this data — and which one a bank could explain to a regulator or a client if asked.
* **The threshold reflects a business trade-off, not a default.** It deliberately leans toward calling more people, because the cost of a missed sale outweighs the cost of an extra call.
* **The interpretability findings translate directly into action** — contact method, contact frequency, and past-subscriber targeting are concrete levers a marketing team can use tomorrow, separate from anything the model itself decides.
* **The project is honest about its own limits** — flagging where results are optimistic, where a pilot rollout is needed before full deployment, and where the model will need to be refreshed as conditions change — because a workflow a business can trust has to be honest about what it doesn't yet know.

## References

Moro, S., Cortez, P., & Rita, P. (2014). A data-driven approach to predict the success of bank telemarketing. *Decision Support Systems*, 62, 22-31. https://doi.org/10.1016/j.dss.2014.03.001
