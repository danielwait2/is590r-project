# Google Calendar API Documentation

**Library:** Google Calendar API
**Library ID:** `/websites/developers_google_workspace_calendar_api`
**Source:** context7 (February 2026)
**Version:** v3

## Overview

The Google Calendar API allows you to read and write calendar events, manage calendars, and sync calendar data. It uses OAuth 2.0 for authentication and provides RESTful endpoints for calendar operations.

## Key Features for Our Project

- **OAuth 2.0 Authentication** - Secure access to user calendars
- **List Events** - Fetch calendar events for scheduling analysis
- **Create Events** - Add events to user calendars
- **Update/Delete Events** - Modify existing calendar entries
- **Free/Busy Queries** - Find available time slots

## Installation

```bash
npm install googleapis
```

## Setup

### 1. Enable Google Calendar API

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create new project or select existing
3. Enable Google Calendar API
4. Create OAuth 2.0 credentials

### 2. Configure OAuth Consent Screen

1. Go to OAuth consent screen
2. Add scopes: `https://www.googleapis.com/auth/calendar.readonly`
3. Add test users for development

### 3. Create OAuth Client

1. Go to Credentials > Create Credentials > OAuth client ID
2. Application type: Web application
3. Authorized redirect URIs: `http://localhost:3000/api/auth/google/callback`
4. Download credentials JSON

### 4. Environment Variables

```bash
# .env.local
GOOGLE_CLIENT_ID=xxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxx
GOOGLE_REDIRECT_URI=http://localhost:3000/api/auth/google/callback
```

## Authentication Flow

### 1. Generate Authorization URL

```typescript
// lib/google-auth.ts
import { google } from 'googleapis'

const oauth2Client = new google.auth.OAuth2(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  process.env.GOOGLE_REDIRECT_URI
)

export function getAuthUrl() {
  const scopes = [
    'https://www.googleapis.com/auth/calendar.readonly',
    'https://www.googleapis.com/auth/calendar.events'
  ]

  return oauth2Client.generateAuthUrl({
    access_type: 'offline',
    scope: scopes,
    prompt: 'consent'
  })
}

export { oauth2Client }
```

### 2. OAuth Callback Handler

```typescript
// app/api/auth/google/callback/route.ts
import { NextResponse } from 'next/server'
import { auth } from '@clerk/nextjs'
import { oauth2Client } from '@/lib/google-auth'
import { prisma } from '@/lib/prisma'

export async function GET(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.redirect('/sign-in')
  }

  const { searchParams } = new URL(request.url)
  const code = searchParams.get('code')

  if (!code) {
    return NextResponse.json({ error: 'No code provided' }, { status: 400 })
  }

  try {
    const { tokens } = await oauth2Client.getToken(code)

    // Store tokens in database
    await prisma.user.update({
      where: { clerkId: userId },
      data: {
        googleAccessToken: tokens.access_token,
        googleRefreshToken: tokens.refresh_token,
        googleTokenExpiry: tokens.expiry_date ? new Date(tokens.expiry_date) : null,
      }
    })

    return NextResponse.redirect('/dashboard?calendar_connected=true')
  } catch (error) {
    console.error('Error getting tokens:', error)
    return NextResponse.redirect('/dashboard?calendar_error=true')
  }
}
```

### 3. Connect Calendar Button

```tsx
// components/ConnectCalendarButton.tsx
'use client'

import { getAuthUrl } from '@/lib/google-auth'

export default function ConnectCalendarButton() {
  function handleConnect() {
    const authUrl = getAuthUrl()
    window.location.href = authUrl
  }

  return (
    <button onClick={handleConnect}>
      Connect Google Calendar
    </button>
  )
}
```

## List Calendar Events

### Fetch User Events

```typescript
// lib/google-calendar.ts
import { google } from 'googleapis'
import { oauth2Client } from './google-auth'
import { prisma } from './prisma'

export async function getUserCalendarEvents(userId: string, timeMin: Date, timeMax: Date) {
  // Get user's tokens from database
  const user = await prisma.user.findUnique({
    where: { clerkId: userId },
    select: {
      googleAccessToken: true,
      googleRefreshToken: true,
      googleTokenExpiry: true,
    }
  })

  if (!user?.googleAccessToken) {
    throw new Error('User has not connected Google Calendar')
  }

  // Set credentials
  oauth2Client.setCredentials({
    access_token: user.googleAccessToken,
    refresh_token: user.googleRefreshToken,
    expiry_date: user.googleTokenExpiry?.getTime(),
  })

  // Check if token needs refresh
  if (user.googleTokenExpiry && new Date() > user.googleTokenExpiry) {
    const { credentials } = await oauth2Client.refreshAccessToken()

    // Update tokens in database
    await prisma.user.update({
      where: { clerkId: userId },
      data: {
        googleAccessToken: credentials.access_token,
        googleTokenExpiry: credentials.expiry_date ? new Date(credentials.expiry_date) : null,
      }
    })

    oauth2Client.setCredentials(credentials)
  }

  const calendar = google.calendar({ version: 'v3', auth: oauth2Client })

  const response = await calendar.events.list({
    calendarId: 'primary',
    timeMin: timeMin.toISOString(),
    timeMax: timeMax.toISOString(),
    singleEvents: true,
    orderBy: 'startTime',
  })

  return response.data.items || []
}
```

