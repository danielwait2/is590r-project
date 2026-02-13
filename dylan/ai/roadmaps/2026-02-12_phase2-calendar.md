# Phase 2: Calendar Integration - Detailed Implementation Plan

**Timeline:** Week 3 (7 days)  
**Status:** Not Started  
**Roadmap:** [2026-02-12_roadmap-phase2.md](./2026-02-12_roadmap-phase2.md) - Track progress

---

## Overview

Integrate Google Calendar API to read user's calendar events via OAuth. Store calendar tokens securely in Supabase and display upcoming events in the dashboard. This enables Phase 4 (free time detection).

**What gets built:**
- Google Calendar API OAuth flow
- Calendar sync functionality (read events)
- Secure token storage (encrypted)
- Calendar events display in dashboard
- Manual sync button

---

## Success Criteria

- [ ] User can click "Connect Calendar" button
- [ ] OAuth flow redirects to Google Calendar permission screen
- [ ] User grants calendar access
- [ ] App receives and stores calendar tokens (encrypted)
- [ ] App fetches next 2 weeks of calendar events
- [ ] Dashboard displays upcoming events with times
- [ ] User can manually refresh calendar data
- [ ] Tokens persist across sessions

---

## Prerequisites

**From Phase 1:**
- [ ] Next.js app deployed and working
- [ ] Supabase Auth working (Google OAuth)
- [ ] User can sign in and see dashboard
- [ ] Database schema created

**New Setup:**
- [ ] Google Calendar API enabled in Google Cloud Console
- [ ] Calendar scopes added to OAuth consent screen

---

## Detailed Implementation Steps

### Day 1: Google Calendar API Setup

#### 1. Enable Calendar API in Google Cloud

1. Go to Google Cloud Console
2. Select your project ("Family Time MVP")
3. APIs & Services → Library
4. Search "Google Calendar API"
5. Click "Enable"

#### 2. Update OAuth Consent Screen

1. Go to OAuth consent screen
2. Edit app
3. Add scope:
   - `https://www.googleapis.com/auth/calendar.events` (read/write calendar events)
4. Save

#### 3. Update OAuth Credentials

Already configured in Phase 1, but verify:
- Authorized redirect URIs include:
  - `http://localhost:3000/auth/callback`
  - `https://[your-app].vercel.app/auth/callback`

#### 4. Install Google APIs Client

```bash
npm install googleapis
```

**Reference:** `ai/guides/google-calendar-api.md` for complete setup

---

### Day 1-2: Update Auth Flow for Calendar Scope

#### 1. Update Supabase Auth Sign-In

Modify `app/auth/sign-in/route.ts` to request calendar scope:

```typescript
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export async function POST(request: Request) {
  const supabase = createClient()
  const origin = request.headers.get('origin')

  const { error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: `${origin}/auth/callback`,
      queryParams: {
        access_type: 'offline',
        prompt: 'consent',
      },
      scopes: 'openid email profile https://www.googleapis.com/auth/calendar.events',
    },
  })

  if (error) {
    return redirect('/error')
  }

  return redirect('/dashboard')
}
```

**Key changes:**
- Added `scopes` parameter with calendar.events scope
- This requests calendar access during initial sign-in

#### 2. Handle Calendar Token Storage

When user completes OAuth, Supabase stores the provider token. We need to save it separately for calendar access.

Update `app/auth/callback/route.ts`:

```typescript
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const requestUrl = new URL(request.url)
  const code = requestUrl.searchParams.get('code')

  if (code) {
    const supabase = createClient()
    const { data: { session } } = await supabase.auth.exchangeCodeForSession(code)

    if (session?.provider_token && session?.provider_refresh_token) {
      // Store calendar tokens in users table
      await supabase
        .from('users')
        .update({
          google_calendar_token: session.provider_token,
          google_calendar_refresh_token: session.provider_refresh_token,
        })
        .eq('id', session.user.id)
    }
  }

  return NextResponse.redirect(requestUrl.origin + '/dashboard')
}
```

---

### Day 2-3: Build Calendar Sync Function

#### 1. Create Calendar API Helper

`lib/google-calendar/client.ts`:

