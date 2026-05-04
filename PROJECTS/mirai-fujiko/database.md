# Database

SQLite for speed and simplicity. Schema designed for clean Postgres migration later.

---

## Philosophy

Every change tracked with source: `ai` (Mirai/Fujiko wrote it) or `manual` (Ikuma wrote it directly in Obsidian). This makes it clear at a glance what the agent did vs. what you decided.

In the vault (Obsidian markdown):
```markdown
- [x] FastAPI scaffold complete <!-- ai: 2025-05-02 -->   ← Mirai marked this
- [x] CLIP integration done                               ← You marked this manually
- [ ] Deploy to staging                                   ← Not done yet
```

In the DB: full history, timestamps, who changed what.

---

## Schema

```sql
-- ─── Projects ────────────────────────────────────────────────────────────

CREATE TABLE projects (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    slug        TEXT NOT NULL UNIQUE,   -- 'caption-generator'
    name        TEXT NOT NULL,          -- 'Caption Generator'
    phase       TEXT NOT NULL,          -- 'phase1', 'phase2a', etc.
    status      TEXT NOT NULL DEFAULT 'not_started',
                                        -- not_started | in_progress | 
                                        --   deployed | paused | complete
    priority    INTEGER DEFAULT 0,      -- lower = higher priority
    vault_path  TEXT,                   -- 'PROJECTS/caption-generator/README.md'
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- ─── Checklist Items ─────────────────────────────────────────────────────

CREATE TABLE checklist_items (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    project_id  INTEGER NOT NULL REFERENCES projects(id),
    section     TEXT,                   -- 'Phase 1: Core Functionality'
    text        TEXT NOT NULL,          -- 'FastAPI backend scaffold'
    done        INTEGER DEFAULT 0,      -- 0 | 1
    source      TEXT DEFAULT 'manual',  -- 'ai' | 'manual'
    done_at     DATETIME,
    done_by     TEXT,                   -- 'mirai' | 'fujiko' | 'ikuma'
    vault_line  INTEGER,                -- line number in vault file
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- ─── Progress Log ────────────────────────────────────────────────────────

CREATE TABLE progress_log (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    date        DATE NOT NULL,
    project_id  INTEGER REFERENCES projects(id),
    entry       TEXT NOT NULL,          -- what happened
    source      TEXT DEFAULT 'manual',  -- 'ai' | 'manual'
    agent       TEXT,                   -- 'mirai' | 'fujiko' | null
    session_id  TEXT,                   -- groups entries from same session
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- ─── Sessions (standup/wrap tracking) ────────────────────────────────────

CREATE TABLE sessions (
    id          TEXT PRIMARY KEY,       -- UUID
    type        TEXT NOT NULL,          -- 'standup' | 'wrap' | 'weekly' | 'chat'
    started_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    ended_at    DATETIME,
    summary     TEXT,                   -- Mirai's end-of-session summary
    focus       TEXT,                   -- what Ikuma said was today's focus
    shipped     TEXT,                   -- what actually got done (JSON array)
    blockers    TEXT,                   -- any blockers noted
    mood        TEXT                    -- optional: Ikuma's energy level
);

-- ─── Content Pipeline ────────────────────────────────────────────────────

CREATE TABLE content_items (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    project_id  INTEGER REFERENCES projects(id),
    type        TEXT NOT NULL,          -- 'linkedin' | 'medium' | 'threads'
    status      TEXT DEFAULT 'draft',   -- draft | review | approved | scheduled | published
    title       TEXT,
    bullets     TEXT,                   -- JSON array of bullet points (Fujiko generated)
    draft       TEXT,                   -- full post draft
    final       TEXT,                   -- approved final version
    scheduled_at DATETIME,
    published_at DATETIME,
    platform_id TEXT,                   -- ID returned by platform API after posting
    source      TEXT DEFAULT 'fujiko',  -- 'fujiko' | 'manual'
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- ─── Metrics ─────────────────────────────────────────────────────────────

CREATE TABLE metrics (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    date        DATE NOT NULL,
    project_id  INTEGER REFERENCES projects(id),
    metric      TEXT NOT NULL,          -- 'users_free' | 'users_paid' | 'mrr' | 'posts_published'
    value       REAL NOT NULL,
    source      TEXT DEFAULT 'manual',  -- 'ai' | 'manual' | 'api'
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- ─── Vault Sync Log ──────────────────────────────────────────────────────

CREATE TABLE vault_syncs (
    id          INTEGER PRIMARY KEY AUTOINCREMENT,
    synced_at   DATETIME DEFAULT CURRENT_TIMESTAMP,
    direction   TEXT NOT NULL,          -- 'db_to_vault' | 'vault_to_db'
    files       TEXT,                   -- JSON array of changed files
    changes     INTEGER DEFAULT 0,      -- number of changes made
    status      TEXT DEFAULT 'ok',      -- 'ok' | 'error'
    error       TEXT                    -- error message if failed
);

-- ─── Indexes ─────────────────────────────────────────────────────────────

CREATE INDEX idx_checklist_project ON checklist_items(project_id);
CREATE INDEX idx_checklist_done ON checklist_items(done);
CREATE INDEX idx_progress_date ON progress_log(date);
CREATE INDEX idx_content_status ON content_items(status);
CREATE INDEX idx_metrics_date ON metrics(date);
CREATE INDEX idx_metrics_project ON metrics(project_id, metric);
```

