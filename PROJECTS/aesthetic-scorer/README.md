# Aesthetic Scorer

**Status:** Not Started (Priority 2)  
**Target Launch:** Month 3-4  
**Revenue Goal:** $500/month by Month 6

---

## Overview

**Problem:** Photographers want to understand why some of their images work better than others.

**Solution:** Upload image → AI scores aesthetic quality (1-10) → breakdown of what works/doesn't → suggestions for improvement.

**Target Users:** Photographers, visual content creators, photographers learning their craft, photography educators

---

## What It Does

### User Flow
1. Photographer uploads image
2. System analyzes aesthetic qualities:
   - Composition
   - Lighting
   - Color harmony
   - Subject clarity
   - Overall visual impact
3. Generates overall score (1-10)
4. Detailed breakdown: what's strong, what could improve
5. Actionable suggestions (e.g., "Increase contrast on subject", "Crop to rule of thirds")

### Features (MVP)
- Single image upload
- Aesthetic score (1-10)
- Breakdown by category (composition, lighting, color, subject)
- Key strengths identified
- Improvement suggestions
- Comparison (upload new version, see score improvement)

### Features (Paid Tier)
- Batch processing (50-500 images)
- Detailed analysis reports (PDF export)
- Before/after comparison
- Style preferences training ("I like cinematic, darker tones")
- Portfolio analysis (score all your images, see patterns)
- API access for photographers
- Detailed feedback (2-3 paragraphs per image)

---

## Technical Architecture

### Stack
- **Frontend:** React
- **Backend:** Python FastAPI
- **Aesthetic Model:** Fine-tuned LLM + CLIP embeddings
- **Database:** PostgreSQL (scores, analyses, user preferences)
- **Deployment:** Similar to Caption Generator

### Data Flow
```
Image Upload
    ↓
CLIP Embedding (visual features)
    ↓
Fine-tuned Model (aesthetic scoring)
    ↓
Category-specific scoring (composition, lighting, etc.)
    ↓
Generate suggestions
    ↓
Store + display to user
```

### Key Questions
- Which aesthetic model to use?
  - Option A: Fine-tune own model on aesthetic datasets
  - Option B: Use pretrained aesthetic model (NIMA, etc.) + enhance with LLM
  - Option C: Hybrid (pretrained + fine-tuning on specific style preferences)
- How to get training data?
  - Photography course datasets
  - Public image datasets with aesthetic ratings
  - User feedback (users rate if suggestions were helpful)

---

## Development Checklist

### Phase 1: Research & Model Selection (Week 1-2)
- [ ] Research existing aesthetic scoring models
- [ ] Identify best datasets for fine-tuning
- [ ] Test pretrained models (NIMA, etc.)
- [ ] Design scoring categories
- [ ] Prototype scoring logic

### Phase 2: Backend Development (Week 2-3)
- [ ] FastAPI scaffold
- [ ] Image upload endpoint
- [ ] CLIP embeddings
- [ ] Aesthetic scoring model integration
- [ ] Category-specific scoring
- [ ] Suggestion generation (LLM prompts)
- [ ] Database schema

### Phase 3: Frontend (Week 3-4)
- [ ] React app scaffold
- [ ] Image upload UI
- [ ] Score display (visual, clear)
- [ ] Category breakdown (bars, colors)
- [ ] Suggestions display
- [ ] Comparison interface
- [ ] Basic styling

### Phase 4: Deployment (Week 4)
- [ ] Docker setup
- [ ] Environment configuration
- [ ] Deploy backend
- [ ] Deploy frontend
- [ ] Database setup

### Phase 5: Beta Testing (Week 5)
- [ ] Recruit 15-20 photographers
- [ ] Gather feedback on:
  - Accuracy of scores (do they agree?)
  - Quality of suggestions
  - What categories matter most
  - Would they pay? How much?
- [ ] Iterate on model/suggestions

### Phase 6: Refinement (Week 5-6)
- [ ] Improve scoring accuracy based on feedback
- [ ] Fine-tune suggestions
- [ ] Add comparison feature
- [ ] Prepare for paid launch

---

## Monetization Strategy

### Free Tier
- 3 scores/day
- Basic aesthetic score
- General suggestions
- No batch processing
- No detailed analysis

### Paid Tier ($12.99/month)
- Unlimited scores
- Detailed 2-3 paragraph analysis
- Category breakdown with reasoning
- Batch processing (50 images/month)
- Before/after comparison
- Export analysis (PDF)
- Style preferences training

### Pro Tier ($39.99/month) - Future
- Everything in Paid
- 500 images/month
- API access
- Portfolio analytics dashboard
- Custom feedback (longer analyses)
- Priority support

---

## Monetization Checklist

- [ ] Scoring accuracy validated (beta feedback)
- [ ] Suggestion quality approved
- [ ] Pricing decided and tested
- [ ] Payment integration (Stripe)
- [ ] Usage tracking
- [ ] Tier limitations coded
- [ ] Billing system
- [ ] Email notifications

---

## User Acquisition

### Phase 1: Photography Communities (Month 3)
- Photography subreddits: r/photography, r/postprocessing
- Photography Discord communities
- Photography Facebook groups
- Photography Slack groups

### Phase 2: Content Strategy (Month 3-4)
- Blog post: "What Makes a Photo Aesthetically Strong?"
- Case study: "I Scored 100 Photos - Here's What I Learned"
- LinkedIn posts on photography + AI
- Tutorial: "How to Use Feedback to Improve Your Photography"

### Phase 3: Partnerships (Month 4+)
- Photography educators (offer free tier for students)
- Photography courses (integrate as tool)
- Photography communities

---

## Success Metrics

### Quality
- User agreement with scores (survey: "Do you agree with this score?")
- Suggestion actionability (users report implementing suggestions)
- User feedback scores

### Revenue
- Free → paid conversion rate (target: 5-10%)
- Average revenue per user
- Monthly recurring revenue

### Usage
- Scores generated per day
- Active users
- Retention rate

---

## Risks & Mitigation

| Risk | Mitigation |
|------|-----------|
| Scoring feels arbitrary | Get feedback early, refine model |
| Photographers disagree with scores | Explain reasoning in detail, allow feedback |
| Model is biased (toward certain styles) | Train on diverse datasets, let users adjust preferences |
| Low demand (photographers don't want this) | Validate in beta, adjust messaging if needed |
| Expensive to run at scale | Monitor API costs, optimize model |

---

## Timeline

- **Month 1:** Caption Generator (priority 1)
- **Month 2:** Caption Generator beta + content
- **Month 3:** Begin Aesthetic Scorer development
- **Month 4:** Aesthetic Scorer beta testing
- **Month 4-5:** Both running, gathering revenue

---

## Links
- [[Caption Generator]] (priority 1)
- [[VisionAIr]] (Part 1 is related - image analysis)
- [[Repo Analyzer]] (priority 3)

---

## Progress Log

**2025-04-29:**
- Project scoped
- Monetization strategy drafted
- Technical approach outlined
- Next: Priority is Caption Generator; return to this in Month 3

---

## Notes

**Why this as #2?**
- Complements Caption Generator (both serve creators)
- Different niche (photographers vs. general creators)
- Good second product to test SaaS model
- Builds on aesthetic expertise from photography background