```typescript
import { google } from 'googleapis'

export function getCalendarClient(accessToken: string) {
  const oauth2Client = new google.auth.OAuth2(
    process.env.GOOGLE_CLIENT_ID,
    process.env.GOOGLE_CLIENT_SECRET
  )

  oauth2Client.setCredentials({
    access_token: accessToken,
  })

  return google.calendar({ version: 'v3', auth: oauth2Client })
}

export async function fetchCalendarEvents(
  accessToken: string,
  timeMin?: Date,
  timeMax?: Date
) {
  const calendar = getCalendarClient(accessToken)

  const now = timeMin || new Date()
  const twoWeeksFromNow = timeMax || new Date(now.getTime() + 14 * 24 * 60 * 60 * 1000)

  try {
    const response = await calendar.events.list({
      calendarId: 'primary',
      timeMin: now.toISOString(),
      timeMax: twoWeeksFromNow.toISOString(),
      singleEvents: true,
      orderBy: 'startTime',
      maxResults: 50,
    })

    return response.data.items || []
  } catch (error: any) {
    console.error('Error fetching calendar events:', error)
    
    // Check if token expired
    if (error.code === 401) {
      throw new Error('CALENDAR_TOKEN_EXPIRED')
    }
    
    throw error
  }
}
```

**Reference:** `ai/guides/google-calendar-api.md` for full API documentation

#### 2. Create Calendar Sync API Route

`app/api/calendar/sync/route.ts`:

```typescript
import { createClient } from '@/lib/supabase/server'
import { fetchCalendarEvents } from '@/lib/google-calendar/client'
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  const supabase = createClient()
  
  // Get authenticated user
  const { data: { user }, error: authError } = await supabase.auth.getUser()
  if (authError || !user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Get calendar tokens
  const { data: userData } = await supabase
    .from('users')
    .select('google_calendar_token, google_calendar_refresh_token')
    .eq('id', user.id)
    .single()

  if (!userData?.google_calendar_token) {
    return NextResponse.json({ error: 'Calendar not connected' }, { status: 400 })
  }

  try {
    // Fetch events from Google Calendar
    const events = await fetchCalendarEvents(userData.google_calendar_token)

    return NextResponse.json({
      success: true,
      events,
      count: events.length,
    })
  } catch (error: any) {
    if (error.message === 'CALENDAR_TOKEN_EXPIRED') {
      // TODO: Implement token refresh in Phase 4
      return NextResponse.json({ error: 'Calendar access expired. Please reconnect.' }, { status: 401 })
    }

    console.error('Calendar sync error:', error)
    return NextResponse.json({ error: 'Failed to sync calendar' }, { status: 500 })
  }
}
```

---

### Day 3-4: Update Dashboard UI

#### 1. Add Connect Calendar Button

Update `app/dashboard/page.tsx` to show calendar connection status:

```typescript
import { createClient } from '@/lib/supabase/server'

export default async function DashboardPage() {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()

  const { data: userData } = await supabase
    .from('users')
    .select('*, interests(*), trusted_sitters(*)')
    .eq('id', user!.id)
    .single()

  const calendarConnected = !!userData?.google_calendar_token

  return (
    <div className="space-y-8">
      <div>
        <h2 className="text-2xl font-bold mb-4">
          Welcome, {userData?.name || 'there'}!
        </h2>
        <p className="text-gray-600">
          {calendarConnected 
            ? "Your calendar is connected. We'll start suggesting activities soon!"
            : "Connect your calendar to get personalized activity suggestions."}
        </p>
      </div>

      {/* Calendar Connection Card */}
      {!calendarConnected ? (
        <div className="bg-blue-50 border border-blue-200 rounded-lg p-6">
          <div className="flex items-start gap-4">
            <div className="flex-shrink-0">
              <svg className="w-12 h-12 text-blue-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" />
              </svg>
            </div>
            <div className="flex-1">
              <h3 className="text-lg font-semibold text-gray-900 mb-2">
                Connect Your Google Calendar
              </h3>
              <p className="text-gray-700 mb-4">
                We'll analyze your free time and suggest activities that match your interests.
                Your calendar events stay private—we only detect time gaps.
              </p>
              <form action="/api/calendar/connect" method="post">
                <button 
                  type="submit"
                  className="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition"
                >
                  Connect Calendar
                </button>
              </form>
            </div>
          </div>
        </div>
      ) : (
        <CalendarEventsDisplay />
      )}

      {/* Want to Try List */}
      {userData?.interests && userData.interests.length > 0 && (
        <div className="bg-white rounded-lg p-6 border">
          <h3 className="font-medium mb-4">Your Want to Try List</h3>
          <ul className="space-y-2">
            {userData.interests.map((interest: any) => (
              <li key={interest.id} className="flex items-center gap-2">
                <span className="text-blue-600">→</span>
                {interest.name}
              </li>
            ))}
          </ul>
        </div>
      )}
    </div>
  )
}
```

#### 2. Create Calendar Connect API Route

`app/api/calendar/connect/route.ts`:

