# Phase 4: Preferences Collection - Roadmap

**Created:** 2026-02-12
**Phase:** 4 of 10
**Timeline:** Week 4
**Status:** Not Started
**Companion Document:** [Phase 4 Plan](./2026-02-12_phase-4_preferences-collection-plan.md)

---

## Overview

**Goal:** Create web form to capture couple preferences

**Duration:** Week 4 (3-4 hours)

---

## Task Checklist

### Preferences Service
- [ ] Create server/services/preferences.js
- [ ] Implement getPreferences() function
- [ ] Implement savePreferences() function
- [ ] Add geocodeAddress() helper (mock for MVP)
- [ ] Add timestamp to saved preferences

### HTML Form
- [ ] Create public/preferences.html
- [ ] Add interests checkboxes (7 categories)
- [ ] Add want-to-try textarea
- [ ] Add location input
- [ ] Add availability fields
- [ ] Add budget dropdown
- [ ] Add form submit JavaScript
- [ ] Add result display section

### Routes
- [ ] Create server/routes/preferences.js
- [ ] Implement GET /:coupleId (serve HTML)
- [ ] Implement GET /:coupleId/data (get JSON)
- [ ] Implement POST /:coupleId (save)
- [ ] Add validation
- [ ] Mount routes in server/index.js

### Testing
- [ ] Load form in browser
- [ ] Fill out and submit form
- [ ] Verify data saved to preferences.json
- [ ] Retrieve preferences via API
- [ ] Test validation (missing required fields)
- [ ] Test with various inputs

---

## Timeline

**Day 1 (2 hours):** Service and routes
**Day 2 (1-2 hours):** HTML form and testing

---

## Success Criteria

1. **Form accessible** at /preferences/couple-1
2. **Can submit** preferences
3. **Data saved** to preferences.json
4. **Can retrieve** saved preferences
5. **Validation** works
6. **Location** geocoded

---

## Dependencies

**Blocked By:** None (can parallelize with Phases 2-3)
**Blocks:** Phase 6 (Matching needs preferences)

---

## Next Phase

**Phase 5: Activity Database**
- [Phase 5 Plan](./2026-02-12_phase-5_activity-database-plan.md)

---

**Reference:** [Phase 4 Plan](./2026-02-12_phase-4_preferences-collection-plan.md)
