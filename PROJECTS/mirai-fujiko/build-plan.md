# Build Plan

Day-by-day checklist. Aggressive but realistic given OpenClaw handles the heavy lifting.

---

## Overview

```
Day 1-2:  VPS + OpenClaw + Mirai core
Day 3:    Memory + standup rhythm
Day 4:    Model router on notebook + voice
Day 5:    Telegram channel
Day 6-7:  Fujiko + content generation
Week 2:   Posting pipeline + scheduling + polish
```

---

## Day 1: VPS Setup + OpenClaw Running

**Goal:** OpenClaw accessible in browser, Mirai responds to "hello"

### Morning (2-3 hours)
- [ ] SSH into Oracle VPS
- [ ] Create non-root user with sudo
- [ ] Configure UFW firewall (ssh, 80, 443 only)
- [ ] Create 2GB swap file
- [ ] Update system packages
- [ ] Install Node.js 22 via NodeSource

### Afternoon (2-3 hours)
- [ ] Install OpenClaw: `curl -fsSL https://openclaw.ai/install.sh | bash`
- [ ] Run `openclaw doctor` — all green
- [ ] Run `openclaw onboard --install-daemon`
  - Provider: Anthropic (for now)
  - Channels: skip
  - Skills: yes
  - Daemon: yes
- [ ] Verify gateway running: `openclaw gateway status`
- [ ] Install Nginx
- [ ] Create Nginx config for `mirai.yourdomain.com`
- [ ] Point DNS A record to Oracle VPS IP
- [ ] Install Certbot + get SSL cert
- [ ] Test: open `https://mirai.yourdomain.com` in browser
- [ ] Type "hello" — Mirai responds ✓

**End of Day 1 check:**
Browser at `mirai.yourdomain.com` → Mirai responds ✓

---

## Day 2: Mirai Personality + Vault Access

**Goal:** Mirai knows your projects, can read vault, standup feels real

### Morning (2-3 hours)
- [ ] Set Mirai system prompt (copy from `agents.md`)
- [ ] Configure GitHub Personal Access Token
  - Scopes: `repo` (read + write to rectified-linear-focus)
- [ ] Install GitHub skill in OpenClaw
- [ ] Test: ask Mirai "what's the status of Caption Generator?"
  - She should read the vault file and give accurate answer ✓
- [ ] Test: ask Mirai "what projects am I working on?"
  - She should list all 6 projects with correct status ✓

### Afternoon (2-3 hours)
- [ ] Test checklist reading:
  - "What's left to do on Caption Generator Phase 1?"
  - Should list unchecked items ✓
- [ ] Test checklist writing:
  - "Mark [item] as done"
  - Check vault file on GitHub — should show `<!-- ai: date -->` ✓
- [ ] Adjust system prompt based on how she sounds — refine tone
- [ ] Set Mirai as default agent (handles ambiguous messages)

**End of Day 2 check:**
Mirai reads vault ✓, writes checklist items with attribution ✓

---

## Day 3: SQLite + Standup Rhythm

**Goal:** Database tracking progress, morning standup working at 8:30am

### Morning (2-3 hours)
- [ ] Create SQLite database at `~/.openclaw/mirai.db`
- [ ] Run schema SQL (copy from `database.md`)
- [ ] Seed projects table (copy seed SQL from `database.md`)
- [ ] Import current checklist items from vault into DB
- [ ] Test: query DB, see all projects ✓

### Afternoon (2-3 hours)
- [ ] Configure standup schedule in OpenClaw:
  - `"30 8 * * *"` (8:30am daily)
- [ ] Configure weekly sync:
  - `"30 8 * * 1"` (8:30am Monday)
- [ ] Write standup skill:
  - Reads recent DB progress log
  - Checks git commits (if any since yesterday)
  - Asks: "What's the focus today?"
  - Records answer in sessions table
- [ ] Test standup manually: "good morning mirai"
  - Should give briefing + ask focus ✓
- [ ] Write wrap skill:
  - Triggered by "wrapping up", "done for today"
  - Summarizes session
  - Updates progress log
  - Sets tomorrow's focus
