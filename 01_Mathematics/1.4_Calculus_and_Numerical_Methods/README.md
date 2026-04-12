# 5.4 Calculus & Numerical Methods

*Focus on practical applications in computer science and engineering*

## Limits & Continuity

### Limit Concept
- Value function approaches as input approaches point
- **Epsilon-delta definition** (formal)
- **Foundation** of calculus

## Derivatives

### Geometric Meaning
- **Slope of tangent line** at point
- Instantaneous rate of change

### Physical Meaning
- Position → Velocity → Acceleration
- `v(t) = dx/dt`, `a(t) = dv/dt = d²x/dt²`

### Chain Rule (Critical for Deep Learning)
```
d/dx[f(g(x))] = f'(g(x)) × g'(x)
```
**Backpropagation** in neural networks uses chain rule repeatedly

## Integration

### Meaning
- **Area under curve**
- **Accumulation** (velocity → distance)
- Reverse operation of differentiation

### Fundamental Theorem of Calculus
```
∫[a to b] f'(x)dx = f(b) - f(a)
```

## Multivariable Calculus

### Partial Derivative
- Differentiate with respect to one variable
- Treat others as constants
- **Notation**: ∂f/∂x

### Gradient Vector
```
∇f = [∂f/∂x, ∂f/∂y, ∂f/∂z]
```
- **Points** in direction of steepest increase
- **Magnitude**: rate of increase

## Taylor Series

### Approximation
```
f(x) ≈ f(a) + f'(a)(x-a) + f''(a)(x-a)²/2! + ...
```

### Computer Implementation
- `sin(x)`, `cos(x)`, `exp(x)` computed via Taylor series
- Polynomial approximation of complex functions

## Numerical Methods

### Newton-Raphson Method
**Find root** of f(x) = 0

```
x_{n+1} = x_n - f(x_n)/f'(x_n)
```

**Applications**:
- Square root calculation
- Optimization
- Solving nonlinear equations

### Floating-Point Arithmetic

#### Issues
- **0.1 + 0.2 ≠ 0.3** in binary representation
- **Epsilon**: Small number for comparisons
- Never use `==` for floats, use `|a - b| < epsilon`

## Signal Processing

### Fourier Transform
**Time domain ↔ Frequency domain**

- Decompose signal into sum of sine/cosine waves
- **Applications**:
  - Audio processing (MP3, noise removal)
  - Image compression (JPEG)
  - Communication systems

### FFT (Fast Fourier Transform)
- **Complexity**: O(N log N) vs O(N²) for DFT
- **Critical** for real-time signal processing

### Laplace Transform
- **Converts** differential equations → algebraic equations
- **Used in**: Control systems, circuit analysis
- **s-domain** analysis

## Control Theory

### PID Controller
**P + I + D** = Control Output

1. **P (Proportional)**: Current error × Kp
   - React to present
   
2. **I (Integral)**: ∫ error × Ki
   - Eliminate accumulated past error
   
3. **D (Derivative)**: d(error)/dt × Kd
   - Predict future based on rate of change

**Use Cases**:
- Temperature control
- Motor speed control
- Autonomous vehicle steering

## Numerical Differentiation & Integration

### Numerical Derivative
```
f'(x) ≈ (f(x+h) - f(x)) / h
```
- For discrete data (sensor readings)

### Numerical Integration
- **Trapezoidal Rule**: Approximate area with trapezoids
- **Simpson's Rule**: Use parabolas for better accuracy

## Interpolation

### Linear Interpolation
```
y = y₁ + (y₂ - y₁) × (x - x₁)/(x₂ - x₁)
```

### Spline Interpolation
- Smooth curve through points
- **Used in**: Animation, path planning

## Activation Functions

### Sigmoid (Logistic Function)
```
σ(x) = 1 / (1 + e^(-x))
```
- Maps (-∞, ∞) to (0, 1)
- **S-curve**
- Probability interpretation

### Other Functions
- ReLU: max(0, x)
- Tanh: Hyperbolic tangent
- Softmax: For multi-class classification

## Convolution

### Definition
```
(f * g)(t) = ∫ f(τ)g(t-τ)dτ
```

### Applications
- **Image filtering** (blur, sharpen, edge detection)
- **CNN** (Convolutional Neural Networks)
- **Signal processing** (FIR filters)

## Optimization

### Linear Regression
**Least Squares Method**
- Find line that minimizes sum of squared residuals
- Analytical solution via calculus

### Gradient Descent
```
θ := θ - α∇J(θ)
```
- Iteratively move towards minimum of loss function J
- α: Learning rate
- **Used in**: Training machine learning models

## Interview Practice
- [ ] Explain derivative as rate of change
- [ ] Apply chain rule for nested functions
- [ ] Describe gradient descent algorithm
- [ ] Implement Newton-Raphson method
- [ ] Explain PID controller components
- [ ] Describe Fourier transform applications
- [ ] Discuss floating-point precision issues
- [ ] Explain convolution in CNN context
- [ ] Derive linear regression closed-form solution
- [ ] Calculate numerical derivative/integral
