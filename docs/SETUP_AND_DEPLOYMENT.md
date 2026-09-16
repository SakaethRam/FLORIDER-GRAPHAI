# Setup and Deployment

## BYOK (Bring Your Own Key)

FloRider AI requires users to supply their own API key to unlock the AI-powered features. This is configured through the BYOK section available via the interface button on the frontend, rather than through a shared or platform-level key.

Practical implications:

- Each user's usage is billed against their own key, not a shared FloRider AI quota.
- No functional LLM features (`/llm`, `/graph`, `/table`) work until a key is configured; deterministic preprocessing (file parsing, schema extraction) can run independently, but the reasoning step cannot.
- Because the backend is stateless (see `ARCHITECTURE.md`), the key is expected to be supplied per session/request from the frontend rather than persisted server-side.

## Local development

1. **Frontend** — React + TypeScript app; standard `npm install && npm run dev` workflow applies for local iteration.
2. **Connector layer** — TypeScript; runs alongside the frontend and routes requests per the rules in `ARCHITECTURE.md`.
3. **Backend** — FastAPI; run with a standard ASGI server (e.g. `uvicorn`) for local testing of `/llm`, `/graph`, and `/table` independently.
4. **LLM layer** — Gemini API; a valid key must be supplied (see BYOK above) for any endpoint that reaches the LLM layer to return a real response.

Since the backend has no database dependency (see `ARCHITECTURE.md`, Database Strategy), local setup does not require provisioning or migrating a datastore before the APIs are usable.

## Deployment shape

- **Frontend** — deployed as a static/SSR site; the production instance is hosted on Netlify at https://florider-ai.netlify.app/home
- **Backend** — deployed as stateless FastAPI microservices, which makes them straightforward to run behind a load balancer or on any container/serverless platform without session-affinity concerns.
- **No persistent storage** — deployment does not need to provision a database; all processing is in-memory and per-request.

## Scaling notes for operators

- Because every API is stateless, scaling is a matter of adding backend instances; no shared session store needs to be introduced first.
- LLM calls across `/llm`, `/graph`, and `/table` are independent of each other and can be scaled or rate-limited separately if one endpoint sees disproportionate load.
- A caching layer is the natural next step for reducing LLM cost under load (see `ARCHITECTURE.md`, Scalability), and would sit in front of the LLM layer without requiring changes to the stateless request/response contract.
