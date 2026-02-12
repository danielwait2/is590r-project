# Product Requirements Document: Family Quality Time App

**Version:** 1.0 (MVP)
**Last Updated:** 2026-02-11
**Status:** Draft
**Product Name:** TBD (working title: "TimeWeaver" or "FamilyOS")

---

## Executive Summary

We're building a **calendar-native family activity recommendation engine** that solves the "Desire-to-Action Gap"—the friction between wanting to spend quality time together and actually making it happen.

Unlike existing family calendar apps (Cozi, TimeTree) that only organize existing plans, or date-night apps (Cobble) that require manual browsing, we **automatically detect free time, match it to family interests, and push specific, actionable suggestions** directly into Google Calendar.

Our competitive advantage is **seamless integration + intelligent context**—we're building an AI-first backend that knows each family deeply (kids' ages, preferences, constraints) and works through multiple interfaces (our app, AI assistants, partner integrations).

**Target Customer:** Dual-income parents with children under 10 who are calendar-dependent and suffer from time poverty and cognitive load.

**MVP Goal:** Validate that families will (1) grant calendar access, (2) act on our suggestions >40% of the time, and (3) pay $8-10/month for the service.

---

## Problem Statement

### The Core Problem: Desire-to-Action Gap

Couples and families have mental "want to try" lists—restaurants, museums, parks, activities—but these desires rarely convert to actual experiences.

**Why this happens:**
1. **Planning friction** - Coordinating schedules, finding what's open, booking, driving directions
2. **Time blindness** - Free time appears in the calendar, but no plan exists to fill it
3. **Discovery effort** - Finding age-appropriate, local, time-appropriate activities requires research
4. **Decision fatigue** - Busy parents are exhausted; "what should we do this weekend?" becomes overwhelming

**Current workarounds fail:**
- Setting recurring "date night planning" reminders → still requires manual research and booking
- Browsing Eventbrite/Yelp when motivated → happens infrequently, misses opportunities
- Asking ChatGPT for ideas → lacks persistent family context, requires re-explaining every time
- Using family calendar apps (Cozi, TimeTree) → only organize existing plans, don't suggest new ones

### Market Evidence

From our research ([ai/guides/quality-time-app-market-research.md](ai/guides/quality-time-app-market-research.md)):

