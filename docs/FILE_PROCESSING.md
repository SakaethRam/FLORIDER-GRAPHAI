# File Processing

FloRider AI accepts several input formats across its endpoints and puts each one through the same deterministic pipeline before anything reaches the LLM layer.

## Supported formats

| Format | Typical route      | Notes |
|--------|---------------------|-------|
| CSV    | `/graph`, `/llm`     | Parsed into rows/columns before entity extraction. |
| JSON   | `/graph`, `/table`, `/llm` | Table endpoint expects a row-oriented JSON array specifically. |
| JSONL  | `/llm`               | Line-delimited records. |
| TXT    | `/graph`, `/llm`     | Treated as unstructured content. |
| PDF    | `/llm`               | Text is extracted before prompt augmentation. |
| Images | `/llm`               | Handled as supporting context for a prompt. |

## Processing pipeline

1. **Content extraction** — the raw file is parsed into text or structured records depending on format (e.g. CSV/JSON into rows, PDF into extracted text).
2. **Truncation for token limits** — extracted content is truncated as needed so the augmented prompt stays within the LLM's context budget. This happens before the LLM call, not as a retry-after-failure step.
3. **Structured embedding into prompts** — rather than pasting raw file content into a prompt, extracted content is embedded in a structured way (e.g. column headers plus a row sample for tabular data) so the LLM receives context it can reason over predictably.

## Why truncation happens before the call, not after

Because `/graph` and `/table` prompts are deliberately kept short and JSON-only (see `PROMPTING_AND_GUARDRAILS.md`), oversized input has to be trimmed proactively. Sending an oversized prompt and hoping the model handles it gracefully would work against the same determinism the rest of the pipeline is built around.

## Practical guidance for integrators

- Since truncation trims rather than summarizes, pre-filter very large CSV/JSON files to the columns or records that actually matter for the query before upload, so nothing relevant gets cut.
- Verify PDF text-extraction quality on scanned or image-heavy documents specifically; the pipeline's stated capability is "content extraction," which does not necessarily imply OCR.
