# Phase 1: OAuth Integration - Roadmap

**Created:** 2026-02-12
**Phase:** 1 of 10
**Timeline:** Weeks 1-2
**Status:** Not Started
**Companion Document:** [Phase 1 Plan](./2026-02-12_phase-1_oauth-integration-plan.md)

---

## Overview

**Goal:** Implement Google Calendar OAuth 2.0 authentication for both partners

**Duration:** Weeks 1-2 (8-12 hours of work)

**Why critical:** OAuth is the foundation. Without it, we can't read or write calendars. This is the hardest phase - expect to spend extra time debugging.

---

## Task Checklist

### OAuth Service Implementation
- [ ] Create server/services/auth.js file
- [ ] Implement initiateOAuthFlow(partnerId) function
  - [ ] Create OAuth2 client
  - [ ] Generate authorization URL with scopes
  - [ ] Include state parameter
  - [ ] Return auth URL
- [ ] Implement handleOAuthCallback(code, state) function
  - [ ] Exchange code for tokens
  - [ ] Parse state to identify partner
  - [ ] Extract access_token, refresh_token, expiry
  - [ ] Call saveTokens()
  - [ ] Return tokens
- [ ] Implement refreshAccessToken(coupleId, partnerId) function
  - [ ] Load refresh token from storage
  - [ ] Call Google token refresh API
  - [ ] Update stored tokens
  - [ ] Return new access token
- [ ] Implement getStoredTokens(coupleId, partnerId) function
  - [ ] Read from users.json
  - [ ] Return token object or null
- [ ] Implement saveTokens(coupleId, partnerId, tokens) function
  - [ ] Load existing users.json
  - [ ] Update partner tokens
  - [ ] Write back to file
- [ ] Add error handling for all functions

### OAuth Routes Implementation
- [ ] Create server/routes/auth.js file
- [ ] Implement GET /auth/google route
  - [ ] Parse partner query parameter
  - [ ] Parse couple query parameter (default couple-1)
  - [ ] Create state object
  - [ ] Call initiateOAuthFlow()
  - [ ] Redirect to Google consent screen
- [ ] Implement GET /auth/callback route
  - [ ] Check for error parameter (user denied)
  - [ ] Extract code and state from query
  - [ ] Parse state to get partnerId, coupleId
  - [ ] Call handleOAuthCallback()
  - [ ] Show success page or error
- [ ] Implement GET /auth/status/:coupleId route
  - [ ] Get tokens for both partners
  - [ ] Return authentication status JSON
  - [ ] Include email and expiry info
- [ ] Test all routes with Postman/browser

### Server Integration
- [ ] Update server/index.js
  - [ ] Import auth routes
  - [ ] Mount routes at /auth
  - [ ] Add error handling middleware
- [ ] Test server starts without errors
- [ ] Verify routes accessible

### UI Updates
- [ ] Update public/index.html
  - [ ] Add OAuth section with connect buttons
  - [ ] Add status display div
  - [ ] Add JavaScript to fetch status
  - [ ] Display connection status for both partners
- [ ] Update public/styles.css
  - [ ] Add OAuth section styles
  - [ ] Add button styles
  - [ ] Add status box styles
- [ ] Test UI loads and looks good

### Token Storage
- [ ] Verify users.json in .gitignore
- [ ] Test token save operation
- [ ] Test token retrieval operation
- [ ] Verify JSON structure matches spec
- [ ] Ensure file permissions correct (readable/writable)

### Testing & Verification
- [ ] Test OAuth flow for Partner 1
  - [ ] Click connect button
  - [ ] Redirected to Google
  - [ ] Approve permissions
  - [ ] Redirected back to success page
  - [ ] Token saved in users.json
- [ ] Test OAuth flow for Partner 2
  - [ ] Repeat above steps
  - [ ] Both tokens now in users.json
- [ ] Test status endpoint
  - [ ] GET /auth/status/couple-1
  - [ ] JSON shows both partners authenticated
  - [ ] Email addresses displayed
- [ ] Test token refresh
  - [ ] Manually expire token (set past date)
  - [ ] Call refresh function
  - [ ] New token saved
  - [ ] Can make API call with new token
- [ ] Test error cases
  - [ ] User denies consent
  - [ ] Invalid state parameter
  - [ ] Missing refresh token

### Documentation
- [ ] Document OAuth flow in comments
- [ ] Add troubleshooting section to README
- [ ] Document common errors and solutions
- [ ] Add example users.json structure

---

## Timeline

### Week 1, Days 1-2 (4-6 hours)
- **OAuth Service:** Build auth.js service module
  - Implement all OAuth functions
  - Test with mock data first
- **Milestone:** Service functions work in isolation

### Week 1, Days 3-4 (3-4 hours)
- **OAuth Routes:** Build routes/auth.js
  - Implement all three routes
  - Integrate with service layer
- **Server Integration:** Wire up routes in index.js
- **Milestone:** Routes respond, can test with browser

