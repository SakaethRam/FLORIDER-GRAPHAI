# FloRider • AI

FloRider AI is a modular AI-powered data intelligence system designed to transform raw inputs into structured insights, graph representations, and entity-relationship models. The platform integrates large language models with deterministic preprocessing pipelines to enable reliable, domain-constrained analysis of datasets and tabular data.

>VISIT FLORIDER WEB: [@FloRider • AI ](https://florider-ai.netlify.app/home)

<img width="1352" height="688" alt="FloRider" src="https://github.com/user-attachments/assets/e19fc2a7-b54b-423c-a545-d0c351986a78" />

---

## *Note
To unlock the full functionality of FloRider AI, users must configure their API key using the BYOK (Bring Your Own Key) section available via the interface button. This enables the AI-powered features and ensures personalized, secure access to the underlying language models.

---

## Overview

FloRider AI provides three core capabilities:

* Natural language and file-based reasoning using LLMs
* Dataset-to-graph transformation for structural analysis
* Tabular data conversion into entity-relationship (ER) models

The system is designed to bridge the gap between unstructured input and structured analytical outputs while maintaining strict domain control.

---

## System Architecture

The system follows a modular service-oriented architecture with clear separation between processing layers.

### Components

1. **Frontend (React + TypeScript)**

   * User interaction layer
   * File uploads and prompt input
   * Visualization of graphs and tables
   * API orchestration via a connector layer

2. **Connector Layer (TypeScript)**

   * Acts as a unified interface between frontend and backend
   * Routes requests dynamically based on input type:

     * Prompt → LLM API
     * File → Graph API
     * Table → Table API

3. **Backend (FastAPI)**

   * Stateless microservices
   * Handles preprocessing, validation, and LLM orchestration
   * Exposes three endpoints:

     * `/llm`
     * `/graph`
     * `/table`

4. **LLM Layer (Gemini API)**

   * Responsible for semantic reasoning and transformation
   * Used selectively with strong prompt constraints

---

### Data Flow

1. User submits input (prompt, file, or table)
2. Connector determines appropriate backend route
3. Backend preprocesses input:

   * File parsing
   * Data normalization
   * Prompt augmentation
4. Structured prompt sent to LLM
5. Response is validated, parsed, and normalized
6. Output returned to frontend for visualization

---

## Design Decisions

### 1. Modular API Design

Instead of a monolithic endpoint, the system uses specialized APIs:

* `#chat` for reasoning tasks
* `#graph` for dataset transformation
* `#table` for ER modeling

This improves:

* Maintainability
* Debugging
* Scalability

---

### 2. Deterministic + LLM Hybrid Approach

The system avoids fully relying on LLMs by combining:

* Deterministic preprocessing:

  * File parsing (CSV, JSON, PDF)
  * Schema extraction
* LLM-based reasoning:

  * Entity extraction
  * Relationship inference

This reduces hallucination and improves reliability.

---

### 3. Stateless Backend

All APIs are stateless:

* No session storage
* No dependency on persistent memory

This enables:

* Horizontal scaling
* Easy deployment in cloud environments

---

## Database Strategy

Currently, FloRider AI operates without a primary database layer and processes all inputs in-memory.

### Rationale

* The system is computation-focused rather than storage-focused
* Most operations are real-time transformations
* Avoids unnecessary latency and infrastructure complexity

### Future Considerations

A database layer may be introduced for:

* Caching LLM responses (Redis)
* Storing processed datasets (MongoDB / PostgreSQL)
* User session tracking and analytics

---

## LLM Prompting Strategy

Prompt engineering is a critical component of FloRider AI. Each API uses structured prompts tailored to its function.

### 1. Graph Model Prompting

* Converts datasets into business entities and relationships
* Enforces:

  * Short outputs
  * JSON-only responses
  * Limited output size

### 2. Table Model Prompting

* Converts tabular data into ER structures
* Uses:

  * Column-based context
  * Sample row injection
* Includes a second prompt for explanation generation

### 3. General LLM Prompting

* Two modes:

  * FloModel (unrestricted)
  * GraphModel (restricted)

GraphModel includes:

* Explicit role definition
* Example queries
* Strict output formatting
* Guardrails enforcement

---

## Guardrails

The system enforces strict domain-specific constraints to ensure reliable and relevant outputs.

<img width="1903" height="906" alt="Chat Hardrills" src="https://github.com/user-attachments/assets/556e8f8d-cf26-49dd-b7d8-06168fac5522" />

### Guardrails

The system must restrict queries to the dataset and domain.

It should appropriately handle or reject unrelated prompts such as:

* General knowledge questions
* Creative writing requests
* Irrelevant topics

Example response:

"This system is designed to answer questions related to the provided dataset only."

### Implementation

* Keyword-based intent detection before LLM call
* Prompt-level constraints reinforcing domain limits
* Output validation and fallback responses

This is a critical evaluation criterion for the system.

---

## File Processing Capabilities

Supported formats:

* CSV
* JSON
* JSONL
* TXT
* PDF
* Images

Processing includes:

* Content extraction
* Truncation for token limits
* Structured embedding into prompts

---

## API Endpoints

### `#chat`

Handles:

* Prompt-based queries
* File-based analysis

Inputs:

* prompt (string)
* file (optional)
* model (flo | graph)

---

### `#graph`

Converts datasets into graph structures.

<img width="1902" height="862" alt="Chat Graph" src="https://github.com/user-attachments/assets/60249006-394e-4b07-ab43-0550dfea40ae" />

Input:

* File (CSV, JSON, TXT)

Output:

* List of entities and relationships

---

### `#table`

Converts tabular JSON data into ER models.

<img width="1902" height="866" alt="Chat Table" src="https://github.com/user-attachments/assets/f02fad0c-e989-4e53-8865-ece69d94e07a" />

Input:

* JSON array

Output:

* Entities
* Relationships
* Table data
* Description

---

## Connector Layer

The frontend uses a unified connector:

* Abstracts API complexity
* Enables dynamic routing
* Standardizes responses

Example routing logic:

* Table data present → `/table`
* File present → `/graph`
* Otherwise → `/llm`

---

## Scalability Considerations

* Stateless APIs enable horizontal scaling
* LLM calls can be parallelized
* Future caching layer can reduce API cost

---

## Future Enhancements

* Streaming LLM responses
* Graph visualization engine
* Authentication and rate limiting
* Multi-model support
* Persistent storage layer
* Real-time collaboration features

---

## Installation

### Backend

```
pip install fastapi uvicorn requests python-multipart PyPDF2
uvicorn main:app --reload
```

### Frontend

```
npm install
npm run dev
```

## Vision

FloRider AI is designed as a foundation for intelligent data systems where structured insights can be generated from any form of input with minimal user effort and maximum interpretability.

---
