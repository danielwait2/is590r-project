# Phase 5: V2 - Advanced Features

**Roadmap:** [2026-02-11-roadmap-phase5-v2.md](./2026-02-11-roadmap-phase5-v2.md) - Track progress

**Created:** 2026-02-11
**Timeline:** Months 4-6 (3 months)
**Status:** Pending Phase 4 Success
**Type:** Product Enhancement & Growth

---

## Overview

Add features that increase retention and differentiation based on user feedback from Phase 4. Prioritize features that drive the highest value for premium subscribers and create competitive moats.

**Strategic Focus:** Convert Phase 4 learnings into product improvements that justify premium pricing and increase long-term retention.

**Decision Point:** Only proceed if Phase 4 hit success criteria (>60% retention, >20% WTP, positive unit economics).

---

## Success Criteria

**User Metrics:**
- ✅ 500 active users (5x growth from Phase 4)
- ✅ >60% week-8 retention (up from week-2)
- ✅ 15%+ premium conversion rate
- ✅ NPS score >50 (up from 40)
- ✅ Profitable at current scale

**Feature Validation:**
- Ship at least 2 of the 5 features below
- Each feature measurably improves retention or conversion
- Premium features drive >20% of conversions

---

## Prerequisites

**Required Before Starting:**
- ✅ Phase 4 completed successfully
- ✅ Decision made to proceed to V2
- ✅ 100+ active users from Phase 4
- ✅ At least 15% paying customers
- ✅ 3+ months of user feedback data collected

**Resource Requirements:**
- Runway for 3 months (cash or revenue)
- Developer time for feature development
- Potentially: ML/data science consultant (for 5C)

---

## Feature Prioritization

Based on Phase 4 feedback, prioritize in this order:

1. **Multi-Calendar Support** (Most requested)
2. **Auto-Insert to Calendar** (Reduces friction)
3. **Mobile App** (Engagement)
4. **ML-Based Recommendations** (Differentiation)
5. **Expanded Activity Sources** (Variety)

**Decision:** Pick 2-3 based on:
- User demand (feedback frequency)
- Impact on retention
- Technical complexity vs timeline
- Competitive differentiation

---

## Detailed Feature Plans

### 5A: Multi-Calendar Support (High Priority)

**Goal:** Sync both partners' calendars to find mutual free time

**User Story:** "My spouse and I both have Google Calendars. I want suggestions for times when we're BOTH free."

**Timeline:** 4-6 weeks

---

#### Implementation:

**1. Database Schema Updates**

Add to Prisma schema:

```prisma
model User {
  // ... existing fields
  calendars Calendar[]
}

model Calendar {
  id              String   @id @default(uuid())
  userId          String
  user            User     @relation(fields: [userId], references: [id])

  calendarId      String   // Google Calendar ID
  calendarName    String   // "Work", "Personal", "Spouse's Calendar"
  refreshToken    String   // Encrypted
  accessToken     String   // Encrypted
  tokenExpiresAt  DateTime

  isPrimary       Boolean  @default(false)
  isActive        Boolean  @default(true)

  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  @@unique([userId, calendarId])
  @@index([userId])
}
```

**2. Multi-Calendar OAuth Flow**

Create `src/app/calendars/add/page.tsx`:

```typescript
export default function AddCalendarPage() {
  return (
    <div className="container mx-auto px-4 py-12">
      <h1 className="text-3xl font-bold mb-6">Add Another Calendar</h1>

      <div className="bg-white rounded-lg p-8 shadow">
        <p className="mb-6">
          Connect your partner's calendar to get suggestions for times when you're both free.
        </p>

        <button
          onClick={() => {
            window.location.href = '/api/auth/google/add-calendar';
          }}
          className="bg-blue-600 text-white px-6 py-3 rounded-lg"
        >
          Connect Google Calendar
        </button>
      </div>

      {/* List existing calendars */}
    </div>
  );
}
```

**3. Merge Events from Multiple Calendars**

Update `src/lib/calendar.ts`:

```typescript
export async function fetchAllCalendarEvents(userId: string) {
  const calendars = await prisma.calendar.findMany({
    where: { userId, isActive: true },
  });

  const allEvents = [];

  for (const calendar of calendars) {
    const events = await fetchGoogleCalendarEvents(
      calendar.calendarId,
      calendar.accessToken
    );
    allEvents.push(...events);
  }

  // Deduplicate events (same event might be on multiple calendars)
  const uniqueEvents = deduplicateEvents(allEvents);

  return uniqueEvents;
}

function deduplicateEvents(events: CalendarEvent[]) {
  const seen = new Set();
  return events.filter(event => {
    const key = `${event.summary}-${event.start.dateTime}`;
    if (seen.has(key)) return false;
    seen.add(key);
    return true;
  });
}
```

