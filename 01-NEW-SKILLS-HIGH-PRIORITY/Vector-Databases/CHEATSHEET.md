# Vector Databases Cheatsheet - Quick Reference

## Core Concepts
- **Vector**: Array of numbers representing data (embeddings)
- **Embedding**: Dense vector representation of text/image/audio
- **Similarity search**: Find vectors close to query vector
- **Nearest neighbor**: Most similar vectors in database
- **Vector database**: Optimized storage and search for embeddings

## Similarity Metrics
- **Cosine similarity**: Angle between vectors (range: -1 to 1)
- **Euclidean distance**: Straight-line distance (L2 norm)
- **Dot product**: Sum of element-wise multiplication
- **Manhattan distance**: Sum of absolute differences (L1 norm)
- **Hamming distance**: Binary vector differences

## Indexing Algorithms
- **HNSW (Hierarchical Navigable Small World)**: Graph-based, fast, accurate
- **IVF (Inverted File Index)**: Clustering-based, balanced speed/accuracy
- **LSH (Locality Sensitive Hashing)**: Hash similar vectors to same bucket
- **Flat index**: Brute-force search (100% recall, slow for large datasets)
- **Product quantization**: Compress vectors for memory efficiency

## Vector Databases
- **Pinecone**: Managed, serverless, easy to use
- **Weaviate**: Open-source, hybrid search, GraphQL API
- **Milvus**: Open-source, Kubernetes-native, highly scalable
- **Qdrant**: Open-source, Rust-based, fast filtering
- **Chroma**: Open-source, simple, embedded DB
- **Faiss**: Facebook library (not full DB, just indexing)
- **pgvector**: PostgreSQL extension (vector column type)

## Embeddings
- **Text embeddings**: OpenAI (text-embedding-3), Cohere, Sentence Transformers
- **Image embeddings**: CLIP, ResNet, Vision Transformers
- **Embedding dimensions**: 384, 768, 1536 (higher = more info, slower search)
- **Embedding models**: Choose based on domain, language, latency needs

## RAG (Retrieval-Augmented Generation)
- **RAG**: Retrieve relevant docs + generate response with LLM
- **Chunking**: Split documents into smaller pieces (100-500 tokens)
- **Query**: User question converted to embedding
- **Retrieval**: Find top-k similar chunks
- **Context**: Pass retrieved chunks to LLM
- **Generation**: LLM generates answer based on context

## Chunking Strategies
- **Fixed-size**: Split by character/token count (simple, may break sentences)
- **Semantic**: Split by meaning (paragraphs, sections)
- **Recursive**: Split recursively until chunk size met
- **Overlap**: Include overlap between chunks (preserve context)

## Hybrid Search
- **Vector + keyword**: Combine semantic and lexical search
- **BM25**: Traditional keyword ranking algorithm
- **Fusion**: Combine scores from multiple search methods
- **Reranking**: Re-score results with cross-encoder model

## Performance Optimization
- **Index tuning**: Adjust HNSW parameters (M, efConstruction, efSearch)
- **Batch operations**: Insert/query multiple vectors at once
- **Metadata filtering**: Pre-filter by metadata before vector search
- **Caching**: Cache frequent queries
- **Quantization**: Compress vectors (reduce memory, faster search)

## Production Considerations
- **Scalability**: Horizontal scaling (sharding, replication)
- **Backup**: Regular backups of vector index
- **Monitoring**: Track latency, throughput, index size
- **Cost**: Storage (embeddings) + compute (search)
- **Multi-tenancy**: Isolate data by namespace/collection

## RAG Evaluation
- **RAGAS**: Retrieval metrics (context_precision, context_recall, faithfulness, answer_relevance)
- **Faithfulness**: Answer grounded in retrieved context
- **Relevance**: Retrieved docs relevant to query
- **Answer quality**: Coherent, accurate, complete

## Use Cases
- **Semantic search**: Find documents by meaning, not just keywords
- **Question answering**: RAG over knowledge base
- **Recommendation**: Find similar products/content
- **Anomaly detection**: Find outliers in embeddings
- **Duplicate detection**: Find near-duplicate documents
- **Image search**: Find similar images

## Best Practices
✅ Choose right embedding model | ✅ Tune chunk size/overlap | ✅ Use hybrid search | ✅ Add metadata for filtering | ✅ Rerank results | ✅ Evaluate with RAGAS | ✅ Cache frequent queries | ✅ Monitor performance | ✅ Backup indexes | ✅ Test different similarity metrics