- [ ] Test wrap: "wrapping up, finished the FastAPI scaffold"
  - Should summarize, log, ask for tomorrow's plan ✓

**End of Day 3 check:**
Standup works ✓, wrap works ✓, DB logs sessions ✓

---

## Day 4: Model Router + Voice

**Goal:** Voice input (Parakeet) + voice output (Kokoro) working in browser

### Morning (3-4 hours, on notebook)
- [ ] Create `~/model-router/` directory
- [ ] Create `main.py` (copy from `model-router.md`)
- [ ] Create `.env` with secrets
- [ ] Install dependencies:
  ```bash
  pip install fastapi uvicorn python-multipart httpx
  pip install nemo_toolkit[asr]  # or faster-whisper
  pip install kokoro-onnx soundfile
  ```
- [ ] Download Parakeet v2 model (first run downloads automatically)
- [ ] Download Kokoro model files
- [ ] Run router: `uvicorn main:app --host 0.0.0.0 --port 8000`
- [ ] Test locally: `curl localhost:8000/health`

### Midday (1 hour)
- [ ] Add model-router route to Cloudflare Tunnel config:
  - Subdomain: `models.yourdomain.com` → `localhost:8000`
- [ ] Restart cloudflared
- [ ] Test from VPS: `curl https://models.yourdomain.com/health`
  - Should return services status ✓
- [ ] Set up as startup service (PM2 or systemd)

### Afternoon (2-3 hours)
- [ ] Configure OpenClaw to use model router for STT:
  - Voice input → POST to `https://models.yourdomain.com/stt`
- [ ] Configure OpenClaw for TTS:
  - Response text → POST to `https://models.yourdomain.com/tts`
  - Stream audio back to browser
- [ ] Test voice input in browser: speak "good morning" → Mirai responds ✓
- [ ] Test voice output: Mirai's response plays as audio ✓
- [ ] Verify fallback: stop router, test again → Whisper API handles it ✓

**End of Day 4 check:**
Voice input ✓, voice output ✓, fallback works ✓

---

## Day 5: Telegram

**Goal:** Mirai accessible via Telegram on phone

### Morning (2 hours)
- [ ] Create Telegram bot via @BotFather
  - Name: Mirai
  - Username: your_mirai_bot
  - Save token
- [ ] Run `openclaw onboard` → add Telegram channel
- [ ] Configure allowFrom: your Telegram user ID only
- [ ] Test: send "hello" on Telegram → Mirai responds ✓
- [ ] Test: send voice message → transcribed → Mirai responds ✓
- [ ] Configure Fujiko on second Telegram topic or via keyword routing

### Afternoon (2 hours)
- [ ] Configure Fujiko agent (system prompt from `agents.md`)
- [ ] Test keyword routing:
  - "what should I post" → routes to Fujiko ✓
  - "good morning" → routes to Mirai ✓
- [ ] Test both agents accessible from phone ✓
- [ ] Fine-tune routing keywords as needed
- [ ] Test voice messages from phone:
  - Send voice → Parakeet transcribes → correct agent responds ✓

**End of Day 5 check:**
Telegram working ✓, both agents reachable ✓, voice on phone ✓

---

## Day 6-7: Fujiko + Content Generation

**Goal:** Fujiko generates post bullets + drafts from session notes

### Day 6 Morning (2-3 hours)
- [ ] Write `content-bullets` skill (prompt from `posting.md`)
- [ ] Test: after wrap session → Fujiko generates 5-7 bullets ✓
- [ ] Bullets stored in `content_items` table (status: draft) ✓
- [ ] Review: do bullets capture what actually happened? Refine prompt.

### Day 6 Afternoon (2-3 hours)
- [ ] Write `post-draft` skill (prompt from `posting.md`)
- [ ] Test: "draft this for LinkedIn" → Fujiko writes full post ✓
- [ ] Draft stored in DB, Ikuma reviews in chat ✓
- [ ] Test: "draft this for Medium" → longer version ✓
- [ ] Test: "adapt for Threads" → short version ✓
- [ ] Refine Fujiko's voice to sound like Ikuma, not AI

