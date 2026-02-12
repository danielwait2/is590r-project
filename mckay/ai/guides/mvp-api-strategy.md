# MVP API Strategy - What We're Actually Building With

**Last Updated:** 2026-02-12
**Status:** Verified and confirmed

---

## TL;DR

✅ **Google Calendar API** - Confirmed working, this is our core integration
❌ **Eventbrite API** - Deprecated, doesn't work for public event search
✅ **Hardcoded Activities** - Our MVP approach, reliable and fast

---

## What We're Using

### 1. Google Calendar API ✅
**Purpose:** Core calendar integration (read events, create suggestions)
**Status:** Fully functional, well-documented
**Guide:** See `google-calendar-api.md`

**What we'll do:**
- OAuth 2.0 to authenticate both partners
- Read calendar events to find busy times
- Calculate overlapping free time
- Create tentative events as suggestions

**APIs we'll use:**
- `GET /calendars/primary/events` - List events
- `POST /calendars/primary/events` - Create suggestions

**Node.js package:** `googleapis`

**No concerns:** This is solid, official, maintained.

---

### 2. Hardcoded Activity Database ✅
**Purpose:** Activity matching (what to suggest)
**Status:** Manual creation, JSON file
**Why:** Eventbrite API doesn't work (see `eventbrite-api-DEPRECATED.md`)

**Implementation:**
```
data/activities.json
├── 20-30 curated activities
├── Multiple categories (dining, classes, arts, outdoors)
├── Real businesses with addresses and URLs
└── Keyword tags for matching
```

**Structure:**
```json
{
  "id": "act-1",
  "name": "Pottery Workshop for Beginners",
  "category": "classes",
  "description": "3-hour hands-on pottery class",
  "location": {
    "address": "456 Art St, Austin, TX",
    "lat": 30.2650,
    "lng": -97.7400
  },
  "duration": 180,
  "price": "$$",
  "keywords": ["pottery", "ceramics", "art", "class", "hands-on"],
  "url": "https://example.com/pottery"
}
```

**Matching logic:**
1. User interests → filter activities by category
2. "Want to try" keywords → match against activity keywords
3. Location → filter by distance from home
4. Duration → match to available time window

**Why hardcoded is fine:**
- ✅ Reliable for demo (no API failures)
- ✅ High-quality curation (we control content)
- ✅ Fast to implement (2-3 hours to populate)
- ✅ No API keys, no rate limits, no costs
- ✅ Perfect for class project scope

---

## What We're NOT Using (and Why)

### Eventbrite API ❌
**Why not:** Public search API deprecated in 2020, API largely unsupported
**See:** `eventbrite-api-DEPRECATED.md`

### PredictHQ / SerpApi ❌
**Why not:** Paid services, overkill for MVP

### Google Places API ⚠️
**Why not now:** Doesn't do events (only static places like restaurants)
**Maybe later:** Could use for restaurant discovery from "want to try" list

### Web Scraping ❌
**Why not:** Against TOS, brittle, not worth the effort for MVP

---

## MVP Data Flow

```
1. User Setup
   ├── OAuth → Google Calendar (both partners)
   ├── Preferences form → preferences.json
   └── "Want to try" list → preferences.json

2. Matching Engine (runs on demand)
   ├── Read calendars → find busy times (Google Calendar API)
   ├── Calculate free time → overlapping windows (our algorithm)
   ├── Load activities → activities.json
   ├── Match interests → filter activities (our logic)
   └── Score & rank → top matches (simple keyword scoring)

3. Suggestion Creation
   ├── Format event details → title, description, time
   ├── Create calendar events → Google Calendar API (both calendars)
   └── Log suggestion → suggestions.json
```

**External API calls:** Only Google Calendar API
**Everything else:** Local data and logic

---

## Technical Stack (Confirmed)

### Backend
- **Node.js + Express** ✅
- **googleapis npm package** ✅ (official Google API client)

