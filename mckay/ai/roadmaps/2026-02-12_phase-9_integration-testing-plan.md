# Phase 9: Integration & Testing - Implementation Plan

**Created:** 2026-02-12
**Phase:** 9 of 10
**Timeline:** Week 7
**Companion Document:** [Phase 9 Roadmap](./2026-02-12_phase-9_integration-testing-roadmap.md)

---

## Project Principles

⚠️ **Test end-to-end flow rigorously.** Find and fix bugs before demo day. Focus on happy path + common errors.

---

## Phase Overview

**Goal:** End-to-end testing and bug fixes

**Key deliverables:**
- Complete OAuth → Preferences → Match → Suggestions flow works
- Bug fixes for issues found
- Error handling for critical paths
- Testing with real Google Calendar accounts
- Documentation of known issues

---

## Testing Strategy

### 1. End-to-End Flow Testing

**Full Happy Path:**

1. **OAuth Setup**
   - [ ] Partner 1 authenticates
   - [ ] Partner 2 authenticates
   - [ ] Both tokens stored correctly
   - [ ] Status shows both connected

2. **Preferences Setup**
   - [ ] Load preferences form
   - [ ] Fill out all fields
   - [ ] Submit form
   - [ ] Preferences saved to JSON
   - [ ] Can retrieve preferences

3. **Calendar State**
   - [ ] Add test events to Partner 1 calendar
   - [ ] Add test events to Partner 2 calendar
   - [ ] Mix of busy times and free windows
   - [ ] Include all-day events
   - [ ] Include recurring events

4. **Matching & Suggestions**
   - [ ] Go to admin dashboard
   - [ ] Click "Generate Suggestions"
   - [ ] Wait for results
   - [ ] Verify free windows found
   - [ ] Verify activities matched
   - [ ] Verify suggestions created
   - [ ] Check suggestions appear in both calendars
   - [ ] Check suggestions show as tentative
   - [ ] Click calendar links (open correctly)

5. **Verification**
   - [ ] Open Partner 1 Google Calendar (web)
   - [ ] Verify suggestion event visible
   - [ ] Check event details (description, location, time)
   - [ ] Open Partner 2 Google Calendar (web)
   - [ ] Verify same suggestion event visible
   - [ ] Test on mobile (optional but recommended)

---

### 2. Error Handling Testing

**OAuth Errors:**
- [ ] User denies consent → show error message
- [ ] Token expires → refresh automatically
- [ ] Refresh fails → show reauthorize message
- [ ] Only one partner authenticated → disable matching

**Calendar Errors:**
- [ ] API returns 401 → handle refresh
- [ ] API returns 500 → show error message
- [ ] Network timeout → show error message
- [ ] Empty calendar → handle gracefully

**Matching Errors:**
- [ ] No preferences set → show error message
- [ ] No free time found → show friendly message
- [ ] No matching activities → show friendly message
- [ ] Event creation fails → log error, continue

---

### 3. Edge Cases

**Calendar Scenarios:**
- [ ] Both calendars completely empty
- [ ] Both calendars completely full (no free time)
- [ ] One empty, one full calendar
- [ ] Free time exists but all on weekdays (not weekends)
- [ ] Free windows too short (< 2 hours)
- [ ] Free time exists but all within 3-day window

**Preferences Scenarios:**
- [ ] No interests selected
- [ ] Empty want-to-try list
- [ ] Very specific interests (limited matches)
- [ ] Very broad interests (many matches)

**Activity Scenarios:**
- [ ] No activities match interests
- [ ] All activities too far away
- [ ] All activities too expensive
- [ ] Activity duration doesn't fit any free window

---

### 4. Data Integrity

**File Operations:**
- [ ] users.json saves correctly
- [ ] preferences.json saves correctly
- [ ] suggestions.json saves correctly
- [ ] Files readable after restart
- [ ] JSON structure valid

**Data Persistence:**
- [ ] OAuth tokens persist after server restart
- [ ] Preferences persist after server restart
- [ ] Suggestion history persists

---

### 5. Browser Compatibility

**Test in:**
- [ ] Chrome (primary)
- [ ] Safari (if available)
- [ ] Firefox (if available)
- [ ] Mobile browser (optional)

**Verify:**
- [ ] Forms work
- [ ] Buttons clickable
- [ ] Results display correctly
- [ ] No console errors

---

## Bug Fix Priorities

### P0 - Critical (Must Fix)
- Blocking bugs that prevent core demo flow
- OAuth failures
- Can't create calendar events
- Server crashes

### P1 - High (Should Fix)
- Non-blocking but visible errors
- Incorrect free time calculation
- Wrong activity matching
- UI errors

### P2 - Medium (Nice to Fix)
- Minor issues that don't affect demo
- Styling issues
- Edge case errors
- Performance issues

### P3 - Low (Document, Don't Fix)
- Known limitations
- Future enhancements
- Rare edge cases

---

## Testing Checklist

### Critical Path (Must Work)
- [ ] OAuth flow completes
- [ ] Preferences save
- [ ] Calendar events readable
- [ ] Free time detected
- [ ] Activities matched
- [ ] Suggestions created in both calendars
- [ ] Admin dashboard shows results

### Important (Should Work)
- [ ] Token refresh automatic
- [ ] Weekend filtering correct
- [ ] Duration filtering correct
- [ ] Match scoring reasonable
- [ ] Error messages user-friendly
- [ ] Calendar links work

### Nice to Have (Good if Works)
- [ ] All-day events handled
- [ ] Recurring events expanded
- [ ] Mobile view acceptable
- [ ] Fast performance
- [ ] Pretty UI

---

## Documentation Tasks

- [ ] Update README with testing notes
- [ ] Document known issues
- [ ] Document browser compatibility
- [ ] Document test account setup
- [ ] Add troubleshooting section

---

## Testing Tools

**Manual Testing:**
- Two Google test accounts
- Google Calendar (web + mobile)
- Browser developer tools
- Server logs (console)

**Debugging:**
- Chrome DevTools for frontend
- Console.log for backend
- Network tab for API calls
- Calendar API explorer (optional)

---

## Common Issues to Check

### Issue: Suggestions not appearing in calendar
**Check:**
- Event creation API succeeds (200 response)
- Event ID returned
- Correct calendar ID (should be 'primary')
- OAuth tokens valid
- User hasn't changed Google Calendar settings

### Issue: Free time calculation wrong
**Check:**
- Busy blocks extracted correctly
- All-day events handled
- Timezone consistent
- Weekend filter logic
- Duration filter logic

### Issue: No activity matches
**Check:**
- activities.json loaded
- Categories match interests
- Keywords match want-to-try
- Distance calculation correct
- Budget filtering correct

---

## Next Steps

After Phase 9:
- Document all tested scenarios
- List known issues
- Verify critical path works 3x
- Move to [Phase 10: Demo Preparation](./2026-02-12_phase-10_demo-preparation-plan.md)

**Phase 9 complete when:** End-to-end flow works reliably with no critical bugs.

---

**Reference Documents:**
- [High-Level Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture](../../aiDocs/architecture.md)
- [MVP Spec](../../aiDocs/mvp.md)
- [Phase 9 Roadmap](./2026-02-12_phase-9_integration-testing-roadmap.md)
