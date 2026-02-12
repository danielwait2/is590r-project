# Phase 1: OAuth Integration - Implementation Plan

**Created:** 2026-02-12
**Phase:** 1 of 10
**Timeline:** Weeks 1-2
**Companion Document:** [Phase 1 Roadmap](./2026-02-12_phase-1_oauth-integration-roadmap.md)

---

## Project Principles

⚠️ **IMPORTANT: This is a clean-slate MVP build**

**DO:**
- Build the simplest thing that works
- Follow Google's OAuth 2.0 best practices
- Store tokens securely
- Handle token refresh properly

**DON'T:**
- Build complex session management
- Add user login/password systems
- Over-engineer token storage
- Add features beyond basic OAuth flow

**Note:** This is the hardest phase. OAuth can be tricky. Budget extra time and don't hesitate to ask for help.

---

## Phase Overview

**Goal:** Implement Google Calendar OAuth 2.0 authentication for both partners

**Why this matters:** OAuth is the foundation of the entire system. Without working OAuth, we can't read or write calendars. This is the critical path.

**Key deliverables:**
- OAuth service module
- OAuth routes (initiate and callback)
- Token storage and retrieval
- Token refresh mechanism
- Simple UI to trigger OAuth

---

## Technical Specifications

### 1. OAuth Service Module

**File:** `server/services/auth.js`

**Purpose:** Centralize all OAuth-related logic

**Required functions:**

#### initiateOAuthFlow(partnerId)
```javascript
/**
 * Initiates OAuth flow for a partner
 * @param {string} partnerId - 'partner1' or 'partner2'
 * @returns {string} authUrl - Google OAuth consent screen URL
 */
```

**Implementation details:**
- Create OAuth2 client using googleapis package
- Generate authorization URL with calendar.events scope
- Include state parameter with partnerId and coupleId
- Return URL for redirect

**Scopes required:**
- `https://www.googleapis.com/auth/calendar.events` (read/write calendar events)

