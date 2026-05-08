#  AI Disclosure — PostureAlert

**Team:** Code Crusaders  
**Project:** PostureAlert — AI-Powered Posture Monitor via Webcam  
**Hackathon Theme:** AI for Wellness & Productivity

---

## Overview

This document transparently describes every AI model, tool, library, and orchestration framework used in the development and functionality of PostureAlert.

---

**What OpenClaw does in PostureAlert:**

OpenClaw acts as the AI orchestration layer between the posture analysis engine and Claude. It defines two named agents with different caching strategies, manages model routing via the `anthropic/*` provider, and supports both direct API key access and a self-hosted gateway (`openclaw serve`).

**OpenClaw agent configuration (inline `openclaw.json` equivalent):**

```json
{
  "env": { "ANTHROPIC_API_KEY": "<user-provided>" },
  "agents": {
    "defaults": {
      "model": { "primary": "anthropic/claude-sonnet-4-20250514" }
    },
    "list": [
      { "id": "coach",   "params": { "cacheRetention": "short" } },
      { "id": "summary", "params": { "cacheRetention": "none"  } }
    ]
  }
}
```

**OpenClaw trigger modes:**

| Trigger | Agent ID | When It Fires |
|---------|----------|--------------|
| Manual Ask | `coach` | User taps "Ask Coach" button |
| Auto-Coach | `coach` | Score below threshold for 60 continuous seconds |
| Session Summary | `summary` | User ends the session |

**OpenClaw connection modes supported:**

- **API Key mode** — User provides Anthropic API key; app calls `api.anthropic.com` directly using OpenClaw's Anthropic provider model ref (`anthropic/claude-sonnet-4-20250514`)
- **Gateway URL mode** — User runs `openclaw serve` locally; app sends requests to `http://localhost:5999/v1/chat/completions` with OpenClaw model refs and agent metadata

---

## 2. AI Model — Anthropic Claude Sonnet 4

| Attribute | Details |
|-----------|---------|
| **Model** | `claude-sonnet-4-20250514` |
| **OpenClaw model ref** | `anthropic/claude-sonnet-4-20250514` |
| **Provider** | Anthropic |
| **API** | Anthropic Messages API (`/v1/messages`) |
| **Routed via** | OpenClaw gateway or direct API key |
| **Purpose** | Ergonomics coaching tips and session summaries |

**What Claude receives (text only — no video, no images):**

Coach agent prompt example:
```
PostureAlert coaching [OpenClaw agent: coach, trigger: user-requested]:
Score: 48/100 | Issues: forward head, left shoulder elevated

Give exactly 2-3 concise, practical ergonomics tips.
```

Summary agent prompt example:
```
PostureAlert session summary [OpenClaw agent: summary]:
Duration: 24m 10s | Good posture: 61% | Alerts: 4 | Final score: 72/100
Issues: forward head, slouched spine

Write a 3-4 sentence session summary with encouragement and 2 improvement suggestions.
```

---

## 3. Local ML Library — MediaPipe Pose (Google)

| Attribute | Details |
|-----------|---------|
| **Library** | `@mediapipe/pose` |
| **Provider** | Google |
| **Runtime** | Browser-local (WebGL + WASM) |
| **Purpose** | Real-time 33-point human pose estimation from webcam |
| **Data sent externally** | None — runs entirely in the browser |

---

## 4. AI Tools Used During Development

| Tool | Provider | Used For |
|------|----------|---------|
| Claude (claude.ai) | Anthropic | Brainstorming algorithm design, code review, writing docs |
| GitHub Copilot | GitHub / OpenAI | Code completion suggestions during development |

---

## 5. Data Privacy Summary

| Data Type | Stays Local | Sent Externally | Destination |
|-----------|------------|-----------------|-------------|
| Webcam video frames | Yes | Never | — |
| MediaPipe pose landmarks | Yes | Never | — |
| Posture text summary | — | Yes | OpenClaw → `api.anthropic.com` |
| User API key | Session memory | Never stored | — |

---

## 6. Compliance Statement

The team confirms that:
- OpenClaw is used as an open-source, self-hostable agent gateway under its published terms
- All AI-generated coaching content is clearly labeled in the UI as "OpenClaw · Claude Sonnet 4"
- No AI-generated code was submitted without review and understanding by team members
- Use of Anthropic's Claude API complies with Anthropic's usage policies
- No personally identifiable information is sent to any external service

---

*Disclosed by Team Code Crusaders*
