# Prompting Strategy and Guardrails

Prompt design is central to how FloRider AI stays reliable despite routing real user input to an LLM. This page documents the prompting approach per endpoint and the guardrail system that keeps the system on-domain.

## Prompting modes

### General LLM prompting (`/llm`)

Two distinct modes, selected via the `model` field:

- **FloModel** — unrestricted general-purpose reasoning. No domain guardrails applied.
- **GraphModel** — restricted mode intended for dataset-scoped queries. Includes:
  - An explicit role definition for the model
  - Example queries to anchor expected behavior
  - Strict output formatting requirements
  - Guardrail enforcement (see below)

### Graph model prompting (`/graph`)

Prompts convert a parsed dataset into business entities and relationships. Constraints applied at the prompt level:

- Short outputs only
- JSON-only responses, no prose
- A hard limit on output size

These constraints exist to keep the extraction step cheap and predictable, since a dataset-to-graph conversion can otherwise produce open-ended, unbounded output.

### Table model prompting (`/table`)

Two-step prompting:

1. A primary prompt converts tabular data into an ER structure, using column headers and a sample of rows as context rather than the full table (to control token usage).
2. A second, separate prompt generates a human-readable explanation of the resulting ER model.

Keeping extraction and explanation as separate prompts means a change to one doesn't risk destabilizing the other's output format.

## Guardrails

The system is designed to answer questions about the provided dataset or domain only, and to decline everything else: general knowledge questions, creative writing requests, and other off-topic prompts. The canonical refusal response is:

> "This system is designed to answer questions related to the provided dataset only."

### Implementation

Guardrail enforcement happens at two points:

1. **Before the LLM call** — keyword-based intent detection screens obviously off-domain prompts before they're even sent to the model, saving the API call entirely.
2. **At the prompt level** — GraphModel prompts carry explicit domain-limiting constraints, reinforcing the restriction even for prompts that pass the keyword screen.
3. **After the LLM call** — output is validated against the expected format, with a fallback response used when validation fails.

This layered approach (pre-filter, prompt constraint, post-validation) is treated as a core evaluation criterion for the system, not an afterthought bolted onto a general-purpose chat flow.

## Why the hybrid approach matters here specifically

Because `/graph` and `/table` both feed structured output directly into a UI (a rendered graph, a rendered ER diagram), an unconstrained or malformed LLM response doesn't just look bad in a chat window, it breaks the render. The combination of deterministic preprocessing plus tightly scoped, format-constrained prompts is what keeps that failure mode rare rather than eliminating LLM involvement altogether.