### Data Storage
- **JSON files** for MVP:
  - `users.json` - OAuth tokens
  - `preferences.json` - User interests
  - `activities.json` - Activity database
  - `suggestions.json` - History

### APIs
- **Google Calendar API v3** ✅
- **No other external APIs**

---

## Development Priorities

### Week 1-2: Google OAuth + Calendar Reading
**Focus:** Get authentication working
- Set up Google Cloud project
- Implement OAuth 2.0 flow
- Store/refresh tokens
- Read calendar events successfully

**Hardest part of the project** - give this time!

### Week 3-4: Free Time Detection
**Focus:** Algorithm to find overlapping availability
- Parse calendar events
- Calculate busy blocks
- Find free windows
- Apply filters (weekends, min duration, etc.)

**No external API needed** - pure logic

### Week 5: Activity Matching
**Focus:** Match activities to preferences
- Create `activities.json` (manual, 2-3 hours)
- Load activities
- Filter by category (interests)
- Keyword search (want to try list)
- Score and rank

**No external API needed** - local data + simple algorithm

### Week 6: Calendar Event Creation
**Focus:** Insert suggestions into calendars
- Format event object (title, time, description, location)
- Call Google Calendar API to create event
- Create in both calendars
- Handle errors

**Google Calendar API** - straightforward once OAuth works

### Week 7: Admin Dashboard + Integration
**Focus:** Tie it all together
- Simple HTML page
- "Run matching" button
- Display results
- End-to-end testing

### Week 8: Polish + Demo Prep
**Focus:** Make it demo-ready
- Bug fixes
- Test with real test accounts
- Demo script
- Screenshots as backup

---

## Risk Assessment

### Low Risk ✅
- **Google Calendar API** - Well-documented, works as advertised
- **OAuth 2.0** - Standard flow, plenty of examples
- **Hardcoded activities** - Under our control, reliable
- **Free time algorithm** - Pure logic, no external dependencies

### Medium Risk ⚠️
- **OAuth token refresh** - Need to handle expiration properly
- **Timezone handling** - Easy to mess up, need to test carefully
- **Demo day internet** - Have backup plan (cached results, screenshots)

### No Risk (Eliminated) ✅
- **Eventbrite API** - Not using it (deprecated anyway)
- **Event data quality** - Controlled by us (hardcoded)
- **API rate limits** - Google's limits are high, not a concern
- **API costs** - Google Calendar API is free for our usage

---

## If We Want to Scale Post-MVP

**Options for real event data:**
1. Manual curation (hire VA, weekly updates)
2. Partner with local venues directly
3. Build web scraper (if comfortable with ethics)
4. Pay for enterprise API (PredictHQ, etc.)

**For now:** Hardcoded is perfect for class project

---

## Summary: What's Real, What's Not

| Component | Status | Source |
|-----------|--------|--------|
| Google Calendar API | ✅ Real, working | Official Google API |
| OAuth 2.0 for Calendar | ✅ Real, working | Official Google OAuth |
| Eventbrite Search API | ❌ Deprecated | Removed 2020 |
| Eventbrite Org API | ⚠️ Exists but useless | Can't search publicly |
| PredictHQ Events API | ✅ Exists but paid | Enterprise service |
| Google Places API | ✅ Exists, no events | Places only, not events |
| Hardcoded activities | ✅ Our approach | Manual creation |

---

## Decision: Confirmed Approach

**For MVP, we're building with:**
1. ✅ Google Calendar API (OAuth + read/write events)
2. ✅ Hardcoded activities in JSON file
3. ✅ Simple matching algorithm (keyword-based)
4. ✅ Manual triggering (admin dashboard button)

**This is:**
- ✅ Technically feasible (APIs are real and work)
- ✅ Demo-ready (reliable, no external dependencies for activity data)
- ✅ Good learning project (OAuth, API integration, algorithms)
- ✅ Achievable in 6-8 weeks

**We are NOT blocked by deprecated APIs.**

The plan is solid. Let's build it.
