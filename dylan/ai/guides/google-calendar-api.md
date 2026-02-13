# Google Calendar API v3 - Implementation Guide

**Source:** https://developers.google.com/calendar/api/v3/reference  
**Official Docs:** https://developers.google.com/calendar  
**Last Updated:** February 12, 2026  
**Status:** ✅ VERIFIED - Real API, production-ready

---

## Overview

Google Calendar API v3 allows you to:
- ✅ Read calendar events
- ✅ Create/update/delete events
- ✅ Detect free/busy times
- ✅ Set reminders and notifications
- ✅ Manage multiple calendars
- ✅ OAuth 2.0 authentication

**Base URL:** `https://www.googleapis.com/calendar/v3`

---

## Authentication

### OAuth 2.0 Setup (Required)

**Step 1: Create Google Cloud Project**
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project
3. Enable Google Calendar API
4. Configure OAuth consent screen

**Step 2: Create OAuth 2.0 Credentials**
1. Go to Google Auth platform → Clients
2. Create OAuth 2.0 Client ID
3. Application type: Web application
4. Add authorized redirect URIs (e.g., `http://localhost:3000/api/auth/callback`)
5. Download credentials JSON

**Step 3: Install Google APIs Client Library**

```bash
npm install googleapis @google-cloud/local-auth
```

### Node.js Authentication Example

```javascript
import { google } from 'googleapis';
import { authenticate } from '@google-cloud/local-auth';

// OAuth 2.0 scopes
const SCOPES = [
  'https://www.googleapis.com/auth/calendar.readonly',  // Read calendars
  'https://www.googleapis.com/auth/calendar.events'     // Write events
];

// Authenticate
const auth = await authenticate({
  scopes: SCOPES,
  keyfilePath: 'credentials.json'
});

// Create calendar client
const calendar = google.calendar({ version: 'v3', auth });
```

### Next.js + NextAuth Example

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';

const handler = NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      authorization: {
        params: {
          scope: 'openid email profile https://www.googleapis.com/auth/calendar.events',
          access_type: 'offline',
          prompt: 'consent'
        }
      }
    })
  ],
  callbacks: {
    async jwt({ token, account }) {
      if (account) {
        token.accessToken = account.access_token;
        token.refreshToken = account.refresh_token;
      }
      return token;
    }
  }
});

export { handler as GET, handler as POST };
```

---

## Core Methods for Our Project

### 1. List Events (Read Calendar)

**Use case:** Detect free time by reading existing events

```javascript
const calendar = google.calendar({ version: 'v3', auth });

const response = await calendar.events.list({
  calendarId: 'primary',
  timeMin: new Date().toISOString(),
  timeMax: new Date(Date.now() + 14 * 24 * 60 * 60 * 1000).toISOString(), // Next 2 weeks
  singleEvents: true,
  orderBy: 'startTime'
});

const events = response.data.items;

