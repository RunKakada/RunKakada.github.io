# 🏗️ CivilGPT
### AI Assistant for Structural Engineering Design and Construction

CivilGPT is a **Retrieval-Augmented Generation (RAG)** application that lets engineers ask natural-language questions about structural codes, design standards, and construction manuals. It retrieves relevant passages from your document library and generates precise, code-cited answers using state-of-the-art LLMs.

---

## 🗂️ Project Structure

```
civilgpt/
├── backend/
│   ├── main.py           ← FastAPI app (routes, CORS, startup)
│   ├── rag_engine.py     ← Core RAG pipeline (ingest → embed → retrieve → LLM)
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
├── frontend/
│   └── src/
│       └── CivilGPT.jsx  ← React chat UI with source cards + settings
├── scripts/
│   └── ingest.py         ← Offline document ingestion CLI
├── data/
│   └── docs/             ← 📁 Put your PDF/DOCX codes here
├── docker-compose.yml
└── README.md
```

---

## ⚙️ System Architecture

```
User Question
      ↓
React Frontend (chat UI)
      ↓
FastAPI Backend  (query router)
      ↓
RAG Orchestrator  (LangChain / LlamaIndex)
    ↙                          ↘
Embedding Model          Civil Eng. Documents
(text-embedding-3-small  (ACI 318, ASCE 7,
 or BGE local)            Eurocode, AISC...)
    ↓                          ↓
Vector Database  ←──────  Ingestion Pipeline
(Chroma / FAISS)          (Chunk → Embed → Index)
    ↓
Top-K Chunks Retrieved
      ↓
LLM (Claude / GPT-4o / Llama 3 / Qwen 2)
      ↓
Structured, Code-Cited Answer  →  Frontend
```

---

## 🚀 Quick Start

### 1. Clone & set up environment

```bash
git clone https://github.com/yourname/civilgpt.git
cd civilgpt

cp backend/.env.example backend/.env
# Edit backend/.env and add your API keys
```

### 2. Add documents

Drop your civil engineering PDFs into `data/docs/`:
- `ACI_318-19.pdf` — Concrete design
- `ASCE_7-22.pdf` — Loads on structures
- `AISC_360-22.pdf` — Steel design
- `Eurocode2_EN1992-1-1.pdf` — EC2 concrete
- Any other standards, design manuals, guides

### 3. Ingest documents (run once)

```bash
cd backend
pip install -r requirements.txt
python ../scripts/ingest.py --folder ../data/docs
```

This chunks all PDFs, generates embeddings, and stores them in `data/chroma_db/`.

### 4a. Start with Docker Compose (recommended)

```bash
docker-compose up --build
```

- Frontend: http://localhost:3000
- Backend API: http://localhost:8000
- API Docs: http://localhost:8000/docs

### 4b. Start manually (development)

**Backend:**
```bash
cd backend
uvicorn main:app --reload --port 8000
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev   # → http://localhost:5173
```

---

## 🤖 LLM Options

| Provider | Model | Setup |
|---|---|---|
| **Claude** (default) | `claude-opus-4-5` | Set `ANTHROPIC_API_KEY` |
| **OpenAI** | `gpt-4o` | Set `OPENAI_API_KEY` |
| **Llama 3** (local) | `llama3` | Install [Ollama](https://ollama.com), run `ollama pull llama3` |
| **Qwen 2** (local) | `qwen2` | Install Ollama, run `ollama pull qwen2` |

Switch provider per-query via the Settings panel in the UI or the `llm_provider` field in the API.

---

## 📡 API Reference

### `POST /api/query`
Ask a structural engineering question.

```json
{
  "question": "What is the minimum slab thickness for a flat plate per ACI 318?",
  "top_k": 5,
  "llm_provider": "claude",
  "stream": false
}
```

**Response:**
```json
{
  "answer": "Per ACI 318-19 Table 8.3.1.1, the minimum thickness for...",
  "sources": [
    { "source": "ACI_318-19.pdf", "page": 84, "text": "...", "score": 0.92 }
  ],
  "tokens_used": 680
}
```

### `POST /api/query/stream`
Same as above but returns Server-Sent Events (SSE) token-by-token.

### `POST /api/ingest`
Re-trigger document ingestion from the server.

```
POST /api/ingest?folder=./data/docs
```

### `GET /api/health`
Returns `{"status": "ok", "engine_ready": true}`.

---

## 🔧 Configuration

| Variable | Default | Description |
|---|---|---|
| `ANTHROPIC_API_KEY` | — | Claude API key |
| `OPENAI_API_KEY` | — | OpenAI key (GPT + embeddings) |
| `EMBEDDING_PROVIDER` | `openai` | `openai` or `huggingface` |
| `VECTOR_DB` | `chroma` | `chroma` or `faiss` |
| `CHROMA_PERSIST_DIR` | `./data/chroma_db` | Where to store the vector index |

---

## 🧩 Extending CivilGPT

- **Add Pinecone**: Replace `Chroma` in `rag_engine.py` with `PineconeVectorStore` from `langchain-pinecone`
- **Add re-ranking**: Insert a `CrossEncoderReranker` after retrieval for better chunk ordering
- **Add memory**: Replace the `chain` with `ConversationalRetrievalChain` to maintain conversation history
- **Add Streamlit UI**: Run `streamlit run app.py` (Streamlit version provided separately)

---

## 📄 License
MIT — use freely, contribute back improvements.
