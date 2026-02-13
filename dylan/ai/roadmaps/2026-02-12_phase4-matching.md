# Phase 4: Free Time Detection & Matching - Detailed Implementation Plan

**Timeline:** Week 5 (7 days)  
**Status:** Not Started  
**Roadmap:** [2026-02-12_roadmap-phase4.md](./2026-02-12_roadmap-phase4.md) - Track progress

---

## Overview

Implement free time detection by analyzing calendar gaps, build activity matching algorithm to connect user's "Want to Try" list with available activities, and insert suggestions directly into Google Calendar. This is the core value proposition of the app.

**What gets built:**
- Free time detection algorithm (3+ hour gaps)
- Activity matching engine (rule-based)
- Suggestion generation logic
- Calendar event insertion with rich descriptions
- Suggestion tracking (accepted/declined)

---

## Success Criteria

- [ ] App detects at least 1 free time block (3+ hours) in next 2 weeks
- [ ] App matches "Want to Try" items with activities in database
- [ ] Suggestion is generated with activity + free time
- [ ] Suggestion appears in user's Google Calendar with full details
- [ ] Event includes: activity name, address, reason (why suggested), babysitter note
- [ ] User can see suggestion in Google Calendar app
- [ ] App tracks if user keeps or deletes suggestion

---

## Prerequisites

**From Previous Phases:**
- [ ] Calendar API working (can read events)
- [ ] Database has 100-150 activities
- [ ] User has "Want to Try" list with at least 1 item
- [ ] Dashboard shows calendar events

---

## Detailed Implementation Steps

### Day 1-2: Free Time Detection

#### 1. Create Free Time Detection Function

`lib/calendar/detect-free-time.ts`:

```typescript
import { differenceInHours, parseISO, format, addDays } from 'date-fns'

export interface FreeTimeBlock {
  start: Date
  end: Date
  durationHours: number
  dayOfWeek: string
  isWeekend: boolean
}

export interface CalendarEvent {
  start: { dateTime?: string; date?: string }
  end: { dateTime?: string; date?: string }
}

export function detectFreeTime(
  events: CalendarEvent[],
  minHours: number = 3
): FreeTimeBlock[] {
  const freeBlocks: FreeTimeBlock[] = []
  
  // Only look at timed events (skip all-day events)
  const timedEvents = events.filter(e => e.start.dateTime && e.end.dateTime)
  
  if (timedEvents.length === 0) {
    // No events = all time is free
    // Return next 7 days with reasonable windows
    const now = new Date()
    for (let i = 0; i < 7; i++) {
      const day = addDays(now, i)
      const start = new Date(day)
      start.setHours(14, 0, 0, 0)  // 2 PM
      const end = new Date(day)
      end.setHours(19, 0, 0, 0)    // 7 PM
      
      freeBlocks.push({
        start,
        end,
        durationHours: 5,
        dayOfWeek: format(start, 'EEEE'),
        isWeekend: start.getDay() === 0 || start.getDay() === 6,
      })
    }
    return freeBlocks
  }
  
  // Find gaps between events
  for (let i = 0; i < timedEvents.length - 1; i++) {
    const currentEnd = parseISO(timedEvents[i].end.dateTime!)
    const nextStart = parseISO(timedEvents[i + 1].start.dateTime!)
    
    const gapHours = differenceInHours(nextStart, currentEnd)
    
    if (gapHours >= minHours) {
      freeBlocks.push({
        start: currentEnd,
        end: nextStart,
        durationHours: gapHours,
        dayOfWeek: format(currentEnd, 'EEEE'),
        isWeekend: currentEnd.getDay() === 0 || currentEnd.getDay() === 6,
      })
    }
  }
  
  // Sort by date (soonest first)
  return freeBlocks.sort((a, b) => a.start.getTime() - b.start.getTime())
}
```

#### 2. Test Free Time Detection

Create test file `scripts/test-free-time.ts`:

```typescript
import { detectFreeTime } from '../lib/calendar/detect-free-time'

// Mock calendar events
const mockEvents = [
  {
    start: { dateTime: '2026-02-15T09:00:00-06:00' },
    end: { dateTime: '2026-02-15T12:00:00-06:00' },
  },
  {
    start: { dateTime: '2026-02-15T17:00:00-06:00' },
    end: { dateTime: '2026-02-15T18:00:00-06:00' },
  },
]

const freeBlocks = detectFreeTime(mockEvents, 3)

console.log('Free time blocks found:', freeBlocks.length)
freeBlocks.forEach(block => {
  console.log(`${block.dayOfWeek}: ${block.start.toLocaleString()} - ${block.end.toLocaleString()} (${block.durationHours}h)`)
})
```

