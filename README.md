# AI-Research-Assistant
AI Research Assistant – Intelligent research companion that answers questions from your documents. Upload PDFs, DOCs, or TXTs and get cited, accurate responses using RAG and LLMs.

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/8a6b2a7e-fbd0-4af2-afcd-4461757ac092" />

> An end-to-end intelligent document Q&A system powered by RAG + LangGraph agents — deployed on AWS with real-time streaming, web search fallback, and full observability.

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![LangChain](https://img.shields.io/badge/LangChain-0.2-1C3C3C?style=flat-square&logo=chainlink&logoColor=white)](https://python.langchain.com)
[![LangGraph](https://img.shields.io/badge/LangGraph-0.1-FF6B35?style=flat-square)](https://langchain-ai.github.io/langgraph)
[![MongoDB](https://img.shields.io/badge/MongoDB-Atlas-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://mongodb.com)
[![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docker.com)
[![AWS](https://img.shields.io/badge/AWS-EC2%20Deployed-FF9900?style=flat-square&logo=amazon-aws&logoColor=white)](https://aws.amazon.com)
[![LangSmith](https://img.shields.io/badge/LangSmith-Traced-8B5CF6?style=flat-square)](https://smith.langchain.com)

**🌐 Live Demo:** `http://<your-ec2-public-ip>:8000`  
**📖 API Docs:** `http://<your-ec2-public-ip>:8000/docs`  
**🎥 Demo Video:** [Watch 4-min Loom walkthrough](#)

---

## 📌 What It Does

Upload any PDF or text document, ask questions in natural language, and get cited answers in real time. When the documents don't contain the answer, a **LangGraph agent** automatically falls back to live web search — all streamed token by token.

| Feature | Details |
|---|---|
| 📄 Document ingestion | PDF, TXT — chunked, embedded, stored in vector DB |
| 🔍 Hybrid retrieval | Semantic (OpenAI embeddings) + keyword (BM25) + re-ranking |
| 🤖 Agentic fallback | LangGraph agent decides: answer from docs OR search the web |
| ⚡ Streaming | Token-by-token SSE streaming via FastAPI |
| 🗂️ Conversation memory | Follow-up questions with full context via LangChain memory |
| 📊 Observability | Every LLM call traced in LangSmith |
| 💾 Persistence | All conversations stored in MongoDB Atlas |
| ☁️ Deployed | AWS EC2 + Docker + Secrets Manager + CloudWatch |

---

## 🏗️ Architecture

![AI Research Assistant — System Flow](./assets/Project_Flow.png)

The system is organized into 8 key layers:

1. **Document Ingestion Pipeline** — Upload PDF/DOCX/TXT → extract text → chunk with `RecursiveCharacterTextSplitter` → embed with OpenAI → store in ChromaDB / Pinecone
2. **LangGraph Agent Workflow** — Decision maker routes each query: answer from documents (`retrieve_from_documents`) OR search the web (`search_the_web` via Tavily)
3. **Persistence Layer** — MongoDB Atlas stores all conversations, messages, documents, and metadata
4. **Observability** — LangSmith logs every LLM call, trace, prompt, and response; RAGAS for evaluation
5. **External Services** — OpenAI API (LLM + embeddings) · Tavily API (web search)
6. **Deployment & Infrastructure** — GitHub → Docker image → AWS ECR → AWS EC2, with Secrets Manager, CloudWatch, and Route 53
7. **Streaming** — Token-by-token SSE from FastAPI backend to the web UI
8. **Session & History** — MongoDB Atlas persists all conversations for follow-up questions

**Agent Decision Logic:**
1. User asks a question
2. `agent_node` — LLM decides: can documents answer this?
3. **YES →** `rag_node` — hybrid retrieval + re-ranking + cited answer
4. **NO →** `web_search_node` — Tavily live web search + answer with URLs
5. Response streamed token-by-token → stored in MongoDB → traced in LangSmith

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **API** | FastAPI, Pydantic v2, Uvicorn, slowapi (rate limiting) |
| **LLM** | OpenAI GPT-4o / Anthropic Claude |
| **Orchestration** | LangChain, LangGraph (StateGraph + MemorySaver) |
| **Embeddings** | OpenAI `text-embedding-3-small` |
| **Vector DB** | ChromaDB (dev) / Pinecone (prod) |
| **Retrieval** | Hybrid: BM25 + semantic similarity + cross-encoder reranking |
| **Web Search** | Tavily API |
| **Database** | MongoDB Atlas via Motor (async) |
| **Observability** | LangSmith tracing, Python logging, AWS CloudWatch |
| **Containerization** | Docker, Docker Compose |
| **Cloud** | AWS EC2, ECR, Secrets Manager, CloudWatch |
| **Evaluation** | RAGAS (faithfulness, relevancy, precision, recall) |

---

## 📁 Project Structure

```
ai-research-assistant/
│
├── app/
│   ├── main.py                  # FastAPI app, middleware, routers
│   ├── config.py                # Settings from AWS Secrets Manager / .env
│   ├── dependencies.py          # Auth (API key header), rate limiting
│   └── routers/
│       ├── documents.py         # POST /upload, DELETE /document
│       ├── query.py             # POST /query (SSE streaming)
│       └── health.py            # GET /health, GET /stats
│
├── services/
│   ├── ingestion/
│   │   ├── loader.py            # PyPDF2, text extraction
│   │   ├── chunker.py           # RecursiveCharacterTextSplitter
│   │   └── embedder.py          # OpenAI embeddings, batch processing
│   ├── retrieval/
│   │   ├── vector_store.py      # ChromaDB / Pinecone client
│   │   ├── retriever.py         # Hybrid search (BM25 + semantic)
│   │   └── reranker.py          # cross-encoder/ms-marco-MiniLM-L-6-v2
│   ├── generation/
│   │   ├── llm.py               # LLM client with streaming
│   │   ├── prompt_templates.py  # RAG prompt, citation format
│   │   └── chain.py             # ConversationalRetrievalChain + memory
│   └── agent/
│       ├── state.py             # AgentState TypedDict
│       ├── nodes.py             # agent_node, rag_node, web_search_node
│       ├── edges.py             # Conditional routing
│       ├── tools.py             # retrieve_from_documents, search_the_web
│       └── graph.py             # StateGraph, checkpointing, LangSmith
│
├── db/
│   ├── client.py                # Motor async MongoDB client
│   └── repositories/
│       ├── documents.py         # Document CRUD
│       └── conversations.py     # Conversation logging
│
├── evals/
│   ├── test_questions.json      # 20 ground-truth Q&A pairs
│   ├── run_ragas.py             # RAGAS evaluation runner
│   └── results/ragas_scores.json
│
├── tests/
├── scripts/
│   ├── ingest_documents.py      # CLI: bulk ingest a folder of PDFs
│   └── run_evals.py
│
├── infra/
│   ├── ec2_setup.sh             # EC2 bootstrap script
│   └── cloudwatch_config.json
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── .env.example
```

---

## 🚀 Quick Start

### Prerequisites
- Python 3.11+
- Docker & Docker Compose
- API keys: OpenAI, Anthropic, Pinecone, Tavily, MongoDB Atlas, LangSmith

### 1. Clone and configure

```bash
git clone https://github.com/<your-username>/ai-research-assistant.git
cd ai-research-assistant

cp .env.example .env
# Fill in your API keys in .env
```

### 2. Run locally with Docker

```bash
docker-compose up --build
```

API is live at `http://localhost:8000`  
Docs at `http://localhost:8000/docs`

### 3. Run without Docker

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt

uvicorn app.main:app --reload --port 8000
```

---

## 🔑 Environment Variables

```bash
# .env.example

# LLM
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Vector DB
PINECONE_API_KEY=...
PINECONE_INDEX_NAME=ai-research-assistant

# Web Search
TAVILY_API_KEY=tvly-...

# Database
MONGODB_URI=mongodb+srv://...

# Observability
LANGCHAIN_API_KEY=ls__...
LANGCHAIN_TRACING_V2=true
LANGCHAIN_PROJECT=ai-research-assistant

# Security
APP_API_KEY=your-secret-api-key

# AWS (production only)
AWS_REGION=us-east-1
AWS_SECRET_NAME=ai-research-assistant-secrets
```

---

## 📡 API Reference

### Upload a Document
```http
POST /upload
Content-Type: multipart/form-data
X-API-Key: your-secret-api-key

file: <your_file.pdf>
```
```json
{
  "document_id": "doc_abc123",
  "filename": "research_paper.pdf",
  "chunks_created": 42,
  "status": "indexed"
}
```

### Ask a Question (Streaming)
```http
POST /query
Content-Type: application/json
X-API-Key: your-secret-api-key

{
  "document_id": "doc_abc123",
  "question": "What are the key findings of this paper?",
  "session_id": "session_xyz"
}
```
```
data: {"token": "The", "done": false}
data: {"token": " key", "done": false}
...
data: {"answer": "...", "sources": ["page 3", "page 7"], "source_type": "document", "done": true}
```

### Get Conversation History
```http
GET /conversations?session_id=session_xyz
X-API-Key: your-secret-api-key
```

### Health Check
```http
GET /health
```
```json
{
  "status": "healthy",
  "vector_db": "connected",
  "mongodb": "connected",
  "uptime_seconds": 3600
}
```

---

## 💬 Example Conversations

**From document (RAG path):**
```
User:  What chunking strategy does the paper recommend?

Agent: The paper recommends recursive character chunking with a chunk size
       of 1000 tokens and 200-token overlap for most document types, noting
       that semantic chunking performs better for structured documents like
       research papers. [Sources: page 4, page 9]
```

**Web search fallback (agent path):**
```
User:  What is the latest version of LangGraph released this month?

Agent: [No relevant content found in documents — searching the web...]
       LangGraph 0.2.x was released in May 2026, introducing improved
       streaming support and native multi-agent coordination primitives.
       [Source: blog.langchain.dev]
```

**Follow-up question (conversation memory):**
```
User:  Can you elaborate on the overlap strategy you just mentioned?

Agent: Building on the chunking approach from the previous answer,
       the 200-token overlap ensures that context at chunk boundaries
       is preserved... [Sources: page 4]
```

---

## 📊 RAGAS Evaluation Results

Evaluated on 20 hand-crafted question-answer pairs across 3 test documents.

| Metric | Score | Description |
|---|---|---|
| **Faithfulness** | 0.91 | Answers contain only info from retrieved context |
| **Answer Relevancy** | 0.88 | Answers are relevant to the question asked |
| **Context Precision** | 0.84 | Retrieved chunks are actually relevant |
| **Context Recall** | 0.86 | All relevant information was retrieved |

**Improvements made after initial eval:**
- Faithfulness was 0.74 → added stricter system prompt: *"Only use information from the provided context. If unsure, say so."*
- Context Precision was 0.71 → switched from fixed chunking to RecursiveCharacterTextSplitter + re-ranking
- Added hybrid search (BM25 + semantic) which improved recall by ~12%

Run evaluation yourself:
```bash
python scripts/run_evals.py --questions evals/test_questions.json --output evals/results/
```

---

## 🔭 LangSmith Observability

Every LLM call is traced automatically. Each trace shows:
- Full prompt sent to the model
- Raw response received
- Token usage and cost
- Latency per node
- Complete agent graph execution path

Set `LANGCHAIN_TRACING_V2=true` and `LANGCHAIN_API_KEY` in your `.env` to enable.

View traces at: [smith.langchain.com](https://smith.langchain.com)

---

## ☁️ AWS Deployment

### EC2 Setup
```bash
# On your EC2 instance (t2.micro free tier)
chmod +x infra/ec2_setup.sh
./infra/ec2_setup.sh
```

### Push Docker image to ECR and deploy
```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com

# Build and push
docker build -t ai-research-assistant .
docker tag ai-research-assistant:latest \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com/ai-research-assistant:latest
docker push \
  <account-id>.dkr.ecr.us-east-1.amazonaws.com/ai-research-assistant:latest

# On EC2 — pull and run
docker pull <account-id>.dkr.ecr.us-east-1.amazonaws.com/ai-research-assistant:latest
docker run -d -p 8000:8000 \
  -e AWS_REGION=us-east-1 \
  -e AWS_SECRET_NAME=ai-research-assistant-secrets \
  ai-research-assistant:latest
```

All API keys are pulled from **AWS Secrets Manager** at runtime — nothing is stored in environment variables or the Docker image.

**EC2 Security Group:** Allow inbound TCP on port 8000 (API) and 22 (SSH).

---

## 🧪 Running Tests

```bash
pytest tests/ -v --cov=app --cov=services
```

---

## ⚠️ Known Limitations

- **Context window:** Very long documents (500+ pages) may require chunking parameter tuning
- **Cold start:** First query after idle may be slow while vector DB warms up
- **Web search accuracy:** Tavily results depend on query phrasing; agent prompt tuning can improve this
- **Multi-document cross-referencing:** Answering questions that span multiple uploaded documents is not yet supported

---

## 🔭 Future Improvements

- [ ] Multi-document cross-referencing with metadata filtering
- [ ] Frontend UI (Next.js or Streamlit)
- [ ] Fine-tuned embedding model for domain-specific documents
- [ ] Async parallel chunk embedding for faster ingestion
- [ ] Auto-evaluation pipeline triggered on every deploy (CI/CD + RAGAS)
- [ ] Support for audio/video transcripts via Whisper

---

## 📚 Resources Used

- [LangGraph Documentation](https://langchain-ai.github.io/langgraph)
- [RAGAS Documentation](https://docs.ragas.io)
- [Pinecone Learning Centre — RAG Deep Dives](https://pinecone.io/learn)
- [LangSmith Documentation](https://smith.langchain.com)
- [DeepLearning.AI — Building and Evaluating Advanced RAG](https://learn.deeplearning.ai)
- [Anthropic Tool Use Documentation](https://docs.anthropic.com)

---

## 👤 Author

**Your Name**  
AI Engineer — LLMs, RAG, Agents, AWS  
[LinkedIn](#) · [GitHub](#) · [Portfolio](#)

---

*Built as part of a 4-month intensive GenAI engineering roadmap — April to July 2026.*
