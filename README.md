# google_cloud_deployed_agent

A Google ADK agent built on Gemini 2.5 Flash with Google Search and URL-fetching sub-agents.

## Project layout

```
google_cloud_deployed_agent/
├── __init__.py          # exposes agent module to the ADK CLI
├── agent.py             # defines root_agent and its sub-agents
├── requirements.txt     # Python dependencies
├── .env                 # API credentials (not checked in)
└── .venv/               # virtual environment
```

The `root_agent` symbol in `agent.py` is what the ADK CLI looks for.

## Prerequisites

- Python 3.10+
- A Gemini API key from https://aistudio.google.com/apikey **or** a Google Cloud project with Vertex AI enabled

## Setup

From the repo root (`c:\dev\google_cloud_python_agent`):

```powershell
# 1. Activate the virtual environment
.\google_cloud_deployed_agent\.venv\Scripts\Activate.ps1

# 2. Install dependencies
pip install -r google_cloud_deployed_agent\requirements.txt
```

## Configure credentials

Edit `google_cloud_deployed_agent\.env`. Pick **one** option:

**Option A — AI Studio (simplest):**
```env
GOOGLE_GENAI_USE_VERTEXAI=FALSE
GOOGLE_API_KEY=your_api_key_here
```

**Option B — Vertex AI (uses `gcloud auth application-default login`):**
```env
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
```

## Run

Run from the **parent** directory (`c:\dev\google_cloud_python_agent`), not from inside the agent folder.

**Web UI** (chat interface at http://localhost:8000):
```powershell
python -m google.adk.cli web
```

**Terminal chat:**
```powershell
python -m google.adk.cli run google_cloud_deployed_agent
```

> **Windows note:** if your machine has Application Control / WDAC enabled, the `adk.exe` shim in `.venv\Scripts\` may be blocked. Always invoke via `python -m google.adk.cli` instead of `adk` directly.

## How it works

`root_agent` orchestrates two specialized sub-agents via `AgentTool`:

- **My_Agent_google_search_agent** — uses `GoogleSearchTool` to search the web
- **My_Agent_url_context_agent** — uses `url_context` to fetch and read page content

The root agent decides which sub-agent (if any) to invoke based on the user's prompt.
