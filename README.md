
# brand-guardian

Automated video compliance QA pipeline for auditing advertising and influencer
content against regulatory standards. Orchestrated by LangGraph, the system
ingests video multimodally, retrieves applicable compliance rules via RAG, and
reasons over them with GPT-4o to produce structured JSON compliance reports.

---

## How it works

1. A video URL is submitted to the API
2. Azure Video Indexer extracts transcripts and on-screen text (OCR)
3. Azure AI Search retrieves relevant compliance rules via semantic search
over indexed regulatory documents (Azure OpenAI Embeddings)
5. Azure OpenAI (GPT-4o) synthesizes the retrieved context and detects violations
6. A structured JSON compliance report is returned

LangGraph orchestrates steps 2-5 as a stateful graph. LangSmith traces each
node for LLM workflow inspection and optimization. Azure Application Insights
provides production-grade telemetry, structured logging, and performance monitoring.

---

## Tech stack

- **Orchestration:** LangGraph, LangSmith
- **Multimodal ingestion:** Azure Video Indexer
- **Retrieval:** Azure AI Search, Azure OpenAI Embeddings
- **Reasoning:** Azure OpenAI (GPT-4o)
- **Observability:** LangSmith, Azure Application Insights
- **API:** FastAPI
- **Containerization:** Docker

---

## Project structure

```
brand-guardian/
├── main.py                           # Application entry point
├── pyproject.toml
└── backend/
    ├── Dockerfile
    ├── data/                         # Compliance reference documents
    │   ├── 1001a-influencer-guide-508_1.pdf
    │   └── youtube-ad-specs.pdf
    ├── scripts/
    │   └── index_documents.py        # Indexes compliance PDFs into Azure AI Search
    └── src/
        ├── api/
        │   ├── server.py             # FastAPI server
        │   └── telemetry.py          # Azure Application Insights setup
        ├── graph/
        │   ├── nodes.py              # LangGraph node definitions
        │   ├── state.py              # Graph state schema
        │   └── workflow.py           # LangGraph workflow
        └── services/
            └── video_indexer.py      # Azure Video Indexer client
```

---

## Setup

### Prerequisites

- Python 3.10+
- Azure subscription with:
  - Azure OpenAI (GPT-4o deployment + embedding model)
  - Azure Video Indexer
  - Azure AI Search
  - Azure Application Insights
- LangSmith account

### Install dependencies

```bash
pip install -e .
```

### Environment variables

Create a `.env` file at the project root:

```env
# Azure OpenAI
AZURE_OPENAI_ENDPOINT=
AZURE_OPENAI_API_KEY=
AZURE_OPENAI_DEPLOYMENT=gpt-4o
AZURE_OPENAI_EMBEDDING_DEPLOYMENT=text-embedding-ada-002

# Azure Video Indexer
AZURE_VIDEO_INDEXER_ACCOUNT_ID=
AZURE_VIDEO_INDEXER_SUBSCRIPTION_KEY=
AZURE_VIDEO_INDEXER_LOCATION=

# Azure AI Search
AZURE_AI_SEARCH_ENDPOINT=
AZURE_AI_SEARCH_KEY=
AZURE_AI_SEARCH_INDEX_NAME=compliance-rules

# Azure Application Insights
APPLICATIONINSIGHTS_CONNECTION_STRING=

# LangSmith
LANGSMITH_API_KEY=
LANGCHAIN_PROJECT=brand-guardian
LANGCHAIN_TRACING_V2=true
```

### Index compliance documents

Run once to embed and index the regulatory PDFs into Azure AI Search:

```bash
python backend/scripts/index_documents.py
```

---

## Usage

### Run locally

```bash
python main.py
```

### Docker

```bash
docker build -t brand-guardian ./backend
docker run --env-file .env -p 8000:8000 brand-guardian
```

---

## Output

The pipeline returns a structured JSON compliance report:

```json
{
  "video_id": "3CW8FDgqvq",
  "compliance_status": "VIOLATION_DETECTED",
  "violations": [
    {
      "rule": "FTC Endorsement Guide 255.5",
      "description": "Missing disclosure of material connection between creator and brand",
      "timestamp": "00:01:23",
      "excerpt": "Check out this product, I absolutely love it..."
    }
  ],
  "summary": "1 violation detected. Sponsored content disclosure absent at 00:01:23."
}
```

---

## Observability

**LangSmith** traces every LangGraph node, exposing prompt inputs, LLM outputs,
and latency per step for workflow optimization.

**Azure Application Insights** provides production telemetry including structured
logging, request tracing, and real-time performance dashboards.
