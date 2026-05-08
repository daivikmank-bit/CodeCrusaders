#  PostureAlert — AI-Powered Posture Monitor via Webcam

**Team:** Code Crusaders  
**Hackathon Theme:** AI for Wellness & Productivity  
**Tech Stack:** HTML5 · MediaPipe Pose · OpenClaw · Claude Sonnet 4
**Working link:** posture-codecrusaders.netlify.app

---

##  Problem

Millions of students and remote workers spend **6–12 hours daily** seated at desks, silently developing chronic neck, back, and shoulder pain from poor posture. Existing solutions are:

- **Too intrusive** — constant popups break focus
- **Too expensive** — hardware-based ergonomic sensors cost hundreds
- **Privacy-invasive** — cloud-based camera analysis streams your video
- **Hard to install** — desktop apps with complex setup requirements

There is no lightweight, browser-native, privacy-first posture tool that just *works*.

---

##  Solution

**PostureAlert** is a zero-install, browser-based posture monitor that:

1. Uses your **front-facing webcam** to analyze sitting posture in real time
2. Runs all **pose inference locally** in the browser (no video ever leaves your device)
3. Fires **non-intrusive alerts** only when posture drops below your chosen threshold
4. Provides **AI-powered ergonomics coaching** via **OpenClaw → Claude Sonnet 4**
5. Shows a **live skeleton overlay** and posture score so you can self-correct
6. Generates **session summaries** so you can track your posture over time

> The entire app is a **single static HTML file** — open it in any modern browser and it works.

---

##  Architecture

```
Browser Webcam Feed
        │
        ▼
MediaPipe Pose (WebGL/WASM — fully local)
        │  33 body landmarks
        ▼
Posture Analysis Engine
  ├── Shoulder tilt detection
  ├── Head-forward posture (ear-over-shoulder)
  ├── Spine compression ratio
  └── Composite Score (0–100)
        │
        ├──[Score OK]──▶ Continue session, update stats
        │
        └──[Score Low]──▶ Alert System
                              │
                              ▼
                     ┌─────────────────────┐
                     │   OpenClaw Gateway   │
                     │  (anthropic/claude-  │
                     │   sonnet-4-*)        │
                     └─────────┬───────────┘
                               │  text-only posture summary
                               ▼
                     Claude Sonnet 4 (Anthropic)
                               │
                               ▼
                     AI Ergonomics Recommendations
```

**Privacy boundary:** Video never leaves the browser. OpenClaw only receives compact text (e.g., `"Score: 42. Issues: forward head, left shoulder drop"`).

---

##  OpenClaw Integration

PostureAlert uses **OpenClaw** as its AI agent gateway to route coaching requests to Claude. OpenClaw provides a clean, configurable agent layer between the posture engine and the underlying model.

### Inline openclaw.json (equivalent config embedded in the app)

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "<user-provided>"
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-20250514"
      }
    },
    "list": [
      { "id": "coach",   "params": { "cacheRetention": "short" } },
      { "id": "summary", "params": { "cacheRetention": "none"  } }
    ]
  }
}
```

### Two Named Agents

| Agent ID | Cache Retention | Purpose |
|----------|----------------|---------|
| `coach`  | `short` | Real-time ergonomics tips (manual + auto triggers) |
| `summary`| `none`  | End-of-session pattern analysis and recommendations |

### Three Trigger Modes

| Trigger | Agent | When |
|---------|-------|------|
| **Manual** | `coach` | User taps the "Ask Coach" button |
| **Auto** | `coach` | Score stays below threshold for 60 seconds |
| **Session End** | `summary` | User clicks "End Session" |

### Two Connection Modes

**Mode A — API Key** (direct Anthropic provider via OpenClaw):
```
POST https://api.anthropic.com/v1/messages
model: claude-sonnet-4-20250514
```

**Mode B — Gateway URL** (self-hosted OpenClaw instance):
```bash
openclaw serve          # start local gateway
# then point the app at: http://localhost:5999/v1

POST http://localhost:5999/v1/chat/completions
model: anthropic/claude-sonnet-4-20250514   # OpenClaw model ref
metadata: { agent_id: "coach", cache_retention: "short" }
```

---

##  Setup & Installation

### Prerequisites

- A modern browser (Chrome 90+, Firefox 88+, Edge 90+)
- A working webcam / front-facing camera
- One of:
  - An [Anthropic API key](https://console.anthropic.com/) — paste directly into the app
  - A running [OpenClaw](https://github.com/openclaw/openclaw) gateway (`openclaw serve`)

### Option A: Just open the file (API Key mode)

```bash
git clone https://github.com/code-crusaders/posturealert.git
cd posturealert
open index.html          # or: npx serve .
```

Enter your Anthropic API key in the **OpenClaw Gateway** panel → API Key tab.

### Option B: Run with OpenClaw gateway

```bash
# 1. Install OpenClaw
npm install -g openclaw

# 2. Onboard with your Anthropic API key
openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"

# 3. Start the gateway
openclaw serve

# 4. Open the app and switch to "Gateway URL" tab
#    Enter: http://localhost:5999/v1
open index.html
```

---

##  Usage

| Step | Action |
|------|--------|
| 1 | Open `index.html` in your browser |
| 2 | Configure OpenClaw (API key or gateway URL) in the sidebar |
| 3 | Click **"Start Session"** and allow camera access |
| 4 | Watch the **live skeleton overlay** and **posture score** |
| 5 | Receive alerts when your score drops below the threshold |
| 6 | Click **"Ask Coach"** anytime for OpenClaw-powered AI tips |
| 7 | Click **"End Session"** to get your OpenClaw session summary |

---

##  Project Structure

```
posturealert/
├── index.html          # Entire application — webcam + pose + OpenClaw coaching
├── README.md           # This file
└── AI_DISCLOSURE.md    # AI & OpenClaw usage disclosure
```

---

## Privacy

-  No video is ever transmitted — pose inference is 100% local (MediaPipe WASM)
-  No backend server — purely client-side static file
-  No data storage — session data clears on page close
-  OpenClaw only receives compact text: `"Score: 55. Issues: forward head, slouched spine"`
-  API key stored in session memory only — never logged or transmitted elsewhere

---

##  License

MIT License
