# Convolutional and recurrent neural networks

## Why learn architectures besides Transformers

Convolutional and recurrent networks show how architecture encodes assumptions about local structure and sequence. They remain useful in vision, signals, edge devices and existing systems. Learning them also clarifies what attention changed and what it did not.

## 1. Convolutional neural networks (CNNs)

A convolutional layer applies a small learned kernel across local neighborhoods. The same kernel weights are reused at many spatial positions; this is parameter sharing. It reduces parameters compared with a dense layer over every pixel and encodes a useful assumption: nearby pixels/signals often relate.

For an image, a convolution takes input channels, output channels, kernel height/width, stride and padding. Output shape depends on all of these; the formula for one spatial dimension is approximately:

out = floor((in + 2*padding - dilation*(kernel-1) - 1)/stride + 1).

A 3×3 kernel does not mean one filter: an output channel generally has one kernel per input channel and sums across input channels. Inspect tensor layout and framework conventions.

### Receptive field and pooling

The receptive field is the input region influencing one output. Stacking layers expands it. Pooling/downsampling reduces spatial resolution and computation but may discard fine detail. Strided convolutions can also downsample.

CNNs work well when local patterns and translation-related reuse are useful. A standard CNN may struggle to represent very long-range relationships without depth, larger kernels or additional mechanisms. Data augmentation such as flips is valid only if the transformed example preserves its label; flipping digits, medical images or directional signals may change meaning.

### CNN workflow

1. Define input layout and label.
2. Establish a simple baseline.
3. Normalize using training-only statistics.
4. Implement a small convolutional model.
5. Verify output shapes with one synthetic batch.
6. Train and inspect learning curves and error examples.
7. Evaluate subgroup and corruption/shift robustness.
8. Measure model size, latency and memory on target hardware.

## 2. Recurrent neural networks (RNNs)

A recurrent network processes a sequence one step at a time while maintaining hidden state:
h_t = f(x_t, h_(t-1)).
The state carries information from earlier steps. This natural sequential structure suits variable-length sequences and streaming signals.

Basic RNNs can struggle with long dependencies because gradients may vanish or explode through many repeated steps. Long short-term memory (LSTM) and gated recurrent unit (GRU) use gates to control what information is retained, updated and emitted. Gates improve practical learning in many cases but do not create unlimited memory.

RNNs are often less parallelizable across sequence positions during training than Transformers. They can still be useful for low-latency streaming, compact deployments and established pipelines.

## 3. CNN versus RNN versus Transformer

| Architecture | Built-in assumption | Parallelism | Typical use |
|---|---|---|---|
| CNN | Nearby patterns repeat across position | High across spatial positions | Vision, audio/signal features, local sequence patterns |
| RNN/LSTM/GRU | Sequence state evolves stepwise | Limited across time steps | Streaming/time-series/legacy sequential tasks |
| Transformer | Content-dependent attention relates sequence positions | High for prompt/training positions, autoregressive decode remains sequential | Language, multimodal, long-context relationships |

These are tendencies. Model quality depends on data, scale, training and task. A Transformer can be inefficient for a small sensor task; a CNN can be excellent for local patterns; an RNN can meet streaming constraints.

## 4. Common training concerns

All architectures need correct tensor shapes, labels, loss, split and evaluation. Image augmentation, sequence padding and masking are task-sensitive. For variable-length sequences, do not let padding positions affect loss or attention. Report the method for handling padding, truncation and class imbalance.

Small overfit tests help catch wiring errors. A model memorizing a few examples is a diagnostic, not evidence of generalization.

## Questions and answers

**Q: Why does a CNN share the same kernel across an image?**  
A: It assumes a local pattern can be useful at multiple positions. Weight sharing reduces parameters and provides a form of translation equivariance.

**Q: Does pooling always help?**  
A: No. It reduces resolution and cost but may discard details. Evaluate accuracy and latency against alternatives.

**Q: Why did Transformers replace many RNNs in NLP?**  
A: Attention can model flexible relationships and process training positions in parallel more effectively. Inference generation is still sequential token by token and can require large memory.

**Q: Are RNNs obsolete?**  
A: No. Their sequential state can fit streaming or compact workloads. Architecture choice follows requirements and measurements.

## Practice

1. Compute output shape for a convolution with input 32×32, kernel 3, stride 1, padding 1.
2. Draw a convolution and label input channels, kernel, output channels and shared weights.
3. Trace hidden state across a four-step RNN input.
4. Explain a vanishing-gradient risk through a long recurrent chain.
5. Choose an architecture for image classification, streaming sensor data and text generation; state assumptions.

## Completion check

You can explain local receptive fields, parameter sharing, recurrent state and gating, and contrast their resource/parallelism assumptions with Transformer attention.