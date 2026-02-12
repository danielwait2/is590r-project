# Implementation Roadmap
**Created:** 2026-02-12
**Project:** Calendar-Native Weekend Experience Concierge MVP
**Timeline:** 6-8 weeks to demo-ready
**Goal:** Build working prototype that demonstrates core value proposition

---

## Project Principles

⚠️ **IMPORTANT: This is a clean-slate MVP build**

**DO:**
- Build the simplest thing that works
- Focus on core demo flow
- Use modern patterns and APIs
- Keep dependencies minimal

**DON'T:**
- Add backwards compatibility layers
- Build abstractions "for future flexibility"
- Include features "just in case"
- Copy/paste legacy code patterns
- Add error handling for impossible scenarios

**We're building for a demo, not production scale. Speed and clarity over robustness.**

---

## Success Criteria

**Minimum viable demo:**
1. Two test Google accounts can authenticate via OAuth
2. System reads events from both calendars
3. System identifies overlapping free time
4. System matches activities to interests
5. System creates suggestions in both Google Calendars
6. Suggestions appear correctly in Google Calendar web/mobile

**Demo presentation:**
- 5-10 minute live walkthrough
- Shows problem → solution → result
- No major bugs or crashes
- Backup plan if live demo fails (video/screenshots)

---

## Phase Breakdown

### Phase 0: Foundation (Week 1)
**Goal:** Environment setup and project scaffolding

