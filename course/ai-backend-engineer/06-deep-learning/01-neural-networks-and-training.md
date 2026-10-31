# Neural networks and deep learning foundations

## Learning outcomes

This chapter builds the mental model needed for PyTorch, computer vision, NLP and transformers. You should understand tensors, layers, activations, logits, loss, gradients, batches, epochs and the difference between fitting parameters and evaluating generalization.

## 1. What makes a neural network a network

A neural network composes simple functions. Each layer transforms its input; nonlinear activations between layers let the total function represent patterns that one linear transformation cannot.

For a dense layer:
z = xW + b
a = activation(z)

Here x is a batch of examples, W is a learned weight matrix, b is a learned bias, z is the pre-activation output and a is the activated output. A network applies several such transformations in sequence.

Without a nonlinear activation, the composition of linear layers is still linear. For example, (xA)B = x(AB). Depth alone would not create a more expressive nonlinear model.

## 2. Tensors, batch shape and dimensions

A tensor is a multidimensional numeric array. A fully connected tabular batch may have shape (batch, features). A batch of RGB images may be represented as (batch, channels, height, width) or another explicitly documented ordering. Token IDs often have shape (batch, sequence_length).

Always name dimensions. “The tensor is size 32×128×64” is incomplete unless you say what each dimension means. A shape mismatch often indicates a conceptual misunderstanding, not just a syntax problem.

A training batch is a subset of examples processed together to use vectorized hardware. Batch size affects memory, gradient noise, throughput and sometimes model behavior. It does not change the number of examples in the dataset.

## 3. A neuron and an activation

A neuron computes a weighted sum plus bias, then applies a function. Common activations:
- **ReLU**: max(0,z), inexpensive and common in hidden layers;
- **sigmoid**: maps a scalar to (0,1), historically common in binary output and gates;
- **tanh**: maps to (-1,1);
- **softmax**: converts a vector of scores into a distribution that sums to one.

Softmax is commonly applied to class logits. It can become numerically unstable if implemented naïvely on large scores; stable implementations subtract the maximum logit. A softmax value is not automatically a calibrated probability.

Hidden-layer design depends on architecture. Output activation and loss must agree: many libraries expect raw logits and compute a stable cross-entropy internally. Applying softmax twice is a common error.

## 4. Logits, prediction and loss

**Logits** are raw model scores before a probability-like transformation. For three classes, logits [2.2, 0.3, -1.1] rank the first class highest. Softmax maps them to normalized scores, but the magnitude of the logits controls confidence.

Training compares predictions with targets using a loss:
- regression often uses MSE or a robust alternative;
- single-label multi-class classification often uses cross-entropy;
- multi-label classification often uses independent sigmoid outputs plus binary cross-entropy;
- language modeling predicts a vocabulary token at each position.

A loss function is a mathematical proxy. The model can optimize that proxy and still fail human needs, so evaluation includes task metrics and error review.

## 5. Forward pass, backward pass, optimizer

1. **Forward pass**: compute predictions from input and current parameters.
2. **Loss calculation**: compare predictions with targets.
3. **Backward pass**: use automatic differentiation/backpropagation to compute parameter gradients.
4. **Optimizer step**: modify parameters using gradients.
5. Repeat over batches and epochs.

An epoch is one pass over the training data; it is not necessarily one optimizer update. With N examples and batch size B, there are roughly ceil(N/B) updates per epoch, subject to dropped or partial batches.

Minimal PyTorch pattern:

~~~python
for inputs, targets in train_loader:
    optimizer.zero_grad()
    logits = model(inputs)
    loss = loss_function(logits, targets)
    loss.backward()
    optimizer.step()
~~~

Real code must also manage device placement, data types, train/eval mode, validation, checkpointing, mixed precision where appropriate, reproducibility and resource use.

## 6. Backpropagation intuition

A computational graph records operations and dependencies. During the forward pass, intermediate values are computed. During the backward pass, the chain rule propagates how much each intermediate and parameter contributed to the final loss.

For z=xw+b and loss L(z), the gradient with respect to w is dL/dw = dL/dz × dz/dw = dL/dz × x. Earlier material on calculus explains why this is repeated backwards through layers. Autograd calculates these derivatives for supported operations; it does not select a correct loss, fix a bad target or verify that gradients correspond to your intended task.

## 7. Optimization and training stability