---

## Seed Data

```sql
-- Insert your projects
INSERT INTO projects (slug, name, phase, status, priority, vault_path) VALUES
('caption-generator',       'Caption Generator',        'phase2a', 'not_started', 1, 'PROJECTS/caption-generator/README.md'),
('aesthetic-scorer',        'Aesthetic Scorer',          'phase2b', 'not_started', 2, 'PROJECTS/aesthetic-scorer/README.md'),
('repo-analyzer',           'Repo Analyzer',             'phase2c', 'not_started', 3, 'PROJECTS/repo-analyzer/README.md'),
('visionair',               'VisionAIr',                 'phase1',  'in_progress', 4, 'PROJECTS/visionair/README.md'),
('fashion-assistant',       'Fashion Assistant',         'phase1',  'in_progress', 5, 'PROJECTS/fashion-assistant/README.md'),
('mlops-energy-forecasting','MLOps Energy Forecasting',  'phase1',  'paused',      6, 'PROJECTS/mlops-energy-forecasting/README.md'),
('mirai-fujiko',            'Mirai & Fujiko System',     'infra',   'in_progress', 0, 'PROJECTS/mirai-fujiko/README.md');
```

---

## Attribution Flow

### When Mirai marks a checklist item done:

**1. DB update:**
```python
db.execute("""
    UPDATE checklist_items
    SET done=1, source='ai', done_at=?, done_by='mirai'
    WHERE id=?
""", (datetime.now(), item_id))
```

**2. Vault sync (writes back to markdown):**
```python
# Replaces: - [ ] FastAPI backend scaffold
# With:     - [x] FastAPI backend scaffold <!-- ai: 2025-05-02 -->
```

**3. GitHub commit:**
```
git commit -m "mirai: mark FastAPI scaffold complete [auto]"
```

### When you mark something manually in Obsidian:

**1. Vault sync detects change (no `<!-- ai: ... -->` comment):**
```python
# Sees: - [x] Task name  (no comment)
# Updates DB: source='manual', done_by='ikuma'
```

**2. DB update:**
```python
db.execute("""
    UPDATE checklist_items
    SET done=1, source='manual', done_at=?, done_by='ikuma'
    WHERE vault_path=? AND vault_line=?
""", (datetime.now(), vault_path, line_number))
```

---

## Vault Sync Design

Runs weekly (or on demand: "sync vault").

```python
async def sync_db_to_vault():
    """
    Read DB state → update markdown files → commit to GitHub
    Only updates items changed since last sync
    """
    changed_files = []
    
    for project in get_all_projects():
        vault_content = fetch_from_github(project.vault_path)
        updated = apply_checklist_changes(vault_content, project)
        
        if updated != vault_content:
            push_to_github(project.vault_path, updated, 
                          f"mirai: weekly vault sync [auto]")
            changed_files.append(project.vault_path)
    
    log_sync("db_to_vault", changed_files)


async def sync_vault_to_db():
    """
    Read markdown files → detect manual changes → update DB
    Runs at standup time to catch overnight edits
    """
    for project in get_all_projects():
        vault_content = fetch_from_github(project.vault_path)
        manual_changes = detect_manual_changes(vault_content, project)
        
        for change in manual_changes:
            update_checklist_item(change, source='manual', done_by='ikuma')
```

---

## Postgres Migration (when needed)

Schema is Postgres-compatible. When you're ready:

```bash
# Export SQLite
sqlite3 mirai.db .dump > mirai_dump.sql

# Minor adjustments needed:
# - AUTOINCREMENT → SERIAL
# - DATETIME → TIMESTAMP
# - INTEGER (boolean) → BOOLEAN

# Import to Postgres
psql mirai_prod < mirai_dump_adjusted.sql
```

Oracle Free Tier includes a managed Postgres instance — zero cost when you're ready.

---

## Links
- [[agents]] (what reads/writes the DB)
- [[infrastructure]] (where SQLite lives)
- [[model-router]] (doesn't use DB directly)
