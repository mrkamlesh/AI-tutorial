# 01 - Semantic Kernel vs LangChain

This tutorial helps you choose an orchestration framework for GenAI apps.

If you are strong in frontend and new to GenAI, think of these frameworks like "state and workflow managers" for AI features.

## Why you need an orchestration framework

Raw LLM API calls are not enough for real apps. You usually need:

- Prompt templates
- Tool/function calling
- Memory or chat history handling
- Retrieval (RAG)
- Evaluation and observability hooks

Frameworks help you avoid ad-hoc code in many places.

## What is Semantic Kernel (SK)?

Semantic Kernel is a Microsoft framework focused on:

- AI "skills" (functions)
- Planners and agents
- Strong integration with Azure ecosystem
- Good fit for enterprise patterns and .NET teams

Use SK when you want a structured, plugin-style architecture and Azure-first defaults.

## What is LangChain?

LangChain is a popular framework with:

- Large ecosystem and community
- Chains, tools, agents, retrievers
- Good integrations across many providers
- Strong support in Python and JavaScript

Use LangChain when you want many adapters quickly and broad community examples.

## Quick comparison

| Area | Semantic Kernel | LangChain |
|---|---|---|
| Best fit | Azure + enterprise workflows | Fast prototyping + broad ecosystem |
| Architecture style | Skills/plugins and planners | Chains/runnables/agents |
| Language strength | .NET, Python, Java | Python, JavaScript/TypeScript |
| Integrations | Strong Microsoft stack | Very wide third-party support |
| Learning curve | Moderate, opinionated | Moderate, many patterns |

## Mental model for a frontend developer

- **React components** are like **AI functions/tools**.
- **State management** is like **conversation + memory state**.
- **API composition** is like **chain/agent orchestration**.
- **Error boundaries** are like **guardrails + fallback prompts**.

## Minimal examples

### LangChain (TypeScript) shape

```ts
// pseudo-shape, not full production code
const retriever = vectorStore.asRetriever();
const chain = createRetrievalChain({
  retriever,
  llm,
});
const result = await chain.invoke({ input: "How many vacation days?" });
```

### Semantic Kernel shape

```ts
// conceptual example
const kernel = createKernel();
kernel.addFunction("hr", "searchPolicy", searchPolicyFn);
const answer = await kernel.invokePrompt("Answer with citations: {{$input}}", {
  input: "How many vacation days?",
});
```

## Which one should you start with?

Practical suggestion for your profile:

1. Start with **LangChain JS/TS** for fast momentum.
2. Learn **Semantic Kernel** next if your target environment is Azure-heavy enterprise.
3. Keep business logic framework-agnostic (prompt templates, retrieval interfaces, eval logic).

## Common beginner mistakes

- Locking all logic to one framework too early
- Mixing retrieval, prompting, and UI logic in one file
- No trace IDs across request -> retrieval -> LLM call
- Skipping prompt versioning

## Practice task

Build one endpoint `/api/ask` with:

- Input: question
- Retrieval: mocked for now
- Output: answer + source list

Then implement the same endpoint in both SK and LangChain style to compare developer experience.
