# SpoilerGuard - Spoiler-Aware Streaming Companion

A Chrome extension that acts as a spoiler-aware chatbot companion for streaming services. It uses RAG (Retrieval-Augmented Generation) to answer questions about what you're watching without revealing future plot points based on your current timestamp.

## Features

- **Spoiler Detection** - Automatically filters responses based on your current position in a movie or series
- **Timestamp Tracking** - Detects where you are in the video and only allows knowledge up to that point
- **Streaming Site Integration** - Auto-detects content on Disney+ and Netflix
- **Multi-Universe Support** - Organize content into universes (e.g., Marvel, Family Guy)
- **MCU Sort Toggle** - Switch between Release Date and Chronological (in-universe timeline) ordering for Marvel content
- **Content Toggle Switches** - Enable/disable specific movies or episodes from the knowledge base
- **Search** - Filter movies and series in the content panel
- **Collapsible Hierarchy** - Browse content by Universe > Series > Season > Episode

## Architecture

```
ChatbotCore-Extension/
├── client/public/extension/     # Chrome Extension (Manifest V3)
│   ├── manifest.json            # Extension config
│   ├── content.js               # Content script (Disney+/Netflix detection, FAB injection)
│   ├── sidebar.html             # Sidebar UI (chat + content panel)
│   ├── sidebar.js               # Sidebar logic (chat, content selection, sorting)
│   ├── popup.html               # Extension popup (backend URL settings)
│   ├── popup.js                 # Popup logic
│   └── background.js            # Service worker
├── server_py/                   # Python Backend
│   ├── main.py                  # FastAPI server, content ingestion, API routes
│   ├── rag.py                   # RAG system with spoiler-aware filtering
│   ├── models.py                # Pydantic models
│   ├── config.py                # Configuration
│   └── dev.py                   # Dev server runner
├── attached_assets/             # Subtitle files (organized by universe)
│   ├── Marvel/                  # MCU movies (.srt files)
│   └── Family_Guy/              # Family Guy episodes (.srt files)
└── pyproject.toml               # Python dependencies
```

## Prerequisites

- Python 3.11+
- An OpenAI API key (set in `.env` as `OPENAI_API_KEY`)
- Google Chrome browser

## Setup

### 1. Install Python Dependencies

```bash
pip install -r pyproject.toml
# or using uv:
uv sync
```

### 2. Configure Environment

Create a `.env` file in the project root:

```
OPENAI_API_KEY=your_api_key_here
```

### 3. Start the Backend Server

```bash
python -c "import uvicorn; from server_py.main import app; uvicorn.run(app, host='0.0.0.0', port=5000)"
```

The server will automatically ingest all `.srt` files from `attached_assets/` subfolders on startup.

### 4. Load the Chrome Extension

1. Open Chrome and go to `chrome://extensions/`
2. Enable **Developer mode** (top right toggle)
3. Click **Load unpacked**
4. Select the `client/public/extension/` folder
5. The extension icon will appear in your toolbar

### 5. Configure Backend URL

1. Click the SpoilerGuard extension icon in Chrome
2. Set the backend URL (default: `http://localhost:5000`)
3. Click Save

## Usage

1. Navigate to Disney+ or Netflix
2. Play a movie or series
3. A floating action button (FAB) appears in the bottom-right corner
4. Click the FAB to open the sidebar
5. Ask questions about what you're watching - the chatbot will only use knowledge up to your current timestamp
6. Use the content panel (grid icon) to browse and toggle content in the knowledge base

## Adding Content

Place `.srt` subtitle files in the `attached_assets/` directory, organized by universe:

```
attached_assets/
├── Marvel/
│   ├── Iron_Man_2008.srt
│   ├── The_Incredible_Hulk_2008.srt
│   └── ...
├── Family_Guy/
│   ├── S01E01_Death_Has_a_Shadow.srt
│   └── S01E02_I_Never_Met_the_Dead_Man.srt
└── Your_Universe/
    └── Your_Movie_2024.srt
```

**Naming conventions:**
- Movies: `Title_Year.srt` (e.g., `Iron_Man_2008.srt`)
- Series episodes: `S01E01_Episode_Title.srt` (e.g., `S01E01_Death_Has_a_Shadow.srt`)
- Universe folders become universe names (underscores replaced with spaces)

After adding files, click the reload button in the content panel or restart the server.

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/content` | List all universes and their content |
| `POST` | `/api/chat` | Send a question with context (season, episode, timestamp) |
| `POST` | `/api/ingest/reload` | Reload all content from `attached_assets/` |
| `POST` | `/api/ingest/file` | Upload and ingest a single `.srt` file |
| `POST` | `/api/ingest/reset` | Clear the knowledge base |

## Tech Stack

- **Frontend**: Vanilla HTML/CSS/JS (Chrome Extension Manifest V3)
- **Backend**: Python, FastAPI, Uvicorn
- **AI/RAG**: LangChain, OpenAI
- **Data**: SRT subtitle parsing, in-memory vector store