Run: `npx tsx scripts/test-free-time.ts`

---

### Day 2-3: Activity Matching Algorithm

#### 1. Create Matching Function

`lib/matching/match-activities.ts`:

```typescript
import { createClient } from '@/lib/supabase/client'

export interface MatchedActivity {
  activityId: string
  activityName: string
  score: number
  matchReason: string
  activity: any
}

export async function matchActivities(
  userId: string,
  limit: number = 5
): Promise<MatchedActivity[]> {
  const supabase = createClient()
  
  // Get user's Want to Try list
  const { data: interests } = await supabase
    .from('interests')
    .select('*')
    .eq('user_id', userId)
  
  if (!interests || interests.length === 0) {
    return []
  }
  
  // Get user's location
  const { data: user } = await supabase
    .from('users')
    .select('location_city')
    .eq('id', userId)
    .single()
  
  // Get all activities in user's city
  const { data: activities } = await supabase
    .from('activities')
    .select('*')
    .eq('city', user?.location_city || 'Austin')
  
  if (!activities) return []
  
  // Simple matching: keyword match between Want to Try and activity name/category
  const matches: MatchedActivity[] = []
  
  for (const interest of interests) {
    const interestKeywords = interest.name.toLowerCase().split(/\s+/)
    
    for (const activity of activities) {
      const activityText = `${activity.name} ${activity.category}`.toLowerCase()
      
      // Check if any keyword matches
      const matchingKeywords = interestKeywords.filter(keyword => 
        activityText.includes(keyword)
      )
      
      if (matchingKeywords.length > 0) {
        matches.push({
          activityId: activity.id,
          activityName: activity.name,
          score: matchingKeywords.length,
          matchReason: `You wanted to try ${interest.name}`,
          activity,
        })
      }
    }
  }
  
  // Remove duplicates and sort by score
  const uniqueMatches = Array.from(
    new Map(matches.map(m => [m.activityId, m])).values()
  )
  
  return uniqueMatches
    .sort((a, b) => b.score - a.score)
    .slice(0, limit)
}
```

#### 2. Test Matching Algorithm

```typescript
// Test script
const matches = await matchActivities('user-id-here', 10)

console.log(`Found ${matches.length} matches:`)
matches.forEach(match => {
  console.log(`- ${match.activityName} (score: ${match.score})`)
  console.log(`  Reason: ${match.matchReason}`)
})
```

---

### Day 3-4: Suggestion Generation

#### 1. Create Suggestion Generator

`lib/suggestions/generate.ts`:

```typescript
import { createClient } from '@/lib/supabase/server'
import { detectFreeTime } from '@/lib/calendar/detect-free-time'
import { matchActivities } from '@/lib/matching/match-activities'
import { format, addHours } from 'date-fns'

export async function generateSuggestions(userId: string) {
  const supabase = createClient()
  
  // 1. Get user's calendar events
  const { data: userData } = await supabase
    .from('users')
    .select('google_calendar_token')
    .eq('id', userId)
    .single()
  
  if (!userData?.google_calendar_token) {
    throw new Error('Calendar not connected')
  }
  
  // Fetch calendar events
  const events = await fetchCalendarEvents(userData.google_calendar_token)
  
  // 2. Detect free time (3+ hour blocks)
  const freeBlocks = detectFreeTime(events, 3)
  
  if (freeBlocks.length === 0) {
    return { suggestions: [], reason: 'No free time blocks found' }
  }
  
  // 3. Match activities
  const matches = await matchActivities(userId, 10)
  
  if (matches.length === 0) {
    return { suggestions: [], reason: 'No matching activities found' }
  }
  
  // 4. Generate suggestions (pair free time with matched activities)
  const suggestions = []
  
  for (let i = 0; i < Math.min(freeBlocks.length, matches.length, 3); i++) {
    const freeBlock = freeBlocks[i]
    const match = matches[i]
    
    // Determine duration (activity-specific or 3 hours default)
    const duration = match.activity.min_duration_hours || 3
    const suggestedStart = freeBlock.start
    const suggestedEnd = addHours(suggestedStart, duration)
    
    // Only suggest if it fits in the free block
    if (suggestedEnd <= freeBlock.end) {
      suggestions.push({
        freeBlock,
        match,
        suggestedStart,
        suggestedEnd,
        duration,
      })
    }
  }
  
  return { suggestions, reason: 'Generated successfully' }
}
```

