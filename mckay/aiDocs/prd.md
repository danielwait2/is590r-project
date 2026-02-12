# Product Requirements Document (PRD)
## Calendar-Native Weekend Experience Concierge

**Version:** 1.0
**Last Updated:** February 12, 2026
**Status:** Planning / MVP Development Phase
**Owner:** Technology Solutions Sales Class Project

> **Note:** This PRD describes the full product vision. For the **class project MVP** (6-8 weeks), we're building a simplified version:
> - 1 test couple (not 50-100 couples)
> - JSON files (not PostgreSQL)
> - Manual triggering (not automated scheduling)
> - See [mvp.md](./mvp.md) and [architecture.md](./architecture.md) for actual MVP scope.

---

## Executive Summary

A calendar-integrated activity suggestion service that helps busy couples turn their "things we keep saying we should do" into actual plans. The product monitors both partners' Google Calendars for overlapping free time, matches it against their stated interests and "want to try" list, and inserts curated local activity suggestions directly into their shared calendar.

**Product Type:** Calendar-native weekend experience concierge
**Target Market:** Experience-seeking couples in [one metro area]
**Key Differentiator:** Only product that combines couple preferences + both calendars + calendar-native delivery
**MVP Timeline:** 6-8 weeks to demo-ready automated MVP (manual pilot skipped for class project)

---

## Problem Statement

### The Core Problem

Dual-income couples, especially those with young children, maintain mental lists of activities they want to do together—restaurants to try, classes to take, events to attend—but these intentions rarely convert to action.

**Why intentions fail to become actions:**
1. **Decision paralysis:** "What should we do this weekend?" leads to 30 minutes of back-and-forth
2. **Coordination friction:** Finding overlapping free time requires manual calendar checking
3. **Discovery effort:** Searching for options that match interests and timing takes effort
4. **Planning inertia:** By the time they decide, the moment has passed or they're too tired

### The Validated Pain Point

- Users report having ongoing lists of "things we keep saying we should try"
- These lists grow but rarely shrink through execution
- The barrier is not desire but execution: planning, coordinating, and committing

### Market Validation

Market research shows:
- **25% of event attendees** cite ticketing platforms as their most useful discovery channel
- **33% specifically use Eventbrite** for finding local experiences
- Growing demand for local, niche, community-driven activities (workshops, classes, interest-based events)
- No existing tool combines couple preferences, shared calendar access, and proactive suggestions

---

## Target Users

### Primary Persona: "The Intentional Couple"

**Demographics:**
- Dual-income couples (with or without children)
- Ages 28-42
- Household income: $100k+
- Urban/suburban location in target metro

**Psychographics:**
- Value experiences over things
- Already attend workshops, classes, or local events (2-4x per month)
- Maintain shared Google Calendar
- See "going out together" as part of their identity as a couple
- Frustrated by the gap between intentions and execution

**Behaviors:**
- Currently search "things to do this weekend" or browse Facebook Events
- Text each other potential date ideas but often don't follow through
- Have a mental "want to try" list (restaurants, activities, venues)
- Book babysitters in advance when they commit to plans

**Goals:**
- Spend quality time together doing new things
- Execute on their "want to try" list without planning overhead
- Maintain their relationship through shared experiences
- Feel like they're "making the most of" their limited free time

**Pain Points:**
- Decision fatigue: too many options, hard to choose
- Coordination overhead: checking both calendars, finding overlaps
- Planning procrastination: "let's figure it out later" becomes never
- Missed opportunities: events/reservations book up while they're deciding

### Initial Beachhead Market

**Geographic:** One metro area with high local event density
- Strong Eventbrite presence (workshops, classes, tastings)
- Active small business event scene
- Density of experience-oriented venues
- Candidates: Austin, Portland, Denver, Seattle, Minneapolis

**Volume:** 50-100 couples for beta (10-20 for manual validation)

