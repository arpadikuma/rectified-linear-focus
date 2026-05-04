# Posting Integrations

Fujiko's publishing pipeline. Draft → Review → Approve → Schedule → Publish.

---

## Overall Flow

```
Fujiko generates bullets (from session notes / Mirai escalation)
    ↓
Ikuma reviews bullets, says "draft this"
    ↓
Fujiko writes full post (platform-specific)
    ↓
Ikuma reviews draft, edits if needed, says "approve"
    ↓
Fujiko asks: "Post now or schedule?"
    ↓
If schedule: Fujiko adds to queue with datetime
    ↓
Scheduler publishes at specified time
    ↓
Fujiko confirms: "Posted to LinkedIn. Here's the link."
```

---

## LinkedIn

### API Setup
LinkedIn uses OAuth 2.0. For personal posting you need:
- LinkedIn Developer App (free)
- OAuth token with `w_member_social` scope

```bash
# Get token (one-time, lasts 60 days — needs refresh)
# 1. Create app at developers.linkedin.com
# 2. Add "Share on LinkedIn" product
# 3. OAuth flow to get access token
# 4. Store in LINKEDIN_ACCESS_TOKEN env var
```

### Post Code (OpenClaw skill)

```python
import httpx
import os

async def post_to_linkedin(text: str, article_url: str = None) -> dict:
    """
    Post text update to LinkedIn.
    Optionally attach article URL for rich preview.
    """
    token = os.getenv("LINKEDIN_ACCESS_TOKEN")
    if not token:
        raise Exception("LinkedIn token not configured")

    # Get user URN (cached after first call)
    urn = await _get_linkedin_urn(token)

    payload = {
        "author": f"urn:li:person:{urn}",
        "lifecycleState": "PUBLISHED",
        "specificContent": {
            "com.linkedin.ugc.ShareContent": {
                "shareCommentary": {
                    "text": text
                },
                "shareMediaCategory": "NONE" if not article_url else "ARTICLE"
            }
        },
        "visibility": {
            "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC"
        }
    }

    # Add article if URL provided
    if article_url:
        payload["specificContent"]["com.linkedin.ugc.ShareContent"]["media"] = [{
            "status": "READY",
            "originalUrl": article_url,
            "shareMediaCategory": "ARTICLE"
        }]

    async with httpx.AsyncClient() as client:
        r = await client.post(
            "https://api.linkedin.com/v2/ugcPosts",
            headers={
                "Authorization": f"Bearer {token}",
                "Content-Type": "application/json",
                "X-Restli-Protocol-Version": "2.0.0"
            },
            json=payload
        )
        r.raise_for_status()
        post_id = r.headers.get("x-restli-id", "unknown")
        return {
            "platform": "linkedin",
            "post_id": post_id,
            "url": f"https://www.linkedin.com/feed/update/{post_id}/"
        }


async def _get_linkedin_urn(token: str) -> str:
    """Get LinkedIn user URN (cached)"""
    if hasattr(_get_linkedin_urn, "_urn"):
        return _get_linkedin_urn._urn
    
    async with httpx.AsyncClient() as client:
        r = await client.get(
            "https://api.linkedin.com/v2/me",
            headers={"Authorization": f"Bearer {token}"}
        )
        r.raise_for_status()
        _get_linkedin_urn._urn = r.json()["id"]
        return _get_linkedin_urn._urn
```

### Token Refresh
LinkedIn tokens expire after 60 days. Set a calendar reminder or use the refresh flow:

```python
async def refresh_linkedin_token():
    """
    Refresh LinkedIn token before it expires.
    Run monthly via scheduler.
    """
    # LinkedIn doesn't support automatic refresh for personal tokens
    # You'll need to re-authorize via OAuth
    # Mirai will remind you 7 days before expiry
    pass
```

---

## Medium

Medium has two options: **Integration Token** (simple) or **API** (more control).

### Option A: Integration Token (Recommended to Start)
```bash
# 1. Go to medium.com/me/settings
# 2. Integration tokens → Generate token
# 3. Store as MEDIUM_INTEGRATION_TOKEN
```

```python
async def post_to_medium(
    title: str,
    content: str,
    tags: list = None,
    publish_status: str = "draft"  # or "public"
) -> dict:
    """
    Create Medium post.
    publish_status: 'draft' (safe) or 'public' (live immediately)
    Recommend: always draft first, then publish from Medium UI
    """
    token = os.getenv("MEDIUM_INTEGRATION_TOKEN")
    user_id = await _get_medium_user_id(token)

    payload = {
        "title": title,
        "contentFormat": "markdown",
        "content": content,
        "tags": tags or [],
        "publishStatus": publish_status
    }

    async with httpx.AsyncClient() as client:
        r = await client.post(
            f"https://api.medium.com/v1/users/{user_id}/posts",
            headers={
                "Authorization": f"Bearer {token}",
                "Content-Type": "application/json"
            },
            json=payload
        )
        r.raise_for_status()
        data = r.json()["data"]
        return {
            "platform": "medium",
            "post_id": data["id"],
            "url": data["url"]
        }


async def _get_medium_user_id(token: str) -> str:
    if hasattr(_get_medium_user_id, "_id"):
        return _get_medium_user_id._id
    
    async with httpx.AsyncClient() as client:
        r = await client.get(
            "https://api.medium.com/v1/me",
            headers={"Authorization": f"Bearer {token}"}
        )
        r.raise_for_status()
        _get_medium_user_id._id = r.json()["data"]["id"]
        return _get_medium_user_id._id
```

