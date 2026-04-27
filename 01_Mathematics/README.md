# Mathematics for Computer Science

## Overview
Mathematical foundations essential for computer science, from discrete mathematics and cryptography to linear algebra, calculus, and probability. These concepts underpin algorithms, graphics, robotics, machine learning, and systems engineering.

**Philosophy**: Mathematics is a tool for modeling and optimizing real-world problems. Even non-math backgrounds can master these concepts with practice.

## Course Structure

### [1.1 Discrete Mathematics & Logic](./1.1_Discrete_Math_and_Logic/)
- Asymptotic growth intuition (ties to **2.1** Big-O; formal **P / NP / NP-completeness** in **[4.2](../04_Theory_of_Computer_Science/4.2_Computability_and_Complexity/)**)
- Propositions and logical operators
- De Morgan's Laws and truth tables
- Tautology and contradiction
- Conditional statements and contrapositive
- Mathematical induction
- Pigeonhole principle
- Set theory and relations
- Functions (injective, surjective, bijective)
- Recursive definitions
- Boolean algebra and circuit simplification
- Graph theory (Euler path, Hamilton path, planar graphs)
- Combinatorics (permutation vs combination)
- Binomial theorem and Pascal's triangle

### [1.2 Number Theory & Cryptography](./1.2_Number_Theory_and_Cryptography/)
- Prime numbers and factorization
- GCD/LCM and Euclidean algorithm
- Modular arithmetic and congruence
- Fermat's Little Theorem
- Euler's totient function
- Chinese Remainder Theorem
- RSA encryption principles
- Hash functions
- Pseudo-random number generators (PRNG, LCG)
- Two's complement
- Parity bit and CRC
- Finite fields (Galois fields)
- Fast exponentiation
- Gray code

### [1.3 Linear Algebra](./1.3_Linear_Algebra/)
- Scalars vs vectors
- Dot product (projection, orthogonality)
- Cross product (normal vectors)
- Matrices and matrix multiplication
- Identity and inverse matrices
- Determinant (geometric meaning)
- Transpose matrix
- Linear transformations (rotation, scaling, shear)
- Affine transformations (4×4 matrices)
- Eigenvalues and eigenvectors
- Basis, dimension, and rank
- Normalization and projection
- 2D rotation matrix
- Quaternions (solving gimbal lock)

### [1.4 Calculus & Numerical Methods](./1.4_Calculus_and_Numerical_Methods/)
- Limits and derivatives
- Integration (area under curve)
- Chain rule (backpropagation)
- Partial derivatives and gradient
- Taylor series approximation
- Newton-Raphson method
- Floating-point arithmetic issues
- Fourier Transform and FFT
- Laplace Transform
- PID controller (P, I, D components)
- Numerical differentiation/integration
- Interpolation (linear, spline)
- Sigmoid and activation functions
- Convolution
- Linear regression and gradient descent

### [1.5 Probability & Statistics](./1.5_Probability_and_Statistics/)
- Probability vs statistics
- Conditional probability
- Bayes' Theorem
- Independent events
- Expectation, variance, standard deviation
- Normal distribution (Gaussian, 68-95-99.7 rule)
- Bernoulli and binomial distributions
- Poisson distribution
- Central Limit Theorem
- Covariance and correlation
- Covariance matrix
- Monte Carlo simulation
- Markov chains
- Entropy (information theory)
- Likelihood vs probability
- Maximum Likelihood Estimation (MLE)
- Kalman filter basics
- A/B testing and p-values
- Confusion matrix (TP, FP, FN, TN)
- Precision, recall, F1 score

## Study Approach

### For Non-Math Backgrounds
1. **Don't memorize formulas** - understand concepts
2. **Focus on applications** - how is it used in CS?
3. **Start with discrete math** - most immediately useful
4. **Build gradually** - from basics to advanced

### Practical Applications by Field

#### Graphics & Game Development
- Linear algebra (vectors, matrices, quaternions)
- Calculus (physics simulation, interpolation)

#### Robotics & Autonomous Vehicles
- **Critical**: Linear algebra (transformations)
- **Critical**: Probability (Kalman filter, sensor fusion)
- Calculus (PID control, kinematics)

#### Machine Learning & AI
- Linear algebra (matrices, eigenvectors)
- Calculus (gradient descent, backpropagation)
- Probability (Bayesian inference, distributions)

#### Cryptography & Security
- Number theory (RSA, modular arithmetic)
- Discrete math (Boolean logic, set theory)

#### Signal Processing
- Calculus (Fourier transform, convolution)
- Probability (noise modeling)

## Key Concepts by Importance

### Fundamental (Must Know)
- Boolean logic and truth tables
- Set theory and functions
- Modular arithmetic
- Vectors and dot product
- Matrices and transformations
- Probability distributions
- Expected value and variance

### Important (Should Know)
- Mathematical induction
- Graph theory basics
- Prime factorization and GCD
- Cross product and normal vectors
- Eigenvalues/eigenvectors
- Bayes' theorem
- Gradient descent

### Advanced (Nice to Know)
- Euler's totient function
- Quaternions
- Laplace transform
- Kalman filter mathematics
- Information theory entropy

## Interview Preparation

### Common Questions
- Explain Bayes' theorem with example
- What is dot product vs cross product?
- How does RSA encryption work?
- Describe gradient descent algorithm
- What is eigenvalue/eigenvector?
- Explain Kalman filter conceptually
- Apply mathematical induction
- Calculate combinations and permutations

### Problem-Solving Tips
- Draw diagrams (vectors, graphs)
- Work through small examples
- Connect to real-world applications
- Explain intuition before formulas

## Recommended Resources
- **Discrete Math**: "Concrete Mathematics" by Knuth
- **Linear Algebra**: 3Blue1Brown YouTube series
- **Probability**: "Think Stats" by Allen Downey
- **All**: Khan Academy, MIT OpenCourseWare

---

*Remember: Mathematics is a tool, not an obstacle. Focus on understanding concepts and their applications in solving real problems.*

## Advanced Topics to Add

- Discrete math: proof by contradiction, strong induction, invariants, recurrence solving, generating functions.
- Number theory: modular inverses, primitive roots, finite fields, elliptic curves, cryptographic assumptions.
- Linear algebra: LU/QR/SVD, conditioning, orthogonality, spectral theorem, PCA derivation.
- Calculus/numerics: convexity, constrained optimization, numerical stability, error bounds, gradient methods.
- Probability/statistics: concentration bounds, Bayesian inference, hypothesis testing rigor, estimation bias/variance.

## Expert Depth Checklist
- [ ] State formal definitions precisely and list the assumptions under which they hold.
- [ ] Prove at least one nontrivial theorem, identity, or algorithmic property from this chapter.
- [ ] Work through concrete examples by hand before using software or calculators.
- [ ] Connect the concept to a CS application: algorithms, cryptography, graphics, ML, control, or systems.
- [ ] Identify a common misconception and provide a counterexample.
- [ ] Implement or simulate one small example when the topic is computational.
- [ ] Cite a textbook-quality source and distinguish intuition from formal proof.
