# Technical Architecture

**Version:** 1.0
**Last Updated:** 2026-02-11
**Status:** Draft

---

## Overview

This document defines the technical architecture for the Family Quality Time App across three phases:

1. **MVP (Chrome Extension)** - Client-side demo for validation (4-5 days)
2. **V1 (Full Product)** - Production web app with backend (12 weeks)
3. **V2+ (Future)** - Advanced features including AI vision integration

---

## MVP Architecture: Chrome Extension

### Design Principles

- **Client-side only** - No backend to minimize complexity
- **Real integrations** - Actual Google Calendar OAuth, not mocked
- **Curated over API** - Hard-coded activities > external APIs for demo
- **Fast to build** - Vanilla JS, no frameworks, simple UI

### Tech Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **Platform** | Chrome Extension (Manifest V3) | Where users already are (calendar browser) |
| **Language** | Vanilla JavaScript | No build step, fast iteration |
| **UI** | HTML + CSS | Simple popup, no framework needed |
| **Storage** | Chrome Local Storage | Persist OAuth tokens + preferences |
| **Calendar** | Google Calendar API v3 | OAuth 2.0, read + write access |
| **Authentication** | `chrome.identity` API | Built-in OAuth for extensions |

### System Architecture

```
┌─────────────────────────────────────────────┐
│           User's Browser (Chrome)            │
│                                              │
│  ┌────────────────────────────────────┐    │
│  │     Chrome Extension                │    │
│  │                                     │    │
│  │  ┌──────────────────────────────┐  │    │
│  │  │  popup.html (UI)             │  │    │
│  │  │  - Free time display         │  │    │
│  │  │  - Want to try form          │  │    │
│  │  │  - Suggestion cards          │  │    │
│  │  └──────────┬───────────────────┘  │    │
│  │             │                       │    │
│  │  ┌──────────▼───────────────────┐  │    │
│  │  │  popup.js (Logic)            │  │    │
│  │  │  - UI event handlers         │  │    │
│  │  │  - State management          │  │    │
│  │  └──────────┬───────────────────┘  │    │
│  │             │                       │    │
│  │  ┌──────────▼───────────────────┐  │    │
│  │  │  calendar.js (API Wrapper)   │  │    │
│  │  │  - fetchEvents()             │  │    │
│  │  │  - createEvent()             │  │    │
│  │  │  - detectFreeTime()          │  │    │
│  │  └──────────┬───────────────────┘  │    │
│  │             │                       │    │
│  │  ┌──────────▼───────────────────┐  │    │
│  │  │  suggestions.js (Matching)   │  │    │
│  │  │  - Curated activities DB     │  │    │
│  │  │  - Keyword matching logic    │  │    │
│  │  └──────────────────────────────┘  │    │
│  │             │                       │    │
│  │  ┌──────────▼───────────────────┐  │    │
│  │  │  background.js (Service)     │  │    │
│  │  │  - OAuth token management    │  │    │
│  │  └──────────────────────────────┘  │    │
│  │             │                       │    │
│  │  ┌──────────▼───────────────────┐  │    │
│  │  │  chrome.storage.local        │  │    │
│  │  │  - OAuth tokens              │  │    │
│  │  │  - Want to try list          │  │    │
│  │  └──────────────────────────────┘  │    │
│  └─────────────┬────────────────────────┘    │
│                │                             │
└────────────────┼─────────────────────────────┘
                 │
                 │ HTTPS API Calls
                 ▼
    ┌────────────────────────────┐
    │  Google Calendar API v3    │
    │  - GET /calendars/primary/ │
    │        events              │
    │  - POST /calendars/primary/│
    │         events             │
    └────────────────────────────┘
```

### File Structure

```
family-time-extension/
├── manifest.json          # Extension configuration
├── popup.html             # Main popup UI
├── popup.css              # Popup styles
├── popup.js               # UI logic and event handlers
├── background.js          # Background service worker
├── calendar.js            # Google Calendar API wrapper
├── suggestions.js         # Curated activities + matching
├── icons/
│   ├── icon16.png
│   ├── icon48.png
│   └── icon128.png
└── README.md              # Installation instructions
```

### Data Flow

**1. Initial Load**
```
User clicks extension icon
  → popup.js loads
  → Check chrome.storage for OAuth token
  → If no token: Show "Connect Calendar" button
  → If token exists: Fetch calendar events
```

