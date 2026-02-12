# Product Requirements Document (PRD)
## Family Quality Time Activity Facilitator

**Version:** 1.0  
**Last Updated:** February 12, 2026  
**Author:** Dylan Mattern  
**Status:** Draft

---

## Executive Summary

A calendar-integrated application that helps busy couples and families bridge the intention-action gap for quality time activities. The app automatically detects free time, matches it with user-stated desires and local activities, and handles the biggest barrier to execution: childcare coordination. By automating both activity discovery AND babysitter booking, we make "we should do something together" actually happen.

### Vision Statement
*Enable families to spend more intentional quality time together by removing the friction between desire and action.*

### Mission
Transform passive calendar management into active relationship building by connecting what families want to do with when they can do it and who can help make it happen.

---

## Problem Statement

### The Pain Points
1. **Intention-Action Gap**: 86% of couples wish they spent more quality time together, yet 52% rarely or never have date nights
2. **Planning Paralysis**: Free time appears but couples waste it on decision fatigue ("what should we do?")
3. **Childcare Coordination Friction**: Even when couples know what they want to do, coordinating childcare requires multiple texts/calls and often results in giving up
4. **Lost Opportunities**: Activities they've been "meaning to try" never happen because there's no system to act on intentions

### Current Alternatives Fall Short
- **Calendar apps** (Google Calendar, Cozi, Cupla): Organize schedules but don't suggest activities
- **Discovery apps** (Eventbrite, Yelp): Provide ideas but require manual searching and don't integrate with calendars
- **Babysitting services** (Care.com, UrbanSitter): Solve childcare but require separate booking workflow
- **No one connects all three**: Free time + activity matching + childcare coordination

---

## Goals & Success Metrics

### Business Goals
1. Help 100,000 families have more intentional quality time by Year 2
2. Achieve 40%+ suggestion acceptance rate (users actually do the activities)
3. Reach 60%+ Month-3 retention (users stay engaged beyond initial enthusiasm)
4. Generate revenue through activity affiliate commissions (primary) and premium features (secondary)

### User Success Metrics
| Metric | Target | Measurement |
|--------|--------|-------------|
| Suggestion Acceptance Rate | 35-45% | % of calendar suggestions that remain on calendar (not deleted) |
| Activity Completion Rate | 60-70% | % of accepted suggestions marked as "completed" |
| Babysitter Booking Success | 70%+ | % of babysitter requests that get accepted |
| User Retention (Month 3) | 40%+ | % of users still active after 3 months |
| Quality Time Increase | 2-4 activities/month | Average activities completed per family |
| NPS Score | 50+ | Net Promoter Score |

### Kill Signals (Pivot or Stop Indicators)
- Suggestion acceptance rate <25% after 50+ users
- Month-3 retention <30%
- Babysitter request success rate <40%
- Users say suggestions feel generic/irrelevant
- Privacy concerns prevent calendar connection

---

## Target Users

### Primary Persona: "The Intentional Parents"
**Demographics:**
- Married/partnered couples with children ages 2-10
- Household income: $75K-150K
- Location: Urban/suburban areas (initially: top 10 metro areas)
- Both partners work (full-time or part-time)
- Use shared digital calendars (Google Calendar primarily)

**Characteristics:**
- Value quality time but struggle to prioritize it
- Have mental lists of "things we should do" that never happen
- Experience decision fatigue on free weekends
- Have 1-3 trusted babysitters (family or regular sitters)
- Willing to spend $50-200 on activities if childcare is handled
- Tech-savvy enough to use calendar integrations

**Pain Points:**
- "We keep saying we should try that restaurant/activity but never follow through"
- "By the time we figure out a babysitter, the opportunity has passed"
- "Sunday evening arrives and we realize we wasted our free Saturday"
- "We scroll through options but can't decide what to do"

**Goals:**
- Spend more intentional time together as a couple
- Create memorable experiences with kids
- Try new activities they've been curious about
- Reduce screen time with meaningful in-person experiences

### Secondary Persona: "The Trusted Sitter"
**Demographics:**
- Grandparents (ages 55-75)
- Regular babysitters (ages 16-35)
- Close family friends

**Characteristics:**
- Want to help but need clear, organized requests
- Have their own schedules/availability
- Prefer simple, straightforward communication
- May not be tech-savvy (grandparents especially)

**Pain Points:**
- Get last-minute or unclear babysitting requests
- Want to help but scheduling is chaotic
- Don't always know when families need them

**Goals:**
- Support families they care about
- Clear communication about expectations
- Advance notice when possible
- Know exactly what's being asked (dates, times, pay)

---

## Core Features

### 1. Calendar Integration & Free Time Detection

**Description:** Connect to Google Calendar to automatically identify free time blocks suitable for activities.

**User Story:** *"As a busy parent, I want the app to automatically find free time in our schedule so I don't have to manually look for opportunities."*

**Requirements:**
- OAuth 2.0 integration with Google Calendar (read + write access)
- Support for multiple calendars (both partners)
- Detect free time blocks of 2+ hours
- Configurable parameters:
  - Minimum block size (default: 2 hours)
  - Preferred days (weekends, evenings, etc.)
  - Time of day preferences (morning, afternoon, evening)
  - Lead time (suggest activities X days in advance)
- Privacy: Only read calendar metadata (free/busy status), not event details
- Sync frequency: Daily check for upcoming free time in next 2 weeks

**Technical Specifications:**
- Use Google Calendar API v3
- Implement incremental sync to minimize API calls
- Cache calendar data with appropriate TTL
- Handle rate limits (1M queries/day per project)
- Support shared calendars and calendar delegation

**Success Criteria:**
- 95%+ accuracy in detecting actual free time
- <5% false positives (suggesting during non-free time)
- Sync completes in <10 seconds per user
- Privacy audit shows no event detail storage

---

### 2. Interest & Preference Capture

**Description:** During onboarding, collect detailed information about family interests, activities they want to try, and experience preferences.

**User Story:** *"As a user, I want to tell the app what kinds of activities we enjoy and want to try so suggestions are personalized to our family."*

**Requirements:**

**2a. Family Profile**
- Location (city, zip code)
- Children ages and count
- Budget comfort level (per activity):
  - Free ($0)
  - Low ($1-30)
  - Medium ($31-75)
  - High ($76-150)
  - Premium ($150+)
- Household constraints:
  - Dietary restrictions
  - Accessibility needs
  - Transportation availability

