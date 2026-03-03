# AI Dietician — RAG-Powered Conversational Nutrition Agent

A stateful, RAG-powered conversational AI dietician using a small CPU-optimized LLM (e.g. Phi-3 Mini) and a curated nutrition knowledge base. Built for mid-evaluation milestones: FastAPI backend, RAG pipeline (LangChain + ChromaDB), and a Next.js chat UI that shows **retrieved sources** for every answer.

## Quick start (local)

### 1. Prerequisites

- Python 3.10+
- [Ollama](https://ollama.ai) installed and running; pull the Q4 quantized model: `ollama pull phi3:3.8b-mini-4k-instruct-q4_K_M`
- Node 18+ (for Next.js frontend)

### 2. Build the vector database (one-time)

From the project root:

```bash
pip install -r requirements.txt
python scripts/build_vectordb.py
```

This merges `data/nutrition_data.json` and `usda_data.json` (USDA as natural-language strings), chunks at ~500 chars, and creates/updates `./chroma_db`. Re-running skips duplicates.

### 3. Run the API

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
# If port 8000 is in use: uvicorn app.main:app --host 0.0.0.0 --port 8001
```

- API: http://localhost:8000  
- Docs: http://localhost:8000/docs  
- Health: http://localhost:8000/health  

**Speed:** Replies are tuned for CPU (short context, 2 RAG docs, max 256 tokens). To allow longer answers set `OLLAMA_NUM_PREDICT=512`. If you still get timeouts, set `OLLAMA_TIMEOUT=180`.

### 4. Run the Next.js chat UI

```bash
cd frontend
npm install
npm run dev
```

Open http://localhost:3000. Set `NEXT_PUBLIC_API_URL=http://localhost:8000` in `frontend/.env.local` if the API is on a different host.

## Docker

### API only

```bash
# Build vector DB first (on host, so it can be mounted)
pip install -r requirements.txt
python scripts/build_vectordb.py

docker compose up --build
```

API: http://localhost:8000. Ollama must be running on the host (e.g. `ollama serve`); the container uses `host.docker.internal:11434`.

### With frontend

Uncomment the `frontend` service in `docker-compose.yml`, then:

```bash
docker compose up --build
```

- API: http://localhost:8000  
- Chat UI: http://localhost:3000  

## Project structure

```
FYP/
├── app/
│   ├── main.py          # FastAPI app, /chat, /health, CORS
│   └── rag_engine.py    # ChromaDB build + load from nutrition_data.json
├── data/
│   └── nutrition_data.json
├── frontend/            # Next.js chat UI with retrieval display
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx     # Chat + "Retrieved sources" section
│   │   └── globals.css
│   └── package.json
├── scripts/
│   └── build_vectordb.py
├── chroma_db/           # Created by build_vectordb.py
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
└── README.md
```

## API

- **POST /chat**  
  Body: `{ "user_input": "How much protein per meal?", "chat_history": [] }`  
  Returns: `{ "response": "...", "sources": [{ "id", "source", "topic", "category", "snippet" }], "model": "phi3:mini" }`

- **GET /health**  
  Returns API and vector DB status.

## Testing

With the API running and Ollama with `phi3:mini` available:

```bash
pip install -r requirements.txt
python scripts/test_app.py
```

This checks: root, health (vector_db + ollama), POST /chat (one RAG+LLM call), GET /retrieve (RAG-only), and optional chat with history. On CPU, the first chat can take 1–3 minutes; the second may be skipped if it times out.

## Mid-evaluation checklist

- [x] Dataset curation (nutrition_data.json)
- [x] Git, Docker, Ollama setup
- [x] Small CPU-optimized LLM (phi3:mini) via Ollama
- [x] RAG pipeline: LangChain + ChromaDB, embeddings from nutrition data
- [x] FastAPI backend with /chat and CORS
- [x] Next.js chat UI with loading state and **retrieved sources** displayed per reply
