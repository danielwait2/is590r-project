# Project Context

## Quick Summary

**What:** Family Quality Time App - Calendar-native activity recommendation engine
**How:** Monitors Google Calendar(s) → finds free time → suggests activities → creates calendar events
**Goal:** Combine best ideas from three research approaches into working MVP

## Research Sources

This project synthesizes research from three team members:

### Daniel's Approach (Scalable Vision)
- **Location:** [daniel/](../daniel/)
- **Strength:** Long-term product strategy, comprehensive library research
- **Key Docs:**
  - PRD: [daniel/aiDocs/prd.md](../daniel/aiDocs/prd.md)
  - MVP: [daniel/aiDocs/mvp.md](../daniel/aiDocs/mvp.md)
  - Architecture: [daniel/aiDocs/architecture.md](../daniel/aiDocs/architecture.md)
  - Roadmaps: [daniel/ai/roadmaps/](../daniel/ai/roadmaps/)
- **Highlights:** Vision API for screenshots, React Email templates, comprehensive monetization

### McKay's Approach (Pragmatic Minimalist)
- **Location:** [mckay/](../mckay/)
- **Strength:** Clean, focused implementation for demo success
- **Key Docs:**
  - PRD: [mckay/aiDocs/prd.md](../mckay/aiDocs/prd.md)
  - MVP: [mckay/aiDocs/mvp.md](../mckay/aiDocs/mvp.md)
  - Architecture: [mckay/aiDocs/architecture.md](../mckay/aiDocs/architecture.md)
  - Implementation Plan: [mckay/ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md](../mckay/ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md)
- **Highlights:** Hardcoded activities, transparent scoring algorithm, explicit scope cuts

### Dylan's Approach (Feature-Driven)
- **Location:** [dylan/](../dylan/)
- **Strength:** Unique features with practical implementation
- **Key Docs:**
  - PRD: [dylan/aiDocs/prd.md](../dylan/aiDocs/prd.md)
  - MVP: [dylan/aiDocs/mvp.md](../dylan/aiDocs/mvp.md)
  - Implementation Plan: [dylan/ai/roadmaps/2026-02-12_implementation-plan.md](../dylan/ai/roadmaps/2026-02-12_implementation-plan.md)
- **Highlights:** Babysitter SMS automation (Twilio), web scraping, Supabase auth

## Tech Stack (To Be Finalized)

**Frontend:**
- TBD (Options: Next.js 14, Chrome Extension, Simple HTML/CSS/JS)

**Backend:**
- TBD (Options: Next.js API Routes, Node.js + Express, Serverless)

**Database:**
- TBD (Options: Supabase, PostgreSQL, JSON files for MVP)

**Auth:**
- Google Calendar OAuth (confirmed)
- TBD for user auth (Options: Supabase Auth, Clerk, none for MVP)

**APIs & Services:**
- ✅ Google Calendar API (all three agree)
- TBD: Twilio SMS (Dylan's approach)
- TBD: Resend email (Daniel's approach)
- TBD: Activities source (hardcoded vs. scraped vs. API)

**Deployment:**
- TBD (Options: Vercel, local dev, Railway)

## Current Focus

**Phase:** Planning & Requirements Synthesis
**Status:** Combining research from three approaches
**Next:** Define unified MVP scope and tech stack

**Current Tasks:**
- [ ] Review all three research approaches
- [ ] Identify best ideas from each
- [ ] Define unified MVP scope
- [ ] Choose tech stack
- [ ] Create implementation roadmap
- [ ] Set up development environment

**Key Decisions Needed:**
1. MVP approach: Chrome extension vs. web app vs. backend service?
2. Timeline: 4-5 days, 6-8 weeks, or 7 weeks?
3. Feature scope: Include babysitter SMS? Vision API? Email automation?
4. Activities source: Hardcoded, scraped, or API?
5. Database: JSON files, Supabase, or PostgreSQL?
6. Target audience: Couples only or families with kids?

## Best Ideas to Consider

From the comparison analysis:

**From Daniel:**
- Vision API for calendar screenshots (privacy-friendly alternative)
- React Email templates for beautiful emails
- BullMQ + Redis for background jobs
- Comprehensive monetization strategy (Stripe, tiers, B2B)

**From McKay:**
- Hardcoded activities (reliable, no API dependencies)
- Transparent scoring algorithm (+10 category, +5 keyword, +3 location, +2 price)
- Explicit "Out of Scope" documentation
- Admin dashboard for manual demo triggering

**From Dylan:**
- Babysitter SMS automation (unique differentiator)
- Web scraping for activities (100-150 in target city)
- Supabase for simple auth + database
- Single-page onboarding

## Project Management

**Changelog:** [ai/changelog.md](../ai/changelog.md) - Track all changes
**Roadmaps:** [ai/roadmaps/](../ai/roadmaps/) - Implementation plans
**Guides:** [ai/guides/](../ai/guides/) - API docs and technical guides

## Behavior Guidelines

See [../CLAUDE.md](../CLAUDE.md) for:
- Ask before complex work
- Keep it simple (no over-engineering)
- Flag uncertainty instead of guessing
- Clean code philosophy
