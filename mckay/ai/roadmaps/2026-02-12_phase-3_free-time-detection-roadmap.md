# Phase 3: Free Time Detection - Roadmap

**Created:** 2026-02-12
**Phase:** 3 of 10
**Timeline:** Week 3-4
**Status:** Not Started
**Companion Document:** [Phase 3 Plan](./2026-02-12_phase-3_free-time-detection-plan.md)

---

## Overview

**Goal:** Calculate overlapping free time between both partners' calendars

**Duration:** Week 3-4 (4-6 hours)

---

## Task Checklist

### Core Algorithm
- [ ] Implement findFreeWindows() function
- [ ] Implement findOverlappingFreeTime() function
- [ ] Implement getAvailableTimeForCouple() main function
- [ ] Add weekend filtering logic
- [ ] Add duration filtering (min 2 hours)
- [ ] Add advance notice filtering (3+ days)

### Filters
- [ ] Build isWeekendTime() helper
- [ ] Build filterWeekendWindows() function
- [ ] Build filterByDuration() function
- [ ] Build filterByAdvanceNotice() function
- [ ] Define DEFAULT_PREFERENCES constant

### Test Endpoint
- [ ] Add GET /calendar/:coupleId/free route
- [ ] Load preferences or use defaults
- [ ] Call getAvailableTimeForCouple()
- [ ] Return JSON with free windows

### Testing
- [ ] Test with both calendars having events
- [ ] Test with one empty calendar
- [ ] Test with no overlapping free time
- [ ] Test weekend filter (Fri 6pm-Sun 11pm)
- [ ] Test duration filter (2hr minimum)
- [ ] Test advance notice (3 days)
- [ ] Test via endpoint in browser

---

## Timeline

**Day 1 (2-3 hours):** Core algorithm
**Day 2 (1-2 hours):** Filters and integration
**Day 3 (1 hour):** Testing and refinement

---

## Success Criteria

1. **Free windows found** from busy blocks
2. **Overlapping logic** works correctly
3. **Weekend filter** applies (Fri 6pm-Sun 11pm)
4. **Duration filter** works (2hr min)
5. **Test endpoint** returns valid JSON
6. **Edge cases** handled (no free time = empty array)

---

## Dependencies

**Blocked By:** Phase 2 (needs busy blocks)
**Blocks:** Phase 6 (Matching Engine needs free time)

---

## Risks & Mitigation

**Risk:** Complex time overlap logic
**Mitigation:** Start simple, test incrementally

**Risk:** Timezone edge cases
**Mitigation:** Keep ISO 8601 format throughout

---

## Next Phase

**Phase 4: Preferences Collection**
- [Phase 4 Plan](./2026-02-12_phase-4_preferences-collection-plan.md)
- Timeline: Week 4

---

**Reference:** [Phase 3 Plan](./2026-02-12_phase-3_free-time-detection-plan.md)
