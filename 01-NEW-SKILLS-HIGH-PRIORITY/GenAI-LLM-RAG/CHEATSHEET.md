# GenAI & LLM Cheatsheet - Quick Reference

## LLM Fundamentals
- **LLM**: Large Language Model (billions of parameters, trained on massive text data)
- **Transformer**: Neural network architecture (attention mechanism)
- **GPT**: Generative Pre-trained Transformer (autoregressive, text generation)
- **BERT**: Bidirectional Encoder (masked language modeling, understanding)
- **Context window**: Max tokens LLM can process (4K, 8K, 32K, 128K)
- **Tokens**: Subword units (~4 chars per token, 1 token ≈ 0.75 words)

## Popular LLMs
- **GPT-4**: OpenAI, best quality, 128K context
- **Claude 3**: Anthropic, long context (200K), safety-focused
- **Gemini 1.5**: Google, multimodal, 1M+ context
- **Llama 3**: Meta, open-source, 70B/405B parameters
- **Mistral**: Open-source, efficient, commercial-friendly

## Prompt Engineering
- **Zero-shot**: No examples, direct instruction
- **Few-shot**: Include 1-5 examples in prompt
- **Chain-of-thought (CoT)**: Ask model to explain reasoning
- **System prompt**: Set model behavior/role
- **Temperature**: Randomness (0=deterministic, 1=creative)
- **Top-p**: Nucleus sampling (cumulative probability cutoff)

## RAG (Retrieval-Augmented Generation)
- **RAG**: Retrieve relevant docs → pass to LLM as context → generate answer
- **Benefits**: Up-to-date info, reduced hallucinations, grounded responses
- **Architecture**: Query → Embedding → Vector DB → Retrieve → Context + Query → LLM → Answer
- **Chunking**: Split docs (100-500 tokens per chunk)
- **Retrieval**: Top-k most similar chunks (k=3-10)

## RAG Optimization
- **Query transformation**: Rewrite query for better retrieval
- **Multi-query**: Generate multiple query variations
- **Reranking**: Cross-encoder re-scores retrieved docs
- **Contextual compression**: Summarize retrieved context
- **Hybrid search**: Vector + keyword search
- **Parent-child retrieval**: Retrieve small chunks, return full context

## LLM APIs
- **OpenAI**: GPT-4, GPT-3.5 (fast, cheap)
- **Anthropic**: Claude (safety, long context)
- **Azure OpenAI**: Enterprise OpenAI with Azure security
- **Cohere**: Generate, Embed, Rerank APIs
- **Function calling**: LLM outputs structured data (JSON)

## Function Calling
- **Use case**: LLM decides when to call external functions/APIs
- **Process**: Describe functions → LLM outputs function name + args → Execute → Return result
- **Examples**: Calculator, search, database query, API call

## Agent Frameworks
- **Agent**: LLM with tools, makes decisions autonomously
- **LangChain**: Python framework (chains, agents, memory)
- **LlamaIndex**: Data framework for LLMs (ingestion, indexing, querying)
- **AutoGPT**: Autonomous agent with goals
- **ReAct**: Reasoning + Action pattern

## Prompt Patterns
- **Role prompting**: "You are an expert in..."
- **Step-by-step**: "Let's solve this step by step"
- **Output format**: "Return as JSON with keys: ..."
- **Constraints**: "Keep answer under 100 words"
- **Chain-of-thought**: "Explain your reasoning before answering"

## Cost Optimization
- **Model selection**: Use cheaper models where possible (GPT-3.5 vs GPT-4)
- **Caching**: Cache responses for repeated queries
- **Prompt compression**: Remove unnecessary tokens
- **Streaming**: Stream responses for better UX
- **Batch processing**: Process multiple requests together

## LLM Evaluation
- **Accuracy**: Correctness of answer
- **Faithfulness**: Answer grounded in retrieved context
- **Relevance**: Answer addresses the question
- **Coherence**: Logical, well-structured response
- **Latency**: Response time
- **RAGAS**: Framework for RAG evaluation

## Production Considerations
- **Latency**: Optimize prompt length, use caching, stream responses
- **Cost**: Monitor token usage, use cheaper models when possible
- **Reliability**: Implement retries, fallback models
- **Safety**: Content filtering, PII detection, rate limiting
- **Monitoring**: Track usage, errors, performance

## Guardrails
- **Input validation**: Check for prompt injection, jailbreaking
- **Output filtering**: Remove toxic/inappropriate content
- **PII detection**: Mask sensitive data
- **Hallucination detection**: Verify facts against sources
- **Rate limiting**: Prevent abuse

## Fine-tuning vs RAG
- **RAG**: Easy, fast, up-to-date, no training needed
- **Fine-tuning**: Better performance, specific domain, requires labeled data
- **When to RAG**: Knowledge updates frequently, limited training data
- **When to fine-tune**: Specific task, style/format requirements, have labeled data

## Best Practices
✅ Start with prompt engineering | ✅ Use RAG for knowledge | ✅ Cache responses | ✅ Stream for UX | ✅ Monitor costs | ✅ Implement guardrails | ✅ Test extensively | ✅ Version prompts | ✅ Evaluate with metrics | ✅ Plan for failure modes