**4. Update Free Time Detection**

```typescript
export async function detectFreeTimeMultiCalendar(userId: string) {
  const allEvents = await fetchAllCalendarEvents(userId);

  // Use existing free time detection algorithm
  // but with merged event list
  return detectFreeTime(allEvents);
}
```

**5. UI for Calendar Management**

Show list of connected calendars:

```typescript
export default async function CalendarsPage() {
  const calendars = await prisma.calendar.findMany({
    where: { userId },
  });

  return (
    <div className="space-y-4">
      {calendars.map(calendar => (
        <div key={calendar.id} className="bg-white p-4 rounded-lg border flex justify-between items-center">
          <div>
            <h3 className="font-semibold">{calendar.calendarName}</h3>
            <p className="text-sm text-gray-600">
              {calendar.isPrimary ? 'Primary Calendar' : 'Additional Calendar'}
            </p>
          </div>

          <div className="flex gap-2">
            <button
              onClick={() => toggleCalendar(calendar.id)}
              className={`px-4 py-2 rounded ${
                calendar.isActive ? 'bg-green-100 text-green-700' : 'bg-gray-100'
              }`}
            >
              {calendar.isActive ? 'Active' : 'Inactive'}
            </button>

            <button
              onClick={() => removeCalendar(calendar.id)}
              className="px-4 py-2 bg-red-100 text-red-700 rounded"
            >
              Remove
            </button>
          </div>
        </div>
      ))}

      <Link href="/calendars/add" className="block w-full bg-blue-600 text-white text-center py-3 rounded-lg">
        + Add Another Calendar
      </Link>
    </div>
  );
}
```

**Testing:**
- Connect 2 calendars from different Google accounts
- Verify events merged correctly
- Test free time detection with overlapping events
- Ensure no duplicate events shown

**Premium Feature:** Make multi-calendar support premium-only

---

### 5B: Auto-Insert to Calendar (High Priority)

**Goal:** One-click accept → event automatically added to calendar

**User Story:** "I don't want to download .ics files. Just add it to my calendar with one click."

**Timeline:** 2-3 weeks

---

#### Implementation:

**1. Request Write Permissions**

Update OAuth scopes to include write access (already have `calendar.events` from Phase 1).

**2. Update Suggestion Acceptance Endpoint**

Update `src/app/api/suggestions/[id]/accept/route.ts`:

```typescript
import { google } from 'googleapis';

export async function POST(
  req: Request,
  { params }: { params: { id: string } }
) {
  const { userId } = auth();

  const suggestion = await prisma.suggestion.findUnique({
    where: { id: params.id },
    include: { user: true },
  });

  if (!suggestion || suggestion.userId !== userId) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }

  // Get user's primary calendar
  const calendar = await prisma.calendar.findFirst({
    where: { userId, isPrimary: true },
  });

  if (!calendar) {
    return NextResponse.json({ error: 'No calendar connected' }, { status: 400 });
  }

  // Create Google Calendar client
  const oauth2Client = new google.auth.OAuth2(
    process.env.GOOGLE_CLIENT_ID,
    process.env.GOOGLE_CLIENT_SECRET,
  );

  oauth2Client.setCredentials({
    refresh_token: decrypt(calendar.refreshToken),
    access_token: decrypt(calendar.accessToken),
  });

  const calendarAPI = google.calendar({ version: 'v3', auth: oauth2Client });

  // Parse suggestion date/time
  const targetDate = new Date(suggestion.targetDate);
  const [startTime, endTime] = suggestion.targetTimeSlot.split(' - ');

  const event = {
    summary: suggestion.title,
    description: `${suggestion.description}\n\n💰 ${suggestion.estimatedCost}\n\n📍 ${suggestion.address}${suggestion.bookingLink ? `\n\nBook here: ${suggestion.bookingLink}` : ''}`,
    location: suggestion.address,
    start: {
      dateTime: parseDateTime(targetDate, startTime),
      timeZone: 'America/Chicago', // TODO: Get from user profile
    },
    end: {
      dateTime: parseDateTime(targetDate, endTime),
      timeZone: 'America/Chicago',
    },
    reminders: {
      useDefault: false,
      overrides: [
        { method: 'popup', minutes: 24 * 60 },
        { method: 'popup', minutes: 60 },
      ],
    },
  };

  // Insert event
  const response = await calendarAPI.events.insert({
    calendarId: 'primary',
    requestBody: event,
  });

  // Update suggestion status
  await prisma.suggestion.update({
    where: { id: params.id },
    data: {
      status: 'accepted',
      calendarEventId: response.data.id, // Save for later reference
    },
  });

  return NextResponse.json({
    success: true,
    eventLink: response.data.htmlLink,
  });
}
```

