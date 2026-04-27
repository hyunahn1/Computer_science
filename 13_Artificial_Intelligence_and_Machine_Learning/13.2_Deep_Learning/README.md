# 13.2 Deep Learning

## Scope
Deep learning requires understanding optimization, representation learning, architecture choices, and hardware constraints. Memorizing model names is not enough.

## Core Concepts
- Multilayer perceptrons
- Activation functions: ReLU, GELU, sigmoid, tanh
- Backpropagation and automatic differentiation
- Optimizers: SGD, momentum, Adam, AdamW
- Initialization, normalization, and residual connections
- Regularization: dropout, weight decay, data augmentation
- CNNs: convolution, pooling, receptive field, ResNet
- RNNs, LSTMs, GRUs and sequence modeling limitations
- Training dynamics: vanishing/exploding gradients, loss landscapes

## Systems Concerns
- GPU acceleration and tensor parallelism basics
- Batch size, memory pressure, gradient accumulation
- Mixed precision and numerical stability
- Checkpointing and reproducibility
- Dataset pipelines and input bottlenecks

## Expert Depth Checklist
- [ ] Derive backpropagation for a two-layer network by hand
- [ ] Implement a small neural network without a high-level training framework
- [ ] Explain why residual connections improve optimization
- [ ] Compare SGD, Adam, and AdamW failure modes
- [ ] Demonstrate overfitting, then reduce it with regularization or data augmentation
- [ ] Inspect gradient norms and diagnose vanishing or exploding gradients
- [ ] Explain convolution as weight sharing and locality bias
- [ ] Measure training throughput and identify whether compute or input pipeline is the bottleneck
- [ ] Run an ablation study and report what changed

## Practice Problems
- [ ] Train an MLP and CNN on a small dataset and compare error modes
- [ ] Reproduce a paper figure or benchmark at small scale
- [ ] Explain every tensor shape in a forward and backward pass

## Primary Sources
- [ ] Deep Learning by Goodfellow, Bengio, and Courville
- [ ] Stanford CS231n notes for CNNs
- [ ] PyTorch or TensorFlow official documentation