**2. Free Time Detection**
```
calendar.fetchEvents(now, +7 days)
  → Filter to Sat/Sun 9am-6pm
  → Find 2-4 hour blocks with no events
  → Return first available block
  → Display in popup UI
```

**3. Suggestion Generation**
```
User clicks "Get Suggestions"
  → Read "want to try" list from storage
  → suggestions.match(freeTime, wantToTry)
  → Filter curated activities by keyword match
  → Return top 2 matches
  → Display suggestion cards
```

**4. Add to Calendar**
```
User clicks "Add to Calendar"
  → Build event object (title, time, location, description)
  → calendar.createEvent(event)
  → POST to Google Calendar API
  → Show success message
  → Event appears in user's calendar
```

### OAuth Flow (MVP)

```
User clicks "Connect Calendar"
  ↓
chrome.identity.getAuthToken({ interactive: true })
  ↓
Google OAuth consent screen
  ↓
User grants calendar.readonly + calendar.events permissions
  ↓
Extension receives access token
  ↓
Store token in chrome.storage.local
  ↓
Use token for all API calls (Authorization: Bearer <token>)
```

**Scopes Required:**
- `https://www.googleapis.com/auth/calendar.readonly` - Read calendar events
- `https://www.googleapis.com/auth/calendar.events` - Create events

### Curated Activities Database (MVP)

Instead of external APIs, use a hard-coded JSON structure:

```javascript
const curatedActivities = [
  {
    id: "act_001",
    title: "Children's Discovery Museum",
    type: "museum",
    description: "Interactive science exhibits for ages 2-10",
    address: "123 Main St, Chicago, IL",
    imageUrl: "https://example.com/museum.jpg",
    estimatedCost: "$35",
    duration: "2-3 hours",
    ageRange: [2, 10],
    tags: ["science", "museum", "indoor", "educational"],
    bookingLink: "https://discoverymuseum.org"
  },
  // 10-15 more activities...
];
```

**Matching Algorithm (Simple):**
```javascript
function matchActivities(wantToTryList, curatedActivities) {
  // 1. Tokenize want-to-try items
  const keywords = wantToTryList
    .map(item => item.toLowerCase().split(' '))
    .flat();

  // 2. Score each activity
  const scored = curatedActivities.map(activity => {
    let score = 0;
    activity.tags.forEach(tag => {
      if (keywords.includes(tag)) score += 1;
    });
    return { activity, score };
  });

  // 3. Sort by score, return top 2
  return scored
    .sort((a, b) => b.score - a.score)
    .slice(0, 2)
    .map(item => item.activity);
}
```

### Limitations (MVP)

- No backend → No automation, user must manually open extension
- No database → No learning from past suggestions
- No real recommendation engine → Simple keyword matching
- Single calendar only → No multi-partner coordination
- Hard-coded activities → Limited to 10-15 pre-curated items
- No email/SMS → Only in-extension notifications

---

## V1 Architecture: Full Product

### Design Principles

- **Calendar-native** - Primary interaction via calendar, not app
- **Proactive, not reactive** - System initiates suggestions, not user
- **Persistent context** - Deep family profile, learns over time
- **Multi-interface** - Web app, email, future: AI assistants, SMS

### Tech Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| **Frontend** | Next.js 14 + React + TypeScript | SSR, SEO, type safety |
| **Styling** | Tailwind CSS | Rapid UI development |
| **Backend** | Node.js + Express | JavaScript everywhere, mature ecosystem |
| **Database** | PostgreSQL | Relational data (users, profiles, suggestions) |
| **ORM** | Prisma | Type-safe DB queries, migrations |
| **Job Scheduler** | BullMQ + Redis | Weekly cron jobs for suggestion generation |
| **Authentication** | Clerk | OAuth + user management |
| **Email** | Resend | Transactional emails (suggestions, reminders) |
| **Hosting** | Vercel (frontend) + Railway (backend + DB) | Easy deployment, auto-scaling |
| **APIs** | Google Calendar, Google Places, Eventbrite | Calendar access + activity discovery |

**Alternative Stack (Python):**
- Backend: FastAPI
- ORM: SQLAlchemy
- Async: Celery + Redis

