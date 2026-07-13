<div align="center">

<img src="public/postprism-logo.png" alt="PostPrism logo" width="120">

# PostPrism

**A hackathon prototype for adapting one piece of content and orchestrating isolated computer-use agents for LinkedIn, X, and Instagram in parallel.**

[![Agent S2.5](https://img.shields.io/badge/agent-Agent%20S2.5-red)](https://github.com/simular-ai/Agent-S)
[![ORGO](https://img.shields.io/badge/VMs-ORGO-green)](https://docs.orgo.ai/)
[![UI-TARS 1.5](https://img.shields.io/badge/grounding-UI--TARS%201.5-blue)](https://github.com/bytedance/UI-TARS)
[![Live demo](https://img.shields.io/badge/demo-live-purple)](https://postprism.lovable.app)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

[Quick start](#quick-start) · [Hosted demo](#hosted-front-end-demo) · [Architecture](#architecture) · [Setup](#setup) · [Technical deep dive](#technical-deep-dive)

</div>

PostPrism is a full-stack experiment built for the ORGO AI hackathon. The credentialed backend creates a separate computer-use agent and ORGO VM per platform, adapts content per platform, executes the agents concurrently, and emits screenshots and progress events to one dashboard. The hosted Lovable build is a front-end-only simulation and does not sign in to or publish on social platforms. Front end: React + TypeScript (Vite). Back end: Flask + [Agent S2.5](https://github.com/simular-ai/Agent-S) with [UI-TARS 1.5](https://github.com/bytedance/UI-TARS) visual grounding.

## Quick start

The front end builds and runs on [Bun](https://bun.sh/); this demo mode needs no API keys.

```bash
git clone https://github.com/zelinewang/postprism-12e78c39.git
cd postprism-12e78c39
bun install
bun run build    # verified: 1769 modules transformed, built in ~2s
bun run dev      # front end at http://localhost:8080
```

For the credentialed experimental backend (ORGO + OpenAI keys), see [Setup](#setup).

---

## Hosted front-end demo

[![Launch PostPrism](https://img.shields.io/badge/launch-live%20demo-purple?style=for-the-badge)](https://postprism.lovable.app)
[![Watch demo video](https://img.shields.io/badge/watch-2%20min%20video-red?style=for-the-badge)](https://drive.google.com/file/d/1VQ-ryiUvUobjEwkwRCKIvOA-i2ifnabP/view?usp=drive_link)

The hosted Lovable deployment is deliberately front-end only. It uses
[`src/services/demoService.ts`](./src/services/demoService.ts) to simulate
parallel progress and results without backend calls, account access, provider
credentials, or publishing.

**What you'll see:**
- A simulated three-platform progress dashboard
- Local platform-specific content adaptation
- Simulated agent actions and completion states
- The interface and information flow used by the credentialed backend path

**Demo workflow:** type one piece of content → the browser adapts it per platform → simulated agents progress in parallel. The demo does not create social posts. The video records the hackathon workflow, not a current end-to-end publishing guarantee.

---

## About this project

Built solo for the ORGO AI hackathon. Implemented surfaces:

- Front end: React + TypeScript (Vite)
- Experimental back end: Flask + Python orchestration
- Agent S2.5 integration with custom optimizations
- ORGO VM management and parallel execution
- Real-time screen streaming over WebSocket
- Progress monitoring

The hardest part: Agent S2.5 released on August 1st and I migrated from S2 to S2.5 under 24 hours before the deadline, which meant standing up a separate UI-TARS grounding endpoint on short notice.

---

## What is implemented

- **Backend frame and progress events.** During the credentialed backend path, each agent step emits its ORGO screenshot and action state over Flask-SocketIO. The hosted demo simulates these events locally.
- **One isolated agent per platform.** The backend initializes separate ORGO computers, grounding agents, and Agent S2.5 instances so the runs do not share browser state.
- **Parallel task execution.** The backend creates one coroutine per platform and awaits them together with `asyncio.gather`.
- **Agent S2.5 + UI-TARS grounding.** Uses [Agent S2.5](https://github.com/simular-ai/Agent-S) for computer use with `ui-tars-1.5-7b` visual grounding and a configurable OpenAI decision model (`gpt-4o-mini` by default).
- **Loop breakers and rate-limit backoff.** `OptimizedAgentManager` detects repeated actions and rewrite attempts, caps steps, and increases per-platform delay after rate-limit errors.

### Success semantics

The backend is an experimental hackathon path, not a verified publishing
service. Its current controller can return `success=True` when it detects a
repeated action or a rewrite attempt, without reading the platform afterward to
confirm that a post exists. It also generates a placeholder `post_url` rather
than extracting a canonical URL from the platform. Treat a success result as
"the controller terminated its run," not as proof of publication; verify the
target account manually.

---

## Architecture

```
postprism-12e78c39/
├── 📄 README.md                          # This README
├── 📄 env.example.txt                    # Environment setup template
│
├── 🎨 src/                               # Frontend (React + TypeScript)
│   ├── 📄 App.tsx                        # Main application entry
│   │                                     # Location: ./src/App.tsx
│   ├── 📄 pages/Index.tsx                # Primary publishing interface
│   │                                     # Location: ./src/pages/Index.tsx
│   ├── 📄 components/
│   │   ├── 📄 ContentInput.tsx           # Content input with AI preprocessing
│   │   ├── 📄 LiveStreamViewer.tsx       # Real-time AI observation dashboard
│   │   ├── 📄 PublishResults.tsx         # Results analytics & tracking
│   │   └── 📄 PlatformCard.tsx           # Platform status display
│   └── 📄 config/api.ts                  # API configuration & demo mode
│
├── 🤖 backend/                           # Backend (Flask + Agent S2.5)
│   ├── 📄 run_fixed.py                   # Backend entry point
│   │                                     # Location: ./backend/run_fixed.py
│   ├── 📄 app_fixed.py                   # Main Flask application
│   │                                     # Location: ./backend/app_fixed.py
│   ├── 📄 requirements.txt               # Dependencies
│   ├── 📄 install_dependencies.sh        # Automated setup
│   │
│   ├── 🧠 agent_s2_controller/
│   │   ├── 📄 optimized_agent_manager.py # Custom Agent S2.5 enhancements
│   │   │                                 # Anti-perfectionism, loop detection
│   │   └── 📄 official_agent_manager.py  # Standard wrapper
│   │
│   ├── 🎥 streaming/
│   │   ├── 📄 video_streamer.py          # Real-time video streaming
│   │   └── 📄 progress_tracker.py        # Progress monitoring
│   │
│   └── 🔄 content_adapters/
│       └── 📄 multi_platform_adapter.py  # AI content optimization
│
└── 📚 docs/archive/                      # Development documentation
```

---

## Setup

**Note:** the credentialed backend is designed around three accounts (LinkedIn, X, Instagram); the hosted demo does not access any account. The architecture can be extended to more platforms/accounts.

### Prerequisites

#### 1. ORGO AI account & VM setup

This enables the parallel architecture.

```bash
# Step 1: Get ORGO API Access
# Sign up at: https://docs.orgo.ai/introduction

# Step 2: Create 3 Dedicated VMs (one for each platform)
LinkedIn VM    → Project ID: "proj_linkedin_abc123" (save this!)
Twitter VM     → Project ID: "proj_twitter_def456" (save this!)
Instagram VM   → Project ID: "proj_instagram_ghi789" (save this!)

# Step 3: Persistent login setup
# For each VM:
1. Connect to VM via ORGO interface
2. Open browser → Navigate to platform → Login
3. Keep browser open, stay logged in
4. Test: Refresh page → Should remain logged in

# Why this works:
# - ORGO VMs maintain state when paused
# - No re-authentication needed = faster publishing
# - Each VM has a unique IP
```

#### 2. Agent S2.5 configuration

```bash
# Required: OpenAI API Key
OPENAI_API_KEY=sk-your_openai_key_here
AGENTS2_5_MODEL=o3-2025-04-16              # Recommended by Agent S2.5 team
# but we use gpt-4o-mini for speed

# Required: UI-TARS 1.5 Grounding Model
AGENTS2_5_GROUNDING_URL=https://your-endpoint.endpoints.huggingface.cloud
AGENTS2_5_GROUNDING_API_KEY=hf_your_token_here
```

#### 3. Environment configuration

Create `.env` file:

```bash
# ORGO AI Configuration
ORGO_API_KEY=your_orgo_api_key_here

# Platform-Specific VM IDs
ORGO_LINKEDIN_PROJECT_ID=proj_linkedin_abc123
ORGO_TWITTER_PROJECT_ID=proj_twitter_def456
ORGO_INSTAGRAM_PROJECT_ID=proj_instagram_ghi789

# AI Model Configuration
OPENAI_API_KEY=sk-your_openai_key_here
AGENTS2_5_MODEL=o3-2025-04-16
# but we use gpt-4o-mini for speed

# UI-TARS 1.5 Configuration
AGENTS2_5_GROUNDING_URL=your_grounding_endpoint
AGENTS2_5_GROUNDING_API_KEY=your_grounding_key
AGENTS2_5_GROUNDING_MODEL=ui-tars-1.5-7b

# Feature toggles
ENABLE_ANTI_PERFECTIONISM=true
ENABLE_LOOP_DETECTION=true
ENABLE_LIVE_STREAMING=true
```

### Installation & launch

```bash
# Clone & setup
git clone https://github.com/zelinewang/postprism-12e78c39.git
cd postprism-12e78c39

# Run automated backend dependencies installation
cd backend && chmod +x install_dependencies.sh && ./install_dependencies.sh

# The setup script creates/updates .env in the project root; edit it with your keys and VM IDs

# Launch
cd ..                                     # Return to project root
bun run dev &                             # Frontend on :8080 (npm run dev also works)
python backend/run_fixed.py               # Backend on :8000

# Open http://localhost:8080 and watch the agents run
```

---

## Technical deep dive

The architecture below maps directly to tracked implementation; there is no
pseudocode API in this section.

### Parallel orchestration

[`PostPrismApp._execute_official_publishing`](./backend/app_fixed.py) builds one
`_publish_single_platform_parallel` coroutine per requested platform and passes
the complete list to `asyncio.gather(..., return_exceptions=True)`. It then
normalizes each `OptimizedPublishResult` and emits per-platform and aggregate
Socket.IO events.

### Agent execution loop

[`OptimizedAgentManager._run_optimized_agent_loop`](./backend/agent_s2_controller/optimized_agent_manager.py)
repeats four concrete operations: capture an ORGO screenshot, call
`AgentS2_5.predict`, emit the frame/action state, and execute the returned action
through `Computer.exec`. The same method contains the repeated-action and
rewrite loop breakers described in [Success semantics](#success-semantics).

### Stream lifecycle

[`VideoStreamer`](./backend/streaming/video_streamer.py) owns session lifecycle,
frame buffers, frame-rate limits, and Socket.IO broadcast state. Agent frames
are emitted by `OptimizedAgentManager` as `video_frame` events with the session,
platform, step, and base64 screenshot payload.

---

## Configuration reference

Use the configuration below (from [`env.example.txt`](./env.example.txt)); the implementation uses `AGENTS2_5_*` variable names.

```bash
# ===== CORE REQUIREMENTS =====
OPENAI_API_KEY=sk-your-openai-api-key-here     # From: https://platform.openai.com/api-keys
ORGO_API_KEY=your-orgo-api-key-here            # From: https://console.orgo.ai/

# ===== PLATFORM VM IDs (Optional but recommended) =====
ORGO_LINKEDIN_PROJECT_ID=your-linkedin-vm-id   # Create at: https://console.orgo.ai/projects
ORGO_TWITTER_PROJECT_ID=your-twitter-vm-id      # Enables persistent login states
ORGO_INSTAGRAM_PROJECT_ID=your-instagram-vm-id  # Faster publishing performance

# ===== AGENT S2.5 CONFIGURATION =====
AGENTS2_5_MODEL=gpt-4o-mini                    # Default: fast & cost-effective
AGENTS2_5_MODEL_TYPE=openai                    # Provider: openai/anthropic
AGENTS2_5_GROUNDING_MODEL=ui-tars-1.5-7b       # Visual UI detection model
AGENTS2_5_GROUNDING_TYPE=huggingface           # Grounding service provider
AGENTS2_5_MAX_STEPS=15                         # Maximum automation steps
AGENTS2_5_STEP_DELAY=1.0                       # Delay between actions (seconds)
AGENTS2_5_MAX_TRAJECTORY_LENGTH=8              # Memory efficiency
AGENTS2_5_ENABLE_REFLECTION=true               # Learning capability
```

### Model selection guide

| Model | Relative trade-off | Intended use |
|-------|--------------------|--------------|
| `gpt-4o-mini` | Faster / lower-cost | Default development path |
| `gpt-4o` | More capable / higher-cost | Accuracy-sensitive experiments |
| `o3-2025-04-16` | Slower reasoning path | Explicit opt-in experiments |

### Automated installation

`install_dependencies.sh` handles:

1. **Python virtual environment** — isolated dependency management
2. **Standard dependencies** — Flask 3.0+, SocketIO 5.3+, OpenAI 1.25+, etc. ([`requirements.txt`](./backend/requirements.txt))
3. **GUI Agents S2.5** — v0.2.5 (Aug 2025) from the [official repository](https://github.com/simular-ai/Agent-S)
4. **ORGO AI client** — virtual desktop orchestration (`pip install orgo`)
5. **Production extras** — Gunicorn, Eventlet
6. **Environment setup** — interactive wizard via `setup_env.py`

Run: `chmod +x install_dependencies.sh && ./install_dependencies.sh`

### Ports

- **Frontend**: `http://localhost:8080` (Vite dev server)
- **Backend**: `http://localhost:8000` (Flask)
- **Health check**: `http://localhost:8000/health`

### Deployment options (see [`DEPLOYMENT_STRATEGY.md`](./DEPLOYMENT_STRATEGY.md))

**Demo mode (no backend):**
```bash
bun install && bun run dev
echo "VITE_DEMO_MODE=true" > .env.local
```

**Credentialed local backend path (experimental):**
```bash
cd backend && chmod +x install_dependencies.sh && ./install_dependencies.sh
cd .. && bun install
cp env.example.txt .env  # edit with your keys
bun run dev & python backend/run_fixed.py
```

**Cloud:** the hosted [Lovable front end](https://postprism.lovable.app) is
forced into front-end-only demo mode. The repository contains Render/Railway
configuration for a separate backend, but that credentialed path is not part of
the public demo and is not claimed as production-ready.

### Troubleshooting (see [`SETUP_GUIDE.md`](./SETUP_GUIDE.md))

```bash
# Backend connection failed
curl http://localhost:8000/health

# OpenAI rate limits — use a cheaper model
AGENTS2_5_MODEL=gpt-4o-mini

# ORGO VM access issues
curl -H "Authorization: Bearer $ORGO_API_KEY" https://api.orgo.ai/health

# Installation verification
python backend/run_fixed.py --test
```

---

## Status

Hackathon project (ORGO AI hackathon, 2025); not actively maintained. Earlier drafts of this README carried pitch-style figures (success rates, ROI, market size) that were estimates, not measurements — they have been removed. The hosted front end is reproducible as a simulation. The credentialed backend is source-documented but was not end-to-end revalidated for this README and does not verify publication state.

## License

[MIT](LICENSE).

## Acknowledgments

- [ORGO AI](https://docs.orgo.ai/) — isolated cloud VMs
- [Agent S2.5](https://github.com/simular-ai/Agent-S) — computer-use agent
- [UI-TARS 1.5](https://github.com/bytedance/UI-TARS) — visual grounding
- [OpenAI](https://platform.openai.com/) — decision models
- Front end scaffolded with [Lovable](https://lovable.dev/)