- **Time poverty is real:** Dual-earner couples work 50+ hours/week each, leaving fragmented free time that must be planned to be used well
- **4+ hours/week** spent manually entering events into family calendars (per Sense's research)
- **Cognitive load of "what to do with kids this weekend"** is a top complaint in parenting communities
- **Latent demand:** Families are actively searching for "smarter" tools beyond basic calendar apps

---

## Target Customer

### Primary Persona: "Calendar-Dependent Parents"

**Demographics:**
- Dual-income households with children under 10
- Household income: $80k-200k+
- Tech-savvy, already use Google Calendar daily
- Urban/suburban (access to variety of local activities)

**Psychographics:**
- Feel guilty about not spending enough quality time with family
- Maintain mental or actual "want to try" lists
- Value experiences over possessions
- Willing to pay for time-saving solutions

**Behavioral:**
- Calendar is their "source of truth" (check it multiple times daily)
- Exhausted by decision-making and planning
- Frustrated by good intentions that never materialize

**Pain Points:**
- Free time appears but gets wasted ("we should do something... but what?")
- Planning a single outing takes 30-60 minutes (research, coordinate, book)
- Same activities on repeat (default to familiar parks/restaurants)
- Partner coordination is effortful ("Are you free? When? What do you want to do?")

### Market Size

- **US dual-income families with kids under 10:** ~10-15 million households
- **Calendar-dependent subset:** ~50% = 5-7 million households
- **Addressable market (willing to grant calendar access + pay):** Conservative estimate 5-10% = 250k-700k households
- **Revenue potential at $10/month:** $30M-84M annual revenue at full penetration

---

## Solution Overview

### Core Value Proposition

**"We turn your free time into quality time—automatically."**

We bridge the gap between **stated desires** (what you want to do), **available time** (when you're free), and **local opportunities** (what's actually happening nearby).

### How It Works (User Experience)

**One-time Setup (5 minutes):**
1. Connect Google Calendar (OAuth)
2. Create family profile:
   - Kids' names, ages
   - Interests/preferences (outdoors, museums, food, etc.)
   - Constraints (budget, max drive time, dietary restrictions)
3. Add "Want to Try" list (optional)

**Ongoing (Zero Manual Effort):**
1. **Every week**, our engine scans your calendar for free 2-4 hour blocks
2. **Matches** free time to your profile + local inventory (Google Places, Eventbrite, curated sources)
3. **Generates** 2-3 specific, actionable suggestions:
   - "This Saturday 10am-12pm: Visit Children's Discovery Museum"
   - Includes: address, booking link, why it's a good fit, estimated cost
4. **Delivers** suggestions via:
   - In-app notification
   - Email digest (Friday mornings)
   - Tentative calendar event (opt-in)
5. **One-click accept** → adds to calendar with all details

**Feedback Loop:**
- User accepts/rejects suggestions
- System learns preferences over time
- Improves recommendation quality

### Key Differentiators

**vs. Family Calendar Apps (Cozi, TimeTree):**
- They organize existing plans; we **proactively suggest new ones**
- They require manual entry; we **auto-detect free time and auto-suggest**

**vs. Date Night Apps (Cobble, Howbout):**
- They're couples-only; we handle **families with kids** (age-appropriate, nap-aware)
- They require browsing; we **push the right suggestion at the right time**

**vs. General AI Assistants (ChatGPT, Gemini):**
- They have no memory; we **remember your family profile permanently**
- They require re-prompting; we **proactively watch your calendar**
- They give generic ideas; we give **hyper-local, time-specific, actionable plans**

**vs. Google Calendar (with Goals feature):**
- Goals are self-improvement tasks; we're **family experiences**
- Goals don't connect to local inventory; we **integrate real-time event data**
- Goals are generic; we're **deeply family-specific** (ages, preferences, constraints)

---

## Competitive Landscape

### Direct Competitors

| Product | Category | Strengths | Weaknesses | Our Advantage |
|---------|----------|-----------|------------|---------------|
| **Cozi** | Family organizer | Market leader, shared calendar, $39-60/year | Manual entry, no AI, no activity suggestions | We auto-suggest experiences |
| **TimeTree** | Shared calendar | Clean UI, good sync | No intelligence, organizing only | We proactively plan |
| **Cobble** | Date night app | Swipe UX, local ideas | Couples-only, manual browsing | Family-focused, auto-push |
| **Google Calendar** | Calendar platform | Ubiquitous, trusted | Generic Goals, no family context | Deep personalization |
| **ChatGPT** | AI assistant | Conversational, flexible | No memory, requires prompting | Persistent context, proactive |

### Market Position

We're creating a **new category**: **"Family Time OS"**—the intelligent layer that sits between your calendar and your family's desires.

**Positioning statement:**
*"For busy parents who want more quality family time but struggle with planning, [Product Name] is the only calendar-native assistant that automatically turns your free time into memorable experiences—without any manual effort."*

---

## MVP Scope

### MVP Philosophy: Simple, Fast, Validated

The MVP exists to validate **three critical assumptions:**

1. **Trust:** Will families grant us calendar read access?
2. **Value:** Will they act on our suggestions (>40% acceptance rate)?
3. **Willingness to Pay:** Will they pay $8-10/month?

**Timeline:** 12 weeks to launch
**Team:** 1-2 developers + 1 designer (optional)

### MVP Features (In-Scope)

#### Must-Have (P0)

**1. User Onboarding**
- Google Calendar OAuth integration (read-only for MVP)
- Family profile creation:
  - Number of kids + ages
  - 5-7 interest categories (dropdown selection)
  - Budget constraint (slider: $0-20, $20-50, $50+)
  - Max drive time (dropdown: 15min, 30min, 45min)
- Simple "Want to Try" list (text input, 5-10 items)

**2. Calendar Analysis**
- Scan connected calendar once per week (Sunday nights)
- Identify free blocks of 2-4 hours on weekends (Sat/Sun 9am-6pm)
- Exclude blocks with existing events

**3. Recommendation Engine**
- **Data sources (MVP):**
  - Google Places API (restaurants, parks, museums)
  - Eventbrite API (local events)
  - Manual curation (10-15 "family gems" per city)
- **Matching logic (rule-based for MVP):**
  - Filter by: kid age-appropriateness, budget, drive time
  - Keyword match to family interests
  - Bias toward "Want to Try" list items
- **Output:** 2 suggestions per week (1 Saturday, 1 Sunday)

**4. Suggestion Delivery**
- **Primary:** Email digest (Friday 8am)
  - Subject: "This weekend's family time ideas"
  - 2 suggestions with: photo, title, time slot, address, description, cost, booking link
- **Secondary:** In-app notification (Friday morning)

**5. One-Click Accept**
- Email has "Add to Calendar" button
- Generates .ics file with event details
- User manually adds to Google Calendar (for MVP; auto-insert in v2)

**6. Feedback Collection**
- After each suggestion, track: Accepted, Ignored, Rejected
- Simple thumbs up/down after event time passes (email follow-up)
- Use to improve future suggestions

**7. Admin Dashboard (Internal Only)**
- View all users, their profiles, suggestions sent, acceptance rates
- Manually trigger suggestion generation (for testing)

#### Nice-to-Have (P1 - Include if Time)

- Multiple calendar support (combine 2 calendars for couples)
- SMS delivery option
- "Snooze" suggestion (move to next week)

#### Explicitly Out of Scope (v2+)

- Auto-insert to calendar (requires write permissions, higher trust bar)
- Real-time booking/reservations
- Babysitter coordination
- Multi-family coordination
- AI/ML-based recommendations (use rule-based for MVP)
- Mobile app (web-only for MVP)
- ChatGPT/Gemini integration
- Social features (share with friends)
- In-app chat/support

### MVP User Flow

```
1. User hears about product (friend referral, ad, etc.)
   ↓
2. Lands on landing page → "Get quality family time, automatically"
   ↓
3. Clicks "Get Started" → OAuth to Google Calendar
   ↓
4. Completes onboarding (5-minute form)
   ↓
5. Confirmation: "You'll get your first suggestions Friday!"
   ↓
6. [Friday 8am] Receives email with 2 suggestions
   ↓
7. Clicks "Add to Calendar" on one suggestion
   ↓
8. [Following Monday] Receives "How was it?" email
   ↓
9. Clicks thumbs up → positive feedback recorded
   ↓
10. [Next Friday] Receives improved suggestions
```

---

## Success Metrics

### Primary Metrics (North Star)

**1. Suggestion Acceptance Rate**
- **Target (MVP):** >40% of suggestions are accepted (added to calendar)
- **Hypothesis:** If users act on suggestions, the product has real value
- **Measurement:** Track "Add to Calendar" clicks

**2. Actual Attendance Rate**
- **Target:** >70% of accepted suggestions actually happen
- **Hypothesis:** Suggestions are realistic and well-matched
- **Measurement:** Post-event survey ("Did you go?")

**3. Willingness to Pay**
- **Target:** >30% of users would pay $8-10/month
- **Hypothesis:** Value is strong enough to justify subscription
- **Measurement:** End-of-trial survey + actual conversion when paywall introduced

### Secondary Metrics

- **Calendar connection rate:** % who complete OAuth (target: >80%)
- **Onboarding completion rate:** % who finish profile setup (target: >70%)
- **Email open rate:** % who open Friday digest (target: >50%)
- **Thumbs up rate:** % positive feedback on attended events (target: >80%)
- **Week 2 retention:** % who get 2nd week of suggestions (target: >60%)

### Learning Goals (Qualitative)

- What types of suggestions get the highest acceptance? (new restaurants? parks? events?)
- What budget tier is most common?
- Do families prefer Saturday or Sunday suggestions?
- What feedback do they give when rejecting suggestions?
- How many suggestions per week is optimal? (2, 3, 5?)

---

## Technical Architecture

### Tech Stack (Recommendation)

**Frontend:**
- **Landing page:** Next.js + Tailwind CSS
- **Onboarding flow:** React + TypeScript
- **Email templates:** React Email or Mjml

**Backend:**
- **API:** Node.js + Express (or Python + FastAPI)
- **Database:** PostgreSQL (store users, profiles, suggestions, feedback)
- **Job scheduler:** Cron jobs or BullMQ (for weekly suggestion generation)

**Integrations:**
- **Google Calendar API:** OAuth 2.0 for read access
- **Google Places API:** Activity discovery
- **Eventbrite API:** Event discovery
- **Email delivery:** SendGrid or Resend
- **Authentication:** Clerk or Auth0

**Hosting:**
- **Web app:** Vercel or Railway
- **Database:** Supabase or Railway
- **Background jobs:** Vercel Cron or separate worker dyno

### System Architecture (Simplified)

```
┌─────────────┐
│   User      │
│  (Browser)  │
└──────┬──────┘
       │
       ↓ OAuth + Profile Setup
┌─────────────────────────┐
│   Web App (Next.js)     │
│   - Landing page        │
│   - Onboarding flow     │
│   - Profile management  │
└───────────┬─────────────┘
            │
            ↓ API Calls
┌─────────────────────────┐
│   API Server            │
│   - User CRUD           │
│   - Calendar sync       │
│   - Suggestion engine   │
│   - Feedback collection │
└───────┬────────┬────────┘
        │        │
        ↓        ↓
┌──────────┐  ┌──────────────────┐
│PostgreSQL│  │ External APIs:   │
│ Database │  │ - Google Calendar│
│          │  │ - Google Places  │
│          │  │ - Eventbrite     │
└──────────┘  └──────────────────┘
        ↑
        │
┌───────┴──────────┐
│  Weekly Cron Job │
│  (Sundays 8pm)   │
│  - Scan calendars│
│  - Generate recs │
│  - Queue emails  │
└──────────────────┘
        ↓
┌──────────────────┐
│  Email Service   │
│  (SendGrid)      │
│  - Friday digest │
│  - Follow-ups    │
└──────────────────┘
```

### Data Models (Core)

**User**
```typescript
{
  id: string
  email: string
  googleCalendarId: string
  createdAt: timestamp
  onboardingCompleted: boolean
}
```

**FamilyProfile**
```typescript
{
  id: string
  userId: string
  numKids: number
  kidAges: number[] // e.g., [3, 7]
  interests: string[] // e.g., ["outdoors", "museums", "food"]
  budgetTier: string // "low" | "medium" | "high"
  maxDriveTime: number // minutes
  wantToTryList: string[] // text items
}
```

**Suggestion**
```typescript
{
  id: string
  userId: string
  generatedAt: timestamp
  targetDate: date
  targetTimeSlot: string // e.g., "10am-12pm"
  activityType: string // "restaurant" | "park" | "museum" | "event"
  title: string
  description: string
  address: string
  bookingLink: string
  estimatedCost: number
  imageUrl: string
  status: "sent" | "accepted" | "rejected" | "ignored"
  feedback: "thumbsUp" | "thumbsDown" | null
}
```

**CalendarScan**
```typescript
{
  id: string
  userId: string
  scannedAt: timestamp
  freeBlocks: {
    date: date
    startTime: string
    endTime: string
  }[]
}
```

### API Endpoints (MVP)

**Authentication**
- `POST /auth/google` - Initiate Google OAuth
- `GET /auth/callback` - OAuth callback handler

**User Management**
- `GET /user/profile` - Get user + family profile
- `PUT /user/profile` - Update family profile

**Suggestions**
- `GET /suggestions` - Get all suggestions for user
- `POST /suggestions/:id/accept` - Mark suggestion as accepted
- `POST /suggestions/:id/reject` - Mark suggestion as rejected
- `POST /suggestions/:id/feedback` - Submit thumbs up/down

**Admin (Internal)**
- `GET /admin/users` - List all users
- `POST /admin/generate-suggestions/:userId` - Manually trigger suggestion generation

---

## Algorithm: Recommendation Engine (MVP)

### Step 1: Detect Free Time

**Input:** Google Calendar events for next 7 days
**Logic:**
1. Filter to Saturday and Sunday
2. Define "business hours": 9am-6pm
3. Find contiguous blocks of 2-4 hours with no events
4. Exclude blocks starting after 4pm (too late for family activities)

**Output:** List of free blocks (e.g., "Saturday 10am-2pm", "Sunday 1pm-4pm")

### Step 2: Query Activity Sources

**For each free block:**

**A. Google Places API**
- Query: "family activities near [user zip code]"
- Filters:
  - Type: park, museum, playground, zoo, aquarium, amusement_park
  - Open during target time slot
  - Rating > 4.0
- Return top 10 results

**B. Eventbrite API**
- Query: Events in [user city] on [target date]
- Filters:
  - Categories: Family & Education, Arts, Food & Drink
  - Start time within target slot
  - Price < user budget tier
- Return top 10 results

**C. Curated Database (Manual)**
- Pre-seeded list of 10-15 "family gems" per city
- Tagged with: age range, type, cost
- Include: local farms, special libraries, hidden parks, kid-friendly cafes

### Step 3: Match & Score

**For each candidate activity:**

**Scoring rubric (0-100 points):**
- **Interest match (30 points):** Keywords in activity description match family interests
- **Want to Try match (25 points):** Activity title/type matches Want to Try list item
- **Age appropriateness (20 points):** Activity age range includes all kids' ages
- **Budget fit (15 points):** Cost within budget tier
- **Drive time (10 points):** Distance < max drive time

**Apply penalties:**
- -20 if activity was suggested in last 4 weeks (avoid repetition)
- -10 if similar activity type was suggested last week

**Sort by score, select top 2**

### Step 4: Format Suggestion

**For each selected activity:**

```json
{
  "title": "Children's Discovery Museum",
  "description": "Interactive science and art exhibits for kids 2-10",
  "targetDate": "Saturday, Feb 15",
  "timeSlot": "10:00am - 12:00pm",
  "address": "123 Main St, Chicago, IL",
  "imageUrl": "https://...",
  "estimatedCost": "$35 (2 adults + 2 kids)",
  "bookingLink": "https://discoverymuseum.org/tickets",
  "whyWeChose": "Perfect for ages 3 and 7, indoors (in case of rain), matches your 'science' interest"
}
```

### Step 5: Deliver

- Insert into database with status "sent"
- Queue email for Friday 8am
- Send in-app notification (if user has app open)

---

## Go-to-Market Strategy

### Pre-Launch (Weeks 1-4)

**Goal:** Recruit 20 beta families

**Tactics:**
1. **Personal network:** Reach out to friends/colleagues with young kids
2. **Parenting communities:** Post in local Facebook groups, Reddit (r/Parenting, city subreddits)
3. **Cold outreach:** Email 50 local parent influencers/bloggers
4. **Pitch:** "Free for 4 weeks, help us build the family app you wish existed"

**Deliverable:** Waitlist landing page with simple signup form

### Launch (Weeks 5-8)

**Goal:** 50 active users, gather feedback

**Tactics:**
1. **Email beta users:** Grant access, send onboarding instructions
2. **Weekly check-ins:** Manual outreach to ask "How was the suggestion?"
3. **Iterate fast:** Fix bugs, improve suggestions based on feedback

### Early Growth (Weeks 9-12)

**Goal:** 100 users, validate willingness to pay

**Tactics:**
1. **Referral program:** Give beta users 3 invite codes
2. **Content marketing:** Write Medium post: "Why our family calendar app is different"
3. **Pricing experiment:** Offer "Premium" tier at $8/month, measure conversion

---

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **Low calendar connection rate** (privacy concerns) | Medium | High | Lead with trust messaging; show example suggestions before OAuth; offer read-only access |
| **Poor suggestion quality** (data sources are bad) | High | Critical | Manually curate 15-20 activities per city; bias toward curated list over APIs |
| **Low acceptance rate** (<20%) | Medium | High | Start with concierge MVP (manual suggestions) to learn what works before automating |
| **Retention drops after week 2** | Medium | High | Add feedback loop; ask "Why didn't you like this?" to improve |
| **Google API rate limits** | Low | Medium | Cache calendar data; only scan once per week, not real-time |
| **Platform risk** (Google builds this) | Low | High | Move fast; build deep family-specific features Google won't prioritize |
| **Can't monetize** (free tier is good enough) | Medium | Critical | Make free tier very limited (1 suggestion/week); premium is table stakes |

---

## Pricing Strategy (Post-MVP)

### Free Tier
- 1 suggestion per week
- Single calendar connection
- Basic interests (3 categories)
- Email delivery only

### Premium Tier ($8-10/month)
- 3 suggestions per week
- Multi-calendar support (combine 2 calendars)
- Advanced interests + Want to Try list
- SMS delivery
- Priority booking links
- Learn from feedback (ML-driven suggestions in v2)

### Future Revenue Streams
- **B2B2C:** Corporate family wellness benefits (sell to employers at $5/employee/month)
- **Affiliate commissions:** Restaurants/venues pay for featured placement
- **API access:** Sell recommendation engine to Cozi, TimeTree, etc.

---

## Success Criteria (MVP)

**Ship or Kill Decision (End of Week 12):**

**Ship to broader beta if:**
- ✅ 40%+ suggestion acceptance rate
- ✅ 70%+ actual attendance rate
- ✅ 30%+ would pay $8-10/month
- ✅ Positive qualitative feedback ("I love this!" vs "It's fine")

**Pivot if:**
- ❌ <20% acceptance rate → Problem: suggestions are bad or trust is too low
- ❌ <50% attendance rate → Problem: suggestions are impractical
- ❌ <10% would pay → Problem: value is not strong enough

**Kill if:**
- ❌ Can't get 20 people to even try it → No demand

---

## Roadmap Beyond MVP

### V2 Features (Months 4-6)
- Auto-insert to Google Calendar (write permissions)
- Mobile app (React Native)
- Multi-calendar coordination (both partners)
- ML-based recommendations (replace rule-based)
- ChatGPT action integration

### V3 Features (Months 7-12)
- Babysitter coordination
- Apple Calendar support
- Multi-family events (coordinate with other families)
- In-app booking/reservations
- Social features (share experiences)

---

## Open Questions

1. **Product Name:** What should we call this? (Ideas: TimeWeaver, FamilyOS, QualityTime, NestPlan)
2. **Geographic scope:** Start with 1 city or top 10 metros?
3. **Curated vs API-first:** How much manual curation is sustainable?
4. **Notification fatigue:** Is weekly the right cadence, or should it be dynamic?
5. **Couple vs family focus:** Start with couples (simpler) or families (bigger pain)?

---

## Appendix: Research References

- [Market Research Report](ai/guides/quality-time-app-market-research.md)
- Competitive analysis: Cozi, TimeTree, Cobble, Google Calendar
- User pain points: Time poverty, cognitive load, desire-to-action gap
- Willingness to pay: $5-12/month is viable based on Cozi pricing

---

**Next Steps:**
1. Review and approve PRD
2. Recruit 5-10 beta families for concierge MVP
3. Begin technical implementation (Google Calendar OAuth + basic web app)
4. Design email templates for suggestion delivery
