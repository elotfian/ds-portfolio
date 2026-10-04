# Data Science / ML Engineering Portfolio - Elaheh Lotfian

PhD in Applied Statistics, with a strong background in statistical modeling, hypothesis testing, and applied research. This repo applies that foundation to production-oriented Data Science / ML Engineering work.

Running a model is the easy part. The harder, more valuable skill is knowing what a metric is actually hiding, why one method was chosen instead of another, and whether the result can be trusted. Each project below documents that reasoning alongside the code, including the statistical thinking behind every method choice, evaluation metric, and tradeoff made along the way.

These notebooks are meant to be read and reused as references.

## Projects

| Project | Description | Techniques |
|---|---|---|
| [California Housing - OLS Regression](ols_project.ipynb) | Predicting median house values using linear regression, implemented from scratch and validated against scikit-learn. Includes residual diagnostics that identified a target-capping artifact and leverage-point outliers in the dataset. | NumPy, OLS from scratch, sklearn, residual analysis, correlation/multicollinearity checks |
| [Leak-Safe Classification Pipeline](leak_safe_classification_pipeline.ipynb) | Built a full scikit-learn Pipeline + ColumnTransformer for a classification task, so every preprocessing step is refit fresh inside each cross-validation fold, rather than fit once on the full dataset before evaluation. Evaluated with stratified cross-validation instead of a single split, then tuned via a regularization sweep chosen by F1 score rather than accuracy alone. | scikit-learn Pipeline/ColumnTransformer, cross-validation, regularization (L2), precision/recall/F1 tradeoff analysis |
| [Tree Ensemble Comparison](tree_ensemble_comparison.ipynb) | Built and compared 8 tree-based classification methods on the same leak-safe pipeline: a single tree, Bagging, Random Forest, AdaBoost, and Gradient Boosting (tuned and untuned), plus XGBoost as an industry-standard alternative. Jointly tuned hyperparameters via GridSearchCV rather than one-at-a-time sweeps, and picked a final model based on which metric matters for the problem rather than the single highest score. | scikit-learn ensembles (Bagging, Random Forest, AdaBoost, Gradient Boosting), XGBoost, GridSearchCV, precision/recall/F1 tradeoff analysis |
| [Bank Marketing Classifier Comparison](bank_marketing_classifier/bank_marketing_classifier_comparison.ipynb) | Built a cost-sensitive classification workflow to predict term deposit subscription, comparing 8 model configurations (including class-weighted variants) on a real, class-imbalanced dataset. Selected F2 over F1/accuracy based on the actual cost asymmetry of the business problem, then found that default-threshold rankings were misleading — proper threshold optimization overturned the apparent winner and left several models statistically indistinguishable. Picked the final model on assumption-fit and interpretability grounds rather than top score, then used SHAP to translate the model into concrete, actionable business recommendations. | scikit-learn (Logistic Regression, LDA, QDA, Naive Bayes, SVM), XGBoost, class-imbalance handling (class weighting vs. threshold optimization), SHAP interpretability, cost-sensitive metric |
| [Customer Segmentation](customer_segmentation/capstone_c.ipynb) | Segmented 200 mall customers into five groups using K-means on age, income, and spending score. PCA was checked and ruled out, since no pair of features correlated above 0.33 in absolute value. When the elbow method and silhouette score disagreed on K, both candidate solutions were plotted and K=5 was chosen, because the extra cluster in K=6 split one coherent group in two. Gender was kept out of the clustering and checked afterward, and no segment skewed by it. Each segment was profiled, given a plain-language name, and matched to a specific marketing action based on its own profile. | scikit-learn (KMeans, StandardScaler), elbow method, silhouette score, PCA (evaluated and rejected), unsupervised learning, business segmentation |

## Background

- PhD, Applied Statistics
- Strong in statistics, data science, machine learning, and optimization
- Building practical software engineering skills: Python (scikit-learn, PyTorch), SQL, MLOps (Docker, FastAPI, MLflow)

## Contact

- Email: e.lotfian@gmail.com
- LinkedIn: [linkedin.com/in/elaheh-lotfian-b51690b5](https://www.linkedin.com/in/elaheh-lotfian-b51690b5)