**OAuth parameters:**
- access_type: 'offline' (to get refresh token)
- scope: 'calendar.events'
- state: JSON with partnerId and coupleId
- redirect_uri: from .env (http://localhost:3000/auth/callback)

#### handleOAuthCallback(code, state)
```javascript
/**
 * Handles OAuth callback from Google
 * @param {string} code - Authorization code from Google
 * @param {string} state - State parameter (contains partnerId, coupleId)
 * @returns {Promise<object>} tokens - Access token, refresh token, expiry
 */
```

**Implementation details:**
- Exchange authorization code for tokens
- Parse state parameter to identify partner
- Extract access_token, refresh_token, expiry_date
- Store tokens in users.json
- Return tokens object

**Error handling:**
- Invalid code: throw error
- Missing refresh_token: throw error (user needs to reauthorize)
- API errors: log and rethrow

#### refreshAccessToken(coupleId, partnerId)
```javascript
/**
 * Refreshes expired access token using refresh token
 * @param {string} coupleId - Couple identifier
 * @param {string} partnerId - 'partner1' or 'partner2'
 * @returns {Promise<string>} newAccessToken
 */
```

**Implementation details:**
- Load refresh token from users.json
- Call Google OAuth API to refresh
- Update access_token and expiry in users.json
- Return new access token

**When to call:** Automatically when API returns 401 Unauthorized

#### getStoredTokens(coupleId, partnerId)
```javascript
/**
 * Retrieves stored OAuth tokens for a partner
 * @param {string} coupleId - Couple identifier
 * @param {string} partnerId - 'partner1' or 'partner2'
 * @returns {object} tokens - Access token, refresh token, expiry
 */
```

**Implementation details:**
- Read from server/data/users.json
- Return token object for specified partner
- Return null if not found

#### saveTokens(coupleId, partnerId, tokens)
```javascript
/**
 * Saves OAuth tokens to storage
 * @param {string} coupleId - Couple identifier
 * @param {string} partnerId - 'partner1' or 'partner2'
 * @param {object} tokens - Token data from Google
 */
```

**Implementation details:**
- Load existing users.json
- Update specified partner's tokens
- Write back to users.json
- Handle file I/O errors gracefully

---

### 2. OAuth Routes

**File:** `server/routes/auth.js`

**Route: GET /auth/google**

**Purpose:** Initiate OAuth flow

**Query parameters:**
- partner: '1' or '2' (which partner is authenticating)
- couple: 'couple-1' (optional, defaults to couple-1 for MVP)

**Implementation:**
```javascript
router.get('/google', (req, res) => {
  const partnerId = req.query.partner === '2' ? 'partner2' : 'partner1';
  const coupleId = req.query.couple || 'couple-1';

  const state = JSON.stringify({ partnerId, coupleId });
  const authUrl = authService.initiateOAuthFlow(state);

  res.redirect(authUrl);
});
```

**Response:** Redirect to Google OAuth consent screen

---

**Route: GET /auth/callback**

**Purpose:** Handle OAuth callback from Google

**Query parameters:**
- code: Authorization code from Google
- state: Original state parameter
- error: Error from Google (if user denied)

**Implementation:**
```javascript
router.get('/callback', async (req, res) => {
  const { code, state, error } = req.query;

  if (error) {
    return res.send(`<h1>Authorization Failed</h1><p>${error}</p>`);
  }

  if (!code || !state) {
    return res.status(400).send('Missing code or state');
  }

  try {
    const { partnerId, coupleId } = JSON.parse(state);
    const tokens = await authService.handleOAuthCallback(code, state);

    res.send(`
      <h1>Success!</h1>
      <p>Partner ${partnerId === 'partner1' ? '1' : '2'} calendar connected.</p>
      <p><a href="/">Back to home</a></p>
    `);
  } catch (err) {
    console.error('OAuth callback error:', err);
    res.status(500).send(`<h1>Error</h1><p>${err.message}</p>`);
  }
});
```

**Success response:** HTML page confirming connection
**Error response:** HTML page with error message

---

**Route: GET /auth/status/:coupleId**

**Purpose:** Check authentication status for both partners

**Implementation:**
```javascript
router.get('/status/:coupleId', (req, res) => {
  const { coupleId } = req.params;

  const partner1Tokens = authService.getStoredTokens(coupleId, 'partner1');
  const partner2Tokens = authService.getStoredTokens(coupleId, 'partner2');

  res.json({
    coupleId,
    partner1: {
      authenticated: !!partner1Tokens?.accessToken,
      email: partner1Tokens?.email || null,
      tokenExpiry: partner1Tokens?.tokenExpiry || null
    },
    partner2: {
      authenticated: !!partner2Tokens?.accessToken,
      email: partner2Tokens?.email || null,
      tokenExpiry: partner2Tokens?.tokenExpiry || null
    }
  });
});
```

**Response format:**
```json
{
  "coupleId": "couple-1",
  "partner1": {
    "authenticated": true,
    "email": "alex@example.com",
    "tokenExpiry": "2026-02-12T15:00:00Z"
  },
  "partner2": {
    "authenticated": true,
    "email": "jordan@example.com",
    "tokenExpiry": "2026-02-12T15:30:00Z"
  }
}
```

---

### 3. Token Storage

**File:** `server/data/users.json`

**Structure:**
```json
{
  "couple-1": {
    "partner1": {
      "email": "alex@example.com",
      "accessToken": "ya29.a0AfH6SMBx...",
      "refreshToken": "1//0gQ1K5Z...",
      "tokenExpiry": "2026-02-12T15:00:00Z",
      "scope": "https://www.googleapis.com/auth/calendar.events"
    },
    "partner2": {
      "email": "jordan@example.com",
      "accessToken": "ya29.a0AfH6SMCy...",
      "refreshToken": "1//0gR2L6A...",
      "tokenExpiry": "2026-02-12T15:30:00Z",
      "scope": "https://www.googleapis.com/auth/calendar.events"
    }
  }
}
```

**Fields:**
- email: User's Google email (from userinfo or ID token)
- accessToken: Short-lived token for API calls (expires in ~1 hour)
- refreshToken: Long-lived token to get new access tokens
- tokenExpiry: ISO 8601 timestamp when access token expires
- scope: Granted scopes (for verification)

**Security notes:**
- This file MUST be in .gitignore
- Contains sensitive OAuth tokens
- Tokens should be stored securely in production (encrypted database)
- For MVP, plain JSON is acceptable for demo purposes

---

### 4. OAuth UI Integration

**Update:** `public/index.html`

**Add OAuth section:**
```html
<section class="oauth-section">
    <h2>Step 1: Connect Calendars</h2>
    <p>Both partners need to connect their Google Calendars.</p>

    <div class="oauth-buttons">
        <a href="/auth/google?partner=1&couple=couple-1" class="btn btn-oauth">
            Connect Partner 1 Calendar
        </a>
        <a href="/auth/google?partner=2&couple=couple-1" class="btn btn-oauth">
            Connect Partner 2 Calendar
        </a>
    </div>

    <div id="oauth-status" class="status-box">
        <p>Loading status...</p>
    </div>
</section>

<script>
// Check OAuth status on page load
async function checkOAuthStatus() {
    try {
        const response = await fetch('/auth/status/couple-1');
        const data = await response.json();

        const statusDiv = document.getElementById('oauth-status');
        statusDiv.innerHTML = `
            <h3>Connection Status</h3>
            <p>Partner 1: ${data.partner1.authenticated ? '✓ Connected' : '✗ Not connected'}
               ${data.partner1.email ? `(${data.partner1.email})` : ''}</p>
            <p>Partner 2: ${data.partner2.authenticated ? '✓ Connected' : '✗ Not connected'}
               ${data.partner2.email ? `(${data.partner2.email})` : ''}</p>
        `;
    } catch (err) {
        console.error('Failed to check OAuth status:', err);
    }
}

// Run on page load
checkOAuthStatus();
</script>
```

**Add CSS for OAuth section:**
```css
.oauth-section {
    margin: 30px 0;
    padding: 20px;
    background: #f8f9fa;
    border-radius: 8px;
}

.oauth-buttons {
    display: flex;
    gap: 15px;
    margin: 20px 0;
}

.btn-oauth {
    background: #4285f4;
    flex: 1;
}

.btn-oauth:hover {
    background: #357ae8;
}

.status-box {
    margin-top: 20px;
    padding: 15px;
    background: white;
    border: 1px solid #ddd;
    border-radius: 4px;
}
```

---

### 5. Server Integration

**Update:** `server/index.js`

**Add OAuth routes:**
```javascript
const authRoutes = require('./routes/auth');

// Mount auth routes
app.use('/auth', authRoutes);
```

**Add error handling middleware:**
```javascript
// Error handling
app.use((err, req, res, next) => {
  console.error('Server error:', err);
  res.status(500).json({
    error: 'Internal server error',
    message: err.message
  });
});
```

---

## API Reference

### Google OAuth 2.0

**Authorization endpoint:**
```
https://accounts.google.com/o/oauth2/v2/auth
```

**Token endpoint:**
```
https://oauth2.googleapis.com/token
```

**Required parameters for authorization:**
- client_id: From Google Cloud Console
- redirect_uri: Must match registered URI exactly
- response_type: 'code'
- scope: 'https://www.googleapis.com/auth/calendar.events'
- access_type: 'offline' (to get refresh token)
- prompt: 'consent' (force consent screen to get refresh token)

**Token exchange:**
- POST to token endpoint with:
  - code: Authorization code
  - client_id: Client ID
  - client_secret: Client secret
  - redirect_uri: Same as authorization
  - grant_type: 'authorization_code'

**Token refresh:**
- POST to token endpoint with:
  - refresh_token: Refresh token
  - client_id: Client ID
  - client_secret: Client secret
  - grant_type: 'refresh_token'

### googleapis npm package

**Installation:**
```bash
npm install googleapis
```

**Usage:**
```javascript
const { google } = require('googleapis');

const oauth2Client = new google.auth.OAuth2(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  process.env.GOOGLE_REDIRECT_URI
);

// Generate auth URL
const authUrl = oauth2Client.generateAuthUrl({
  access_type: 'offline',
  scope: ['https://www.googleapis.com/auth/calendar.events'],
  state: JSON.stringify({ partnerId, coupleId })
});

// Exchange code for tokens
const { tokens } = await oauth2Client.getToken(code);

// Refresh token
oauth2Client.setCredentials({ refresh_token: refreshToken });
const { credentials } = await oauth2Client.refreshAccessToken();
```

---

## Testing Criteria

### OAuth Flow Testing
- [ ] Can click "Connect Partner 1" button
- [ ] Redirects to Google consent screen
- [ ] Consent screen shows correct app name
- [ ] Consent screen requests calendar.events scope
- [ ] Can approve consent
- [ ] Redirects back to callback URL
- [ ] Success page displays
- [ ] Token saved in users.json
- [ ] Repeat for Partner 2
- [ ] Both partners' tokens visible in users.json

### Token Storage Testing
- [ ] users.json contains both partners' tokens
- [ ] Access token present and non-empty
- [ ] Refresh token present and non-empty
- [ ] Token expiry is future timestamp
- [ ] Email address stored correctly
- [ ] JSON structure matches specification

### Token Refresh Testing
- [ ] Can manually expire token (set expiry to past)
- [ ] Refresh function successfully gets new token
- [ ] New token saved to users.json
- [ ] New expiry is future timestamp
- [ ] Can make API call with refreshed token

### Status Endpoint Testing
- [ ] GET /auth/status/couple-1 returns JSON
- [ ] Shows authentication status for both partners
- [ ] Shows email addresses when authenticated
- [ ] Shows token expiry times
- [ ] Updates when new tokens added

### UI Testing
- [ ] OAuth status loads on homepage
- [ ] Shows connection status for both partners
- [ ] Updates after successful OAuth
- [ ] Displays email addresses correctly
- [ ] "Connect" buttons work

### Error Handling Testing
- [ ] User denies consent → shows error message
- [ ] Invalid redirect URI → Google shows error
- [ ] Missing client credentials → server error
- [ ] Expired refresh token → reauthorization required
- [ ] Network error → graceful failure

---

## Common Issues & Solutions

### Issue: Redirect URI mismatch
**Symptoms:** Error from Google: "redirect_uri_mismatch"
**Solution:**
- Check Google Cloud Console OAuth credentials
- Ensure redirect URI exactly: `http://localhost:3000/auth/callback`
- No trailing slash, http not https
- Restart server after changes

### Issue: No refresh token received
**Symptoms:** tokens.refresh_token is undefined
**Solution:**
- Add `access_type: 'offline'` to auth URL
- Add `prompt: 'consent'` to force consent screen
- User may need to revoke access and reauthorize
- Check test user is added to OAuth consent screen

### Issue: Token expired, refresh fails
**Symptoms:** API returns 401, refresh returns error
**Solution:**
- Check refresh token stored correctly
- Verify client credentials in .env
- User may need to reauthorize (refresh tokens can expire if not used)
- Check Google Cloud Console for API quotas

### Issue: State parameter lost
**Symptoms:** Can't identify which partner authenticated
**Solution:**
- Ensure state parameter in auth URL
- Parse state in callback correctly
- URL-encode state if contains special characters

### Issue: CORS errors in browser
**Symptoms:** Fetch fails with CORS error
**Solution:**
- OAuth flow should use redirects, not fetch
- Callback is server-side, not AJAX
- Status endpoint is AJAX, ensure Express CORS configured

---

## Blockers & Dependencies

### Blocked By
- Phase 0: Foundation (need server + Google Cloud setup)

### Blocks
- Phase 2: Calendar Reading (needs OAuth tokens)
- Phase 3: Free Time Detection (needs calendar access)
- All subsequent phases (everything needs authentication)

### Dependencies
- Google Cloud project configured (Phase 0)
- OAuth credentials created (Phase 0)
- Test Google accounts available
- Internet connection for OAuth flow

---

## Next Steps

After completing Phase 1:
1. Verify both partners can authenticate
2. Confirm tokens stored in users.json
3. Test token refresh mechanism
4. Move to [Phase 2: Calendar Reading](./2026-02-12_phase-2_calendar-reading-plan.md)
5. Review [Phase 2 Roadmap](./2026-02-12_phase-2_calendar-reading-roadmap.md)

**Phase 1 is complete when:**
- Both partners authenticated via OAuth
- Tokens stored and retrievable
- Token refresh works
- Status endpoint shows connection
- No OAuth errors in normal flow

---

**Reference Documents:**
- [High-Level Implementation Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture Documentation](../../aiDocs/architecture.md)
- [MVP Specification](../../aiDocs/mvp.md)
- [Phase 1 Roadmap](./2026-02-12_phase-1_oauth-integration-roadmap.md)
- [Google OAuth 2.0 Documentation](https://developers.google.com/identity/protocols/oauth2)
