# MVP Specification: Family Quality Time Activity Facilitator
## Demo-Ready Minimum Viable Product

**Version:** 1.0  
**Last Updated:** February 12, 2026  
**Author:** Dylan Mattern  
**Goal:** Working demo in 6-8 weeks

---

## MVP Philosophy

**Purpose:** Build the simplest version that demonstrates the core value proposition:
> "I tell you what I want to do → You find free time in my calendar → You suggest specific activities → You help me book a babysitter"

**Success = Demo-Ready:** Something you can show to 10 couples that actually works end-to-end, even if it's rough around the edges.

---

## Core User Journey (Demo Flow)

### The 5-Minute Demo Experience

1. **Sign up with Google** (30 seconds)
   - OAuth, connects calendar automatically
   
2. **Quick onboarding** (2 minutes)
   - Location: "Austin, TX"
   - Kids: "2 kids, ages 4 and 7"
   - Want to Try: Add 3-5 items ("Try kayaking," "Visit food trucks," "Go to museum")
   - Trusted sitters: Add mom's phone number

3. **See first suggestion** (30 seconds)
   - Calendar shows suggestion for Saturday
   - "Try: Kayaking at Lady Bird Lake - You wanted to try kayaking"
   - Click "I'm interested"

4. **Automatic babysitter request** (1 minute)
   - "Need a babysitter? Yes"
   - SMS sent to mom: "Dylan needs a sitter Saturday 1:30-5:30pm. Reply YES or NO"
   - Mom replies "YES"
   - Confirmation shown: "Your mom confirmed!"

5. **Activity on calendar** (30 seconds)
   - Event updated with all details
   - Shows babysitter confirmation
   - Ready to go!

**Total demo time:** 5 minutes from signup to confirmed activity with babysitter

---

## Technical Architecture (Simplified)

### Stack

**Frontend:**
- **Next.js** (React framework)
- **Tailwind CSS** (styling)
- **Vercel** (hosting - free tier)

**Backend:**
- **Next.js API Routes** (serverless functions)
- **Supabase** (PostgreSQL database + auth - free tier)
- **Vercel serverless functions** (no separate backend server needed)

**External Services:**
- **Google Calendar API** (calendar integration)
- **Twilio** (SMS - pay per message, ~$0.01 each)
- **Cheerio** (web scraping library for Node.js)

**Why this stack:**
- Everything can be in one Next.js project
- Free hosting on Vercel
- No server management
- Easy to deploy and show demos
- Single codebase

---

## Feature Set: What's In, What's Out

### ✅ IN: Demo Must-Haves

