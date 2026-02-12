# System Architecture

**Project:** Calendar-Native Weekend Experience Concierge
**Version:** MVP 1.0
**Last Updated:** 2026-02-12

> **Important:** This is the **AUTHORITATIVE technical specification** for the MVP. All implementation should follow this document's structure, data models, and API specifications. See [ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md](../ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md) for phase-by-phase implementation guidance.

**MVP Scope:** 1 test couple, Node.js + Express, JSON files, 6-8 weeks

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         USER LAYER                          │
├─────────────────┬───────────────────────┬───────────────────┤
│  Partner 1      │   Partner 2           │  Admin Dashboard  │
│  Google Calendar│   Google Calendar     │  (Manual Trigger) │
└────────┬────────┴──────────┬────────────┴──────────┬────────┘
         │                   │                       │
         └───────────────────┼───────────────────────┘
                             │
                    ┌────────▼────────┐
                    │   Web Frontend  │
                    │  (HTML/CSS/JS)  │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Express Server │
                    │   (Node.js)     │
                    └────────┬────────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────▼────┐      ┌──────▼──────┐     ┌─────▼─────┐
    │  OAuth  │      │  Matching   │     │ Calendar  │
    │ Service │      │   Engine    │     │  Service  │
    └────┬────┘      └──────┬──────┘     └─────┬─────┘
         │                  │                   │
         │           ┌──────▼──────┐            │
         │           │ Activities  │            │
         │           │  Database   │            │
         │           │ (JSON File) │            │
         │           └─────────────┘            │
         │                                      │
         └──────────────┬───────────────────────┘
                        │
              ┌─────────▼─────────┐
              │  Google Calendar  │
              │       API v3      │
              │  (External API)   │
              └───────────────────┘
```

**Implementation Reference:** See [Phase Roadmaps](../ai/roadmaps/) for detailed phase-by-phase implementation guidance.

---

## System Components

### 1. Frontend Layer

#### Admin Dashboard
**Technology:** HTML + CSS + Vanilla JavaScript
**Purpose:** Manual triggering and results display
**Location:** `/public/admin.html`
**Implementation:** See [Phase 8 Plan](../ai/roadmaps/2026-02-12_phase-8_admin-dashboard-plan.md)

**Features:**

1. **Couple Status Display:**
   - Partner 1 calendar: ✓ Connected / ✗ Not connected (with email)
   - Partner 2 calendar: ✓ Connected / ✗ Not connected (with email)
   - Preferences: ✓ Set / ✗ Not set
   - Overall status: "Ready to generate suggestions" or "Complete setup first"
   - Auto-loads on page load via `/auth/status/:coupleId` and `/preferences/:coupleId/data`

2. **Generate Suggestions Button:**
   - Triggers POST to `/matching/:coupleId`
   - Disabled if setup incomplete (either partner not authenticated or no preferences)
   - Shows loading state during matching ("Generating...")
   - Re-enables after completion

3. **Results Display:**
   - Shows free time windows found (list with dates/times/durations)
   - Shows activities matched count
   - Shows suggestions created with details:
     - Activity name, time, location
     - Match reason (why suggested)
     - Match score
     - Links to view in Google Calendar (Partner 1 and Partner 2)
   - Friendly messages for empty results

4. **Activity Log:**
   - Real-time log of actions (newest first)
   - Timestamps for each entry
   - Different message types: info, success, error
   - Examples: "Dashboard loaded", "Generated 2 suggestions", "Error: No free time found"

5. **Navigation:**
   - Link back to home page
   - Could add links to preferences, OAuth setup (future)

**Why simple:** MVP doesn't need React/Vue complexity
- No build step required
- Easy to debug
- Fast to implement
- Sufficient for demo purposes

**UI Flow:**
1. User navigates to /admin
2. Dashboard loads, fetches couple status
3. If not ready: Shows what's missing, generate button disabled
4. If ready: Shows all connected, generate button enabled
5. User clicks "Generate Suggestions"
6. Button shows loading state, makes POST request to /matching/:coupleId
7. Server runs matching engine (4-5 seconds)
8. Display results (free windows, matches, suggestions with calendar links)
9. Activity log updated with "Generated X suggestions"
10. User clicks calendar links to view in Google Calendar

**Error Handling:**
- Network errors: Show error message, keep button enabled for retry
- API errors: Display error.message from server response
- Empty results: Show friendly message (not error)
- Reauth needed: Show link to reconnect calendar

---

### 2. Backend Layer

#### Express Server
**Technology:** Node.js + Express
**Port:** 3000 (development)
**Entry Point:** `server/index.js`

**Responsibilities:**
- Route HTTP requests
- Handle OAuth callbacks
- Serve static files
- Coordinate between services
- Error handling

**Key Routes:**
```
GET  /                          → Landing page
GET  /health                    → Health check endpoint

# OAuth Routes (Phase 1)
GET  /auth/google               → Initiate OAuth flow (with ?partner=1|2&couple=couple-1)
GET  /auth/callback             → OAuth callback handler (from Google)
GET  /auth/status/:coupleId     → Get authentication status (both partners)

# Preferences Routes (Phase 4)
GET  /preferences/:coupleId     → Get preferences form (HTML)
GET  /preferences/:coupleId/data → Get preferences data (JSON)
POST /preferences/:coupleId     → Save preferences

# Calendar Routes (Phase 2-3)
GET  /calendar/:coupleId/events → Get calendar events for both partners (debug)
GET  /calendar/:coupleId/busy   → Get busy blocks for both partners (debug)
GET  /calendar/:coupleId/free   → Get overlapping free time windows

# Matching Routes (Phase 6-7)
POST /matching/:coupleId        → Trigger matching engine (creates suggestions)

