# Phase 5: Activity Database - Roadmap

**Created:** 2026-02-12
**Phase:** 5 of 10
**Timeline:** Week 5
**Status:** Not Started
**Companion Document:** [Phase 5 Plan](./2026-02-12_phase-5_activity-database-plan.md)

---

## Overview

**Goal:** Create and populate curated activity database

**Duration:** Week 5 (4-6 hours, mostly research/data entry)

---

## Task Checklist

### Activity Data Collection (Manual Work)
- [ ] Research 5-6 dining activities in target city
- [ ] Research 4-5 arts/culture activities
- [ ] Research 3-4 music venues
- [ ] Research 4-5 classes/workshops
- [ ] Research 4-5 outdoor activities
- [ ] Research 2-3 sports activities
- [ ] Research 2-3 entertainment venues
- [ ] Total: 20-30 activities

### Data Entry
- [ ] Create activity JSON objects
- [ ] Add name, description for each
- [ ] Add location (name, address, lat/lng)
- [ ] Add duration estimates
- [ ] Add price levels and amounts
- [ ] Add relevant keywords
- [ ] Add URL links
- [ ] Add availability info
- [ ] Verify all required fields present

### Activity Service
- [ ] Create server/services/activities.js
- [ ] Implement loadActivities() function
- [ ] Implement searchActivities() function
- [ ] Implement calculateDistance() (Haversine)
- [ ] Add comparePriceLevels() helper
- [ ] Add filtering logic (category, keywords, location, budget)

### Testing
- [ ] Load activities in code
- [ ] Test search by category
- [ ] Test search by keywords
- [ ] Test distance calculation
- [ ] Test budget filtering
- [ ] Verify all activities valid

---

## Timeline

**Day 1-2 (3-4 hours):** Research and data collection
**Day 3 (1-2 hours):** Service implementation and testing

---

## Success Criteria

1. **20-30 activities** in activities.json
2. **All categories** covered
3. **Quality data** (real venues, accurate info)
4. **Service functions** work correctly
5. **Search/filter** logic tested
6. **Distance calc** accurate

---

## Dependencies

**Blocked By:** None (can parallelize)
**Blocks:** Phase 6 (Matching needs activities)

---

## Next Phase

**Phase 6: Matching Engine**
- [Phase 6 Plan](./2026-02-12_phase-6_matching-engine-plan.md)

---

**Reference:** [Phase 5 Plan](./2026-02-12_phase-5_activity-database-plan.md)
