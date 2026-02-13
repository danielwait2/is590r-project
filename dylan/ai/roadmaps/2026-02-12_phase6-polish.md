# Phase 6: Polish & Demo Prep - Detailed Implementation Plan

**Timeline:** Week 7 (7 days)  
**Status:** Not Started  
**Roadmap:** [2026-02-12_roadmap-phase6.md](./2026-02-12_roadmap-phase6.md) - Track progress

---

## Overview

Final week to polish the app, fix bugs, optimize performance, create demo script, and prepare for presentation. Focus on user experience, visual design, and ensuring the demo flows smoothly.

**What gets built:**
- UI polish and visual improvements
- Error handling and edge cases
- Loading states and feedback
- Demo script and walkthrough
- Bug fixes
- Performance optimizations
- Documentation for handoff

---

## Success Criteria

- [ ] App looks professional (clean UI, consistent design)
- [ ] All core flows work smoothly without errors
- [ ] Demo script written and rehearsed
- [ ] Demo takes 5-7 minutes to show full flow
- [ ] No critical bugs blocking demo
- [ ] App deployed to production
- [ ] Demo data seeded (test user, activities, calendar events)

---

## Prerequisites

**From Previous Phases:**
- [ ] All Phase 1-5 features working
- [ ] App deployed to Vercel
- [ ] Database has real activity data
- [ ] Twilio webhook configured

---

## Detailed Implementation Steps

### Day 1-2: UI Polish

#### 1. Create Design System

`app/globals.css` - Add design tokens:

```css
:root {
  /* Colors */
  --color-primary: #2563eb;      /* Blue */
  --color-primary-hover: #1d4ed8;
  --color-secondary: #10b981;    /* Green */
  --color-danger: #ef4444;       /* Red */
  --color-warning: #f59e0b;      /* Amber */
  
  --color-text: #111827;         /* Gray-900 */
  --color-text-secondary: #6b7280; /* Gray-500 */
  --color-text-muted: #9ca3af;   /* Gray-400 */
  
  --color-bg: #ffffff;
  --color-bg-secondary: #f9fafb; /* Gray-50 */
  --color-border: #e5e7eb;       /* Gray-200 */
  
  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;
  
  /* Border radius */
  --radius-sm: 0.375rem;
  --radius-md: 0.5rem;
  --radius-lg: 0.75rem;
  
  /* Shadows */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
}
```

#### 2. Improve Dashboard Layout

`app/dashboard/layout.tsx` - Add sidebar navigation:

```typescript
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen bg-gray-50">
      {/* Header */}
      <header className="bg-white border-b border-gray-200 sticky top-0 z-10">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4">
          <div className="flex items-center justify-between">
            <h1 className="text-2xl font-bold text-gray-900">
              Family Time 💙
            </h1>
            <div className="flex items-center gap-4">
              <Link href="/dashboard" className="text-sm text-gray-600 hover:text-gray-900">
                Dashboard
              </Link>
              <Link href="/dashboard/activities" className="text-sm text-gray-600 hover:text-gray-900">
                Activities
              </Link>
              <Link href="/dashboard/sitters" className="text-sm text-gray-600 hover:text-gray-900">
                Babysitters
              </Link>
              <form action="/auth/sign-out" method="post">
                <button className="text-sm text-gray-600 hover:text-gray-900">
                  Sign Out
                </button>
              </form>
            </div>
          </div>
        </div>
      </header>

      {/* Main content */}
      <main className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        {children}
      </main>
    </div>
  )
}
```

#### 3. Add Empty States

For suggestions, calendar events, activities, sitters - create friendly empty states with clear CTAs.

Example empty state component:

```typescript
export function EmptyState({
  icon,
  title,
  description,
  action,
}: {
  icon: React.ReactNode
  title: string
  description: string
  action?: React.ReactNode
}) {
  return (
    <div className="bg-white border-2 border-dashed border-gray-300 rounded-lg p-12 text-center">
      <div className="flex justify-center mb-4">
        {icon}
      </div>
      <h3 className="text-lg font-medium text-gray-900 mb-2">{title}</h3>
      <p className="text-gray-600 mb-6">{description}</p>
      {action}
    </div>
  )
}
```

