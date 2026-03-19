# 03 - Azure OpenAI and LLM APIs

This tutorial explains how to call LLMs safely and cleanly using Azure OpenAI.

## Core concepts first

In Azure OpenAI, you usually work with:

- **Resource**: your Azure OpenAI instance
- **Deployment**: your model alias in Azure (for example, GPT-4.1 deployment name)
- **API version**: endpoint behavior version
- **Authentication**: API key or Azure AD token

Important: in code, you often call a **deployment name**, not raw model name.

## Typical call flow

1. Build messages (`system`, `user`, optional `assistant`)
2. Attach parameters (`temperature`, `max_tokens`, `top_p`, etc.)
3. Send request to chat completions endpoint
4. Parse response text + token usage
5. Log `traceId`, latency, token cost estimate

## Basic TypeScript example (Node API route)

```ts
import OpenAI from "openai";

// Azure OpenAI uses a custom base URL; api-version is required (e.g. "2024-02-15-preview")
const client = new OpenAI({
  apiKey: process.env.AZURE_OPENAI_API_KEY,
  baseURL: `${process.env.AZURE_OPENAI_ENDPOINT}/openai/deployments/${process.env.AZURE_OPENAI_DEPLOYMENT}`,
  defaultQuery: { "api-version": process.env.AZURE_OPENAI_API_VERSION ?? "2024-02-15-preview" },
  defaultHeaders: { "api-key": process.env.AZURE_OPENAI_API_KEY! },
});

export async function askLLM(question: string) {
  const response = await client.chat.completions.create({
    model: process.env.AZURE_OPENAI_DEPLOYMENT!,
    messages: [
      { role: "system", content: "You are a helpful assistant." },
      { role: "user", content: question },
    ],
    temperature: 0.2,
    max_tokens: 2048,
  });

  return response.choices[0]?.message?.content ?? "";
}
```

## Embeddings (for RAG indexing)

Embeddings use a separate Azure deployment (e.g. `text-embedding-ada-002`):

```ts
const embeddingClient = new OpenAI({
  apiKey: process.env.AZURE_OPENAI_API_KEY,
  baseURL: `${process.env.AZURE_OPENAI_ENDPOINT}/openai/deployments/${process.env.AZURE_OPENAI_EMBEDDING_DEPLOYMENT}`,
  defaultQuery: { "api-version": "2024-02-15-preview" },
  defaultHeaders: { "api-key": process.env.AZURE_OPENAI_API_KEY! },
});

const { data } = await embeddingClient.embeddings.create({
  model: process.env.AZURE_OPENAI_EMBEDDING_DEPLOYMENT!,
  input: "Employees receive 15 days annual vacation",
});
// data[0].embedding → number[] (1536 for ada-002)
```

## Streaming (for better UX)

```ts
const stream = await client.chat.completions.create({
  model: process.env.AZURE_OPENAI_DEPLOYMENT!,
  messages: [{ role: "user", content: question }],
  stream: true,
});

for await (const chunk of stream) {
  const text = chunk.choices[0]?.delta?.content ?? "";
  if (text) process.stdout.write(text);
}
```

## API design best practices

- Keep one server-side "LLM gateway" module
- Never call LLM API directly from browser client with secrets
- Add timeout + retry + circuit breaker
- Use idempotency key for retried requests when possible

## Parameter guidelines

- `temperature`:
  - 0.0-0.3 for factual RAG
  - 0.6+ for creative tasks
- `max_tokens`:
  - Cap response length to control latency/cost
- `top_p`:
  - Usually keep default unless you know why to tune

## Error handling matrix

- `429` rate limit -> exponential backoff + retry
- `5xx` transient -> retry with jitter
- `401/403` auth/config issue -> fail fast, alert
- Timeout -> fallback answer with retry option

## Security essentials

- Keep keys in environment variables / secret manager
- Redact PII before logging prompts
- Store only required prompt/response slices
- Add content filtering if your use-case needs moderation

## Cost control

- Cache embeddings and repeated answers
- Use smaller model for query rewrite/classification
- Use larger model only for final answer if needed
- Track tokens per endpoint and per tenant

## Frontend-oriented integration pattern

- Backend endpoint `/api/chat`
- Streaming response to UI for perceived speed
- Render citations and latency
- Include `traceId` in UI for support/debug

## Practice task

Implement:

- `POST /api/ai/complete`
- Retry on `429`
- Return `{ answer, usage, traceId }`
- Add a simple dashboard card: requests, errors, avg latency
