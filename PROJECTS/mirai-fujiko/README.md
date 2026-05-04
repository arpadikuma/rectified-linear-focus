# Mirai & Fujiko: AI Chief of Staff System

**Status:** Architecture phase → Building  
**Target:** Fully operational in 2 weeks  
**Lives in:** Oracle VPS + local notebook models

---

## What This Is

A persistent, voice-enabled AI team that runs 24/7 and simulates working in a real product team. Two agents with distinct roles, accessible from browser, Telegram, and eventually Discord.

**Mirai** — Chief of Staff. Runs your day. Standups, progress tracking, vault updates, accountability.  
**Fujiko** — Strategist. Maps your direction. Content, product decisions, posting, timeline pressure.

---

## System Overview

```
┌─────────────────────────────────────────────────────────┐
│                  ORACLE FREE VPS                         │
│                  (4 ARM64 cores, 24GB RAM)               │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  OpenClaw Gateway (port 18789)                  │    │
│  │    ├── Agent: Mirai (Chief of Staff)            │    │
│  │    ├── Agent: Fujiko (Strategist)               │    │
│  │    ├── SQLite DB (→ Postgres when needed)       │    │
│  │    └── Scheduler (standups, wraps, syncs)       │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  ┌──────────────────────────────────────────────── ┐    │
│  │  Nginx Reverse Proxy                            │    │
│  │    ├── mirai.yourdomain.com → WebChat UI        │    │
│  │    └── SSL via Let's Encrypt                    │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Model Router (internal only)                   │    │
│  │    Calls local or cloud depending on            │    │
│  │    notebook availability                        │    │
│  └─────────────────────────────────────────────────┘    │
└───────────────────────┬─────────────────────────────────┘
                        │
          ┌─────────────┴──────────────┐
          │                            │
          │ Cloudflare Tunnel          │ Public HTTPS
          ▼                            ▼
┌─────────────────┐         ┌──────────────────────┐
│  YOUR NOTEBOOK  │         │  CHANNELS            │
│                 │         │                      │
│  FastAPI Router │         │  Browser WebChat     │
│  :8000          │         │  (desktop + mobile)  │
│   ├── /stt      │         │                      │
│   │   Parakeet  │         │  Telegram Bot        │
│   ├── /tts      │         │  (mobile-first)      │
│   │   Kokoro    │         │                      │
│   └── /llm      │         │  Discord (later)     │
│       Ollama    │         └──────────────────────┘
└─────────────────┘
          │ fallback
          ▼
┌─────────────────────────────────────────────────────────┐
│  CLOUD API FALLBACKS                                     │
│   STT: OpenAI Whisper API                                │
│   TTS: OpenAI TTS                                        │
│   LLM: Anthropic Claude / OpenAI                        │
│   IMG: Replicate / DALL-E (future)                      │
└─────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────┐
│  EXTERNAL INTEGRATIONS                                   │
│   GitHub API  → vault read/write (rectified-linear-focus)│
│   LinkedIn API → posting                                 │
│   Medium API  → posting                                  │
│   Threads API → posting                                  │
└─────────────────────────────────────────────────────────┘
```

---

## Files in This Folder

- `README.md` — This file (system overview)
- `agents.md` — Mirai + Fujiko personalities, system prompts, skills
- `infrastructure.md` — VPS setup, Cloudflare Tunnel, Nginx, OpenClaw install
- `model-router.md` — FastAPI router on notebook (STT/TTS/LLM + fallbacks)
- `database.md` — SQLite schema, attribution system, vault sync
- `skills/` — Individual skill designs for each agent
- `build-plan.md` — Day-by-day build checklist
- `posting.md` — LinkedIn, Medium, Threads API integration

---

## Links
- [[rectified-linear-focus/README]] (the vault this system manages)
- [[caption-generator/README]] (first product Fujiko tracks)
- [[visionair/README]] (project Mirai tracks)