// Find gaps (free time) between events
function findFreeTime(events) {
  const freeSlots = [];
  for (let i = 0; i < events.length - 1; i++) {
    const currentEnd = new Date(events[i].end.dateTime);
    const nextStart = new Date(events[i + 1].start.dateTime);
    const gapHours = (nextStart - currentEnd) / (1000 * 60 * 60);
    
    if (gapHours >= 3) {  // 3+ hour gap
      freeSlots.push({
        start: currentEnd,
        end: nextStart,
        duration: gapHours
      });
    }
  }
  return freeSlots;
}
```

### 2. Insert Event (Create Suggestion)

**Use case:** Add activity suggestion to user's calendar

```javascript
await calendar.events.insert({
  calendarId: 'primary',
  requestBody: {
    summary: '💡 Try: Kayaking at Lady Bird Lake',
    description: `You wanted to try kayaking!
    
📍 Lady Bird Lake Rowing Club
⏰ 2:00 PM - 5:00 PM
💰 $45 per person for 2-hour rental
🔗 Book: https://example.com/kayaking

Why we suggested this: You added kayaking to your Want to Try list 3 months ago.

🚼 Babysitter: Request sent to Sarah and Mom`,
    location: '2418 Stratford Dr, Austin, TX 78746',
    start: {
      dateTime: '2026-02-22T14:00:00-06:00',
      timeZone: 'America/Chicago'
    },
    end: {
      dateTime: '2026-02-22T17:00:00-06:00',
      timeZone: 'America/Chicago'
    },
    attendees: [
      { email: 'partner@example.com' }
    ],
    reminders: {
      useDefault: false,
      overrides: [
        { method: 'popup', minutes: 1440 },  // 1 day before
        { method: 'popup', minutes: 120 }     // 2 hours before
      ]
    },
    colorId: '9'  // Blue color (suggestions are blue)
  }
});
```

### 3. Update Event (Confirm Babysitter)

**Use case:** Update event description after babysitter confirms

```javascript
await calendar.events.patch({
  calendarId: 'primary',
  eventId: 'event_id_here',
  requestBody: {
    description: `You wanted to try kayaking!
    
📍 Lady Bird Lake Rowing Club
⏰ 2:00 PM - 5:00 PM
💰 $45 per person
🔗 Book: https://example.com/kayaking

✅ Babysitter confirmed: Sarah will be there at 1:30 PM!`
  }
});
```

### 4. Delete Event (User Declined Suggestion)

**Use case:** Track when user removes suggestion from calendar

```javascript
try {
  await calendar.events.delete({
    calendarId: 'primary',
    eventId: 'event_id_here'
  });
  console.log('Event deleted - user declined suggestion');
} catch (error) {
  console.error('Event already deleted or not found');
}
```

### 5. Watch for Calendar Changes (Webhooks)

**Use case:** Get notified when user accepts/declines suggestions

```javascript
// Set up webhook
await calendar.events.watch({
  calendarId: 'primary',
  requestBody: {
    id: 'unique-channel-id',
    type: 'web_hook',
    address: 'https://yourdomain.com/api/webhooks/calendar'
  }
});

// Webhook endpoint receives notifications when calendar changes
// POST /api/webhooks/calendar
export async function POST(request) {
  const channelId = request.headers.get('x-goog-channel-id');
  const resourceId = request.headers.get('x-goog-resource-id');
  const resourceState = request.headers.get('x-goog-resource-state');
  
  if (resourceState === 'sync') {
    // Initial sync notification
    return Response.json({ received: true });
  }
  
  if (resourceState === 'exists') {
    // Calendar was updated - re-fetch events to see what changed
    const events = await calendar.events.list({
      calendarId: 'primary',
      updatedMin: new Date(Date.now() - 60000).toISOString()  // Last minute
    });
    
    // Check if our suggestion event still exists
    // If deleted → user declined
    // If still there → user accepted
  }
  
  return Response.json({ received: true });
}
```

---

## Event Resource Structure (What You Can Set)

```javascript
{
  // Basic info
  summary: string,              // Event title (required)
  description: string,          // Event description (HTML supported)
  location: string,             // Free-text location
  
  // Timing
  start: {
    dateTime: string,           // RFC3339 format: "2026-02-22T14:00:00-06:00"
    timeZone: string            // IANA timezone: "America/Chicago"
  },
  end: {
    dateTime: string,
    timeZone: string
  },
  
  // Attendees
  attendees: [
    {
      email: string,            // Required
      displayName: string,
      responseStatus: 'needsAction' | 'accepted' | 'declined' | 'tentative'
    }
  ],
  
  // Reminders
  reminders: {
    useDefault: boolean,
    overrides: [
      {
        method: 'email' | 'popup',
        minutes: number         // Minutes before event
      }
    ]
  },
  
  // Visual
  colorId: string,              // "1" = lavender, "9" = blue, etc.
  
  // Metadata
  extendedProperties: {
    private: { key: 'value' },  // Only you can see
    shared: { key: 'value' }    // All attendees can see
  }
}
```

### Available Event Colors

```javascript
const EVENT_COLORS = {
  '1': 'Lavender',
  '2': 'Sage',
  '3': 'Grape',
  '4': 'Flamingo',
  '5': 'Banana',
  '6': 'Tangerine',
  '7': 'Peacock',
  '8': 'Graphite',
  '9': 'Blueberry',  // Recommended for suggestions
  '10': 'Basil',
  '11': 'Tomato'
};
```

---

## Rate Limits & Quotas

**Free tier quotas:**
- 1,000,000 queries per day (more than enough)
- 10 queries per second per user (QPU)

**Incremental sync** (recommended):
- Reduces API calls by only fetching changes since last sync
- Use `syncToken` returned from previous request

```javascript
// First sync
const response1 = await calendar.events.list({
  calendarId: 'primary',
  timeMin: new Date().toISOString()
});
const syncToken = response1.data.nextSyncToken;

