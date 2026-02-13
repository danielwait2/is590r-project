# Implementation Plan - Family Quality Time App

**Timeline:** 7 weeks  
**Goal:** Demo-ready MVP  
**Solo developer:** Dylan  
**Created:** February 12, 2026  
**Last Updated:** February 12, 2026 (aligned with MVP spec)

---

## ⚠️ Clean Code Philosophy

This is a **greenfield project** - we're building from scratch with modern tools.

**Avoid:**
- Over-engineering (no "future-proof" abstractions we don't need today)
- Cruft (no unused code, commented-out sections, or "just in case" features)
- Legacy compatibility (no workarounds for old browsers, deprecated APIs, or backwards compatibility)

**Embrace:**
- Modern standards (ES modules, async/await, latest Next.js patterns)
- Simple solutions (if it works and is readable, ship it)
- Delete over comment (remove unused code instead of commenting it out)

If something feels like "best practice cargo cult" but adds complexity, skip it for MVP.

---

## Key Decisions (Aligned with MVP Spec)

**Authentication:** Using **Supabase Auth only** (not NextAuth) - simpler, fewer dependencies  
**Timeline:** 7 weeks (sequential phases, no overlaps)  
**Activity Target:** 100-150 activities in **Austin, TX only**  
**Onboarding:** Single-page with 4 inputs (location, kids, Want to Try list, one sitter)  
**Dependencies:** Minimal - removed next-auth, zod, @headlessui, @heroicons

---

## Overview

Build a working demo that shows:
1. User connects Google Calendar
2. App detects free time
3. App suggests activity matching user's interests
4. Activity appears in calendar with details
5. App sends SMS to babysitters
6. Babysitter responds YES
7. Calendar updates with confirmed sitter

---

## Phase 1: Foundation (Week 1-2)

**Goal:** Basic app infrastructure + authentication

### Deliverables
- [ ] Next.js 14 project with TypeScript
- [ ] Supabase project created
- [ ] Database schema deployed
- [ ] User signup/login working
- [ ] Deployed to Vercel

### Key Tasks
- Set up Next.js with App Router
- Create Supabase project
- Design and create database tables
- Implement Supabase Auth (Google OAuth)
- Build basic UI (landing, signup, dashboard)
- Build single-page onboarding (location, kids, Want to Try list, one sitter)
- Connect local dev to Supabase
- Deploy to Vercel

### Success Criteria
User can sign up, log in, and see empty dashboard.

---

## Phase 2: Calendar Integration (Week 3)

**Goal:** Read user's calendar with OAuth

### Deliverables
- [ ] Google Calendar OAuth working
- [ ] App reads calendar events
- [ ] Calendar data displayed in UI

### Key Tasks
- Create Google Cloud project
- Enable Calendar API
- Set up OAuth 2.0 credentials
- Implement OAuth flow with Supabase Auth
- Build calendar sync function
- Store calendar tokens securely (encrypted in Supabase)
- Display user's calendar events in dashboard

### Success Criteria
User connects Google Calendar via OAuth, app displays their upcoming events.

---

## Phase 3: Activity Discovery (Week 4)

**Goal:** Scrape and curate activities for Austin

### Deliverables
- [ ] Web scraping working (Google Maps + Yelp)
- [ ] Activity database seeded with 100-150 activities
- [ ] Activities stored in database
- [ ] Basic activity browsing UI

### Key Tasks
- Implement web scraping (Cheerio + Axios)
- Scrape Google Maps for activities in Austin
- Scrape Yelp for restaurants/businesses in Austin
- Manually curate and clean scraped data
- Store activities in Supabase (name, address, category, rating)
- Build simple activity browsing view
- Test data quality (at least 100 activities)

### Success Criteria
Database contains 100-150 quality activities in Austin, TX. Activity browsing view shows scraped data.

---

## Phase 4: Free Time Detection & Matching (Week 5)

**Goal:** Detect free time, match with activities, insert suggestions

### Deliverables
- [ ] Free time detection working
- [ ] Matching algorithm working (rule-based)
- [ ] Suggestion generation working
- [ ] Events inserted into Google Calendar

### Key Tasks
- Implement free time detection (3+ hour gaps in calendar)
- Build simple matching algorithm (match "Want to Try" list with activities)
- Generate suggestions (activity + free time + location)
- Format calendar event with rich description
- Insert event via Calendar API
- Add suggestion metadata (extended properties)
- Handle event deletion (track declines)
- Test full flow: Want to Try → free time → suggestion → calendar

### Success Criteria
User adds "kayaking" to Want to Try list → app detects free time Saturday 2-6pm → suggestion appears in Google Calendar: "Try: Kayaking at Lady Bird Lake" with full details.

---

## Phase 5: Babysitter Feature (Week 6)

**Goal:** Automated SMS requests to trusted babysitters

### Deliverables
- [ ] Babysitter roster UI
- [ ] SMS sending working
- [ ] SMS receiving (webhook) working
- [ ] Calendar updates with confirmed sitter

### Key Tasks
- Create Twilio account + phone number
- Build "Trusted Sitters" UI (add/edit/remove)
- Implement SMS sending (request message)
- Set up Twilio webhook
- Parse YES/NO responses
- Update suggestion status in database
- Update calendar event with sitter confirmation
- Send confirmation SMS back to sitter
- Handle multiple sitters (send to all, first accepts wins)
- Test with real phone numbers

### Success Criteria
User gets activity suggestion → app texts 3 sitters → Sarah replies YES → calendar updates "Babysitter: Sarah confirmed" → Sarah gets confirmation text.

---

## Phase 6: Polish & Demo (Week 7)

**Goal:** Demo-ready, looks good, works reliably

### Deliverables
- [ ] Clean, professional UI
- [ ] Demo script/walkthrough
- [ ] Error handling on critical paths
- [ ] Demo video recorded
- [ ] README with setup instructions

### Key Tasks
- UI polish (Tailwind styling)
- Add loading states
- Error handling (failed API calls, etc.)
- Empty states ("No suggestions yet")
- Onboarding flow refinement
- Write demo script
- Record demo video (5 min)
- Test end-to-end flow multiple times
- Fix critical bugs
- Document known limitations

### Success Criteria
Can demo the entire flow in 5 minutes without manual intervention.

---

## Tech Stack Summary

| Layer | Technology | Why |
|-------|-----------|-----|
| **Frontend** | Next.js 14 (App Router) | React framework, great DX |
| **Backend** | Next.js API Routes | Serverless, no separate backend |
| **Database** | Supabase (PostgreSQL) | Free tier, built-in auth |
| **Auth** | Supabase Auth | Built-in OAuth, simple setup |
| **Calendar** | Google Calendar API | Read/write calendar events |
| **SMS** | Twilio | Send/receive SMS |
| **Styling** | Tailwind CSS | Fast, utility-first |
| **Deployment** | Vercel | Free, automatic deploys |
| **Activity Data** | Web scraping (Cheerio) | Free alternative to APIs |

---

## Dependencies to Install

**Core:**
- next@14, react, react-dom, typescript
- @supabase/supabase-js, @supabase/auth-helpers-nextjs

**APIs:**
- googleapis (Google Calendar)
- twilio (SMS)

**Scraping:**
- cheerio, axios

**UI:**
- tailwindcss, postcss, autoprefixer

**Utils:**
- date-fns

---

## Environment Variables Needed

**Supabase:**
- NEXT_PUBLIC_SUPABASE_URL
- NEXT_PUBLIC_SUPABASE_ANON_KEY
- SUPABASE_SERVICE_ROLE_KEY (for server-side operations)

**Google Calendar:**
- GOOGLE_CLIENT_ID
- GOOGLE_CLIENT_SECRET

**Twilio:**
- TWILIO_ACCOUNT_SID
- TWILIO_AUTH_TOKEN
- TWILIO_PHONE_NUMBER

---

## Critical Path (Must Work for Demo)

1. **User signs up** → Creates account
2. **User connects Google Calendar** → OAuth flow
3. **User adds interests** → "Want to try kayaking"
4. **App detects free time** → "Saturday 2-6pm free"
5. **App generates suggestion** → Matches kayaking + free time
6. **Suggestion appears in calendar** → With full details
7. **User adds babysitter** → Sarah +1-555-0123
8. **App sends SMS** → "Can you babysit Saturday 1:30-6pm?"
9. **Sitter replies YES** → Via SMS
10. **Calendar updates** → "Babysitter: Sarah confirmed ✓"

---

## Out of Scope (Not for MVP)

- Apple Calendar support
- Multiple calendar accounts
- Recurring suggestions
- Payment processing
- In-app messaging
- Photo uploads
- Social sharing
- Mobile app
- Email notifications
- Activity reviews/ratings
- Partner coordination (both must accept)
- Advanced matching (ML/AI)
- Real-time notifications
- Analytics dashboard
- Interest rating cards (Love/Like/Neutral/Dislike) - simplified to "Want to Try" list only
- Multiple trusted sitters - just 1 sitter for demo
- Multi-city support - Austin only for MVP

---

## Risks & Mitigation

### Risk 1: OAuth Flow is Complex
**Impact:** High (blocks calendar integration)  
**Mitigation:** Use NextAuth.js (handles OAuth), follow Google's official guide, allocate extra time  
**Contingency:** Manual calendar input for demo if OAuth fails

### Risk 2: Web Scraping Breaks
**Impact:** Medium (no activity data)  
**Mitigation:** Scrape early, cache results, have manual backup list  
**Contingency:** Manually curate 50 activities minimum

### Risk 3: Twilio Webhook Issues
**Impact:** Medium (babysitter feature doesn't work)  
**Mitigation:** Use ngrok for local testing, test thoroughly  
**Contingency:** Manual SMS simulation for demo

### Risk 4: Running Out of Time
**Impact:** High (incomplete demo)  
**Mitigation:** Start with critical path, cut nice-to-haves  
**Contingency:** Demo just calendar suggestions without babysitter feature

---

## Success Metrics (Demo)

### Must Have
- ✅ User can sign up
- ✅ User can connect Google Calendar
- ✅ App detects at least 1 free time block
- ✅ App generates at least 1 suggestion
- ✅ Suggestion appears in Google Calendar
- ✅ SMS sent to babysitter
- ✅ Babysitter can reply YES
- ✅ Calendar updates with confirmation

### Nice to Have
- Multiple activity categories
- 3+ suggestions per week
- Multiple babysitters
- Suggestion history view
- Interest editing
- Pretty UI

---

## Weekly Checkpoints

### End of Week 2
- [ ] App deployed to Vercel
- [ ] User can sign up with Google (Supabase Auth)
- [ ] Database tables created
- [ ] Single-page onboarding working

### End of Week 4
- [ ] Google Calendar connected
- [ ] 100-150 activities scraped for Austin
- [ ] Activities stored in database

### End of Week 5
- [ ] Free time detection working
- [ ] Matching algorithm complete
- [ ] Suggestions appear in Google Calendar

### End of Week 6
- [ ] SMS sending to babysitter works
- [ ] Webhook receives YES/NO responses
- [ ] Calendar updates with confirmation

### End of Week 7
- [ ] Full flow tested end-to-end
- [ ] UI polished
- [ ] Demo video recorded
- [ ] MVP complete

---

## Next Steps

1. ✅ Read this plan
2. ⏭️ Start Phase 1: Set up Next.js project
3. ⏭️ Create detailed plan for current phase (optional)
4. ⏭️ Begin implementation
