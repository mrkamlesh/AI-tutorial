# 04 - Vector Database: Qdrant and Chroma

This tutorial covers where embeddings live and how retrieval happens.

## Why a vector database?

A vector DB helps you:

- Store embeddings efficiently
- Run nearest-neighbor search quickly
- Filter results using metadata
- Scale retrieval as document count grows

Without it, semantic search becomes slow and hard to maintain.

## Qdrant vs Chroma (beginner view)

| Area | Qdrant | Chroma |
|---|---|---|
| Deployment | Strong production server options | Great for local/dev and rapid prototypes |
| Performance | Strong indexing and filtering | Good for small-medium workloads |
| Operations | Better operational controls | Simpler setup |
| Ecosystem | Good SDKs, growing usage | Popular in tutorial/prototype workflows |

Rule of thumb:

- Start with **Chroma** for local prototype speed
- Move to **Qdrant** for production-grade control and scale

## Data model design

Each vector record should include:

- `id`: stable unique ID
- `embedding`: numeric vector (dimension must match model — e.g. **1536** for `text-embedding-ada-002`)
- `text`: original chunk text
- `metadata`:
  - `doc_id`
  - `title`
  - `source`
  - `page`
  - `chunk_index`
  - `updated_at`

Good metadata is required for useful citations and filters.

> **Important:** Never mix embeddings from different models in the same collection. Dimension and meaning space differ.

## Indexing pipeline checklist

- Chunk text consistently
- Use one embedding model version per collection
- Store model/version in metadata
- Upsert changed chunks only
- Remove deleted document chunks

## Retrieval strategies

### 1) Pure vector search

- Best for semantic matching
- Can miss exact keywords/acronyms

### 2) Hybrid search

- Vector + keyword (BM25 style)
- Better for enterprise docs with terms and IDs

### 3) Filtered retrieval

- Filter by metadata (`department=hr`, `year=2026`)
- Improves relevance in multi-domain corpora

## Practical tuning knobs

- `k` (top results): start with 8, tune to 12-20
- Minimum score threshold: reject weak matches
- Chunk size/overlap: tune with evaluation set
- Optional reranking for better final context

## Common mistakes

- Mixing embeddings from different models in same collection
- No metadata, so no traceable citations
- Very high `k` producing noisy prompts
- Reindexing everything on every upload

## Frontend impact

Better retrieval quality directly improves:

- Answer trust
- Citation usefulness
- Perceived intelligence of assistant

Even with a strong LLM, poor retrieval means poor UX.

## Quick code snippets

### Chroma (local)

```ts
import { ChromaClient } from "chromadb";

const client = new ChromaClient();
const collection = await client.getOrCreateCollection({ name: "policies", metadata: { "hnsw:space": "cosine" } });

// Add (embedding dim must match model — 1536 for text-embedding-ada-002)
await collection.add({
  ids: ["chunk-1"],
  embeddings: [[0.1, -0.2, ...]],
  documents: ["Employees receive 15 days annual vacation"],
  metadatas: [{ doc_id: "hr-handbook", page: 15 }],
});

// Query
const results = await collection.query({ queryEmbeddings: [queryVector], nResults: 8 });
```

### Qdrant

```ts
import { QdrantClient } from "@qdrant/js-client-rest";

const client = new QdrantClient({ url: "http://localhost:6333" });
await client.upsert("policies", { points: [{ id: "chunk-1", vector: embedding, payload: { text, doc_id, page } }] });
const { points } = await client.search("policies", { vector: queryEmbedding, limit: 8 });
```

> **Note:** Package APIs may vary; check `chromadb` and `@qdrant/js-client-rest` docs for your version.

## Practice task

Build a mini benchmark:

1. Index 100 policy chunks in Chroma
2. Run 20 representative questions
3. Measure top-3 relevance manually
4. Repeat with Qdrant and compare latency + relevance