#### 4. Improve Form Styling

Make all forms consistent:
- Clear labels
- Helpful placeholder text
- Inline validation errors
- Disabled state styling
- Loading indicators

---

### Day 2-3: Error Handling & Edge Cases

#### 1. Add Global Error Boundary

`app/error.tsx`:

```typescript
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="bg-white rounded-lg shadow-lg p-8 max-w-md text-center">
        <h2 className="text-2xl font-bold text-gray-900 mb-4">
          Something went wrong
        </h2>
        <p className="text-gray-600 mb-6">
          {error.message || 'An unexpected error occurred'}
        </p>
        <button
          onClick={reset}
          className="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
        >
          Try again
        </button>
      </div>
    </div>
  )
}
```

#### 2. Handle Calendar Token Expiration

Add refresh token logic:

`lib/google-calendar/refresh-token.ts`:

```typescript
import { google } from 'googleapis'

export async function refreshAccessToken(refreshToken: string) {
  const oauth2Client = new google.auth.OAuth2(
    process.env.GOOGLE_CLIENT_ID,
    process.env.GOOGLE_CLIENT_SECRET
  )

  oauth2Client.setCredentials({
    refresh_token: refreshToken,
  })

  const { credentials } = await oauth2Client.refreshAccessToken()
  return credentials.access_token
}
```

Update calendar sync to use refresh token if access token expired.

#### 3. Handle Common Errors

- Calendar not connected → show "Connect Calendar" CTA
- No Want to Try items → show "Add interests" CTA
- No activities matched → suggest adding more interests
- No free time found → show next available time
- SMS send failure → log error, show in UI

---

### Day 3-4: Loading States & Feedback

#### 1. Add Loading Skeleton Components

Create reusable skeletons:

```typescript
export function ActivityCardSkeleton() {
  return (
    <div className="bg-white border rounded-lg p-4 animate-pulse">
      <div className="h-5 bg-gray-200 rounded w-3/4 mb-3"></div>
      <div className="h-4 bg-gray-100 rounded w-1/2 mb-2"></div>
      <div className="h-4 bg-gray-100 rounded w-1/4"></div>
    </div>
  )
}
```

Use in all async views while data loads.

#### 2. Add Toast Notifications

Install sonner for toast notifications:

```bash
npm install sonner
```

Add to layout:

```typescript
import { Toaster } from 'sonner'

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Toaster position="top-right" />
      </body>
    </html>
  )
}
```

Use for success/error feedback:

```typescript
import { toast } from 'sonner'

// Success
toast.success('Babysitter added successfully!')

// Error
toast.error('Failed to connect calendar')

// Loading
toast.loading('Generating suggestions...')
```

#### 3. Add Button Loading States

```typescript
<button
  disabled={loading}
  className="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:bg-gray-300 disabled:cursor-not-allowed flex items-center gap-2"
>
  {loading && (
    <svg className="animate-spin h-4 w-4" viewBox="0 0 24 24">
      <circle className="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" strokeWidth="4" fill="none" />
      <path className="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
    </svg>
  )}
  {loading ? 'Generating...' : 'Generate Suggestions'}
</button>
```

---

### Day 4-5: Demo Script & Test Data

#### 1. Create Demo Script

`demo-script.md`:

```markdown
# Family Time App - Demo Script (5-7 minutes)

## Setup (Before Demo)
- [ ] Test user signed in
- [ ] Calendar connected with events
- [ ] 3 trusted sitters added
- [ ] "Want to Try" list: kayaking, italian restaurant, hiking
- [ ] 100+ activities in database
- [ ] No existing suggestions

## Demo Flow

### 1. Introduction (1 min)
"Hi! I'm demoing Family Time, an app that helps busy couples finally do the things they keep talking about doing."

### 2. Show Problem (1 min)
"The problem: Dylan and Sarah have a list of activities they want to try together—kayaking, new restaurants, hiking trails—but weekends slip by without actually doing them. The issue isn't lack of ideas; it's connecting what they want with when they're free."

### 3. Show Onboarding (1 min)
- Navigate to dashboard
- Show "Want to Try" list: kayaking, italian restaurant, hiking
- Show calendar integration (events displayed)
- Show trusted babysitters roster (3 sitters)

### 4. Core Feature: Generate Suggestions (2-3 min)
- Click "Generate Suggestions"
- Watch loading state
- Success! "Generated 2 suggestions"
- Open Google Calendar in new tab
- **Show calendar event:**
  - Title: "💡 Try: Lady Bird Lake Kayaking"
  - Date: This Saturday at 2 PM
  - Description includes:
    - "You wanted to try kayaking" (match reason)
    - Address, rating, price
    - "Babysitter: We'll coordinate when you accept"
- Go back to app dashboard
- Show suggestions with match reasons

### 5. Babysitter Feature (1-2 min)
- "Now the magic happens—when Dylan accepts this suggestion, the app automatically texts all trusted babysitters."
- Show SMS sent (via Twilio console or phone screenshot)
- Show SMS content:
  - Date, time, duration
  - Number of kids, ages
  - Estimated pay
  - Reply YES/NO
- Show sitter reply: "YES"
- **Calendar updates**: "Babysitter: Sarah Johnson confirmed ✓"
- Other sitters get "Position filled" message

### 6. Wrap Up (30 sec)
"That's it! Family Time bridges the desire-to-action gap by:
1. Connecting what you want with when you're free
2. Suggesting specific opportunities
3. Coordinating babysitters automatically

All automatically, in your calendar."

## Backup Talking Points
- Calendar privacy: "We only detect time gaps, event details stay private"
- Activity data: "Curated from Google Maps and Yelp"
- Future features: Weather awareness, Apple Calendar, booking integration
```

#### 2. Create Demo Seed Script

`scripts/seed-demo-data.ts`:

```typescript
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)

async function seedDemoData() {
  const demoUserId = 'demo-user-id'  // Replace with actual user ID
  
  // 1. Add Want to Try items
  const interests = [
    { user_id: demoUserId, name: 'kayaking' },
    { user_id: demoUserId, name: 'italian restaurant' },
    { user_id: demoUserId, name: 'hiking' },
  ]
  
  await supabase.from('interests').insert(interests)
  
  // 2. Add trusted sitters
  const sitters = [
    { user_id: demoUserId, name: 'Sarah Johnson', phone: '+15551234567', relationship: 'friend' },
    { user_id: demoUserId, name: 'Mike Davis', phone: '+15559876543', relationship: 'family' },
    { user_id: demoUserId, name: 'Emma Wilson', phone: '+15555555555', relationship: 'regular_sitter' },
  ]
  
  await supabase.from('trusted_sitters').insert(sitters)
  
  // 3. Add kids
  await supabase.from('kids').insert([
    { user_id: demoUserId, name: 'Oliver', age: 4 },
    { user_id: demoUserId, name: 'Emma', age: 7 },
  ])
  
  console.log('✅ Demo data seeded!')
}

seedDemoData()
```

---

### Day 5-6: Bug Fixing & Testing

#### 1. Create Test Checklist

Test every flow:

**Authentication:**
- [ ] Sign up with Google
- [ ] Sign in with existing account
- [ ] Sign out

**Onboarding:**
- [ ] Complete onboarding form
- [ ] Submit with all fields
- [ ] Skip optional fields

**Calendar:**
- [ ] Connect Google Calendar
- [ ] View calendar events
- [ ] Refresh calendar manually
- [ ] Handle expired token

**Activities:**
- [ ] Browse activities page
- [ ] Filter by category
- [ ] View activity details