**Qualification criteria:**
- Both partners willing to connect Google Calendar
- Currently attend local events/activities 1+ times per month
- Have expressed frustration with date planning process
- Comfortable with technology (OAuth flows, calendar management)

---

## Market Analysis & Competitive Landscape

### Existing Solutions

#### **Facebook Events / Meta Local**
**What they do:**
- Curate local events based on interests, friends, location
- Category filters, time-based browsing ("This weekend")
- Social proof via friends' attendance

**Gaps:**
- Requires proactive browsing (pull, not push)
- Not designed around couple dynamics (two calendars, shared preferences)
- Delivers notifications, not calendar entries
- Generic discovery, not personalized execution

#### **Google Search ("things to do near me")**
**What they do:**
- Ad-hoc search results for local activities
- Google Maps integration for nearby venues

**Gaps:**
- Manual search required weekly
- No calendar integration
- Doesn't track couple's "want to try" list
- No proactive suggestions

#### **Eventbrite, Yelp Events**
**What they do:**
- Event discovery and ticketing platforms
- Search and browse by category, date, location

**Gaps:**
- Requires browsing/searching (user must initiate)
- No calendar awareness
- No couple-specific preferences or coordination

#### **Date Night Apps (Cobble, Hive, etc.)**
**What they do:**
- Curated date ideas based on preferences
- Swipe-based selection
- Local recommendations

