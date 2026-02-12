# Phase 7: Suggestion Creation - Roadmap

**Created:** 2026-02-12
**Phase:** 7 of 10
**Timeline:** Week 6
**Status:** Not Started
**Companion Document:** [Phase 7 Plan](./2026-02-12_phase-7_suggestion-creation-plan.md)

---

## Overview

**Goal:** Insert suggestions as calendar events in both Google Calendars

**Duration:** Week 6 (4-6 hours)

---

## Task Checklist

### Calendar Event Creation
- [ ] Add createSuggestionEvent() to calendar service
- [ ] Format event per spec (title, description, location, times)
- [ ] Set transparency to 'tentative'
- [ ] Create in Partner 1 calendar
- [ ] Create in Partner 2 calendar
- [ ] Return event IDs and links
- [ ] Handle API errors

### Event Description
- [ ] Implement formatEventDescription() function
- [ ] Include activity description
- [ ] Include match reasons
- [ ] Include details (duration, price, location)
- [ ] Include activity URL
- [ ] Add accept/decline instructions

### Suggestion Logging
- [ ] Implement logSuggestion() function
- [ ] Save to suggestions.json
- [ ] Include all metadata
- [ ] Track calendar event IDs
- [ ] Track calendar links
- [ ] Add timestamps

### Integration
- [ ] Update matching service to create events
- [ ] Call createSuggestionEvent() for top suggestions
- [ ] Log each created suggestion
- [ ] Handle creation failures gracefully
- [ ] Return created suggestions with links

### Testing
- [ ] Create suggestion via API
- [ ] Check Partner 1 calendar (web + mobile)
- [ ] Check Partner 2 calendar (web + mobile)
- [ ] Verify tentative status
- [ ] Verify description format
- [ ] Click calendar links
- [ ] Check suggestions.json updated

---

## Timeline

**Day 1 (2-3 hours):** Event creation function
**Day 2 (1-2 hours):** Logging and integration
**Day 3 (1 hour):** Testing with real calendars

---

## Success Criteria

1. **Events created** in both calendars via API
2. **Tentative status** shows correctly
3. **Description** formatted per spec
4. **Links work** (open in Google Calendar)
5. **Logged** to suggestions.json
6. **Visible** in Google Calendar web and mobile

---

## Dependencies

**Blocked By:** Phase 6 (Matching Engine)
**Blocks:** Phase 8 (Admin Dashboard needs this working)

---

## Next Phase

**Phase 8: Admin Dashboard**
- [Phase 8 Plan](./2026-02-12_phase-8_admin-dashboard-plan.md)

---

**Reference:** [Phase 7 Plan](./2026-02-12_phase-7_suggestion-creation-plan.md)