### System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                         Frontend                             │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Next.js Web App (Vercel)                              │ │
│  │                                                         │ │
│  │  Pages:                                                 │ │
│  │  - Landing page (/)                                     │ │
│  │  - Onboarding flow (/setup)                            │ │
│  │  - Dashboard (/dashboard)                              │ │
│  │  - Profile management (/profile)                       │ │
│  │  - Suggestion history (/suggestions)                   │ │
│  │                                                         │ │
│  │  Components:                                            │ │
│  │  - SuggestionCard, FamilyProfileForm,                  │ │
│  │    CalendarPreview, FeedbackWidget                     │ │
│  └────────────┬────────────────────────────────────────────┘ │
│               │                                              │
└───────────────┼──────────────────────────────────────────────┘
                │
                │ REST API calls
                ▼
┌──────────────────────────────────────────────────────────────┐
│                       Backend API                            │
│                                                              │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  Express.js API Server (Railway)                       │ │
│  │                                                         │ │
│  │  Routes:                                                │ │
│  │  - POST /auth/google (OAuth)                           │ │
│  │  - GET/PUT /user/profile                               │ │
│  │  - GET /suggestions                                    │ │
│  │  - POST /suggestions/:id/accept                        │ │
│  │  - POST /suggestions/:id/feedback                      │ │
│  │  - GET /calendar/sync                                  │ │
│  │                                                         │ │
│  │  Services:                                              │ │
│  │  - authService (OAuth handling)                        │ │
│  │  - calendarService (Google Calendar API)               │ │
│  │  - recommendationEngine (suggestion generation)        │ │
│  │  - activityService (Places/Eventbrite API)            │ │
│  │  - emailService (Resend integration)                   │ │
│  └────────┬──────────┬────────────────┬────────────────────┘ │
│           │          │                │                      │
└───────────┼──────────┼────────────────┼──────────────────────┘
            │          │                │
            ▼          ▼                ▼
  ┌─────────────┐  ┌──────────┐  ┌──────────────┐
  │ PostgreSQL  │  │  Redis   │  │ External APIs│
  │  (Railway)  │  │ (Railway)│  │              │
  │             │  │          │  │ - Google Cal │
  │ Tables:     │  │ Used by: │  │ - Places API │
  │ - users     │  │ - BullMQ │  │ - Eventbrite │
  │ - profiles  │  └──────────┘  └──────────────┘
  │ - suggestions│
  │ - feedback  │
  │ - calendar  │
  │   _scans    │
  └─────────────┘

