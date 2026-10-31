# Linear algebra for AI

## Why it appears everywhere

A dataset can be represented as a matrix, a single example as a vector, model parameters as another vector or matrix, and a neural-network layer as a matrix multiplication plus a bias. Embeddings are vectors; attention compares and combines vectors. The goal is not to memorize abstract proofs first. Learn the operations, shapes, geometry and assumptions well enough to reason about model behavior.

## 1. Scalars, vectors, matrices and tensors

- A **scalar** is one number, such as 0.7.
- A **vector** is an ordered one-dimensional collection, such as [2, 3].
- A **matrix** is a two-dimensional rectangular array.
- A **tensor** is the general name for a multi-dimensional numerical array.

The **shape** records dimension lengths. A matrix with 100 rows and 8 columns has shape (100, 8). One common convention is rows=examples and columns=features, but libraries and model operations may use other conventions. Always inspect and state the convention.

A **feature** is one measured input dimension. An **embedding** is a learned vector representation. The coordinates do not usually have independent human-readable meanings; relationships among vectors are often what matter.

## 2. Vector arithmetic and geometry

For vectors a=[a1,...,an] and b=[b1,...,bn] with equal length:
- addition is coordinatewise;
- scalar multiplication scales every coordinate;
- the dot product is a·b = sum over i of ai bi;
- the Euclidean norm is ||a||₂ = sqrt(sum ai²).

Worked example: a=[1,2], b=[3,4]. Then a·b=1×3+2×4=11. The norm of a is sqrt(1+4)=sqrt(5). The angle relation is a·b=||a|| ||b|| cos(theta). A positive dot product indicates an angle below 90 degrees; zero indicates orthogonality; a negative value indicates an obtuse angle.

The **cosine similarity** is (a·b)/(||a|| ||b||). For the example it is 11/sqrt(5×25), about 0.984. Cosine similarity compares direction and ignores vector magnitude. If either vector is zero, the expression is undefined; a production implementation must define how to handle that case.

The **L1 norm** sums absolute coordinates; the **L2 norm** is Euclidean length. Norms quantify size and appear in regularization, optimization and nearest-neighbor search. Do not confuse a vector norm with a probability.

## 3. Matrices and multiplication

If A has shape (m,n) and B has shape (n,p), then AB is defined and has shape (m,p). The shared inner dimension n must match. Each output value is a row-column dot product.

Worked shape example:
- X: 32 examples × 10 features → (32,10)
- W: 10 features × 4 output units → (10,4)
- XW: (32,4)
- bias b: one value per output unit, shape (4,)
- XW+b: broadcast b over 32 rows → (32,4)

A fully connected neural-network layer is often written Y = XW + b. Libraries may store W transposed, so the code's convention must be checked instead of inferred from a diagram.

~~~python
import numpy as np

X = np.array([[1.0, 2.0], [3.0, 4.0]])       # (2, 2): two rows
W = np.array([[0.5, -1.0], [2.0, 0.25]])    # (2, 2)
b = np.array([0.1, 0.2])                    # (2,)
Y = X @ W + b
print(Y.shape)  # (2, 2)
~~~

NumPy uses **@** for matrix multiplication and asterisk operator for elementwise multiplication. Broadcasting aligns dimensions from the right and permits dimensions of length 1 to expand; it does not mean arbitrary shapes are compatible. Print shapes at each step when debugging.

## 4. Linear transformations and basis

A matrix can represent a linear transformation: it rotates, reflects, scales, projects or combines coordinates. A **basis** is a set of independent vectors that can represent every vector in a space through linear combinations. A coordinate representation depends on the chosen basis.

A linear model computes a weighted sum. The weights describe how input coordinates contribute to output. Adding a nonlinear activation later lets a neural network represent more complex functions; stacking linear layers without nonlinearities remains a single linear transformation (ignoring other operations).

## 5. Projection, rank and dependence

A projection maps a vector onto a subspace. In least-squares regression, the fitted values can be understood as the projection of targets onto the column space of the design matrix. This explains why fitting can be impossible to uniquely identify every coefficient when features are redundant.

- **Linear independence**: no vector in a set can be expressed as a combination of the others.
- **Rank**: number of independent rows or columns.
- **Singular matrix**: no inverse; commonly caused by exact feature dependence or insufficient independent observations.
- **Conditioning**: sensitivity of a numerical solution to small changes in inputs. Ill-conditioned systems can amplify measurement or rounding errors.

Do not use a matrix inverse casually to solve regression or linear systems. Numerical libraries provide solvers based on stable factorizations.

## 6. Eigenvalues and singular values

An eigenvector v of square matrix A satisfies Av=lambda v. The transformation leaves v's direction unchanged and scales it by eigenvalue lambda. Eigen concepts help explain stability, covariance structure and some optimization behavior.

For a general rectangular matrix, singular value decomposition gives A=U Sigma V-transpose. Singular values describe strengths along important orthogonal directions. SVD powers dimensionality reduction methods such as PCA and helps diagnose rank and conditioning. PCA is not simply “drop columns with small variance”; it rotates the coordinate system and preserves directions of greatest variance under its assumptions.

## 7. Common applications

- Tabular ML: X is examples by features.
- Linear/logistic model: scores are XW+b.
- Embedding retrieval: compare query and document vectors by dot product or cosine.
- Attention: compare query and key vectors, then weight value vectors.
- Convolution: local tensor operations over image or signal dimensions.
- PCA: transform correlated features into orthogonal principal directions.
- Optimization: gradient vectors point toward increasing loss; moving against them can reduce loss.

## 8. Errors worth catching

1. **Shape error**: the inner dimensions do not match.
2. **Orientation error**: rows and columns were interpreted differently from the library convention.
3. **Broadcast error**: a bias was aligned along the wrong axis.
4. **Elementwise/matmul confusion**: asterisk operator used where a matrix product was intended.
5. **Scale error**: one feature has a much larger numeric scale and dominates distance or optimization.
6. **Zero-vector similarity**: division by zero.
7. **Silent dtype conversion**: integer arrays truncate fractional results or low precision causes numerical instability.
8. **Interpreting embedding coordinates literally**: semantic meaning is generally distributed, not one coordinate per concept.

## Questions and answers

**Q: What is the intuition behind a dot product?**  
A: It combines coordinate overlap and magnitude. Geometrically it equals the product of lengths times cosine of angle. In a linear model it is a weighted sum.

**Q: Why are embeddings compared by cosine?**  
A: It emphasizes orientation and can reduce the effect of vector magnitude. Dot product may work better when magnitude carries useful signal or when the model was trained with that scoring function. Match the index metric to the model and evaluate it.

**Q: What is the difference between a matrix and a tensor?**  
A: A matrix is two-dimensional. Tensor is the general multidimensional-array term; in ML code even scalars/vectors/matrices may be called tensors.

**Q: Does normalization always improve retrieval?**  
A: No. Normalization changes the metric's meaning. Compare the model's intended scoring function and measure retrieval quality on representative queries.

## Practice

1. Implement dot product and cosine similarity for short lists; handle unequal lengths and zero vectors.
2. Verify the output shape of XW+b for 5 examples, 3 features, and 2 output units.
3. Create two feature columns where one is 1000 times larger; observe what happens to Euclidean distances.
4. Explain why duplicate features can make coefficient estimates non-unique.
5. In NumPy, compare **A * B** with **A @ B** on nontrivial 2×2 arrays.

## Completion check

You can state the shapes and meanings of X, W, b, embeddings and batch dimensions; perform dot-product/cosine calculations by hand; explain why feature scaling, rank and conditioning matter; and debug simple NumPy shape errors.