**Gaps:**
- Another app to check (doesn't leverage existing calendar behavior)
- No automatic free time detection
- Limited calendar integration
- Requires opening app to see suggestions

### Our Differentiation

**We are the ONLY product that combines:**
1. **Both partners' calendars** → Find mutually free time automatically
2. **Explicit couple preferences** → "Want to try" list + interests inventory
3. **Calendar-native delivery** → Suggestions appear in existing coordination tool
4. **Proactive suggestions** → Push, not pull (they don't need to remember to check)

### Positioning

**We are NOT:**
- A discovery tool competing with Facebook/Google
- A comprehensive family logistics platform
- A mass-market consumer app

**We ARE:**
- A calendar-native weekend experience concierge
- An execution tool, not a discovery tool
- A focused, high-intent service for experience-seeking couples
- A "set it and forget it" suggestion engine

**Positioning Statement:**
> "The weekend experience concierge for busy couples who know what they want to do together but struggle to make it happen. We turn your 'want to try' list into calendar events when you're both actually free."

---

## Product Vision & Goals

### Vision (12-18 months)

Become the default "date planning" tool for experience-seeking couples in [target city], known for:
- Reliably surfacing high-quality, perfectly-timed activity suggestions
- Dramatically increasing the frequency of "we actually did something" weekends
- Learning couple preferences over time to improve match quality

### Goals

#### Business Goals
1. **Validation:** 40%+ suggestion acceptance rate in manual pilot (5-10 couples, 4 weeks)
2. **Engagement:** 60%+ of accepted suggestions result in actual attendance
3. **Retention:** 70%+ of beta users still active after 8 weeks
4. **NPS:** Net Promoter Score of 40+ among active users

#### User Goals
1. Reduce "what should we do this weekend?" decision time from 30 minutes to 30 seconds
2. Execute on 2-4 items from their "want to try" list per month (vs. 0-1 currently)
3. Increase frequency of "we did something together" weekends by 50%+
4. Feel like they're "making the most of" their relationship and limited free time

#### Learning Goals (Class Project)
1. Master Google Calendar API and OAuth 2.0 flows
2. Build multi-user coordination system (shared state, permissions)
3. Design and implement matching/recommendation algorithm
4. Integrate with external APIs (events, venues, activities)
5. Practice lean validation methodology (manual MVP → automated)

---

## User Stories & Use Cases

### Core User Journey

**Setup Flow (One-time, ~20-30 minutes per partner)**

1. **As a couple**, we want to sign up together so that we can both participate from the start
   - Partner A creates account
   - Partner A invites Partner B via email/link
   - Partner B completes signup

2. **As a user**, I want to connect my Google Calendar so the app knows when I'm free
   - OAuth flow with clear permissions explanation
   - Read access to calendar events (title, time, busy/free status)
   - Write access to create tentative events
   - Confirmation screen showing what data is accessed

3. **As a couple**, we want to define our availability preferences so suggestions land at realistic times
   - Select preferred days (Friday evening, Saturday, Sunday)
   - Define time windows (morning 9am-12pm, afternoon 1-5pm, evening 6-10pm)
   - Set minimum duration (e.g., 2 hours for dinner, 3 hours for activities)
   - Set advance notice needed (e.g., 3 days minimum)

4. **As a couple**, we want to share our interests and "want to try" list so suggestions match what we care about
   - Select interest categories (dining, arts, music, sports, outdoors, learning, etc.)
   - Free-form "want to try" list (specific restaurants, activities, venues)
   - Location preferences (neighborhoods, maximum distance)
   - Budget preferences ($ to $$$$)

**Weekly Flow (Ongoing, ~30 seconds per suggestion)**

5. **As a user**, I want suggestions to appear in my calendar when I'm planning my week so I don't need to check another app
   - Monday morning: Matching engine runs
   - Finds 1-2 overlapping free time windows in next 2 weekends
   - Matches against interests + local opportunities
   - Creates tentative calendar events for both partners

6. **As a user**, I want to easily accept or decline suggestions so I can curate my calendar with minimal effort
   - Suggestion appears as tentative event in Google Calendar
   - Event title: "Suggested: [Activity Name]"
   - Event description includes: what, where, why matched, link for details/booking
   - Accept = keep event or change to "confirmed"
   - Decline = delete event

7. **As a user**, I want the app to learn from my decisions so suggestions get better over time
   - Track accept vs. decline patterns
   - Track confirmed attendance (if event remains on calendar past event time)
   - Optional post-activity feedback: "How was it?" (thumbs up/down)

**Feedback Loop**

8. **As a couple**, we want to see what we've done together so we feel accomplished
   - Dashboard showing: accepted suggestions, completed activities, "want to try" items checked off
   - Memory/history view: "You've tried 8 new restaurants this quarter"

---

## MVP Feature Specification

### ~~Phase 1: Manual Validation Pilot~~ (SKIPPED for class project)

**Note:** For this class project, we're skipping the manual validation pilot and going straight to building the automated MVP. This phase is documented for future reference if the project continues beyond the class.

<details>
<summary>Manual Pilot Approach (Optional - Not Being Built)</summary>

**Goal:** Validate that couples want this service before building automation

**Features:**
- Google Form for interests + "want to try" list + availability preferences
- Manual Google Calendar sharing (users share calendars directly with pilot operator)
- Manual suggestion creation (operator creates calendar events each Monday)
- Manual tracking sheet (accept/decline rates, feedback)
- Weekly feedback survey

**Success Criteria:**
- 5-10 couples recruited and onboarded
- 40%+ suggestion acceptance rate
- 3+ couples report "would miss this if you stopped"

**Tools:** Google Forms, Google Sheets, manual calendar access

</details>

---

### Automated MVP (Weeks 1-8) - **CURRENT APPROACH**

**Goal:** Build minimally viable automated system for beta users

#### Core Features (Must-Have)

**1. User Authentication & Onboarding**
- [ ] User signup (email + password)
- [ ] Partner invitation system (send invite link to partner)
- [ ] Google OAuth 2.0 integration
  - Read access to calendar events
  - Write access to create events
  - Clear permission explanations
- [ ] Couple account creation (link two users as a couple)

**2. Preference Collection**
- [ ] Interests inventory form
  - Category selection (dining, arts, music, classes/workshops, outdoors, sports, etc.)
  - Subcategory refinement (e.g., dining → Italian, sushi, BBQ, farm-to-table)
- [ ] "Want to try" list input (free-form text, items saved as structured data)
- [ ] Availability preferences
  - Day/time window selection
  - Minimum duration slider
  - Advance notice requirement
- [ ] Location preferences (home address, preferred neighborhoods, max distance)
- [ ] Budget preferences ($ to $$$$)

**3. Calendar Integration**
- [ ] Read both partners' calendars
- [ ] Detect "busy" blocks (existing events)
- [ ] Identify overlapping free time windows
  - Must match availability preferences
  - Must meet minimum duration
  - Must be within advance notice window
  - Must be within next 2 weekends (Friday evening - Sunday)

**4. Event/Activity Discovery**
- [ ] Integrate with Eventbrite API
  - Search by location, date, category
  - Filter for workshops, classes, tastings, small venue events
- [ ] Integrate with Google Places API
  - Search for restaurants, cafes, museums on "want to try" list
  - Get details (rating, price level, photos)
- [ ] Manual curation backup (admin can add custom suggestions)

**5. Matching Engine**
- [ ] Run nightly (Sunday/Monday for upcoming weekends)
- [ ] For each couple:
  - Query both calendars for free time
  - Find overlapping windows matching preferences
  - Query event APIs for available activities
  - Score activities based on:
    - Interest category match
    - "Want to try" list match (keyword search)
    - Location (distance from home)
    - Budget fit
    - Time window fit
  - Select top 1-2 suggestions

**6. Suggestion Delivery**
- [ ] Create tentative event in both partners' Google Calendars
- [ ] Event structure:
  - Title: "Suggested: [Activity Name]"
  - Time: Suggested time window
  - Location: Venue address
  - Description:
    - What it is
    - Why it matched ("You said you wanted to try pottery classes")
    - Price/booking info
    - Link to details/tickets
    - How to accept/decline instructions

**7. Feedback Collection**
- [ ] Track event status changes (accepted = kept, declined = deleted)
- [ ] Post-activity survey (2 days after event):
  - "Did you go?" (yes/no)
  - "How was it?" (thumbs up/down + optional comment)
- [ ] Store feedback for learning

**8. Admin Dashboard (Basic)**
- [ ] View all couples and their preferences
- [ ] See suggestion history (sent, accepted, declined)
- [ ] View feedback/survey responses
- [ ] Manually trigger suggestion generation for testing

#### Nice-to-Have (Phase 2+)

- [ ] User dashboard (view upcoming/past suggestions)
- [ ] Edit preferences without re-onboarding
- [ ] "Want to try" list management (add/remove items)
- [ ] Notification system (email summary of weekly suggestions)
- [ ] Learning algorithm (improve matching based on accept/decline patterns)
- [ ] Group coordination (invite another couple to join an activity)
- [ ] Venue partnerships (promoted suggestions)

---

## Technical Requirements

### Architecture Overview

**Tech Stack (MVP - 6-8 weeks):**
- **Backend:** Node.js + Express
- **Data Storage:** JSON files (no database for MVP)
- **Authentication:** Google OAuth 2.0 only (no JWT/passwords)
- **Hosting:** Local development (localhost:3000)
- **Triggering:** Manual (admin dashboard) - no scheduled jobs

**Tech Stack (V2/Future):**
- **Database:** PostgreSQL (migrate from JSON)
- **Hosting:** Railway, Render, or Vercel (cloud deployment)
- **Scheduled Jobs:** Node-cron or BullMQ (automated weekly suggestions)
- **Authentication:** Add user accounts with JWT
- **External APIs:**
  - Google Calendar API (read/write events)
  - Eventbrite API (event discovery)
  - Google Places API (venue details)

**Key Components:**

1. **Auth Service:** Handle signup, login, OAuth flows
2. **Calendar Service:** Read/write Google Calendar events
3. **Matching Engine:** Find free time + match activities
4. **Event Discovery Service:** Query external APIs
5. **Notification Service:** Create calendar events, send emails
6. **Admin API:** Dashboard data and manual triggers

### Data Models

**Users Table:**
```
- user_id (UUID, PK)
- email
- password_hash
- google_access_token
- google_refresh_token
- google_calendar_id
- created_at
- updated_at
```

**Couples Table:**
```
- couple_id (UUID, PK)
- partner_1_id (FK to users)
- partner_2_id (FK to users)
- home_address (geocoded)
- max_distance_miles
- created_at
- updated_at
```

**Preferences Table:**
```
- couple_id (FK to couples, PK)
- interests (JSON array of categories/subcategories)
- want_to_try (JSON array of items)
- availability (JSON: days, time windows, min duration, advance notice)
- budget_level (enum: $, $$, $$$, $$$$)
- updated_at
```

**Suggestions Table:**
```
- suggestion_id (UUID, PK)
- couple_id (FK to couples)
- event_type (enum: eventbrite_event, google_place, custom)
- event_external_id (ID from external API)
- event_name
- event_description
- event_location (address)
- suggested_time_start
- suggested_time_end
- match_reason (text: why this was suggested)
- status (enum: sent, accepted, declined, completed)
- calendar_event_id (Google Calendar event ID)
- created_at
- updated_at
```

**Feedback Table:**
```
- feedback_id (UUID, PK)
- suggestion_id (FK to suggestions)
- attended (boolean)
- rating (enum: up, down, neutral)
- comment (text, optional)
- created_at
```

### API Integrations

**Google Calendar API:**
- Scopes needed: `calendar.events`, `calendar.readonly`
- Operations: List events (read), Insert events (write), Delete events (write)
- Rate limits: 1M requests/day (plenty for MVP)
- Handle OAuth refresh tokens (store securely)

**Eventbrite API:**
- Endpoint: `/events/search/`
- Filters: location, date_range, categories
- Rate limits: 1000 requests/hour (should be sufficient)
- Focus on categories: classes, workshops, food & drink, music (small venue)

**Google Places API:**
- Endpoints: Place Search, Place Details
- Used for: restaurants, cafes, museums from "want to try" list
- Rate limits: Consider costs ($17/1000 requests for Place Details)

### Security & Privacy

**Data Protection:**
- Encrypt Google OAuth tokens at rest
- Use HTTPS for all communications
- JWT tokens for session management (short expiration)
- Rate limiting on auth endpoints

**Privacy Considerations:**
- Clear permission explanations during OAuth
- Only read calendar metadata (title, time, busy/free) — no full event details
- Users can revoke access anytime via Google account settings
- Data deletion on account closure

**Compliance:**
- Not collecting payment info (no PCI compliance needed for MVP)
- GDPR considerations: right to data export, right to deletion
- Terms of Service + Privacy Policy required

---

## Out of Scope (For MVP)

**Explicitly NOT building:**
- ❌ Mobile apps (iOS/Android) — web-only for MVP
- ❌ Real-time calendar monitoring/webhooks — batch processing only
- ❌ Babysitter coordination
- ❌ Payment/booking integration
- ❌ Social features (sharing, friend recommendations)
- ❌ Multi-city support — one metro area only
- ❌ Advanced ML/AI recommendation algorithms — simple scoring is fine
- ❌ Weekday suggestions — weekends only
- ❌ Multi-family coordination (group activities)
- ❌ Transportation/logistics planning
- ❌ Budget tracking or expense splitting

**Can be added later if validated:**
- Notification preferences (email, push)
- Rescheduling/cancellation management
- Historical activity tracking dashboard
- Integration with other calendar systems (Outlook, iCal)
- White-label for corporate benefits programs

---

## Success Metrics

### Validation Phase (Manual Pilot)

**Primary Metrics:**
- **Acceptance Rate:** % of suggestions kept on calendar (Target: 40%+)
- **Completion Rate:** % of accepted suggestions attended (Target: 60%+)
- **User Satisfaction:** "Would miss if stopped" responses (Target: 60%+)

**Secondary Metrics:**
- Time to complete setup (Target: <30 min per partner)
- % of couples completing setup (Target: 80%+)
- Feedback sentiment (qualitative)

### MVP Phase (Automated Beta)

**Activation Metrics:**
- % of signups completing OAuth (Target: 70%+)
- % of couples completing full setup (Target: 60%+)
- Time to first suggestion received (Target: <7 days)

**Engagement Metrics:**
- Weekly Active Couples (checking calendar) (Target: 70%+)
- Suggestion Acceptance Rate (Target: 35%+)
- Suggestion Completion Rate (Target: 50%+)
- Avg suggestions accepted per couple per month (Target: 2+)

**Retention Metrics:**
- Week 4 retention (Target: 60%+)
- Week 8 retention (Target: 50%+)
- Churn reasons (qualitative)

**Quality Metrics:**
- Suggestion relevance score (user feedback) (Target: 3.5+/5)
- "Want to try" list completion rate (Target: 20%+ items tried in 3 months)
- NPS (Target: 40+)

**Learning Metrics:**
- Acceptance rate improvement over time (does learning work?)
- Match quality by activity type (which categories work best?)
- Optimal suggestion frequency (1 vs. 2 per week)

---

## Go-to-Market Strategy

### Beta Recruitment (Validation Phase)

**Target: 5-10 couples for manual pilot, 50-100 for automated beta**

**Channels:**
1. **Local parenting groups** (Facebook, Nextdoor)
   - Post: "Free date planning service for busy couples (4-week pilot)"
2. **Friends & family**
   - Personal outreach to couples matching persona
3. **Local event venues**
   - Partner with 2-3 venues to recruit attendees
4. **University communities**
   - Faculty/staff newsletters (dual-income demographic)

**Offer:**
- Free service for beta period
- "Help shape a product for busy couples"
- First 10 couples get lifetime free access

### Positioning & Messaging

**Value Propositions:**
- Primary: "We turn your 'want to try' list into calendar events when you're both free"
- Secondary: "Stop deciding, start doing"
- Tertiary: "Date planning, handled"

**Messaging Framework:**
- **Problem:** "You know what you want to do together, but planning takes effort"
- **Solution:** "We put perfectly-timed suggestions directly in your calendar"
- **How:** "One-time setup. Weekly suggestions. Accept or decline in 5 seconds."
- **Benefit:** "More experiences, less planning"

### Future Monetization (Post-MVP)

**For couples:**
- Freemium: 1 suggestion/week free, 2-3/week = $8-12/month
- Or: First month free, then $10/month unlimited

**For venues:**
- Promoted suggestions: $50-100/month per venue
- Affiliate/commission: 5-10% of bookings driven

**For partners:**
- White-label for corporate benefits: $5-10k/year per company
- Integration with date night subscription boxes

---

## Risks & Mitigation Strategies

### Technical Risks

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Google OAuth rejection/revocation | High | Low | Clear permission explanations; minimal scope requests; handle revocation gracefully |
| Event API data quality (stale/incomplete) | High | Medium | Start with Eventbrite (higher quality); supplement with manual curation; focus on evergreen activities |
| Calendar "lies" (blank ≠ free) | High | High | Explicit availability preferences during setup; weekend-only scope; user feedback loop |
| Matching algorithm poor quality | Medium | Medium | Start simple (keyword matching); iterate based on feedback; manual curation backup |
| Rate limiting on APIs | Medium | Low | Efficient caching; batch processing; monitor usage |

### Product Risks

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Setup friction too high (dropout) | High | High | Make setup feel valuable ("date profile design"); progressive disclosure; allow partial completion |
| Suggestion fatigue (too many) | Medium | Medium | Limit to 1-2/week; quality over quantity; let users adjust frequency |
| Privacy concerns (calendar access) | High | Medium | Clear explanations; minimal data access; allow read-only mode initially |
| Low acceptance rates | High | Medium | Manual validation first; iterate on match quality; user feedback loop |
| Timing always wrong | High | Medium | Explicit availability prefs; learn from patterns; weekend-only scope |

### Market Risks

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Market too small (niche) | Medium | Medium | Fine for class project; validate in one city first; expand if traction |
| Facebook adds this feature | High | Low | They focus on discovery, not execution; we have integration moat |
| Users don't see value | High | Medium | Manual pilot validates value first; pivot if needed |
| Can't monetize effectively | Low | N/A | Not required for class project; multiple paths available |

### Execution Risks

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Scope creep (feature bloat) | High | High | Ruthless prioritization; "out of scope" list; focus on core loop |
| Timeline slippage | Medium | Medium | Start with manual validation; build automated incrementally |
| Can't recruit beta users | Medium | Medium | Multiple recruitment channels; incentivize participation |
| Technical complexity underestimated | Medium | Medium | Use well-documented APIs; start simple; get help when stuck |

---

## Validation Plan (Class Project Approach)

### Demo-Ready Automated MVP (Weeks 1-8)

**Note:** For this class project, we're skipping the manual pilot validation and building directly to a demo-ready automated system. This approach prioritizes demonstrating technical competence over market validation.

**Objectives:**
1. Build and deploy automated system
2. Validate tech stack choices
3. Demonstrate core functionality with 1 test couple
4. Show technical skills (OAuth, API integration, algorithms)
5. Prepare compelling demo presentation

**Milestones:**
- Week 1-2: OAuth + Google Calendar integration (hardest part)
- Week 3-4: Free time detection + Preferences collection
- Week 5-6: Activity database + Matching engine
- Week 6-7: Suggestion creation + Calendar event insertion
- Week 7: Admin dashboard + Integration testing
- Week 8: Demo preparation + Final testing

**Success Criteria = Class Project Completion:**
- ✅ Working OAuth flows (both partners authenticated)
- ✅ Calendar read/write functional (events created in both calendars)
- ✅ Matching algorithm implemented (keyword-based scoring)
- ✅ 1 test couple demo works end-to-end
- ✅ Demo-ready presentation (5-10 minute walkthrough)
- ✅ Technical competence demonstrated (OAuth, APIs, algorithms)

---

## Open Questions

**To be resolved before/during build:**

1. **Which city to target for beta?**
   - Depends on team location and event data quality
   - Recommend: Austin, Portland, Denver (strong Eventbrite presence)

2. **Suggestion frequency: 1 or 2 per week?**
   - Start with 1, let users request more
   - Test 1 vs. 2 in pilot phase

3. **How to handle "want to try" list matches?**
   - Keyword search? (simple)
   - Semantic similarity? (complex)
   - Start simple, iterate

4. **What if both partners don't complete setup?**
   - Allow partial setup, send reminder emails
   - Or require both before activation (stricter)

5. **Post-activity feedback: required or optional?**
   - Optional for MVP (reduce friction)
   - Incentivize with "see your history" dashboard

6. **How to handle sold-out events or broken links?**
   - Check availability before suggesting (if API supports)
   - Accept that some suggestions will fail; learn from feedback

7. **Should we allow single users (not couples)?**
   - Out of scope for MVP (adds complexity)
   - Consider in Phase 2 if demand exists

---

## Appendix

### References

- Market research file: `marketResearch_perplexity.md`
- User persona research: [To be added]
- Competitive analysis: Sections 2-3 of this document

### Glossary

- **Couple:** Two users linked in the system with shared preferences
- **Suggestion:** A calendar event created by the system as a recommendation
- **Acceptance:** User keeps the suggestion on their calendar
- **Completion:** User attended the suggested activity (confirmed via feedback)
- **Want to try list:** User-submitted list of specific activities/venues they're interested in
- **Free time window:** Calendar time block where both partners show no conflicts
- **Match score:** Internal ranking of how well an activity fits couple's preferences

---

**Document Status:** Ready for review and technical planning
**Next Steps:**
1. Review and approve PRD
2. Set up development environment
3. Begin Phase 1 manual validation pilot
4. Create technical architecture document
