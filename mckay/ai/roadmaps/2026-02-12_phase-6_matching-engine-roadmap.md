# Phase 6: Matching Engine - Roadmap

**Created:** 2026-02-12
**Phase:** 6 of 10
**Timeline:** Week 5-6
**Status:** Not Started
**Companion Document:** [Phase 6 Plan](./2026-02-12_phase-6_matching-engine-plan.md)

---

## Overview

**Goal:** Core logic to match activities to free time

**Duration:** Week 5-6 (6-8 hours)

---

## Task Checklist

### Matching Service
- [ ] Create server/services/matching.js
- [ ] Implement generateSuggestions() main function
- [ ] Implement calculateActivityScore() function
- [ ] Implement generateMatchReasons() function
- [ ] Implement matchActivitiesToTimeWindows() function
- [ ] Add orchestration logic (load prefs, find free time, search activities, score, match)

### Scoring Algorithm
- [ ] Category match: +10 points
- [ ] Keyword match: +5 per match
- [ ] Location proximity: +1-3 based on distance
- [ ] Price match: +2 if within budget
- [ ] Test scoring with sample data

### Match Reasons
- [ ] Generate human-readable reasons
- [ ] Include category matches
- [ ] Include want-to-try matches
- [ ] Include distance info
- [ ] Format as readable string

### Routes
- [ ] Create server/routes/matching.js
- [ ] Implement POST /:coupleId
- [ ] Call generateSuggestions()
- [ ] Return JSON result
- [ ] Mount routes in server/index.js

### Testing
- [ ] Test with preferences + free time + activities
- [ ] Verify scoring algorithm
- [ ] Check match reasons make sense
- [ ] Test edge cases (no free time, no matches)
- [ ] Verify top suggestions returned

---

## Timeline

**Week 5, Day 3-4 (3-4 hours):** Core matching logic
**Week 6, Day 1 (2-3 hours):** Scoring and integration
**Week 6, Day 2 (1-2 hours):** Testing and refinement

---

## Success Criteria

1. **generateSuggestions()** works end-to-end
2. **Scoring** produces relevant rankings
3. **Match reasons** are clear and accurate
4. **Returns** top 1-2 suggestions
5. **Handles** edge cases gracefully
6. **Ready** for Phase 7 (suggestion creation)

---

## Dependencies

**Blocked By:**
- Phase 3: Free Time Detection
- Phase 4: Preferences
- Phase 5: Activity Database

**Blocks:** Phase 7 (Suggestion Creation)

---

## Next Phase

**Phase 7: Suggestion Creation**
- [Phase 7 Plan](./2026-02-12_phase-7_suggestion-creation-plan.md)

---

**Reference:** [Phase 6 Plan](./2026-02-12_phase-6_matching-engine-plan.md)
