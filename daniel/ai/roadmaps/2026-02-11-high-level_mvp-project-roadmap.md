# Family Quality Time App - Project Roadmap

**Created:** 2026-02-11
**Status:** Planning
**Timeline:** MVP Demo (4-5 days) → V1 Production (12 weeks) → V2+ (TBD)

---

## Document Alignment

This roadmap synthesizes the following source documents:

**Strategic Foundation:**
- **PRD** (`aiDocs/prd.md`) - Product vision, market research, success metrics
- **MVP Spec** (`aiDocs/mvp.md`) - Chrome extension demo (Phase 1)
- **Architecture** (`aiDocs/architecture.md`) - Technical implementation details
- **Context** (`aiDocs/context.md`) - Current focus and immediate next steps

**Key Decisions Reflected:**
- Phase 1 (MVP Demo): Chrome extension approach per mvp.md
- Phases 2-4 (V1 Production): 12-week timeline per PRD
- Tech stack: Next.js, Node/Express, PostgreSQL, Resend per architecture.md
- Recommendation engine: Rule-based scoring algorithm detailed in architecture.md
- Vision API: Technical implementation specs from architecture.md

---

## Development Philosophy

**Keep It Clean:**
- No over-engineering - solve current problems, not hypothetical future ones
- No legacy compatibility layers - this is greenfield, build it right
- No cruft - if it's not serving a clear purpose, don't build it
- Ship working features over perfect abstractions

**Build Only What's Needed:**
- Start with simplest solution that validates the core hypothesis
- Add complexity only when proven necessary
- Delete code that isn't being used
- Avoid "just in case" features

---

## Phase 0: Foundation (Current - Completed)

**Goal:** Set up project, validate technical approach, gather real documentation

**Deliverables:**
- ✅ Project structure (aiDocs/, ai/, .gitignore)
- ✅ PRD with market research and competitive analysis
- ✅ MVP spec (Chrome extension demo - 4-5 day scope)
- ✅ Architecture document (MVP, V1, V2+)
- ✅ Real API documentation (sharp, OpenAI) from context7
- ✅ CLAUDE.md with behavioral guidelines

**Key Decisions Made:**
- MVP Demo: Chrome extension (fastest path to validate)
- V1 Production: Next.js + Node.js/Express + PostgreSQL
- Email delivery: Resend (chosen over SendGrid for better DX)
- Curated activities for MVP (not API-first)
- Google Calendar as primary interface
- 12-week timeline for V1 launch (Phases 2-4)

**Next Step:** → Phase 1 (MVP Demo)

---

## Phase 1: MVP - Chrome Extension Demo (4-5 days)

**Goal:** Build functional proof-of-concept to validate core value proposition

**Note:** This is a demo/prototype, not production software. The purpose is to validate assumptions about calendar access, suggestion quality, and user value before building the full product in Phases 2-4.

**Success Criteria:**
- Real Google Calendar OAuth integration (not mocked)
- Correctly detects free time from actual calendar
- Shows 2 relevant, curated suggestions
- Successfully creates event in Google Calendar
- Demo completes in <60 seconds
- Gets "wow, that's useful!" reaction

**Phases:**

### 1A: Setup & Calendar Access (Day 1)
- Google Cloud Project setup (enable Calendar API v3)
- OAuth 2.0 credentials for Chrome extension
- Extension boilerplate (Manifest V3 format)
  - manifest.json (permissions, OAuth config)
  - popup.html/css (UI skeleton)
  - background.js (service worker)
  - calendar.js (API wrapper)
  - icons/ (16px, 48px, 128px)
- Test: Can read calendar events via API using chrome.identity

### 1B: Free Time Detection (Day 2)
- Calendar event fetching logic
- Free block detection algorithm (2-4 hour windows)
- Weekend filtering (Sat/Sun 9am-6pm)
- UI display of next free time
- Test: Accurately finds open time slots

### 1C: Suggestions Engine (Day 3)
- Curate 10-15 great local family activities
- Build simple keyword matching logic
- "Want to try" form with local storage
- Suggestion card UI design
- Test: Suggestions feel relevant and actionable

### 1D: Calendar Integration (Day 4)
- Event creation via Google Calendar API
- Handle success/error states
- Polish UI (styles, animations, responsive)
- Test: Event appears in Google Calendar immediately

### 1E: Testing & Demo Prep (Day 5)
- End-to-end testing of full flow
- Bug fixes and edge cases
- Record demo video (backup for live demos)
- Write installation instructions
- Test with 3-5 real users (friends/family)

