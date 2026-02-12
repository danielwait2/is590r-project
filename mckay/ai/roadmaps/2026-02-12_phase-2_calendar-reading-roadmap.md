# Phase 2: Calendar Reading - Roadmap

**Created:** 2026-02-12
**Phase:** 2 of 10
**Timeline:** Week 3
**Status:** Not Started
**Companion Document:** [Phase 2 Plan](./2026-02-12_phase-2_calendar-reading-plan.md)

---

## Overview

**Goal:** Read calendar events from Google Calendar API for both partners

**Duration:** Week 3 (4-6 hours of work)

**Why this phase matters:** Validates OAuth works and provides data for free time detection.

---

## Task Checklist

### Calendar Service Implementation
- [ ] Create server/services/calendar.js file
- [ ] Implement getCalendarEvents() function
  - [ ] Load OAuth tokens from auth service
  - [ ] Create OAuth2 client
  - [ ] Call Calendar API with correct parameters
  - [ ] Handle API response
  - [ ] Return events array
- [ ] Implement extractBusyBlocks() function
  - [ ] Filter cancelled events
  - [ ] Extract start/end times
  - [ ] Handle all-day events (convert to 9am-5pm)
  - [ ] Handle timed events
  - [ ] Sort by start time
  - [ ] Return busy blocks array
- [ ] Implement getBusyBlocksForCouple() function
  - [ ] Call getCalendarEvents for partner1
  - [ ] Call getCalendarEvents for partner2
  - [ ] Extract busy blocks for each
  - [ ] Return combined object
- [ ] Add token refresh handling
  - [ ] Catch 401 errors
  - [ ] Call auth service refresh
  - [ ] Retry API call
  - [ ] Handle refresh failures

### Test Endpoints
- [ ] Create server/routes/calendar.js file
- [ ] Implement GET /calendar/:coupleId/events
  - [ ] Parse coupleId param
  - [ ] Parse date range query params
  - [ ] Call calendar service
  - [ ] Return JSON response
  - [ ] Add error handling
- [ ] Implement GET /calendar/:coupleId/busy
  - [ ] Parse parameters
  - [ ] Get busy blocks for couple
  - [ ] Return JSON response
- [ ] Mount calendar routes in server/index.js
- [ ] Test both endpoints in browser

### Event Parsing
- [ ] Handle standard timed events
- [ ] Handle all-day events (date vs dateTime)
- [ ] Handle recurring events (verify singleEvents works)
- [ ] Filter cancelled events
- [ ] Handle tentative events (count as busy)
- [ ] Test with various event types

### Testing & Verification
- [ ] Test reading Partner 1 calendar
  - [ ] API call succeeds
  - [ ] Events returned
  - [ ] Correct date range
- [ ] Test reading Partner 2 calendar
  - [ ] Same verification as Partner 1
- [ ] Test busy blocks extraction
  - [ ] Format correct
  - [ ] Times accurate
  - [ ] All-day events converted
- [ ] Test token refresh
  - [ ] Manually expire token
  - [ ] API call triggers refresh
  - [ ] Succeeds after refresh
- [ ] Test with real calendar data
  - [ ] Add test events to both calendars
  - [ ] Verify events appear in API response
  - [ ] Check busy blocks correct
- [ ] Test edge cases
  - [ ] Empty calendar (no events)
  - [ ] Full calendar (many events)
  - [ ] Mix of event types

### Integration
- [ ] Import calendar service in server
- [ ] Add calendar routes to server
- [ ] Test end-to-end flow
- [ ] Verify no errors in console
- [ ] Update landing page with calendar status

### Documentation
- [ ] Add comments to calendar service
- [ ] Document event format
- [ ] Document busy block format
- [ ] Add API examples to README

---

## Timeline

### Day 1 (2-3 hours)
- **Calendar Service:** Build getCalendarEvents function
  - Integrate with googleapis package
  - Test with one partner's calendar
- **Milestone:** Can fetch events from Google Calendar API

### Day 2 (1-2 hours)
- **Busy Blocks:** Build extractBusyBlocks function
  - Handle different event formats
  - Test with sample events
