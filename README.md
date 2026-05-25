# google_cloud_python_agent

A pair of Google ADK agents built on Gemini. Together they serve as a **Python skills helper** — the first agent helps you learn and build Python through web search and URL reading, and the second demonstrates Python tool-calling by providing live **weather and time** lookups for any US city.

## Agents

| Folder | Agent | What it does |
| --- | --- | --- |
| `google_cloud_deployed_agent` | Python skills builder | Helps grow your Python skills using `GoogleSearchTool` and `url_context` sub-agents for web search and page-fetching |
| `weather_time_agent` | Weather & time assistant | Answers questions like *"What's the weather in Miami, FL?"* or *"What time is it in Seattle?"* via the Open-Meteo API |

Both expose a `root_agent` symbol — that's what the ADK CLI loads.

## Project layout

```
google_cloud_python_agent/                  # repo root — launch adk from here
├── google_cloud_deployed_agent/
│   ├── __init__.py                         # exposes agent module to the ADK CLI
│   ├── agent.py                            # Python skills builder (search + URL sub-agents)
│   ├── requirements.txt                    # shared Python dependencies
│   ├── .env                                # API credentials (not checked in)
│   └── .venv/                              # virtual environment
└── weather_time_agent/
    ├── __init__.py
    ├── agent.py                            # weather + time tool functions
    └── .env                                # API credentials (not checked in)
```

## Prerequisites

- Python 3.10+
- A Gemini API key from https://aistudio.google.com/apikey **or** a Google Cloud project with Vertex AI enabled

## Setup

From the repo root (`c:\dev\google_cloud_python_agent`):

```powershell
# 1. Activate the virtual environment
.\google_cloud_deployed_agent\.venv\Scripts\Activate.ps1

# 2. Install dependencies (google-adk + requests for the weather agent)
pip install -r google_cloud_deployed_agent\requirements.txt
```

## Configure credentials

Put a `.env` file in **each** agent folder. Pick **one** option:

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

Run from the **repo root** (`c:\dev\google_cloud_python_agent`), not from inside an agent folder. The dropdown is built from sibling sub-folders of the launch directory, so both agents only show up when launched from the parent.

**Web UI** (chat interface at http://localhost:8000):
```powershell
python -m google.adk.cli web
```

Pick `google_cloud_deployed_agent` or `weather_time_agent` from the dropdown at the top.

**Terminal chat:**
```powershell
python -m google.adk.cli run google_cloud_deployed_agent
# or
python -m google.adk.cli run weather_time_agent
```

> **Windows note:** if your machine has Application Control / WDAC enabled, the `adk.exe` shim in `.venv\Scripts\` may be blocked. Always invoke via `python -m google.adk.cli` instead of `adk` directly.

## How it works

### `google_cloud_deployed_agent` — Python skills helper
`root_agent` orchestrates two specialized sub-agents via `AgentTool`:

- **My_Agent_google_search_agent** — uses `GoogleSearchTool` to search the web for Python docs, tutorials, and examples
- **My_Agent_url_context_agent** — uses `url_context` to fetch and read a specific page when you supply a URL

The root agent decides which sub-agent (if any) to invoke based on the user's prompt.

### `weather_time_agent` — weather & time
A flat agent with two tool functions you can call directly through natural language:

- **`get_weather(city)`** — geocodes the city via Open-Meteo, then fetches the current temperature and conditions
- **`get_current_time(city)`** — geocodes the city, then returns the current local time in the resolved timezone

Both helpers accept input like `"Miami"`, `"Miami, FL"`, or `"New York, NY"` — anything after a comma is stripped and the city name is URL-encoded before the API call.
