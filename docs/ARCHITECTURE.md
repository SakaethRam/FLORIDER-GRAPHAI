# Architecture

FloRider AI is a service-oriented system split into four layers, each with a narrow, well-defined responsibility. This document expands on the top-level README with the reasoning behind the split and how a request moves through the system end to end.

## Layers

### 1. Frontend (React + TypeScript)

Owns everything the user sees and does: prompt entry, file upload, and rendering results as graphs or tables. It never talks to the LLM or backend microservices directly; every outbound call goes through the connector layer, which keeps the UI free of routing logic.

### 2. Connector layer (TypeScript)

Sits between the frontend and the backend and is the single place that decides which backend route a given input should hit. It exists so the frontend can stay dumb about backend topology: if a new endpoint is added later, only the connector's routing table changes, not every call site in the UI.

Routing rule, in priority order:

1. Table data present → `/table`
2. File present → `/graph`
3. Otherwise → `/llm`

### 3. Backend (FastAPI)

Three stateless microservices, one per endpoint (`/llm`, `/graph`, `/table`). Each handles its own preprocessing, validation, and LLM orchestration rather than sharing a monolithic request handler. Statelessness is a deliberate constraint: no session storage, no dependency on persistent memory, which is what lets the backend scale horizontally without a shared-state coordination problem.

### 4. LLM layer (Gemini API)

Used selectively, not as a default fallback for every request. Deterministic preprocessing (file parsing, schema extraction) happens before the LLM is invoked, and the LLM is scoped to the specific reasoning task each endpoint needs (entity extraction, relationship inference, ER modeling) rather than being handed the raw input unconstrained.

## Request lifecycle

1. User submits a prompt, file, or table through the frontend.
2. The connector layer inspects the input and picks a route using the priority rule above.
3. The backend preprocesses the input deterministically: parsing the file format, normalizing the data, and augmenting the prompt with that structured context.
4. The augmented, structured prompt is sent to the LLM layer.
5. The LLM response is validated, parsed, and normalized against the endpoint's expected output shape before it leaves the backend.
6. The frontend receives structured output and renders it as a graph, table, or chat response.

## Why this split

- **Modular over monolithic** — `/llm`, `/graph`, and `/table` are separate endpoints rather than one general-purpose route, which keeps each one's preprocessing and prompt logic independently debuggable and scalable.
- **Deterministic + LLM hybrid** — preprocessing (CSV/JSON/PDF parsing, schema extraction) is handled with regular code, and the LLM is reserved for the parts that genuinely require semantic reasoning (entity extraction, relationship inference). This narrows the LLM's job and reduces hallucination surface area.
- **Stateless backend** — no session or persistent memory dependency, so any instance can serve any request, which simplifies horizontal scaling and cloud deployment.

## Data strategy

FloRider AI currently has no primary database layer; all input is processed in-memory. This follows from the system being computation-focused rather than storage-focused: most operations are one-shot transformations (input in, structured output out), so persisting intermediate state would add latency and infrastructure complexity without a corresponding benefit. If a caching layer is added later (see Scalability below), it would sit alongside this in-memory model rather than replace it.

## Scalability

- Stateless APIs mean horizontal scaling is a matter of adding instances, not managing session affinity.
- LLM calls across the three endpoints can be parallelized independently since none of them share state.
- A caching layer is the natural next infrastructure addition, aimed at reducing repeated LLM cost for identical or near-identical inputs rather than at solving a correctness problem.