**Deliverables:**
- Google Cloud project created
- OAuth 2.0 credentials configured (redirect URI: http://localhost:3000/auth/callback)
- Google Calendar API enabled in Cloud Console
- Node.js project initialized with package.json
- Basic Express server running (port 3000)
- Project structure in place (per architecture.md file structure)
- Dependencies installed: express, googleapis, dotenv
- .env file configured with Google credentials
- .gitignore configured (exclude .env, google-credentials.json, data/users.json)

**Blockers:**
- Need Google account with Cloud Console access
- May need credit card for Google Cloud (usually not required for Calendar API)
- Need to add test users to OAuth consent screen

**Success check:**
- Can run `npm start` and see server on localhost:3000
- Can access Google Cloud Console and see project
- Environment variables load correctly from .env
- File structure matches architecture.md specification

---

### Phase 1: OAuth Integration (Weeks 1-2)
**Goal:** Get Google Calendar authentication working

**Deliverables:**
- OAuth service (server/services/auth.js)
- OAuth routes (GET /auth/google?partner=1 or 2, GET /auth/callback)
- OAuth initiation with correct scopes: `https://www.googleapis.com/auth/calendar.events`
- OAuth callback handler (exchange code for tokens)
- Token storage (save to data/users.json with structure: couple-1 → partner1/partner2)
- Token refresh logic (handle expiration using refresh token)
- Basic UI to trigger OAuth flow (simple HTML buttons)

**Key challenges:**
- OAuth redirect URIs configured correctly in Google Cloud Console
- Token storage format matches architecture.md data structure
- Refresh token handling (store both access_token and refresh_token)
- Linking both partners to same couple-id

**Success check:**
- Can click "Connect Partner 1 Calendar" → Google consent screen → redirected back with token
- Can repeat for Partner 2 (separate Google account)
- Tokens visible in data/users.json with correct structure
- Tokens work (can make test API call to list calendar events)
- Token refresh works when access token expires

**This is the hardest part. Budget extra time.**

**API Reference:** Google Calendar API v3, googleapis npm package OAuth2 client

---

### Phase 2: Calendar Reading (Week 3)
**Goal:** Read calendar events via Google Calendar API

**Deliverables:**
- Calendar service module (server/services/calendar.js)
- Function to list events for date range (GET /calendars/primary/events endpoint)
- Function to parse event start/end times
- Function to identify busy blocks from events
- Test endpoint to view calendar events (for debugging)

**Key challenges:**
- Handling token refresh on API calls (401 → refresh → retry)
- Parsing different event formats (all-day vs timed events)
- Timezone handling (ISO 8601 format)
- API parameters: timeMin, timeMax, singleEvents=true (expand recurring)

**Success check:**
- Can call API and get list of events from test calendar (next 2 weeks)
- Events display correctly (title, start time, end time)
- Works for both partners
- Busy blocks extracted correctly for free time calculation
- Token refresh handled automatically on expiration

**API Reference:** Google Calendar API v3 - GET /calendars/primary/events
**Rate Limit:** 10 QPS per user (plenty for MVP)

---

### Phase 3: Free Time Detection (Week 3-4)
**Goal:** Calculate overlapping free time between calendars

**Deliverables:**
- Algorithm to find busy blocks (extract from calendar events)
- Algorithm to find free windows (gaps in timeline)
- Algorithm to find overlapping free time between both partners
- Filter by preferences (weekends only: Fri 6pm-Sun 11pm, min 2hr duration, 3+ days advance notice)
- Test endpoint to show free time (for debugging)

**Key challenges:**
- Overlapping time logic (date/time math, ISO 8601 handling)
- Handling edge cases (all-day events, midnight boundaries, recurring events)
- Filtering logic (day of week, time windows, duration checks)
- Timezone consistency

**Success check:**
- Given two calendars with events, correctly identifies free windows
- Finds overlapping windows where BOTH partners are free
- Filters work correctly:
  - Only shows weekend windows (Friday evening, Saturday, Sunday)
  - Respects minimum 2-hour duration
  - Only shows windows 3+ days in advance
- Returns empty array if no free time (doesn't crash)
- Edge cases handled (all-day events marked as busy)

**Per MVP spec:** Weekend-only scope (Friday 6pm - Sunday 11pm), 2-hour minimum, 3-day advance notice

---

### Phase 4: Preferences Collection (Week 4)
**Goal:** Capture user preferences for matching

**Deliverables:**
- Preferences HTML form (public/preferences.html)
- Routes: GET /preferences/:coupleId (display form), POST /preferences/:coupleId (save)
- Form fields per architecture.md:
  - Interests: checkboxes (dining, arts, music, classes, outdoors, sports, etc.)
  - Want to try: text area (free-form list)
  - Location: address input (with geocoding for lat/lng)
  - Availability: day/time window selection, min duration (120 min default), advance notice (3 days default)
  - Budget: price level selector ($, $$, $$$, $$$$)
- Preferences stored in data/preferences.json with structure matching architecture.md
- Data validation (basic: required fields, sensible defaults)

**Key challenges:**
- Form design (keep it simple but capture all needed data)
- Geocoding location to lat/lng for distance calculations
- Data validation (ensure all required fields present)
- Linking preferences to couple-id

**Success check:**
- Can fill out form and submit
- Data saved to data/preferences.json with correct structure
- Can retrieve and display saved preferences
- Location geocoded to lat/lng
- Preferences match data structure in architecture.md

---

### Phase 5: Activity Database (Week 5)
**Goal:** Create and load activity database

**Deliverables:**
- activities.json with 20-30 curated activities (hardcoded, not API)
- Activity loading service (server/services/activities.js)
- Activity search/filter functions
- Distance calculation (Haversine formula for location proximity)

**Key challenges:**
- Finding real businesses for target city
- Writing good descriptions
- Keyword tagging for matching
- Location data (lat/lng for distance calculation)

**Success check:**
- activities.json is populated with structured data
- Can load activities in code
- Can filter by category matching interests
- Can search by keyword (want-to-try list)
- Distance calculation works for location filtering

**This is manual work (research and data entry).**

**Note:** Per architecture.md, using hardcoded activities (not Eventbrite API which is deprecated). This provides reliable demo data and high-quality curation.

---

### Phase 6: Matching Engine (Week 5-6)
**Goal:** Core logic to match activities to free time

**Deliverables:**
- Matching service (server/services/matching.js)
- Main generateSuggestions(coupleId) function (orchestrates full flow)
- Activity scoring algorithm (category match +10, keyword match +5, location proximity +1-3, price match +2)
- Distance calculation using Haversine formula (location filtering)
- Activity-to-time-window matching (check duration fits window)
- Match reason generation (explain why suggested)
- Limit to 1-2 suggestions per run

**Key challenges:**
- Scoring logic (balance category vs keyword vs location per architecture.md algorithm)
- Avoiding duplicate suggestions (one activity per window)
- Generating human-readable match reasons
- Ensuring activity duration fits time window
- Distance calculation accuracy

**Success check:**
- Given preferences and free time, returns ranked activities
- Match reasons make sense ("Matches your interest in arts and your want-to-try item: pottery class")
- Picks appropriate time windows for activity duration (e.g., 3-hour activity → 3-hour window)
- Location filtering works (within max distance)
- Top 1-2 suggestions selected
- Score calculation follows architecture.md algorithm

**Algorithm per architecture.md:**
- Category match (interests): +10
- Keyword match (want-to-try): +5
- Location (<5mi: +3, <10mi: +2, <15mi: +1)
- Price match: +2

---

### Phase 7: Suggestion Creation (Week 6)
**Goal:** Insert suggestions into Google Calendars

**Deliverables:**
- Function to create calendar event (in calendar service)
- API call: POST /calendars/primary/events
- Event format per architecture.md:
  - Title: "Suggested: [Activity Name]"
  - Time: start/end from time window
  - Location: venue address
  - Description: what, why (match reason), details, link
  - transparency: "tentative" (shows as tentative in calendar)
- Create in both partners' calendars (2 API calls)
- Log suggestions to data/suggestions.json with full metadata
- Return calendar event IDs and links

**Key challenges:**
- Event formatting (tentative status, complete descriptions)
- Creating in multiple calendars (sequential API calls)
- Error handling (what if one succeeds, one fails? rollback or partial?)
- Match reason generation (clear, human-readable)
- ISO 8601 datetime formatting

**Success check:**
- Can create event via API (POST to Google Calendar)
- Event appears in Google Calendar web/app for both partners
- Event shows as tentative (not confirmed)
- Description includes match reason ("Based on your interest in arts") and activity details
- Works for both partners simultaneously
- Suggestion logged to data/suggestions.json with correct structure
- Calendar event IDs stored for tracking

**API Reference:** POST /calendars/primary/events - Google Calendar API v3
**Event structure matches architecture.md specification**

---

### Phase 8: Admin Dashboard (Week 7)
**Goal:** UI to trigger and monitor matching

**Deliverables:**
- Admin HTML page (public/admin.html per architecture.md)
- Route: GET /admin
- Display couple status (both calendars connected? preferences set?)
- "Generate Suggestions" button (triggers POST /matching/:coupleId)
- Results display:
  - Free time windows found (count + list)
  - Activities matched (count + list)
  - Suggestions created (count + details + calendar links)
- Activity log (recent actions with timestamps)
- Vanilla JavaScript (no framework needed per architecture.md)

**Key challenges:**
- Async UI updates (button click → API call → display results without page refresh)
- Error display (user-friendly messages)
- Making it look decent (minimal styling, basic CSS)
- Displaying structured results (JSON → readable format)

**Success check:**
- Can click button and trigger full matching flow
- Results display on page (no page refresh)
- Can see what happened (free windows, matched activities, suggestions created)
- Calendar links work (open Google Calendar)
- Errors displayed gracefully (not console.log)
- Activity log shows recent actions

**Per architecture.md:** Simple HTML/CSS/Vanilla JS, no React/Vue complexity

---

### Phase 9: Integration & Testing (Week 7)
**Goal:** End-to-end flow works reliably

**Deliverables:**
- Full flow: OAuth → Preferences → Match → Suggestions → Calendar
- Bug fixes
- Error handling for critical paths
- Test with real Google Calendar accounts

**Key challenges:**
- Finding and fixing edge cases
- Handling real-world calendar data
- Timezone issues

**Success check:**
- Complete flow works start to finish
- No crashes on happy path
- Errors are handled gracefully (don't crash, show message)

---

### Phase 10: Demo Preparation (Week 8)
**Goal:** Ready for presentation

**Deliverables:**
- Demo script (what to say, what to click)
- Test accounts pre-configured
- Test calendars have sample events
- Backup plan (screenshots, video recording)
- Presentation slides (optional)

**Key challenges:**
- Demo day nerves
- Internet connectivity on demo day
- Something breaking at the last minute

**Success check:**
- Can run demo 3 times without issues
- Demo takes 5-10 minutes
- Value proposition is clear to audience
- Have backup if live demo fails

---

## Critical Path

**Must complete in order:**
1. **OAuth (Weeks 1-2)** → Everything depends on this
2. **Calendar Reading (Week 3)** → Needed for free time detection
3. **Free Time Detection (Week 3-4)** → Needed for matching
4. **Matching Engine (Week 5-6)** → Core logic
5. **Suggestion Creation (Week 6)** → Final output

**Can parallelize:**
- Preferences form (Week 4) can be built anytime before matching
- Activity database (Week 5) can be built anytime before matching
- Admin dashboard (Week 7) can be started early, refined later

---

## Risk Areas & Mitigation

### High Risk
**OAuth complexity**
- Risk: Most developers struggle with OAuth first time
- Mitigation: Start early, follow official Google examples, ask for help
- Fallback: Use hardcoded tokens if really stuck (for demo only)

**Calendar API issues**
- Risk: Token expiry, API errors, unexpected data formats
- Mitigation: Build robust token refresh, test with varied calendar data
- Fallback: Mock API responses if needed for demo

**Demo day technical failure**
- Risk: Internet down, API rate limit, OAuth breaks
- Mitigation: Record successful demo run, take screenshots, have backup plan
- Fallback: Show recorded demo, explain what would happen live

### Medium Risk
**Timezone handling**
- Risk: Events show at wrong times, free time calculation off
- Mitigation: Test with events in multiple timezones, use moment.js or similar
- Fallback: Hardcode to one timezone for MVP

**Activity data quality**
- Risk: Not enough good activities, matching doesn't work well
- Mitigation: Start populating activities.json early, iterate on keywords
- Fallback: Fewer but higher-quality activities

**Scope creep**
- Risk: Adding features that aren't needed for demo
- Mitigation: Stick to roadmap, say "that's post-MVP" to new ideas
- Fallback: Cut features if behind schedule

### Low Risk
**UI polish**
- Risk: Looks ugly
- Mitigation: Spend 2-3 hours on basic CSS in Week 7
- Fallback: Ugly but functional is fine for class project

**Performance**
- Risk: Slow response times
- Mitigation: Profile if needed, but unlikely to matter for 1 couple
- Fallback: Acceptable for demo scale

---

## Weekly Milestones

**Week 1:**
- ✅ Environment setup complete
- ✅ OAuth flow initiated (redirect working)

**Week 2:**
- ✅ OAuth complete (tokens stored)
- ✅ Can make test API call

**Week 3:**
- ✅ Can read calendar events
- ✅ Free time detection working

**Week 4:**
- ✅ Preferences form working
- ✅ Free time filter logic complete

**Week 5:**
- ✅ Activities database populated
- ✅ Matching algorithm implemented

**Week 6:**
- ✅ Can create suggestions in calendar
- ✅ Full matching flow works

**Week 7:**
- ✅ Admin dashboard functional
- ✅ End-to-end flow tested

**Week 8:**
- ✅ Demo script ready
- ✅ Backup plan prepared
- ✅ Demo rehearsed 3x

---

## Project File Structure (Per Architecture.md)

**This is the required structure - build to this spec:**

```
mckay/
├── server/
│   ├── index.js                    # Express app entry point
│   ├── routes/
│   │   ├── auth.js                 # OAuth routes
│   │   ├── preferences.js          # Preferences routes
│   │   └── matching.js             # Matching routes
│   ├── services/
│   │   ├── auth.js                 # OAuth service
│   │   ├── calendar.js             # Google Calendar API
│   │   ├── matching.js             # Matching engine
│   │   └── activities.js           # Activity database
│   ├── config/
│   │   └── google-credentials.json # OAuth client credentials (gitignored)
│   └── data/
│       ├── users.json              # OAuth tokens (gitignored)
│       ├── preferences.json        # User preferences
│       ├── activities.json         # Activity database
│       └── suggestions.json        # Suggestion history
├── public/
│   ├── index.html                  # Landing page
│   ├── preferences.html            # Preferences form
│   ├── admin.html                  # Admin dashboard
│   └── styles.css                  # Basic styling
├── .env                            # Environment variables (gitignored)
├── .gitignore                      # Git ignore rules
├── package.json                    # Dependencies
└── README.md                       # Setup instructions
```

**Key files to gitignore:**
- `.env`
- `server/config/google-credentials.json`
- `server/data/users.json` (contains OAuth tokens)

---

## Dependencies

**External:**
- Google Cloud Console access
- Two Google test accounts for OAuth testing
- Internet connectivity for API calls
- Google Cloud project with Calendar API enabled

**Skills:**
- Node.js / Express basics
- REST API concepts
- OAuth 2.0 understanding (will learn)
- JSON data manipulation
- Basic HTML/CSS/JS

**Tools:**
- Code editor (VS Code)
- Git for version control
- Postman or curl for API testing (optional)
- Chrome DevTools for debugging
- nodemon for development (auto-restart)

**Tech Stack (per architecture.md):**
- **Node.js 18+**
- **Express 4.18+**
- **googleapis npm package v118.0.0+** (Google Calendar API client)
- **dotenv** for environment config
- **JSON files** for data storage (MVP only - no database)

**npm dependencies:**
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

## What Success Looks Like

**Week 4 checkpoint:**
- OAuth works
- Can read calendars
- Free time detection algorithm written

**If on track:** Continue as planned
**If behind:** Cut preferences form complexity, use hardcoded preferences

**Week 6 checkpoint:**
- Matching engine works
- Can create calendar events

**If on track:** Add polish, build dashboard
**If behind:** Skip dashboard, trigger via API endpoint (Postman)

**Week 8 final:**
- Can demo full flow
- Audience understands value
- Technical skills demonstrated

**Minimum passing demo:**
- Show OAuth screen
- Show calendars with events
- Click button (or API call)
- Show suggestions appear in calendar

---

## Explicitly Out of Scope (Per MVP.md)

**DO NOT build these for MVP - they are explicitly excluded:**

### User Management (Not Needed)
- ❌ User signup/registration system
- ❌ Email/password login flows
- ❌ Password hashing/authentication
- ❌ Email verification
- ❌ Multiple couples support (just one test couple: couple-1)

### Automation (Not Needed)
- ❌ Scheduled jobs (cron/node-cron)
- ❌ Automatic weekly suggestions
- ❌ Email notifications
- ❌ Push notifications
- ❌ Real-time calendar monitoring/webhooks

### Advanced Features (Not Needed)
- ❌ Feedback collection (thumbs up/down)
- ❌ Learning algorithm (improve over time)
- ❌ Historical dashboard
- ❌ "Want to try" list management UI
- ❌ Edit preferences UI (re-enter if needed)
- ❌ Multiple suggestion batches
- ❌ Reschedule/cancel features

### External APIs (Not Used)
- ❌ Eventbrite API (deprecated, use hardcoded activities.json)
- ❌ Google Places API (use hardcoded location data)
- ❌ Paid event APIs

### Production Concerns (Not Needed)
- ❌ PostgreSQL database (use JSON files)
- ❌ Production hosting/deployment
- ❌ HTTPS/SSL certificates
- ❌ Rate limiting (handled by Google)
- ❌ Error logging services
- ❌ Analytics
- ❌ Performance optimization
- ❌ GDPR compliance
- ❌ Terms of Service / Privacy Policy

### UI Polish (Not Needed)
- ❌ Beautiful UI/UX design
- ❌ Mobile responsiveness
- ❌ React/Vue frameworks
- ❌ Loading spinners
- ❌ Extensive form validation
- ❌ Elaborate error messages

**Why this is okay:** This is a demo project, not a product. Goal is to prove the concept works. Can add polish later if pursuing further.

---

## Post-MVP Possibilities

**If project continues beyond class:**
1. Real event API integration (paid service like PredictHQ)
2. Automated scheduling (cron jobs for weekly suggestions)
3. Multi-couple support and user signup/login system
4. Feedback collection and learning algorithm
5. Email notifications
6. Mobile app
7. PostgreSQL migration for scale

**Don't build these now. Demo the MVP first.**

---

## Resources

**Project Documentation:**
- [architecture.md](../../aiDocs/architecture.md) - **Technical architecture, API specs, data structures** (AUTHORITATIVE)
- [mvp.md](../../aiDocs/mvp.md) - **MVP scope, features, what's in/out** (AUTHORITATIVE)
- [prd.md](../../aiDocs/prd.md) - **Product requirements, user stories, success metrics** (AUTHORITATIVE)
- [ai/guides/google-calendar-api.md](../guides/google-calendar-api.md) - API reference (if exists)

**Official Docs:**
- Google Calendar API v3: developers.google.com/workspace/calendar/api/v3/reference
- OAuth 2.0: developers.google.com/identity/protocols/oauth2
- Node.js googleapis: github.com/googleapis/google-api-nodejs-client
- googleapis npm: www.npmjs.com/package/googleapis

**Key API Endpoints:**
- GET /calendars/primary/events - List calendar events
- POST /calendars/primary/events - Create calendar event
- Scopes: https://www.googleapis.com/auth/calendar.events

**When stuck:**
- Check project docs first (architecture.md, mvp.md)
- Then check official Google docs
- Search GitHub issues for googleapis package
- Ask for help (don't spend 2+ hours stuck)

---

## Notes

**Remember:**
- This is a class project, not a startup
- Demo > polish
- Working > perfect
- Learning > shipping

**Avoid:**
- Over-engineering
- Premature optimization
- Feature creep
- Analysis paralysis

**Focus:**
- Does it demonstrate the concept?
- Does it show technical competence?
- Can I explain how it works?

---

**Next Steps:**
1. Review this roadmap
2. Create Week 1 detailed plan (environment setup + OAuth start)
3. Set up Google Cloud project
4. Begin Phase 1 (OAuth integration)

**Let's build this thing.**
