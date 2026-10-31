# Calculus and optimization for AI

## Why this matters

Training a model means choosing parameter values that make a loss function small. Calculus tells us how the loss changes when parameters change. Optimization uses that information to choose updates. You need the operational intuition and enough notation to trace gradients through a model; advanced proofs can come later.

## 1. Functions and derivatives

A function maps input to output. A derivative measures the local rate of change. For a one-variable function f(x), f'(x) is the slope of the tangent at x. If f'(x)>0, a small increase in x increases f locally; if f'(x)<0, it decreases f locally.

For f(x)=x², f'(x)=2x. At x=3 the slope is 6. A small step from x=3 to 2.9 changes the function from 9 to 8.41, approximately a decrease of 0.6 predicted by slope×change (-0.6), with a small second-order difference.

A derivative is a local approximation, not a guarantee about a large step. Non-smooth functions such as ReLU have a kink; practical optimizers use a subgradient convention at such points.

## 2. Partial derivatives and gradients

For f(x1,...,xn), the partial derivative with respect to xi changes only xi while holding other coordinates fixed. The **gradient** is the vector of partial derivatives:
grad f = [df/dx1, ..., df/dxn].
The gradient points in the direction of steepest local increase under Euclidean geometry. Its negative points toward local decrease.

For f(w1,w2)=w1²+3w2², gradient=[2w1,6w2]. At [1,1], gradient=[2,6]. The second coordinate has a steeper effect, so a circular step can zig-zag through the narrow valley unless the learning rate is controlled.

## 3. Gradient descent

Given loss L(theta), gradient descent updates parameters:
theta_next = theta - learning_rate × grad L(theta).

The learning rate controls step size. Too large: the updates overshoot or diverge. Too small: learning is slow. A practical training log should include loss, relevant validation metrics, learning rate, and signs of instability such as NaN or exploding gradient norm.

A one-parameter example: L(w)=(w-3)². Its derivative is 2(w-3). Starting w=0 with learning rate 0.1:
- gradient = -6;
- new w = 0 - 0.1×(-6) = 0.6;
- next gradient = -4.8 and w=1.08.
The parameter moves toward 3, the minimum. If learning rate were 1.1, the first update would jump to 6.6 and then alternate farther or nearer depending on the quadratic curvature.

## 4. Chain rule and backpropagation

The chain rule says that for a composition y=f(g(x)), dy/dx = (df/dg)(dg/dx). Neural networks are compositions of layers, so gradients are propagated from output back toward earlier parameters. This efficient application of the chain rule is called **backpropagation**.

Small example:
- z = wx + b;
- a = sigmoid(z);
- loss for target y is L = -(y log a + (1-y)log(1-a)).
To update w, chain together dL/da × da/dz × dz/dw. Since dz/dw=x, input magnitude directly scales this parameter's gradient. Large input scales can create large gradients; normalized features often improve optimization.

Backpropagation computes derivatives; the optimizer chooses parameter updates. They are related but not the same process.

## 5. Loss functions

A loss converts a prediction and target into a numeric penalty:
- mean squared error is common for regression but can heavily punish outliers;
- absolute error is more robust but has a kink at zero;
- cross-entropy is common for classification probabilities;
- ranking/contrastive losses compare relative or embedding relationships;
- token-level language-model loss is commonly next-token cross-entropy.

Training loss is an optimization target, not the entire product objective. A model can minimize a proxy loss while being unsafe, unfair, poorly calibrated or operationally expensive. Validation metrics test a broader set of desired behaviors.

## 6. Batch, stochastic and mini-batch updates

- **Batch gradient descent** uses the full training set per update: stable estimate but potentially expensive.
- **Stochastic gradient descent** uses one example per update: noisy and cheap updates.
- **Mini-batch training** uses a small batch and is common on accelerators.

A batch is a computational grouping, not necessarily independent data. Correlated examples and poorly constructed batches can distort learning.

## 7. Momentum, Adam and regularization

Momentum accumulates a smoothed direction from recent gradients, reducing oscillation in some settings. Adam adapts update scales using running estimates of gradient means and squared values. AdamW decouples weight decay from the gradient-based update in a way often preferred for neural networks. Optimizer choice does not compensate for bad data, leakage, poor labels or an unsuitable model.

**L2 regularization** penalizes large parameter magnitudes; **weight decay** is a related parameter-shrinkage mechanism, though equivalence depends on optimizer details. Early stopping uses validation behavior to stop before the model overfits. Dropout randomly disables activations during training as a regularizer; inference behavior differs.

## 8. Numerical gradient check

Finite differences approximate a derivative:
f'(x) ≈ [f(x+epsilon)-f(x-epsilon)]/(2epsilon).

~~~python
def loss(w):
    return (w - 3.0) ** 2

def analytic_gradient(w):
    return 2.0 * (w - 3.0)

w = 0.8
epsilon = 1e-5
numerical = (loss(w + epsilon) - loss(w - epsilon)) / (2 * epsilon)
print(numerical, analytic_gradient(w))
~~~

This is useful for validating a small custom implementation. It is slow for large parameter sets and susceptible to floating point error if epsilon is poorly chosen. Deep-learning libraries implement automatic differentiation to construct and apply the chain rule efficiently.

## 9. Diagnosing training curves

- Training and validation losses both high: model may be underfit, data/features poor, or optimization failing.
- Training loss falls while validation loss rises: likely overfitting or a split/distribution issue.
- Loss is NaN: inspect learning rate, invalid values, logarithms, precision, and data.
- Loss barely changes: inspect gradient flow, labels, optimizer, scale, learning rate and whether parameters are actually trainable.
- Excellent validation score with suspiciously easy training: inspect leakage, duplicates and evaluation code.

These are hypotheses to investigate, not automatic diagnoses. Plot curves and inspect representative errors.

## Questions and answers

**Q: Is the gradient the same as the loss?**  
A: No. Loss is a scalar measure of error. Gradient is the vector of local partial derivatives describing how each parameter affects that loss.

**Q: Why subtract the gradient?**  
A: The gradient points toward increasing loss locally. Moving a small amount in the opposite direction tends to reduce it.

**Q: Does backpropagation train a network by itself?**  
A: Backpropagation computes gradients. An optimizer applies a rule to those gradients to update parameters.

**Q: Why can a model still fail if training loss is low?**  
A: It may memorize training data, exploit leakage, miss real deployment cases, be badly calibrated, or optimize a proxy unrelated to the desired outcome.

## Practice

1. Compute the gradient and first two updates for L(w)=(w-5)² from w=0 at learning rate 0.1.
2. Draw a two-layer computation graph and label the chain-rule factors.
3. Compare finite-difference and analytic derivatives for a simple quadratic.
4. Sketch training/validation curves for underfit, healthy fit and overfit.
5. Explain why feature scaling affects gradient descent even if a linear model can represent the same predictions.

## Completion check

You can describe a derivative, gradient, learning rate, loss, chain rule and backpropagation; perform a gradient-descent update by hand; and use training curves as evidence rather than guessing.