The learning rate is often the most consequential optimizer setting. Track train and validation loss, gradient norms and learning rate. Exploding activations or gradients can produce instability; vanishing gradients can make early layers learn slowly. Initialization, normalization, residual paths, activation choice and architecture influence these issues.

Common optimizers:
- SGD: update by gradient; momentum often smooths movement.
- Adam: adaptive update based on running gradient statistics.
- AdamW: decoupled weight decay commonly used for deep networks.

An optimizer can make training practical but does not guarantee the global optimum. Neural losses are non-convex and large-scale training is approximate.

## 8. Regularization and generalization

- **Weight decay**: discourages excessively large weights.
- **Dropout**: randomly masks activations during training; disabled or adjusted at inference.
- **Data augmentation**: applies label-preserving input transformations; validity depends on task.
- **Early stopping**: stop when validation evidence worsens or stops improving.
- **Label smoothing**: softens target distributions in some classification settings.
- **More data / better coverage**: often valuable, provided labels and sampling are sound.

A regularizer can reduce one form of overfitting but also hurt if too strong or misapplied. Validation must mirror intended deployment.

## 9. Normalization and residual connections

Batch normalization uses batch-derived statistics during training and stored estimates during inference. Layer normalization normalizes across feature dimensions within an example and is common in sequence models. They are not interchangeable: their axes and behavior under small batches differ.

A residual connection adds an input to a transformed path, often written y=F(x)+x. It gives information and gradients a direct route through deeper networks. Shapes must be compatible; a projection may be used when dimensions differ.

## 10. Train/eval mode and inference

Training mode may enable dropout and update normalization statistics. Evaluation mode disables training-specific behavior. In PyTorch, call **model.train()** for training and **model.eval()** for evaluation/inference, and use inference/no-gradient context to avoid storing unnecessary autograd state.

A frequent production bug is serving in training mode, creating nondeterministic outputs or changed normalization state. Also define input preprocessing, tokenization, model version and output schema as part of the artifact contract.

## 11. Resource intuition

Parameter count is only one part of memory. Training generally needs parameters, gradients, optimizer state, activations and temporary buffers. Inference often needs parameters, activations and runtime overhead. Precision, batch size, sequence length, architecture and backend change the total. A model that fits for inference may not fit for training.

Measure peak memory and throughput on the target hardware. Do not promise that a model fits from parameter count alone. Gradient accumulation can emulate a larger effective batch in some settings but does not make all memory disappear, especially activation memory.

## 12. Debugging a training run

- First test the pipeline on a tiny batch and inspect shapes, dtypes and labels.
- Verify that loss changes when parameters change and gradients are finite.
- Overfit a tiny dataset as a diagnostic. If the model cannot memorize a handful of clean examples, suspect a wiring, target, optimizer or gradient issue. This does not prove the model generalizes.
- Compare train and validation curves.
- Inspect examples, predictions and class balance, not only aggregate scores.
- Save checkpoints with model config, tokenizer/preprocessing version and code/data provenance.

## Questions and answers

**Q: What is the difference between an epoch and a batch?**  
A: A batch is a group processed in one forward/backward pass. An epoch is one pass through the training dataset, typically containing many batches.

**Q: Are softmax outputs guaranteed true probabilities?**  
A: They sum to one and can be interpreted as model probabilities, but may be miscalibrated. Evaluate calibration if probability meaning matters.

**Q: Why use a nonlinear activation?**  
A: Without nonlinearities, multiple dense layers collapse algebraically into one linear transformation and cannot represent general nonlinear relationships.

**Q: What does autograd do?**  
A: It computes derivatives for recorded operations. It does not decide whether your data, loss, split or objective is correct.

**Q: Should I always train a neural network?**  
A: No. Start from a simpler baseline and use a neural network when measured task needs and available data justify its complexity and cost.

## Practice

1. Given batch shape (16, 20), weights (20, 3), bias (3,), compute each layer output shape.
2. Diagram forward and backward paths for a two-layer network.
3. Explain why training and evaluation modes differ when dropout is present.
4. Train a tiny network to overfit a small synthetic dataset, then explain why that is only a wiring check.
5. Compare a neural model to a logistic-regression baseline using the same split and metrics.

## Completion check

You can trace a tensor through layers, explain logits and cross-entropy, outline the training loop, distinguish autograd from optimization, and diagnose a run using curves and a tiny-set overfit check.