---

## Threads

Threads uses Meta's Graph API (same as Instagram).

### API Setup
```bash
# 1. Create Meta Developer account
# 2. Create app → Add Threads product
# 3. Get access token with threads_content_publish scope
# 4. Store as THREADS_ACCESS_TOKEN
```

```python
async def post_to_threads(text: str) -> dict:
    """
    Post text to Threads.
    Threads: 500 char limit, text-only or with link.
    """
    token = os.getenv("THREADS_ACCESS_TOKEN")
    user_id = os.getenv("THREADS_USER_ID")

    # Step 1: Create media container
    async with httpx.AsyncClient() as client:
        r = await client.post(
            f"https://graph.threads.net/v1.0/{user_id}/threads",
            params={
                "media_type": "TEXT",
                "text": text[:500],  # enforce limit
                "access_token": token
            }
        )
        r.raise_for_status()
        container_id = r.json()["id"]

        # Step 2: Publish container
        r2 = await client.post(
            f"https://graph.threads.net/v1.0/{user_id}/threads_publish",
            params={
                "creation_id": container_id,
                "access_token": token
            }
        )
        r2.raise_for_status()
        return {
            "platform": "threads",
            "post_id": r2.json()["id"],
            "url": f"https://www.threads.net/@yourusername"
        }
```

---

## Scheduler

Simple cron-style scheduler for queued posts.

```python
# In OpenClaw scheduler config or as a standalone cron

async def check_scheduled_posts():
    """Run every 5 minutes. Publish posts whose scheduled_at has passed."""
    
    pending = db.execute("""
        SELECT * FROM content_items
        WHERE status = 'scheduled'
        AND scheduled_at <= datetime('now')
        ORDER BY scheduled_at ASC
    """).fetchall()

    for post in pending:
        try:
            result = await publish_post(post)
            db.execute("""
                UPDATE content_items
                SET status='published',
                    published_at=datetime('now'),
                    platform_id=?
                WHERE id=?
            """, (result["post_id"], post["id"]))
        except Exception as e:
            logger.error(f"Failed to publish post {post['id']}: {e}")
            # Don't mark as failed immediately — retry up to 3 times


async def publish_post(post: dict) -> dict:
    """Route to correct platform publisher"""
    if post["type"] == "linkedin":
        return await post_to_linkedin(post["final"])
    elif post["type"] == "medium":
        return await post_to_medium(
            title=post["title"],
            content=post["final"],
            publish_status="public"
        )
    elif post["type"] == "threads":
        return await post_to_threads(post["final"])
    else:
        raise ValueError(f"Unknown platform: {post['type']}")
```

---

## Fujiko's Content Workflow (OpenClaw Skill)

```python
# skill: content-bullets
# Triggered after wrap session or "what should I post"

BULLETS_PROMPT = """
You are Fujiko, content strategist for Ikuma.

Based on today's session notes:
{session_notes}

And the current project status:
{project_context}

Generate 5-7 bullet points for a technical post.
These are raw material Ikuma will use to write his own post.

Rules:
- Each bullet = one specific, concrete idea (not generic)
- Pull from real things that happened today
- Find the interesting angle (what would a developer actually want to know?)
- Format: "• [bullet]"
- End with: "Platform suggestion: [LinkedIn/Medium/Threads] — [one sentence reason]"
"""

# skill: post-draft
# Triggered when Ikuma says "draft this" after reviewing bullets

DRAFT_PROMPT = """
You are Fujiko, content strategist for Ikuma.

Write a {platform} post from these bullet points:
{bullets}

Ikuma's voice:
- Direct, no fluff
- Specific (numbers, real examples)
- Honest about what didn't work
- Photographer-turned-AI-engineer perspective
- Never uses: "I'm excited to share", "game-changer", "revolutionary"

Platform specs:
LinkedIn: 150-300 words, hook in first line, 3-4 hashtags, link to repo
Medium: 800-1500 words, technical depth, code where relevant
Threads: Under 500 chars, punchy, informal

Write the post. Then on a new line write:
"SUGGESTED HASHTAGS:" followed by 3-5 hashtags.
"SUGGESTED SCHEDULE:" followed by best day/time to post.
"""
```

---

## Setup Checklist

- [ ] LinkedIn Developer app created
- [ ] LinkedIn OAuth token obtained (scope: w_member_social)
- [ ] LinkedIn URN retrieved and tested
- [ ] Medium Integration Token generated
- [ ] Medium user ID retrieved and tested
- [ ] Threads Meta Developer app created
- [ ] Threads access token obtained
- [ ] Threads user ID configured
- [ ] Scheduler running (every 5 min check)
- [ ] Test post sent to each platform (then deleted)
- [ ] Token expiry reminders set (LinkedIn: 60 days)

---

## Links
- [[agents]] (Fujiko's skills that use these)
- [[database]] (content_items table)
- [[build-plan]] (when to set this up)
