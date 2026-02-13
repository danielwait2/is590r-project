# Phase 1 Roadmap: MVP Chrome Extension

**Status:** Ready to Start
**Timeline:** 4-5 days
**Detailed Plan:** [2026-02-11-phase1-mvp-chrome-extension.md](./2026-02-11-phase1-mvp-chrome-extension.md)

---

## ⚠️ Development Philosophy

**Avoid over-engineering, cruft, and legacy-compatibility features in this clean code project.**

- Build only what's needed for this phase
- No "just in case" features
- Delete unused code immediately
- Keep it simple and focused

---

## Quick Overview

Build a functional Chrome extension that validates the core value proposition: "Free time + desires → specific plan in 10 seconds." Client-side only demo to test user reactions before building the full production web app.

---

## Task Checklist

### 1A: Setup & Calendar Access (Day 1)
- [ ] Google Cloud Project setup
- [ ] OAuth 2.0 credentials created
- [ ] Extension boilerplate files created
- [ ] Icons generated
- [ ] OAuth flow tested and working

### 1B: Free Time Detection (Day 2)
- [ ] Free time detection algorithm implemented
- [ ] UI displays next free time block
- [ ] Edge cases tested (empty calendar, fully booked, all-day events)

### 1C: Suggestions Engine (Day 3)
- [ ] Curated activities database created (10+ activities)
- [ ] Keyword matching algorithm implemented
- [ ] Suggestions display in UI
- [ ] "Add to Calendar" buttons functional

### 1D: Calendar Integration (Day 4)
- [ ] Event creation working
- [ ] Events appear in Google Calendar
- [ ] Loading states implemented
- [ ] Error handling comprehensive

### 1E: Testing & Demo Prep (Day 5)
- [ ] End-to-end testing complete
- [ ] Demo video recorded (<2 min)
- [ ] README.md created
- [ ] 5+ user testing sessions conducted
- [ ] User feedback collected

---

## Success Criteria

- [ ] Real Google Calendar OAuth integration works
- [ ] Correctly detects free time from actual calendar
- [ ] Shows 2 relevant, curated suggestions
- [ ] Successfully creates event in Google Calendar
- [ ] Demo completes in <60 seconds
- [ ] Gets "wow, that's useful!" reaction from 5+ testers

---

## Key Deliverables

- [ ] Working Chrome extension (unpacked .zip)
- [ ] manifest.json with OAuth configuration
- [ ] Curated activities database (10+ items)
- [ ] Demo video (<2 minutes)
- [ ] User feedback document from 5+ testers
- [ ] README.md with installation instructions

---

## Notes & Decisions

(Track key decisions and learnings during implementation)

---

## Completion Checklist

Before moving to Phase 2:
- [ ] All tasks completed
- [ ] Success criteria met
- [ ] Deliverables created
- [ ] Demo video shared with 5+ people
- [ ] User feedback analyzed
- [ ] Decision made: proceed to Phase 2 or pivot

---

**Created:** 2026-02-11
**Last Updated:** 2026-02-11
**Next Phase:** [2026-02-11-roadmap-phase2-v1-core.md](./2026-02-11-roadmap-phase2-v1-core.md)
