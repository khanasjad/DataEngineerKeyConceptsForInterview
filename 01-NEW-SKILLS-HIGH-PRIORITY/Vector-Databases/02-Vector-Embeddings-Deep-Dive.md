# Chapter 02: Vector Embeddings Deep-Dive

**Understanding the Foundation of Vector Databases**

## What Are Embeddings?

Embeddings are dense numerical representations that capture the semantic meaning of data.

### Text Embeddings Example

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')

# Convert text to vectors
text1 = "I love cats"
text2 = "I adore felines"
text3 = "I enjoy programming"

embedding1 = model.encode(text1)  # [0.23, -0.45, 0.67, ..., 0.12]
embedding2 = model.encode(text2)  # [0.22, -0.44, 0.66, ..., 0.11]  ← Similar!
embedding3 = model.encode(text3)  # [-0.56, 0.78, -0.23, ..., 0.45]  ← Different

# Similarity
from sklearn.metrics.pairwise import cosine_similarity

sim_1_2 = cosine_similarity([embedding1], [embedding2])[0][0]  # 0.92 (high)
sim_1_3 = cosine_similarity([embedding1], [embedding3])[0][0]  # 0.15 (low)
```

## Types of Embeddings

### 1. Text Embeddings

**Word2Vec (Word-level):**
```python
from gensim.models import Word2Vec

sentences = [["cat", "sits", "mat"], ["dog", "runs", "park"]]
model = Word2Vec(sentences, vector_size=100, window=5, min_count=1)

# Get vector
cat_vector = model.wv['cat']
```

**Sentence Transformers (Sentence-level):**
```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-mpnet-base-v2')
embeddings = model.encode([
    "This is a sentence",
    "This is another sentence"
])
```

**OpenAI Embeddings:**
```python
from openai import OpenAI

client = OpenAI()
response = client.embeddings.create(
    model="text-embedding-ada-002",
    input="Your text here"
)
embedding = response.data[0].embedding  # 1536 dimensions
```

### 2. Image Embeddings

**CLIP (OpenAI):**
```python
import clip
import torch

model, preprocess = clip.load("ViT-B/32")

image = preprocess(Image.open("photo.jpg")).unsqueeze(0)
with torch.no_grad():
    image_embedding = model.encode_image(image)
```

**ResNet:**
```python
from torchvision import models, transforms

model = models.resnet50(pretrained=True)
model = torch.nn.Sequential(*list(model.children())[:-1])  # Remove last layer

transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor()
])

image = transform(Image.open("photo.jpg")).unsqueeze(0)
with torch.no_grad():
    embedding = model(image).squeeze()
```

### 3. Multimodal Embeddings

**CLIP (Text + Image in same space):**
```python
import clip

model, preprocess = clip.load("ViT-B/32")

# Text embedding
text = clip.tokenize(["a photo of a cat"])
with torch.no_grad():
    text_embedding = model.encode_text(text)

# Image embedding
image = preprocess(Image.open("cat.jpg")).unsqueeze(0)
with torch.no_grad():
    image_embedding = model.encode_image(image)

# Similarity
similarity = cosine_similarity(text_embedding, image_embedding)
```

## Embedding Properties

### 1. Semantic Similarity
```python
model = SentenceTransformer('all-MiniLM-L6-v2')

sentences = [
    "The cat sat on the mat",
    "A feline rested on the rug",  # Similar meaning
    "Python is a programming language"  # Different meaning
]

embeddings = model.encode(sentences)

# Similarity matrix
from sklearn.metrics.pairwise import cosine_similarity
similarities = cosine_similarity(embeddings)

print(similarities)
# [[1.00, 0.78, 0.12],   ← Sentence 1 vs all
#  [0.78, 1.00, 0.15],   ← Sentence 2 vs all
#  [0.12, 0.15, 1.00]]   ← Sentence 3 vs all
```

### 2. Vector Arithmetic
```python
# king - man + woman ≈ queen
king = model.wv['king']
man = model.wv['man']
woman = model.wv['woman']

result = king - man + woman
# Find most similar word
similar = model.wv.similar_by_vector(result, topn=1)
print(similar)  # [('queen', 0.85)]
```

### 3. Dimensionality

**Common Dimensions:**
- Word2Vec: 100-300
- Sentence Transformers: 384-768
- OpenAI ada-002: 1536
- CLIP: 512
- ResNet: 2048

**Trade-off:**
- Higher dimensions → Better representation, slower search
- Lower dimensions → Faster search, less accurate

## Distance Metrics

### 1. Cosine Similarity

**Formula:** Angle between vectors

```python
import numpy as np

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

vec_a = np.array([1, 2, 3])
vec_b = np.array([2, 3, 4])

sim = cosine_similarity(vec_a, vec_b)
# Output: 0.992 (very similar direction)
```

**When to use:** Text embeddings, direction matters more than magnitude

### 2. Euclidean Distance (L2)

**Formula:** Straight-line distance

```python
def euclidean_distance(a, b):
    return np.linalg.norm(a - b)

dist = euclidean_distance(vec_a, vec_b)
# Output: 1.732
```

**When to use:** Image embeddings, magnitude matters

### 3. Dot Product

**Formula:** Sum of element-wise products

```python
def dot_product(a, b):
    return np.dot(a, b)

score = dot_product(vec_a, vec_b)
```

**When to use:** Fast approximation, normalized vectors

## Dimensionality Reduction

### PCA (Principal Component Analysis)

```python
from sklearn.decomposition import PCA

# Original: 1536 dimensions
embeddings = get_embeddings(texts)  # Shape: (n_samples, 1536)

# Reduce to 128 dimensions
pca = PCA(n_components=128)
reduced_embeddings = pca.fit_transform(embeddings)

# Trade-off: 90-95% variance retained, 10x faster search
```

### t-SNE (Visualization)

```python
from sklearn.manifold import TSNE

# Reduce to 2D for plotting
tsne = TSNE(n_components=2)
embeddings_2d = tsne.fit_transform(embeddings)

# Plot
import matplotlib.pyplot as plt
plt.scatter(embeddings_2d[:, 0], embeddings_2d[:, 1])
```

## Best Practices

### 1. Normalize Embeddings

```python
from sklearn.preprocessing import normalize

# Normalize to unit length
embeddings_normalized = normalize(embeddings, norm='l2')

# Now cosine similarity = dot product (faster!)
```

### 2. Batch Processing

```python
# BAD: One at a time
for text in texts:
    embedding = model.encode(text)

# GOOD: Batch
embeddings = model.encode(texts, batch_size=32)
```

### 3. Caching

```python
import hashlib
import pickle

cache = {}

def get_embedding_cached(text):
    key = hashlib.md5(text.encode()).hexdigest()
    
    if key in cache:
        return cache[key]
    
    embedding = model.encode(text)
    cache[key] = embedding
    return embedding
```

## Summary

✅ **Embeddings:** Dense numerical representations of data
✅ **Types:** Text, image, multimodal
✅ **Properties:** Semantic similarity, vector arithmetic
✅ **Metrics:** Cosine, Euclidean, dot product
✅ **Optimization:** Dimensionality reduction, normalization, caching

**Continue to Chapter 03 for Similarity Search Algorithms! 🚀**
