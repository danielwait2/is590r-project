# Phase 2: Calendar Reading - Implementation Plan

**Created:** 2026-02-12
**Phase:** 2 of 10
**Timeline:** Week 3
**Companion Document:** [Phase 2 Roadmap](./2026-02-12_phase-2_calendar-reading-roadmap.md)

---

## Project Principles

⚠️ **IMPORTANT: This is a clean-slate MVP build**

**DO:**
- Build the simplest calendar reading logic
- Focus on getting event data we need (start/end times)
- Handle token refresh automatically
- Keep API calls minimal

**DON'T:**
- Over-engineer event parsing
- Add caching (not needed for MVP)
- Handle every possible event format
- Build complex retry logic

---

## Phase Overview

**Goal:** Read calendar events from Google Calendar API for both partners

**Why this matters:** Need calendar data to find free time. This phase validates OAuth works and sets up for free time detection.

**Key deliverables:**
- Calendar service module with Google Calendar API integration
- Function to list events for date range
- Function to extract busy blocks from events
- Test endpoint to view calendar data
- Token refresh handling

---

## Technical Specifications

### 1. Calendar Service Module

**File:** `server/services/calendar.js`

**Purpose:** Interact with Google Calendar API v3

**Required functions:**

#### getCalendarEvents(coupleId, partnerId, startDate, endDate)
```javascript
/**
 * Gets calendar events for a partner
 * @param {string} coupleId - Couple identifier
 * @param {string} partnerId - 'partner1' or 'partner2'
 * @param {string} startDate - ISO 8601 date (e.g., '2026-02-15T00:00:00Z')
 * @param {string} endDate - ISO 8601 date (e.g., '2026-03-01T00:00:00Z')
 * @returns {Promise<Array>} events - Array of calendar events
 */
```

**Implementation details:**
- Load OAuth tokens using auth service
- Create OAuth2 client with tokens
- Call Google Calendar API: GET /calendars/primary/events
- Handle token expiry (401 → refresh → retry)
- Return array of event objects

**API parameters:**
- timeMin: startDate (ISO 8601)
- timeMax: endDate (ISO 8601)
- singleEvents: true (expand recurring events)
- orderBy: 'startTime'
- maxResults: 100 (enough for 2 weeks)

**Response format:**
```javascript
[
  {
    id: 'event123',
    summary: 'Team Meeting',
    start: { dateTime: '2026-02-15T10:00:00-06:00' },
    end: { dateTime: '2026-02-15T11:00:00-06:00' },
    status: 'confirmed'
  },
  // ... more events
]
```

#### extractBusyBlocks(events)
```javascript
/**
 * Extracts busy time blocks from calendar events
 * @param {Array} events - Calendar events from Google API
 * @returns {Array} busyBlocks - Array of {start, end} objects
 */
```

**Implementation details:**
- Filter out cancelled events (status !== 'cancelled')
- Extract start and end times
- Handle all-day events (mark 9am-5pm as busy)
- Convert to standard format
- Sort by start time
- Return array of busy blocks

**Busy block format:**
```javascript
[
  {
    start: '2026-02-15T10:00:00-06:00',
    end: '2026-02-15T11:00:00-06:00'
  },
  // ... more blocks
]
```

#### getBusyBlocksForCouple(coupleId, startDate, endDate)
```javascript
/**
 * Gets busy blocks for both partners
 * @param {string} coupleId - Couple identifier
 * @param {string} startDate - Start of date range
 * @param {string} endDate - End of date range
 * @returns {Promise<object>} Object with partner1 and partner2 busy blocks
 */
```

**Implementation details:**
- Call getCalendarEvents for both partners
- Extract busy blocks for each
- Return object with both partners' data

**Response format:**
```javascript
{
  partner1: {
    email: 'alex@example.com',
    busyBlocks: [
      { start: '...', end: '...' },
      // ...
    ]
  },
  partner2: {
    email: 'jordan@example.com',
    busyBlocks: [
      { start: '...', end: '...' },
      // ...
    ]
  }
}
```

---

### 2. Token Refresh Integration

**Automatic token refresh:**

```javascript
async function makeCalendarAPICall(oauth2Client, apiCall) {
  try {
    return await apiCall();
  } catch (error) {
    if (error.code === 401) {
      // Token expired, refresh and retry
      console.log('Token expired, refreshing...');
      await oauth2Client.refreshAccessToken();
      return await apiCall(); // Retry with new token
    }
    throw error; // Re-throw other errors
  }
}
```