---

### Day 4-5: Insert Suggestions into Calendar

#### 1. Create Calendar Event Insertion Function

`lib/google-calendar/insert-event.ts`:

```typescript
import { google } from 'googleapis'
import { format } from 'date-fns'

export async function insertSuggestion(
  accessToken: string,
  activity: any,
  startTime: Date,
  endTime: Date,
  matchReason: string
) {
  const oauth2Client = new google.auth.OAuth2(
    process.env.GOOGLE_CLIENT_ID,
    process.env.GOOGLE_CLIENT_SECRET
  )
  
  oauth2Client.setCredentials({ access_token: accessToken })
  const calendar = google.calendar({ version: 'v3', auth: oauth2Client })
  
  const event = {
    summary: `💡 Try: ${activity.name}`,
    description: `${matchReason}

📍 ${activity.address || 'Address not available'}
⏰ ${format(startTime, 'h:mm a')} - ${format(endTime, 'h:mm a')}
${activity.rating ? `⭐ ${activity.rating}/5 rating` : ''}
${activity.price_level ? `💰 ${' $'.repeat(activity.price_level)} price range` : ''}

${activity.requires_babysitter ? '🚼 Babysitter: We\'ll coordinate when you accept this suggestion' : ''}

This suggestion was added by Family Time App.`,
    location: activity.address || `${activity.city}, ${activity.state}`,
    start: {
      dateTime: startTime.toISOString(),
      timeZone: 'America/Chicago',
    },
    end: {
      dateTime: endTime.toISOString(),
      timeZone: 'America/Chicago',
    },
    colorId: '9',  // Blue color for suggestions
    reminders: {
      useDefault: false,
      overrides: [
        { method: 'popup', minutes: 1440 },  // 1 day before
      ],
    },
    extendedProperties: {
      private: {
        suggestionId: crypto.randomUUID(),
        activityId: activity.id,
        createdByApp: 'FamilyTimeApp',
      },
    },
  }
  
  const response = await calendar.events.insert({
    calendarId: 'primary',
    requestBody: event,
  })
  
  return response.data
}
```

**Reference:** `ai/guides/google-calendar-api.md` for event structure

#### 2. Create Suggestion API Route

`app/api/suggestions/generate/route.ts`:

```typescript
import { createClient } from '@/lib/supabase/server'
import { generateSuggestions } from '@/lib/suggestions/generate'
import { insertSuggestion } from '@/lib/google-calendar/insert-event'
import { NextResponse } from 'next/server'

export async function POST(request: Request) {
  const supabase = createClient()
  
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }
  
  try {
    // Generate suggestions
    const { suggestions, reason } = await generateSuggestions(user.id)
    
    if (suggestions.length === 0) {
      return NextResponse.json({ success: true, count: 0, reason })
    }
    
    // Get user's calendar token
    const { data: userData } = await supabase
      .from('users')
      .select('google_calendar_token')
      .eq('id', user.id)
      .single()
    
    // Insert each suggestion into calendar
    const inserted = []
    
    for (const suggestion of suggestions) {
      const calendarEvent = await insertSuggestion(
        userData!.google_calendar_token,
        suggestion.match.activity,
        suggestion.suggestedStart,
        suggestion.suggestedEnd,
        suggestion.match.matchReason
      )
      
      // Save to database
      const { data: dbSuggestion } = await supabase
        .from('suggestions')
        .insert({
          user_id: user.id,
          activity_id: suggestion.match.activityId,
          suggested_date: format(suggestion.suggestedStart, 'yyyy-MM-dd'),
          suggested_start_time: format(suggestion.suggestedStart, 'HH:mm:ss'),
          suggested_end_time: format(suggestion.suggestedEnd, 'HH:mm:ss'),
          google_calendar_event_id: calendarEvent.id,
          status: 'pending',
          match_reason: suggestion.match.matchReason,
        })
        .select()
        .single()
      
      inserted.push(dbSuggestion)
    }
    
    return NextResponse.json({
      success: true,
      count: inserted.length,
      suggestions: inserted,
    })
  } catch (error: any) {
    console.error('Suggestion generation error:', error)
    return NextResponse.json(
      { error: error.message || 'Failed to generate suggestions' },
      { status: 500 }
    )
  }
}
```

