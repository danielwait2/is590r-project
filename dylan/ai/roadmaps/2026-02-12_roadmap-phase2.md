# Phase 2 Roadmap: Calendar Integration

**Status:** Not Started  
**Timeline:** Week 3 (7 days)  
**Detailed Plan:** [2026-02-12_phase2-calendar.md](./2026-02-12_phase2-calendar.md)

---

## ⚠️ Development Philosophy

**Avoid over-engineering, cruft, and legacy-compatibility features in this clean code project.**

- Build only what's needed for this phase
- No "just in case" features
- Delete unused code immediately
- Keep it simple and focused

---

## Quick Overview

Integrate Google Calendar API to read user's calendar events via OAuth. Display upcoming events in dashboard. Enables free time detection in Phase 4.

---

## Task Checklist

### Google Calendar API Setup (Day 1)
- [ ] Enable Google Calendar API in Google Cloud Console
- [ ] Update OAuth consent screen with calendar scope
- [ ] Verify OAuth redirect URIs
- [ ] Install `googleapis` npm package

### Update Auth Flow (Day 1-2)
- [ ] Add calendar scope to sign-in OAuth request
- [ ] Update auth callback to store calendar tokens
- [ ] Save tokens to Supabase users table
- [ ] Test OAuth flow with calendar permissions

### Calendar Sync Function (Day 2-3)
- [ ] Create `lib/google-calendar/client.ts` helper
- [ ] Implement `fetchCalendarEvents()` function
- [ ] Create `/api/calendar/sync` API route
- [ ] Handle token expiration errors
- [ ] Test fetching events from Google Calendar

### Dashboard UI Updates (Day 3-4)
- [ ] Add "Connect Calendar" button to dashboard
- [ ] Create `/api/calendar/connect` route
- [ ] Show connection status (connected/not connected)
- [ ] Add privacy notice about calendar data

### Display Calendar Events (Day 4-5)
- [ ] Create `calendar-events.tsx` client component
- [ ] Display events with date, time, and title
- [ ] Add manual "Refresh" button
- [ ] Show loading state during sync
- [ ] Show error state if sync fails
- [ ] Handle empty state (no events)

### Testing & Polish (Day 5-7)
- [ ] Test calendar connection flow end-to-end
- [ ] Test with various event types (all-day, timed, recurring)
- [ ] Test refresh button
- [ ] Test error handling (expired token, network error)
- [ ] Verify tokens saved correctly in database
- [ ] Check for console errors
- [ ] Test on production deployment

---

## Success Criteria

- [ ] User can click "Connect Calendar" and complete OAuth
- [ ] App fetches next 2 weeks of calendar events
- [ ] Dashboard displays events with dates and times
- [ ] User can manually refresh calendar data
- [ ] Tokens persist across sessions
- [ ] Error handling for expired/revoked access
- [ ] Privacy notice displayed
- [ ] No console errors

---

## Key Deliverables

- [ ] Calendar OAuth flow integrated
- [ ] Calendar sync API endpoint (`/api/calendar/sync`)
- [ ] Calendar events display component
- [ ] Manual refresh functionality
- [ ] Token storage in Supabase
- [ ] Error handling for common issues

---

## Notes & Decisions

<!-- Track decisions during implementation -->

---

## Completion Checklist

Before moving to Phase 3:
- [ ] All tasks completed
- [ ] Success criteria met
- [ ] Deliverables created
- [ ] Calendar connection tested
- [ ] Events display correctly
- [ ] Error handling works
- [ ] No critical bugs
- [ ] Ready for Phase 3 (Activity Discovery)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026  
**Previous Phase:** [Phase 1: Foundation](./2026-02-12_roadmap-phase1.md)  
**Next Phase:** [Phase 3: Activity Discovery](./2026-02-12_roadmap-phase3.md)
