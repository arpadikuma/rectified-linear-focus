# Repo Analyzer

**Status:** Not Started (Priority 3)  
**Target Launch:** Month 5+  
**Revenue Goal:** $200/month by Month 8

---

## Overview

**Problem:** Developers want to understand project quality, architecture, and improvement opportunities for interviews/portfolios/learning.

**Solution:** Paste GitHub repo link → AI analyzes codebase → generates report on architecture, code quality, improvement suggestions → generates resume bullets.

**Target Users:** Junior developers, students, developers interviewing, portfolio builders, engineering managers reviewing code

---

## What It Does

### User Flow
1. Developer enters GitHub repo URL (public)
2. System clones repo + analyzes
3. Generates report:
   - Architecture overview
   - Code quality assessment
   - Tech stack breakdown
   - Best practices found
   - Improvements suggested
   - Key learnings for resume
4. Export as PDF or share link
5. Get 3-5 resume bullet points

### Features (MVP)
- GitHub repo analysis
- Architecture summary (2-3 paragraphs)
- Tech stack identified
- Code quality score
- 3-5 improvement suggestions
- 5 resume bullets generated
- Simple dashboard (view past analyses)

### Features (Paid Tier)
- Batch analysis (10 repos at once)
- Detailed 10+ page PDF report
- Code quality metrics (complexity, coverage, etc.)
- Team comparison (how does this compare to industry standards)
- Interview preparation guide (how to talk about this in interviews)
- Custom feedback (longer analysis)
- API access for recruiters/teams

---

## Technical Architecture

### Stack
- **Frontend:** React
- **Backend:** Python FastAPI
- **GitHub API:** Fetch repo structure, code
- **Code Analysis:** AST parsing + LLM analysis
- **Database:** PostgreSQL (analyses, user projects)
- **Deployment:** Similar to other products

### Data Flow
```
GitHub Repo URL
    ↓
Clone/fetch via GitHub API
    ↓
Extract file structure, key files
    ↓
AST parse for code metrics (complexity, lines, etc.)
    ↓
LLM analysis (architecture, quality, suggestions)
    ↓
Generate resume bullets
    ↓
Generate PDF report
    ↓
Store + display to user
```

### Key Questions
- What metrics to calculate?
  - Lines of code
  - Code complexity (cyclomatic)
  - Number of functions/classes
  - Dependencies
  - Test coverage (if available)
- What to analyze with LLM?
  - Architecture patterns
  - Best practices
  - Code quality observations
  - Resume bullet generation

---

## Development Checklist

### Phase 1: Research & Prototyping (Week 1)
- [ ] Test GitHub API
- [ ] Prototype code parsing
- [ ] Design analysis prompts
- [ ] Research code quality metrics
- [ ] Plan report generation

### Phase 2: Backend Development (Week 2-3)
- [ ] FastAPI scaffold
- [ ] GitHub API integration
- [ ] Repo cloning/fetching
- [ ] AST parsing + metrics
- [ ] LLM analysis integration
- [ ] Resume bullet generation
- [ ] PDF generation
- [ ] Database schema

### Phase 3: Frontend (Week 3-4)
- [ ] React app scaffold
- [ ] Repo URL input
- [ ] Analysis results display
- [ ] Report visualization
- [ ] Resume bullets editor
- [ ] PDF download button
- [ ] User dashboard (past analyses)

### Phase 4: Deployment (Week 4)
- [ ] Docker setup
- [ ] Environment config
- [ ] Deploy backend + frontend
- [ ] GitHub Auth (optional)

### Phase 5: Beta Testing (Week 5)
- [ ] Recruit 10-15 developers (students + early career)
- [ ] Gather feedback on:
  - Accuracy of analysis
  - Usefulness of resume bullets
  - Report quality
  - Would they pay?
- [ ] Iterate based on feedback

---

## Monetization Strategy

### Free Tier
- 2 analyses/month
- Basic report (2-3 pages)
- 3 resume bullets
- No PDF export
- 24-hour link share (expires)

### Paid Tier ($9.99/month)
- Unlimited analyses
- Full PDF report
- 5-7 resume bullets
- Interview prep guide
- Permanent link storage
- Past analyses dashboard

### Pro Tier ($29.99/month) - Future
- Everything in Paid
- Batch analysis (10 repos)
- Team analysis (compare repos)
- Advanced metrics dashboard
- API access
- Priority support

---

## Monetization Checklist

- [ ] Report quality validated in beta
- [ ] Resume bullets are actually useful
- [ ] Pricing decided
- [ ] Payment integration
- [ ] Usage tracking
- [ ] Tier limitations
- [ ] Billing system

---

## User Acquisition

### Phase 1: Student Communities (Month 5)
- Reddit: r/learnprogramming, r/webdev, r/coding
- Dev communities: Dev.to, Indie Hackers
- Student Discord/communities

### Phase 2: Content Strategy (Month 5-6)
- Blog post: "How to Analyze Your Code for Interviews"
- Case study: "I Analyzed 20 Junior Dev Repos - Here's What I Learned"
- Twitter: Tips for portfolio review
- LinkedIn: Article on code quality

### Phase 3: Partnerships (Month 6+)
- Coding bootcamps (integrate for student projects)
- Career coaching services
- Interview preparation platforms

---

## Success Metrics

### Quality
- User feedback on report accuracy
- Resume bullet utility (users report getting interviews)
- User satisfaction scores

### Revenue
- Free → paid conversion
- Average revenue per user
- Monthly recurring revenue

### Usage
- Analyses per day
- Active users
- Retention

---

## Risks & Mitigation

| Risk | Mitigation |
|------|-----------|
| Analysis feels generic | Get feedback early, improve prompts |
| Resume bullets don't help | Validate with users, iterate |
| GitHub repos vary widely | Design analysis to handle different tech stacks |
| Low demand | Validate demand with students first |
| GitHub API rate limits | Implement caching, smart querying |

---

## Timeline

- **Month 1-4:** Caption Generator + Aesthetic Scorer
- **Month 5:** Begin Repo Analyzer development
- **Month 6:** Beta testing
- **Month 6+:** Launch with lower priority than other products

---

## Links
- [[Caption Generator]] (priority 1)
- [[Aesthetic Scorer]] (priority 2)
- [[Content Calendar]]

---

## Progress Log

**2025-04-29:**
- Project scoped
- Monetization strategy drafted
- Timeline planned (Month 5+)
- Next: Focus on Caption Generator first

---

## Notes

**Why this as #3?**
- Lower priority than creator-focused products
- Smaller market (but still viable)
- Good "third product" to diversify income
- Tests ability to expand beyond creator niche
- Can be built alongside other projects