**Integration with auth service:**
- When 401 error occurs, call authService.refreshAccessToken()
- Update tokens in storage
- Retry API call with new token
- If refresh fails, throw error (user needs to reauthorize)

---

### 3. Test Endpoint

**File:** `server/routes/calendar.js`

**Route: GET /calendar/:coupleId/events**

**Purpose:** View calendar events for debugging

**Query parameters:**
- start: Start date (optional, default: today)
- end: End date (optional, default: today + 14 days)

**Implementation:**
```javascript
router.get('/:coupleId/events', async (req, res) => {
  try {
    const { coupleId } = req.params;
    const startDate = req.query.start || new Date().toISOString();
    const endDate = req.query.end || getDatePlusDays(14).toISOString();

    // Get events for both partners
    const partner1Events = await calendarService.getCalendarEvents(
      coupleId, 'partner1', startDate, endDate
    );
    const partner2Events = await calendarService.getCalendarEvents(
      coupleId, 'partner2', startDate, endDate
    );

    res.json({
      coupleId,
      dateRange: { start: startDate, end: endDate },
      partner1: {
        eventCount: partner1Events.length,
        events: partner1Events
      },
      partner2: {
        eventCount: partner2Events.length,
        events: partner2Events
      }
    });
  } catch (error) {
    console.error('Error fetching calendar events:', error);
    res.status(500).json({ error: error.message });
  }
});
```

**Response format:**
```json
{
  "coupleId": "couple-1",
  "dateRange": {
    "start": "2026-02-15T00:00:00Z",
    "end": "2026-03-01T00:00:00Z"
  },
  "partner1": {
    "eventCount": 8,
    "events": [...]
  },
  "partner2": {
    "eventCount": 12,
    "events": [...]
  }
}
```

---

**Route: GET /calendar/:coupleId/busy**

**Purpose:** View busy blocks (for debugging free time detection)

**Implementation:**
```javascript
router.get('/:coupleId/busy', async (req, res) => {
  try {
    const { coupleId } = req.params;
    const startDate = req.query.start || new Date().toISOString();
    const endDate = req.query.end || getDatePlusDays(14).toISOString();

    const busyData = await calendarService.getBusyBlocksForCouple(
      coupleId, startDate, endDate
    );

    res.json({
      coupleId,
      dateRange: { start: startDate, end: endDate },
      ...busyData
    });
  } catch (error) {
    console.error('Error fetching busy blocks:', error);
    res.status(500).json({ error: error.message });
  }
});
```

---

### 4. Event Parsing Details

**Handle different event formats:**

**Standard timed event:**
```json
{
  "start": { "dateTime": "2026-02-15T10:00:00-06:00" },
  "end": { "dateTime": "2026-02-15T11:00:00-06:00" }
}
```

**All-day event:**
```json
{
  "start": { "date": "2026-02-15" },
  "end": { "date": "2026-02-16" }
}
```
Treat as busy from 9am-5pm on the date(s).

**Recurring event:**
Use `singleEvents: true` in API call to automatically expand recurring events into individual instances.

**Cancelled event:**
```json
{
  "status": "cancelled"
}
```
Skip these - not busy time.

**Tentative event:**
```json
{
  "status": "tentative"
}
```
Still count as busy (conservative approach).

---

## API Reference

### Google Calendar API v3

**Base URL:**
```
https://www.googleapis.com/calendar/v3
```

**Endpoint: List Events**
```
GET /calendars/primary/events
```

**Parameters:**
- `timeMin` (string): Lower bound for event start time (ISO 8601)
- `timeMax` (string): Upper bound for event start time (ISO 8601)
- `singleEvents` (boolean): Expand recurring events (set to true)
- `orderBy` (string): Sort order ('startTime' requires singleEvents=true)
- `maxResults` (integer): Max number of events (default 250, max 2500)

**Authentication:**
```javascript
Authorization: Bearer {access_token}
```

**Rate Limits:**
- 10 queries per second per user
- 1,000,000 queries per day per project

**Response format:**
```json
{
  "items": [
    {
      "id": "event_id",
      "status": "confirmed",
      "summary": "Event title",
      "description": "Event description",
      "location": "Event location",
      "start": {
        "dateTime": "2026-02-15T10:00:00-06:00",
        "timeZone": "America/Chicago"
      },
      "end": {
        "dateTime": "2026-02-15T11:00:00-06:00",
        "timeZone": "America/Chicago"
      }
    }
  ]
}
```

**Error responses:**
- 401 Unauthorized: Token expired or invalid
- 403 Forbidden: Rate limit exceeded or quota exceeded
- 404 Not Found: Calendar not found