---

### Day 3-4: Add Suggestion UI to Dashboard

#### 1. Create Suggestions Display Component

`app/dashboard/suggestions.tsx`:

```typescript
'use client'

import { useState } from 'react'
import { format, parseISO } from 'date-fns'

export default function SuggestionsDisplay({ initialSuggestions }: { initialSuggestions: any[] }) {
  const [suggestions, setSuggestions] = useState(initialSuggestions)
  const [loading, setLoading] = useState(false)

  const generateSuggestions = async () => {
    setLoading(true)
    
    try {
      const response = await fetch('/api/suggestions/generate', {
        method: 'POST',
      })
      
      const data = await response.json()
      
      if (data.success && data.count > 0) {
        alert(`Generated ${data.count} suggestions! Check your Google Calendar.`)
        window.location.reload()
      } else {
        alert(data.reason || 'No suggestions could be generated right now.')
      }
    } catch (error) {
      alert('Error generating suggestions. Please try again.')
    } finally {
      setLoading(false)
    }
  }

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div>
          <h3 className="text-lg font-semibold">Your Suggestions</h3>
          <p className="text-sm text-gray-600">
            Activity ideas matched to your free time
          </p>
        </div>
        <button
          onClick={generateSuggestions}
          disabled={loading}
          className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:bg-gray-300"
        >
          {loading ? 'Generating...' : 'Generate Suggestions'}
        </button>
      </div>

      {suggestions.length === 0 ? (
        <div className="bg-gray-50 border-2 border-dashed border-gray-300 rounded-lg p-8 text-center">
          <p className="text-gray-600">
            No suggestions yet. Click "Generate Suggestions" to get started!
          </p>
        </div>
      ) : (
        <div className="space-y-4">
          {suggestions.map((suggestion: any) => (
            <div key={suggestion.id} className="bg-white border rounded-lg p-6">
              <div className="flex items-start gap-4">
                <div className="flex-1">
                  <h4 className="font-semibold text-lg mb-2">
                    {suggestion.activity.name}
                  </h4>
                  <p className="text-sm text-gray-600 mb-3">
                    {suggestion.match_reason}
                  </p>
                  <div className="flex items-center gap-4 text-sm text-gray-700">
                    <span>📅 {format(parseISO(suggestion.suggested_date), 'MMM d, yyyy')}</span>
                    <span>⏰ {suggestion.suggested_start_time} - {suggestion.suggested_end_time}</span>
                  </div>
                  <div className="mt-3 flex items-center gap-2">
                    <span className={`px-2 py-1 rounded text-xs font-medium ${
                      suggestion.status === 'pending' ? 'bg-yellow-100 text-yellow-800' :
                      suggestion.status === 'accepted' ? 'bg-green-100 text-green-800' :
                      'bg-gray-100 text-gray-800'
                    }`}>
                      {suggestion.status}
                    </span>
                    <span className="text-xs text-gray-500">
                      Added to your calendar
                    </span>
                  </div>
                </div>
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  )
}
```

#### 2. Update Dashboard Page

Update `app/dashboard/page.tsx` to include suggestions:

```typescript
import SuggestionsDisplay from './suggestions'

// Fetch suggestions
const { data: suggestions } = await supabase
  .from('suggestions')
  .select('*, activity:activities(*)')
  .eq('user_id', user!.id)
  .eq('status', 'pending')
  .order('suggested_date', { ascending: true })

// ... (in JSX)
<SuggestionsDisplay initialSuggestions={suggestions || []} />
```

---

### Day 5-6: End-to-End Testing

#### 1. Test Complete Flow

**Setup:**
1. Sign in as test user
2. Ensure "Want to Try" list has: "kayaking", "italian restaurant"
3. Ensure calendar has some events with 3+ hour gaps

**Test Steps:**
- [ ] Click "Generate Suggestions" button
- [ ] Wait for processing (5-10 seconds)
- [ ] Verify success message shows count
- [ ] Open Google Calendar in browser
- [ ] Verify suggestion event appears with:
  - Title: "💡 Try: [Activity Name]"
  - Blue color
  - Rich description with address, time, reason
  - Correct date/time in free block