### Week 2, Day 1 (2-3 hours)
- **First OAuth Test:** Run full OAuth flow
  - Test with Partner 1
  - Debug redirect URI issues
  - Get first token stored
- **Milestone:** One partner successfully authenticated

### Week 2, Day 2 (2-3 hours)
- **Second OAuth Test:** Partner 2 authentication
  - Repeat OAuth flow
  - Both tokens in storage
- **UI Updates:** Add status display to homepage
- **Milestone:** Both partners authenticated, status displays

### Week 2, Day 3 (1-2 hours)
- **Token Refresh:** Test and debug refresh logic
- **Error Handling:** Test edge cases
- **Polish:** Clean up error messages, add logging
- **Milestone:** Phase 1 complete, ready for Phase 2

---

## Success Criteria

### Must Pass All:
1. **OAuth Flow Works:** Can click button, authenticate, get redirected back
2. **Tokens Stored:** Both partners' tokens in users.json with correct structure
3. **Status Displays:** /auth/status shows both partners authenticated
4. **Refresh Works:** Can refresh expired token and get new one
5. **No Errors:** Normal flow completes without errors
6. **UI Updated:** Homepage shows connection status
7. **Ready for Phase 2:** Have valid tokens to make Calendar API calls

### Phase 1 Complete When:
- All task checklist items checked
- All success criteria pass
- Both test accounts authenticated
- Tokens verified working (can make test API call)
- Ready to begin Phase 2 (Calendar Reading)

---

## Dependencies

**Blocked By:**
- Phase 0: Foundation (need server + Google Cloud setup)

**Requires:**
- Google Cloud OAuth credentials configured
- Test Google accounts available
- Internet connection for OAuth flow
- .env configured with client ID and secret

**Blocks:**
- Phase 2: Calendar Reading
- Phase 3: Free Time Detection
- All subsequent phases

---

## Risks & Mitigation

### Risk: OAuth redirect URI mismatch
**Impact:** OAuth flow fails with Google error
**Probability:** High (common mistake)
**Mitigation:**
- Triple-check redirect URI in Google Cloud Console
- Must be exactly: `http://localhost:3000/auth/callback`
- No trailing slash, http not https
**Fallback:** Can fix quickly once identified

### Risk: No refresh token received
**Impact:** Can't refresh expired access tokens, need to reauthorize
**Probability:** Medium
**Mitigation:**
- Use `access_type: 'offline'` in auth URL
- Use `prompt: 'consent'` to force consent screen
- Test immediately after OAuth to catch this
**Fallback:** User reauthorizes when needed (acceptable for MVP)

### Risk: Token storage issues
**Impact:** Tokens not saved, OAuth flow succeeds but can't retrieve tokens
**Probability:** Low
**Mitigation:**
- Test file I/O early
- Add error logging for file operations
- Verify file permissions
**Fallback:** Use in-memory storage for demo (not persistent)

### Risk: OAuth complexity causes delays
**Impact:** Phase takes longer than 2 weeks
**Probability:** Medium (OAuth is tricky)
**Mitigation:**
- Start early in Week 1
- Ask for help when stuck >1 hour
- Follow official Google examples closely
**Fallback:** Use hardcoded tokens for demo (not recommended but possible)

### Risk: User denies consent
**Impact:** Can't authenticate partner
**Probability:** Low (we control test accounts)
**Mitigation:**
- Use dedicated test accounts
- Clear error message if denied
- Allow retry
**Fallback:** N/A - need approval for system to work

---

## Blocked By

**Phase 0 must be complete:**
- Server running
- Google Cloud project configured
- OAuth credentials created
- .env configured

---

## Blocks

**These phases cannot start until Phase 1 complete:**
- Phase 2: Calendar Reading (needs tokens)
- Phase 3: Free Time Detection (needs calendar access)
- Phase 7: Suggestion Creation (needs tokens to write calendar)

**Critical Path:** Phase 1 is on the critical path. Delays here impact entire timeline.

---

## Milestone: OAuth Complete

**Definition of Done:**
- [ ] All tasks completed
- [ ] All success criteria met
- [ ] Both partners authenticated
- [ ] Tokens stored and retrievable
- [ ] Token refresh tested and working
- [ ] Status endpoint functional
- [ ] UI shows connection status
- [ ] Can make test Calendar API call (verify in Phase 2 start)

**Celebration checkpoint:** OAuth is the hardest part! Take a break when this works.

---

## Next Phase

**Phase 2: Calendar Reading**
- [Phase 2 Plan](./2026-02-12_phase-2_calendar-reading-plan.md)
- [Phase 2 Roadmap](./2026-02-12_phase-2_calendar-reading-roadmap.md)
- Timeline: Week 3
- Now that we have OAuth working, we can actually read calendars!

---

**Reference Documents:**
- [High-Level Implementation Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture Documentation](../../aiDocs/architecture.md)
- [MVP Specification](../../aiDocs/mvp.md)
- [Phase 1 Plan](./2026-02-12_phase-1_oauth-integration-plan.md)
- [Google OAuth 2.0 Docs](https://developers.google.com/identity/protocols/oauth2)
