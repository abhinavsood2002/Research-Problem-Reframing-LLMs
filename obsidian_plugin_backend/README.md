# Research Frames Backend

FastAPI backend for the Research Frames Obsidian plugin. Generates AI research
frames from a user's notes and PDFs using a local vLLM-served model, with a
PostgreSQL store and real-time WebSocket updates.

## Getting Started

### Prerequisites

- Python 3.12
- Docker + `docker-compose` (for PostgreSQL)
- `tmux` (used by `run_all.sh`)
- NVIDIA GPU with CUDA 12.x — required by vLLM. Default config in `run_llm.sh`
  serves `openai/gpt-oss-120b`; edit it if you want a different model.

### Install

```bash
# 1. Clone and enter the repo
git clone <repo-url> obsidian_plugin_backend
cd obsidian_plugin_backend

# 2. Create the venv and install dependencies
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 3. Configure environment variables
cp .env.example .env   # or create .env manually
# Edit .env — minimum required:
#   DATABASE_URL=postgresql://research_user:research_password@localhost:5432/research_frames
#   HOST=0.0.0.0
#   PORT=8000
#   LOG_LEVEL=info
```

### Run

```bash
# Starts PostgreSQL (Docker), vLLM, and the FastAPI server in tmux sessions.
./run_all.sh

# Stops everything.
./stop_all.sh
```

To run pieces individually:

```bash
docker-compose up -d   # PostgreSQL only
./run_llm.sh           # vLLM server only
python main.py         # FastAPI server only
```

Health check:

```bash
curl http://localhost:8000/health
```

## Code Overview

- **`main.py`** — FastAPI app: auth, WebSocket manager, all HTTP endpoints,
  lifespan that starts/stops the background worker.
- **`db/`** — Async PostgreSQL access (asyncpg) using a repository-per-table
  pattern. `models.py` holds the Pydantic models.
- **`modules/`** — Core logic.
  - `frame_queue.py` — persistent pickle-backed task queue (`frame_queue.pkl`).
  - `strategic_background_worker.py` — polls the queue and runs generation.
  - `content_processor.py` — assembles notes/PDFs into LLM input.
  - `pdf_manager.py`, `pdf_summary_extractor.py` — PDF ingest and summarization.
- **`handlers/llm_handler.py`** — vLLM API client (OpenAI-compatible).
- **`strategies/`** — Pluggable frame-generation strategies (`base_strategy.py`
  + concrete implementations registered in `__init__.py`).

The flow is: client updates context → `/generate-frame` enqueues a task → the
background worker picks it up, runs the chosen strategy through the LLM, stores
the frame in PostgreSQL, and pushes a WebSocket event to the client.