**Deliverables:**
- Working Chrome extension (.zip for manual installation)
- Manifest V3 configuration with Google Calendar OAuth
- Curated activities database (10-15 items as JSON)
- Demo video (<2 minutes)
- Installation guide (README.md)
- Initial user feedback notes (3-5 testers)

**What We'll Learn:**
- Do people grant calendar access?
- Are curated suggestions better than API results?
- What types of activities get accepted?
- Is the UX intuitive?

**What We Won't Build (per mvp.md):**
- Backend/database (client-side only - everything in Chrome storage)
- User accounts (single-user demo)
- Automation (manual trigger via extension icon)
- ML recommendations (simple keyword matching, curated activities)
- Mobile version (desktop Chrome only)
- External API calls (Google Places, Eventbrite - use curated list instead)
- Family profile setup (hard-code demo assumptions: 2 kids, ages 3 and 7)
- Multi-calendar support (single calendar only)

**Next Step:** → User testing → Phase 2 decision

---

## Phase 2: V1 Core - Web App Foundation (Weeks 1-4)

**Goal:** Build production web app with essential features

**Context:** This begins the 12-week timeline to launch production V1 (as outlined in PRD). Phases 2-4 = 12 weeks total.

**Success Criteria:**
- 50 beta users onboarded
- >40% suggestion acceptance rate
- Email delivery working reliably
- User can manage profile and preferences

**Phases:**

### 2A: Infrastructure Setup (Week 1)
- Next.js 14 project initialization (TypeScript + Tailwind CSS)
- PostgreSQL database on Railway
- Prisma ORM setup and migrations
- Clerk authentication integration
- Vercel deployment pipeline
- Resend email service integration (decision: Resend over SendGrid)
- Environment configuration (dev, staging, prod)

### 2B: User Onboarding (Week 2)
- Landing page design and copy
- Google Calendar OAuth flow (web-based)
- Family profile creation form
  - Kid ages
  - Interests (multi-select)
  - Budget tier (slider)
  - Max drive time (dropdown)
  - "Want to try" list (textarea)
- Profile storage in database
- Test: User can complete onboarding in <5 minutes

### 2C: Calendar Sync & Analysis (Week 3)
- Google Calendar API integration (server-side)
- Free time detection service
- Calendar scan storage (for history)
- Manual sync trigger (user-initiated)
- Error handling (token refresh, rate limits)
- Test: Accurately detects free time for various calendar patterns

### 2D: Suggestion Generation (Week 4)
- Activity data sources integration
  - Google Places API wrapper (family activities, parks, museums, etc.)
  - Eventbrite API wrapper (local events)
  - Curated activities database (manual seed - 10-15 per city as per MVP learnings)
- Rule-based recommendation engine (see architecture.md for scoring algorithm)
  - Interest matching (30 points)
  - "Want to Try" list prioritization (25 points)
  - Age appropriateness filtering (20 points)
  - Budget constraints (15 points)
  - Drive time constraints (10 points)
  - Recency penalties (avoid repetition)
- Suggestion storage in database
- Test: Generates 2 relevant suggestions per user with >30 score threshold

**Deliverables:**
- Web app live at familytime.com (staging)
- Database schema implemented
- Core APIs functional
- 10-20 beta users onboarded

**What We'll Learn:**
- Is web onboarding easier than Chrome extension?
- What's the drop-off rate in onboarding? (Target: <30% per PRD)
- Are API-sourced suggestions better than curated?
- What interest categories are most common?
- Does the scoring algorithm (from architecture.md) produce relevant suggestions?

**Next Step:** → Phase 3

---

## Phase 3: V1 Polish - Automation & Delivery (Weeks 5-8)

**Goal:** Add weekly automation, email delivery, and feedback loops

**Success Criteria:**
- Suggestions sent automatically every week
- >50% email open rate
- >70% of accepted suggestions actually happen
- User feedback collected and stored

**Phases:**

### 3A: Email System (Week 5)
- Resend integration (already chosen in Phase 2A)
- Email template design (React Email or MJML as per PRD)
  - Suggestion digest (2 suggestions with photos)
  - "Add to Calendar" button (.ics file generation)
  - Follow-up survey ("How was it?")
- Email scheduling system (BullMQ jobs)
- Deliverability setup (SPF, DKIM, DMARC records)
- Test: Emails render correctly across clients (Gmail, Outlook, Apple Mail)

