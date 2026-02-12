# Technical Architecture

**Version:** 1.0
**Last Updated:** 2026-02-11
**Status:** Draft

---

## Overview

This document defines the technical architecture for the Family Quality Time App across six development phases:

1. **Phase 1: MVP (Chrome Extension)** - Client-side demo for validation (4-5 days)
2. **Phase 2-4: V1 (Full Product)** - Production web app with backend (12 weeks total)
   - Phase 2: Core web app foundation (4 weeks)
   - Phase 3: Polish & automation (4 weeks)
   - Phase 4: Beta launch & validation (4 weeks)
3. **Phase 5: V2 (Advanced Features)** - Enhanced features & growth (3 months)
4. **Phase 6: V3 (Ecosystem)** - Platform expansion & moat building (6 months)

**Roadmap Reference:** See [ai/roadmaps/](../ai/roadmaps/) for detailed implementation plans for each phase.

---

## Product Evolution Overview

### Phase Progression

| Phase | Name | Duration | Goal | Key Deliverables |
|-------|------|----------|------|------------------|
| **Phase 1** | MVP Demo | 4-5 days | Validate core value prop | Chrome extension, demo video |
| **Phase 2** | V1 Core | 4 weeks | Build production foundation | Web app, auth, database, manual suggestions |
| **Phase 3** | V1 Polish | 4 weeks | Automation & UX | Weekly cron jobs, email delivery, polished UI |
| **Phase 4** | V1 Beta | 4 weeks | Validation & monetization | 100 users, pricing, metrics validation |
| **Phase 5** | V2 Features | 3 months | Growth & retention | Multi-calendar, mobile app, ML recommendations |
| **Phase 6** | V3 Ecosystem | 6 months | Moat building | AI integrations, bookings, B2B partnerships |

### Technical Evolution

**Data Flow Progression:**

```
Phase 1 (MVP):
User → Extension → Google Calendar API → User
(Client-side only, manual trigger)

Phase 2-3 (V1 Core & Polish):
Cron Job → Backend → Calendar API → Recommendation Engine → Email → User
(Server-side automation, persistent storage)

Phase 5 (V2):
Multiple Calendars → ML Model → Mobile App + Web + Email → User
(Multi-channel, intelligent recommendations)

Phase 6 (V3):
AI Assistants → API → Booking Partners → Stripe → User
(Platform ecosystem, network effects)
```

### Architecture Complexity Growth

| Capability | Phase 1 | Phases 2-4 | Phase 5 | Phase 6 |
|------------|---------|------------|---------|---------|
| **Frontend** | Vanilla JS popup | Next.js web app | + React Native mobile | + AI assistant UIs |
| **Backend** | None | Node.js/Express | + ML service | + Partner APIs |
| **Database** | Chrome storage | PostgreSQL | + Analytics DB | + Data warehouse |
| **APIs** | Calendar only | + Places, Eventbrite | + Yelp, TripAdvisor | + Booking platforms |
| **Automation** | Manual | Weekly cron | + Real-time triggers | + Event-driven |
| **Users** | 5 testers | 100 beta | 500 active | 1,000+ |

**Decision Points:**
- **After Phase 1:** Proceed to V1 only if demo gets "wow" reactions from 5+ people
- **After Phase 4:** Proceed to V2 only if >40% acceptance rate, >30% WTP, positive feedback
- **After Phase 5:** Proceed to V3 only if >60% retention, 15%+ conversion, profitable unit economics

---

## MVP Architecture: Chrome Extension

**Implementation Plan:** [Phase 1 Detailed Plan](../ai/roadmaps/2026-02-11-phase1-mvp-chrome-extension.md) | [Phase 1 Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase1-mvp.md)

**Timeline:** 4-5 days | **Type:** Proof-of-concept demo

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

**Approach:** Hard-coded JSON array of 10-15 curated family activities

**Activity Schema:**
- ID, title, type (museum/park/restaurant/event)
- Description, address, image URL, booking link
- Estimated cost, duration, age range
- Tags for matching (e.g., "science", "outdoor", "food")