// Subsequent syncs (only get changes)
const response2 = await calendar.events.list({
  calendarId: 'primary',
  syncToken: syncToken
});
```

---

## Best Practices for Our Project

### 1. Privacy: Don't Store Event Details

```javascript
// ✅ GOOD: Only store what we need
const freeTime = events.map(event => ({
  start: event.start.dateTime,
  end: event.end.dateTime,
  isBusy: true
}));

// ❌ BAD: Don't store
// event.summary (could be private: "Doctor appointment")
// event.description (sensitive info)
// event.attendees (other people's emails)
```

### 2. Use Extended Properties to Track Suggestions

```javascript
// When creating suggestion, add metadata
await calendar.events.insert({
  // ...
  extendedProperties: {
    private: {
      suggestionId: 'sug_123',          // Our internal ID
      activityId: 'act_456',            // Activity from our database
      createdByApp: 'FamilyTimeApp',
      version: '1.0'
    }
  }
});

// Later, identify which events are our suggestions
const events = await calendar.events.list({ calendarId: 'primary' });
const ourSuggestions = events.data.items.filter(event => 
  event.extendedProperties?.private?.createdByApp === 'FamilyTimeApp'
);
```

### 3. Handle Multiple Calendars

```javascript
// Get all calendars for user
const calendarList = await calendar.calendarList.list();

// For each calendar, check for events
for (const cal of calendarList.data.items) {
  const events = await calendar.events.list({
    calendarId: cal.id,
    timeMin: new Date().toISOString()
  });
  // Process events...
}
```

### 4. Error Handling

```javascript
try {
  await calendar.events.insert({...});
} catch (error) {
  if (error.code === 401) {
    // Token expired - refresh it
  } else if (error.code === 403) {
    // Permission denied - user revoked access
  } else if (error.code === 429) {
    // Rate limit - back off and retry
  }
}
```

---

## Testing in Development

### Using OAuth Playground

1. Go to [OAuth Playground](https://developers.google.com/oauthplayground/)
2. Select "Calendar API v3"
3. Authorize APIs
4. Exchange authorization code for tokens
5. Use access token to test API calls

### Test Account

For development, create a test Google account and add sample events to test free time detection.

---

## Cost

**FREE** - Google Calendar API has no costs (within quotas)

**Quotas:**
- 1M queries/day (way more than needed)
- 10 queries/second per user

---

## Key Methods Reference

### events.list()
```javascript
calendar.events.list({
  calendarId: 'primary' | 'email@example.com',
  timeMin: string,      // RFC3339: "2026-02-12T00:00:00Z"
  timeMax: string,
  maxResults: number,   // Max 2500
  singleEvents: boolean,
  orderBy: 'startTime' | 'updated',
  syncToken: string     // For incremental sync
})
```

### events.insert()
```javascript
calendar.events.insert({
  calendarId: 'primary',
  requestBody: { /* Event object */ },
  sendUpdates: 'all' | 'none' | 'externalOnly'
})
```

### events.patch()
```javascript
calendar.events.patch({
  calendarId: 'primary',
  eventId: string,
  requestBody: { /* Partial event object */ }
})
```

### events.delete()
```javascript
calendar.events.delete({
  calendarId: 'primary',
  eventId: string,
  sendUpdates: 'all' | 'none' | 'externalOnly'
})
```

### events.watch()
```javascript
calendar.events.watch({
  calendarId: 'primary',
  requestBody: {
    id: string,           // Unique channel ID
    type: 'web_hook',
    address: string       // Your webhook URL
  }
})
```

### freebusy.query()
```javascript
calendar.freebusy.query({
  requestBody: {
    timeMin: string,
    timeMax: string,
    items: [
      { id: 'primary' },
      { id: 'partner@example.com' }
    ]
  }
})
```

---

## Implementation Plan for Our MVP

### Week 1: OAuth Setup
1. Create Google Cloud project
2. Enable Calendar API
3. Set up OAuth consent screen
4. Create credentials
5. Test authentication flow

### Week 2: Read Events (Free Time Detection)
1. Implement `events.list()`
2. Parse events to find gaps
3. Filter for 3+ hour gaps
4. Store free time blocks in database

### Week 3: Write Events (Create Suggestions)
1. Implement `events.insert()`
2. Format suggestion with description
3. Set reminders
4. Add to user's calendar

### Week 4: Webhooks (Track Acceptance)
1. Set up `events.watch()`
2. Create webhook endpoint
3. Detect event deletion (user declined)
4. Detect event kept (user accepted)

---

## Common Pitfalls & Solutions

### Problem: Token Expires
**Solution:** Store refresh token, automatically refresh when access token expires

```javascript
const { google } = require('googleapis');
const oauth2Client = new google.auth.OAuth2(
  CLIENT_ID,
  CLIENT_SECRET,
  REDIRECT_URI
);

