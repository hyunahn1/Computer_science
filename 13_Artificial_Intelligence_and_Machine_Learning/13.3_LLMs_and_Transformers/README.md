# 13.3 LLMs & Transformers

## Scope
LLMs combine architecture, data, optimization, retrieval, evaluation, and product constraints. Study the transformer mechanism and the surrounding system, not only prompting.

## Transformer Fundamentals
- Tokenization and vocabulary
- Embeddings and positional encodings
- Scaled dot-product attention
- Multi-head attention
- Feed-forward networks
- Layer normalization and residual connections
- Causal masking vs bidirectional attention
- Encoder, decoder, and encoder-decoder architectures

## LLM Engineering
- Pretraining, supervised fine-tuning, preference tuning
- Prompting, tool use, structured outputs
- RAG: chunking, embeddings, retrieval, reranking, grounding
- Vector databases and approximate nearest neighbor search
- Context windows and attention cost
- Quantization and inference latency
- Evaluation: exact match, human evals, model-graded evals, regression suites
- Safety: hallucination, prompt injection, data leakage, jailbreaks

## Expert Depth Checklist
- [ ] Derive scaled dot-product attention and explain the purpose of the scaling factor
- [ ] Trace tensor shapes through one transformer block
- [ ] Explain why attention is O(n^2) in sequence length
- [ ] Compare BERT-style bidirectional attention with GPT-style causal attention
- [ ] Implement a minimal self-attention layer
- [ ] Build a small RAG prototype and measure retrieval precision/recall
- [ ] Create an evaluation set with expected outputs and failure categories
- [ ] Explain when to use prompting, RAG, fine-tuning, or a smaller task-specific model
- [ ] Analyze a hallucination or prompt-injection failure and propose mitigations
- [ ] Measure latency, token throughput, context length, and cost for an inference path

## Practice Problems
- [ ] Read "Attention Is All You Need" and annotate each equation
- [ ] Implement chunking and retrieval, then test a query that should fail
- [ ] Compare two prompting strategies using the same evaluation set

## Primary Sources
- [ ] Attention Is All You Need
- [ ] The Illustrated Transformer, for visual intuition
- [ ] Provider and framework docs for inference, fine-tuning, and safety evaluations
