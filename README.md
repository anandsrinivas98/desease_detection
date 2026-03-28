# 🌿 CropVision — AI-Powered Crop Disease Diagnostics

CropVision is a full-stack agricultural AI platform that helps farmers diagnose crop diseases using image uploads or voice descriptions in their native Indian language. It combines Google Gemini's vision and audio AI with a local FAISS RAG engine, multilingual TTS, and a modern glassmorphism React UI.

---

## 📋 Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Installation](#installation)
- [Running the Application](#running-the-application)
- [API Reference](#api-reference)
- [Supported Languages](#supported-languages)
- [Architecture Overview](#architecture-overview)
- [Disease Knowledge Base](#disease-knowledge-base)
- [Database Schema](#database-schema)
- [Performance Optimizations](#performance-optimizations)
- [Known Limitations](#known-limitations)
- [Troubleshooting](#troubleshooting)

---

## ✨ Features

- **Image Diagnostics** — Upload a crop photo and get instant AI-powered disease identification with treatment recommendations
- **Voice Diagnostics** — Speak symptoms in your regional language; AI transcribes, diagnoses, and responds in the same language
- **Multilingual UI** — Full interface translation in 7 Indian languages (English, Hindi, Tamil, Telugu, Marathi, Kannada, Malayalam)
- **Voice Readout (TTS)** — Results are read aloud in the selected language using Google TTS via gTTS
- **Local RAG Engine** — FAISS + sentence-transformers knowledge base for fast offline disease lookup (~200ms)
- **Gemini Fallback** — Gemini 2.5 Flash handles complex cases not covered by local RAG
- **Scan History** — All diagnoses saved to Neon PostgreSQL with timestamp and method
- **Marketplace** — Agricultural product store UI
- **Advisory Portal** — Expert advisory interface

---

## 🛠 Tech Stack

### Backend
| Technology | Purpose |
|---|---|
| Python 3.11 | Runtime |
| FastAPI | Async REST API framework |
| Google Gemini 2.5 Flash | Vision AI (image) + Audio AI (voice) |
| FAISS (faiss-cpu) | Local vector similarity search |
| sentence-transformers | Multilingual text embeddings (`paraphrase-multilingual-MiniLM-L12-v2`) |
| gTTS | Google Text-to-Speech for regional languages |
| Pillow | Image compression before Gemini upload |
| SQLAlchemy (async) | ORM with asyncpg driver |
| Neon PostgreSQL | Serverless cloud database |
| uvicorn | ASGI server with hot reload |

### Frontend
| Technology | Purpose |
|---|---|
| React 19 | UI framework |
| Vite 8 | Build tool and dev server |
| Tailwind CSS 4 | Utility-first styling |
| Framer Motion | Animations and transitions |
| i18next + react-i18next | Internationalization (7 languages) |
| react-dropzone | Drag-and-drop image upload |
| lucide-react | Icon library |
| react-router-dom 7 | Client-side routing |

---

## 📁 Project Structure

```
disease_detection/
├── backend/
│   ├── .env                          # Environment variables (API keys, DB URL)
│   ├── app/
│   │   ├── main.py                   # FastAPI app entry point, lifespan, CORS
│   │   ├── core/
│   │   │   └── config.py             # Settings loaded from .env
│   │   ├── api/
│   │   │   └── routes/
│   │   │       ├── diagnostics.py    # /analyze-image, /analyze-voice, /tts
│   │   │       ├── history.py        # Scan history endpoints
│   │   │       └── store.py          # Marketplace endpoints
│   │   ├── db/
│   │   │   ├── database.py           # Async SQLAlchemy engine, pool config, migrations
│   │   │   └── models.py             # ScanModel ORM table
│   │   ├── schemas/
│   │   │   └── diagnostics_schema.py # Pydantic response models
│   │   └── services/
│   │       ├── ml_engine.py          # Gemini image analysis with compression + retry
│   │       ├── rag_engine.py         # Gemini voice/audio analysis with retry
│   │       ├── local_rag.py          # FAISS local RAG engine
│   │       └── crop_knowledge.py     # 12-disease knowledge base for RAG
│   └── venv/                         # Python virtual environment
│
└── frontend/
    ├── index.html
    ├── package.json
    ├── vite.config.js
    └── src/
        ├── App.jsx                   # Router, navbar, language toggle
        ├── main.jsx                  # React entry point
        ├── index.css                 # Global styles, glassmorphism classes
        ├── locales/
        │   └── i18n.js               # i18next config with all 7 language translations
        ├── components/
        │   ├── diagnostics/
        │   │   ├── DiagnosticHub.jsx # Main page: image upload + voice recording
        │   │   └── ResultCard.jsx    # Diagnosis result display + TTS playback
        │   ├── history/
        │   │   └── HistoryView.jsx   # Past scan history
        │   ├── store/
        │   │   └── Marketplace.jsx   # Product marketplace
        │   ├── advisory/
        │   │   └── AdvisoryPortal.jsx# Expert advisory
        │   └── layout/
        │       └── AmbientBackground.jsx # Animated background orbs
        └── utils/
            └── animations.js         # Framer Motion animation presets
```

---

## ✅ Prerequisites

Before starting, ensure you have:

- **Python 3.11+** — [Download](https://www.python.org/downloads/)
- **Node.js 18+** — [Download](https://nodejs.org/)
- **Google Gemini API Key** — [Get one free](https://aistudio.google.com/app/apikey)
- **Neon PostgreSQL account** — [Sign up free](https://neon.tech/) (or use any PostgreSQL instance)

---

## 🔐 Environment Setup

A `.env.example` template file is included in `backend/` with all required variables and instructions.

### Step 1 — Copy the template

```bash
cd backend
cp .env.example .env
```

On Windows:
```bash
copy .env.example .env
```

### Step 2 — Fill in your values

Open `backend/.env` and replace the placeholder values:

```env
# Neon PostgreSQL connection string
NEON_DATABASE_URL=postgresql://your_user:your_password@your_host.neon.tech/your_db?sslmode=require

# Google Gemini API Key
GEMINI_API_KEY=your_gemini_api_key_here
```

> ⚠️ **Never commit `.env` to version control.** It is already listed in `.gitignore`. Only `.env.example` (with placeholder values) is committed.

### Getting your Gemini API Key
1. Go to [https://aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)
2. Click **Create API Key**
3. Copy and paste into `backend/.env` as `GEMINI_API_KEY`

> **Free tier limit:** 20 requests/day for Gemini 2.5 Flash. Quota resets at midnight Pacific Time. Upgrade at [https://ai.google.dev](https://ai.google.dev) for unlimited requests.

### Getting your Neon Database URL
1. Sign up at [https://neon.tech](https://neon.tech)
2. Create a new project
3. Go to **Connection Details** → copy the **Connection string**
4. Paste into `backend/.env` as `NEON_DATABASE_URL`

---

## 📦 Installation

### Backend

```bash
cd backend

# Create virtual environment
python -m venv venv

# Activate (Windows)
venv\Scripts\activate

# Activate (Mac/Linux)
source venv/bin/activate

# Install dependencies
pip install fastapi uvicorn sqlalchemy asyncpg python-dotenv google-genai Pillow gtts faiss-cpu sentence-transformers
```

### Frontend

```bash
cd frontend

# Install dependencies
npm install
```

---

## 🚀 Running the Application

Open **two separate terminals**:

**Terminal 1 — Backend:**
```bash
cd backend
venv\Scripts\activate
uvicorn app.main:app --reload --port 8000
```

**Terminal 2 — Frontend:**
```bash
cd frontend
npm run dev
```

Then open your browser at: **http://localhost:5173**

The backend API runs at: **http://localhost:8000**

API documentation (Swagger UI): **http://localhost:8000/docs**

---

## 📡 API Reference

### `POST /api/diagnostics/analyze-image`
Analyzes a crop image using Gemini Vision AI.

| Field | Type | Description |
|---|---|---|
| `file` | File (multipart) | Crop image (JPG, PNG, WEBP) |
| `language` | string | Locale code e.g. `en-IN`, `hi-IN`, `ta-IN` |

**Response:**
```json
{
  "disease": "Whitefly Infestation",
  "confidence": 0.95,
  "treatment_chemical": "Spray Imidacloprid...",
  "treatment_organic": "Neem oil spray...",
  "climate_impact": "Warm dry weather...",
  "method": "image",
  "scanned_at": "2026-03-29T10:00:00"
}
```

---

### `POST /api/diagnostics/analyze-voice`
Analyzes a voice recording describing crop symptoms.

| Field | Type | Description |
|---|---|---|
| `voice_note` | File (multipart) | Audio file (WebM/OGG) |
| `language` | string | Locale code e.g. `en-IN`, `ta-IN` |

**Flow:**
1. Gemini 2.5 Flash transcribes and diagnoses the audio
2. Local FAISS RAG enhances treatment data if confidence ≥ 0.50
3. Returns structured diagnosis in the selected language

---

### `POST /api/diagnostics/tts`
Converts diagnosis text to speech audio.

| Field | Type | Description |
|---|---|---|
| `text` | string | Text to speak |
| `language` | string | Locale code |

**Response:** `audio/mpeg` stream (MP3)

Text is split into sentence chunks and all chunks are generated in parallel for faster response.

---

### `GET /health`
Health check endpoint.

```json
{ "status": "ok", "service": "CropVision Cloud Microservice", "db": "Neon Postgres DB Active" }
```

---

## 🌐 Supported Languages

| Language | Locale Code | Script |
|---|---|---|
| English (India) | `en-IN` | Latin |
| Hindi | `hi-IN` | Devanagari |
| Telugu | `te-IN` | Telugu |
| Tamil | `ta-IN` | Tamil |
| Marathi | `mr-IN` | Devanagari |
| Kannada | `kn-IN` | Kannada |
| Malayalam | `ml-IN` | Malayalam |

Language selection affects:
- UI text (navbar, buttons, labels)
- AI diagnosis response language
- Voice readout (TTS) language
- Voice recording transcription language

---

## 🏗 Architecture Overview

```
User Browser
    │
    ├── Image Upload ──────────────────────────────────────────────────────────┐
    │                                                                           │
    ├── Voice Recording ─────────────────────────────────────────────────────┐ │
    │                                                                         │ │
    └── Language Selector (i18n)                                              │ │
                                                                              ▼ ▼
                                                                    FastAPI Backend
                                                                         │
                                              ┌──────────────────────────┼──────────────────────────┐
                                              │                          │                          │
                                    /analyze-image              /analyze-voice                   /tts
                                              │                          │                          │
                                    Pillow compress            Gemini 2.5 Flash            gTTS parallel
                                    (768px, JPEG 82%)          audio transcription          chunk generation
                                              │                          │                          │
                                    Gemini 2.5 Flash           Local FAISS RAG              MP3 stream
                                    vision analysis            (if score ≥ 0.50)            to browser
                                              │                          │
                                    JSON response              Enhanced result
                                              │                          │
                                              └──────────┬───────────────┘
                                                         │
                                              Neon PostgreSQL
                                              (async fire-and-forget)
```

---

## 🧠 Disease Knowledge Base

The local RAG engine (`crop_knowledge.py`) contains 12 pre-indexed crop diseases:

| # | Disease | Confidence |
|---|---|---|
| 1 | Whitefly Infestation | 0.92 |
| 2 | Aphid Infestation | 0.93 |
| 3 | Early Blight (Alternaria) | 0.88 |
| 4 | Late Blight (Phytophthora) | 0.91 |
| 5 | Powdery Mildew | 0.94 |
| 6 | Leaf Rust | 0.89 |
| 7 | Nutrient Deficiency (Chlorosis) | 0.82 |
| 8 | Root Rot / Damping Off | 0.87 |
| 9 | Spider Mite Infestation | 0.90 |
| 10 | Healthy Plant | 0.95 |
| 11 | Mosaic Virus | 0.86 |
| 12 | Bacterial Blight | 0.85 |

Each entry contains symptom keywords, chemical treatment, organic alternative, and climate impact data. The FAISS index uses cosine similarity with the `paraphrase-multilingual-MiniLM-L12-v2` model (~471MB, downloaded once on first run).

---

## 🗄 Database Schema

**Table: `crop_scans`**

| Column | Type | Description |
|---|---|---|
| `id` | INTEGER (PK) | Auto-increment primary key |
| `method` | VARCHAR(50) | `"image"` or `"voice"` |
| `disease` | VARCHAR(255) | Diagnosed disease name |
| `confidence` | FLOAT | Confidence score (0.0–1.0) |
| `treatment_chemical` | VARCHAR(2000) | Synthetic treatment recommendation |
| `treatment_organic` | VARCHAR(2000) | Organic/natural alternative |
| `climate_impact` | VARCHAR(2000) | Weather/climate context |
| `scanned_at` | TIMESTAMP WITH TIME ZONE | Scan timestamp (auto) |

The database schema is auto-created on first startup. Column sizes are auto-migrated via an idempotent `DO $$ BEGIN ... END $$` block on every startup.

---

## ⚡ Performance Optimizations

| Optimization | Impact |
|---|---|
| Image compression to 768px JPEG 82% | Reduces upload size by ~60-80%, faster Gemini inference |
| `temperature=0.1`, `max_output_tokens=1024` | Deterministic, faster Gemini responses |
| Fire-and-forget DB writes (`asyncio.create_task`) | API responds immediately without waiting for DB |
| Local FAISS RAG for voice (~200ms) | Avoids Gemini call for known diseases |
| Parallel TTS chunk generation (`asyncio.gather`) | First audio chunk plays in ~2-3s while rest load |
| `pool_pre_ping=True`, `pool_recycle=300` | Prevents stale Neon DB connections |
| Gemini 429 retry with actual `retryDelay` | Respects rate limit timing, avoids wasted retries |

---

## ⚠️ Known Limitations

- **Gemini Free Tier:** 20 requests/day. Quota resets at midnight Pacific Time. Upgrade at [ai.google.dev](https://ai.google.dev) for production use.
- **Local RAG:** Only covers 12 diseases. Unknown diseases fall back to Gemini automatically.
- **Voice TTS:** Quality depends on gTTS Google servers. Requires internet connection.
- **FAISS model download:** First startup downloads ~471MB model from HuggingFace. Subsequent startups use cache.
- **Windows symlinks warning:** HuggingFace cache shows a symlink warning on Windows. This is cosmetic and does not affect functionality. Enable Developer Mode in Windows settings to suppress it.

---

## 🔧 Troubleshooting

### Backend won't start
```
Check: Is venv activated? Run: venv\Scripts\activate
Check: Is .env file present in backend/ folder?
Check: Is NEON_DATABASE_URL correct?
```

### 429 Rate Limit Error
```
Cause: Gemini free tier (20 req/day) exhausted
Fix: Wait until midnight Pacific Time for quota reset
     OR upgrade to Gemini paid tier at https://ai.google.dev
```

### DB connection closed error
```
Cause: Neon serverless DB idles out after inactivity
Fix: Already handled — pool_pre_ping=True auto-reconnects
     If persists, restart the backend server
```

### Voice recording not working
```
Check: Browser microphone permission is granted
Check: Use Chrome or Edge (best WebM/Opus support)
Check: Tap mic button once to START, tap again to STOP & analyze
Note: Recording auto-stops after 15 seconds
```

### TTS not playing audio
```
Check: Backend is running on port 8000
Check: Browser allows audio autoplay (click somewhere on page first)
Check: Internet connection (gTTS requires Google servers)
```

### FAISS model download slow
```
Note: First run downloads ~471MB model — this is one-time only
      Subsequent startups load from local cache in ~2 seconds
      Set HF_HUB_DISABLE_SYMLINKS_WARNING=1 to suppress Windows warning
```

### Frontend not loading
```
Check: npm install was run in frontend/ folder
Check: Vite dev server is running on port 5173
Check: Backend CORS allows http://localhost:5173
```

---

## 📄 License

This project is for educational and demonstration purposes.

---

*Built with FastAPI, React, Google Gemini, FAISS, and gTTS.*