# Admin Routes (Phase 8)
GET  /admin                     → Admin dashboard
```

**Implementation:** See [Phase 0 Plan](../ai/roadmaps/2026-02-12_phase-0_foundation-plan.md) for server setup details.

---

### 3. Service Layer

#### 3.1 OAuth Service
**File:** `server/services/auth.js`
**Purpose:** Handle Google OAuth 2.0 authentication
**Implementation:** See [Phase 1 Plan](../ai/roadmaps/2026-02-12_phase-1_oauth-integration-plan.md) for detailed OAuth implementation

**Functions:**

**initiateOAuthFlow(partnerId)**
- Creates OAuth2 client using googleapis package
- Generates authorization URL with calendar.events scope
- Includes state parameter with partnerId and coupleId (JSON stringified)
- Returns URL for redirect to Google consent screen
- OAuth parameters:
  - access_type: 'offline' (to get refresh token)
  - scope: 'calendar.events'
  - state: JSON with partnerId and coupleId
  - prompt: 'consent' (force consent screen to ensure refresh token)

**handleOAuthCallback(code, state)**
- Exchanges authorization code for tokens via Google API
- Parses state parameter to identify partner
- Extracts access_token, refresh_token, expiry_date from response
- Stores tokens in users.json via saveTokens()
- Returns tokens object
- Error handling:
  - Invalid code: throw error
  - Missing refresh_token: throw error (user needs to reauthorize)
  - API errors: log and rethrow

**refreshAccessToken(coupleId, partnerId)**
- Loads refresh token from users.json
- Calls Google OAuth API to refresh
- Updates access_token and expiry in users.json
- Returns new access token
- Called automatically when API returns 401 Unauthorized

**getStoredTokens(coupleId, partnerId)**
- Reads from server/data/users.json
- Returns token object for specified partner
- Returns null if not found

**saveTokens(coupleId, partnerId, tokens)**
- Loads existing users.json
- Updates specified partner's tokens
- Writes back to users.json
- Handles file I/O errors gracefully

**Dependencies:**
- `googleapis` npm package (OAuth2 client)
- `data/users.json` (token storage - must be in .gitignore)

**OAuth Flow:**
1. User clicks "Connect Calendar" with partner=1 or partner=2
2. Server generates auth URL with state parameter
3. Redirect to Google consent screen
4. User approves permissions (calendar.events scope)
5. Google redirects to /auth/callback?code=...&state=...
6. Server exchanges code for tokens (access + refresh)
7. Store tokens in users.json with email and expiry
8. Redirect to success page showing partner connected

**Scopes Requested:**
- `https://www.googleapis.com/auth/calendar.events` (read + write events)

**OAuth Configuration (Google Cloud Console):**
- Application type: Web application
- Authorized redirect URI: `http://localhost:3000/auth/callback` (must match exactly)
- OAuth consent screen: External (for testing with personal accounts)
- Test users: Add both partner test account emails

**Token Refresh Strategy:**
- Access tokens expire in ~1 hour
- Refresh tokens are long-lived (until revoked)
- Auto-refresh on 401 Unauthorized from Calendar API
- Retry API call with new token after refresh
- If refresh fails, user must reauthorize

**Security Notes:**
- Store tokens in users.json (in .gitignore)
- Never commit OAuth client secret or credentials file
- In production, encrypt tokens in database
- Use HTTPS in production
- Validate state parameter to prevent CSRF

---

#### 3.2 Calendar Service
**File:** `server/services/calendar.js`
**Purpose:** Interact with Google Calendar API
**Implementation:** See [Phase 2 Plan](../ai/roadmaps/2026-02-12_phase-2_calendar-reading-plan.md) and [Phase 3 Plan](../ai/roadmaps/2026-02-12_phase-3_free-time-detection-plan.md)

**Functions:**

**getCalendarEvents(coupleId, partnerId, startDate, endDate)**
- Loads OAuth tokens using auth service
- Creates OAuth2 client with tokens
- Calls: GET /calendars/primary/events via googleapis
- API parameters:
  - timeMin: startDate (ISO 8601)
  - timeMax: endDate (ISO 8601)
  - singleEvents: true (expand recurring events)
  - orderBy: 'startTime' (requires singleEvents=true)
  - maxResults: 100 (sufficient for 2 weeks)
- Handles token expiry: 401 → refresh → retry
- Returns: Array of event objects
- Event format: { id, summary, start: {dateTime}, end: {dateTime}, status }

**extractBusyBlocks(events)**
- Filters out cancelled events (status !== 'cancelled')
- Handles different event formats:
  - Timed events: Use dateTime directly
  - All-day events: Convert to 9am-5pm blocks on the date(s)
  - Tentative events: Still count as busy (conservative approach)
- Extracts start and end times to ISO 8601 strings
- Sorts by start time
- Returns: Array of { start, end } busy blocks

**getBusyBlocksForCouple(coupleId, startDate, endDate)**
- Calls getCalendarEvents for both partners
- Extracts busy blocks for each using extractBusyBlocks()
- Returns: { partner1: {email, busyBlocks}, partner2: {email, busyBlocks} }

**findFreeWindows(busyBlocks, dateRange, preferences)**
- Creates timeline from start to end of date range
- Marks busy blocks as unavailable
- Finds gaps (free windows) between busy blocks
- Applies filters:
  1. Weekend only: Fri 6pm-Sun 11pm (isWeekendTime() function)
  2. Minimum duration: default 120 minutes from preferences
  3. Advance notice: default 3 days from now
- Returns: Sorted array of free windows with metadata
- Window format: { start, end, durationMinutes, dayOfWeek, isWeekend }

**findOverlappingFreeTime(partner1Free, partner2Free)**
- For each window in partner1Free
- For each window in partner2Free
- Finds intersection (overlapping time)
- If overlap exists and meets min duration, adds to results
- Returns: Array of overlapping windows when both partners are free

**getAvailableTimeForCouple(coupleId, preferences)**
- Main orchestration function
- Loads preferences (or uses defaults)
- Calculates date range (next 2 weeks from today)
- Gets busy blocks for both partners
- Finds free windows for each partner individually
- Finds overlapping free windows between both
- Applies all filters (weekend, duration, advance notice)
- Returns: Sorted list of available time windows

**createSuggestionEvent(coupleId, suggestion)**
- Gets tokens for both partners
- Formats event per specification:
  - summary: "Suggested: [Activity Name]"
  - description: Formatted with activity details, match reasons, instructions
  - location: Activity venue address
  - start/end: From suggestion time window
  - transparency: 'tentative' (shows as tentative in calendar)
  - colorId: '11' (optional, red for visibility)
- Creates event in Partner 1's calendar via events.insert API
- Creates identical event in Partner 2's calendar
- Returns: { partner1EventId, partner1EventLink, partner2EventId, partner2EventLink }
- Error handling: If one fails, log error and continue with other

**formatEventDescription(suggestion)**
- Creates formatted description string:
  - Activity description
  - Match reasons (why suggested)
  - Duration, price, location details
  - Link to activity URL
  - Instructions to accept/decline
- Returns: Formatted string for calendar event description

**API Integration Details:**
- Uses `googleapis` npm package: `google.calendar({ version: 'v3', auth: oauth2Client })`
- OAuth2 client created per partner with stored credentials
- Automatic token refresh on 401 errors via wrapper function
- Rate limits: 10 QPS per user, 1M requests/day (well within MVP usage)