**3. Update Email Template**

Change "Add to Calendar" button to call API instead of downloading .ics:

```typescript
<Button
  style={button}
  href={`${process.env.NEXT_PUBLIC_APP_URL}/api/suggestions/${suggestion.id}/accept`}
>
  Add to Calendar (Auto-Insert)
</Button>
```

**4. Add "Tentative" Option**

Allow users to mark as "maybe":

```typescript
export async function POST(req: Request) {
  const { status } = await req.json(); // 'confirmed' or 'tentative'

  const event = {
    // ... existing fields
    status: status === 'tentative' ? 'tentative' : 'confirmed',
    transparency: status === 'tentative' ? 'transparent' : 'opaque',
  };

  // ... rest of implementation
}
```

**Testing:**
- Click "Add to Calendar" in email
- Verify event appears in Google Calendar within 5 seconds
- Test "tentative" vs "confirmed" status
- Handle duplicate clicks gracefully (don't create duplicates)

---

### 5C: ML-Based Recommendations (Medium Priority)

**Goal:** Replace rule-based scoring with ML predictions

**User Story:** "I want suggestions that get better over time based on what I actually accept."

**Timeline:** 6-8 weeks

**Prerequisites:** At least 3 months of feedback data (100+ suggestions with feedback)

---

#### Implementation:

**1. Data Collection & Feature Engineering**

Create training dataset:

```typescript
// Export to CSV for ML training
export async function exportTrainingData() {
  const suggestions = await prisma.suggestion.findMany({
    include: {
      user: {
        include: { profile: true },
      },
    },
  });

  const trainingData = suggestions.map(s => ({
    // Features
    kidAges: s.user.profile.kidAges,
    interests: s.user.profile.interests,
    budgetTier: s.user.profile.budgetTier,
    maxDriveTime: s.user.profile.maxDriveTimeMinutes,
    activityType: s.activityType,
    dayOfWeek: new Date(s.targetDate).getDay(),
    timeOfDay: s.targetTimeSlot,
    estimatedCostNumeric: parseCost(s.estimatedCost),

    // Label (what we're predicting)
    accepted: s.status === 'accepted' ? 1 : 0,
  }));

  return trainingData;
}
```

**2. Train Simple Model**

Use Python (scikit-learn) or TensorFlow.js:

```python
# train_model.py
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split

# Load data
df = pd.read_csv('training_data.csv')

# Features
X = df[['kidAges', 'interests', 'budgetTier', 'activityType', ...]]
y = df['accepted']

# Train/test split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Train model
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)

# Evaluate
accuracy = model.score(X_test, y_test)
print(f"Accuracy: {accuracy}")

# Export model
import joblib
joblib.dump(model, 'recommendation_model.pkl')
```

**3. Inference Endpoint**

Create `src/app/api/ml/predict/route.ts`:

```typescript
import { spawn } from 'child_process';

export async function POST(req: Request) {
  const { features } = await req.json();

  // Call Python script for inference
  const python = spawn('python3', ['predict.py', JSON.stringify(features)]);

  let prediction = '';

  python.stdout.on('data', (data) => {
    prediction += data.toString();
  });

  await new Promise((resolve) => python.on('close', resolve));

  return NextResponse.json({ score: parseFloat(prediction) });
}
```

**4. Integrate into Recommendation Engine**

Update `src/lib/recommendations.ts`:

```typescript
export async function scoreActivityML(activity: Activity, profile: FamilyProfile) {
  const features = {
    kidAges: profile.kidAges,
    interests: profile.interests,
    budgetTier: profile.budgetTier,
    activityType: activity.type,
    // ... more features
  };

  const response = await fetch('/api/ml/predict', {
    method: 'POST',
    body: JSON.stringify({ features }),
  });

  const { score } = await response.json();

  return score; // 0-1 probability of acceptance
}
```

**5. A/B Test ML vs Rules**

```typescript
const useML = Math.random() < 0.5; // 50% of users get ML

if (useML) {
  const mlScore = await scoreActivityML(activity, profile);
  return mlScore * 100; // Scale to 0-100
} else {
  return scoreActivityRuleBased(activity, profile); // Existing algorithm
}

// Track which version performed better
await prisma.analyticsEvent.create({
  data: {
    userId,
    event: 'suggestion_generated',
    properties: { algorithm: useML ? 'ml' : 'rules' },
  },
});
```

**Testing:**
- Compare ML vs rule-based acceptance rates
- Ensure ML predictions make intuitive sense
- Monitor for bias (e.g., always suggesting same type)

**Note:** This is advanced. Consider hiring ML consultant or using AutoML services (Vertex AI, AWS SageMaker).

---

### 5D: Mobile App (Low-Medium Priority)

**Goal:** Native iOS/Android app for better engagement

**Timeline:** 8-12 weeks (substantial effort)

**Recommendation:** Only pursue if:
- Web app retention is strong (>60%)
- Users specifically request mobile app
- Resources available for React Native development

---

#### High-Level Plan:

**1. Tech Stack**
- React Native + Expo
- Shared API with web app
- Push notifications (OneSignal or FCM)

**2. Core Features (MVP)**
- Sign in with Clerk
- View suggestions
- Accept/reject suggestions
- Push notifications for new suggestions

**3. Implementation**

```bash
# Initialize Expo app
npx create-expo-app family-time-mobile

# Install dependencies
npm install @clerk/clerk-expo expo-notifications
```

**4. Shared API**

Reuse existing Next.js API routes. Mobile app calls same endpoints.

**5. Push Notifications**

```typescript
// Replace email with push notification
import * as Notifications from 'expo-notifications';

export async function sendPushNotification(userId: string, suggestion: Suggestion) {
  const user = await prisma.user.findUnique({ where: { id: userId } });

  if (!user.pushToken) return;

  await Notifications.scheduleNotificationAsync({
    content: {
      title: "New activity suggestion! ✨",
      body: suggestion.title,
      data: { suggestionId: suggestion.id },
    },
    trigger: null, // Send immediately
  });
}
```

**Decision Point:** Mobile app is high effort. Only build if web metrics justify it.

---

### 5E: Expanded Activity Sources

**Goal:** More variety and local coverage

**Timeline:** 3-4 weeks

---

#### Implementation:

**1. Yelp API Integration**

```bash
npm install yelp-fusion
```

```typescript
import yelp from 'yelp-fusion';

const client = yelp.client(process.env.YELP_API_KEY);

export async function searchYelpActivities(location: string, interests: string[]) {
  const response = await client.search({
    term: interests.join(' '),
    location: location,
    categories: 'restaurants,parks,museums',
    limit: 20,
  });

  return response.jsonBody.businesses.map(business => ({
    title: business.name,
    description: business.categories.map(c => c.title).join(', '),
    address: business.location.display_address.join(', '),
    imageUrl: business.image_url,
    estimatedCost: business.price || '$$',
    rating: business.rating,
    type: 'restaurant',
    tags: business.categories.map(c => c.alias),
  }));
}
```

**2. Local Business Partnerships**

Reach out to local businesses for featured placement:

```markdown
Email template:
---
Subject: Partnership opportunity - FamilyTime app

Hi [Business Name],

We're building a tool that helps families discover local activities. We have 500+ users in [City].

Would you like to be featured in our weekly suggestions?

Benefits:
- Direct exposure to engaged families
- Highlight special offers/events
- Track referrals and bookings

Let me know if you're interested!
```

Store partner activities in database:

```prisma
model PartnerActivity {
  id              String   @id @default(uuid())
  businessName    String
  contactEmail    String
  // ... same fields as CuratedActivity
  isFeatured      Boolean  @default(false)
  partnershipTier String   // 'free', 'basic', 'premium'
}
```

**3. City-Specific Curations**

Hire local curators or use crowd-sourcing:

```
Create "Chicago Hidden Gems" database
- 50+ activities per city
- Verified by locals
- Seasonal updates (summer vs winter activities)
```

---

## Deliverables

**Features Shipped:**
- ✅ At least 2 of the 5 features above
- ✅ All features tested and bug-free
- ✅ Premium features driving conversions

**Metrics Improvement:**
- ✅ Retention increased by >10%
- ✅ Premium conversion increased by >5%
- ✅ User satisfaction (NPS) improved

**Infrastructure:**
- ✅ ML pipeline (if 5C chosen)
- ✅ Mobile app deployed (if 5D chosen)
- ✅ Partner dashboard (if 5E chosen)

---

## Risks & Mitigation

**Feature Overload:**
- Risk: Too many features, none done well
- Mitigation: Focus on 2 features max
- Validation: Each feature must improve retention by >5%

**ML Accuracy Issues:**
- Risk: ML predictions worse than rules
- Mitigation: A/B test rigorously, keep rules as fallback
- Validation: ML must outperform rules by >10% acceptance rate

**Mobile App Complexity:**
- Risk: Mobile app takes too long, diverts focus
- Mitigation: Only build if web app proven
- Validation: User research showing mobile demand

---

## References

- [High-Level Roadmap](./2026-02-11-high-level_mvp-project-roadmap.md)
- [Architecture Document](../../aiDocs/architecture.md)
- [Google Calendar API](../guides/google-calendar-api_context7.md)

---

**Last Updated:** 2026-02-11
**Version:** 1.0