- [ ] Verify suggestion appears in app dashboard
- [ ] Verify `suggestions` table has record

#### 2. Test Edge Cases

- [ ] No Want to Try items → should show "Add interests first"
- [ ] No free time → should show "No free time found"
- [ ] No matching activities → should show "No matches"
- [ ] Want to Try: "kayaking" → should match kayaking activities
- [ ] Want to Try: "pizza" → should match Italian restaurants with pizza

---

### Day 6-7: Polish & Bug Fixes

#### 1. Improve Match Reasons

Make match reasons more specific:
- "You wanted to try kayaking" ✅
- "You added this to your Want to Try list 3 days ago" ✅
- "Based on your interest in outdoor activities" ✅

#### 2. Handle Duplicate Suggestions

Prevent suggesting the same activity twice:

```typescript
// Check if activity was already suggested recently
const { data: recentSuggestions } = await supabase
  .from('suggestions')
  .select('activity_id')
  .eq('user_id', userId)
  .gte('created_at', new Date(Date.now() - 7 * 24 * 60 * 60 * 1000).toISOString())

const recentActivityIds = recentSuggestions?.map(s => s.activity_id) || []

// Filter out recently suggested activities
const newMatches = matches.filter(m => !recentActivityIds.includes(m.activityId))
```

---

## Technical Stack & Tools

**New in Phase 4:**
- Free time detection algorithm
- Activity matching algorithm (rule-based)
- Calendar event insertion
- Suggestion tracking

**References:**
- `ai/guides/google-calendar-api.md` - Event insertion
- `aiDocs/mvp.md` (lines 563-628) - Matching algorithm approach

---

## File Structure Created

```
app/
├── api/
│   └── suggestions/
│       └── generate/
│           └── route.ts          # Generate suggestions API
├── dashboard/
│   ├── suggestions.tsx           # Suggestions display component
│   └── page.tsx                  # Updated with suggestions

lib/
├── calendar/
│   └── detect-free-time.ts       # Free time detection
├── matching/
│   └── match-activities.ts       # Activity matching
├── suggestions/
│   └── generate.ts               # Suggestion generator
└── google-calendar/
    └── insert-event.ts           # Insert calendar events

scripts/
└── test-free-time.ts             # Test free time detection
```

---

## Testing Strategy

### Algorithm Testing
- [ ] Test free time detection with various calendar patterns
- [ ] Test matching with different Want to Try items
- [ ] Verify suggestions only use detected free time
- [ ] Test with empty calendar (should still generate suggestions)

### Integration Testing
- [ ] Generate suggestions via API
- [ ] Verify events appear in Google Calendar
- [ ] Verify suggestions saved to database
- [ ] Test with multiple Want to Try items
- [ ] Test with various free time patterns

### UI Testing
- [ ] Suggestions display correctly
- [ ] Generate button works
- [ ] Loading state shows during generation
- [ ] Success message appears
- [ ] Suggestions update after generation

---

## Risks & Mitigation

### Risk 1: No Free Time Detected
**Impact:** High - can't generate suggestions  
**Mitigation:** Use lenient threshold (2 hours instead of 3)  
**Contingency:** Suggest times outside calendar (weekends)

### Risk 2: Poor Matching Quality
**Impact:** Medium - irrelevant suggestions  
**Mitigation:** Test with various Want to Try phrases, refine keywords  
**Contingency:** Fall back to random high-rated activities

### Risk 3: Calendar Event Insertion Fails
**Impact:** High - suggestions don't appear  
**Mitigation:** Add comprehensive error handling, retry logic  
**Contingency:** Show suggestions in app only (manual calendar add)

---

## Deliverables

- [ ] Free time detection working
- [ ] Activity matching algorithm implemented
- [ ] Suggestions generated and inserted into calendar
- [ ] Suggestions display in dashboard
- [ ] Generate suggestions button
- [ ] Database tracking of suggestions

---

## Definition of Done

Phase 4 is complete when:

- ✅ App detects free time from calendar
- ✅ App matches Want to Try list with activities
- ✅ Suggestions appear in Google Calendar
- ✅ Suggestions display in dashboard
- ✅ User can generate new suggestions
- ✅ All success criteria met
- ✅ Ready for Phase 5 (Babysitter Feature)

**Next Phase:** Phase 5 - Babysitter Feature (Week 6)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026
