# Google Calendar API - Verified Documentation

**Status:** ✅ **CONFIRMED WORKING** - Official Google API, actively maintained
**Last Updated:** 2026-02-12
**API Version:** v3
**Base URL:** `https://www.googleapis.com/calendar/v3`

---

## Official Documentation

- **API Reference:** [developers.google.com/workspace/calendar/api/v3/reference](https://developers.google.com/workspace/calendar/api/v3/reference)
- **Overview Guide:** [developers.google.com/workspace/calendar/api/guides/overview](https://developers.google.com/workspace/calendar/api/guides/overview)
- **OAuth Scopes:** [developers.google.com/workspace/calendar/api/auth](https://developers.google.com/workspace/calendar/api/auth)
- **Node.js Client:** [googleapis.dev/nodejs/googleapis/latest/calendar](https://googleapis.dev/nodejs/googleapis/latest/calendar/classes/Calendar.html)

---

## OAuth 2.0 Scopes (What We Need)

### For Reading Calendars
**Scope:** `https://www.googleapis.com/auth/calendar.readonly`
- **Permission:** Read-only access
- **Description:** "See and download any calendar you can access"
- **What it does:** View all events across calendars

**Scope:** `https://www.googleapis.com/auth/calendar.events.readonly`
- **Permission:** Read-only for events
- **Description:** View events across all calendars
- **What it does:** Same as above but more specific to events

**Scope:** `https://www.googleapis.com/auth/calendar.freebusy`
- **Permission:** Check availability only
- **Description:** "Check your availability status"
- **What it does:** Query free/busy time without seeing event details

### For Writing/Creating Events
**Scope:** `https://www.googleapis.com/auth/calendar.events`
- **Permission:** Read + Write events
- **Description:** "Allows viewing and editing events across all calendars"
- **What it does:** Create, update, delete events

**Scope:** `https://www.googleapis.com/auth/calendar`
- **Permission:** Full access
- **Description:** "See, edit, share, and permanently delete all the calendars you can access"
- **What it does:** Everything (overkill for our MVP)

### Recommended for Our MVP
```
calendar.events.readonly  (for reading both calendars)
calendar.events           (for creating suggestion events)
```

Or just use:
```
calendar.events  (covers both read and write)
```

---

## Key Endpoints We Need

### 1. List Calendar Events
**GET** `/calendars/{calendarId}/events`

**What it does:** Returns events on the specified calendar

**Parameters:**
- `calendarId` (required) - Calendar identifier (use `'primary'` for user's primary calendar)
- `timeMin` (optional) - Lower bound for event start time (RFC3339 timestamp)
- `timeMax` (optional) - Upper bound for event start time
- `singleEvents` (optional) - Whether to expand recurring events into instances

**Example Request:**
```javascript
GET https://www.googleapis.com/calendar/v3/calendars/primary/events?timeMin=2026-02-15T00:00:00Z&timeMax=2026-03-01T00:00:00Z
```

**Response:** Array of event objects with `start`, `end`, `summary`, `location`, etc.

**Use case:** Find existing events to determine busy times

---

### 2. Create Calendar Event
**POST** `/calendars/{calendarId}/events`

**What it does:** Creates a new event

**Request Body:**
```json
{
  "summary": "Suggested: Pottery Workshop",
  "description": "Based on your interest in arts & crafts...",
  "start": {
    "dateTime": "2026-02-22T14:00:00-06:00",
    "timeZone": "America/Chicago"
  },
  "end": {
    "dateTime": "2026-02-22T17:00:00-06:00",
    "timeZone": "America/Chicago"
  },
  "location": "123 Art Street, Austin, TX",
  "transparency": "tentative"
}
```

**Key fields:**
- `summary` - Event title (what user sees)
- `description` - Details, match reason, links
- `start.dateTime` - ISO 8601 timestamp
- `end.dateTime` - ISO 8601 timestamp
- `location` - Address (appears in calendar)
- `transparency` - Set to `"tentative"` so it doesn't block time

**Response:** Created event object with `id`

**Use case:** Insert suggestions into both partners' calendars

---

### 3. Check Free/Busy Time
**POST** `/freeBusy`

**What it does:** Returns free/busy information for multiple calendars

**Request Body:**
```json
{
  "timeMin": "2026-02-22T00:00:00Z",
  "timeMax": "2026-02-23T00:00:00Z",
  "items": [
    {"id": "partner1@gmail.com"},
    {"id": "partner2@gmail.com"}
  ]
}
```

**Response:**
```json
{
  "calendars": {
    "partner1@gmail.com": {
      "busy": [
        {"start": "2026-02-22T09:00:00Z", "end": "2026-02-22T10:00:00Z"},
        {"start": "2026-02-22T14:00:00Z", "end": "2026-02-22T15:30:00Z"}
      ]
    },
    "partner2@gmail.com": {
      "busy": [...]
    }
  }
}
```

**Use case:** Find overlapping free time without reading full event details

**Alternative:** We can also just use `events.list` and calculate free time ourselves

---

## Node.js Implementation (googleapis package)

### Installation
```bash
npm install googleapis
```

### Basic Setup
```javascript
const { google } = require('googleapis');
const calendar = google.calendar('v3');

// With OAuth2 client (after auth flow)
const oauth2Client = new google.auth.OAuth2(
  CLIENT_ID,
  CLIENT_SECRET,
  REDIRECT_URI
);

oauth2Client.setCredentials({
  access_token: user.accessToken,
  refresh_token: user.refreshToken
});
```

### List Events
```javascript
const events = await calendar.events.list({
  auth: oauth2Client,
  calendarId: 'primary',
  timeMin: new Date('2026-02-15').toISOString(),
  timeMax: new Date('2026-03-01').toISOString(),
  singleEvents: true,
  orderBy: 'startTime'
});

console.log(events.data.items); // Array of events
```

### Create Event
```javascript
const event = await calendar.events.insert({
  auth: oauth2Client,
  calendarId: 'primary',
  requestBody: {
    summary: 'Suggested: Pottery Class',
    description: 'Matches your interest in arts...',
    start: {
      dateTime: '2026-02-22T14:00:00-06:00',
      timeZone: 'America/Chicago'
    },
    end: {
      dateTime: '2026-02-22T17:00:00-06:00',
      timeZone: 'America/Chicago'
    },
    location: '123 Art St, Austin, TX',
    transparency: 'tentative'
  }
});

console.log('Event created:', event.data.htmlLink);
```

### Check Free/Busy
```javascript
const freeBusy = await calendar.freebusy.query({
  auth: oauth2Client,
  requestBody: {
    timeMin: new Date('2026-02-22').toISOString(),
    timeMax: new Date('2026-02-23').toISOString(),
    items: [
      { id: 'partner1@gmail.com' },
      { id: 'partner2@gmail.com' }
    ]
  }
});

console.log(freeBusy.data.calendars);
```

---

## Rate Limits

**Quota:** 1,000,000 requests per day (per project)
**Per-user limit:** 10 queries per second (QPS)

**For our MVP:** Rate limits are NOT a concern. We're only making a handful of requests per couple per day.

---

## What's Confirmed for Our MVP

✅ **OAuth 2.0 integration** - Well-documented, works as expected
✅ **Reading calendar events** - `events.list` endpoint confirmed
✅ **Creating calendar events** - `events.insert` endpoint confirmed
✅ **Free/busy queries** - `freebusy.query` endpoint confirmed (optional for us)
✅ **Node.js client library** - `googleapis` npm package is official and maintained
✅ **Tentative events** - `transparency: "tentative"` field is supported

---

## Potential Issues / Gotchas

⚠️ **OAuth token refresh** - Access tokens expire in ~1 hour, need to handle refresh
⚠️ **Timezone handling** - Must include timezone in dateTime fields
⚠️ **Event transparency** - Setting to `"tentative"` doesn't block time, but still shows on calendar
⚠️ **Calendar access** - Users must grant permissions explicitly during OAuth flow

---

## Sources

- [API Reference | Google Calendar | Google for Developers](https://developers.google.com/workspace/calendar/api/v3/reference)
- [Google Calendar API overview | Google for Developers](https://developers.google.com/workspace/calendar/api/guides/overview)
- [Choose Google Calendar API scopes | Google for Developers](https://developers.google.com/workspace/calendar/api/auth)
- [googleapis documentation](https://googleapis.dev/nodejs/googleapis/latest/calendar/classes/Calendar.html)
