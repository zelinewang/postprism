<div align="center">

<img src="public/lovable-uploads/88784487-172c-4e13-87e3-3ecd85d7d29d.png" alt="PostPrism logo" width="120">

# PostPrism

**Publish one piece of content to LinkedIn, X, and Instagram at once — by running an OpenAI-driven computer-use agent in its own cloud VM for each platform, in parallel, while you watch each one work live.**

[![Agent S2.5](https://img.shields.io/badge/agent-Agent%20S2.5-red)](https://github.com/simular-ai/Agent-S)
[![ORGO](https://img.shields.io/badge/VMs-ORGO-green)](https://docs.orgo.ai/)
[![UI-TARS 1.5](https://img.shields.io/badge/grounding-UI--TARS%201.5-blue)](https://github.com/bytedance/UI-TARS)
[![Live demo](https://img.shields.io/badge/demo-live-purple)](https://postprism.lovable.app)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

[Quick start](#quick-start) · [Live demo](#live-demo) · [Architecture](#architecture) · [Setup](#setup) · [Technical deep dive](#technical-deep-dive)

</div>

PostPrism is a full-stack app built for the ORGO AI hackathon. Rather than a black-box automation, it gives each platform its own computer-use agent in an isolated ORGO cloud VM, streams every agent's screen back to one dashboard, and rewrites the post per platform. Front end: React + TypeScript (Vite). Back end: Flask + [Agent S2.5](https://github.com/simular-ai/Agent-S) with [UI-TARS 1.5](https://github.com/bytedance/UI-TARS) visual grounding.

## Quick start

The front end builds and runs on [Bun](https://bun.sh/); this demo mode needs no API keys.

```bash
git clone https://github.com/zelinewang/postprism-12e78c39.git
cd postprism-12e78c39
bun install
bun run build    # verified: 1769 modules transformed, built in ~2s
bun run dev      # front end at http://localhost:8080
```

For the full agent back end (ORGO + OpenAI keys and real parallel publishing), see [Setup](#setup).

---

## Live demo

[![Launch PostPrism](https://img.shields.io/badge/launch-live%20demo-purple?style=for-the-badge)](https://postprism.lovable.app)
[![Watch demo video](https://img.shields.io/badge/watch-2%20min%20video-red?style=for-the-badge)](https://drive.google.com/file/d/1VQ-ryiUvUobjEwkwRCKIvOA-i2ifnabP/view?usp=drive_link)

**What you'll see:**
- The real-time dashboard showing all three agents at once
- Agent S2.5 navigating LinkedIn, X, and Instagram in parallel
- The same input rewritten per platform (LinkedIn professional, X casual, Instagram visual)
- Each agent's on-screen decisions as it works — no black box

**Workflow:** type one piece of content → each platform's agent adapts and posts it in its own VM → watch all three run in parallel.

---

## About this project

Built solo for the ORGO AI hackathon. Scope accomplished:

- Front end: React + TypeScript (Vite)
- Back end: Flask + Python orchestration
- Agent S2.5 integration with custom optimizations
- ORGO VM management and parallel execution
- Real-time screen streaming over WebSocket
- Progress monitoring

The hardest part: Agent S2.5 released on August 1st and I migrated from S2 to S2.5 under 24 hours before the deadline, which meant standing up a separate UI-TARS grounding endpoint on short notice.

---

## What it does

- **Watch the agents work.** The dashboard streams each VM's screen with the agent's decisions overlaid, so the automation isn't a black box — you see every read, click, and type.
- **One agent per platform, in isolation.** ORGO gives each platform its own cloud VM (isolated browser, persistent login, per-VM fingerprint), so the three runs don't interfere and don't re-authenticate each time. ORGO's fast boot and auto-pause are what make spinning these up per run practical — see the [ORGO docs](https://docs.orgo.ai/introduction).
- **Parallel, not sequential.** The three platform agents run at the same time via `asyncio.gather`, instead of one after another.
- **Agent S2.5 + UI-TARS grounding.** Uses [Agent S2.5](https://github.com/simular-ai/Agent-S) for computer use with `ui-tars-1.5-7b` visual grounding, and OpenAI models for decisions (default `gpt-4o-mini`, up to `o3` for accuracy).
- **Custom optimizations** in `backend/agent_s2_controller/optimized_agent_manager.py`: an anti-perfectionism check that stops endless content re-edits, loop detection, and adaptive rate limiting.

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
│   ├── 📄 run_fixed.py                   # Production startup script
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

**Note:** the demo manages 3 accounts (LinkedIn, X, Instagram); the design generalizes to more platforms/accounts.

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

### Custom Agent S2.5 optimizations

```python
class ProductionOptimizations:
    """Custom enhancements beyond official Agent S2.5"""

    def anti_perfectionism_engine(self, action_history, content):
        """Prevents endless editing loops."""
        edit_attempts = self._count_edit_actions(action_history)
        semantic_change = self._measure_content_delta(content)

        if edit_attempts > 3 and semantic_change < 0.05:
            return ForceCompletion(reason="minimal_improvement")

    def intelligent_loop_detection(self, action_sequence):
        """Cycle prevention via sequence pattern analysis."""
        pattern_prob = self._lstm_pattern_analyzer(action_sequence)
        if pattern_prob > 0.85:
            return BreakLoop(strategy="intelligent_completion")

    def adaptive_rate_limiting(self, platform_responses):
        """Dynamic API spacing based on real-time behavior"""
        optimal_delay = self._calculate_adaptive_delay(
            platform_load=self._estimate_load(platform_responses),
            error_rate=self._recent_error_rate(),
            time_factor=self._time_of_day_multiplier()
        )
        return min(optimal_delay, 30.0)  # Cap at 30 seconds
```

### Real-time streaming

```python
class LiveAIObservatory:
    """Real-time AI observation."""

    async def stream_ai_thinking(self, session_id):
        while self.streaming_active:
            # Capture all 3 platforms simultaneously
            frames = await asyncio.gather(*[
                self.capture_vm_with_ai_overlay(vm_id)
                for vm_id in self.active_vms
            ])

            # Add AI decision visualization
            enhanced_frames = [
                self._overlay_ai_decisions(frame, ai_state)
                for frame, ai_state in zip(frames, self.agent_states)
            ]

            # Stream to frontend
            await self._broadcast_frames(session_id, enhanced_frames)
            await asyncio.sleep(0.5)  # 2 FPS
```

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

| Model | Speed | Cost/session (approx.) | Best for |
|-------|-------|--------------|----------|
| `gpt-4o-mini` | Fast | ~$0.01 | Default: development & most use cases |
| `gpt-4o` | Medium | ~$0.03 | Better accuracy, acceptable cost |
| `o3-2025-04-16` | Slow | ~$0.10+ | Maximum accuracy, high cost |

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

**Full production:**
```bash
cd backend && chmod +x install_dependencies.sh && ./install_dependencies.sh
cd .. && bun install
cp env.example.txt .env  # edit with your keys
bun run dev & python backend/run_fixed.py
```

**Cloud:** front end on [Lovable](https://postprism.lovable.app); back end on Railway/Render with the env vars above.

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

Hackathon project (ORGO AI hackathon, 2025); not actively maintained. Earlier drafts of this README carried pitch-style figures (success rates, ROI, market size) that were estimates, not measurements — they have been removed. What remains is reproducible from the code and the live demo.

## License

[MIT](LICENSE).

## Acknowledgments

- [ORGO AI](https://docs.orgo.ai/) — isolated cloud VMs
- [Agent S2.5](https://github.com/simular-ai/Agent-S) — computer-use agent
- [UI-TARS 1.5](https://github.com/bytedance/UI-TARS) — visual grounding
- [OpenAI](https://platform.openai.com/) — decision models
- Front end scaffolded with [Lovable](https://lovable.dev/)