**Weekend Filtering Logic:**

Weekend definition:
- Friday: 18:00 (6pm) - 23:59
- Saturday: 00:00 - 23:59 (all day)
- Sunday: 00:00 - 23:00 (11pm)

isWeekendTime(dateTime) logic:
- day = date.getDay() where 0=Sun, 5=Fri, 6=Sat
- hour = date.getHours()
- If day === 5: return hour >= 18 (Friday after 6pm)
- If day === 6: return true (All Saturday)
- If day === 0: return hour < 23 (Sunday before 11pm)
- Otherwise: return false

**Free Time Algorithm (Complete):**
```
1. Get events from both calendars (next 2 weeks)
2. Extract busy blocks for each partner:
   - Filter cancelled events
   - Handle all-day events (9am-5pm)
   - Handle recurring events (already expanded by singleEvents=true)
3. For each partner, find free windows:
   - Start with full date range
   - Subtract each busy block
   - Remaining gaps are free windows
4. Find overlapping free windows:
   - Compare partner1 free windows with partner2 free windows
   - Calculate intersection for each pair
   - Keep only windows where both are free
5. Apply preference filters:
   - Weekend only: Keep only Fri 6pm - Sun 11pm windows
   - Minimum duration: Keep only windows >= minDuration (default 120 min)
   - Advance notice: Keep only windows >= advanceNotice days from now (default 3)
6. Sort by start time
7. Return sorted list of available windows
```

**Default Preferences (if not set):**
- minDuration: 120 minutes (2 hours)
- advanceNotice: 3 days
- weekendOnly: true
- maxDistance: 15 miles (for activity matching)

**Error Handling:**
- 401 Unauthorized: Trigger token refresh, retry API call
- 403 Forbidden: User revoked access, notify admin
- 429 Too Many Requests: Backoff and retry (unlikely with our volume)
- 500 Server Error: Log and return error to user
- Network failures: Retry 3x with exponential backoff

---

#### 3.3 Matching Engine
**File:** `server/services/matching.js`
**Purpose:** Core logic to match activities to free time
**Implementation:** See [Phase 6 Plan](../ai/roadmaps/2026-02-12_phase-6_matching-engine-plan.md) and [Phase 7 Plan](../ai/roadmaps/2026-02-12_phase-7_suggestion-creation-plan.md)

**Main Function:**