**Matching Algorithm:**
- Simple keyword matching between user's "want to try" list and activity tags
- Score-based ranking
- Return top 2 matches

**Reference:** See [Phase 1 detailed plan](../ai/roadmaps/2026-02-11-phase1-mvp-chrome-extension.md) for implementation code

### Limitations (MVP)

- No backend → No automation, user must manually open extension
- No database → No learning from past suggestions
- No real recommendation engine → Simple keyword matching
- Single calendar only → No multi-partner coordination
- Hard-coded activities → Limited to 10-15 pre-curated items
- No email/SMS → Only in-extension notifications

---

## V1 Architecture: Full Product

**Implementation Plans:**
- [Phase 2: Core Web App](../ai/roadmaps/2026-02-11-phase2-v1-core-web-app.md) | [Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase2-v1-core.md)
- [Phase 3: Polish & Automation](../ai/roadmaps/2026-02-11-phase3-v1-polish-automation.md) | [Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase3-v1-polish.md)
- [Phase 4: Beta Launch](../ai/roadmaps/2026-02-11-phase4-v1-launch-beta.md) | [Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase4-v1-launch.md)

**Timeline:** 12 weeks (Phases 2-4 combined) | **Type:** Production MVP

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

**High-Level Flow:**

1. **Detect Free Time**
   - Fetch calendar events for next 7 days
   - Filter to weekend business hours (Sat/Sun 9am-6pm)
   - Find contiguous 2-4 hour blocks with no events
   - Store in database

2. **Query Activity Sources**
   - Google Places API (parks, museums, zoos, aquariums)
   - Eventbrite API (local events)
   - Curated database (manually seeded activities)
   - Filter by: location, date, budget, rating

3. **Match & Score**
   - Interest match (30 points)
   - "Want to Try" match (25 points)
   - Age appropriateness (20 points)
   - Budget fit (15 points)
   - Drive time (10 points)
   - Penalties: Recently suggested (-20), Similar type last week (-10)
   - Minimum threshold: 30 points

4. **Generate Suggestions**
   - Select top 2 free blocks
   - Score all activities for each block
   - Choose best match per block
   - Store in database

5. **Deliver via Email**
   - Send via Resend API
   - Schedule for Friday 8am
   - Include title, description, address, cost, booking link
   - Use React Email templates

**Reference:** See [Phase 2 detailed plan](../ai/roadmaps/2026-02-11-phase2-v1-core-web-app.md) for full implementation

---

## V2+ Architecture: Future Enhancements

**Implementation Plans:**
- [Phase 5: V2 Advanced Features](../ai/roadmaps/2026-02-11-phase5-v2-advanced-features.md) | [Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase5-v2.md)
- [Phase 6: V3 Ecosystem](../ai/roadmaps/2026-02-11-phase6-v3-ecosystem.md) | [Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase6-v3.md)

**Timeline:** Phase 5 (3 months), Phase 6 (6 months) | **Type:** Growth & scale

### Phase 5 Features (V2)

