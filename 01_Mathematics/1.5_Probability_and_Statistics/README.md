# 5.5 Probability & Statistics

*Essential for sensor fusion, data analysis, machine learning*

## Probability Fundamentals

### Probability vs Statistics
- **Probability**: Model → Predict data
- **Statistics**: Data → Infer model

### Conditional Probability
```
P(A|B) = P(A ∩ B) / P(B)
```
Probability of A given B occurred

### Bayes' Theorem
```
P(A|B) = P(B|A) × P(A) / P(B)
```

**Components**:
- P(A|B): Posterior probability
- P(B|A): Likelihood
- P(A): Prior probability
- P(B): Evidence

**Applications**:
- Spam filtering
- Medical diagnosis
- Sensor fusion (Kalman filter core)

### Independent Events
```
P(A ∩ B) = P(A) × P(B)
```
Events don't influence each other

## Random Variables

### Expectation (Expected Value)
```
E[X] = Σ x × P(x)   (discrete)
E[X] = ∫ x × f(x)dx  (continuous)
```
**Average** value over many trials

### Variance
```
Var(X) = E[(X - μ)²]
```
**Spread** of distribution

### Standard Deviation
```
σ = √Var(X)
```
**Root-mean-square deviation** from mean

## Probability Distributions

### Normal Distribution (Gaussian)
```
f(x) = (1/√(2πσ²)) × exp(-(x-μ)²/(2σ²))
```

**Bell curve**
- μ: Mean (center)
- σ: Standard deviation (width)

**68-95-99.7 Rule**:
- 68% within 1σ
- 95% within 2σ
- 99.7% within 3σ

**Used for**: Natural noise, measurement errors

### Bernoulli & Binomial

#### Bernoulli Trial
- Two outcomes: Success (p), Failure (1-p)
- Single experiment

#### Binomial Distribution
- n Bernoulli trials
- Count of successes
- P(k successes) = C(n,k) × p^k × (1-p)^(n-k)

### Poisson Distribution
- **Events per unit time/space**
- λ: Average rate
- P(k events) = (λ^k × e^(-λ)) / k!

**Examples**:
- Network packet arrivals
- Customer arrivals at store
- Radioactive decay

## Statistical Concepts

### Central Limit Theorem
**Sample means** → Normal distribution (as n increases)

**Implication**: Why normal distribution is ubiquitous

### Covariance
```
Cov(X, Y) = E[(X - μx)(Y - μy)]
```
**Measure** of how two variables change together

### Correlation Coefficient
```
ρ = Cov(X, Y) / (σx × σy)
```
- Range: [-1, 1]
- +1: Perfect positive correlation
- 0: No correlation
- -1: Perfect negative correlation

### Covariance Matrix
**Multivariate distribution**
```
Σ = [[Var(X₁),  Cov(X₁,X₂)],
     [Cov(X₂,X₁), Var(X₂)]]
```

**Used in**: Kalman filter, PCA, multivariate Gaussian

## Simulation & Estimation

### Monte Carlo Simulation
- **Random sampling** to approximate solutions
- Estimate π by throwing darts at circle
- Risk analysis, physics simulation

### Markov Chain
**Memoryless property**: Future depends only on present, not past

**State transition** matrix
- Weather prediction
- Text generation
- PageRank algorithm

## Information Theory

### Entropy
```
H(X) = -Σ P(x) log₂ P(x)
```

**Measure** of uncertainty/information content
- High entropy: Unpredictable, more information
- Low entropy: Predictable, less information

**Applications**:
- Data compression lower bound
- Decision tree splitting (ID3, C4.5)

### Likelihood vs Probability
- **Probability**: P(data | parameters)
  - Given model, predict data
  
- **Likelihood**: L(parameters | data)
  - Given data, how likely are parameters?

### Maximum Likelihood Estimation (MLE)
- **Find parameters** that maximize likelihood of observed data
- **Method**: Set derivative of log-likelihood to zero

## Kalman Filter

### Basic Idea
**Predict** + **Update**

1. **Prediction**: Use model to predict next state
2. **Update**: Combine prediction with measurement
3. **Kalman Gain**: Weighting factor between prediction and measurement

**Applications**:
- GPS/IMU sensor fusion
- Object tracking
- Robotics state estimation
- Autonomous vehicles

**Key**: Assumes Gaussian noise

## Hypothesis Testing

### A/B Testing
- Compare two versions
- **Null hypothesis**: No difference
- **p-value**: Probability of observing data if null is true
- **Significance level**: α = 0.05 (common threshold)

**If p < α**: Reject null, difference is statistically significant

## Machine Learning Metrics

### Confusion Matrix
```
                Predicted
              Positive | Negative
Actual  Pos     TP    |    FN
        Neg     FP    |    TN
```

### Precision & Recall
```
Precision = TP / (TP + FP)  (How many selected are relevant?)
Recall = TP / (TP + FN)     (How many relevant are selected?)
```

### F1 Score
```
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```
Harmonic mean of precision and recall

## Interview Practice
- [ ] Apply Bayes' theorem to real problem
- [ ] Calculate conditional probability
- [ ] Explain central limit theorem importance
- [ ] Compute expectation and variance
- [ ] Interpret covariance and correlation
- [ ] Describe Kalman filter conceptually
- [ ] Explain MLE vs MAP estimation
- [ ] Calculate precision, recall, F1 score
- [ ] Describe Monte Carlo method
- [ ] Explain entropy in information theory