┌──────────────────────────────────────────────────────────────┐
│                   Background Jobs (BullMQ)                   │
│                                                              │
│  Weekly Cron (Sundays 8pm):                                 │
│  1. Fetch all active users                                  │
│  2. For each user:                                          │
│     - Sync calendar (detect free time)                      │
│     - Generate 2 suggestions                                │
│     - Store in DB                                           │
│     - Queue email for Friday 8am                            │
│                                                              │
│  Daily Cron (Fridays 8am):                                  │
│  - Send suggestion emails to all users                      │
│                                                              │
│  Weekly Cron (Mondays 10am):                                │
│  - Send "How was it?" follow-up emails                      │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                      Email Delivery                          │
│                                                              │
│  Resend / SendGrid                                          │
│  - Suggestion digest (Friday 8am)                           │
│  - Follow-up survey (Monday 10am)                           │
│  - Onboarding emails (welcome, tips)                        │
└──────────────────────────────────────────────────────────────┘
```

### Database Schema

**Users**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  clerk_user_id VARCHAR(255) UNIQUE NOT NULL,
  google_calendar_id VARCHAR(255),
  google_refresh_token TEXT,
  onboarding_completed BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Family Profiles**
```sql
CREATE TABLE family_profiles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  num_kids INTEGER,
  kid_ages INTEGER[] NOT NULL,
  interests TEXT[] NOT NULL, -- e.g., ['outdoors', 'museums', 'food']
  budget_tier VARCHAR(20) CHECK (budget_tier IN ('low', 'medium', 'high')),
  max_drive_time_minutes INTEGER,
  want_to_try_list TEXT[], -- User's "want to try" items
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Suggestions**
```sql
CREATE TABLE suggestions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  generated_at TIMESTAMP DEFAULT NOW(),
  target_date DATE NOT NULL,
  target_time_slot VARCHAR(50), -- e.g., "10:00 AM - 2:00 PM"
  activity_type VARCHAR(50), -- 'restaurant', 'park', 'museum', 'event'
  title VARCHAR(255) NOT NULL,
  description TEXT,
  address TEXT,
  booking_link TEXT,
  estimated_cost VARCHAR(50),
  image_url TEXT,
  status VARCHAR(20) DEFAULT 'sent' CHECK (status IN ('sent', 'accepted', 'rejected', 'ignored')),
  feedback VARCHAR(20) CHECK (feedback IN ('thumbs_up', 'thumbs_down')),
  feedback_comment TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

**Calendar Scans**
```sql
CREATE TABLE calendar_scans (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  scanned_at TIMESTAMP DEFAULT NOW(),
  free_blocks JSONB, -- Array of { date, startTime, endTime }
  created_at TIMESTAMP DEFAULT NOW()
);
```

### API Endpoints (V1)

**Authentication**
```
POST   /api/auth/google          Initiate Google OAuth flow
GET    /api/auth/callback        OAuth callback handler
POST   /api/auth/logout          Logout user
```

**User & Profile**
```
GET    /api/user/profile         Get user + family profile
PUT    /api/user/profile         Update family profile
DELETE /api/user/account         Delete account
```

**Suggestions**
```
GET    /api/suggestions          Get all suggestions for user
GET    /api/suggestions/:id      Get single suggestion
POST   /api/suggestions/:id/accept    Mark as accepted
POST   /api/suggestions/:id/reject    Mark as rejected
POST   /api/suggestions/:id/feedback  Submit thumbs up/down + comment
```

**Calendar**
```
GET    /api/calendar/sync        Manually trigger calendar sync
GET    /api/calendar/free-time   Get detected free time blocks
```

**Admin (Internal)**
```
GET    /api/admin/users          List all users
POST   /api/admin/generate/:userId   Manually generate suggestions
GET    /api/admin/metrics        System metrics (acceptance rate, etc.)
```

### Recommendation Engine Algorithm (V1)

**Step 1: Detect Free Time**
```typescript
async function detectFreeTime(userId: string): Promise<FreeBlock[]> {
  // 1. Fetch calendar events for next 7 days
  const events = await googleCalendar.getEvents({
    userId,
    timeMin: new Date(),
    timeMax: addDays(new Date(), 7)
  });

  // 2. Filter to Saturdays and Sundays
  const weekendDays = getWeekendDates(7);

  // 3. For each weekend day, find free blocks
  const freeBlocks = weekendDays.flatMap(day => {
    return findFreeBlocksInDay({
      day,
      events,
      businessHours: { start: '09:00', end: '18:00' },
      minBlockSize: 2, // hours
      maxBlockSize: 4
    });
  });

  // 4. Store in database
  await db.calendarScans.create({
    userId,
    scannedAt: new Date(),
    freeBlocks
  });

  return freeBlocks;
}
```

**Step 2: Query Activity Sources**
```typescript
async function queryActivities(
  location: string,
  targetDate: Date,
  profile: FamilyProfile
): Promise<Activity[]> {
  // A. Google Places API
  const places = await googlePlaces.search({
    query: 'family activities',
    location: location,
    radius: profile.maxDriveTimeMinutes * 1000, // approx meters
    type: ['park', 'museum', 'zoo', 'aquarium'],
    openOn: targetDate,
    minRating: 4.0
  });

  // B. Eventbrite API
  const events = await eventbrite.search({
    location: location,
    startDate: targetDate,
    categories: ['Family & Education', 'Arts', 'Food & Drink'],
    priceMax: getBudgetMax(profile.budgetTier)
  });

  // C. Curated database (manually seeded)
  const curated = await db.curatedActivities.findMany({
    where: {
      city: location,
      ageRange: { contains: profile.kidAges }
    }
  });

  return [...places, ...events, ...curated];
}
```

**Step 3: Match & Score**
```typescript
function scoreActivity(
  activity: Activity,
  profile: FamilyProfile,
  recentSuggestions: Suggestion[]
): number {
  let score = 0;

  // Interest match (30 points)
  const interestMatch = activity.tags.some(tag =>
    profile.interests.includes(tag)
  );
  if (interestMatch) score += 30;

  // Want to Try match (25 points)
  const wantToTryMatch = profile.wantToTryList.some(item =>
    activity.title.toLowerCase().includes(item.toLowerCase())
  );
  if (wantToTryMatch) score += 25;

  // Age appropriateness (20 points)
  const ageMatch = profile.kidAges.every(age =>
    age >= activity.ageRange[0] && age <= activity.ageRange[1]
  );
  if (ageMatch) score += 20;

  // Budget fit (15 points)
  if (activity.cost <= getBudgetMax(profile.budgetTier)) {
    score += 15;
  }

  // Drive time (10 points)
  if (activity.driveTimeMinutes <= profile.maxDriveTimeMinutes) {
    score += 10;
  }

  // Penalties
  // - Suggested in last 4 weeks
  const recentlyRecommended = recentSuggestions.some(s =>
    s.title === activity.title &&
    differenceInWeeks(new Date(), s.generatedAt) < 4
  );
  if (recentlyRecommended) score -= 20;

  // - Similar type suggested last week
  const similarTypeLastWeek = recentSuggestions.some(s =>
    s.activityType === activity.type &&
    differenceInWeeks(new Date(), s.generatedAt) < 1
  );
  if (similarTypeLastWeek) score -= 10;

  return score;
}
```

**Step 4: Generate Suggestions**
```typescript
async function generateSuggestions(userId: string): Promise<Suggestion[]> {
  // 1. Get user profile
  const profile = await db.familyProfiles.findUnique({ where: { userId } });

  // 2. Detect free time
  const freeBlocks = await detectFreeTime(userId);

  if (freeBlocks.length === 0) {
    return []; // No free time, no suggestions
  }

  // 3. For each free block, query activities
  const suggestions: Suggestion[] = [];

  for (const block of freeBlocks.slice(0, 2)) { // Top 2 free blocks
    const activities = await queryActivities(
      profile.location,
      block.date,
      profile
    );

    // 4. Score activities
    const recentSuggestions = await db.suggestions.findMany({
      where: {
        userId,
        generatedAt: { gte: subWeeks(new Date(), 4) }
      }
    });

    const scored = activities.map(activity => ({
      activity,
      score: scoreActivity(activity, profile, recentSuggestions)
    }));

    // 5. Select top match for this block
    const topMatch = scored.sort((a, b) => b.score - a.score)[0];

    if (topMatch && topMatch.score > 30) { // Minimum threshold
      suggestions.push({
        userId,
        targetDate: block.date,
        targetTimeSlot: `${block.startTime} - ${block.endTime}`,
        activityType: topMatch.activity.type,
        title: topMatch.activity.title,
        description: topMatch.activity.description,
        address: topMatch.activity.address,
        bookingLink: topMatch.activity.bookingLink,
        estimatedCost: topMatch.activity.cost,
        imageUrl: topMatch.activity.imageUrl,
        status: 'sent'
      });
    }
  }

  // 6. Store suggestions
  await db.suggestions.createMany({ data: suggestions });

  return suggestions;
}
```

**Step 5: Deliver via Email**
```typescript
async function sendSuggestionEmail(
  userId: string,
  suggestions: Suggestion[]
): Promise<void> {
  const user = await db.users.findUnique({ where: { id: userId } });

  await resend.emails.send({
    from: 'FamilyTime <suggestions@familytime.com>',
    to: user.email,
    subject: "This weekend's family time ideas",
    react: SuggestionEmailTemplate({ suggestions }),
    scheduledAt: nextFriday8AM() // Friday 8am
  });
}
```

---

## V2+ Architecture: Future Enhancements

### Vision API Integration (Calendar Image Analysis)

**Use Case:** Analyze calendar screenshots to detect free time without OAuth

**Flow:**
```
User uploads calendar screenshot
  → Sharp: Add text overlays (date markers, annotations)
  → Convert to base64
  → OpenAI Vision API: Analyze image
  → Extract free time blocks
  → Generate suggestions