### API Route to Fetch Events

```typescript
// app/api/calendar/events/route.ts
import { auth } from '@clerk/nextjs'
import { NextResponse } from 'next/server'
import { getUserCalendarEvents } from '@/lib/google-calendar'

export async function GET(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { searchParams } = new URL(request.url)
  const timeMinStr = searchParams.get('timeMin')
  const timeMaxStr = searchParams.get('timeMax')

  const timeMin = timeMinStr ? new Date(timeMinStr) : new Date()
  const timeMax = timeMaxStr ? new Date(timeMaxStr) : new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)

  try {
    const events = await getUserCalendarEvents(userId, timeMin, timeMax)

    return NextResponse.json({ events })
  } catch (error) {
    console.error('Error fetching calendar events:', error)
    return NextResponse.json({ error: 'Failed to fetch events' }, { status: 500 })
  }
}
```

## Create Calendar Event

```typescript
// lib/google-calendar.ts
export async function createCalendarEvent(
  userId: string,
  eventData: {
    summary: string
    description?: string
    start: Date
    end: Date
    attendees?: string[]
  }
) {
  const user = await prisma.user.findUnique({
    where: { clerkId: userId },
    select: {
      googleAccessToken: true,
      googleRefreshToken: true,
    }
  })

  if (!user?.googleAccessToken) {
    throw new Error('User has not connected Google Calendar')
  }

  oauth2Client.setCredentials({
    access_token: user.googleAccessToken,
    refresh_token: user.googleRefreshToken,
  })

  const calendar = google.calendar({ version: 'v3', auth: oauth2Client })

  const event = {
    summary: eventData.summary,
    description: eventData.description,
    start: {
      dateTime: eventData.start.toISOString(),
      timeZone: 'America/New_York',
    },
    end: {
      dateTime: eventData.end.toISOString(),
      timeZone: 'America/New_York',
    },
    attendees: eventData.attendees?.map(email => ({ email })),
  }

  const response = await calendar.events.insert({
    calendarId: 'primary',
    requestBody: event,
  })

  return response.data
}
```

### API Route to Create Event

```typescript
// app/api/calendar/events/create/route.ts
import { auth } from '@clerk/nextjs'
import { NextResponse } from 'next/server'
import { createCalendarEvent } from '@/lib/google-calendar'

export async function POST(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const body = await request.json()
  const { summary, description, start, end, attendees } = body

  try {
    const event = await createCalendarEvent(userId, {
      summary,
      description,
      start: new Date(start),
      end: new Date(end),
      attendees,
    })

    return NextResponse.json({ event })
  } catch (error) {
    console.error('Error creating calendar event:', error)
    return NextResponse.json({ error: 'Failed to create event' }, { status: 500 })
  }
}
```

## Find Free/Busy Times

```typescript
// lib/google-calendar.ts
export async function getFreeBusyTimes(
  userId: string,
  timeMin: Date,
  timeMax: Date
) {
  const user = await prisma.user.findUnique({
    where: { clerkId: userId },
    select: {
      googleAccessToken: true,
      googleRefreshToken: true,
    }
  })

  if (!user?.googleAccessToken) {
    throw new Error('User has not connected Google Calendar')
  }

  oauth2Client.setCredentials({
    access_token: user.googleAccessToken,
    refresh_token: user.googleRefreshToken,
  })

  const calendar = google.calendar({ version: 'v3', auth: oauth2Client })

  const response = await calendar.freebusy.query({
    requestBody: {
      timeMin: timeMin.toISOString(),
      timeMax: timeMax.toISOString(),
      items: [{ id: 'primary' }],
    },
  })

  const busyTimes = response.data.calendars?.primary?.busy || []

  return busyTimes.map(slot => ({
    start: new Date(slot.start!),
    end: new Date(slot.end!),
  }))
}
```

## Implementation for Our Project

### Find Available Family Times

