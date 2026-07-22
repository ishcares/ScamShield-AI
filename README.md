# ScamShield AI Agent 🛡️
### *Autonomous Payments Fraud & Social Engineering Detection Pipeline*

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Google ADK](https://img.shields.io/badge/Google%20ADK-Agent%20Builder-4285F4?logo=google-cloud)](https://cloud.google.com/products/agent-builder)
[![MongoDB MCP](https://img.shields.io/badge/Partner-MongoDB%20Atlas%20MCP-00ED64?logo=mongodb)](https://www.mongodb.com/atlas)
[![Gemini 2.5 Flash](https://img.shields.io/badge/AI-Gemini%202.5%20Flash-8B5CF6?logo=google)](https://deepmind.google/technologies/gemini/)

---

> **System Overview:** ScamShield AI is an autonomous, multi-step agentic pipeline designed to detect payment fraud, recruitment-based social engineering, phishing, and digital identity impersonation. The system processes suspicious text and transaction screenshots, executes context-aware threat intelligence lookups, calculates a weighted risk index, and generates structured security event reports in under 10 seconds.

*Note: I designed the system architecture, routing pipelines, and E2E integration, utilizing AI-assisted execution for backend service components and frontend interface layouts.*

---

## 🌐 Live Deployments

| Service | URL |
|---|---|
| **Frontend** (React + Vite on Vercel) | https://scamshield-ai-kappa.vercel.app/ |
| **Backend API** (Express on Render) | https://scamshield-ai-p4ci.onrender.com/ |
| **ADK Agent Bridge** (Python FastAPI) | https://scamshield-agent.onrender.com/health |
| **Agent Status** | https://scamshield-ai-p4ci.onrender.com/api/agent-status |
| **Health Check** | https://scamshield-ai-p4ci.onrender.com/health |

---



## 🏗️ Architecture

```
[User Input: Text / WhatsApp Screenshot]
          │
          ▼
[React Frontend] ──POST /api/analyze──► [Express.js Backend]
                                                │
                                                ▼
                                    ┌───────────────────────┐
                                    │   agentService.js     │
                                    │  (HTTP Bridge Client) │
                                    └──────────┬────────────┘
                                               │ POST /analyze
                                               ▼
                              ┌─────────────────────────────────┐
                              │   agent.py (FastAPI + Google ADK)│
                              │   Agent: ScamShieldAI_Agent      │
                              │   Model: gemini-2.5-flash         │
                              │   Tools: MongoDB MCP Server ──►  │
                              │          $vectorSearch            │
                              │          insert-many              │
                              │          aggregate                │
                              └─────────────┬───────────────────┘
                                            │ JSON result
                                            ▼
                              [6-Step Streaming NDJSON Pipeline]
                                            │
                ┌───────────────────────────┼───────────────────────────┐
                │                           │                           │
          Step 1: Entity             Step 2: MongoDB            Step 3: Trust
          Extraction                 MCP Vector Search          Score Index
          (OCR + ADK)               (Partner Integration)      (Weighted algo)
                │                           │                           │
          Step 4: AI                 Step 5: Persist            Step 6: FIR
          Verdict                    to Atlas via MCP           Template
          (CRITICAL/HIGH/…)          insert-many               cybercrime.gov.in
```

---

## 🤖 Core Agent Architecture (Multi-Step Reasoning & Tool Execution)

Unlike passive chatbots, ScamShield operates as a true autonomous agent by executing a deterministic, stateful decision loop:

| Production Agent Standard | System Implementation |
|---|---|
| **Multi-step reasoning** | 6-step discrete pipeline: Entity extraction → vector ledger lookup → trust index scoring → risk classification → persistence → escalation |
| **Tool Calling & MCP Integration** | Intercepts reasoning steps to invoke MongoDB MCP `aggregate` (vector search) + `insert-many` (audit-trail persistence) |
| **Google Cloud Agent Builder** | Standardized on `google.adk.agents.Agent` with `Runner` + `InMemorySessionService` for session-state isolation |
| **Partner MCP integration** | Runs `mongodb-mcp-server` via stdio JSON-RPC, spawned dynamically by the ADK runtime environment |
| **Resilient High-Uptime Design** | Automatic failover to direct Mongoose queries if the bridge or MCP service experiences cold starts or latency spikes |
| **Real-time UX Control** | NDJSON streaming pipeline exposes live step-by-step security execution to the frontend dashboard |

---

## 🔌 MongoDB Partner Track — MCP Integration Details

**Partner:** MongoDB Atlas MCP Server (`mongodb-mcp-server`)

The Model Context Protocol (MCP) server is registered directly inside the agent's toolset as an active capability, exposing database query primitives directly to the model's reasoning loop.

1. **Vector-Based Threat Lookup (MCP `aggregate` tool):**
   ```json
   {
     "tool": "aggregate",
     "args": {
       "db": "scamshield",
       "collection": "scamreports",
       "pipeline": [
         { "$vectorSearch": {
             "index": "scam_embedding_index",
             "path": "embedding",
             "queryVector": [...1536 dims...],
             "numCandidates": 50,
             "limit": 3
         }},
         { "$project": { "investigationSummary": 1, "riskLevel": 1, "score": { "$meta": "vectorSearchScore" }}}
       ]
     }
   }
   ```

2. **Telemetry Logging & Persistence (MCP `insert-many` tool):**
   ```json
   {
     "tool": "insert-many",
     "args": {
       "db": "scamshield",
       "collection": "scamreports",
       "documents": [{ ...full audit report with 1536-dim embedding... }]
     }
   }
   ```

---

## 💡 Key Architectural Features

| Feature | Technical Implementation |
|---|---|
| **Multilingual OCR Processing** | Tesseract.js `eng+hin` parses WhatsApp/SMS screenshots containing Hinglish dialect data |
| **Vector Similarity Matcher** | Computes 1536-dimensional embeddings using `gemini-embedding-2` for semantic threat analysis |
| **Fast-Track Risk Bypass** | Client-side and server-side regex check immediately flags critical scam keywords for instant escalation |
| **NDJSON Event Streaming** | Express backend pipes real-time telemetry updates to client over standard HTTP stream response |
| **High-Uptime Failover** | Direct Mongoose driver fallback protects runtime logic from MCP container startup latency |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| AI Agent Framework | **Google ADK** (`google.adk.agents.Agent`) — Vertex AI Agent Builder |
| AI Model | **Gemini 2.5 Flash** (`gemini-2.5-flash`) |
| Embeddings | **Gemini Embedding 2** (`gemini-embedding-2`, 1536 dims) |
| Partner MCP | **MongoDB Atlas MCP Server** (`mongodb-mcp-server`) |
| Database | **MongoDB Atlas** (Vector Search — cosine, 1536 dims) |
| OCR | **Tesseract.js** (eng+hin, in-memory) |
| Backend | **Node.js + Express** (NDJSON streaming, rate limiting) |
| ADK Bridge | **Python FastAPI + Uvicorn** (HTTP bridge to ADK runner) |
| Frontend | **React + Vite + TailwindCSS** (glassmorphism, SVG animations) |
| Deployment | **Render** (backend + ADK bridge), **Vercel** (frontend) |

---

## 🚀 Run Locally

### Prerequisites
- Node.js ≥ 18
- Python ≥ 3.10
- MongoDB Atlas URI **or** local MongoDB
- Gemini API Key from [aistudio.google.com](https://aistudio.google.com)

### 1. Clone & configure

```bash
git clone https://github.com/ishcares/scamshield-ai.git
cd scamshield-ai

# Copy and fill in your keys
cp .env .env.local
# Edit .env → set GEMINI_API_KEY and MONGODB_URI
```

### 2. Start the Python ADK Agent Bridge

```bash
python -m venv venv
# Windows
venv\Scripts\activate          
# Mac/Linux
# source venv/bin/activate     

pip install -r requirements.txt
python agent.py serve          # Starts FastAPI on :8080
```

Health check: http://localhost:8080/health

### 3. Start the Express Backend

```bash
cd server
npm install
npm run dev                    # Starts on :5000
```

Server proxies Step 1 through `agentService.js` → ADK bridge.

### 4. Start the React Frontend

```bash
cd client
npm install
npm run dev                    # Starts on :5173
```

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/analyze` | 6-step NDJSON streaming analysis |
| `GET` | `/api/reports` | Paginated ledger (filter by `riskLevel`) |
| `GET` | `/api/reports/recent` | Last 10 scam reports |
| `GET` | `/api/fir/:reportId` | Download pre-filled security incident report `.txt` |
| `GET` | `/api/agent-status` | Live ADK + MCP health status |
| `GET` | `/health` | Server + MongoDB connection status |

---

## 🧩 MongoDB Atlas Vector Index Setup

Create this index in **Atlas Search** on the `scamreports` collection:

```json
{
  "fields": [{
    "type": "vector",
    "path": "embedding",
    "numDimensions": 1536,
    "similarity": "cosine"
  }]
}
```

**Index Name:** `scam_embedding_index`

> If the Atlas vector index isn't configured, the system automatically falls back to in-memory cosine similarity — no crash, no downtime.

---

## 🌍 Regional Coverage and Localized Integration Details

To demonstrate real-world applicability in targeted markets, the system includes deep coverage parameters for localized Indian fraud patterns:
- **Hinglish OCR parsing** — decodes localized socio-linguistic patterns in messages
- **Pre-filled Regulatory Incident Report generator** — outputs structured complaint files matching formatting specifications for India's Ministry of Home Affairs (MHA) cybercrime portal
- **Hotline Escalar** — surfaces national consumer protection and cyber crime hotline routes (1930 Helpline)
- **UPI payment handle heuristics** — flags specific high-volume digital wallet patterns (`@paytm`, `@okaxis`, `@ybl`)
- **Localized scam classification taxonomy** — targets localized threats (Onboarding Fee Fraud, Task-Based scams, Fake Internships)

---

## 📄 License

MIT License — see [LICENSE](LICENSE)

Built for the **Google Cloud Rapid Agent Hackathon 2026** — MongoDB Partner Track.
