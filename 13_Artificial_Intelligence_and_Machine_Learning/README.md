# Artificial Intelligence & Machine Learning

## Overview
Foundational and advanced topics in AI, Machine Learning, and Deep Learning. Connecting mathematical foundations from **01_Mathematics** to modern intelligent systems. Includes Transformer architectures and large language models.

## Course Structure

### [13.1 ML Fundamentals](./13.1_ML_Fundamentals/)
- Supervised vs Unsupervised Learning
- Linear Regression and Logistic Regression
- Decision Trees and Random Forests
- SVM, K-Means Clustering, PCA
- Cross-Validation, Bias-Variance Tradeoff
- Evaluation Metrics (Accuracy, Precision, Recall, F1, ROC/AUC)

### [13.2 Deep Learning](./13.2_Deep_Learning/)
- Artificial Neural Networks (MLP, Activation Functions)
- Backpropagation and Gradient Descent
- Convolutional Neural Networks (CNNs: ResNet, VGG)
- Recurrent Neural Networks (RNNs, LSTMs, GRUs)
- Optimizers (Adam, RMSprop) and Regularization (Dropout, Batch Normalization)

### [13.3 LLMs & Transformers](./13.3_LLMs_and_Transformers/)
- Transformer Architecture (Self-Attention mechanism)
- Large Language Models (GPT, BERT)
- Prompt Engineering and Fine-Tuning
- RAG (Retrieval-Augmented Generation) and Vector Databases
- MLOps and Model Deployment Basics

## Study Approach
1. Connect mathematical concepts (Linear Algebra, Calculus) into real training pipelines
2. Understand what neural network layers are actually doing
3. Read papers or guides on the Attention mechanism (Transformer)

## Interview Preparation
- Understand the difference between ML and Deep Learning
- Explain backpropagation to a software engineer
- Trade-offs between embedding vs fine-tuning for LLMs

## Advanced Topics to Add

- ML theory: empirical risk minimization, bias/variance, regularization, calibration, uncertainty.
- Optimization: gradient descent variants, initialization, normalization, loss landscapes, numerical stability.
- Deep learning: CNN/RNN/Transformer mechanics, attention derivation, scaling laws awareness.
- Data: leakage, sampling bias, labeling quality, train/validation/test discipline, dataset drift.
- LLM systems: RAG evaluation, embeddings, reranking, prompt injection, hallucination, latency/cost trade-offs.
- MLOps: experiment tracking, reproducibility, model monitoring, rollback, safety evaluation.

## Expert Depth Checklist
- [ ] State the statistical assumption, objective function, data distribution, and evaluation target.
- [ ] Derive or explain the core optimization step rather than treating the model as a black box.
- [ ] Build a baseline, run an experiment, and report reproducible metrics with a fixed split/seed when possible.
- [ ] Analyze failure modes: leakage, overfitting, underfitting, class imbalance, hallucination, drift, bias, or unsafe output.
- [ ] Compare model quality, latency, cost, interpretability, and operational risk.
- [ ] Read at least one primary paper, textbook chapter, or official framework documentation for the method.