```

**Implementation:**

**1. Image Processing with Sharp**
```typescript
import sharp from 'sharp';

async function annotateCalendarImage(
  imageBuffer: Buffer,
  annotations: string[]
): Promise<Buffer> {
  // Load base image
  const image = sharp(imageBuffer);

  // Create text overlays
  const overlays = annotations.map((text, index) => ({
    input: {
      text: {
        text: text,
        font: 'Arial',
        width: 300,
        rgba: true,
        align: 'center'
      }
    },
    gravity: 'north',
    top: 50 + (index * 40) // Stack annotations vertically
  }));

  // Composite text onto image
  const annotatedImage = await image
    .composite(overlays)
    .jpeg()
    .toBuffer();

  return annotatedImage;
}
```

**2. Vision API Analysis with OpenAI**
```typescript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function analyzeCalendarImage(
  imageBuffer: Buffer
): Promise<FreeBlock[]> {
  // Convert to base64
  const base64Image = `data:image/jpeg;base64,${imageBuffer.toString('base64')}`;

  // Call Vision API
  const response = await client.chat.completions.create({
    model: 'gpt-5.2',
    messages: [
      {
        role: 'user',
        content: [
          {
            type: 'text',
            text: `Analyze this calendar screenshot.
                   Identify all free time blocks (2+ hours) on weekends.
                   Return as JSON: [{ date: "YYYY-MM-DD", startTime: "HH:MM", endTime: "HH:MM" }]`
          },
          {
            type: 'image_url',
            image_url: { url: base64Image }
          }
        ]
      }
    ]
  });

  // Parse response
  const freeBlocks = JSON.parse(response.choices[0].message.content);
  return freeBlocks;
}
```

**3. Combined Workflow**
```typescript
async function processCalendarScreenshot(
  imageBuffer: Buffer
): Promise<Suggestion[]> {
  // 1. Annotate image with helpful markers
  const annotatedImage = await annotateCalendarImage(imageBuffer, [
    'Looking for 2+ hour free blocks',
    'Focus: Saturdays & Sundays'
  ]);

  // 2. Analyze with Vision API
  const freeBlocks = await analyzeCalendarImage(annotatedImage);

  // 3. Generate suggestions for detected free time
  const suggestions = await generateSuggestionsForBlocks(freeBlocks);

  return suggestions;
}
```

**Tech Stack for Vision Feature:**
- **sharp** (`/lovell/sharp`) - Text overlays, image compositing
- **openai-node** (`/openai/openai-node`) - Vision API, chat completions
- **Docs:** See [ai/guides/sharp-text-overlay-docs.md](../ai/guides/sharp-text-overlay-docs.md) and [ai/guides/openai-sdk-docs.md](../ai/guides/openai-sdk-docs.md)

### Other V2+ Features

**Multi-Calendar Coordination**
```typescript
// Merge events from multiple calendars (both partners)
async function mergeFamilyCalendars(
  calendar1Id: string,
  calendar2Id: string
): Promise<CalendarEvent[]> {
  const [events1, events2] = await Promise.all([
    googleCalendar.getEvents({ calendarId: calendar1Id }),
    googleCalendar.getEvents({ calendarId: calendar2Id })
  ]);

  // Combine and deduplicate
  return [...events1, ...events2].sort((a, b) =>
    a.start.dateTime.localeCompare(b.start.dateTime)
  );
}
```

**ML-Based Recommendations (Replace Rule-Based)**
```typescript
// Use ML model trained on user feedback
async function mlRecommendation(
  profile: FamilyProfile,
  freeBlock: FreeBlock,
  feedback: Feedback[]
): Promise<Activity[]> {
  // Feature engineering
  const features = {
    kidAges: profile.kidAges,
    interests: encodeInterests(profile.interests),
    dayOfWeek: freeBlock.date.getDay(),
    timeOfDay: freeBlock.startTime,
    previousAcceptances: feedback.filter(f => f.accepted).map(f => f.activityType)
  };

  // Call ML model endpoint
  const predictions = await mlService.predict(features);

  return predictions;
}
```

**ChatGPT Action Integration**
```typescript
// Expose as OpenAI plugin/action
export const chatGPTAction = {
  name: 'family_time_suggestions',
  description: 'Get personalized family activity suggestions',
  parameters: {
    type: 'object',
    properties: {
      userId: { type: 'string' },
      date: { type: 'string', format: 'date' }
    }
  },
  async execute({ userId, date }) {
    return await generateSuggestions(userId);
  }
};
```

---

## Security & Privacy

### Data Protection

**Sensitive Data:**
- Google Calendar refresh tokens (encrypted at rest)
- User email addresses
- Family profile (kid ages, interests)

**Encryption:**
```typescript
import crypto from 'crypto';