- **Combined Function:** Build getBusyBlocksForCouple
- **Milestone:** Can extract busy blocks for both partners

### Day 3 (1-2 hours)
- **Test Endpoints:** Build calendar routes
  - /events endpoint
  - /busy endpoint
- **Token Refresh:** Add automatic refresh logic
- **Testing:** Verify all functions work with real data
- **Milestone:** Phase 2 complete, ready for Phase 3

---

## Success Criteria

### Must Pass All:
1. **API Integration:** Can call Google Calendar API successfully
2. **Events Returned:** Get list of events for both partners
3. **Busy Blocks:** Can extract busy time blocks from events
4. **Token Refresh:** Automatic refresh on expiry works
5. **Test Endpoints:** Both /events and /busy endpoints working
6. **Event Parsing:** Handles timed, all-day, and recurring events
7. **No Errors:** Normal flow completes without errors

### Phase 2 Complete When:
- All task checklist items checked
- All success criteria pass
- Can view calendar events in browser via test endpoint
- Busy blocks look correct
- Ready to calculate free time (Phase 3)

---

## Dependencies

**Blocked By:**
- Phase 1: OAuth Integration (need valid tokens)

**Requires:**
- OAuth tokens in users.json
- Test calendars with events added
- googleapis npm package installed
- Internet connection for API calls

**Blocks:**
- Phase 3: Free Time Detection (needs busy blocks)
- Phase 7: Suggestion Creation (needs calendar writing)

---

## Risks & Mitigation

### Risk: Token refresh doesn't work
**Impact:** API calls fail after token expires
**Probability:** Medium
**Mitigation:**
- Test refresh logic early
- Follow googleapis examples
- Log refresh attempts
**Fallback:** Reauthorize user if refresh fails

### Risk: Event parsing misses formats
**Impact:** Some events not counted as busy
**Probability:** Low
**Mitigation:**
- Test with varied event types
- Handle all-day and timed events
- Check Google API docs for formats
**Fallback:** Add parsing as issues found

### Risk: API rate limits
**Impact:** Calls fail during testing
**Probability:** Very Low (10 QPS is plenty)
**Mitigation:**
- Don't poll API rapidly
- Cache results if needed
**Fallback:** Add small delays between calls

### Risk: Timezone issues
**Impact:** Events show at wrong times
**Probability:** Low
**Mitigation:**
- Keep all times in ISO 8601
- Don't convert timezones yet
- Test with events in different zones
**Fallback:** Document timezone assumptions

---

## Blocked By

**Phase 1 must be complete:**
- OAuth tokens available
- Both partners authenticated
- Tokens valid and refreshable

---

## Blocks

**These phases need Phase 2 complete:**
- Phase 3: Free Time Detection (needs busy blocks)
- Phase 7: Suggestion Creation (needs Calendar API working)

---

## Milestone: Calendar Reading Complete

**Definition of Done:**
- [ ] All tasks completed
- [ ] All success criteria met
- [ ] Can read events from both calendars
- [ ] Busy blocks extracted correctly
- [ ] Token refresh working
- [ ] Test endpoints functional
- [ ] Tested with real calendar data
- [ ] Ready for Phase 3 (Free Time Detection)

**Celebration checkpoint:** You can now read calendars! The data pipeline is working.

---

## Next Phase

**Phase 3: Free Time Detection**
- [Phase 3 Plan](./2026-02-12_phase-3_free-time-detection-plan.md)
- [Phase 3 Roadmap](./2026-02-12_phase-3_free-time-detection-roadmap.md)
- Timeline: Week 3-4
- Use busy blocks to find overlapping free time

---

**Reference Documents:**
- [High-Level Implementation Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture Documentation](../../aiDocs/architecture.md)
- [MVP Specification](../../aiDocs/mvp.md)
- [Phase 2 Plan](./2026-02-12_phase-2_calendar-reading-plan.md)
- [Google Calendar API](https://developers.google.com/workspace/calendar/api/v3/reference/events/list)
