# Clipify — AI YouTube Video Clipper powered by DeepSeek AI

**Clipify** is a self-hosted AI video clipping platform designed to transform long YouTube videos (podcasts, songs & music videos, stand-up comedy specials, interviews, and live streams) into viral 9:16 short-form clips for YouTube Shorts, TikTok, and Instagram Reels.

It is styled in a **OLED Deep Black** canvas with **Normal Red** brand accents, highlighted with **Gold** and **Silver** metallic details.

---

## 1. Quick Architecture Overview

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Frontend** | Next.js 15, React 19, Tailwind CSS v4, ShadCN | Styled in Black, Normal Red, Gold, and Silver aesthetics |
| **Backend API** | FastAPI (Python 3.11+), PydanticAI | REST API + Server-Sent Events (SSE) streaming |
| **AI Engine** | **DeepSeek API** (`deepseek-chat` / `deepseek-reasoner`) | Evaluates virality, hook potential, and timestamps |
| **Video Processing** | `yt-dlp`, FFmpeg, OpenCV | Video downloading, face-tracking 9:16 crop, subtitle rendering |
| **Transcription** | `youtube_captions`, `whisper` (local), or `assemblyai` | Word-level timestamp generation |
| **Task Queue & DB** | Redis + ARQ Worker, PostgreSQL | Asynchronous background processing |

---

## 2. Connecting Your DeepSeek API Key

### Step 1: Obtain your DeepSeek API Key
1. Go to the [DeepSeek Platform](https://platform.deepseek.com/).
2. Log in or create an account.
3. Navigate to **API Keys** and click **Create new API key**.
4. Copy your key (starts with `sk-...`).

### Step 2: Configure `.env` in the repository
Open `supoclip-main/.env` (or copy from `.env.example` if not already present):

```env
# ==============================================
# AI Provider: DeepSeek
# ==============================================
DEEPSEEK_API_KEY=sk-your-actual-deepseek-api-key-here
LLM=deepseek:deepseek-chat

# Optional: To use DeepSeek-R1 (reasoning model)
# LLM=deepseek:deepseek-reasoner
```

---

## 3. Transcription Configuration

You have three options for speech-to-text:

1. **Free & Instant for YouTube (`youtube_captions` - Default)**:
   ```env
   TRANSCRIPTION_PROVIDER=youtube_captions
   ```
   *No extra API keys required. Extracts accurate YouTube captions directly.*

2. **Local & Free with Whisper (`whisper`)**:
   ```env
   TRANSCRIPTION_PROVIDER=whisper
   WHISPER_MODEL_SIZE=base
   ```
   *Runs locally on your CPU/GPU without third-party cloud costs.*

3. **Cloud Transcription (`assemblyai`)**:
   ```env
   TRANSCRIPTION_PROVIDER=assemblyai
   ASSEMBLY_AI_API_KEY=your_assemblyai_key
   ```
   *Best for multi-speaker diarization and noisy audio.*

---

## 4. How to Launch Clipify

### Method A: Docker Desktop (Recommended for Windows)

Since video clipping requires **FFmpeg**, **PostgreSQL**, and **Redis**, the easiest and cleanest way on Windows is using Docker Desktop:

1. Download and install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/) (ensure WSL 2 backend is enabled).
2. Open Docker Desktop.
3. Open PowerShell in `supoclip-main`:
   ```powershell
   cd supoclip-main
   docker compose up -d --build
   ```
4. Once built and running:
   - **Clipify Web App:** [http://localhost:3107](http://localhost:3107)
   - **Backend API:** [http://localhost:8000/docs](http://localhost:8000/docs)

To stop the containers:
```powershell
docker compose down
```

---

### Method B: Native Windows Setup (Without Docker)

If you prefer running directly on your Windows host:

#### 1. Install System Requirements via `winget`
In an Administrator PowerShell prompt:
```powershell
# 1. Install FFmpeg (required for video trimming & cropping)
winget install Gyan.FFmpeg

# 2. Install PostgreSQL (required for tasks database)
winget install PostgreSQL.PostgreSQL

# 3. Restart PowerShell so PATH variables take effect
ffmpeg -version
```

#### 2. Start the Backend API & Worker
```bash
cd supoclip-main/backend

# Create virtual environment
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate

# Install dependencies
pip install -e .

# Run database migrations
alembic upgrade head

# Run backend API server
uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload

# In a separate terminal, run background worker
python src/worker_main.py
```

#### 3. Start the Next.js Frontend
```bash
cd supoclip-main/frontend

# Install dependencies
pnpm install
# or: npm install

# Start development server
pnpm dev
# or: npm run dev
```
Open [http://localhost:3107](http://localhost:3107) in your browser.

---

## 5. Using Clipify to Clip Content

1. **Open Clipify** at `http://localhost:3107`.
2. **Paste a YouTube URL** in the main input box:
   - **Podcasts:** e.g., Joe Rogan, Lex Fridman, Huberman Lab interviews.
   - **Songs & Music Videos:** AI identifies catchy hooks, solos, or choruses.
   - **Stand-up Comedy:** AI detects punchlines, setups, and crowd laughs.
3. Select your clip length preference (e.g. *15–30s*, *30–60s*, or *Auto*).
4. Select your **Framing** (*Face-Centered 9:16 Crop* or *Split-Screen*).
5. Click the red **Generate** button.
6. Watch the live progress updates via Server-Sent Events (SSE).
7. Download your 9:16 shorts or edit captions directly in the studio.

