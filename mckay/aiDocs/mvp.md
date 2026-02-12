# MVP Technical Specification
## Demo-Ready Calendar Integration Prototype

> **Overview:** This document specifies the **simplified MVP** we're actually building for the class project. For the full product vision, see [prd.md](./prd.md). For technical architecture, see [architecture.md](./architecture.md). For implementation plan, see [ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md](../ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md).

**Goal:** Build the minimum viable system to demonstrate core functionality in a live demo
**Timeline:** 6-8 weeks
**Scope:** Automated MVP for 1 test couple (couple-1) - prove the concept works, not build a product
**Approach:** Node.js + Express backend with JSON file storage (no database for MVP)
**Users:** 1 test couple with 2 separate Google accounts

---

## Demo Scenario

**What we'll show:**
1. Two partners (can be test accounts) have connected their Google Calendars
2. They've set up their preferences (interests, availability, "want to try" list)
3. System finds overlapping free time in their calendars
4. System matches an activity to their preferences
5. System inserts a suggested event into both calendars
6. Partners can see it in their Google Calendar (accept/decline)

**Demo duration:** 5-10 minutes
**Audience:** Class presentation, potential users, stakeholders

---

## MVP Scope Clarifications

**This MVP is scoped for:**
- **1 test couple only** (couple-1 with partner1 and partner2)
- **JSON file storage** (no database - users.json, preferences.json, activities.json, suggestions.json)
- **Manual triggering** (admin dashboard button, not automated scheduling)
- **Local development** (localhost:3000, not cloud deployment)
- **Hardcoded activities** (20-30 activities in JSON, no external event APIs)
- **Demo focus** (prove concept works, not production-ready)

**Reference:** See [architecture.md](./architecture.md) for detailed technical specification and [ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md](../ai/roadmaps/2026-02-12_high-level_mvp-implementation-plan.md) for 10-phase implementation plan.

---

## Core Functional Requirements

### Must Have (Core Demo Flow)

#### 1. **Google Calendar OAuth Integration**
- Authenticate two test users via Google OAuth 2.0
- Request permissions:
  - `calendar.readonly` - read calendar events
  - `calendar.events` - write calendar events
