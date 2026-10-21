# CodeMind — AI Coding Assistant

> Chat with any codebase. Index a GitHub repo or upload a ZIP, then ask questions in plain English and get answers with **exact file references and line numbers.**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Deploy%20on%20Vercel-000000?logo=vercel)](https://your-backend.vercel.app)
[![Frontend](https://img.shields.io/badge/Frontend-React%20%2B%20Vite-61DAFB?logo=react)](https://github.com/Ayesha-Zafar-03/codemind.ai)
[![Backend](https://img.shields.io/badge/Backend-FastAPI-009688?logo=fastapi)](https://github.com/Ayesha-Zafar-03/codemind.ai)

**CodeMind** is a Retrieval-Augmented Generation (RAG) code assistant — a "mini-Copilot" — that turns any repository into a searchable, conversational knowledge base. It parses source code into functions and classes, embeds it into vectors, and lets you semantically search and chat with it, with every answer cited to the precise file and line.

---

## ✨ Features

- **Repo ingestion** — index any public GitHub repository by URL, or upload a ZIP archive.
- **AST-based code chunking** — splits code into functions/classes across **Python, JavaScript, TypeScript, JSX, TSX, Go, and Java** (not naive text splitting).
- **Semantic search** — natural-language search over the codebase using vector similarity (Qdrant ANN).
- **RAG chat with citations** — ask questions and get answers grounded in retrieved code, with file + line references (Groq / LLaMA).
- **Dependency graph** — visualizes import relationships between files.
- **Code viewer** — Monaco-based file explorer and viewer.
- **Serverless by design** — FastAPI backend deployed as a Vercel serverless function; React/Vite frontend on Vercel.

---

## 🖼️ Screenshots / Demo

> 📌 Replace these placeholders with real screenshots or a short Loom video (recruiters engage far more with a 30-second demo).

| Index a repo | Semantic search | Chat with citations |
|---|---|---|
| `docs/screenshot-index.png` | `docs/screenshot-search.png` | `docs/screenshot-chat.png` |

**Live demo:** https://codemind-mu.vercel.app/

---

## 🧠 How it works

```
 GitHub URL / ZIP
        │
        ▼
┌───────────────────────────┐
│  Ingest & Parse (FastAPI) │  1. Download tarball OR extract ZIP
│                           │  2. AST-parse files → functions/classes
└───────────┬───────────────┘  3. Store raw code + chunks in MongoDB
            │
            ▼
┌───────────────────────────┐
│  Embed (Nomic)            │  4. Generate vectors (nomic-embed-text)
│  + Store (Qdrant)         │  5. Upsert vectors into Qdrant
└───────────┬───────────────┘
            │
            ▼
┌───────────────────────────┐
│  Query (Frontend)         │  6. User asks → embed query
│                           │  7. Vector search → top chunks
│                           │  8. Groq LLM answers w/ citations
└───────────────────────────┘
```

### Architecture

```
┌──────────────┐      ┌────────────────────┐      ┌─────────────────────┐
│  React/Vite  │─────▶│  FastAPI (Vercel   │─────▶│  MongoDB Atlas      │
│  Frontend    │      │  Serverless Func)  │      │  (code + chunks)    │
└──────────────┘      └─────────┬──────────┘      └─────────────────────┘
                                 ├──────────────▶ Qdrant (vector DB)
                                 ├──────────────▶ Nomic (embeddings API)
                                 └──────────────▶ Groq (LLaMA chat LLM)
```

---

## 🛠️ Tech Stack

| Layer | Technologies |
|---|---|
| **Frontend** | React 19, Vite, TailwindCSS, Monaco Editor, Axios, Lucide icons |
| **Backend** | FastAPI, Python 3.12, Motor (async MongoDB), Pydantic |
| **Vector DB** | Qdrant (similarity search + storage) |
| **Embeddings** | Nomic `nomic-embed-text` |
| **LLM** | Groq (LLaMA) for RAG chat |
| **Infra** | Vercel (serverless functions + static hosting) |

---

## 📁 Project Structure

```
codemind.ai/
├── frontend/                 # React + Vite app
│   ├── src/
│   │   ├── api/client.js     # Axios API client (VITE_API_URL)
│   │   ├── components/       # UploadScreen, ChatPanel, SearchPanel, ...
│   │   └── App.jsx
│   ├── package.json
│   └── vite.config.js
├── backend/                  # FastAPI app (deployed as Vercel function)
│   ├── app/
│   │   ├── main.py           # FastAPI entrypoint (ASGI app)
│   │   ├── api/routes/       # repos, search, chat routers
│   │   ├── core/config.py    # settings (env-driven)
│   │   ├── db/database.py    # MongoDB connection
│   │   ├── services/
│   │   │   ├── ast_parser/   # language-aware code chunking
│   │   │   ├── embeddings/   # Nomic embedder + Qdrant client
│   │   │   ├── ingestion/    # GitHub/ZIP ingestion pipeline
│   │   │   └── rag/          # RAG chat service
│   ├── requirements.txt
│   └── vercel.json           # serverless config + rewrites
├── Dockerfile                # alternative: long-running host (Render/Railway)
└── render.yaml
```

---

## 🚀 Getting Started (Local)

### Prerequisites
- Node.js 18+ and Python 3.12+
- A MongoDB instance (local or [MongoDB Atlas](https://www.mongodb.com/atlas))
- A Qdrant instance ([Qdrant Cloud](https://cloud.qdrant.io) free tier)
- API keys: [Nomic](https://app.nomic.ai) and [Groq](https://console.groq.com)

### 1. Backend
```bash
cd backend
python -m venv venv && source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt

# create backend/.env (see Environment Variables below)
cp backend/.env.example backend/.env   # then fill in real values

uvicorn app.main:app --reload --port 8000
```

### 2. Frontend
```bash
cd frontend
npm install
# frontend/.env  ->  VITE_API_URL=http://127.0.0.1:8000
npm run dev
```
Open http://localhost:5173.

---

## 🔐 Environment Variables

Copy `backend/.env.example` → `backend/.env` and fill in:

| Variable | Description | Required |
|---|---|---|
| `MONGODB_URL` | MongoDB connection string | ✅ yes |
| `DATABASE_NAME` | DB name (default `ai_coding_assistant`) | no |
| `QDRANT_URL` | Qdrant cluster URL | ✅ yes |
| `QDRANT_API_KEY` | Qdrant API key (if cloud) | ✅ if cloud |
| `QDRANT_COLLECTION` | Collection name (default `code_chunks`) | no |
| `NOMIC_API_KEY` | Nomic embeddings key | ✅ yes |
| `GROQ_API_KEY` | Groq LLM key (chat) | ✅ for chat |
| `GITHUB_TOKEN` | Higher GitHub rate limits / private repos | ⚠️ optional |
| `ALLOWED_ORIGINS` | Comma-separated CORS origins (prod) | ✅ in prod |

> ⚠️ **Never commit `.env` files.** Keep secrets in Vercel project settings (or your local `.env`, which is git-ignored).

---

## 🌐 API Endpoints

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/health` | Health check |
| `POST` | `/api/repos/ingest` | Index a GitHub repo by URL |
| `POST` | `/api/repos/upload` | Upload & index a ZIP file |
| `GET` | `/api/repos/` | List indexed repos |
| `GET` | `/api/repos/{id}/status` | Ingestion status |
| `GET` | `/api/repos/{id}/files` | List files in a repo |
| `GET` | `/api/repos/{id}/graph` | Dependency graph |
| `POST` | `/api/search/semantic` | Semantic code search |
| `POST` | `/api/chat/message` | RAG chat with citations |

---

## ☁️ Deployment (Vercel)

This repo is designed for **two Vercel projects**:

### Backend project (root = `backend/`)
- Vercel auto-detects the Python function from `vercel.json`.
- `vercel.json` rewrites all `/api/*` traffic to the FastAPI serverless function.
- Set **all backend env vars** (above) in Vercel project settings.
- `maxDuration` is set to 300s (Vercel Pro). On Hobby it must be ≤ 60s.

### Frontend project (root = `frontend/`)
- Framework: Vite. Build: `vite build`, output: `dist`.
- Set `VITE_API_URL` = your backend URL (e.g. `https://<backend>.vercel.app`).
- If backend and frontend are the **same** Vercel project, leave `VITE_API_URL` empty (same-origin).

> **Note on limits:** Vercel serverless functions cap request bodies (Hobby ≈ 4.5 MB, Pro = 25 MB), so very large ZIP uploads should use a direct object-storage (presigned URL) path. Long-running indexing is bounded by `maxDuration`.

---

## 🧭 Roadmap

- [ ] Direct-to-object-storage uploads for large repos (500 MB+)
- [ ] Streaming chat responses
- [ ] More languages (Rust, C++, Kotlin)
- [ ] Multi-repo workspaces
- [ ] Auth & private repos

---

## 👤 Author

**Mariam Gilani**
- GitHub: [@MariamGilani4347](https://github.com/MariamGilani4347) 
- Project: [codemind.ai](https://github.com/MariamGilani4347/codemind.ai)

---

## 📄 License

This project is provided for demonstration and portfolio purposes.