const algorithm = 'aes-256-gcm';
const key = process.env.ENCRYPTION_KEY; // 32 bytes

function encrypt(text: string): string {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(algorithm, key, iv);

  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');

  const authTag = cipher.getAuthTag();

  return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
}

// Store Google refresh tokens encrypted
await db.users.update({
  where: { id: userId },
  data: {
    googleRefreshToken: encrypt(refreshToken)
  }
});
```

**OAuth Scope Minimization:**
- MVP: Request only `calendar.readonly` + `calendar.events`
- Never request `calendar` (full write access)
- Clearly communicate what we access

**Data Retention:**
- Delete calendar scans after 30 days
- Delete suggestions after 90 days
- User can delete account + all data anytime

---

## Performance & Scalability

### Optimization Strategies

**Caching:**
```typescript
// Cache Google Calendar events (5 min TTL)
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 300 });

async function getCachedEvents(userId: string) {
  const cacheKey = `events:${userId}`;

  let events = cache.get(cacheKey);
  if (!events) {
    events = await googleCalendar.getEvents({ userId });
    cache.set(cacheKey, events);
  }

  return events;
}
```

**Database Indexing:**
```sql
-- Speed up common queries
CREATE INDEX idx_suggestions_user_date ON suggestions(user_id, target_date);
CREATE INDEX idx_suggestions_status ON suggestions(status);
CREATE INDEX idx_calendar_scans_user ON calendar_scans(user_id);
```

**Rate Limiting:**
```typescript
import rateLimit from 'express-rate-limit';

