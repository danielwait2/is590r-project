# Phase 4 Roadmap: Free Time Detection & Matching

**Status:** Not Started  
**Timeline:** Week 5 (7 days)  
**Detailed Plan:** [2026-02-12_phase4-matching.md](./2026-02-12_phase4-matching.md)

---

## ⚠️ Development Philosophy

**Avoid over-engineering, cruft, and legacy-compatibility features in this clean code project.**

- Build only what's needed for this phase
- No "just in case" features
- Delete unused code immediately
- Keep it simple and focused

---

## Quick Overview

Implement free time detection, build activity matching algorithm, generate suggestions, and insert them into Google Calendar. Core value proposition of the app.

---

## Task Checklist

### Free Time Detection (Day 1-2)
- [ ] Create `lib/calendar/detect-free-time.ts`
- [ ] Implement gap detection (3+ hours between events)
- [ ] Handle empty calendar case
- [ ] Handle all-day events
- [ ] Test with various calendar patterns
- [ ] Create test script for algorithm

### Activity Matching (Day 2-3)
- [ ] Create `lib/matching/match-activities.ts`
- [ ] Implement keyword matching (Want to Try → activities)
- [ ] Score matches by relevance
- [ ] Sort by score and rating
- [ ] Test with different Want to Try items
- [ ] Handle no matches case

### Suggestion Generation (Day 3-4)
- [ ] Create `lib/suggestions/generate.ts`
- [ ] Combine free time + matched activities
- [ ] Check activity fits in free block
- [ ] Prevent duplicate suggestions (same activity in 7 days)
- [ ] Create `/api/suggestions/generate` endpoint

### Calendar Event Insertion (Day 4-5)
- [ ] Create `lib/google-calendar/insert-event.ts`
- [ ] Format event with rich description
- [ ] Set event color (blue for suggestions)
- [ ] Add suggestion metadata (extended properties)
- [ ] Insert event via Calendar API
- [ ] Save suggestion to database

### Dashboard UI (Day 5-6)
- [ ] Create suggestions display component
- [ ] Add "Generate Suggestions" button
- [ ] Show pending suggestions
- [ ] Display match reasons
- [ ] Add loading state during generation

### End-to-End Testing (Day 6-7)
- [ ] Test full flow: Want to Try → free time → suggestion → calendar
- [ ] Verify events appear in Google Calendar
- [ ] Test with multiple Want to Try items
- [ ] Test with various calendar patterns
- [ ] Fix any bugs discovered
- [ ] Verify database records created

---

## Success Criteria

- [ ] App detects free time from calendar (3+ hour gaps)
- [ ] App matches Want to Try list with activities
- [ ] Suggestions appear in Google Calendar with full details
- [ ] Suggestions display in dashboard
- [ ] User can generate new suggestions
- [ ] Match reasons are clear and specific
- [ ] No duplicate suggestions in short timeframe

---

## Key Deliverables

- [ ] Free time detection algorithm
- [ ] Activity matching engine
- [ ] Suggestion generator
- [ ] Calendar event insertion
- [ ] Suggestions display UI
- [ ] Generate suggestions API endpoint

---

## Notes & Decisions

<!-- Track decisions during implementation -->

---

## Completion Checklist

Before moving to Phase 5:
- [ ] All tasks completed
- [ ] Success criteria met
- [ ] Suggestion appears in Google Calendar
- [ ] Dashboard shows suggestions
- [ ] Matching algorithm tested
- [ ] No critical bugs
- [ ] Ready for Phase 5 (Babysitter Feature)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026  
**Previous Phase:** [Phase 3: Activity Discovery](./2026-02-12_roadmap-phase3.md)  
**Next Phase:** [Phase 5: Babysitter Feature](./2026-02-12_roadmap-phase5.md)
