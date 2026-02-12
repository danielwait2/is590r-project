# Project Context: Family Quality Time Activity Facilitator

## Overview
Calendar-integrated app that bridges the intention-action gap for couples and families by automatically detecting free time, suggesting personalized activities, and coordinating babysitters via SMS.

## Key Documents
- **PRD:** `aiDocs/prd.md` - Full product requirements and vision
- **MVP:** `aiDocs/mvp.md` - Demo-ready minimum viable product (6-8 weeks)
- **Implementation Plan:** `ai/roadmaps/2026-02-12_implementation-plan.md` - 6 phases, week-by-week roadmap
- **Market Research:** `ai/guides/Family_quality_time_market_research.md`

## Tech Stack

### Frontend
- **Next.js** (React framework)
- **Tailwind CSS** (styling)
- **Vercel** (hosting)

### Backend
- **Next.js API Routes** (serverless functions)
- **Supabase** (PostgreSQL + auth)

### External Services
- **Google Calendar API** (calendar integration)
- **Twilio** (SMS for babysitter coordination)
- **Cheerio/Puppeteer** (web scraping for activities)

### Development
- **TypeScript** (type safety)
- **Git** (version control)

## Current Focus

**Phase:** MVP Planning & Setup

**Immediate Next Steps:**
- Set up Next.js project with TypeScript
- Configure Supabase (database + auth)
- Implement Google Calendar OAuth
- Build basic onboarding flow

**This Week's Goal:**
- Complete development environment setup
- User can sign up and connect Google Calendar

**Blockers/Questions:**
- None currently

## Core Value Proposition
> "Tell us what you want to do → We find free time → We suggest activities → We coordinate your babysitter"

**Demo target:** 5-minute flow from signup to confirmed activity with babysitter booked

## Success Metrics (MVP)
- 80%+ complete onboarding
- Suggestions feel relevant (match Want to Try list)
- 70%+ babysitter SMS acceptance rate
- At least 1 test couple actually does suggested activity
- Zero crashes during demo

## Target User (MVP)
- Austin, TX area initially
- Couples with kids ages 2-10
- Use Google Calendar
- Have 1+ trusted babysitters

## Constraints/Decisions
- **Solo developer** → Simple stack, avoid over-engineering
- **6-8 week timeline** → Cut scope aggressively
- **Demo-first** → Prove concept before scaling
- **Web scraping** → Populate activities faster than manual curation
- **One city (Austin)** → 200-300 activities, quality over coverage
- **Free hosting** → Vercel + Supabase free tiers (~$1-2/month)