**2b. Interest Categories** (Visual card-based selection)
Categories to rate (Love / Like / Neutral / Dislike):
- Outdoor activities (hiking, parks, beaches)
- Food & dining (restaurants, cafes, food tours)
- Arts & culture (museums, galleries, theater)
- Active/sports (biking, swimming, sports)
- Entertainment (movies, concerts, comedy)
- Learning experiences (classes, workshops)
- Relaxation (spas, quiet cafes, nature walks)
- Social (meetups, group activities)
- Kids activities (playgrounds, kid museums, family events)
- At-home experiences (movie night, cooking together)

**2c. "Want to Try" List**
- Free-form input: "Things we keep saying we should do"
- Examples shown: "Try that new Italian place," "Go kayaking," "Visit the botanical garden"
- Can add specific items or general categories
- Each item has priority (Must Try / Interested / Someday)
- Add timestamps to track how long items have been on the list

**2d. Experience Preferences**
- Adventure vs. Relaxed (slider)
- Social vs. Intimate (slider)  
- Structured vs. Spontaneous (slider)
- Indoor vs. Outdoor preference (by season)
- Preferred day/time patterns
- Babysitter availability patterns

**Technical Specifications:**
- Onboarding flow: 5-7 minutes to complete
- Mobile-first design (swipeable cards)
- Progress indicator to reduce abandonment
- Auto-save (draft mode if user exits)
- Editable at any time (preferences evolve)
- Both partners can input separately, app shows overlap/differences

**Success Criteria:**
- 85%+ complete onboarding
- Average completion time <7 minutes
- Users add at least 5 items to "Want to Try" list
- Both partners complete preferences in 70%+ of couples

---

### 3. Activity Discovery & Matching Engine

**Description:** Intelligent matching algorithm that connects free time + user preferences + local opportunities to generate personalized activity suggestions.

**User Story:** *"As a user, I want to receive specific activity suggestions that match what I said I wanted to try, fit my schedule, and work for my family."*

**Requirements:**

**3a. Activity Database**
Structure for each activity:
- Name and description
- Category (maps to interest categories)
- Location (address, neighborhood)
- Duration (typical time required)
- Cost range
- Age appropriateness
- Time of day suitability (morning/afternoon/evening)
- Seasonality (year-round, summer only, etc.)
- Indoor/outdoor
- Booking required (yes/no)
- Source (curated/API)
- Affiliate link (if applicable)
- Ratings/reviews

**3b. Data Sources** (MVP: Manual Curation, V2: API Integration)

*MVP Phase:*
- Manually curated database: 100-200 activities per city
- Focus on quality over quantity
- Research from local publications, Google Maps, Yelp
- Top 10 metro areas only (NYC, LA, SF, Chicago, Austin, Seattle, Boston, Portland, Denver, San Diego)

*Future Phase:*
- Google Places API (venues and restaurants)
- Yelp Fusion API (ratings and reviews)
- Eventbrite API (scheduled events)
- OpenWeather API (outdoor activity timing)

**3c. Matching Algorithm**

Priority scoring based on:

1. **Desire Match (40% weight)**
   - Direct match to "Want to Try" list: +50 points
   - Category interest level: +20 (love), +10 (like), 0 (neutral), -20 (dislike)
   - Item age on list: +5 points per month (prioritize old desires)

2. **Preference Alignment (25% weight)**
   - Adventure level matches preference: +15 points
   - Indoor/outdoor matches preference: +10 points
   - Social vs. intimate matches: +10 points

3. **Practical Fit (20% weight)**
   - Within budget range: +20 points, exceeds budget: -30 points
   - Age-appropriate for kids: +15 points
   - Season-appropriate: +10 points
   - Weather-appropriate (outdoor activities): +10 points

4. **Novelty & Variety (15% weight)**
   - Haven't done this category recently: +15 points
   - Haven't visited this location: +10 points
   - Balance variety across categories

**Suggestion Logic:**
- Generate suggestions 5-7 days before free time block
- Max 1 suggestion per free time block (avoid decision paralysis)
- Suggest 2-4 activities per month maximum (avoid spam)
- Include context: "You added this to your list 3 months ago" or "You said you love outdoor activities"

**Technical Specifications:**
- Scoring algorithm runs nightly on all users with upcoming free time
- Results cached and updated as calendars change
- A/B test different scoring weights
- Machine learning phase (post-MVP): Learn from acceptance patterns

**Success Criteria:**
- Suggestions feel personal (measured via user feedback)
- 35%+ acceptance rate
- Variety across categories (no repetitive suggestions)
- Activities match stated preferences in 90%+ of cases

---

### 4. Automated Babysitter Coordination (CORE DIFFERENTIATOR)

**Description:** Intelligent babysitter request system that automatically contacts trusted sitters when activities require childcare, handles responses, and confirms arrangements.

**User Story:** *"As a parent, I want the app to automatically ask my trusted babysitters if they're available so I don't have to coordinate manually."*

#### 4a. Trusted Sitter Management

**Requirements:**
- Users can add unlimited trusted sitters
- Sitter profile contains:
  - Name
  - Relationship (Grandma, Regular Sitter, Friend, etc.)
  - Contact method preference (SMS, Email, or both)
  - Phone number
  - Email address
  - Typical hourly rate (user-defined)
  - Availability notes (optional: "Usually available weekends")
  - Priority level (Ask First, Backup, Emergency Only)
  - Response history (acceptance rate, average response time)

**Interface:**
- "Manage Trusted Sitters" page in settings
- Simple form to add/edit sitters
- Visual roster with quick stats ("Sarah: 8 sits this year, 100% acceptance rate")
- Can mark sitters as inactive/archived

#### 4b. Automatic Babysitter Request Workflow

**Trigger:** User accepts an activity suggestion that requires childcare

**Request Logic:**
1. **Determine if childcare needed:**
   - Activity category tagged as "adults only" or "date night"
   - User manually indicates "need babysitter" when accepting
   - Activity timing overlaps with children's typical care hours