### 3B: Background Jobs (Week 6)
- BullMQ + Redis setup
- Weekly cron job: Scan calendars (Sundays 8pm)
- Weekly cron job: Send emails (Fridays 8am)
- Weekly cron job: Follow-up surveys (Mondays 10am)
- Job monitoring and error handling
- Test: Jobs run reliably every week

### 3C: Feedback Collection (Week 7)
- Suggestion status tracking (sent, accepted, rejected, ignored)
- Thumbs up/down feedback form
- Feedback comments (optional text)
- Analytics dashboard (admin-only)
  - Acceptance rate by activity type
  - Most popular interests
  - Average response time
- Test: Feedback is captured and displayed

### 3D: Recommendation Improvements (Week 8)
- Feedback-based filtering (don't repeat rejected types)
- Recency penalty (avoid suggesting same place twice)
- Diversity scoring (vary activity types week-to-week)
- "Want to try" prioritization
- Test: Week 2+ suggestions feel more personalized

**Deliverables:**
- Fully automated weekly cycle
- Email templates live
- Feedback system functional
- Admin dashboard for monitoring

**What We'll Learn:**
- Is weekly cadence right, or should it be more/less frequent?
- Which activity types have highest acceptance?
- Do users engage with follow-up surveys?
- What feedback themes emerge?

**Next Step:** → Beta launch → Phase 4 decision

---

## Phase 4: V1 Launch - Beta Testing (Weeks 9-12)

**Goal:** Recruit 100 beta users, validate willingness to pay, iterate on feedback

**Success Criteria:**
- 100 active users
- >60% week-2 retention
- >30% would pay $8-10/month
- NPS score >40

**Phases:**

### 4A: User Acquisition (Week 9)
- Beta waitlist landing page
- Social media launch (Reddit, Facebook groups)
- Email 50 parent influencers/bloggers
- Referral system (3 invite codes per user)
- Test: Can we recruit 50 users in week 1?

### 4B: User Experience Refinement (Week 10)
- User interview sessions (5-10 users)
- A/B test email subject lines
- A/B test suggestion formats (cards vs list)
- Optimize onboarding flow based on drop-off data
- Fix top 5 reported bugs
- Test: Onboarding completion rate improves

### 4C: Monetization Experiment (Week 11)
- Pricing page design
- Stripe integration
- Free tier limits (1 suggestion/week)
- Premium tier features (3 suggestions/week, SMS, multi-calendar)
- Paywall after 4 weeks free trial
- Test: Conversion rate to paid

### 4D: Data Analysis & Decision (Week 12)
- Compile all metrics (acceptance rate, retention, WTP)
- User feedback synthesis
- Cost analysis (API usage, hosting)
- Ship-or-pivot decision
- Roadmap for next phase

**Deliverables:**
- 100 beta users in system
- Payment flow functional
- Full metrics report
- Go/no-go decision for V2

**What We'll Learn:**
- Will people pay for this?
- What's the unit economics?
- What features are most requested?
- Should we pivot focus (couples vs families)?

**Next Step:** → If metrics hit targets, move to Phase 5

---

## Phase 5: V2 - Advanced Features (Months 4-6)

**Goal:** Add features that increase retention and differentiation

**Prioritized by User Feedback:**

### 5A: Multi-Calendar Support
- Sync 2 calendars (both partners)
- Merge events intelligently
- Detect mutual free time
- Both partners receive suggestions

### 5B: Auto-Insert to Calendar
- Request write permissions
- One-click accept → auto-creates event
- Tentative event option (appears as "maybe")
- Reduce friction from email → calendar

### 5C: ML-Based Recommendations
- Collect 3+ months of feedback data (minimum threshold)
- Feature engineering: kid ages, interests, day/time, previous acceptances
- Train simple collaborative filtering or regression model
- Replace rule-based scoring (from architecture.md) with ML predictions
- A/B test ML vs rules (measure acceptance rate lift)

### 5D: Mobile App (React Native)
- iOS + Android app
- Push notifications for suggestions
- In-app calendar view
- Faster than email for engagement

### 5E: Expanded Activity Sources
- Partner with local businesses (featured placement)
- Yelp API integration (restaurants)
- Mommy blogger recommendations (curated)
- City-specific hidden gems database

**Deliverables:**
- At least 2 of the above features shipped
- User retention improves >10%
- Premium conversion improves >5%

**Next Step:** → Phase 6

---

## Phase 6: V3 - Ecosystem Expansion (Months 7-12)

**Goal:** Build moat through integrations and network effects

### 6A: AI Assistant Integration
- ChatGPT action/plugin (architecture pattern defined in architecture.md)
- Google Assistant integration
- Alexa skill
- Voice-based suggestions
- Expose API endpoint for family_time_suggestions action

### 6B: Social Features
- Multi-family event coordination
- Share suggestions with friends
- Group events (playdate scheduling)
- Activity reviews and ratings

### 6C: Booking & Reservations
- In-app restaurant reservations (OpenTable API)
- Museum ticket purchasing
- Babysitter marketplace integration
- End-to-end experience

### 6D: Apple Calendar Support
- Apple Calendar API integration
- iCloud sync
- Expand to non-Google users

### 6E: B2B Partnerships
- Corporate wellness programs
- Integrate with Cozi, TimeTree (white-label)
- Sell API access to family apps

**Deliverables:**
- 1,000+ active users
- Multiple revenue streams (subscriptions + B2B)
- Defensible competitive moat

---

## Future: Vision API Integration (Post-V3)

**Goal:** Enable calendar analysis without OAuth for privacy-conscious users

### Technical Implementation (from architecture.md):

**Workflow:**
1. User uploads calendar screenshot
2. Sharp library processes image and adds text annotations (date markers)
3. OpenAI Vision API (gpt-4-vision or later) analyzes annotated image
4. Extract free time blocks from visual analysis
5. Generate suggestions based on extracted data

**Tech Stack:**
- `sharp` library (`/lovell/sharp`) - Image processing, text overlays, compositing
- `openai-node` SDK (`/openai/openai-node`) - Vision API integration
- See detailed docs: `ai/guides/sharp-text-overlay-docs.md` and `ai/guides/openai-sdk-docs.md`

**Code Pattern:**
```typescript
// 1. Annotate image with Sharp
const annotatedImage = await sharp(imageBuffer)
  .composite([{ input: textOverlayBuffer, gravity: 'north' }])
  .jpeg()
  .toBuffer();

// 2. Analyze with OpenAI Vision
const response = await openai.chat.completions.create({
  model: 'gpt-4-vision',
  messages: [{
    role: 'user',
    content: [
      { type: 'text', text: 'Extract free time blocks...' },
      { type: 'image_url', image_url: { url: base64Image } }
    ]
  }]
});

// 3. Parse JSON response with free blocks
const freeBlocks = JSON.parse(response.choices[0].message.content);
```

**Use Cases:**
- Users hesitant to grant OAuth access
- Apple Calendar users (before official integration)
- Quick one-time analysis without account creation
- Demo tool for marketing

**Technical Complexity:**
- Medium-high (vision API accuracy, error handling)
- Requires user education (how to take good screenshots)
- Fallback to manual time entry if vision fails
- Image quality and calendar app UI variations affect accuracy

**Priority:** Low (nice-to-have, not core value prop)
**Timeline:** Post-V3 (Months 12+)
**Dependencies:** Proven OAuth flow working well first

---

## Success Milestones

**MVP Success (Phase 1):**
- ✅ 5 people say "this is really useful"
- ✅ Demo video gets shared organically
- ✅ Real calendar OAuth works (not mocked)
- ✅ Demo completes in <60 seconds
- ✅ Gets "wow, that's useful!" reaction (per mvp.md)

**V1 Success (Phases 2-4 - from PRD):**
- ✅ 100 beta users onboarded
- ✅ >40% suggestion acceptance rate (primary metric)
- ✅ >70% actual attendance rate
- ✅ >30% willing to pay $8-10/month
- ✅ >50% email open rate
- ✅ >60% week-2 retention
- ✅ Positive unit economics

**V2 Success (Phase 5):**
- ✅ 500 active users
- ✅ >60% week-8 retention
- ✅ 15%+ premium conversion
- ✅ Profitable at current scale

**V3 Success (Phase 6):**
- ✅ 1,000+ active users
- ✅ Multiple revenue streams
- ✅ Defensible competitive moat
- ✅ Considering Series A fundraising

---

## Risk Mitigation

**Low Calendar Connection Rate:**
- Lead with trust messaging
- Show example suggestions before OAuth
- Offer vision API alternative (screenshot upload)

**Poor Suggestion Quality:**
- Start curated-first, API-second
- Collect feedback aggressively
- Iterate weekly on matching algorithm

**Retention Drop After Week 2:**
- Add feedback loop ("Why didn't you like this?")
- Experiment with cadence (2x/week, daily)
- Improve suggestion relevance via ML

**Can't Monetize:**
- Make free tier very limited (1 suggestion/week per PRD)
- Premium tier: $8-10/month (3 suggestions/week, multi-calendar, SMS)
- B2B backup plan (sell to Cozi, corporate wellness, API access)
- Affiliate revenue from activity bookings
- Cost structure from architecture.md: ~$60/mo for 100 users, ~$660/mo for 1,000 users

**Platform Risk (Google Builds This):**
- Move fast, ship V1 in 12 weeks
- Build deep family-specific features
- Create network effects (social, multi-family)

---

## Principles to Maintain

**Throughout All Phases:**

1. **Ship fast, iterate faster**
   - Every feature should ship in <2 weeks
   - If it takes longer, break it down further

2. **User feedback trumps gut feel**
   - Talk to 5 users before building anything big
   - Measure everything, decide based on data

3. **Keep the codebase clean**
   - Delete unused code immediately
   - Refactor when adding 3rd instance of duplication
   - No "temporary hacks" - fix it or don't ship it

4. **Avoid premature optimization**
   - Don't build for 10,000 users when you have 10
   - Don't add caching until you measure it's needed
   - Don't generalize until you have 3 use cases

5. **Focus on the magic moment**
   - "Free time + desires → specific plan in 10 seconds"
   - Every feature should reinforce this core value
   - Cut anything that distracts from it

---

## Decision Points

**After Phase 1 (MVP Demo):**
- If demo fails to resonate → Pivot focus or kill project
- If demo gets strong reaction → Move to Phase 2

**After Phase 4 (V1 Beta):**
- If <20% acceptance rate → Pivot to different use case
- If <10% willing to pay → Rethink monetization
- If metrics hit targets → Move to Phase 5

**After Phase 5 (V2):**
- If retention still low → Consider pivot to B2B only
- If retention strong → Scale user acquisition
- If profitable → Consider fundraising for Phase 6

---

## What We're NOT Building

**To maintain focus, we explicitly will not:**

- Generic calendar app (we're activity-focused)
- Task management system (not our job)
- Meal planning (too different from experiences)
- Kids' activity scheduler (we're family time, not logistics)
- Social network (only tactical social features in V3)
- Marketplace for local businesses (not until we have scale)

**Keep saying no to:**
- Feature requests that don't serve core value prop
- Integrations that add complexity without clear ROI
- "Nice to have" features when core features aren't nailed
- Anything that makes onboarding longer than 5 minutes

---

## Questions to Answer Along the Way

**Phase 1:**
- Do people trust us with calendar access?
- Is email delivery enough, or need push notifications?

**Phase 2-4:**
- Weekly cadence right, or more/less frequent?
- Couples vs families - which converts better?
- What budget tier is most common?

**Phase 5-6:**
- Which V2 features drive retention most?
- Is B2B a viable revenue stream?
- Should we build mobile app or focus on web?

---

## References

**Source Documents:**
- [PRD](../../aiDocs/prd.md) - Product requirements, market research, 12-week V1 timeline
- [MVP Spec](../../aiDocs/mvp.md) - Chrome extension demo details (4-5 days)
- [Architecture](../../aiDocs/architecture.md) - Tech stack, recommendation algorithm, Vision API implementation
- [Context](../../aiDocs/context.md) - Current focus and immediate next steps
- [Market Research](../guides/quality-time-app-market-research.md) - Competitive analysis and user pain points

**Technical Guides (Library Documentation from context7):**
- [Next.js](../guides/nextjs_context7.md) - React framework with App Router and SSR (Phase 2-6)
- [Prisma](../guides/prisma_context7.md) - TypeScript ORM for PostgreSQL (Phase 2-6)
- [Clerk](../guides/clerk_context7.md) - Authentication and user management (Phase 2-6)
- [Resend](../guides/resend_context7.md) - Email API and delivery (Phase 3-6)
- [BullMQ](../guides/bullmq_context7.md) - Job queue with Redis for background tasks (Phase 3-6)
- [Stripe](../guides/stripe_context7.md) - Payment processing and subscriptions (Phase 4-6)
- [React Email](../guides/react-email_context7.md) - React components for email templates (Phase 3-6)
- [Google Calendar API](../guides/google-calendar-api_context7.md) - Calendar integration (Phase 1-6)
- [Sharp](../guides/sharp-text-overlay-docs.md) - Image processing for Vision API (Future/V2+)
- [OpenAI SDK](../guides/openai-sdk-docs.md) - Vision API integration (Future/V2+)

---

**Last Updated:** 2026-02-11
**Next Review:** After Phase 1 completion
**Version:** 1.2 (Added library documentation references from context7)
