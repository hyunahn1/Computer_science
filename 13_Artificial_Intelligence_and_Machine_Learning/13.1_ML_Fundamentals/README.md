# 13.1 ML Fundamentals

## Scope
Machine learning is statistical modeling plus engineering discipline. Do not treat models as black boxes: study assumptions, optimization, evaluation, and failure modes.

## Core Concepts
- Supervised, unsupervised, and self-supervised learning
- Training, validation, and test splits
- Bias-variance trade-off
- Loss functions and empirical risk minimization
- Linear regression and logistic regression
- Decision trees, random forests, gradient boosting
- SVM, k-means, PCA
- Feature engineering and leakage
- Regularization: L1, L2, early stopping
- Cross-validation and hyperparameter search

## Evaluation
- Accuracy, precision, recall, F1
- ROC/AUC and PR curves
- Calibration and threshold selection
- Confusion matrix analysis
- Class imbalance and cost-sensitive evaluation
- Offline metrics vs product/business metrics

## Expert Depth Checklist
- [ ] Derive ordinary least squares for linear regression
- [ ] Derive logistic regression gradient for binary classification
- [ ] Explain maximum likelihood estimation for common models
- [ ] Implement train/validation/test splitting without leakage
- [ ] Compare high-bias and high-variance failure cases on a plotted learning curve
- [ ] Train a baseline model and justify why it is a fair baseline
- [ ] Explain why accuracy can be misleading under class imbalance
- [ ] Calibrate probabilities and choose a threshold based on a cost matrix
- [ ] Reproduce one model result with a fixed seed and documented environment

## Practice Problems
- [ ] Build a classifier, report confusion matrix, PR curve, ROC curve, and calibration
- [ ] Diagnose overfitting and propose regularization or data changes
- [ ] Explain when a simple linear model is preferable to a larger model

## Primary Sources
- [ ] The Elements of Statistical Learning, selected chapters
- [ ] Pattern Recognition and Machine Learning, selected chapters
- [ ] scikit-learn user guide for practical model behavior
