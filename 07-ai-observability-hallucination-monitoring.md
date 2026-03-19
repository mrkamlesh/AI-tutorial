# 07 - AI Observability and Hallucination Monitoring

If you do not measure your AI system, you cannot trust it in production.

This tutorial covers what to log, what to monitor, and how to detect hallucinations early.

## What is AI observability?

AI observability means tracking the full request path:

- User input
- Retrieval results
- Prompt version
- Model response
- Quality/safety outcomes

It is similar to API observability, but with extra LLM-specific signals.

## Minimum telemetry per request

Log these fields for every RAG call:

- `trace_id`
- `conversation_id`
- `user_question_hash` (avoid raw PII when possible)
- `retrieved_doc_ids`
- `retrieval_scores`
- `prompt_version`
- `model_deployment`
- `latency_ms`
- `input_tokens`, `output_tokens`
- `answer_confidence`
- `citation_count`

## Hallucination detection signals

Use a mix of heuristics + review:

- No citation but high-confidence claim
- Citation points to unrelated chunk
- Named entities in answer not present in context
- Numeric values not present in retrieved text

## Practical monitoring metrics

- **Answer groundedness rate**: % answers fully supported by context
- **Citation validity rate**: % citations that actually support claims
- **No-context fallback rate**: how often system correctly declines
- **User correction rate**: users reporting wrong answers
- **P95 latency** and **cost per request**

## Alerting suggestions

Trigger alerts when:

- hallucination rate > threshold (for example 3%)
- citation validity drops sharply
- retrieval returns empty for known good questions
- latency or token spend spikes unexpectedly

## Human-in-the-loop workflow

Create a review queue for:

- low-confidence responses
- high-impact domains (HR, finance, legal)
- responses flagged by users

Use reviewer feedback to improve:

- chunking
- prompts
- retrieval strategy
- guardrails

## Dashboard starter layout

Panel 1: traffic and latency

Panel 2: quality metrics (groundedness, citation validity)

Panel 3: safety metrics (blocked prompts, policy violations)

Panel 4: top failed questions and failure reasons

## Frontend integration ideas

- Attach `traceId` to every chat bubble metadata
- Add "Was this accurate?" thumbs up/down
- If downvote, capture reason category:
  - wrong fact
  - missing citation
  - not relevant
  - unsafe response

This feedback loop is extremely valuable for improving your RAG system.

## Practice task

Implement a weekly quality report:

- Top 20 failed questions
- Hallucination trend by day
- Citation validity trend
- Prompt version comparison

Then pick one fix and measure impact next week.