- Store access tokens (can be in-memory for demo, doesn't need to persist)
- Handle token refresh

**Why it's essential:** This is the core technical challenge and product differentiator

#### 2. **Calendar Reading**
- Read calendar events from both partners for next 2 weeks
- Parse events to identify "busy" time blocks
- Calculate "free" time windows

**Why it's essential:** Need to prove we can detect free time

#### 3. **Free Time Detection**
- Find overlapping free time between both calendars
- Apply basic filters:
  - Weekends only (Friday 6pm - Sunday 11pm)
  - Minimum 2-hour blocks
  - At least 3 days in advance

**Why it's essential:** This is the automation value - no manual coordination needed

#### 4. **Preference Collection**
- Simple web form to input:
  - Interest categories (checkboxes: dining, arts, music, classes, outdoors)
  - "Want to try" list (text area - free form)
  - Location (single address input)
- Store preferences (JSON file or simple database is fine)

**Why it's essential:** Need to show personalization, not random suggestions

#### 5. **Activity Matching (Simplified)**
- **Option A (Simplest):** Hardcoded activity database
  - 20-30 pre-populated activities with categories, locations, descriptions
  - Simple keyword matching against interests
  - Filter by location proximity (basic)

- **Option B (More Impressive):** Live API integration
  - Call Eventbrite API with location + date range
  - Filter by matched interest categories
  - Parse results into standard format

**Why it's essential:** Need to show matching logic works

**Recommendation:** Start with Option A (hardcoded), add Option B if time allows

#### 6. **Suggestion Creation**
- Generate suggestion with:
  - Activity name
  - Time window (specific start/end time within free block)
  - Location
  - Description with "why this matched" explanation
- Insert as tentative event into both Google Calendars via API
- Event format:
  ```
  Title: "Suggested: [Activity Name]"
  Time: [Suggested window]
  Location: [Address]
  Description:
    - What: [Activity description]
    - Why: "Based on your interest in [category]"
    - Details: [Link or info]
    - How to respond: Keep = accept, Delete = decline
  ```

**Why it's essential:** This is the product - suggestions in calendar

#### 7. **Admin Dashboard (Manual Trigger)**
- Simple web page showing:
  - List of couples (or just one test couple)
  - Button: "Run Matching for [Couple Name]"
  - Display results:
    - Found free time windows
    - Matched activities
    - Suggestion created (link to Google Calendar)
  - Activity log (what happened when)

**Why it's essential:** Need a way to trigger the demo and show what happened

---

## Technical Architecture

### Stack Recommendation

**Backend:**
- **Node.js + Express** (recommended)
  - Excellent Google API client library (`googleapis` npm package)
  - Fast to prototype
  - Good OAuth support
- Alternative: Python + Flask (also solid Google API support)

**Frontend:**
- **Simple HTML/CSS/JavaScript**
  - No framework needed for MVP
  - Forms for preferences
  - Admin dashboard with vanilla JS
- Alternative: React if you're already comfortable (but adds complexity)

**Data Storage:**
- **JSON files** (simplest - fine for demo)
  - `users.json` - user OAuth tokens
  - `preferences.json` - couple preferences
  - `activities.json` - activity database (if using hardcoded)
  - `suggestions.json` - suggestion history
- Alternative: SQLite (if you want simple database practice)

**Hosting:**
- **Local development** (run on localhost for demo)
- **Optional:** Deploy to Railway/Render for remote demo access

**No need for:**
- ❌ Production database (PostgreSQL)
- ❌ User authentication system (email/password)
- ❌ Password hashing/security (it's a demo)
- ❌ Session management
- ❌ Email notifications
- ❌ Production hosting/scaling

---

## System Components

### 1. OAuth Service (`/auth`)

**Endpoints:**
- `GET /auth/google` - Initiate Google OAuth flow
- `GET /auth/google/callback` - Handle OAuth callback
- `GET /auth/status` - Check if users are authenticated

**Functions:**
```javascript
// Pseudo-code
initiateOAuthFlow(userId) {
  // Redirect to Google OAuth consent screen
  // Request calendar permissions
}

handleOAuthCallback(code) {
  // Exchange code for access token
  // Store token in users.json
  // Link to couple if both partners authenticated
}
```

### 2. Calendar Service (`/calendar`)

**Functions:**
```javascript
getCalendarEvents(accessToken, startDate, endDate) {
  // Call Google Calendar API
  // Return list of events with start/end times
}

findFreeTimeWindows(calendar1Events, calendar2Events) {
  // Calculate busy blocks for both calendars
  // Find overlapping free windows
  // Filter by: weekends, minimum duration, advance notice
  // Return array of free time slots
}

createCalendarEvent(accessToken1, accessToken2, eventDetails) {
  // Create event in partner 1's calendar
  // Create event in partner 2's calendar
  // Return event IDs
}
```

### 3. Preferences Service (`/preferences`)

**Endpoints:**
- `GET /preferences/:coupleId` - Get couple preferences
- `POST /preferences/:coupleId` - Save/update preferences

**Data structure:**
```json
{
  "coupleId": "couple-1",
  "interests": ["dining", "arts", "classes"],
  "wantToTry": [
    "pottery class",
    "Italian restaurant downtown",
    "hiking trail with waterfall"
  ],
  "location": {
    "address": "123 Main St, Austin, TX",
    "lat": 30.2672,
    "lng": -97.7431
  },
  "availability": {
    "days": ["friday_evening", "saturday", "sunday"],
    "minDuration": 120,
    "advanceNotice": 3
  }
}
```

### 4. Activity Database (`/activities`)

**If using hardcoded activities:**

`activities.json`:
```json
[
  {
    "id": "act-1",
    "name": "Beginner Pottery Workshop",
    "category": "classes",
    "description": "Learn hand-building and wheel throwing",
    "location": {
      "address": "456 Art St, Austin, TX",
      "lat": 30.2650,
      "lng": -97.7400
    },
    "duration": 180,
    "price": "$$",
    "keywords": ["pottery", "ceramics", "art", "class", "hands-on"],
    "url": "https://example.com/pottery-class"
  },
  {
    "id": "act-2",
    "name": "Osteria Moderna",
    "category": "dining",
    "description": "Modern Italian cuisine, farm-to-table",
    "location": {
      "address": "789 Food Ave, Austin, TX",
      "lat": 30.2700,
      "lng": -97.7500
    },
    "duration": 120,
    "price": "$$$",
    "keywords": ["italian", "restaurant", "fine dining", "pasta", "wine"],
    "url": "https://example.com/osteria"
  }
  // ... 18-28 more activities covering different categories
]
```

**Functions:**
```javascript
searchActivities(interests, keywords, location, maxDistance) {
  // Load activities from activities.json
  // Filter by category (match interests)
  // Keyword search in wantToTry list
  // Filter by distance from location
  // Return ranked matches
}
```

**If using live APIs:**
```javascript
searchEventbriteAPI(location, dateRange, categories) {
  // Call Eventbrite API
  // Parse results
  // Return standardized format
}
```

### 5. Matching Engine (`/matching`)

**Main function:**
```javascript
generateSuggestions(coupleId) {
  // 1. Load couple preferences
  const prefs = loadPreferences(coupleId);

  // 2. Get calendar events for both partners
  const events1 = getCalendarEvents(partner1Token, startDate, endDate);
  const events2 = getCalendarEvents(partner2Token, startDate, endDate);

  // 3. Find overlapping free time
  const freeWindows = findFreeTimeWindows(events1, events2);

  // 4. Search for matching activities
  const activities = searchActivities(
    prefs.interests,
    prefs.wantToTry,
    prefs.location,
    maxDistance = 15 // miles
  );

  // 5. Score and rank activities
  const rankedActivities = scoreActivities(activities, prefs);

  // 6. Match activities to time windows
  const suggestions = matchActivitiesToTimeWindows(
    rankedActivities,
    freeWindows
  );

  // 7. Create top 1-2 calendar events
  for (let suggestion of suggestions.slice(0, 2)) {
    createCalendarEvent(partner1Token, partner2Token, {
      title: `Suggested: ${suggestion.name}`,
      start: suggestion.timeWindow.start,
      end: suggestion.timeWindow.end,
      location: suggestion.location.address,
      description: formatDescription(suggestion, prefs)
    });
  }

  // 8. Log suggestions
  saveSuggestions(coupleId, suggestions);

  return suggestions;
}
```

### 6. Admin Dashboard (`/admin`)

**Simple web page:**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Calendar Concierge - Admin</title>
  <style>
    /* Basic styling */
    body { font-family: Arial; padding: 20px; }
    .couple-card { border: 1px solid #ccc; padding: 15px; margin: 10px 0; }
    .button { padding: 10px 20px; background: #007bff; color: white; border: none; cursor: pointer; }
    .log { background: #f5f5f5; padding: 10px; margin: 10px 0; font-family: monospace; font-size: 12px; }
  </style>
</head>
<body>
  <h1>Calendar Concierge - Admin Dashboard</h1>

  <div class="couple-card">
    <h2>Test Couple: Alex & Jordan</h2>
    <p>Status: Both calendars connected ✓</p>
    <p>Interests: Dining, Arts, Classes</p>
    <button class="button" onclick="runMatching('couple-1')">
      Generate Suggestions
    </button>
    <div id="results-couple-1" class="log"></div>
  </div>

  <h2>Activity Log</h2>
  <div id="activity-log" class="log"></div>

  <script>
    async function runMatching(coupleId) {
      const results = document.getElementById(`results-${coupleId}`);
      results.innerHTML = "Running matching engine...";

      const response = await fetch(`/api/matching/${coupleId}`, {
        method: 'POST'
      });

      const data = await response.json();

      results.innerHTML = `
        <h3>Results:</h3>
        <p>Free time windows found: ${data.freeWindows.length}</p>
        <p>Activities matched: ${data.matchedActivities.length}</p>
        <p>Suggestions created: ${data.suggestionsCreated.length}</p>
        <h4>Suggestions:</h4>
        <ul>
          ${data.suggestionsCreated.map(s => `
            <li>
              <strong>${s.name}</strong><br>
              ${s.timeWindow.start} - ${s.timeWindow.end}<br>
              Match reason: ${s.matchReason}<br>
              <a href="${s.calendarLink}" target="_blank">View in Google Calendar</a>
            </li>
          `).join('')}
        </ul>
      `;

      // Add to activity log
      addToLog(`Generated ${data.suggestionsCreated.length} suggestions for ${coupleId}`);
    }

    function addToLog(message) {
      const log = document.getElementById('activity-log');
      const timestamp = new Date().toLocaleString();
      log.innerHTML = `[${timestamp}] ${message}<br>` + log.innerHTML;
    }
  </script>
</body>
</html>
```

---

## File Structure

```
calendar-concierge-mvp/
├── server/
│   ├── index.js                 # Main Express app
│   ├── services/
│   │   ├── auth.js              # OAuth handling
│   │   ├── calendar.js          # Google Calendar API
│   │   ├── matching.js          # Core matching logic
│   │   └── activities.js        # Activity search/database
│   ├── data/
│   │   ├── users.json           # OAuth tokens
│   │   ├── preferences.json     # Couple preferences
│   │   ├── activities.json      # Activity database
│   │   └── suggestions.json     # Suggestion history
│   └── config/
│       └── google-credentials.json  # OAuth client credentials
├── public/
│   ├── index.html               # Landing page
│   ├── preferences.html         # Preferences form
│   ├── admin.html               # Admin dashboard
│   └── styles.css               # Basic styling
├── .env                         # Environment variables
├── package.json                 # Dependencies
└── README.md                    # Setup instructions
```

---

## API Endpoints

### Authentication
- `GET /auth/google?partner=1` - Start OAuth for Partner 1
- `GET /auth/google?partner=2` - Start OAuth for Partner 2
- `GET /auth/callback` - OAuth callback handler
- `GET /auth/status/:coupleId` - Check auth status

### Preferences
- `GET /preferences/:coupleId` - Get preferences
- `POST /preferences/:coupleId` - Save preferences

### Matching
- `POST /matching/:coupleId` - Run matching engine (manual trigger)
- `GET /matching/:coupleId/history` - Get past suggestions

### Admin
- `GET /admin` - Admin dashboard (HTML page)
- `GET /api/couples` - List all couples
- `GET /api/activities` - List all activities

---

## Dependencies (Node.js)

```json
{
  "dependencies": {
    "express": "^4.18.0",
    "googleapis": "^118.0.0",
    "dotenv": "^16.0.0",
    "node-fetch": "^3.3.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

**Core libraries:**
- `express` - Web server
- `googleapis` - Google Calendar API client
- `dotenv` - Environment variables
- `node-fetch` - HTTP requests (if using Eventbrite API)

---

## Setup & Configuration

### 1. Google Cloud Project Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create new project: "Calendar Concierge MVP"
3. Enable Google Calendar API
4. Create OAuth 2.0 credentials:
   - Application type: Web application
   - Authorized redirect URI: `http://localhost:3000/auth/callback`
5. Download credentials JSON
6. Add test users (your two test accounts) to OAuth consent screen

### 2. Environment Variables

`.env` file:
```
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:3000/auth/callback
PORT=3000
```

### 3. Initial Data Setup

Create empty JSON files:
```json
// data/users.json
{
  "couple-1": {
    "partner1": {
      "email": "alex@example.com",
      "accessToken": null,
      "refreshToken": null
    },
    "partner2": {
      "email": "jordan@example.com",
      "accessToken": null,
      "refreshToken": null
    }
  }
}

// data/preferences.json
{}

// data/suggestions.json
[]
```

Populate `data/activities.json` with 20-30 sample activities (see structure above)

---

## Demo Flow / User Journey

### Setup Phase (Before Demo)

1. **OAuth both test accounts:**
   - Navigate to `http://localhost:3000/auth/google?partner=1`
   - Login as Partner 1 (Alex), grant permissions
   - Navigate to `http://localhost:3000/auth/google?partner=2`
   - Login as Partner 2 (Jordan), grant permissions
   - Verify both tokens stored in `users.json`

2. **Set preferences:**
   - Navigate to `http://localhost:3000/preferences/couple-1`
   - Fill out form:
     - Interests: ✓ Dining, ✓ Arts, ✓ Classes
     - Want to try: "pottery class", "Italian restaurant", "wine tasting"
     - Location: Austin, TX
   - Submit
   - Verify saved in `preferences.json`

3. **Verify test calendars have some events:**
   - Add 3-4 events to each test calendar (meetings, appointments)
   - Leave some free time on an upcoming weekend

### Live Demo (5-10 minutes)

**Minute 1-2: Show the problem**
- "Meet Alex and Jordan, a busy couple who maintain a list of things they want to try together"
- Show their separate Google Calendars with events
- "They both have free time Saturday afternoon, but coordinating and deciding what to do takes effort"

**Minute 3-4: Show the setup**
- Navigate to preferences page
- "They did a one-time setup: connected calendars, shared interests, and their 'want to try' list"
- Show preferences form (already filled out)

**Minute 5-7: Show the magic**
- Navigate to admin dashboard
- "Every week, the system automatically finds when they're both free and suggests activities"
- Click "Generate Suggestions" button
- Show results:
  - "Found 3 overlapping free time windows"
  - "Matched 8 activities to their interests"
  - "Created 2 suggestions in their calendars"
- Show the suggestions created (time, activity, match reason)

**Minute 8-9: Show the result**
- Open Alex's Google Calendar (live)
- Show tentative event: "Suggested: Pottery Workshop"
- Show event details with description
- Open Jordan's Google Calendar (live)
- Show same event appears in both calendars
- "They see it when planning their week, accept or decline in 5 seconds"

**Minute 10: Explain the value**
- "No app to remember to check"
- "No 30-minute 'what should we do?' conversation"
- "Their 'want to try' list becomes actual plans"

---

## Testing Checklist

### Pre-Demo Testing

- [ ] OAuth flow works for both partners
- [ ] Tokens stored and retrievable
- [ ] Can read events from both test calendars
- [ ] Free time detection finds overlapping windows
- [ ] Activity matching returns relevant results
- [ ] Calendar event creation works (appears in both calendars)
- [ ] Admin dashboard displays correctly
- [ ] "Run Matching" button triggers successfully
- [ ] Results display correctly in dashboard
- [ ] Created events visible in Google Calendar web/app

### Edge Cases to Handle

- [ ] OAuth token expired → refresh token works
- [ ] No free time found → display friendly message
- [ ] No matching activities → display message, suggest broadening interests
- [ ] API rate limits → handle gracefully (unlikely in demo)
- [ ] Both partners not authenticated → show warning in dashboard

---

## What's Intentionally Missing (Out of Scope)

These are NOT needed for a working demo:

### User Management
- ❌ User signup/registration
- ❌ Login/logout flows
- ❌ Password management
- ❌ Email verification
- ❌ Multiple couples (just one test couple is fine)

### Data Persistence
- ❌ Production database (PostgreSQL)
- ❌ Database migrations
- ❌ Data backups
- ❌ User data encryption (beyond what Google handles)

### Automation
- ❌ Scheduled jobs (cron)
- ❌ Automatic weekly suggestions
- ❌ Email notifications
- ❌ Push notifications

### Advanced Features
- ❌ Feedback collection (thumbs up/down)
- ❌ Learning algorithm (improve matches over time)
- ❌ Historical dashboard (past suggestions)
- ❌ "Want to try" list management UI
- ❌ Edit preferences without re-entering everything
- ❌ Multiple suggestion batches
- ❌ Reschedule/cancel suggestions

### Real-World Concerns
- ❌ Production hosting/deployment
- ❌ HTTPS/SSL certificates
- ❌ Rate limiting
- ❌ Error logging/monitoring
- ❌ Analytics
- ❌ Performance optimization
- ❌ Multi-tenancy
- ❌ Data privacy compliance (GDPR, etc.)
- ❌ Terms of Service / Privacy Policy

### Polish
- ❌ Beautiful UI/UX design
- ❌ Mobile responsiveness
- ❌ Loading states/spinners
- ❌ Form validation (beyond basic)
- ❌ Error messages (beyond console.log)

**Why this is okay:**
- It's a demo, not a product
- Goal is to prove the concept works
- Can add polish later if pursuing further

---

## Success Criteria for MVP Demo

### Technical Success
- ✅ OAuth integration functional (both partners authenticated)
- ✅ Can read calendar events via Google Calendar API
- ✅ Free time detection algorithm works
- ✅ Activity matching produces relevant results
- ✅ Can write events to Google Calendar (visible in web/app)
- ✅ Admin dashboard allows manual triggering
- ✅ End-to-end flow works without crashes

### Demo Success
- ✅ Can complete full demo in 5-10 minutes
- ✅ Audience understands the value proposition
- ✅ Technical functionality is clearly demonstrated
- ✅ No major bugs or errors during presentation
- ✅ Questions can be answered with working code

### Learning Success (Class Project)
- ✅ Demonstrates OAuth 2.0 implementation
- ✅ Shows API integration (Google Calendar)
- ✅ Exhibits multi-user coordination logic
- ✅ Presents algorithm design (matching)
- ✅ Code is readable and documented
- ✅ Can explain technical decisions

---

## Development Timeline (6-8 Weeks)

### Week 1-2: Foundation
- Set up project structure
- Google Cloud project + OAuth credentials
- Basic Express server
- OAuth flow implementation (hardest part - give it time!)
- Test: Can authenticate and get tokens

### Week 3-4: Calendar Integration
- Google Calendar API integration
- Read calendar events
- Free time detection algorithm
- Test: Can find overlapping free time

### Week 4-5: Matching Logic
- Create activity database (JSON with 20-30 activities)
- Implement activity matching algorithm
- Scoring/ranking logic
- Test: Given preferences, returns relevant activities

### Week 5-6: Suggestion Creation
- Calendar event creation via API
- Format suggestion events properly
- Test: Events appear in both Google Calendars

### Week 6-7: Admin Dashboard
- Build simple HTML dashboard
- Manual trigger button
- Results display
- Activity log
- Test: Can run full flow from dashboard

### Week 7-8: Polish & Testing
- End-to-end testing
- Bug fixes
- Demo script preparation
- Backup plan (screenshots if live demo fails)
- Documentation

---

## Risk Mitigation

### Risk: OAuth flow breaks during demo
**Mitigation:**
- Test OAuth thoroughly before demo day
- Have tokens pre-authenticated and stored
- Take screenshots as backup
- Explain flow verbally if needed

### Risk: Google Calendar API errors during demo
**Mitigation:**
- Run full demo 2-3 times before presentation
- Have cached results ready to display
- Show pre-created suggestions in calendar
- Record video of working demo as backup

### Risk: No internet during demo
**Mitigation:**
- Use local mock data for matching (activities.json)
- Pre-run matching and show results in dashboard
- Show architecture diagrams to explain flow
- Have slides as backup

### Risk: Running out of time in development
**Mitigation:**
- Week 1-2 focus on OAuth (hardest part)
- If behind, use mock data instead of live APIs
- Hardcode one couple instead of building multi-couple support
- Use ugly but functional UI (focus on backend)
- Can demo with Postman/curl if no time for dashboard

---

## Optional Enhancements (If Time Allows)

**If ahead of schedule, consider adding:**

1. **Live Eventbrite API Integration** (instead of hardcoded activities)
   - More impressive in demo
   - Shows real-world API integration
   - Adds ~1 week development time

2. **Simple User Dashboard** (not just admin)
   - View upcoming suggestions
   - See past suggestions
   - Basic profile page
   - Adds ~3-5 days development time

3. **Feedback Collection**
   - After event time passes, ask "Did you go?"
   - Display feedback in dashboard
   - Adds ~2-3 days development time

4. **Nicer UI**
   - Use a CSS framework (Bootstrap, Tailwind)
   - Make forms and dashboard look professional
   - Adds ~3-5 days development time

5. **Deploy to Cloud** (Railway, Render)
   - Remote demo access (no localhost dependency)
   - Shareable link for testing
   - Adds ~1-2 days for deployment/config

**Prioritization:** Only add if core functionality is solid and tested!

---

## Questions to Resolve Before Starting

1. **Which language/framework?**
   - Recommendation: Node.js + Express (best Google API support)
   - Alternative: Python + Flask (also good)

2. **Hardcoded activities or live API?**
   - Start with hardcoded (faster, more reliable for demo)
   - Add Eventbrite API if time allows

3. **Data storage?**
   - Start with JSON files (simplest)
   - Upgrade to SQLite if you want database practice

4. **What city for activities?**
   - Use your local city (easier to verify data)
   - Or pick Austin/Portland/Denver (good event density)

5. **How many test couples?**
   - Just one for MVP (couple-1)
   - Easy to expand later

6. **Local only or deploy?**
   - Start local (localhost demo)
   - Deploy if time allows (more impressive)

---

## Summary: Is This Functional?

**Yes, this MVP is functional and achievable in 6-8 weeks.**

**What makes it doable:**
- ✅ Focused on core functionality only
- ✅ No user management complexity
- ✅ Simple data storage (JSON files)
- ✅ Manual triggering (no scheduling complexity)
- ✅ One test couple (no multi-tenancy)
- ✅ Well-documented Google APIs
- ✅ Clear scope boundaries

**What makes it impressive:**
- ✅ Real OAuth integration (hardest part of many apps)
- ✅ Live Google Calendar integration
- ✅ Multi-user coordination
- ✅ Working algorithm (matching logic)
- ✅ End-to-end functional demo
- ✅ Solves a real problem

**What you'll learn:**
- OAuth 2.0 flows
- REST API design
- External API integration
- Algorithm design
- Multi-user systems
- Demo presentation skills

**Verdict: This is well-scoped for a class project demo. You should be able to build this in 6-8 weeks and deliver a compelling demonstration.**

---

## Next Steps

1. **Review this MVP spec** - Any questions or concerns?
2. **Choose your tech stack** - Node.js or Python?
3. **Set up development environment** - Install dependencies
4. **Create Google Cloud project** - Get OAuth credentials
5. **Start with Week 1-2 tasks** - OAuth flow (hardest part first!)

Ready to start building?
