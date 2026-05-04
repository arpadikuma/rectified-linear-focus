# Caption Generator

**Status:** Not Started (Priority 1)  
**Target Launch:** Month 1-2  
**Revenue Goal:** $500/month by Month 4

---

## Overview

**Problem:** Creators spend hours writing engaging captions for their content.

**Solution:** Upload image → AI generates multiple caption options → pick favorite → auto-format for platform.

**Target Users:** Instagram photographers, TikTok creators, LinkedIn content creators, content agencies

---

## What It Does

### User Flow
1. Creator uploads image (or batch of images)
2. System generates 3-5 caption options in different styles
3. Creator selects favorite
4. Can refine/edit
5. Export to platform-specific format (Instagram, Twitter, LinkedIn)
6. Optional: Auto-post to multiple platforms

### Features (MVP)
- Single image upload
- 3 caption styles: Cinematic, Emotional, Professional
- Platform formatting (Instagram, Twitter, LinkedIn)
- Hashtag suggestions
- Copy-to-clipboard

### Features (Paid Tier)
- Batch uploads (10+ images at once)
- Custom brand voice training
- Scheduling suggestions
- Advanced analytics (which captions perform best)
- API access
- Priority support

---

## Technical Architecture

### Stack
- **Frontend:** React (simple, fast iteration)
- **Backend:** Python FastAPI
- **Image Analysis:** CLIP embeddings (fast, accurate)
- **Caption Generation:** Claude API (quality, multi-style)
- **Database:** PostgreSQL (user data, captions history)
- **Deployment:** Vercel (frontend) + Railway/Render (backend)

### Data Flow
```
Image Upload
    ↓
CLIP Embedding (visual understanding)
    ↓
LLM Prompt (with style instructions)
    ↓
Generate 5 captions
    ↓
Format for platform
    ↓
Store + display to user
```

### Key Decisions
- **Why CLIP:** Fast, no fine-tuning needed, good visual understanding
- **Why Claude:** Best multi-style capability, cost-effective at scale
- **Why FastAPI:** Fast, easy async handling for image processing
- **Why PostgreSQL:** Reliable, easy to query caption history

---

## Development Checklist

### Phase 1: Core Functionality (Week 1-2)
- [ ] FastAPI backend scaffold
- [ ] Image upload endpoint
- [ ] CLIP embedding generation
- [ ] Claude API integration
- [ ] Caption style prompts (3 styles)
- [ ] Response formatting
- [ ] Local testing working

### Phase 2: Frontend (Week 2-3)
- [ ] React app scaffold
- [ ] Image upload UI
- [ ] Caption display (nice formatting)
- [ ] Style selector
- [ ] Copy/export buttons
- [ ] Basic styling (Tailwind)
- [ ] Mobile responsive

### Phase 3: Platform Export (Week 3)
- [ ] Instagram format (hashtags, line breaks)
- [ ] Twitter format (140 char limit)
- [ ] LinkedIn format (professional tone)
- [ ] Hashtag suggestions integrated
- [ ] Scheduling hints (best posting times)

### Phase 4: Deployment (Week 4)
- [ ] Docker setup for backend
- [ ] Environment variables configured
- [ ] Database migrations
- [ ] Deploy to Railway/Render
- [ ] Frontend to Vercel
- [ ] Domain setup
- [ ] Error monitoring (Sentry)

### Phase 5: Beta Testing (Week 4-5)
- [ ] Create beta testing form
- [ ] Recruit 20 beta users (creators on Instagram/TikTok)
- [ ] Gather feedback on:
  - Caption quality
  - UX/usability
  - What features they'd pay for
  - What doesn't work
- [ ] Document feedback
- [ ] Prioritize fixes

### Phase 6: Monetization (Week 5-6)
- [ ] Design pricing tiers
- [ ] Implement payment (Stripe)
- [ ] Usage tracking (API calls per tier)
- [ ] Free tier limitations
- [ ] Paid tier features
- [ ] Launch paid beta

---

## Pricing Strategy

### Free Tier
- 5 captions/day
- 3 styles (Cinematic, Emotional, Professional)
- Basic export (copy to clipboard)
- No batch processing

### Paid Tier ($9.99/month)
- Unlimited captions
- 5+ custom styles
- All export formats
- Batch processing (50 images/month)
- Style customization
- Scheduling suggestions

### Pro Tier ($29.99/month) - Future
- Everything in Paid
- 500 images/month batch
- API access
- Analytics dashboard
- Priority support
- Custom integrations

---

## Monetization Checklist

- [ ] Payment processor selected (Stripe)
- [ ] Pricing decided
- [ ] Free tier defined (what's limited)
- [ ] Paid tier defined (what's included)
- [ ] Usage tracking implemented
- [ ] Billing system working
- [ ] Subscription management (pause, cancel)
- [ ] Email notifications (payment, quota reset)

---

## User Acquisition

### Phase 1: Direct Outreach (Week 4-5)
- Reach out to 50 creators on Instagram
- Share beta link
- Gather feedback
- Ask for testimonials

### Phase 2: Content (Week 6+)
- Write 2-3 blog posts:
  - "How to Write Captions That Convert"
  - "Caption Generator Case Study: 5 Creators Test It"
  - "AI Caption Tools: What Works?"
- Post on LinkedIn/Twitter
- Ask beta users to share

### Phase 3: Community (Month 2+)
- Photography subreddits
- Creator communities (Discord, Slack groups)
- Hashtag campaigns (#captionsforless #aifortheactress)

---

## Success Metrics

### Usage
- Captions generated per day
- Active users per week
- User retention (% coming back)

### Revenue
- Free users → paid conversion rate (target: 5-10%)
- Average revenue per user
- Monthly recurring revenue

### Quality
- User feedback scores (1-5)
- Support tickets (fewer = better UX)
- Feature requests (what do they want next)

---

## Risks & Mitigation

| Risk | Mitigation |
|------|-----------|
| Image upload fails | Robust error handling, clear feedback |
| Captions are generic | Fine-tune prompts based on user feedback |
| Low conversion to paid | Adjust free tier limits, gather feedback on pricing |
| API costs too high | Monitor usage, optimize prompt length |
| Users want different styles | Track requests, add top 2-3 requested |

---

## Links
- [[VisionAIr]] (related image analysis project)
- [[Aesthetic Scorer]] (next priority project)
- [[Content Calendar]] (for marketing posts)

---

## Progress Log

**2025-04-29:**
- Project created and scoped
- Pricing strategy drafted
- Architecture designed
- Next: Begin Phase 1 (FastAPI backend)

---

## Notes

**Why this project first?**
- Fastest path to deployed product
- Clear user demand (creators need captions)
- Simplest monetization
- Builds momentum for other projects
- Can build on VisionAIr Part 3 learnings