**generateSuggestions(coupleId)** - Orchestrates the entire matching process
1. Load couple preferences via preferencesService (throw error if missing)
2. Get overlapping free time windows via calendarService.getAvailableTimeForCouple()
3. Search for matching activities via activitiesService.searchActivities() (filter by interests, keywords, location, budget)
4. Score and rank activities using calculateActivityScore()
5. Match activities to time windows (find activities that fit each window's duration)
6. Create calendar events for top 2 suggestions in both partners' calendars
7. Log suggestions to suggestions.json with timestamp and match details
8. Return results with success status, free windows, matched activities, and created suggestions

**Scoring Algorithm:**

**calculateActivityScore(activity, preferences)** - Returns numeric score for activity match quality

Scoring components:
- Category match (interests): +10 points if activity.category in preferences.interests
- Keyword matches (want-to-try list): +5 points each (fuzzy matching, case-insensitive)
- Location proximity: +3 (<5 miles), +2 (5-10 miles), +1 (10-15 miles) using Haversine distance
- Budget match: +2 if activity price <= preference budget

Typical score range: 0-30+ points. Good matches: 15+ points.

**Match Reason Generation:**

**generateMatchReasons(activity, preferences)** - Creates human-readable explanation of why activity was matched

Returns string with:
- Category match: "Matches your interest in [category]"
- Want-to-try match: 'On your want-to-try list: "[keyword]"' (shows first match)
- Location: "[distance] miles from you"

Example: "Matches your interest in arts. On your want-to-try list: pottery class. 2.3 miles from you"

**Time Window Matching Logic:**

**matchActivitiesToTimeWindows(activities, timeWindows)** - Matches activities to available time slots

Process:
- For each time window, find first activity that fits within duration
- Create suggestion with activity details, time window, score, and match reasons
- Remove matched activity to prevent duplicates
- Limit to 2 suggestions maximum (MVP constraint)
- Returns array of suggestion objects with suggestedStart, suggestedEnd, activity, score, matchReasons

**Suggestion Logging:**

**logSuggestion(coupleId, suggestion, calendarEventIds, calendarLinks)** - Saves suggestion to audit log

Saves to server/data/suggestions.json with:
- id: "sugg-" + timestamp
- coupleId, activityId, activityName
- suggestedTime: { start, end }
- matchReason, matchScore, status='sent'
- calendarEventIds: { partner1, partner2 }
- calendarLinks: { partner1, partner2 }
- createdAt: ISO 8601 timestamp

Appends to existing suggestions array, returns logged record.

**Error Handling:**
- No preferences: Throw error "Preferences not set for this couple"
- No free time: Return success with message
- No matching activities: Return success with message
- Calendar event creation fails: Log error, continue with remaining suggestions
- All errors logged to console with context

---

#### 3.4 Activity Database Service
**File:** `server/services/activities.js`
**Purpose:** Load and search activity database
**Implementation:** See [Phase 5 Plan](../ai/roadmaps/2026-02-12_phase-5_activity-database-plan.md)

**Functions:**

**loadActivities()** - Reads server/data/activities.json, parses JSON, returns array of activity objects. Throws error if file not found or invalid JSON.

**searchActivities(filters)** - Loads all activities and applies filters sequentially:
1. Category filter: Keep only activities where category matches preferences.interests
2. Keyword filter: Fuzzy matching (case-insensitive, substring) against want-to-try list
3. Location filter: Keep only activities within maxDistance (calculated via Haversine formula)
4. Budget filter: Keep only activities where price <= budget ($ < $$ < $$$ < $$$$)

Returns filtered array of activities (no sorting - done by matching engine after scoring).

**calculateDistance(lat1, lon1, lat2, lon2)** - Implements Haversine formula for great-circle distance. Returns distance in miles (float). Used for location filtering and scoring.

**comparePriceLevels(activityPrice, budgetPrice)** - Converts price symbols ($ to $$$$) to numeric levels (1-4). Returns difference. Negative = activity cheaper, zero = same level, positive = activity more expensive.

**Activity Categories:**
Must match preference interest checkboxes:
- dining
- arts
- music
- classes
- outdoors
- sports
- entertainment

**No external API** - All data is local JSON file
- Ensures reliability for demo
- No API keys or rate limits
- Curated, high-quality activities
- Manually maintained (20-30 activities for MVP)

---

### 4. Data Layer

#### 4.1 Data Files (JSON Storage)

**users.json** - OAuth token storage

Structure: `{ "couple-id": { "partner1": {...}, "partner2": {...} } }`

Token fields per partner:
- `email`: User's Google email from OAuth (string)
- `accessToken`: Short-lived token for API calls (string, expires in ~1 hour)
- `refreshToken`: Long-lived token to get new access tokens (string, persists until revoked)
- `tokenExpiry`: When access token expires (ISO 8601 string)
- `scope`: Granted OAuth scopes (string, typically "calendar.events")

**Security:** MUST be in .gitignore - contains sensitive OAuth tokens. Plain JSON for MVP, should be encrypted database in production.

**preferences.json** - User preferences and interests

Structure: `{ "couple-id": { interests, wantToTry, location, availability, budget, updatedAt } }`

Preference fields per couple:
- `interests`: Array of categories (dining, arts, music, classes, outdoors, sports, entertainment)
- `wantToTry`: Array of free-form strings used for keyword matching
- `location.address`: Home address or general area (string)
- `location.lat` / `location.lng`: Latitude/longitude (floats, geocoded from address)
- `availability.days`: Always ["friday_evening", "saturday", "sunday"] for MVP
- `availability.minDuration`: Minimum activity duration in minutes (integer, default 120)
- `availability.advanceNotice`: Days of advance notice needed (integer, default 3)
- `budget`: Price level $ to $$$$ (string)
- `updatedAt`: Last update timestamp (ISO 8601 string)

**Collection:** Via `/preferences/couple-1` HTML form (see Phase 4 Plan). Geocoding converts address to lat/lng on save.

**activities.json** - Hardcoded activity database

Structure: Array of activity objects

Activity fields:
- `id`: Unique identifier (string)
- `name`: Activity name (string)
- `category`: One of: dining, arts, music, classes, outdoors, sports, entertainment
- `description`: What the activity involves (string, used in calendar event)
- `location.name`: Venue name (string)
- `location.address`: Full address (string, used in calendar event location)
- `location.lat` / `location.lng`: Latitude/longitude (floats, for distance calculation)
- `duration`: Activity duration in minutes (integer, for time window matching)
- `price`: Price level $ to $$$$ (string, for budget filtering)
- `priceAmount`: Actual price in dollars (integer, for display)
- `keywords`: Array of strings for want-to-try matching
- `url`: Link to activity/venue website (string, included in calendar event)
- `daysAvailable`: Days offered (array, for future filtering)
- `timeSlots`: When offered (array, for future filtering)
- `notes`: Additional information (string, optional)

**MVP Activity Database:**
- 20-30 curated activities covering all categories
- Mix of dining (5-6), arts (4-5), music (3-4), classes (4-5), outdoors (4-5), sports (2-3), entertainment (2-3)
- All with accurate lat/lng coordinates for target city
- Keywords carefully chosen to match common want-to-try items
- See [Phase 5 Plan](../ai/roadmaps/2026-02-12_phase-5_activity-database-plan.md) for sample activity structure

**suggestions.json** - Audit log of created suggestions

Structure: Array of suggestion records

Suggestion fields:
- `id`: Unique suggestion ID (string, generated as "sugg-" + timestamp)
- `coupleId`: Which couple (string)
- `activityId`: Activity from activities.json (string)
- `activityName`: Activity name (string, for display)
- `suggestedTime.start` / `suggestedTime.end`: When suggestion starts/ends (ISO 8601 strings)
- `matchReason`: Human-readable explanation (string, shown in calendar)
- `matchScore`: Numeric score from matching algorithm (integer)
- `status`: "sent" (string, for future tracking of accepted/declined)
- `calendarEventIds.partner1` / `calendarEventIds.partner2`: Google Calendar event IDs (strings)
- `calendarLinks.partner1` / `calendarLinks.partner2`: Direct links to events in Google Calendar (strings)
- `createdAt`: When suggestion was created (ISO 8601 string)

**Purpose:** Audit log of all suggestions created, used for admin dashboard history

#### 4.2 Data Flow

**Setup Flow:**
```
User Input (form) → POST /preferences/:coupleId → preferences.json
OAuth Flow → Google API → users.json (tokens)
```

**Suggestion Flow:**
```
Admin clicks "Generate"
  → POST /matching/:coupleId
  → Load preferences.json
  → Load users.json (tokens)
  → Call Google Calendar API (read events)
  → Calculate free time (in-memory)
  → Load activities.json
  → Match + score (in-memory)
  → Call Google Calendar API (create events)
  → Save to suggestions.json
  → Return results to dashboard
```

---

### 5. External Integrations

#### Google Calendar API v3
**Base URL:** `https://www.googleapis.com/calendar/v3`
**Authentication:** OAuth 2.0 (access token in Authorization header)

**Endpoints Used:**

**GET /calendars/primary/events**
- **Purpose:** List calendar events
- **Parameters:**
  - `timeMin` (ISO 8601 timestamp)
  - `timeMax` (ISO 8601 timestamp)
  - `singleEvents=true` (expand recurring events)
- **Response:** Array of event objects
- **Rate Limit:** 10 QPS per user

**POST /calendars/primary/events**
- **Purpose:** Create new calendar event (suggestion)
- **Body fields:**
  - `summary`: Event title, prefixed with "Suggested: " to identify suggestions
  - `description`: Formatted with activity details, match reasons, instructions (accept/decline)
  - `start.dateTime` / `end.dateTime`: ISO 8601 timestamps with timezone
  - `start.timeZone` / `end.timeZone`: Should match couple's location (detect from preferences)
  - `location`: Full venue address for map integration
  - `transparency`: "tentative" (shows as tentative in calendar UI)
  - `colorId`: "11" for red (optional, for visibility)
- **Response:** Created event object with `id` and `htmlLink`
- **Rate Limit:** 10 QPS per user
- **Implementation:** See [Phase 7 Plan](../ai/roadmaps/2026-02-12_phase-7_suggestion-creation-plan.md)

**Error Handling:**
- 401 Unauthorized → Refresh token and retry
- 403 Forbidden → User revoked access, notify admin to reauthorize
- 429 Too Many Requests → Backoff and retry (unlikely with MVP volume)
- 500 Server Error → Log error, show friendly message to user
- Network timeout → Retry 3x with exponential backoff

---

## Technology Stack

### Backend
- **Runtime:** Node.js 18+
- **Framework:** Express 4.18+
- **Google API Client:** `googleapis` npm package
- **Environment:** dotenv for config

### Frontend
- **HTML5** + **CSS3** + **Vanilla JavaScript**
- No framework (keeps it simple)

### Data Storage
- **JSON files** (MVP only)
- Future: PostgreSQL for production

### Development Tools
- **nodemon** for auto-restart
- **Postman** for API testing (optional)

### Dependencies
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "googleapis": "^118.0.0",
    "dotenv": "^16.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

---

## File Structure

```
mckay/
├── server/
│   ├── index.js                    # Express app entry point
│   ├── routes/
│   │   ├── auth.js                 # OAuth routes (Phase 1)
│   │   ├── calendar.js             # Calendar debug routes (Phase 2-3)
│   │   ├── preferences.js          # Preferences routes (Phase 4)
│   │   └── matching.js             # Matching routes (Phase 6-7)
│   ├── services/
│   │   ├── auth.js                 # OAuth service (Phase 1)
│   │   ├── calendar.js             # Google Calendar API (Phase 2-3, 7)
│   │   ├── preferences.js          # Preferences service (Phase 4)
│   │   ├── activities.js           # Activity database (Phase 5)
│   │   └── matching.js             # Matching engine (Phase 6)
│   ├── config/
│   │   └── google-credentials.json # OAuth client credentials (gitignored)
│   └── data/
│       ├── users.json              # OAuth tokens (gitignored)
│       ├── preferences.json        # User preferences
│       ├── activities.json         # Activity database (20-30 activities)
│       └── suggestions.json        # Suggestion history (audit log)
├── public/
│   ├── index.html                  # Landing page (Phase 0)
│   ├── preferences.html            # Preferences form (Phase 4)
│   ├── admin.html                  # Admin dashboard (Phase 8)
│   └── styles.css                  # Basic styling (all phases)
├── .env                            # Environment variables (gitignored)
├── .gitignore                      # Git ignore rules
├── package.json                    # Dependencies
└── README.md                       # Setup instructions
```

**Implementation Order:** See [High-Level Implementation Plan](../ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md)

**Environment Variables (.env):**
```
# Google OAuth Configuration
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:3000/auth/callback

# Server Configuration
PORT=3000
NODE_ENV=development

# Couple Configuration (for MVP)
DEFAULT_COUPLE_ID=couple-1
```

**.gitignore Configuration:**
```
# Environment
.env
.env.local
.env.*.local

# Dependencies
node_modules/

# Sensitive data
server/config/google-credentials.json
server/data/users.json

# Logs
*.log
npm-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo
```

**Files in Git vs Gitignored:**

✅ **Commit to Git:**
- `server/data/preferences.json` (no sensitive data, can start empty)
- `server/data/activities.json` (curated data, needed for matching)
- `server/data/suggestions.json` (audit log, can start empty)
- All `.js` files (code)
- All `.html` and `.css` files (UI)
- `package.json` (dependencies list)
- `.gitignore` (ignore rules)
- `README.md` (documentation)

❌ **Never Commit (in .gitignore):**
- `.env` (contains API secrets)
- `server/config/google-credentials.json` (OAuth client secret)
- `server/data/users.json` (contains OAuth access tokens)
- `node_modules/` (installed dependencies)

**Initial File Contents:**

- `server/data/users.json` - Create manually with empty couple-1 structure (all fields null), add to .gitignore
- `server/data/preferences.json` - Empty object `{}`
- `server/data/suggestions.json` - Empty array `[]`
- `server/data/activities.json` - Empty array `[]` (populate in Phase 5)

---

## Security Architecture

### Authentication
- **OAuth 2.0** for Google Calendar access
- **No password storage** (delegated to Google)
- **Token storage:** In JSON files (encrypted in production)

### Authorization
- **Scope limitation:** Only request `calendar.events` (minimal necessary access)
- **Token refresh:** Automatic refresh when expired
- **Revocation:** Users can revoke via Google account settings

### Data Protection
- **Sensitive files in .gitignore:**
  - `.env` (API keys)
  - `google-credentials.json` (OAuth client secret)
  - `data/users.json` (access tokens)
- **HTTPS required** in production
- **No PII stored** beyond calendar access

### API Security
- **Rate limiting:** Handled by Google (10 QPS)
- **Error handling:** Don't expose internal errors to user
- **Input validation:** Sanitize user inputs (preferences form)

---

## Deployment Architecture

### MVP (Local Development)
```
Local Machine
├── Node.js server (localhost:3000)
├── JSON files (local filesystem)
└── Google OAuth (redirect to localhost)
```

**Pros:** Fast, no deployment complexity
**Cons:** Only accessible locally

### Optional: Production Deployment

**Platform:** Railway / Render / Vercel

**Architecture:**
```
Internet
  ↓
Railway (Cloud)
  ├── Node.js server (public URL)
  ├── Persistent volume (JSON files)
  └── Environment variables (.env)
  ↓
Google Calendar API
```

**Changes needed:**
- Update OAuth redirect URI to production URL
- Use persistent storage for JSON files
- Set environment variables in platform
- Enable HTTPS

**Not required for class project demo**

---

## Performance Considerations

### Latency
- **OAuth flow:** ~2-3 seconds (Google redirect)
- **Calendar read:** ~500ms per calendar (2 API calls)
- **Free time calculation:** <100ms (in-memory)
- **Activity matching:** <50ms (local data)
- **Calendar write:** ~500ms per event (2 API calls)
- **Total:** ~3-4 seconds for full matching cycle

### Scalability (Not a concern for MVP)
- **Max couples:** ~10-20 for MVP
- **API quota:** 1M requests/day (more than enough)
- **JSON file size:** <1MB total (fine for MVP)

### Optimization (Future)
- Cache calendar events (refresh every 15 min)
- Batch calendar reads (use `freebusy` API)
- Move to database for >50 couples

---

## Error Handling Strategy

**General Principles:**
- Log all errors to console with context (function name, parameters, stack trace)
- Show user-friendly messages (never expose internal errors)
- Fail gracefully (return empty results rather than crashing)
- Retry transient failures (network, API rate limits)
- Don't retry permanent failures (invalid credentials, bad data)

### OAuth Errors

**User denies permission:** In /auth/callback route, check for error=access_denied in query params. Show "Authorization Failed" message with link to try again.

**Token expired (401 from API):** Implement auto-refresh pattern: try API call → catch 401 → refresh token → retry once. Never retry more than once (infinite loop risk).

**Refresh token invalid or expired:** If refresh fails with 'invalid_grant', return error indicating user must reauthorize with requiresReauth flag and partnerId.

**Only one partner authenticated:** In matching engine, check both partner tokens exist. Throw error if either is missing: "Both partners must be authenticated."

### API Errors

**Network failure:** Implement retryWithBackoff function with exponential backoff: 1s, 2s, 4s delays. Max 3 retries, then throw error.

**Rate limit (429 Too Many Requests):** Unlikely with MVP volume. If encountered, wait 5 seconds and retry once.

**Google API errors:** Map error codes to user-friendly messages:
- 401: "Authentication expired"
- 403: "Calendar access revoked"
- 404: "Calendar not found"
- 429: "Too many requests, try again"
- 500+: "Google Calendar temporarily unavailable"
- Default: "Error accessing calendar"

### Data Errors

**Missing preferences:** In matching engine, check if preferences exist. Throw error if null: "Preferences not set for this couple. Please complete the preferences form first."

**Missing or corrupt JSON file:** Implement loadJSONFile pattern with error handling:
- ENOENT (file doesn't exist): Create file with default value, return default
- SyntaxError (corrupt JSON): Log error, optionally backup corrupt file, return default value
- Other errors: Rethrow

**activities.json missing:** In activities service, verify file exists and contains valid array with length > 0. If not, throw: "Activity database not available. Please contact admin."

### Matching Engine Errors

**No free time found:** Return success with empty results and message: "No overlapping free time found in the next 2 weeks. Try adding more free time to your calendars." This is not an error - just empty results.

**No matching activities:** Return success with empty matchedActivities and message: "No activities match your preferences. Try broadening your interests or want-to-try list."

**Calendar event creation fails:** Wrap each suggestion creation in try-catch. Log error but continue with next suggestion. Don't fail entire process if one event creation fails.

### User-Facing Error Messages

**Setup Issues:**
- "Please connect your calendar first" → Link to /auth/google?partner=1
- "Partner 2 calendar not connected" → Link to /auth/google?partner=2
- "Please set your preferences first" → Link to /preferences/couple-1

**Matching Issues:**
- "No overlapping free time in the next 2 weeks"
- "No activities match your interests. Try broadening your preferences."
- "Calendar access expired. Please reconnect your calendar."

**Technical Issues:**
- "Failed to read calendar. Please try again."
- "Google Calendar is temporarily unavailable. Please try again later."
- "An unexpected error occurred. Please contact support."

### Admin Dashboard Error Display

**Show errors in results:** In admin.html, check response.ok. If false, display error-box with error message from data.error. If data.requiresReauth is true, include reconnect link for appropriate partner.

**Activity log for debugging:** Log all key events with timestamps. Types: Success ("Generated 2 suggestions"), Info ("Error: No free time found"), Warning ("Calendar API failed - retrying"), Error ("CRITICAL: Both calendar event creation failed").

### Error Prevention

**Input validation:** In preferences route, validate required fields:
- At least one interest required (check array length > 0)
- Location address required (check exists and not empty)
- Return 400 status with clear error message if validation fails

**Defensive coding:**
- Use optional chaining for safe property access: `partner1Tokens?.email || 'Unknown'`
- Provide defaults for missing values: `window.durationMinutes || 0`
- Type check critical values: if minDuration not a number, default to 120

**Logging for debugging:** Log key decision points with context:
- "Starting matching for couple: [coupleId]"
- "Found free windows: [count]"
- "Matched activities: [count]"
- "Creating suggestions: [count]"

---

## Testing Strategy

### Manual Testing (MVP)
**Implementation:** See [Phase 9 Plan](../ai/roadmaps/2026-02-12_phase-9_integration-testing-plan.md) for comprehensive testing checklist

**Critical Path (Must Work):**
- OAuth flow completes for both partners
- Preferences save and load correctly
- Calendar events readable from Google API
- Free time windows detected and filtered correctly
- Activities matched and scored
- Suggestions created in both calendars as tentative events
- Admin dashboard shows results and calendar links work

**Important (Should Work):**
- Token refresh automatic on expiry
- Weekend filtering correct (Fri 6pm - Sun 11pm)
- Duration filtering correct (min 2 hours)
- Advance notice filtering (3+ days)
- Match scoring reasonable and explainable
- Error messages user-friendly
- Calendar links open correct events

**Nice to Have (Good if Works):**
- All-day events handled (converted to 9am-5pm)
- Recurring events expanded correctly
- Mobile view acceptable
- Fast performance (<5 seconds for matching)
- Pretty UI styling

### Test Scenarios

**1. Happy Path:**
- Both calendars connected via OAuth
- Preferences set with interests and want-to-try list
- Free time exists on weekend (Saturday afternoon)
- Activities match interests and keywords
- Suggestions created successfully in both calendars
- Events show as tentative with full description

**2. No Free Time:**
- Both calendars fully booked for next 2 weeks
- System returns friendly message: "No overlapping free time found"
- No errors, graceful handling

**3. No Matching Activities:**
- Free time exists
- But no activities match interests/keywords/location/budget
- System returns: "No activities match your preferences"
- No errors, graceful handling

**4. Token Expiry:**
- Manually set token expiry to past
- Trigger calendar read
- System auto-refreshes token
- Retries API call successfully
- No user intervention needed

**5. Partial Setup:**
- Only Partner 1 authenticated
- Admin dashboard shows "Not ready" status
- Generate button disabled
- Clear message to complete setup

**6. Edge Cases:**
- Empty calendars (both partners)
- One empty, one full calendar
- Free time only on weekdays (filtered out)
- Free windows too short (<2 hours, filtered out)
- Free time within 3-day window (filtered out)
- All-day events in calendar
- Recurring events (weekly meetings)
- Cancelled events (should be ignored)

### Test Data Setup

**Test Accounts:**
- Partner 1: Create dedicated test Gmail (e.g., testpartner1@gmail.com)
- Partner 2: Create second test Gmail (e.g., testpartner2@gmail.com)

**Sample Calendar Events:**

Partner 1:
- Monday-Friday: Work meetings (9am-5pm)
- Wednesday evening: Gym (6pm-7pm)
- Thursday evening: Book club (7pm-9pm)
- Saturday morning: Dentist (10am-11am)
- **Free: Saturday afternoon (2pm-6pm)**
- **Free: Sunday morning (10am-2pm)**

Partner 2:
- Monday-Friday: Work (9am-6pm)
- Tuesday evening: Soccer (6pm-8pm)
- Friday evening: Dinner with parents (6pm-9pm)
- Saturday morning: Errands (9am-12pm)
- **Free: Saturday afternoon (2pm-7pm)**
- Sunday: All day busy

**Overlapping Free Time:**
- Saturday 2pm-6pm (4 hours) ✓

**Test Preferences:**
- Interests: Dining, Arts, Classes
- Want to try: "pottery class", "Italian restaurant", "wine tasting"
- Location: Austin, TX (or your target city)
- Min duration: 120 minutes
- Advance notice: 3 days
- Budget: $$$

**Test Activities Database:**
- Must include activity matching "pottery class" (arts category, pottery keyword)
- Should include Italian restaurant and wine tasting activities
- Mix of other activities across all categories
- All with realistic lat/lng for target city

### Browser Compatibility Testing

**Test in:**
- Chrome (primary browser for demo)
- Safari (if on Mac)
- Firefox (optional)
- Mobile browser (optional but recommended)

**Verify:**
- Forms submit correctly
- AJAX requests work
- Results display properly
- No JavaScript console errors
- OAuth redirects work

### Performance Testing

**Expected Latency:**
- OAuth flow: ~2-3 seconds (Google redirect)
- Calendar read (both): ~1 second (2 API calls)
- Free time calculation: <100ms (in-memory)
- Activity matching: <50ms (local data)
- Calendar write (both): ~1 second (2 API calls)
- **Total end-to-end: ~4-5 seconds**

### Data Integrity Testing

**Verify:**
- users.json saves correctly after OAuth
- preferences.json saves correctly after form submit
- suggestions.json appends (doesn't overwrite) on each suggestion
- Files persist after server restart
- JSON structure valid (no corruption)
- Sensitive files in .gitignore (users.json, .env, google-credentials.json)

---

## Monitoring & Logging

### MVP Logging
- **Console.log** for development
- Log key events:
  - OAuth success/failure
  - Calendar API calls
  - Matching results
  - Errors

### Future: Production Logging
- Structured logs (JSON format)
- Log levels (debug, info, warn, error)
- External logging service (Loggly, Papertrail)

---

## API Rate Limits & Quotas

### Google Calendar API
- **Quota:** 1,000,000 requests/day per project
- **Per-user limit:** 10 queries per second
- **Our usage:** ~10 requests per couple per day

**Calculation:**
- 2 calendar reads (events.list) = 2 requests
- 2 calendar writes (events.insert) = 2 requests
- Total per suggestion cycle = 4 requests
- 1 cycle per couple per day = 4 requests/couple/day
- 100 couples = 400 requests/day
- **Well within limits**

---

## Future Enhancements (Post-MVP)

### Database Migration
- Move from JSON to PostgreSQL
- Schema for users, couples, preferences, activities, suggestions
- Enable multi-tenancy

### Real-Time Event Data
- Integrate paid event API (PredictHQ, SerpApi)
- Or web scraper for local event sites
- Update activities.json automatically

### Advanced Features
- Feedback collection (thumbs up/down)
- Learning algorithm (improve matches over time)
- Email notifications
- Mobile-responsive UI
- Scheduled jobs (cron) for automatic suggestions

### Scalability
- Redis for caching calendar data
- Queue system for batch processing
- Horizontal scaling (multiple server instances)

---

## Implementation Gotchas & Best Practices

**Critical Issues to Avoid:**

### OAuth & Authentication

**OAuth Redirect URI Mismatch:**
- MUST be exactly: `http://localhost:3000/auth/callback`
- No trailing slash, HTTP not HTTPS for localhost
- Must match Google Cloud Console configuration exactly
- Common error: "redirect_uri_mismatch"

**Missing Refresh Token:**
- MUST use `access_type: 'offline'` in auth URL
- MUST use `prompt: 'consent'` to force consent screen
- Without this, refresh_token will be undefined
- User may need to revoke access and reauthorize

**State Parameter:**
- Use state parameter to track which partner is authenticating
- Format: `JSON.stringify({ partnerId: 'partner1', coupleId: 'couple-1' })`
- Parse on callback to identify partner
- Prevents CSRF attacks

### Calendar API

**Recurring Events:**
- MUST use `singleEvents: true` in API parameters
- This expands recurring events into individual instances
- Without this, weekly meetings only show once

**All-Day Events:**
- Check for `start.date` vs `start.dateTime`
- All-day events use `date` field (just date, no time)
- Convert to timed event (9am-5pm) for busy block extraction

**Token Refresh:**
- Access tokens expire in ~1 hour
- MUST implement auto-refresh on 401 errors
- Pattern: try API call → catch 401 → refresh → retry once
- Never retry more than once (infinite loop risk)

**Time Zones:**
- Keep all times in ISO 8601 format with timezone offset
- Don't convert to local time in calculations
- JavaScript Date handles ISO 8601 correctly
- Use same timezone for event creation

### Free Time Detection

**Weekend Definition:**
- Friday: 18:00 (6pm) to 23:59
- Saturday: 00:00 to 23:59 (all day)
- Sunday: 00:00 to 23:00 (11pm, not midnight)
- Use `date.getDay()` where 0=Sunday, 5=Friday, 6=Saturday

**Minimum Duration Filter:**
- Apply AFTER finding overlapping windows
- Duration in minutes (default 120)
- Calculate: `(end - start) / (1000 * 60)` for milliseconds to minutes

**Advance Notice Filter:**
- Filter out windows starting within N days from now
- Calculate cutoff: `new Date() + advanceNotice days`
- Compare window.start >= cutoff

### Matching Engine

**Activity Duration Matching:**
- Activity must fit within time window
- Don't match activity longer than window
- Calculate remaining time after activity ends

**Score Interpretation:**
- Typical good match: 15+ points
- Category match: +10 (most important)
- Keyword matches: +5 each (can add up quickly)
- Location: +1 to +3 (tiebreaker)
- Budget: +2 (preference, not requirement)

**Keyword Matching:**
- Use fuzzy matching (case-insensitive, substring)
- Check if keyword in want-to-try OR want-to-try in keyword
- Example: "pottery" matches "pottery class" and vice versa

### Calendar Event Creation

**Event Format:**
- Title: MUST prefix with "Suggested: " for identification
- Transparency: MUST be "tentative" (not "busy")
- Description: Include match reason, details, instructions
- Location: Use full address for map integration

**Event Description Template:**

Format: 🎯 Suggested Activity → What: [description] → Why: [match reasons] → Details: (duration, price, location) → More info: [url] → Instructions: (keep to accept, delete to decline) → Footer: "Generated by Calendar Concierge"

**Error Handling:**
- If one partner's event creation fails, continue with other
- Log error but don't fail entire suggestion process
- Suggestion is still useful if in one calendar

### File Operations

**JSON File Handling:**
- Always use `JSON.parse()` with try-catch
- Handle `ENOENT` error (file doesn't exist) separately
- Create file with defaults if missing
- Use `JSON.stringify(data, null, 2)` for readable format

**users.json Security:**
- MUST be in .gitignore
- Contains OAuth access tokens (sensitive)
- In production, encrypt or use database
- Never commit to Git

**File Paths:**
- Use `path.join(__dirname, '../data/users.json')` not hardcoded paths
- Works across operating systems
- Relative to current file location

### Admin Dashboard

**Status Checking:**
- Check BOTH partners authenticated before enabling generate button
- Check preferences exist before enabling
- Show clear status messages for each requirement

**Results Display:**
- Show friendly messages for empty results (not errors)
- "No free time found" is success with empty array, not error
- Link directly to calendar events (use `htmlLink` from API response)

**Activity Log:**
- Log key events for debugging
- Include timestamps
- Newest entries first (prepend to log)

### Common Debugging Tips

**OAuth Issues:**
- Check Google Cloud Console: OAuth credentials configured correctly
- Check .env: GOOGLE_CLIENT_ID and GOOGLE_CLIENT_SECRET set
- Check redirect URI: matches exactly in code and console
- Check test users: added to OAuth consent screen

**Calendar Reading Issues:**
- Check tokens: valid and not expired
- Check API response: log full response for debugging
- Check events: use /calendar/:coupleId/events debug endpoint
- Check timezone: events have timezone offset

**Free Time Issues:**
- Log busy blocks: verify extraction correct
- Log free windows before filtering: ensure finding gaps
- Log after each filter: see which filter removes windows
- Check weekend logic: verify day of week calculation

**Matching Issues:**
- Log activities loaded: verify file read successful
- Log filtered activities: see what passes each filter
- Log scores: verify scoring algorithm
- Log match reasons: ensure human-readable

**Testing:**
- Use two real Google accounts for testing
- Add real calendar events (varied types)
- Use realistic test data (real addresses, activities)
- Test on same computer/network as demo
- Take screenshots of working flow as backup

---

## Key Design Decisions

### Why JSON Files?
- ✅ Simple for MVP
- ✅ No database setup required
- ✅ Easy to inspect/debug
- ✅ Fast for small datasets (<100 couples)
- ⚠️ Not suitable for production scale

### Why Hardcoded Activities?
- ✅ Eventbrite API deprecated
- ✅ Reliable for demo (no API failures)
- ✅ High-quality curation
- ✅ No API keys or rate limits
- ⚠️ Manual maintenance required

### Why Manual Triggering?
- ✅ Simpler than cron jobs
- ✅ Good for demo (on-demand)
- ✅ Easier to debug
- ⚠️ Not scalable (need automation later)

### Why Node.js?
- ✅ Best Google API support (`googleapis` package)
- ✅ Fast to prototype
- ✅ Single language (JS for front + back)
- ✅ Large community, good docs

---

## Implementation Phases

This architecture is implemented across 10 phases. See the roadmap documents for detailed implementation guidance:

**Foundation & Authentication:**
- [Phase 0: Foundation](../ai/roadmaps/2026-02-12_phase-0_foundation-plan.md) - Project setup, Google Cloud config, basic server
- [Phase 1: OAuth Integration](../ai/roadmaps/2026-02-12_phase-1_oauth-integration-plan.md) - Google OAuth 2.0, token management

**Calendar Integration:**
- [Phase 2: Calendar Reading](../ai/roadmaps/2026-02-12_phase-2_calendar-reading-plan.md) - Read events, extract busy blocks
- [Phase 3: Free Time Detection](../ai/roadmaps/2026-02-12_phase-3_free-time-detection-plan.md) - Find overlapping free windows, weekend filtering

**Preferences & Activities:**
- [Phase 4: Preferences Collection](../ai/roadmaps/2026-02-12_phase-4_preferences-collection-plan.md) - Web form, preferences storage
- [Phase 5: Activity Database](../ai/roadmaps/2026-02-12_phase-5_activity-database-plan.md) - Curated activities, search/filter

**Matching & Suggestions:**
- [Phase 6: Matching Engine](../ai/roadmaps/2026-02-12_phase-6_matching-engine-plan.md) - Scoring algorithm, activity-to-time matching
- [Phase 7: Suggestion Creation](../ai/roadmaps/2026-02-12_phase-7_suggestion-creation-plan.md) - Create calendar events, logging

**UI & Testing:**
- [Phase 8: Admin Dashboard](../ai/roadmaps/2026-02-12_phase-8_admin-dashboard-plan.md) - Manual trigger UI, results display
- [Phase 9: Integration & Testing](../ai/roadmaps/2026-02-12_phase-9_integration-testing-plan.md) - End-to-end testing, bug fixes
- [Phase 10: Demo Preparation](../ai/roadmaps/2026-02-12_phase-10_demo-preparation-plan.md) - Demo script, test data, backup materials

**Timeline:** 6-8 weeks (Phases 0-9), plus demo prep week

**High-Level Plan:** [2026-02-12_high-level_mvp-implementation-plan.md](../ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md)

---

## References

**External APIs:**
- **Google Calendar API:** [developers.google.com/workspace/calendar](https://developers.google.com/workspace/calendar/api/v3/reference)
- **OAuth 2.0:** [developers.google.com/identity/protocols/oauth2](https://developers.google.com/identity/protocols/oauth2)
- **Node.js googleapis:** [googleapis.dev/nodejs/googleapis](https://googleapis.dev/nodejs/googleapis/latest/calendar/classes/Calendar.html)

**Project Documentation:**
- **PRD:** [aiDocs/prd.md](./prd.md)
- **MVP Spec:** [aiDocs/mvp.md](./mvp.md)
- **Context:** [aiDocs/context.md](./context.md)

**Implementation Guides:**
- **Phase Plans:** [ai/roadmaps/](../ai/roadmaps/)
- **API Guides:** [ai/guides/](../ai/guides/)