// Set credentials
oauth2Client.setCredentials({
  refresh_token: stored_refresh_token
});

// Token auto-refreshes when needed
const calendar = google.calendar({ version: 'v3', auth: oauth2Client });
```

### Problem: User Has Multiple Calendars
**Solution:** Ask which calendar(s) to monitor during onboarding

```javascript
// Get all calendars
const list = await calendar.calendarList.list();

// Let user select which ones to monitor
const selectedCalendars = ['primary', 'work@example.com'];

// Query all selected calendars
for (const calId of selectedCalendars) {
  const events = await calendar.events.list({ calendarId: calId });
  // Combine results...
}
```

### Problem: All-Day Events
**Solution:** Handle both `dateTime` and `date` fields

```javascript
function getEventTime(event) {
  // Timed event
  if (event.start.dateTime) {
    return new Date(event.start.dateTime);
  }
  // All-day event
  if (event.start.date) {
    return new Date(event.start.date + 'T00:00:00');
  }
}
```

---

## Security Best Practices

### 1. Store Tokens Securely
```javascript
// ❌ DON'T: Store in plain text
const token = 'ya29.a0AfH6SMB...';

// ✅ DO: Encrypt tokens in database
import { encrypt, decrypt } from './crypto';

const encrypted = encrypt(token);
await db.users.update({
  where: { id: userId },
  data: { googleToken: encrypted }
});
```

### 2. Minimal Scopes
```javascript
// ❌ DON'T: Request full access
const SCOPES = ['https://www.googleapis.com/auth/calendar'];

// ✅ DO: Only request what you need
const SCOPES = [
  'https://www.googleapis.com/auth/calendar.events',  // Read/write events only
  'https://www.googleapis.com/auth/calendar.freebusy' // Read free/busy (optional)
];
```

### 3. Privacy-First Messaging
Show users exactly what you're accessing:
> "We only read free time blocks from your calendar, never event details. Your meetings, appointments, and personal events stay private."

---

## TypeScript Types

```typescript
import { calendar_v3 } from 'googleapis';

type CalendarEvent = calendar_v3.Schema$Event;

interface EventInsertParams {
  calendarId: string;
  requestBody: calendar_v3.Schema$Event;
  sendUpdates?: 'all' | 'none' | 'externalOnly';
}

// Usage
const event: CalendarEvent = {
  summary: 'Activity Suggestion',
  start: {
    dateTime: '2026-02-22T14:00:00-06:00',
    timeZone: 'America/Chicago'
  },
  end: {
    dateTime: '2026-02-22T17:00:00-06:00',
    timeZone: 'America/Chicago'
  }
};
```

---

## Links & Resources

- **API Reference:** https://developers.google.com/calendar/api/v3/reference
- **Node.js Quickstart:** https://developers.google.com/calendar/api/quickstart/nodejs
- **Create Events Guide:** https://developers.google.com/calendar/create-events
- **OAuth Guide:** https://developers.google.com/identity/protocols/oauth2
- **Node.js Samples:** https://github.com/googleworkspace/node-samples

---

## For Our MVP

**What we need:**
- ✅ OAuth 2.0 authentication
- ✅ Read events (`events.list`)
- ✅ Create events (`events.insert`)
- ✅ Update events (`events.patch`)
- ✅ Track deletions (optional: `events.watch`)

**What we can skip:**
- ❌ Multiple calendar support (just primary)
- ❌ Recurring events (treat as separate events)
- ❌ Calendar sharing features
- ❌ Advanced permissions

**Estimated implementation time:** 1-2 weeks for full calendar integration
