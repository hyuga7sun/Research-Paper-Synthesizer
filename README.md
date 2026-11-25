# PaperSynth: Multi-Paper Literature Review & Citation Matrix Engine

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688.svg?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED.svg?logo=docker&logoColor=white)](https://docs.docker.com/compose/)

**PaperSynth** is an open-core, production-grade RAG and structured extraction service that ingests multiple academic PDFs simultaneously, cross-references their methodologies and findings, and outputs a verifiable literature review matrix with page-level inline citations.

Built for researchers, R&D analysts, and engineers who need to synthesize dozens of papers into structured Markdown, CSV, or JSON reports without hallucinations.

---

## Key Features

* **Layout-Aware PDF Parsing:** Uses MIT-licensed `Docling` / `pypdfium2` to cleanly extract multi-column text, section headers, and tables while preserving exact page numbers.
* **Cross-Paper Metadata RAG:** Indexes document chunks in `pgvector` with paper IDs, titles, authors, and sections to retrieve contrasting claims across multiple studies.
* **Strict Structured Synthesis:** Uses Pydantic schemas with LLM structured outputs to guarantee deterministic comparison matrices (Datasets, Baselines, Key Metrics, Findings, Limitations, and Research Gaps).
* **Verifiable Page Citations:** Every synthesized claim includes inline source references (`[Paper Title, p. 4]`) grounded directly in retrieved context.
* **Production-Ready Architecture:** Async background processing (`ARQ` + `Redis`), tenant-isolated vector namespaces, API key authentication, rate limiting, and automated `pytest` accuracy evaluations.

---

## Architecture

```
+-------------+       +------------------+       +--------------------+
|  Client /   | ----> |   FastAPI REST   | ----> |   Redis Job Queue  |
|  Web UI     | <---- |  (Auth & Limits) | <---- |   & Status Store   |
+-------------+       +------------------+       +--------------------+
                                                           |
                                                           v
+-------------+       +------------------+       +--------------------+
|  LLM API    | <---> |  ARQ Background  | ----> |  Docling Layout    |
| (Structured)|       |      Worker      |       |     PDF Parser     |
+-------------+       +------------------+       +--------------------+
                               |
                               v
                      +------------------+
                      |   PostgreSQL +   |
                      |     pgvector     |
                      +------------------+
```

---

## Tech Stack (100% Commercial-Friendly)

Every core dependency is licensed under **MIT**, **Apache 2.0**, or **BSD**, allowing unrestricted commercial hosting and enterprise adoption:

* **API Framework:** FastAPI + Pydantic v2 (MIT)
* **PDF Parsing:** Docling / pypdfium2 (MIT / Apache 2.0)
* **Database & Vector Store:** PostgreSQL 16 + `pgvector` (PostgreSQL License)
* **Async Queue:** ARQ + Redis (MIT / BSD)
* **LLM Orchestration:** OpenAI / Google Gemini SDK with Structured Outputs (Apache 2.0 / MIT)
* **Evaluations:** `pytest` + golden dataset assertions (MIT)

---

## Quick Start

### Prerequisites
* Docker & Docker Compose
* An OpenAI or Google Gemini API key

### 1. Clone and Configure
```bash
git clone https://github.com/hyuga7sun/research-paper-synthesizer.git
cd research-paper-synthesizer
cp .env.example .env
```

Edit `.env` and add your API credentials:
```env
LLM_PROVIDER=openai
OPENAI_API_KEY=sk-...
API_SECRET_KEY=dev-secret-key-change-in-prod
DATABASE_URL=postgresql+asyncpg://postgres:postgres@db:5432/papersynth
REDIS_URL=redis://redis:6379/0
```

### 2. Launch Services
```bash
docker compose up --build -d
```

The API will be live at `http://localhost:8000` with interactive OpenAPI docs at `http://localhost:8000/docs`.

---

## API Usage

### 1. Submit a Batch of Research Papers
```bash
curl -X POST "http://localhost:8000/v1/synthesize" \
  -H "X-API-Key: dev-secret-key-change-in-prod" \
  -F "files=@paper_1.pdf" \
  -F "files=@paper_2.pdf" \
  -F "files=@paper_3.pdf" \
  -F "focus_query=Compare model architectures, evaluation datasets, and reported latency trade-offs."
```

**Response (`202 Accepted`):**
```json
{
  "job_id": "job_8f92a1c4",
  "status": "queued",
  "paper_count": 3
}
```

### 2. Poll Job Status & Retrieve Matrix
```bash
curl -X GET "http://localhost:8000/v1/jobs/job_8f92a1c4?format=json" \
  -H "X-API-Key: dev-secret-key-change-in-prod"
```

**Example Output:**
```json
{
  "job_id": "job_8f92a1c4",
  "status": "completed",
  "synthesis": {
    "executive_summary": "Across all three studies, hybrid sparse-dense retrieval outperforms pure dense embeddings on domain-specific benchmarks by 6.4% to 11.2% [Paper 1, p. 5; Paper 3, p. 8].",
    "comparison_matrix": [
      {
        "paper_title": "Hybrid Retrieval at Scale",
        "methodology": "BM25 + ColBERTv2 late interaction",
        "datasets": ["BEIR", "SciFact"],
        "key_metrics": "nDCG@10: 0.742 (+8.1% over baseline)",
        "limitations": "Requires 3.2x index storage overhead [p. 9]"
      }
    ],
    "identified_research_gaps": [
      "None of the evaluated frameworks benchmark multi-hop reasoning latency under concurrent multi-tenant load [Paper 1, p. 11; Paper 2, p. 7]."
    ]
  }
}
```

---

## Project Structure

```text
papersynth/
├── app/
│   ├── api/               # Versioned FastAPI routes, auth, and rate limiting
│   ├── core/              # Config, structured logging, and security utilities
│   ├── db/                # SQLAlchemy models, migrations, and pgvector queries
│   ├── services/
│   │   ├── parser.py      # Layout-aware PDF extraction with page metadata
│   │   ├── retriever.py   # Multi-document hybrid/vector retrieval
│   │   └── synthesizer.py # Pydantic-enforced cross-paper LLM synthesis
│   └── workers/           # ARQ background worker definitions
├── evals/
│   ├── dataset/           # Sample academic PDFs and expected ground truth
│   └── test_accuracy.py   # Automated citation & schema faithfulness tests
├── docker-compose.yml
├── Dockerfile
├── pyproject.toml
└── README.md
```

---

## Running Evaluations & Tests

PaperSynth includes an automated evaluation suite to verify schema compliance, citation grounding, and extraction accuracy before deployment:

```bash
docker compose exec api pytest evals/ -v
```

---

## Deployment & Commercial Hosting

This repository contains the complete open-core engine licensed under **Apache 2.0**. You can self-host it freely on any VPS, AWS ECS, or Railway/Render instance using the provided `Dockerfile`.

For production multi-tenant SaaS deployments:
1. Set `ENVIRONMENT=production` and rotate `API_SECRET_KEY`.
2. Configure managed PostgreSQL with `pgvector` enabled (e.g., Supabase, Neon, or AWS RDS).
3. Connect usage metering webhooks (`app/api/webhooks.py`) to Stripe for per-page or subscription billing.

---

## License

Licensed under the [Apache License, Version 2.0](LICENSE). You are free to use, modify, distribute, and build commercial services on top of this codebase.