```typescript
// lib/scheduling.ts
import { getFreeBusyTimes } from './google-calendar'

export async function findAvailableFamilyTimes(
  userIds: string[],
  timeMin: Date,
  timeMax: Date,
  durationMinutes: number
) {
  // Get busy times for all family members
  const allBusyTimes = await Promise.all(
    userIds.map(userId => getFreeBusyTimes(userId, timeMin, timeMax))
  )

  // Flatten and merge overlapping busy times
  const mergedBusyTimes = mergeBusyTimes(allBusyTimes.flat())

  // Find gaps that fit the duration
  const availableSlots = findAvailableSlots(
    mergedBusyTimes,
    timeMin,
    timeMax,
    durationMinutes
  )

  return availableSlots
}

function mergeBusyTimes(busyTimes: { start: Date; end: Date }[]) {
  if (busyTimes.length === 0) return []

  const sorted = busyTimes.sort((a, b) => a.start.getTime() - b.start.getTime())
  const merged = [sorted[0]]

  for (let i = 1; i < sorted.length; i++) {
    const current = sorted[i]
    const last = merged[merged.length - 1]

    if (current.start <= last.end) {
      last.end = new Date(Math.max(last.end.getTime(), current.end.getTime()))
    } else {
      merged.push(current)
    }
  }

  return merged
}

function findAvailableSlots(
  busyTimes: { start: Date; end: Date }[],
  timeMin: Date,
  timeMax: Date,
  durationMinutes: number
) {
  const availableSlots: { start: Date; end: Date }[] = []
  const durationMs = durationMinutes * 60 * 1000

  let currentTime = timeMin

  for (const busy of busyTimes) {
    const gapDuration = busy.start.getTime() - currentTime.getTime()

    if (gapDuration >= durationMs) {
      availableSlots.push({
        start: currentTime,
        end: busy.start,
      })
    }

    currentTime = busy.end
  }

  // Check final gap
  const finalGap = timeMax.getTime() - currentTime.getTime()
  if (finalGap >= durationMs) {
    availableSlots.push({
      start: currentTime,
      end: timeMax,
    })
  }

  return availableSlots
}
```

### API Route for Available Times

```typescript
// app/api/families/[familyId]/available-times/route.ts
import { auth } from '@clerk/nextjs'
import { NextResponse } from 'next/server'
import { findAvailableFamilyTimes } from '@/lib/scheduling'
import { prisma } from '@/lib/prisma'

export async function GET(
  request: Request,
  { params }: { params: { familyId: string } }
) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { searchParams } = new URL(request.url)
  const timeMinStr = searchParams.get('timeMin')
  const timeMaxStr = searchParams.get('timeMax')
  const duration = parseInt(searchParams.get('duration') || '60')

  const timeMin = timeMinStr ? new Date(timeMinStr) : new Date()
  const timeMax = timeMaxStr ? new Date(timeMaxStr) : new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)

  // Get family members
  const family = await prisma.family.findUnique({
    where: { id: params.familyId },
    include: {
      members: {
        include: {
          user: true
        }
      }
    }
  })

  if (!family) {
    return NextResponse.json({ error: 'Family not found' }, { status: 404 })
  }

  const userIds = family.members.map(m => m.user.clerkId)

  try {
    const availableSlots = await findAvailableFamilyTimes(
      userIds,
      timeMin,
      timeMax,
      duration
    )

    return NextResponse.json({ availableSlots })
  } catch (error) {
    console.error('Error finding available times:', error)
    return NextResponse.json({ error: 'Failed to find available times' }, { status: 500 })
  }
}
```

## Event Types

```typescript
interface CalendarEvent {
  id: string
  summary: string
  description?: string
  start: {
    dateTime?: string
    date?: string
    timeZone?: string
  }
  end: {
    dateTime?: string
    date?: string
    timeZone?: string
  }
  attendees?: {
    email: string
    responseStatus: 'needsAction' | 'declined' | 'tentative' | 'accepted'
  }[]
}
```

## Best Practices for Our App

1. **Token Refresh** - Always check token expiry and refresh when needed
2. **Error Handling** - Handle quota limits and API errors gracefully
3. **Batch Requests** - Use batch API for multiple operations
4. **Timezone Handling** - Always specify timezone in events
5. **Permission Scopes** - Request minimal necessary scopes
6. **User Privacy** - Only access calendar data when absolutely needed
7. **Caching** - Cache calendar data to reduce API calls

## Common Patterns

### Check Calendar Connection

```typescript
const user = await prisma.user.findUnique({
  where: { clerkId: userId },
  select: { googleAccessToken: true }
})

const isConnected = !!user?.googleAccessToken
```

### Disconnect Calendar

```typescript
await prisma.user.update({
  where: { clerkId: userId },
  data: {
    googleAccessToken: null,
    googleRefreshToken: null,
    googleTokenExpiry: null,
  }
})
```

## Resources

- Official Docs: https://developers.google.com/calendar/api/v3/reference
- OAuth 2.0: https://developers.google.com/identity/protocols/oauth2
- Node.js Client: https://github.com/googleapis/google-api-nodejs-client
- Events API: https://developers.google.com/calendar/api/v3/reference/events
- Free/Busy: https://developers.google.com/calendar/api/v3/reference/freebusy
