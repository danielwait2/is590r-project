# Project Context

## Project Overview
**Family Quality Time App** - Calendar-native activity recommendation engine that solves the "Desire-to-Action Gap" for busy families.

## Key Documents
- **PRD:** [aiDocs/prd.md](prd.md) - Full product requirements and strategy
- **MVP:** [aiDocs/mvp.md](mvp.md) - Chrome extension demo spec
- **Architecture:** [aiDocs/architecture.md](architecture.md) - Technical architecture (MVP, V1, V2+)
- **Market Research:** [ai/guides/quality-time-app-market-research.md](../ai/guides/quality-time-app-market-research.md)
- **Changelog:** [ai/changelog.md](../ai/changelog.md) - Brief notes about each change to the codebase

## Current Focus

**Phase:** Phase 1 - MVP Chrome Extension Demo (Ready to Start)
**Timeline:** 4-5 days
**Goal:** Functional demo showing calendar integration + suggestions + event creation

**Immediate Next Steps:**
- [ ] Set up Google Cloud Project for Calendar API
- [ ] Build Chrome extension boilerplate (manifest.json, popup UI)
- [ ] Implement OAuth flow
- [ ] Create curated activities database (10-15 items)
- [ ] Build free time detection logic
- [ ] Implement "Add to Calendar" functionality

**Success Criteria:**
- Real Google Calendar connection (OAuth works)
- Detects actual free time from user's calendar
- Shows 2 curated suggestions
- Successfully adds event back to Google Calendar
- Demo completes in <60 seconds

## Project Instructions

For detailed project instructions and behavior guidelines, see [../CLAUDE.md](../CLAUDE.md). 