const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // Limit each IP to 100 requests per windowMs
});

app.use('/api/', apiLimiter);
```

**Horizontal Scaling:**
- Stateless API servers (can add more dynos)
- BullMQ for distributed job processing
- PostgreSQL connection pooling

---

## Monitoring & Observability

### Key Metrics

**Product Metrics:**
- Suggestion acceptance rate (target: >40%)
- Email open rate (target: >50%)
- Week 2 retention (target: >60%)
- Thumbs up rate (target: >80%)

**System Metrics:**
- API response time (p95 < 200ms)
- Background job success rate (>99%)
- Google Calendar API quota usage
- Database query performance

**Logging:**
```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Log all API requests
app.use((req, res, next) => {
  logger.info({
    method: req.method,
    path: req.path,
    userId: req.user?.id,
    timestamp: new Date()
  });
  next();
});
```

---

## Deployment

### Environments

**Development:**
- Local: `localhost:3000` (Next.js) + `localhost:4000` (API)
- Database: Docker PostgreSQL
- Redis: Docker Redis

**Staging:**
- Frontend: Vercel preview deployment
- Backend: Railway staging environment
- Database: Railway staging DB

**Production:**
- Frontend: Vercel (`familytime.com`)
- Backend: Railway production (`api.familytime.com`)
- Database: Railway production DB (backups enabled)

### CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install
      - run: npm run build
      - uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}

  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: railway up
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
```

---

## Costs (Estimate)

### MVP (Chrome Extension)
- **Hosting:** $0 (client-side only)
- **Google Calendar API:** Free (up to 1M requests/day)
- **Total:** $0/month

### V1 (First 100 Users)
- **Vercel:** Free tier (hobby plan)
- **Railway:** ~$10/month (Postgres + Redis + API server)
- **Resend:** Free tier (up to 3,000 emails/month)
- **Google Calendar API:** Free
- **Google Places API:** ~$50/month (estimated 10,000 requests)
- **Eventbrite API:** Free
- **Total:** ~$60/month

### V1 (1,000 Users)
- **Vercel:** ~$20/month
- **Railway:** ~$100/month
- **Resend:** ~$40/month (12,000 emails/month)
- **Google Places API:** ~$500/month
- **Total:** ~$660/month

---

## Open Technical Questions

1. **Calendar sync frequency:** Weekly is enough, or need daily?
2. **Image storage:** Where to host activity images? (S3, Cloudinary, embed URLs?)
3. **Email vs SMS:** Is email enough, or add Twilio SMS?
4. **Multi-city support:** How to handle location detection? (IP geolocation, manual input?)
5. **Mobile app priority:** React Native in V2, or wait for V3?

---

## References

- [PRD](prd.md) - Product requirements
- [MVP Spec](mvp.md) - Chrome extension demo
- [Sharp Docs](../ai/guides/sharp-text-overlay-docs.md) - Image processing
- [OpenAI SDK Docs](../ai/guides/openai-sdk-docs.md) - Vision API
- [Google Calendar API](https://developers.google.com/calendar/api)
- [Google Places API](https://developers.google.com/maps/documentation/places/web-service)
- [Eventbrite API](https://www.eventbrite.com/platform/api)