### Day 7 (2-3 hours)
- [ ] Wire Mirai → Fujiko content escalation:
  - At end of wrap: "Interesting thing happened with X — want content bullets?"
- [ ] Write `content-calendar` skill:
  - "What's queued?" → shows scheduled posts
  - "What's been published this week?" → shows recent
- [ ] Test full flow:
  - Morning standup ✓
  - Work session ✓
  - "wrapping up, finished X" ✓
  - Mirai suggests content ✓
  - Fujiko generates bullets ✓
  - "draft LinkedIn post" ✓
  - Ikuma reviews ✓
  - "approve" ✓
  - "schedule for Tuesday 9am" ✓
  - Confirm in content_items table ✓

**End of Day 7 check:**
Full content workflow end-to-end ✓

---

## Week 2: Posting + Polish

### Day 8-9: LinkedIn Posting
- [ ] Create LinkedIn Developer app
- [ ] Complete OAuth flow, get access token
- [ ] Store token in env
- [ ] Implement `post_to_linkedin()` (code from `posting.md`)
- [ ] Test: approve a post → Fujiko asks "post now or schedule?"
- [ ] Test: "post now" → appears on LinkedIn ✓
- [ ] Test: "schedule for Tuesday 9am" → scheduler queues it ✓
- [ ] Scheduler running, publishes at correct time ✓

### Day 10-11: Medium + Threads
- [ ] Get Medium Integration Token
- [ ] Implement `post_to_medium()` (code from `posting.md`)
- [ ] Test: draft post → creates Medium draft ✓
- [ ] Create Threads Meta Developer app
- [ ] Implement `post_to_threads()` (code from `posting.md`)
- [ ] Test: short post → appears on Threads ✓

### Day 12-14: Polish + Vault Sync
- [ ] Write vault sync job:
  - `sync_db_to_vault()` — updates markdown from DB
  - `sync_vault_to_db()` — detects manual Obsidian changes
- [ ] Schedule: vault sync every Sunday
- [ ] Test: mark item done in Obsidian → DB detects it (manual attribution) ✓
- [ ] Test: Mirai marks item done → vault shows `<!-- ai: date -->` ✓
- [ ] Set up weekly backup script (copy DB to safe location)
- [ ] Write health check cron (restarts OpenClaw if needed)
- [ ] Mobile browser test: all functions working on phone ✓
- [ ] Performance check: voice latency acceptable? ✓
- [ ] Documentation: update this build plan with any deviations

---

## Full System Test (End of Week 2)

Run through this scenario end-to-end:

```
8:30am: Mirai sends standup (scheduled)
  ✓ Reviews yesterday's progress
  ✓ Notes git commits
  ✓ Asks today's focus

Morning: Work on Caption Generator
  ✓ Ask Mirai "what's left on Phase 1?"
  ✓ Mirai reads vault, lists items

Midday: "mark FastAPI scaffold as done"
  ✓ Mirai marks in DB + vault
  ✓ GitHub commit created with [auto] tag

Afternoon: Stuck on CLIP integration
  ✓ Tell Mirai "blocked on CLIP deps"
  ✓ Mirai logs blocker, suggests breaking it down

End of day: "wrapping up, fixed the CLIP issue finally"
  ✓ Mirai summarizes session
  ✓ Updates progress log
  ✓ "Interesting debug story — want content bullets?"

Content: "yes, LinkedIn post"
  ✓ Fujiko generates 5-7 bullets
  ✓ Ikuma reviews: "draft this"
  ✓ Fujiko drafts LinkedIn post
  ✓ Ikuma edits, says "approve"
  ✓ "Schedule for Tuesday 8:30am"
  ✓ Scheduler queues it

Tuesday 8:30am: Post publishes automatically
  ✓ Fujiko confirms: "Posted. [link]"
```

---

## Progress Log

**2025-04-29:**
- Architecture document created
- Build plan drafted
- Next: Begin Day 1 (VPS setup)

---

## Links
- [[agents]] (Mirai + Fujiko design)
- [[infrastructure]] (VPS setup steps)
- [[model-router]] (notebook FastAPI)
- [[database]] (schema + seed)
- [[posting]] (API integrations)