**Suggestions:**
- [ ] Generate suggestions
- [ ] View suggestions in dashboard
- [ ] Check Google Calendar for events
- [ ] Handle no free time
- [ ] Handle no matches

**Babysitters:**
- [ ] Add babysitter
- [ ] View roster
- [ ] Delete babysitter
- [ ] SMS sent when suggestion generated
- [ ] Receive SMS webhook
- [ ] Handle YES response
- [ ] Handle NO response
- [ ] Handle multiple sitters

**Error Cases:**
- [ ] No calendar connected
- [ ] No Want to Try items
- [ ] No babysitters
- [ ] Network error
- [ ] Invalid phone number

#### 2. Fix Critical Bugs

Priority 1 (blocks demo):
- Anything that causes app crash
- Suggestions not generating
- SMS not sending/receiving
- Calendar not updating

Priority 2 (polish):
- UI glitches
- Loading states
- Error messages
- Edge cases

---

### Day 6-7: Demo Rehearsal & Final Polish

#### 1. Rehearse Demo

- Practice demo script 3-5 times
- Time yourself (aim for 5-7 minutes)
- Have backup plan if something fails
- Prepare answers to common questions

#### 2. Create Demo Video (Optional)

Record screen walkthrough for backup.

#### 3. Final Deployment Check

- [ ] Environment variables set in Vercel
- [ ] Twilio webhook points to production
- [ ] Database populated with activities
- [ ] Demo user account ready
- [ ] All features working in production

---

## Technical Stack & Tools

**New in Phase 6:**
- Sonner (toast notifications)
- Demo script and test data
- Error boundaries

---

## File Structure Created

```
app/
├── error.tsx                     # Global error boundary
├── dashboard/
│   └── layout.tsx                # Polished navigation
└── globals.css                   # Design tokens

components/
├── empty-state.tsx               # Reusable empty states
├── loading-skeleton.tsx          # Loading components
└── toast-provider.tsx            # Toast notifications

scripts/
└── seed-demo-data.ts             # Demo data seeder

docs/
└── demo-script.md                # Demo walkthrough
```

---

## Testing Strategy

### Comprehensive Testing
- [ ] Test all user flows end-to-end
- [ ] Test error cases
- [ ] Test on different devices (mobile, desktop)
- [ ] Test on different browsers (Chrome, Safari, Firefox)
- [ ] Test with real calendar data
- [ ] Test with real SMS (Twilio)

### Demo Dry Run
- [ ] Run through demo script 3+ times
- [ ] Time demo (should be 5-7 minutes)
- [ ] Test on production environment
- [ ] Have backup plan ready

---

## Risks & Mitigation

### Risk 1: Demo Fails During Presentation
**Impact:** Critical  
**Mitigation:** Rehearse 3+ times, test on production  
**Contingency:** Have video recording as backup

### Risk 2: Network Issues During Demo
**Impact:** High  
**Mitigation:** Test internet connection beforehand  
**Contingency:** Use localhost if possible, or show screenshots

### Risk 3: SMS Delays During Demo
**Impact:** Medium  
**Mitigation:** Pre-send SMS before demo, show received message  
**Contingency:** Explain that SMS typically takes 5-10 seconds

---

## Deliverables

- [ ] Polished UI with consistent design
- [ ] All error cases handled gracefully
- [ ] Loading states and feedback everywhere
- [ ] Demo script written and rehearsed
- [ ] Demo data seeded
- [ ] Bug fixes completed
- [ ] App deployed and working
- [ ] Ready to demo!

---

## Definition of Done

Phase 6 is complete when:

- ✅ App looks professional and polished
- ✅ All core flows work without errors
- ✅ Demo script written and practiced
- ✅ Demo data seeded and tested
- ✅ No critical bugs
- ✅ App deployed to production
- ✅ Ready for demo presentation
- ✅ **MVP COMPLETE!** 🎉

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026
