# 02 - RAG Pipeline Design

This file turns the "RAG concept" into an implementable pipeline.

From your source guide, the core flow is:

1. Ingest documents
2. Chunk text
3. Create embeddings
4. Store vectors
5. Retrieve relevant chunks
6. Generate answer with citations

## RAG architecture (production mindset)

Split into two pipelines:

- **Offline indexing pipeline** (document -> vector store)
- **Online query pipeline** (user query -> retrieved context -> LLM answer)

## Offline indexing pipeline

### Step 1: Document ingestion

- Source types: PDF, DOCX, HTML, Notion pages, wikis
- Normalize text and metadata
- Keep metadata: `doc_id`, `title`, `source_url`, `page`, `updated_at`

### Step 2: Chunking

Why chunk?

- Embedding models and LLMs work better with smaller passages
- Better retrieval precision

Recommended defaults:

- Chunk size: 400-800 tokens
- Overlap: 50-120 tokens
- Keep semantic boundaries (headings, paragraphs)

### Step 3: Embeddings

- Convert each chunk to vector
- Store vector + raw text + metadata
- Re-embed only changed chunks (incremental indexing)

### Step 4: Storage

- Save in vector DB
- Use collections per tenant or use-case when needed
- Keep a version field for re-indexing strategy

## Online query pipeline

### Step 1: Query processing

- Detect language
- Optional query rewrite (improve recall)
- Create query embedding

### Step 2: Retrieval

- Use top-k vector search (example: 8-20 chunks)
- Optional hybrid search: vector + BM25
- Optional reranker for relevance refinement

### Step 3: Context assembly

- Deduplicate chunks
- Keep token budget in mind
- Prefer fewer high-quality chunks over many noisy ones

### Step 4: Generation

- Prompt includes:
  - User question
  - Retrieved context
  - System rules ("answer only from context")
- Ask for structured output:
  - `answer`
  - `citations[]`
  - `confidence`

### Step 5: Post-processing

- Enforce output schema
- Validate citations reference retrieved chunks
- Provide fallback message when context is insufficient

## API design suggestion

Keep frontend simple with one route:

- `POST /api/rag/ask`
  - Request: `{ question, conversationId }`
  - Response: `{ answer, citations, traceId, latencyMs }`

For indexing:

- `POST /api/rag/index`
- `POST /api/rag/reindex`
- `GET /api/rag/index/status`

## Quality checks

Track these from day one:

- Retrieval precision at k
- Citation coverage rate
- "I do not know" rate
- Hallucination rate (manual + automated checks)
- P95 latency

## Frontend integration tips

- Show citation cards under each answer
- Add loading phases: "Searching docs", "Generating answer"
- Display confidence label if available
- Provide "Report incorrect answer" action

## Common failure modes

- Chunks too large -> irrelevant context
- Chunks too small -> lost meaning
- Missing metadata -> bad citations
- No fallback path -> confident wrong answers
- No index refresh strategy -> stale answers

## Practice task

Design your own HR assistant RAG flow with:

- 20 sample policy documents
- Chunking + embeddings
- Hybrid retrieval
- Answer with citations in UI