**Priority Features:**
1. Multi-calendar support (both partners' calendars)
2. Auto-insert to Google Calendar (write permissions)
3. Mobile app (React Native)
4. ML-based recommendations (replace rule-based)
5. Expanded activity sources (Yelp, TripAdvisor, etc.)

### Phase 6 Features (V3)

**Ecosystem Expansion:**
1. AI assistant integrations (ChatGPT, Google Assistant, Alexa)
2. In-app booking/reservations (Stripe payment processing)
3. Multi-family coordination
4. B2B partnerships (corporate wellness)
5. Apple Calendar support
6. Social features (share experiences)

---

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

**Implementation Overview:**

1. **Image Processing** (Sharp library)
   - Add text overlays to calendar screenshots
   - Annotate with date markers
   - Convert to optimized format for Vision API

2. **Vision API Analysis** (OpenAI)
   - Send annotated image to GPT-4 Vision
   - Prompt: Identify 2+ hour free blocks on weekends
   - Parse JSON response with date/time slots

3. **Suggestion Generation**
   - Use detected free time blocks
   - Generate recommendations via standard engine
   - Return suggestions without requiring OAuth

**Tech Stack:**
- Sharp for image processing
- OpenAI Node.js SDK for Vision API
- See: [Sharp Guide](../ai/guides/sharp-text-overlay-docs.md) | [OpenAI Guide](../ai/guides/openai-sdk-docs.md)

### Other V2+ Features

**Multi-Calendar Coordination (Phase 5)**
- Merge events from both partners' calendars
- Find mutual free time
- Support up to 2 calendars per account
- Deduplicate shared events

**ML-Based Recommendations (Phase 5)**
- Replace rule-based scoring with machine learning
- Train on user feedback data (acceptances/rejections)
- Features: kid ages, interests, time preferences, historical patterns
- Improve accuracy over time

**ChatGPT Action Integration (Phase 6)**
- Expose API as OpenAI plugin/action
- Users ask ChatGPT for family activity suggestions
- ChatGPT calls FamilyTime API with user context
- Returns formatted suggestions in conversation

**Reference:** See Phase 5-6 detailed plans for implementation

---

## Payment & Monetization Architecture

**Timeline:** Phase 4 (freemium model), Phase 6 (transaction fees)

### Subscription Billing (Phase 4)

**Tech Stack:**
- **Payment processor:** Stripe
- **Integration:** Stripe Checkout + Customer Portal
- **Webhook handling:** Next.js API routes

**Pricing Tiers:**

| Feature | Free | Premium ($9.99/mo) |
|---------|------|-------------------|
| Suggestions/week | 1 | 3 |
| Calendars | 1 | 2 |
| Interests | 3 categories | Unlimited |
| Email delivery | ✅ | ✅ |
| SMS delivery | ❌ | ✅ |
| Mobile app | ❌ | ✅ |
| ML recommendations | ❌ | ✅ |

**Database Schema Addition:**

```sql
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  stripe_customer_id VARCHAR(255) UNIQUE NOT NULL,
  stripe_subscription_id VARCHAR(255) UNIQUE,
  stripe_price_id VARCHAR(255),

  status VARCHAR(50) CHECK (status IN ('active', 'canceled', 'past_due', 'trialing')),
  tier VARCHAR(20) CHECK (tier IN ('free', 'premium')),

  current_period_start TIMESTAMP,
  current_period_end TIMESTAMP,
  cancel_at_period_end BOOLEAN DEFAULT FALSE,

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(user_id)
);

CREATE INDEX idx_subscriptions_user ON subscriptions(user_id);
CREATE INDEX idx_subscriptions_stripe_customer ON subscriptions(stripe_customer_id);
```

**Stripe Webhook Handling:**

**Events to handle:**
- `customer.subscription.created` - New subscription started
- `customer.subscription.updated` - Subscription modified
- `customer.subscription.deleted` - Subscription canceled

**Actions:**
- Verify webhook signature
- Update subscription status in database
- Sync billing period dates
- Handle downgrades to free tier

**Implementation:** Next.js API route at `/api/webhooks/stripe`

### Transaction Fees (Phase 6)

**Use Case:** Commission on bookings made through the app

**Implementation:**
- Stripe Connect for marketplace payments
- Partner venues onboarded as Stripe Connected Accounts
- App takes 10-15% commission on bookings
- Automatic payouts to partners

**Example Flow:**
```
User books museum tickets ($40)
  → Stripe charges user $40
  → App receives $6 (15% commission)
  → Museum receives $34 (85% payout)
```

**Reference:** See [Stripe Guide](../ai/guides/stripe_context7.md) for detailed implementation

---

## Security & Privacy

### Data Protection

**Sensitive Data:**
- Google Calendar refresh tokens (encrypted at rest)
- User email addresses
- Family profile (kid ages, interests)

**Encryption:**
- Algorithm: AES-256-GCM
- Encrypt at rest: Google refresh tokens, sensitive user data
- Key management: Environment variables (32-byte key)
- Format: `IV:AuthTag:EncryptedData` (hex encoded)

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
- Google Calendar events: 5-minute TTL (NodeCache)
- Activity search results: 15-minute TTL
- User profiles: 10-minute TTL
- Pattern: Check cache → Fetch if miss → Store → Return

**Database Indexing:**
```sql
-- Speed up common queries
CREATE INDEX idx_suggestions_user_date ON suggestions(user_id, target_date);
CREATE INDEX idx_suggestions_status ON suggestions(status);
CREATE INDEX idx_calendar_scans_user ON calendar_scans(user_id);
```

**Rate Limiting:**
- 100 requests per 15 minutes per IP
- Applied to all `/api/*` routes
- Library: express-rate-limit

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
- Library: Winston
- Format: JSON structured logs
- Levels: error, warn, info, debug
- Outputs: error.log (errors only), combined.log (all)
- Middleware: Log all API requests (method, path, userId, timestamp)

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

**GitHub Actions workflow:**
- Trigger: Push to main branch
- Frontend: Build Next.js → Deploy to Vercel
- Backend: Deploy to Railway
- Secrets: VERCEL_TOKEN, RAILWAY_TOKEN (stored in GitHub)

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

### Strategic Documents
- [PRD](prd.md) - Product requirements and strategy
- [MVP Spec](mvp.md) - Chrome extension demo
- [Context](context.md) - Current project overview and focus
- [Market Research](../ai/guides/quality-time-app-market-research.md) - Competitive analysis and user insights

### Implementation Roadmaps

**High-Level:**
- [Overall Project Roadmap](../ai/roadmaps/2026-02-11-high-level_mvp-project-roadmap.md) - All 6 phases overview

**Phase 1 (MVP - 4-5 days):**
- [Phase 1 Detailed Plan](../ai/roadmaps/2026-02-11-phase1-mvp-chrome-extension.md)
- [Phase 1 Roadmap Tracker](../ai/roadmaps/2026-02-11-roadmap-phase1-mvp.md)

**Phases 2-4 (V1 - 12 weeks):**
- [Phase 2 Detailed Plan](../ai/roadmaps/2026-02-11-phase2-v1-core-web-app.md) | [Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase2-v1-core.md)
- [Phase 3 Detailed Plan](../ai/roadmaps/2026-02-11-phase3-v1-polish-automation.md) | [Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase3-v1-polish.md)
- [Phase 4 Detailed Plan](../ai/roadmaps/2026-02-11-phase4-v1-launch-beta.md) | [Roadmap](../ai/roadmaps/2026-02-11-roadmap-phase4-v1-launch.md)

**Phase 5 (V2 - 3 months):**
- [Phase 5 Detailed Plan](../ai/roadmaps/2026-02-11-phase5-v2-advanced-features.md)
- [Phase 5 Roadmap Tracker](../ai/roadmaps/2026-02-11-roadmap-phase5-v2.md)

**Phase 6 (V3 - 6 months):**
- [Phase 6 Detailed Plan](../ai/roadmaps/2026-02-11-phase6-v3-ecosystem.md)
- [Phase 6 Roadmap Tracker](../ai/roadmaps/2026-02-11-roadmap-phase6-v3.md)

### Technical Guides

**Core Libraries:**
- [Google Calendar API](../ai/guides/google-calendar-api_context7.md) - Calendar integration
- [Clerk](../ai/guides/clerk_context7.md) - Authentication
- [Prisma](../ai/guides/prisma_context7.md) - Database ORM
- [Next.js](../ai/guides/nextjs_context7.md) - Frontend framework
- [Resend](../ai/guides/resend_context7.md) - Email delivery
- [React Email](../ai/guides/react-email_context7.md) - Email templates
- [BullMQ](../ai/guides/bullmq_context7.md) - Job scheduling
- [Stripe](../ai/guides/stripe_context7.md) - Payment processing (Phase 6)

**Advanced Features:**
- [Sharp](../ai/guides/sharp-text-overlay-docs.md) - Image processing for Vision API
- [OpenAI SDK](../ai/guides/openai-sdk-docs.md) - Vision API and AI features

### External APIs
- [Google Calendar API](https://developers.google.com/calendar/api)
- [Google Places API](https://developers.google.com/maps/documentation/places/web-service)
- [Eventbrite API](https://www.eventbrite.com/platform/api)