**googleapis npm usage:**
```javascript
const { google } = require('googleapis');

const calendar = google.calendar({ version: 'v3', auth: oauth2Client });

const response = await calendar.events.list({
  calendarId: 'primary',
  timeMin: startDate,
  timeMax: endDate,
  singleEvents: true,
  orderBy: 'startTime',
  maxResults: 100
});

const events = response.data.items;
```

---

## Data Structures

### Calendar Event Object (simplified)
```javascript
{
  id: string,              // Unique event ID
  summary: string,         // Event title
  start: {
    dateTime: string,      // ISO 8601 datetime (timed event)
    date: string          // Date only (all-day event)
  },
  end: {
    dateTime: string,
    date: string
  },
  status: string          // 'confirmed', 'tentative', 'cancelled'
}
```

### Busy Block Object
```javascript
{
  start: string,          // ISO 8601 datetime
  end: string            // ISO 8601 datetime
}
```

---

## Testing Criteria

### Calendar API Integration Testing
- [ ] Can call Google Calendar API with valid token
- [ ] API returns events list
- [ ] Events parsed correctly (start/end times)
- [ ] All-day events handled correctly
- [ ] Recurring events expanded
- [ ] Cancelled events filtered out

### Token Refresh Testing
- [ ] API call with expired token triggers refresh
- [ ] Refresh successful, new token stored
- [ ] Retry succeeds with new token
- [ ] Multiple API calls work without re-authentication

### Busy Blocks Testing
- [ ] extractBusyBlocks returns correct format
- [ ] Busy blocks sorted by start time
- [ ] All-day events converted to 9am-5pm blocks
- [ ] Empty calendar returns empty array

### Test Endpoint Testing
- [ ] GET /calendar/couple-1/events returns JSON
- [ ] Both partners' events included
- [ ] Event count matches actual events
- [ ] GET /calendar/couple-1/busy returns busy blocks
- [ ] Busy blocks look correct

### Integration Testing
- [ ] Can read Partner 1 calendar
- [ ] Can read Partner 2 calendar
- [ ] Both calendars read in same request
- [ ] Works with real test calendar data
- [ ] No errors with empty calendar
- [ ] No errors with full calendar (many events)

---

## Common Issues & Solutions

### Issue: 401 Unauthorized error
**Symptoms:** API call fails with 401
**Solution:**
- Token expired - implement refresh logic
- Token invalid - user needs to reauthorize
- Check token is correctly loaded from storage

### Issue: All-day events missing time
**Symptoms:** Event has `date` not `dateTime`
**Solution:**
- Check for `start.date` vs `start.dateTime`
- Convert all-day to timed event (9am-5pm)
- Handle both formats in parsing logic

### Issue: Recurring events show once
**Symptoms:** Weekly meeting only shows one instance
**Solution:**
- Ensure `singleEvents: true` in API parameters
- This expands recurring events to individual instances

### Issue: Too many events (pagination needed)
**Symptoms:** Calendar has >100 events in 2 weeks
**Solution:**
- For MVP: Increase maxResults to 250
- For production: Implement pagination with nextPageToken

### Issue: Timezone confusion
**Symptoms:** Events showing at wrong times
**Solution:**
- Keep all times in ISO 8601 format with timezone
- Don't convert to local time yet
- JavaScript Date handles ISO 8601 correctly

---

## Blockers & Dependencies

### Blocked By
- Phase 1: OAuth Integration (need valid tokens)

### Blocks
- Phase 3: Free Time Detection (needs busy blocks)

### Dependencies
- Valid OAuth tokens in users.json
- Test calendars with some events
- Internet connection for API calls

---

## Next Steps

After completing Phase 2:
1. Verify can read events from both calendars
2. Confirm busy blocks extracted correctly
3. Test with various event types (all-day, recurring, etc.)
4. Move to [Phase 3: Free Time Detection](./2026-02-12_phase-3_free-time-detection-plan.md)
5. Review [Phase 3 Roadmap](./2026-02-12_phase-3_free-time-detection-roadmap.md)

**Phase 2 is complete when:**
- Can read calendar events via API
- Busy blocks extracted correctly
- Token refresh works automatically
- Test endpoints working
- Ready to calculate free time

---

**Reference Documents:**
- [High-Level Implementation Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture Documentation](../../aiDocs/architecture.md)
- [MVP Specification](../../aiDocs/mvp.md)
- [Phase 2 Roadmap](./2026-02-12_phase-2_calendar-reading-roadmap.md)
- [Google Calendar API Docs](https://developers.google.com/workspace/calendar/api/v3/reference)