```typescript
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export async function POST(request: Request) {
  const supabase = createClient()
  const origin = request.headers.get('origin')

  // This will re-trigger OAuth with calendar scope
  // (should already have it from initial sign-in, but ensures it)
  const { error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: `${origin}/auth/callback`,
      queryParams: {
        access_type: 'offline',
        prompt: 'consent',
      },
      scopes: 'openid email profile https://www.googleapis.com/auth/calendar.events',
    },
  })

  if (error) {
    return redirect('/dashboard?error=calendar_connect_failed')
  }

  return redirect('/dashboard')
}
```

---

### Day 4-5: Display Calendar Events

#### 1. Create Calendar Events Client Component

`app/dashboard/calendar-events.tsx`:

```typescript
'use client'

import { useEffect, useState } from 'react'
import { format, parseISO } from 'date-fns'

interface CalendarEvent {
  id: string
  summary: string
  start: { dateTime?: string; date?: string }
  end: { dateTime?: string; date?: string }
}

export default function CalendarEventsDisplay() {
  const [events, setEvents] = useState<CalendarEvent[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  const syncCalendar = async () => {
    setLoading(true)
    setError(null)

    try {
      const response = await fetch('/api/calendar/sync', {
        method: 'POST',
      })

      if (!response.ok) {
        const data = await response.json()
        throw new Error(data.error || 'Failed to sync calendar')
      }

      const data = await response.json()
      setEvents(data.events)
    } catch (err: any) {
      setError(err.message)
    } finally {
      setLoading(false)
    }
  }

  useEffect(() => {
    syncCalendar()
  }, [])

  if (loading) {
    return (
      <div className="bg-white rounded-lg p-6 border">
        <div className="animate-pulse">
          <div className="h-4 bg-gray-200 rounded w-1/4 mb-4"></div>
          <div className="space-y-3">
            <div className="h-12 bg-gray-100 rounded"></div>
            <div className="h-12 bg-gray-100 rounded"></div>
            <div className="h-12 bg-gray-100 rounded"></div>
          </div>
        </div>
      </div>
    )
  }

  if (error) {
    return (
      <div className="bg-red-50 border border-red-200 rounded-lg p-6">
        <h3 className="text-red-800 font-medium mb-2">Calendar Error</h3>
        <p className="text-red-700 text-sm mb-4">{error}</p>
        <button
          onClick={syncCalendar}
          className="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
        >
          Try Again
        </button>
      </div>
    )
  }

  return (
    <div className="bg-white rounded-lg p-6 border">
      <div className="flex items-center justify-between mb-4">
        <h3 className="font-medium">Your Calendar (Next 2 Weeks)</h3>
        <button
          onClick={syncCalendar}
          className="text-sm text-blue-600 hover:text-blue-700"
        >
          Refresh
        </button>
      </div>

      {events.length === 0 ? (
        <p className="text-gray-500 text-sm">No upcoming events found.</p>
      ) : (
        <div className="space-y-3">
          {events.map((event) => {
            const start = event.start.dateTime 
              ? parseISO(event.start.dateTime)
              : event.start.date 
              ? parseISO(event.start.date)
              : null

            return (
              <div key={event.id} className="flex items-start gap-3 p-3 bg-gray-50 rounded-lg">
                <div className="flex-shrink-0 w-16 text-center">
                  {start && (
                    <>
                      <div className="text-xs text-gray-500 uppercase">
                        {format(start, 'MMM')}
                      </div>
                      <div className="text-xl font-bold text-gray-900">
                        {format(start, 'd')}
                      </div>
                      {event.start.dateTime && (
                        <div className="text-xs text-gray-500">
                          {format(start, 'h:mm a')}
                        </div>
                      )}
                    </>
                  )}
                </div>
                <div className="flex-1 min-w-0">
                  <p className="font-medium text-gray-900 truncate">
                    {event.summary || 'Untitled Event'}
                  </p>
                  {event.start.dateTime && event.end.dateTime && (
                    <p className="text-sm text-gray-500">
                      {format(parseISO(event.end.dateTime), 'h:mm a')}
                    </p>
                  )}
                </div>
              </div>
            )
          })}
        </div>
      )}

      <p className="text-xs text-gray-500 mt-4">
        <strong>Privacy:</strong> We only use this to detect free time. Event details stay private.
      </p>
    </div>
  )
}
```

#### 2. Update Dashboard to Use Component

Update `app/dashboard/page.tsx`:

```typescript
import CalendarEventsDisplay from './calendar-events'

// ... (in the JSX, replace the CalendarEventsDisplay placeholder)
```

---

### Day 5-6: Token Encryption

#### 1. Install Encryption Library

```bash
npm install @supabase/supabase-js
```

Supabase doesn't automatically encrypt custom columns. For MVP, we'll store tokens as-is but acknowledge the security concern.

**For production (post-MVP):** Implement encryption using a library like `crypto`:

```typescript
import crypto from 'crypto'

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY! // 32-byte key
const IV_LENGTH = 16

export function encrypt(text: string): string {
  const iv = crypto.randomBytes(IV_LENGTH)
  const cipher = crypto.createCipheriv('aes-256-cbc', Buffer.from(ENCRYPTION_KEY), iv)
  let encrypted = cipher.update(text)
  encrypted = Buffer.concat([encrypted, cipher.final()])
  return iv.toString('hex') + ':' + encrypted.toString('hex')
}

export function decrypt(text: string): string {
  const parts = text.split(':')
  const iv = Buffer.from(parts.shift()!, 'hex')
  const encryptedText = Buffer.from(parts.join(':'), 'hex')
  const decipher = crypto.createDecipheriv('aes-256-cbc', Buffer.from(ENCRYPTION_KEY), iv)
  let decrypted = decipher.update(encryptedText)
  decrypted = Buffer.concat([decrypted, decipher.final()])
  return decrypted.toString()
}
```

**Note:** For Phase 2 MVP, skip encryption. Add TODO comment for Phase 6 security hardening.

---

### Day 6-7: Testing & Edge Cases

#### 1. Test Calendar Connection

- [ ] Click "Connect Calendar" button
- [ ] Verify Google OAuth screen appears
- [ ] Grant calendar access
- [ ] Verify redirect back to dashboard
- [ ] Check that `google_calendar_token` is saved in database
- [ ] Verify calendar events are displayed

#### 2. Test Calendar Sync

- [ ] Click "Refresh" button
- [ ] Verify events update
- [ ] Add a new event in Google Calendar
- [ ] Refresh in app
- [ ] Verify new event appears

#### 3. Test Edge Cases

- [ ] User with no upcoming events (should show "No upcoming events")
- [ ] User with 50+ events (should show first 50)
- [ ] User with all-day events (should display correctly)
- [ ] User revokes calendar access in Google (should show error)
- [ ] Token expires (should show error with "Reconnect" option)

#### 4. Test Error Handling

- [ ] Disconnect internet, try to sync (should show error)
- [ ] Manually delete `google_calendar_token` from database, try to sync (should prompt to reconnect)
- [ ] Check browser console for errors

---

## Technical Stack & Tools

**New in Phase 2:**
- `googleapis` npm package (Google Calendar API client)
- Google Calendar API v3
- OAuth 2.0 with calendar scope

**References:**
- `ai/guides/google-calendar-api.md` - Complete Calendar API guide
- `ai/guides/supabase-api.md` - Token storage

---

## File Structure Created

```
app/
├── api/
│   └── calendar/
│       ├── connect/
│       │   └── route.ts          # Initiate calendar OAuth
│       └── sync/
│           └── route.ts          # Sync calendar events
├── dashboard/
│   ├── calendar-events.tsx       # Display calendar events (client component)
│   └── page.tsx                  # Updated with calendar UI
lib/
└── google-calendar/
    └── client.ts                 # Calendar API helper functions
```

---

## Testing Strategy

### Manual Testing
- [ ] User can connect calendar via OAuth
- [ ] Calendar events display correctly
- [ ] Refresh button works
- [ ] All-day events display correctly
- [ ] Events are sorted chronologically
- [ ] No sensitive event details are logged

### Database Verification
- [ ] Check `users` table for `google_calendar_token`
- [ ] Verify token is valid (not expired)
- [ ] Confirm RLS policies allow user to read their own tokens

---

## Risks & Mitigation

### Risk 1: Token Expires
**Impact:** High - calendar stops working  
**Mitigation:** Store refresh token, implement refresh logic in Phase 4  
**Contingency:** Show "Reconnect Calendar" button

### Risk 2: User Revokes Access
**Impact:** Medium - calendar stops working  
**Mitigation:** Handle 401 errors gracefully, prompt to reconnect  
**Contingency:** User can reconnect anytime

### Risk 3: Rate Limiting
**Impact:** Low - only affects frequent manual syncs  
**Mitigation:** Cache events, limit sync frequency  
**Contingency:** Show cached data

---

## Deliverables

- [ ] Calendar OAuth flow working
- [ ] Calendar events displayed in dashboard
- [ ] Manual sync button
- [ ] Error handling for expired tokens
- [ ] Privacy notice displayed
- [ ] Tests passing

---

## Definition of Done

Phase 2 is complete when:

- ✅ User can connect Google Calendar
- ✅ App fetches and displays next 2 weeks of events
- ✅ User can manually refresh calendar data
- ✅ Tokens stored securely in Supabase
- ✅ Error handling for expired/revoked tokens
- ✅ No console errors
- ✅ All success criteria met
- ✅ Ready for Phase 3 (Activity Discovery)

**Next Phase:** Phase 3 - Activity Discovery (Week 4)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026