**1. Google Calendar Integration**
- OAuth login (use Google's identity)
- Read calendar to detect free time (basic: look for 3+ hour gaps)
- Write suggestions as calendar events
- That's it - no fancy sync, no webhooks

**2. Simple Onboarding**
- One page with 4 inputs:
  - Location (city, state)
  - Kids (ages - just a text field)
  - Want to Try list (textarea, one per line)
  - One trusted sitter (name, phone)
- Save to database
- Done in <2 minutes

**3. Activity Discovery via Web Scraping**
- **Target sites for scraping:**
  - Google Maps/Places (search results for "things to do in Austin")
  - Yelp (top-rated family activities)
  - Local event sites (Austin360, Do512, etc.)
  
- **Scrape structure:**
  - Activity name
  - Location/address
  - Rating
  - Brief description
  - Link to more info

- **Scraping strategy:**
  - Build scrapers for 2-3 sites
  - Run manually to populate database (not real-time)
  - Scrape ~50 activities per category for demo city (Austin)
  - Total: 200-300 activities for demo

- **Data refresh:**
  - Manual for MVP (you run the scraper when needed)
  - Store in database
  - No need for real-time updates for demo

**4. Basic Matching Algorithm**
Simple rule-based matching (no ML):

```javascript
function matchActivity(userWants, freeTime, activities) {
  // Score each activity
  activities.forEach(activity => {
    let score = 0;
    
    // Check if matches "Want to Try" list
    userWants.forEach(want => {
      if (activity.name.includes(want) || activity.category.includes(want)) {
        score += 50; // Direct match
      }
    });
    
    // Check if timing works
    if (activityFitsTimeSlot(activity, freeTime)) {
      score += 20;
    }
    
    // Check if in their city
    if (activity.city === user.city) {
      score += 10;
    }
    
    activity.score = score;
  });
  
  // Return highest scoring activity
  return activities.sort((a, b) => b.score - a.score)[0];
}
```

**5. Calendar Event Creation**
- Insert event into Google Calendar
- Format:
  - Title: "💡 Try: [Activity Name]"
  - Description: "You wanted to try [item from list]! [Activity details] [Link]"
  - Time: During free time block
  - Color: Blue (distinguishable)

**6. SMS Babysitter Coordination**
- User clicks "I'm interested" → prompt "Need babysitter?"
- If yes, send SMS via Twilio:
  ```
  Hi [Sitter Name]!
  Dylan needs a babysitter Saturday 2/22 from 1:30-5:30pm (4 hrs).
  Kids: 2 (ages 4, 7)
  Reply YES or NO
  ```
- Handle YES/NO responses (simple keyword matching)
- Update calendar event with confirmation
- Send confirmation to user

**7. Simple Dashboard**
One page showing:
- Next suggestion (if any)
- Babysitter request status
- "Want to Try" list (can add/remove)
- Trusted sitter info

### ❌ OUT: Not in Demo

**Cut these to ship faster:**
- Multiple calendars (just use primary calendar)
- Both partners' preferences (just one user for demo)
- Interest rating cards (just Want to Try list)
- Budget preferences (all activities for demo)
- Activity completion tracking (nice to have)
- Feedback collection (too much)
- Multiple trusted sitters (just one for demo)
- Email notifications (SMS only)
- Sitter priority levels (just one sitter)
- Payment tracking (mention cash payment in SMS)
- User settings/preferences page (bake into onboarding)
- Activity history (not needed for demo)
- Suggestion acceptance rate analytics (you'll track manually)

---

## Database Schema (Minimal)

```sql
-- Users (simplified - use Supabase auth)
users
  - id (from Supabase auth)
  - google_calendar_id
  - google_access_token (encrypted)
  - google_refresh_token (encrypted)

-- Families (simplified)
families
  - id
  - user_id (FK)
  - location_city
  - location_state
  - kids_ages (text: "4, 7")

-- Want to Try Items
want_to_try
  - id
  - family_id (FK)
  - description (text)
  - created_at

-- Trusted Sitter (singular)
trusted_sitter
  - id
  - family_id (FK)
  - name
  - phone
  - relationship

-- Activities (scraped data)
activities
  - id
  - name
  - description
  - category
  - city
  - state
  - address
  - rating
  - source_url
  - scraped_at

-- Suggestions
suggestions
  - id
  - family_id (FK)
  - activity_id (FK)
  - suggested_datetime
  - calendar_event_id
  - status (pending/accepted/declined)
  - created_at

-- Babysitter Requests
babysitter_requests
  - id
  - suggestion_id (FK)
  - sitter_id (FK)
  - date
  - start_time
  - end_time
  - status (pending/accepted/declined)
  - sent_at
  - responded_at
```

---

## Implementation Plan: 6-8 Weeks

### Week 1-2: Foundation
**Goal:** Basic app with Google Calendar connection

- [ ] Set up Next.js project
- [ ] Set up Supabase (database + auth)
- [ ] Implement Google OAuth
- [ ] Connect Google Calendar API (read access)
- [ ] Basic UI: Home page + onboarding flow
- [ ] Save user data to database

**Deliverable:** User can sign up and connect calendar

---

### Week 3-4: Activity Discovery
**Goal:** Scrape activities and show them

- [ ] Build web scrapers (Cheerio/Puppeteer)
  - Google Maps scraper (search "things to do in Austin")
  - Yelp scraper (family activities)
  - Local site scraper (Austin360 or Do512)
- [ ] Run scrapers to populate ~200 activities
- [ ] Store activities in database
- [ ] Build simple admin interface to view scraped activities
- [ ] Test data quality

**Deliverable:** Database has 200+ Austin activities

**Scraping approach:**
```javascript
// Example: Scrape Google Maps search results
const axios = require('axios');
const cheerio = require('cheerio');

async function scrapeGoogleMaps(query, city) {
  const searchUrl = `https://www.google.com/maps/search/${query}+${city}`;
  const { data } = await axios.get(searchUrl);
  const $ = cheerio.load(data);
  
  const activities = [];
  $('.section-result').each((i, element) => {
    activities.push({
      name: $(element).find('.section-result-title').text(),
      address: $(element).find('.section-result-location').text(),
      rating: $(element).find('.section-result-rating').text(),
      category: query
    });
  });
  
  return activities;
}

// Run for different categories
scrapeGoogleMaps('family restaurants', 'Austin TX');
scrapeGoogleMaps('outdoor activities', 'Austin TX');
scrapeGoogleMaps('museums', 'Austin TX');
```

---

### Week 5: Free Time Detection + Matching
**Goal:** Generate personalized suggestions

- [ ] Build free time detection algorithm
  - Query calendar for next 2 weeks
  - Find gaps of 3+ hours
  - Focus on weekends for demo
- [ ] Build matching algorithm (rule-based)
  - Match Want to Try list with activity names/categories
  - Score activities
  - Pick best match
- [ ] Create calendar event with suggestion
- [ ] Test with your own calendar

**Deliverable:** App can detect free time and suggest activities

---

### Week 6: Babysitter SMS Integration
**Goal:** Automatic babysitter coordination

- [ ] Set up Twilio account
- [ ] Build SMS sending function
- [ ] Build SMS webhook receiver (for YES/NO responses)
- [ ] Implement request flow:
  - User accepts suggestion → prompt for babysitter
  - Send SMS to trusted sitter
  - Parse response
  - Update calendar event
  - Notify user
- [ ] Test with real phone number

**Deliverable:** End-to-end babysitter coordination works

---

### Week 7: Polish + Testing
**Goal:** Demo-ready UI and bug fixes

- [ ] Clean up UI (Tailwind styling)
- [ ] Add loading states
- [ ] Error handling (what if calendar API fails?)
- [ ] Test full flow multiple times
- [ ] Fix bugs
- [ ] Deploy to Vercel

**Deliverable:** Stable demo environment

---

### Week 8: Demo Prep + First Users
**Goal:** Ready to show real people

- [ ] Write demo script
- [ ] Test with 3-5 friends/family
- [ ] Gather feedback
- [ ] Quick iterations
- [ ] Prepare for demo day

**Deliverable:** Validated with real users

---

## Web Scraping Strategy (Detailed)

### Target Sites for Austin Demo

**1. Google Maps/Places**
- Search queries:
  - "family activities Austin"
  - "outdoor activities Austin"
  - "restaurants Austin"
  - "museums Austin"
  - "parks Austin"
- Scrape: Name, rating, address, category
- **Caution:** Google may block scrapers - use rate limiting, rotate user agents

**2. Yelp**
- Categories to scrape:
  - Arts & Entertainment
  - Active Life
  - Restaurants (family-friendly)
  - Parks
- Scrape: Name, rating, price range, description, link
- **Note:** Yelp has Fusion API (free tier) - might be easier than scraping

**3. Local Austin Sites**
- **Do512.com** (Austin events calendar)
  - Scrape: Family events, outdoor activities
  - Usually easier to scrape (less protection)
- **Austin360.com** (local news/events)
  - Things to do section
- **Austin Parks Foundation**
  - Park listings and activities

### Scraper Implementation Tips

**Tools:**
- **Cheerio:** Lightweight, fast, good for static HTML
- **Puppeteer:** For JavaScript-heavy sites (Google Maps)
- **Playwright:** Alternative to Puppeteer, faster

**Best practices:**
```javascript
// Add delays between requests
await sleep(2000); // 2 seconds

// Rotate user agents
const userAgents = [
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...',
  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...'
];

// Handle errors gracefully
try {
  const data = await scrapeWebsite(url);
} catch (error) {
  console.log(`Failed to scrape ${url}: ${error.message}`);
  return []; // Return empty, don't crash
}

// Save raw HTML for debugging
fs.writeFileSync('debug.html', htmlContent);
```

**Legal considerations:**
- Respect robots.txt
- Don't overwhelm servers (rate limiting)
- For demo purposes, small-scale scraping is generally okay
- Consider using official APIs when available (Yelp Fusion API is free)

### Alternative: Hybrid Approach

If scraping is too fragile:
- **Yelp Fusion API:** 5,000 calls/day free - use this for restaurants/venues
- **Eventbrite API:** Free tier for public events
- **Manual curation:** Scrape 50 activities manually as fallback
- **Combination:** Use APIs where available, scrape where not

**MVP recommendation:** Start with Yelp API (easier) + manual backup for demo

---

## API Endpoints (Simplified)

```
POST   /api/auth/google          - Google OAuth callback
GET    /api/auth/me              - Get current user

POST   /api/onboarding           - Save onboarding data

GET    /api/calendar/free-time   - Get free time blocks
POST   /api/suggestions/generate - Generate suggestion for free time
GET    /api/suggestions          - Get current suggestions

POST   /api/sitter/request       - Send babysitter SMS
POST   /api/webhooks/sms         - Twilio SMS webhook (incoming responses)

GET    /api/activities           - Get activities (filtered by city)
POST   /api/admin/scrape         - Trigger scraper (manual for MVP)
```

---

## Demo Script (5 Minutes)

### Setup (before demo):
- Pre-populate database with 200 Austin activities
- Have a phone ready to receive SMS
- Clear calendar with at least one 3-hour free block on Saturday

### Live demo:

**Minute 1: "The Problem"**
- "Couples want to spend quality time together but never follow through"
- "Planning is hard, babysitting is harder"
- "We automate both"

**Minute 2: "Onboarding"**
- Sign up with Google (OAuth)
- Enter: Austin, 2 kids (4, 7)
- Want to Try: "kayaking, food trucks, museum"
- Trusted sitter: "Mom - [phone number]"

**Minute 3: "The Magic"**
- App scans calendar
- Shows free Saturday afternoon
- Suggests: "Try: Kayaking at Lady Bird Lake - You wanted to try kayaking!"
- Click "I'm interested"

**Minute 4: "Babysitter Automation"**
- Prompt: "Need a babysitter?"
- Click "Yes"
- SMS sent to mom (show on screen)
- Mom replies "YES" (have someone ready)
- Instant confirmation: "Your mom confirmed for Saturday 1:30-5:30pm!"

**Minute 5: "The Result"**
- Calendar now shows full event with:
  - Activity details
  - Babysitter confirmed
  - Time and location
  - Link to activity
- "From 'we should do something' to 'it's happening' in 5 minutes"

**Questions:** Answer any questions

---

## Success Metrics for Demo

**Technical success:**
- [ ] User can sign up and connect Google Calendar
- [ ] Free time is detected correctly
- [ ] Activity suggestion appears on calendar
- [ ] SMS is sent and received
- [ ] YES/NO responses are parsed correctly
- [ ] Calendar event updates with babysitter confirmation
- [ ] Zero crashes during demo

**User success (test with 5-10 couples):**
- [ ] 80%+ complete onboarding (not too confusing)
- [ ] Suggestion feels relevant (matches Want to Try list)
- [ ] SMS message is clear to babysitters
- [ ] At least 1 couple actually does the suggested activity
- [ ] Positive feedback: "I would use this"

**Kill signals:**
- Suggestions feel random/irrelevant
- Babysitters confused by SMS
- Too many technical failures
- Users say "too complicated" or "I'll just text my mom myself"

---

## Cost Estimate (MVP)

**Monthly costs:**
- **Hosting:** $0 (Vercel free tier)
- **Database:** $0 (Supabase free tier: 500MB, 2GB bandwidth)
- **Google Calendar API:** $0 (free up to quota)
- **Twilio SMS:** ~$0.01 per message
  - 10 users × 4 requests/month = 40 messages = $0.40/month
- **Domain (optional):** $12/year = $1/month

**Total: ~$1-2/month for demo**

**One-time costs:**
- Development time: 6-8 weeks (your time)
- No other costs for MVP

---

## Technical Risks & Mitigations

### Risk 1: Web Scraping Breaks
**Problem:** Sites change HTML structure, scrapers stop working

**Mitigation:**
- Use multiple sources (if one breaks, others still work)
- Have 100 manually curated activities as backup
- For demo, pre-scrape and store - don't scrape in real-time
- Consider Yelp API as fallback (more stable)

### Risk 2: Google Calendar API Quota
**Problem:** Hit rate limits with too many users

**Mitigation:**
- Free tier: 1M queries/day (way more than needed for demo)
- Implement caching (don't re-fetch calendar every page load)
- For MVP, this is unlikely to be an issue

### Risk 3: SMS Delivery Issues
**Problem:** Twilio messages don't arrive or are delayed

**Mitigation:**
- Test with multiple phone numbers before demo
- Have manual fallback ("Copy this message and text your sitter")
- Use verified numbers in Twilio (required for free trial)

### Risk 4: Google OAuth Complexity
**Problem:** OAuth flow is complicated, bugs are hard to debug

**Mitigation:**
- Use Next-Auth library (simplifies OAuth)
- Test extensively before demo
- Have detailed error messages
- Google OAuth is well-documented - follow examples

### Risk 5: Suggestion Quality
**Problem:** Suggested activities don't match user's Want to Try list

**Mitigation:**
- Test matching algorithm with real data
- Start with exact string matching (simpler)
- Manually verify top 20 activities make sense
- For demo, pre-select 10 great activities that always score high

---

## Post-Demo: What's Next?

After successful demo with 10 couples, consider:

**Quick wins (1-2 weeks each):**
1. Support multiple trusted sitters
2. Better matching algorithm (fuzzy string matching)
3. Email notifications (in addition to SMS)
4. Activity completion feedback
5. Weekly "you're free this weekend" reminders

**Medium-term (1-2 months):**
1. Expand to 2-3 more cities
2. Replace scraping with proper APIs (Yelp Fusion, Google Places)
3. Support both partners (separate onboarding)
4. Interest categories (not just Want to Try list)
5. Mobile-responsive polish

**Long-term (3-6 months):**
1. Real-time calendar sync (webhooks)
2. Machine learning recommendations
3. Payment integration for babysitters
4. Native mobile apps
5. Monetization (affiliates)

---

## Development Checklist

### Prerequisites
- [ ] Node.js installed
- [ ] Git repository set up
- [ ] Google Cloud Console project (for Calendar API)
- [ ] Twilio account created
- [ ] Supabase project created
- [ ] Vercel account created

### Phase 1: Setup (Week 1)
- [ ] `npx create-next-app@latest family-time-app`
- [ ] Install dependencies: `npm install @supabase/supabase-js cheerio axios twilio`
- [ ] Set up Supabase connection
- [ ] Configure Google OAuth
- [ ] Create basic home page

### Phase 2: Core Features (Weeks 2-6)
- [ ] Implement onboarding flow
- [ ] Build web scrapers
- [ ] Populate activities database
- [ ] Build free time detection
- [ ] Build matching algorithm
- [ ] Implement calendar event creation
- [ ] Build SMS integration
- [ ] Test end-to-end flow

### Phase 3: Polish (Week 7)
- [ ] UI styling (Tailwind)
- [ ] Error handling
- [ ] Loading states
- [ ] Mobile responsive
- [ ] Deploy to Vercel

### Phase 4: Testing (Week 8)
- [ ] Test with 5 users
- [ ] Fix critical bugs
- [ ] Gather feedback
- [ ] Prepare demo script

---

## Environment Variables (.env.local)

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Google Calendar API
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:3000/api/auth/callback

# Twilio
TWILIO_ACCOUNT_SID=your-account-sid
TWILIO_AUTH_TOKEN=your-auth-token
TWILIO_PHONE_NUMBER=your-twilio-phone

# App
NEXTAUTH_SECRET=your-nextauth-secret
NEXTAUTH_URL=http://localhost:3000
```

---

## Demo Day Checklist

### 24 Hours Before:
- [ ] Deploy latest code to Vercel
- [ ] Test full flow on production URL
- [ ] Scrape fresh activity data
- [ ] Clear test data from database
- [ ] Prepare phone for receiving SMS
- [ ] Test SMS with all demo phone numbers
- [ ] Create Google Calendar with free time blocks
- [ ] Charge laptop (for live demo)

### 1 Hour Before:
- [ ] Test internet connection
- [ ] Open all necessary tabs
- [ ] Clear browser cache
- [ ] Sign out of any existing sessions
- [ ] Have backup demo video ready (in case of technical failure)
- [ ] Phone on, SMS notifications on

### During Demo:
- [ ] Speak clearly and slowly
- [ ] Show the problem first (why this matters)
- [ ] Walk through each step
- [ ] Show SMS on phone
- [ ] Show final calendar event
- [ ] Ask for questions
- [ ] Get feedback after

---

## Quick Start Commands

```bash
# Create project
npx create-next-app@latest family-time-app --typescript --tailwind --app

# Install dependencies
npm install @supabase/supabase-js @supabase/auth-helpers-nextjs
npm install next-auth
npm install twilio
npm install cheerio axios
npm install date-fns

# Run development server
npm run dev

# Run scraper (manual)
node scripts/scrape-activities.js

# Deploy to Vercel
vercel --prod
```

---

## Resources & References

**Documentation:**
- [Next.js Docs](https://nextjs.org/docs)
- [Supabase Docs](https://supabase.com/docs)
- [Google Calendar API](https://developers.google.com/calendar/api/guides/overview)
- [Twilio SMS](https://www.twilio.com/docs/sms)
- [Cheerio](https://cheerio.js.org/)

**Tutorials:**
- [Next.js + Google OAuth](https://next-auth.js.org/providers/google)
- [Web Scraping with Cheerio](https://www.freecodecamp.org/news/web-scraping-in-nodejs/)
- [Twilio SMS in Node.js](https://www.twilio.com/docs/sms/quickstart/node)

**Example code:**
- [Next.js + Supabase template](https://github.com/vercel/next.js/tree/canary/examples/with-supabase)
- [Google Calendar API examples](https://github.com/googleworkspace/node-samples)

---

## Conclusion

This MVP is designed to be **achievable in 6-8 weeks for a solo developer** while still demonstrating the core value proposition. By using web scraping instead of manual curation, you can populate the activity database quickly. By keeping the feature set minimal, you can ship a working demo fast.

**Remember:** The goal is to validate the idea, not build a production app. Get it working, show it to people, gather feedback, then iterate.

**Next step:** Set up the Next.js project and start with Google Calendar integration (Week 1).

Good luck! 🚀
