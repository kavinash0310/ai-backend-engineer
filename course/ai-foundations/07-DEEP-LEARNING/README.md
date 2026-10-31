# Deep Learning

Study neural networks as parameterized functions, not as a framework recipe.

## Sequence

Artificial neuron and perceptron → multilayer perceptron → activations/logits/loss → forward pass → chain rule/backpropagation → initialization/optimizers/regularization → CNNs and sequence models → attention → transformers → PyTorch training/debugging.

The key bridge is tensor shape and gradient flow: explain every layer's input/output shape, learnable parameters, loss contribution and gradient path. Use the official [PyTorch Learn the Basics](https://docs.pytorch.org/tutorials/beginner/basics/intro.html) after understanding vectors and derivatives. Larger-scale model training is a distinct distributed-compute discipline.


## Detailed lessons

- [Neural networks and training](01-neural-networks-and-training.md) — tensors, layers, logits, losses, autograd, optimization, regularization and debugging.
- [CNNs and sequence models](02-cnn-and-sequence-models.md) — convolution, receptive fields, RNN/LSTM/GRU and architecture trade-offs.
- [Transformer attention](../08-GENERATIVE-AI-AND-LLMS/01-transformers-and-attention.md) — token flow, Q/K/V, masks and decoder mechanics.
- Use the official [PyTorch beginner tutorials](https://docs.pytorch.org/tutorials/beginner/basics/intro.html) to implement these concepts.

- [Computer vision foundations](03-computer-vision.md) — image tasks, preprocessing, transfer learning and detection/segmentation metrics.

- [Multimodal AI foundations](04-multimodal-ai.md) — image/audio/video representations, tasks, evaluation and backend constraints.
