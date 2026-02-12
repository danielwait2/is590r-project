# Project Context

## Quick Summary

**What:** Calendar-native weekend experience concierge for busy couples
**How:** Monitors both partners' Google Calendars → finds overlapping free time → matches activities to interests → inserts suggestions directly into calendars
**MVP:** 6-8 weeks, Node.js + Express, 1 test couple, JSON files, manual triggering
**Goal:** Demo-ready class project proving the concept works

**Implementation Status:** Planning complete, ready to start Phase 0 (Foundation & Setup)

---

## Core Documents

### Product & Planning

- **PRD:** [prd.md](./prd.md) - Product requirements & market positioning
- **MVP:** [mvp.md](./mvp.md) - Demo-focused technical spec (6-8 weeks, 1 couple)
- **Architecture:** [architecture.md](./architecture.md) - **AUTHORITATIVE** technical specification

### Implementation

- **Implementation Roadmap:** [ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md](../ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md) - 10-phase plan
- **API Strategy:** [ai/guides/mvp-api-strategy.md](../ai/guides/mvp-api-strategy.md) - What APIs we're using
- **Google Calendar API:** [ai/guides/google-calendar-api.md](../ai/guides/google-calendar-api.md) - Verified API docs
- **Phase Roadmaps:** [ai/roadmaps/](../ai/roadmaps/) - Detailed phase-by-phase plans

### Project Management

- **[Changelog]** (../ai/changelog.md) - Brief notes about each change to the codebase

## Product Summary

Calendar-native weekend experience concierge for busy couples. Monitors both partners' Google Calendars, finds overlapping free time, matches activities to their interests, inserts suggestions directly into their calendars.

**Target:** Experience-seeking couples in one metro area
**Goal:** Turn "things we keep saying we should do" into actual calendar events

## Behavior

- Whenever creating plan docs and roadmap docs, save them in ai/roadmaps. Prefix the name with the date. Add a note that we need to avoid over-engineering, cruft, and legacy-compatibility features in this clean code project. Make sure they reference each other.
- Whenever finishing with implementing a plan / roadmap doc pair, make sure the roadmap isup to date (tasks checked off, etc.) Then move the ocs to ai/roadmpas/complete. Then update ai/changelog.md accordingly.

## Tech Stack Summary

- **Backend:** Node.js + Express, Google Calendar API
- **Data:** JSON files (MVP), hardcoded activities
- **Frontend:** Simple HTML/CSS/JS, admin dashboard
- **Details:** See [architecture.md](./architecture.md)

## Current Focus

**Phase:** Phase 0 - Foundation & Setup
**Timeline:** 6-8 weeks to demo-ready MVP
**Approach:** Automated MVP (Node.js + Express backend)
**Scope:** 1 test couple (couple-1) for demo

**Planning Status:**

- [x] Market research completed
- [x] PRD written
- [x] MVP spec defined
- [x] Architecture documented
- [x] APIs verified (Google Calendar ✅, Eventbrite ❌ deprecated)
- [x] 10-phase implementation roadmap created
- [ ] Development environment setup
- [ ] Google Cloud project + OAuth credentials

**Next Steps:**

1. Follow 10-phase roadmap: [ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md](../ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md)
2. Set up Google Cloud project and OAuth credentials (Phase 0)
3. Build per [architecture.md](./architecture.md) specification
4. Use verified APIs per [ai/guides/](../ai/guides/)

## Key Decisions

**2026-02-12:**

- **Target:** Couples (not families with kids) - simpler for MVP
- **MVP Approach:** Automated Node.js backend (not manual pilot, not Chrome extension)
- **Scope:** 1 test couple for demo (not multiple couples)
- **Timeline:** 6-8 weeks to demo-ready MVP
- **Tech Stack:** Node.js + Express + JSON files (no PostgreSQL database for MVP)
- **APIs:** Google Calendar API only (Eventbrite deprecated)
- **Activities:** Hardcoded 20-30 activities in JSON file
- **Triggering:** Manual via admin dashboard (no automated scheduling)
- **Deployment:** Local development for demo (no cloud hosting required)
- **Weekend-only scope:** Avoids "calendar lies" problem (free time ≠ actually free)

## MVP Scope (6-8 weeks)

**In Scope:**

- OAuth 2.0 with Google Calendar API
- Calendar integration (read events, create suggestions)
- Free time detection (overlapping windows for both partners)
- Basic matching algorithm (keyword-based scoring)
- Hardcoded activity database (20-30 activities in JSON)
- Admin dashboard (manual trigger for demo)
- **1 test couple only** (couple-1)
- **JSON file storage** (no database)

**Out of Scope (V2/Future):**

- User accounts/authentication system
- Automated scheduling (cron jobs)
- Email/SMS notifications
- Learning algorithms
- Multiple couples support
- PostgreSQL database
- Mobile app

**Details:** See [prd.md](./prd.md), [mvp.md](./mvp.md), and [architecture.md](./architecture.md)