2. **Generate request details:**
   - Date and day of week
   - Start time (30 minutes before activity)
   - End time (30 minutes after activity ends)
   - Total duration (e.g., "4 hours")
   - Location (user's home address)
   - Number of children and ages
   - Estimated pay (hourly rate × duration)
   - Activity context: "Dylan and Sarah are going kayaking"
   - Special instructions (optional: bedtime, dietary needs, etc.)

3. **Contact sitters in priority order:**
   - **Round 1:** Send to all "Ask First" priority sitters simultaneously
   - Wait 2 hours for responses
   - **Round 2:** If no acceptance, send to "Backup" sitters
   - Wait 2 hours for responses
   - **Round 3:** If still no acceptance, notify user to contact sitters manually

**Request Message Template (SMS/Email):**

```
Hi [Sitter Name]!

Dylan & Sarah need a babysitter:

📅 Date: Saturday, Feb 22
⏰ Time: 1:30pm - 5:30pm (4 hours)
👶 Kids: 2 children (ages 4 and 7)
💰 Pay: $105 (4 hours × $26/hour)
📍 Location: [Home Address]
ℹ️ Reason: Kayaking at Blue Lake

Can you do it?
→ Reply YES to accept
→ Reply NO if unavailable

Reply by Thursday 6pm.

View details: [Link to app]
```

**Sitter Response Handling:**

**If sitter replies YES:**
- Immediately notify user: "Great news! Sarah accepted your babysitter request for Saturday"
- Stop sending requests to other sitters
- Send confirmation to sitter with all details
- Add to both user's and sitter's calendars (if sitter has calendar integration)
- Send reminder 24 hours before
- Mark request as "Confirmed"

**If sitter replies NO:**
- Log response
- Continue to next round of sitters
- Thank sitter: "No problem! Thanks for letting us know."

**If no response after 2 hours:**
- Continue to next round
- Optional: Send single reminder after 4 hours total

**If no sitter accepts:**
- Notify user: "No sitter has accepted yet. Would you like to:"
  - Contact sitters manually
  - Try professional babysitting service (Care.com link)
  - Reschedule activity
  - Cancel activity

#### 4c. Sitter Communication System

**Technical Implementation:**
- **SMS:** Twilio API for sending/receiving texts
- **Email:** SendGrid or similar for email notifications
- **Response parsing:** Handle various response formats
  - "yes", "YES", "Yes!", "I can do it", "sure" → Accept
  - "no", "NO", "sorry can't", "not available" → Decline
  - Ambiguous responses → Ask for clarification

**Two-way communication:**
- Sitters can ask questions via SMS/email
- Messages route to user in app (with push notification)
- User can respond, creating SMS/email thread
- Keep communication simple and text-based (no app required for sitters)

#### 4d. Sitter Dashboard (Optional Feature for Engaged Sitters)

For sitters who want more engagement:
- Simple web dashboard (no app download)
- View upcoming requests and past history
- Set availability calendar
- View payment history
- One-click accept/decline

**Positioning:** "Want an easier way to manage babysitting requests? Try our sitter dashboard" (sent after 3rd request)

#### 4e. Payment Handling (Post-MVP)

**MVP Approach:**
- Display estimated pay in request
- User pays sitter directly (cash, Venmo, etc.)
- App tracks estimated amounts for tax purposes
- "Mark as Paid" button for tracking

**Future Phase:**
- Integrated payment processing (Stripe Connect)
- Automatic payment after sitting confirmed complete
- Sitter receives funds directly
- Platform fee: 3-5% of transaction

**Technical Specifications:**
- SMS gateway: Twilio (pay-per-message pricing)
- Email service: SendGrid (free tier: 100 emails/day)
- Response parsing: Simple keyword matching initially, NLP later
- Database tables:
  - `trusted_sitters` (sitter profiles)
  - `babysitter_requests` (all requests sent)
  - `babysitter_responses` (all responses received)
  - `sitting_sessions` (confirmed arrangements)
- Rate limiting: Max 10 sitters contacted per request
- Privacy: Sitters only see necessary information (not full family profile)

**Success Criteria:**
- 70%+ of requests get accepted by a sitter
- Average response time <3 hours
- 90%+ of sitters rate communication as "clear and helpful"
- <5% coordination errors (wrong date/time communicated)
- Users save average 15 minutes per activity (vs. manual coordination)

---

### 5. Calendar Event Creation & Management

**Description:** Insert activity suggestions directly into user's Google Calendar with all relevant details and context.

**User Story:** *"As a user, I want to see activity suggestions appear on my calendar so I can easily plan around them and remember the context."*

**Requirements:**

**5a. Suggestion Event Format**
- **Title:** "[Suggestion] Try: Kayaking at Blue Lake"
- **Date/Time:** During detected free time block
- **Duration:** Activity typical duration + buffer time
- **Location:** Activity address (clickable for directions)
- **Description includes:**
  - Why suggested: "You added kayaking to your Want to Try list 3 months ago"
  - Activity details and what to expect
  - Cost estimate: "$45 per person for 2-hour rental"
  - Booking link (if applicable)
  - Affiliate disclosure (transparent)
  - Babysitter status: "Babysitter request sent to Sarah and Mom"
- **Event color:** Distinct color (e.g., light blue) to differentiate from regular events
- **Attendees:** Both partners invited
- **Reminders:** 1 day before and 2 hours before

**5b. Event Interaction Flows**

**User accepts suggestion (keeps event on calendar):**
- System detects event not deleted after 24 hours → mark as "accepted"
- Trigger babysitter request workflow (if needed)
- Send confirmation: "Great! We sent babysitter requests. You'll hear back soon."
- Update event description with babysitter status
- Track as accepted for analytics

**User modifies event (changes time/date):**
- Detect modification via calendar sync
- If babysitter was confirmed, notify sitter of change
- Ask user: "You changed the activity time. Want us to update the babysitter request?"
- Re-send requests with new time if user confirms

**User deletes event (declines suggestion):**
- System detects deletion → mark as "declined"
- Optional: Quick feedback form: "Why wasn't this a good fit?" (1-click options)
  - Not interested in this activity
  - Bad timing
  - Too expensive
  - Other plans came up
  - Generic/not personalized
- Cancel any pending babysitter requests
- Use feedback to improve future suggestions

**User marks activity as complete:**
- After activity date passes, event description updates:
  - "How was it? Rate your experience" (link to feedback)
- Track completion for analytics
- Use feedback to improve matching algorithm

**5c. Calendar Sync**
- Bidirectional sync: Changes in calendar reflect in app
- Sync frequency: Every 30 minutes + immediate webhook-based updates
- Conflict detection: Don't suggest activities during newly-added events
- Handle edge cases:
  - User manually creates similar event → don't double-suggest
  - Multiple calendars → sync across all connected calendars

**Technical Specifications:**
- Google Calendar API Events.insert() for creating events
- Use watch() and notifications for real-time updates
- Store event IDs to track acceptance/deletion
- Handle API errors gracefully (rate limits, network issues)
- Event templates with variable substitution
- Webhook endpoint for calendar change notifications

**Success Criteria:**
- 100% of suggestions successfully create calendar events
- <2% event sync errors
- Users understand event is a suggestion (not commitment) via clear formatting
- Calendar doesn't feel "spammed" (max 2-4 events per month)

---

### 6. Feedback & Learning Loop

**Description:** Collect feedback on suggestions and completed activities to improve future recommendations.

**User Story:** *"As a user, I want the app to learn what I like so suggestions get better over time."*

**Requirements:**

**6a. Implicit Feedback (Automatic)**
- Track acceptance vs. declination patterns
- Track which categories get accepted most
- Note timing patterns (prefer weekends vs. weeknights)
- Observe budget patterns (accept $50 activities but decline $100+)
- Track completion rate (accepted but didn't do it = lower quality suggestion)

**6b. Explicit Feedback (User-Initiated)**

**After declining suggestion:**
- Quick 1-click feedback: "Why wasn't this a good fit?"
  - Not interested in this activity type
  - Bad timing
  - Too expensive
  - Already did something similar recently
  - Not a good fit for our family

**After completing activity:**
- Rate experience 1-5 stars
- Quick reflection: "What did you love?" (optional text)
- "Would you do this again?" Yes/No
- "Add similar activities to my list?" Yes/No

**Monthly check-in:**
- "You've tried 3 new activities this month! 🎉"
- Review your Want to Try list: "Still interested in wine tasting?" Keep/Remove
- Add new items you've been thinking about

**6c. Algorithm Adjustments**
- Increase scoring for categories with high acceptance
- Decrease scoring for declined categories
- Adjust budget range based on actual spending patterns
- Learn temporal preferences (time of day, day of week)
- Identify "hidden preferences" (user says they like X but never accepts X suggestions)

**Technical Specifications:**
- Store all feedback in analytics database
- Run weekly batch job to update user preference weights
- A/B test algorithm changes on subset of users
- Dashboard for monitoring suggestion quality metrics
- Machine learning model (post-MVP) for collaborative filtering

**Success Criteria:**
- Acceptance rate improves over time (month 3 > month 1)
- 50%+ of users provide feedback on at least one suggestion
- Users feel suggestions are "getting better" (measured in surveys)
- Reduce generic/irrelevant suggestions to <10%

---

### 7. User Dashboard & Management

**Description:** Central hub for managing all aspects of the app: preferences, trusted sitters, activity history, and upcoming suggestions.

**User Story:** *"As a user, I want one place to see my upcoming activities, manage my preferences, and track what we've done together."*

**Dashboard Sections:**

**7a. Overview/Home**
- Upcoming activity suggestions (next 2 weeks)
- Babysitter request status
- Quick stats: "3 activities this month, 12 this year"
- Recent activity history with photos/notes

**7b. Want to Try List**
- All items with status (Pending, Scheduled, Done)
- Sort by priority, date added, category
- Quick add/edit/remove
- Visual indicator for items on list >3 months (priority boost)

**7c. Preferences**
- Edit family profile
- Update interest ratings
- Adjust experience preferences
- Calendar connection status
- Notification settings

**7d. Trusted Sitters**
- Roster with stats (acceptance rate, sits this year)
- Add/edit/remove sitters
- View pending and past requests
- Payment tracking (who you owe, paid status)

**7e. Activity History**
- Calendar view of completed activities
- Filter by category, date, rating
- Notes and memories from each activity
- Export annual summary ("Your year in quality time")

**7f. Settings**
- Account management
- Privacy controls
- Notification preferences
- Subscription/billing (if premium)
- Data export and deletion

**Technical Specifications:**
- Responsive web app (mobile-first)
- React or Next.js frontend
- Real-time updates (WebSocket for babysitter responses)
- Fast load times (<2 seconds)
- Accessible (WCAG 2.1 AA compliant)

**Success Criteria:**
- 80%+ of users visit dashboard at least weekly
- Average session duration >3 minutes
- Users successfully manage sitters without support tickets
- Mobile usability score >85/100

---

## User Flows

### Critical User Flow 1: Initial Onboarding

**Goal:** Get user from signup to first suggestion in <10 minutes

1. **Landing & Signup**
   - Value proposition: "Turn free time into quality time"
   - Sign up with Google (OAuth)
   - Immediate calendar permission request with privacy messaging

2. **Family Profile** (1-2 minutes)
   - Location (zip code)
   - Kids: ages and count
   - Budget comfort level (slider)

3. **Interest Capture** (3-4 minutes)
   - Swipeable cards with activity categories
   - Rate each: Love/Like/Neutral/Pass
   - Visual, engaging, fast

4. **Want to Try List** (2-3 minutes)
   - "What have you been meaning to do together?"
   - Show examples for inspiration
   - Add 3-5 items minimum
   - Can add more later

5. **Trusted Sitters** (2-3 minutes)
   - "Who can watch your kids when you need them?"
   - Add 1-3 trusted sitters with contact info
   - Optional: Add more later

6. **Calendar Sync** (30 seconds)
   - "Checking your calendar for free time..."
   - Progress indicator
   - Success confirmation

7. **Welcome & First Suggestion** (if free time exists)
   - "We found 3 openings in the next 2 weeks"
   - Show first suggestion immediately
   - "Check your calendar - we added it there too!"
   - Explain how to accept/decline

**Success Criteria:**
- 85%+ complete onboarding
- Average time <10 minutes
- At least 3 Want to Try items added
- At least 1 trusted sitter added
- Both partners complete within 48 hours (70% of couples)

---

### Critical User Flow 2: Receiving and Accepting a Suggestion

**Trigger:** User has free time detected in calendar

1. **Suggestion Created**
   - Algorithm generates personalized suggestion
   - Event inserted into Google Calendar
   - User receives notification (push + email)

2. **User Views Suggestion**
   - Push notification: "You have free time Saturday - we found something perfect for you!"
   - Clicks notification → opens app or calendar
   - Sees event in calendar with full details
   - Reads why it was suggested

3. **User Decides**
   - **Option A: Accepts (keeps event on calendar)**
     - Does nothing (passive accept) OR
     - Clicks "I'm interested" in app
     - Proceeds to step 4
   
   - **Option B: Declines (deletes event)**
     - Deletes event from calendar
     - Optional feedback prompt appears
     - Quick 1-click reason for decline
     - System learns from feedback

4. **Babysitter Coordination Triggered** (if accepting)
   - App detects acceptance
   - Asks: "Need a babysitter for this?"
   - User confirms yes/no
   - If yes → automatic request sent to trusted sitters

5. **User Monitors Babysitter Status**
   - Push notification: "Babysitter request sent to Sarah and Mom"
   - Can view status in app: "Waiting for responses..."
   - Receives update when sitter responds

6. **Confirmation**
   - Push notification: "Good news! Sarah can babysit Saturday 1:30-5:30pm"
   - Calendar event updates with confirmed sitter info
   - Details finalized

7. **Pre-Activity Reminders**
   - 1 day before: "Tomorrow: Kayaking at 2pm! Sarah confirmed for babysitting."
   - 2 hours before: "Kayaking in 2 hours. Sarah arrives at 1:30pm."

8. **Post-Activity Feedback**
   - Day after: "How was kayaking? Rate your experience"
   - Quick 5-star rating + optional text
   - "Mark as complete" → moves to activity history
   - Feedback improves future suggestions

**Success Criteria:**
- 35%+ accept suggestions (don't delete event)
- 70%+ who accept successfully book babysitter
- 60%+ of accepted activities are completed
- <10% coordination errors (wrong time/date communicated)

---

### Critical User Flow 3: Babysitter Request & Response

**Actors:** User (parents), Trusted Sitter, System

**Phase 1: Request Sent (From User Perspective)**

1. User accepts activity suggestion in calendar
2. System prompts: "Need a babysitter?"
3. User confirms: "Yes, send requests"
4. System shows: "Sending request to Sarah and Grandma (your 'Ask First' sitters)"
5. User sees status: "Waiting for responses... We'll notify you when someone accepts"

**Phase 2: Sitter Receives Request (From Sitter Perspective)**

1. Sitter receives SMS/Email:
   ```
   Hi Sarah!
   
   Dylan & Emily need a babysitter:
   
   📅 Saturday, Feb 22
   ⏰ 1:30pm - 5:30pm (4 hours)  
   👶 2 kids (ages 4 and 7)
   💰 $105 ($26/hour)
   📍 123 Main St
   ℹ️ Kayaking at Blue Lake
   
   Can you do it?
   Reply YES or NO by Thursday 6pm
   ```

2. Sitter replies via text: "YES"

**Phase 3: Confirmation (System Handles)**

1. System receives "YES" from Sarah
2. System immediately:
   - Stops sending requests to other sitters
   - Sends confirmation SMS to Sarah: "Perfect! Dylan will see you Saturday at 1:30pm. Details: [link]"
   - Sends notification to user: "Sarah accepted! She'll be there Saturday at 1:30pm."
   - Updates calendar event with sitter confirmation
   - Logs in database

**Phase 4: Reminders (Automatic)**

1. **24 hours before:**
   - To user: "Reminder: Kayaking tomorrow at 2pm. Sarah babysitting 1:30-5:30pm."
   - To sitter: "Reminder: Babysitting for Dylan tomorrow, Sat 1:30-5:30pm at 123 Main St"

2. **2 hours before:**
   - To sitter only: "Babysitting starts in 2 hours (1:30pm today)"

**Phase 5: Post-Sitting (Optional)**

1. After end time, user gets prompt: "Mark babysitting as complete?"
2. User confirms and optionally marks as paid
3. Thank you message to sitter: "Thanks for babysitting! Dylan marked it complete."

**Alternative Flows:**

**If NO sitter accepts:**
- After 4 hours with no acceptance, notify user
- Offer options:
  - Contact sitters manually
  - Try backup sitters
  - Professional service (Care.com link)
  - Reschedule activity

**If sitter needs to cancel:**
- Sitter texts: "Sorry, can't do Saturday anymore"
- System detects cancellation keywords
- Notifies user immediately
- Asks: "Should we contact other sitters?"
- Re-initiates request flow if user confirms

**Success Criteria:**
- 70%+ of requests get accepted
- Average response time <3 hours
- <5% cancellations after confirmation
- Zero instances of miscommunication about time/date
- Sitters rate system "easy to use" (4+ stars)

---

## Technical Architecture

### System Components

**1. Frontend (Web App)**
- **Technology:** React + Next.js
- **Hosting:** Vercel or Netlify
- **Features:**
  - Responsive design (mobile-first)
  - Progressive Web App (PWA) for mobile experience
  - Real-time updates via WebSocket
  - Google OAuth integration

**2. Backend (API Server)**
- **Technology:** Node.js + Express OR Python + FastAPI
- **Hosting:** Railway, Render, or AWS ECS
- **Features:**
  - RESTful API for frontend
  - Background job processing
  - Calendar sync service
  - Matching algorithm
  - Babysitter coordination logic

**3. Database**
- **Primary:** PostgreSQL (relational data)
- **Caching:** Redis (session management, API response caching)
- **Schema:** (see Database Schema section below)

**4. Background Jobs**
- **Technology:** Bull (Node.js) or Celery (Python)
- **Jobs:**
  - Daily calendar sync
  - Nightly suggestion generation
  - Babysitter request timeouts
  - Reminder notifications
  - Analytics aggregation

**5. External Integrations**
- **Google Calendar API:** Calendar read/write
- **Twilio:** SMS sending/receiving
- **SendGrid:** Email notifications
- **Stripe:** Payment processing (post-MVP)
- **Google Places API:** Activity data (post-MVP)
- **Weather API:** Outdoor activity timing (optional)

**6. Monitoring & Analytics**
- **Application Monitoring:** Sentry (error tracking)
- **Analytics:** Mixpanel or Amplitude (user behavior)
- **Logging:** Datadog or CloudWatch
- **Uptime Monitoring:** Pingdom or UptimeRobot

### Database Schema (Key Tables)

```sql
-- Users & Authentication
users
  - id (PK)
  - email
  - google_id
  - google_access_token (encrypted)
  - google_refresh_token (encrypted)
  - created_at, updated_at

-- Family/Household Management
families
  - id (PK)
  - name
  - location_zip
  - location_city
  - location_state
  - budget_range (enum: free, low, medium, high, premium)
  - kids_count
  - kids_ages (array)
  - created_at

family_members
  - id (PK)
  - family_id (FK)
  - user_id (FK)
  - role (enum: partner_1, partner_2, single_parent)

-- Interests & Preferences
family_interests
  - id (PK)
  - family_id (FK)
  - category (enum: outdoor, dining, arts, sports, etc.)
  - rating (enum: love, like, neutral, dislike)
  - updated_at

want_to_try_items
  - id (PK)
  - family_id (FK)
  - description (text)
  - category (optional)
  - priority (enum: must_try, interested, someday)
  - status (enum: pending, scheduled, completed)
  - added_date
  - completed_date

experience_preferences
  - id (PK)
  - family_id (FK)
  - adventure_level (1-10 scale)
  - social_level (1-10 scale)
  - structured_level (1-10 scale)
  - indoor_outdoor_preference (seasonal JSON)

-- Calendar Sync
calendar_connections
  - id (PK)
  - user_id (FK)
  - calendar_id (Google Calendar ID)
  - calendar_name
  - sync_token (for incremental sync)
  - last_synced_at
  - is_active

free_time_blocks
  - id (PK)
  - family_id (FK)
  - start_datetime
  - end_datetime
  - duration_hours
  - detected_at
  - suggestion_created (boolean)

-- Activity Database
activities
  - id (PK)
  - name
  - description
  - category
  - subcategory
  - location_address
  - location_city
  - location_lat, location_lng
  - duration_hours (typical)
  - cost_range (enum)
  - cost_min, cost_max
  - age_appropriate_min, age_appropriate_max
  - indoor_outdoor (enum)
  - season_availability (array: spring, summer, fall, winter, year_round)
  - time_of_day (array: morning, afternoon, evening, night)
  - booking_required (boolean)
  - booking_link
  - affiliate_link
  - data_source (enum: curated, google_places, eventbrite)
  - source_id (external ID)
  - rating_avg
  - rating_count
  - created_at, updated_at

-- Suggestions
suggestions
  - id (PK)
  - family_id (FK)
  - activity_id (FK)
  - free_time_block_id (FK)
  - suggested_datetime
  - calendar_event_id (Google Calendar event ID)
  - status (enum: pending, accepted, declined, completed)
  - match_score (algorithm score)
  - match_reasons (JSON: why this was suggested)
  - created_at
  - accepted_at
  - declined_at
  - completed_at

suggestion_feedback
  - id (PK)
  - suggestion_id (FK)
  - feedback_type (enum: decline_reason, completion_rating)
  - rating (1-5 stars, nullable)
  - decline_reason (enum)
  - notes (text, optional)
  - created_at

-- Babysitter System
trusted_sitters
  - id (PK)
  - family_id (FK)
  - name
  - relationship (text: "Grandma", "Regular sitter", etc.)
  - phone_number
  - email
  - contact_preference (enum: sms, email, both)
  - hourly_rate (decimal)
  - priority_level (enum: ask_first, backup, emergency)
  - availability_notes (text, optional)
  - is_active (boolean)
  - acceptance_rate (calculated)
  - avg_response_time_hours (calculated)
  - total_sits (calculated)
  - created_at

babysitter_requests
  - id (PK)
  - suggestion_id (FK)
  - family_id (FK)
  - activity_date
  - start_time
  - end_time
  - duration_hours
  - num_children
  - children_ages (array)
  - estimated_pay
  - special_instructions (text, optional)
  - status (enum: pending, accepted, declined, cancelled, completed)
  - request_round (1, 2, 3 for priority rounds)
  - created_at

babysitter_request_recipients
  - id (PK)
  - babysitter_request_id (FK)
  - trusted_sitter_id (FK)
  - sent_at
  - sent_via (enum: sms, email)
  - response_status (enum: pending, accepted, declined, no_response)
  - responded_at
  - response_text (raw response)

sitting_sessions
  - id (PK)
  - babysitter_request_id (FK)
  - trusted_sitter_id (FK - who accepted)
  - family_id (FK)
  - date
  - start_time
  - end_time
  - actual_duration_hours (if different from planned)
  - agreed_pay
  - payment_status (enum: pending, paid, disputed)
  - paid_at
  - completed_at
  - notes

-- Analytics & Tracking
user_events
  - id (PK)
  - user_id (FK)
  - event_type (enum: signup, suggestion_viewed, suggestion_accepted, etc.)
  - event_properties (JSON)
  - created_at
```

### API Endpoints (Key Routes)

**Authentication**
- `POST /auth/google` - OAuth login
- `POST /auth/logout` - Sign out
- `GET /auth/me` - Get current user

**Onboarding**
- `POST /onboarding/family-profile` - Save family details
- `POST /onboarding/interests` - Save interest ratings
- `POST /onboarding/want-to-try` - Add Want to Try items
- `POST /onboarding/sitters` - Add trusted sitters
- `POST /onboarding/complete` - Mark onboarding done

**Calendar**
- `POST /calendar/connect` - Connect Google Calendar
- `GET /calendar/sync` - Trigger manual sync
- `GET /calendar/free-time` - Get detected free time blocks

**Suggestions**
- `GET /suggestions` - Get active suggestions
- `GET /suggestions/:id` - Get suggestion details
- `POST /suggestions/:id/accept` - Accept suggestion
- `POST /suggestions/:id/decline` - Decline with feedback
- `POST /suggestions/:id/complete` - Mark as done with rating

**Babysitters**
- `GET /sitters` - Get trusted sitter roster
- `POST /sitters` - Add new sitter
- `PUT /sitters/:id` - Update sitter
- `DELETE /sitters/:id` - Remove sitter
- `POST /sitters/request` - Manually request babysitter
- `GET /sitters/requests` - Get babysitter request history
- `POST /sitters/requests/:id/cancel` - Cancel request
- `POST /webhooks/sms` - Twilio webhook for SMS responses

**User Management**
- `GET /profile` - Get user profile
- `PUT /profile` - Update profile
- `PUT /preferences` - Update preferences
- `GET /activity-history` - Get completed activities
- `DELETE /account` - Delete account

**Admin** (Internal)
- `POST /admin/activities` - Add curated activity
- `PUT /admin/activities/:id` - Update activity
- `GET /admin/metrics` - System metrics dashboard

### Security & Privacy

**Authentication & Authorization**
- Google OAuth 2.0 for user authentication
- JWT tokens for session management
- Role-based access control (user vs. admin)

**Data Encryption**
- All data encrypted in transit (TLS 1.3)
- Sensitive data encrypted at rest (Google tokens, payment info)
- AES-256 encryption for PII

**Privacy Measures**
- Minimal calendar data storage (only free/busy, not event details)
- No sale of user data (explicitly stated in ToS)
- GDPR-compliant data export and deletion
- Transparent data usage policy
- SOC 2 Type II compliance roadmap (Year 2)

**API Security**
- Rate limiting (per user, per IP)
- API key authentication for third-party integrations
- Input validation and sanitization
- SQL injection prevention (parameterized queries)
- XSS protection
- CSRF protection

**Third-Party Data**
- Google Calendar API: Minimal scope requests (read calendar, write events only)
- Twilio: PII not stored, only phone numbers
- Affiliate links: No user tracking beyond clicks

---

## MVP Scope & Phasing

### MVP Features (Months 1-4)

**Must-Have:**
✅ Google Calendar OAuth integration
✅ Free time detection (basic algorithm)
✅ Family profile & interest onboarding
✅ Want to Try list management
✅ Manually curated activity database (100-200 activities, 3 cities)
✅ Basic matching algorithm (rule-based, no ML)
✅ Calendar event creation with suggestion details
✅ Trusted sitter roster management
✅ Automated babysitter request via SMS/email
✅ Two-way SMS response handling (YES/NO)
✅ Babysitter confirmation and reminders
✅ Basic user dashboard
✅ Activity acceptance/decline tracking
✅ Post-activity feedback (simple rating)

**Acceptable Limitations:**
- Manual activity curation (no API integrations yet)
- Limited to 3 metro areas initially
- Rule-based matching (no machine learning)
- No payment processing (track manually)
- Basic SMS parsing (YES/NO only)
- No sitter dashboard (SMS/email only)
- Simple mobile-responsive web (not native app)

**Success Criteria for MVP:**
- 50 families complete onboarding
- 35%+ suggestion acceptance rate
- 70%+ babysitter request success rate
- 40%+ month-3 retention
- NPS score >40

---

### Post-MVP Enhancements (Months 5-8)

**Phase 2: Intelligence & Scale**
- Machine learning recommendation algorithm
- Collaborative filtering ("couples like you enjoyed...")
- Expand to 10 metro areas
- Google Places API integration (supplement curated data)
- Weather API integration (outdoor activity optimization)
- Advanced SMS parsing (handle more natural responses)
- Sitter web dashboard (optional for engaged sitters)

**Phase 3: Monetization & Growth**
- Freemium tier structure (2 suggestions/month free)
- Premium subscription ($8.99/month)
- Affiliate link integration (track commissions)
- Booking integration (direct booking from app)
- Integrated payment processing (Stripe)
- Referral program (invite friends, earn credits)

**Phase 4: Advanced Features**
- Native mobile apps (iOS, Android)
- At-home activity suggestions (movie night kits, cooking experiences)
- Social features (share completed activities with friends)
- Babysitting co-op feature (reciprocal sitting credits)
- Multi-family activity suggestions (group outings)
- Annual "Year in Review" summary
- Gift mode (gift subscription to another couple)

---

## Monetization Strategy

### Primary Revenue: Affiliate Commissions

**Model:** Free app with revenue from activity booking commissions

**Affiliate Partners:**
- **Viator/GetYourGuide:** 8-10% commission on tours/activities
- **Booking.com:** 4-5% commission on restaurant/venue bookings
- **OpenTable:** $1-2 per seated diner
- **Eventbrite:** 5-7% commission on ticket sales
- **Local partnerships:** 10-15% commission on curated experiences

**Revenue Projection:**
- Average activity cost: $75
- Average commission: 6% = $4.50 per completed activity
- If 100K active families complete 2 activities/month
- = 200K activities × $4.50 = $900K/month = $10.8M/year

**Why This Works:**
- Aligns incentives: We win when users actually DO activities
- No user friction (everything is free)
- Larger TAM (no paywall to entry)
- Focus on suggestion quality (our only metric that matters)

### Secondary Revenue: Premium Features (Optional)

**Free Tier:**
- 2 activity suggestions per month
- Basic babysitter coordination
- Limited to Google Calendar
- Standard support

**Premium Tier ($8.99/month or $89/year):**
- Unlimited activity suggestions
- Priority babysitter matching
- Apple Calendar integration
- Advanced filters (dietary restrictions, accessibility)
- "Date Night Packages" (activity + restaurant combo)
- Babysitting payment integration
- Concierge support (chat with real human)
- Annual "Year in Review" keepsake

**Conversion Assumption:** 12% of free users convert to premium

**Revenue Projection (Conservative):**
- Year 1: 40K users × 12% × $89 = $426K
- Year 2: 120K users × 12% × $89 = $1.28M
- Year 3: 220K users × 12% × $89 = $2.35M

**Note:** Market research shows affiliate revenue will dominate (87-96% of total), so premium is optional revenue stream, not primary focus.

### Tertiary Revenue: B2B Licensing (Future)

Pivot option if B2C traction is strong:
- License technology to HR departments for employee wellness
- Corporate team-building suggestion engine
- $2K-5K per year per company (500-2000 employees)

---

## Risks & Mitigation

### Risk 1: Low Suggestion Acceptance Rate
**Risk:** Suggestions feel generic, users ignore them
**Impact:** Core value proposition fails, no revenue
**Mitigation:**
- Obsessive onboarding (get preferences right)
- Start with manual curation (quality over quantity)
- Limit to 2-4 suggestions/month (not spammy)
- A/B test suggestion formats
- Collect feedback on every declined suggestion
**Kill Signal:** <25% acceptance after 50 users → major pivot needed

### Risk 2: Poor Retention (Users Churn Quickly)
**Risk:** B2C median retention is 49%, fitness apps lose 71% by month 3
**Impact:** Can't build sustainable user base
**Mitigation:**
- Build habit loops (weekly check-ins, monthly summaries)
- Quick wins (show value in first week)
- Both partners engaged (not just one)
- Continuous value delivery (not just initial novelty)
- Gamification (streak tracking, milestone celebrations)
**Kill Signal:** <30% month-3 retention → consider B2B pivot

### Risk 3: Babysitter Coordination Complexity
**Risk:** Technical challenges, sitter confusion, coordination errors
**Impact:** Biggest differentiator becomes biggest liability
**Mitigation:**
- Extremely clear SMS/email messaging
- Test with 20 real families before launch
- Manual oversight in MVP (human verification)
- Fallback to manual coordination if system fails
- Sitter feedback loop (ask them what's confusing)
**Kill Signal:** <40% sitter acceptance rate or frequent coordination errors

### Risk 4: Privacy Concerns Prevent Adoption
**Risk:** Users reluctant to grant calendar access
**Impact:** Can't detect free time, core functionality broken
**Mitigation:**
- Privacy-first messaging (we only read free time blocks)
- Transparent data policy (no event details stored)
- Social proof (testimonials from early users)
- Security certifications (SOC 2 roadmap)
- Optional "view-only trial" mode
**Kill Signal:** <50% of signups grant calendar access

### Risk 5: High Customer Acquisition Cost
**Risk:** CAC $18-30 for lifestyle apps, may not be profitable
**Impact:** Can't achieve 3:1 LTV:CAC ratio
**Mitigation:**
- Viral referral mechanics (couples invite couples)
- Content marketing (SEO-optimized blog content)
- Community building (Reddit, parenting forums)
- Influencer partnerships (micro-influencers)
- Product-led growth (free tier drives word of mouth)
**Kill Signal:** CAC >$50 and not improving → reassess channels

### Risk 6: Activity Data Quality Issues
**Risk:** Curated database is incomplete, outdated, or inaccurate
**Impact:** Bad suggestions, users lose trust
**Mitigation:**
- Start small (3 cities, 100 activities each, meticulously researched)
- Quarterly manual review and updates
- User feedback loop (report outdated info)
- Partner with local publications for data quality
- API integration for real-time data (post-MVP)
**Kill Signal:** >10% of suggestions are outdated/incorrect

---

## Success Metrics & KPIs

### North Star Metric
**Completed Quality Time Activities per Family per Month**
- Target: 2-4 activities/month
- Why: Directly measures if we're achieving our mission

### Primary Metrics

**User Acquisition:**
- Signups per week
- Onboarding completion rate (target: 85%)
- Calendar connection rate (target: 90%)
- Time to first suggestion (target: <7 days)

**Engagement:**
- Monthly Active Users (MAU)
- Suggestion acceptance rate (target: 35-45%)
- Activity completion rate (target: 60-70%)
- Dashboard visits per week (target: 2+)

**Retention:**
- Day 7 retention (target: 70%)
- Day 30 retention (target: 50%)
- Month 3 retention (target: 40%)
- Month 6 retention (target: 30%)

**Babysitter Coordination:**
- Babysitter request success rate (target: 70%)
- Average sitter response time (target: <3 hours)
- Coordination error rate (target: <5%)
- Sitter satisfaction (target: 4+ stars)

**Quality:**
- Suggestion relevance score (user-rated, target: 4+/5)
- NPS score (target: 50+)
- Activity rating average (target: 4+/5)
- Feedback response rate (target: 50%)

**Revenue:**
- Monthly Recurring Revenue (MRR)
- Affiliate commission per user per month (target: $5-10)
- Premium conversion rate (target: 12%)
- Customer Lifetime Value (LTV) (target: $200+)
- Customer Acquisition Cost (CAC) (target: <$30)
- LTV:CAC ratio (target: 3:1 minimum)

---

## Competitive Analysis

### Direct Competitors

**1. Cupla (Couples Calendar App)**
- **What they do:** Shared calendar for couples
- **What they don't do:** Activity suggestions, local discovery
- **Our advantage:** We're proactive (we suggest), they're passive (just organize)

**2. Cozi Family Organizer**
- **What they do:** Family calendar, shopping lists, to-do lists
- **What they don't do:** Activity discovery, personalization
- **Our advantage:** We focus on quality time, not logistics

**3. Eventbrite / Meetup**
- **What they do:** Event discovery platforms
- **What they don't do:** Calendar integration, personalization, babysitter coordination
- **Our advantage:** Fully automated (they require manual searching)

**4. Care.com / UrbanSitter**
- **What they do:** Find and book babysitters
- **What they don't do:** Activity suggestions, calendar integration
- **Our advantage:** We solve both problems (activity + childcare) in one flow

### Indirect Competitors

**Google Calendar:** Passive scheduling tool (we add intelligence)
**Yelp/Google Maps:** Generic discovery (we personalize to stated desires)
**Date Night Subscription Boxes:** Physical products (we're digital, local, flexible)

### Competitive Moat

**Our Unique Combination:**
1. Automatic free time detection (not manual)
2. Personalized to stated desires (not generic discovery)
3. Local activity matching (not national/generic)
4. Integrated babysitter coordination (removes biggest barrier)
5. Calendar-native (suggestions appear where users already look)

**No competitor offers all five.** Each solves 1-2 pieces, we solve the entire chain.

---

## Go-to-Market Strategy

### Phase 1: Local Launch (Months 1-3)

**Target Markets:** Austin, TX (initial focus)
- Mid-size city with high activity density
- Strong family demographic
- Active tech community (early adopters)
- Rich local culture (festivals, outdoor activities, food scene)

**Launch Tactics:**
1. **Friends & Family Beta:** 20-30 couples
2. **Local Parenting Groups:** Facebook groups, mom meetups
3. **Influencer Partnerships:** 3-5 micro-influencers (10K-50K followers)
4. **ProductHunt Launch:** Gain initial tech community traction
5. **PR:** Local news outlets ("Austin startup helps families reconnect")

**Goal:** 500 families in first 3 months

### Phase 2: Metro Expansion (Months 4-8)

**Add 2-3 cities:** Denver, Portland, San Diego
- Similar demographics to Austin
- Strong activity cultures
- Manageable curation workload

**Expansion Tactics:**
1. **Referral Program:** Austin users invite friends in other cities
2. **Content Marketing:** City-specific guides ("50 family activities in Denver")
3. **Paid Ads:** Facebook/Instagram targeting couples with young kids
4. **Partnerships:** Local family bloggers, parenting magazines

**Goal:** 5,000 families across 4 cities

### Phase 3: Scale (Months 9-12)

**Add 6 more cities:** Top 10 metros (NYC, LA, SF, Chicago, Seattle, Boston)
**Goal:** 40,000 families by end of Year 1

**Scaling Tactics:**
1. **Viral Growth:** Strong referral mechanics ("Invite 3 couples, get 2 months free")
2. **SEO Content:** Rank for "date night ideas [city]", "family activities [city]"
3. **Paid Acquisition:** Scale Facebook/Instagram ads (optimize for CAC <$30)
4. **B2B Pilot:** Test with 2-3 companies for employee wellness

---

## Open Questions & Future Research

1. **Activity Curation:** Should we hire local "activity scouts" in each city or crowdsource?

2. **Sitter Compensation Model:** Should we stay out of payments entirely or take 3-5% transaction fee?

3. **Frequency Optimization:** What's the ideal suggestion frequency? Weekly? Bi-weekly?

4. **Partner Dynamics:** If one partner is much more engaged, how do we engage the other?

5. **At-Home Activities:** Should we prioritize these (lower barrier) or focus on "going out"?

6. **Social Features:** Should completed activities be shareable with friends? Privacy concerns?

7. **Freemium vs. Free Forever:** Can affiliate revenue alone sustain the business?

8. **B2B vs. B2C:** If B2C retention struggles, when should we pivot to enterprise?

9. **Gamification:** Streaks, badges, milestones - helpful or gimmicky?

10. **International Expansion:** Stay US-only or expand to Canada/UK/Australia?

---

## Appendix

### User Testing Plan

**Pre-Launch Testing:**
- 10 couple interviews about "things you've been meaning to do"
- 20 families in closed beta (4-week test)
- 5 trusted sitters in usability testing (SMS flow)

**Post-Launch Testing:**
- Monthly user surveys (NPS, feature requests)
- Quarterly in-depth interviews (10 families)
- A/B testing on suggestion formats
- Analytics monitoring (Mixpanel/Amplitude)

### Regulatory Considerations

**Privacy Compliance:**
- GDPR (if expanding to EU)
- CCPA (California users)
- COPPA (children's data - minimal exposure)

**Babysitting Liability:**
- Clear ToS: We're a coordination platform, not a vetting service
- No liability for sitter-family interactions
- Encourage users to conduct their own background checks
- Insurance recommendation for sitters

**Affiliate Disclosures:**
- FTC-compliant disclosure on affiliate links
- Transparent messaging in app and marketing

---

## Version History

**v1.0 - February 12, 2026**
- Initial PRD draft
- Core features defined
- Babysitter coordination workflow detailed
- MVP scope and phasing outlined

---

**Document Owner:** Dylan Mattern  
**Stakeholders:** Engineering Team, Design Team, Marketing Team  
**Review Cycle:** Quarterly or as major features are added  
**Feedback:** Open to all team members and beta users
