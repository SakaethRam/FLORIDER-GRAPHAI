# API Reference

FloRider AI's backend exposes three stateless FastAPI endpoints. This page documents each one's inputs, outputs, and intended use, as a companion to the connector's routing logic described in `ARCHITECTURE.md`.

All endpoints require a user-supplied API key (BYOK) to reach the underlying LLM; see `SETUP_AND_DEPLOYMENT.md` for how that key is configured.

## `/llm` (`#chat`)

Handles prompt-based queries and file-backed analysis. This is the default route when neither a file nor table data is present, or when a file is supplied for general-purpose reasoning rather than graph extraction.

**Inputs**

| Field   | Type              | Required | Notes |
|---------|-------------------|----------|-------|
| prompt  | string            | Yes      | The user's natural-language query. |
| file    | file              | No       | Optional supporting document; see `FILE_PROCESSING.md` for supported formats. |
| model   | `"flo"` \| `"graph"` | Yes   | Selects the prompting mode: `flo` is unrestricted general reasoning, `graph` is domain-restricted with guardrails enforced. See `PROMPTING_AND_GUARDRAILS.md`. |

**Output**

A normalized chat-style response. When `model` is `graph`, out-of-domain prompts are rejected with a fixed refusal message rather than answered.

## `/graph`

Converts an uploaded dataset into a graph structure of entities and relationships. This is the route the connector selects whenever a file is present without table data.

**Inputs**

| Field | Type              | Required | Notes |
|-------|-------------------|----------|-------|
| file  | CSV / JSON / TXT  | Yes      | Parsed deterministically before being handed to the LLM layer for entity/relationship inference. |

**Output**

A list of entities and the relationships inferred between them, structured for direct rendering as a graph in the frontend.

**Prompting notes** — graph prompts are deliberately constrained: short outputs, JSON-only responses, and a limited output size, to keep extraction deterministic-ish and cheap even though the underlying model call is not.

## `/table`

Converts tabular JSON data into an entity-relationship (ER) model.

**Inputs**

| Field | Type        | Required | Notes |
|-------|-------------|----------|-------|
| data  | JSON array  | Yes      | Row-oriented tabular data. Column headers and a sample of rows are injected into the prompt as context. |

**Output**

| Field         | Description |
|---------------|-------------|
| entities      | Extracted entities from the table. |
| relationships | Inferred relationships between entities. |
| table         | The normalized table data. |
| description   | A generated explanation of the ER model, produced by a second prompt separate from the extraction prompt. |

## Connector routing (for reference)

The frontend never calls these endpoints directly by name; the connector layer picks one per request:

```
table data present  -> /table
file present         -> /graph
otherwise             -> /llm
```

See `ARCHITECTURE.md` for the full request lifecycle these endpoints sit inside.

## Error handling conventions

- Out-of-domain queries against `/llm` in `graph` mode return a fixed refusal rather than a best-effort answer (see `PROMPTING_AND_GUARDRAILS.md`).
- Because all three endpoints are stateless, a failed request can simply be retried by the client without any server-side cleanup.
