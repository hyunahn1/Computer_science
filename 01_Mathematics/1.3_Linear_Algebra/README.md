# 5.3 Linear Algebra

*Note: This is a comprehensive overview. For complete mathematical rigor, please refer to dedicated linear algebra textbooks.*

## Vectors & Scalars

### Scalar vs Vector
- **Scalar**: Magnitude only (speed: 50 km/h)
- **Vector**: Magnitude + Direction (velocity: 50 km/h North)

### Vector Operations

#### Addition/Subtraction
- **Geometric**: Triangle rule, Parallelogram rule
- **Physics**: Relative velocity, force composition

#### Dot Product (Inner Product)
```
A · B = |A| |B| cos(θ)
A · B = Ax×Bx + Ay×By + Az×Bz
```

**Meaning**: Projection of A onto B (or vice versa)

**When A · B = 0**: Vectors are **orthogonal** (perpendicular)

**Use Cases**:
- Check if perpendicular
- Calculate angle between vectors
- Determine if ahead/behind (positive/negative)
- Similarity measure (cosine similarity)

#### Cross Product (Outer Product)
```
A × B = |A| |B| sin(θ) n̂
```
where n̂ is normal vector

**Direction**: Right-hand rule
**Magnitude**: Area of parallelogram

**Use Cases**:
- Find normal vector (for planes/surfaces)
- Determine rotation axis
- Check front/back face (culling in graphics)
- Torque calculation

## Matrices

### Matrix Multiplication
- **(M×N) × (N×K) = (M×K)**
- **Meaning**: Composition of linear transformations

### Identity Matrix (I)
```
I = [[1, 0, 0],
     [0, 1, 0],
     [0, 0, 1]]
```
- AI = IA = A

### Inverse Matrix (A⁻¹)
- **AA⁻¹ = I**
- **Exists** if determinant ≠ 0
- **Use**: Solve Ax = b → x = A⁻¹b

### Determinant
**Geometric Meaning**: 
- 2D: Area scaling factor
- 3D: Volume scaling factor
- det = 0: Dimension reduction (singular matrix)

### Transpose (Aᵀ)
- Swap rows and columns
- (AB)ᵀ = BᵀAᵀ
- **Use**: Convert row to column vector, dot product as matrix form

## Linear Transformations

### Types
- **Rotation**: Preserve angles and lengths
- **Scaling**: Change size
- **Shear**: Slant/skew
- **Reflection**: Mirror

### Affine Transformation
- Linear transformation + Translation
- **Why 4×4 matrices**: Homogeneous coordinates allow translation as matrix multiplication

```
[x']   [a b c tx] [x]
[y'] = [d e f ty] [y]
[z']   [g h i tz] [z]
[1 ]   [0 0 0 1 ] [1]
```

## Eigenvalues & Eigenvectors

### Definition
**Av = λv**
- v: Eigenvector (direction unchanged by transformation)
- λ: Eigenvalue (scaling factor)

### Applications
- Principal Component Analysis (PCA)
- Vibration analysis
- Google PageRank
- Stability analysis (control systems)

## Vector Spaces

### Basis & Dimension
- **Basis**: Set of linearly independent vectors spanning space
- **Dimension**: Number of basis vectors
- **Standard basis** (3D): î = [1,0,0], ĵ = [0,1,0], k̂ = [0,0,1]

### Rank
- Number of linearly independent rows/columns
- **Full rank**: rank = min(rows, cols)
- **Use**: Determine if system Ax=b has solution

## Vector Operations

### Normalization
```
v̂ = v / |v|
```
- Unit vector (length = 1)
- Keep direction, normalize magnitude

### Projection
```
proj_b(a) = (a · b̂) b̂
```
- Shadow of a onto b
- Component of a in direction of b

## 2D Rotation Matrix
```
R(θ) = [[cos(θ), -sin(θ)],
        [sin(θ),  cos(θ)]]
```

Rotates vector counterclockwise by angle θ

## 3D Rotations & Quaternions

### Gimbal Lock Problem
- Using Euler angles (roll, pitch, yaw)
- Lose degree of freedom in certain orientations
- Causes singularities

### Quaternion Solution
- **4D complex numbers**: q = w + xi + yj + zk
- **Advantages**:
  - No gimbal lock
  - Smooth interpolation (Slerp)
  - More efficient than matrices
  - Used in game engines, robotics

## Interview Practice
- [ ] Calculate dot product and interpret meaning
- [ ] Find cross product and normal vector
- [ ] Multiply matrices and explain as transformation composition
- [ ] Determine if vectors are orthogonal
- [ ] Compute determinant and interpret geometrically
- [ ] Normalize vector
- [ ] Project vector onto another
- [ ] Explain why 4×4 matrices for 3D graphics
- [ ] Describe gimbal lock and quaternion solution
- [ ] Find eigenvalues and eigenvectors
