# Eventbrite API - DEPRECATED / NOT VIABLE

**Status:** ❌ **DOES NOT WORK FOR OUR USE CASE**
**Last Updated:** 2026-02-12
**Issue:** Public Event Search API was deprecated in 2020, API is largely unsupported as of 2025

---

## Critical Finding

**The Eventbrite public Event Search API no longer exists.**

From official sources:
> "In December 2019, Eventbrite announced they were removing public access to the Event Search API (GET /v3/events/search/), and after February 20, 2020, all requests to the Event Search API were to be denied."

**As of April 2025:**
> "The Eventbrite API is now deprecated and unsupported by Eventbrite, with their support team stating they are no longer supporting the API at all."

---

## What We Planned vs. What Actually Exists

### What We Wanted
- Search for local events by location, date, and category
- Filter for workshops, classes, tastings, etc.
- Get event details (name, time, location, description)

### What's Actually Available
**Only organization-specific or venue-specific endpoints:**

1. **GET /v3/events/:event_id/** - Retrieve a single event by ID
   - ❌ Requires knowing the event ID already (useless for discovery)

2. **GET /v3/organizations/:organization_id/events/** - List events by organization
   - ❌ Requires knowing the organization ID first
   - ❌ Only shows events from ONE organization at a time

3. **GET /v3/venues/:venue_id/events/** - List events by venue
   - ❌ Requires knowing the venue ID first
   - ❌ Only shows events from ONE venue at a time

**Bottom line:** You can't search for "pottery classes in Austin this weekend" anymore. The API is effectively useless for our use case.

---

## Why This Matters for Our MVP

Our original plan assumed we could:
1. Query Eventbrite for events matching user interests
2. Filter by location and date
3. Get a list of relevant activities

**This is no longer possible with Eventbrite API.**

---

## Alternative Options

### Option 1: Hardcoded Activity Database (RECOMMENDED for MVP)
**Status:** ✅ **Easiest, most reliable**

- Create `activities.json` with 20-30 curated local activities
- Manually populate with real businesses (pottery studios, restaurants, etc.)
- No API dependency, no rate limits, no auth needed
- Perfect for demo/MVP

**Pros:**
- ✅ Works immediately
- ✅ No API keys needed
- ✅ No rate limits
- ✅ Reliable for demo
- ✅ Can add quality descriptions

**Cons:**
- ❌ Manual maintenance
- ❌ Limited to one city
- ❌ Not "real-time" event data
- ❌ Can't scale without automation

**For class project MVP:** This is the right choice.

---

### Option 2: Web Scraping (Complex, Risky)
**Status:** ⚠️ **Possible but not recommended**

Use tools like Puppeteer/Cheerio to scrape Eventbrite's public website.

**Pros:**
- ✅ Access to real event data
- ✅ No official API needed

**Cons:**
- ❌ Against Eventbrite's Terms of Service
- ❌ Brittle (breaks when HTML changes)
- ❌ Rate limiting / IP bans possible
- ❌ Ethical gray area
- ❌ Takes significant dev time

**Verdict:** Not worth it for MVP

---

### Option 3: Third-Party Event Aggregation APIs
**Status:** ⚠️ **Paid services, mixed quality**

Several services aggregate event data from multiple sources:

#### PredictHQ
- **URL:** [predicthq.com](https://www.predicthq.com/compare/eventbrite-vs-predicthq)
- **What it is:** Enterprise event data API
- **Coverage:** "Hundreds of sources"
- **Cost:** Paid (likely expensive)
- **Pros:** Comprehensive, reliable
- **Cons:** Likely requires enterprise contract, overkill for MVP

#### SerpApi / SearchAPI
- **URL:** [serpapi.com/google-events-api](https://serpapi.com/google-events-api)
- **What it is:** Scrapes Google Events search results
- **Coverage:** Whatever Google shows for "events near [location]"
- **Cost:** Paid (usage-based)
- **Pros:** Accesses Google's event aggregation
- **Cons:** Costs money, still a scraping service

#### Rakuten Events API (aggregator)
- **What it is:** Aggregates from ~70 event sources
- **Coverage:** Multiple platforms
- **Cost:** Unknown (likely paid)

**Verdict:** All are paid services, overkill for MVP

---

### Option 4: Google Places API
**Status:** ✅ **EXISTS but doesn't do events**

Google Places API can find:
- Restaurants, cafes, museums, parks (places)
- Business details (hours, ratings, photos)
- NOT events (no time-based activities)

**Use case for us:**
- ✅ Good for "restaurants you wanted to try"
- ✅ Good for "museums" or "coffee shops"
- ❌ NOT good for "pottery class this Saturday" or "wine tasting event"

**Verdict:** Useful for static venues, not for events

**Documentation:** [developers.google.com/maps/documentation/places](https://developers.google.com/maps/documentation/places/web-service/overview)

---

### Option 5: Meetup API
**Status:** ⚠️ **Restricted, requires approval**

Meetup has an API, but:
- Requires business account
- Approval process for API access
- Focused on Meetup groups, not general events

**Verdict:** Too niche, not worth the effort

---

## Recommendation for Our MVP

**Use Option 1: Hardcoded Activity Database**

### Why?
1. **Speed:** Can create it in 1-2 hours
2. **Reliability:** No API dependencies
3. **Demo quality:** Curated, high-quality suggestions
4. **Learning focus:** Spend time on Google Calendar integration (the hard part), not fighting deprecated APIs

### Implementation
Create `data/activities.json`:
```json
[
  {
    "id": "act-1",
    "name": "Beginner Pottery Workshop",
    "category": "classes",
    "description": "Learn hand-building and wheel throwing techniques",
    "location": {
      "address": "Clay Space Studio, 456 Art St, Austin, TX",
      "lat": 30.2650,
      "lng": -97.7400
    },
    "duration": 180,
    "price": "$$",
    "keywords": ["pottery", "ceramics", "art", "class", "hands-on", "crafts"],
    "url": "https://www.clayspacestudio.com/classes"
  }
  // ... 19-29 more activities
]
```

### Activity Categories to Include
- **Classes/Workshops:** Pottery, cooking, painting, dance, photography
- **Restaurants:** Italian, sushi, BBQ, farm-to-table (from "want to try" lists)
- **Experiences:** Wine tasting, brewery tours, escape rooms
- **Arts:** Museums, galleries, theater shows
- **Outdoors:** Hiking trails, parks, botanical gardens
- **Music:** Small venue concerts, open mic nights

### How to Populate
1. Google "pottery classes in Austin" → find real businesses
2. Add 3-5 options per category
3. Use real addresses, real URLs
4. Write compelling descriptions

**Time investment:** 2-3 hours to create a solid database

---

## Future: If We Want Real Event Data Later

**Post-MVP options:**
1. Partner directly with local venues (manual curation)
2. Hire a VA to update activities weekly
3. Build web scraper (if comfortable with gray area)
4. Pay for PredictHQ or similar (if this becomes a real product)

**For class project:** Hardcoded data is perfectly acceptable and preferred.

---

## Sources

- [Eventbrite v3 Search API is deprecated. Turning off Feb 20, 2020 · Issue #83](https://github.com/Automattic/eventbrite-api/issues/83)
- [Eventbrite API Depreciated - Make Community](https://community.make.com/t/eventbrite-api-depreciated/80369)
- [A better alternative to Eventbrite API for finding events - PredictHQ](https://www.predicthq.com/compare/eventbrite-vs-predicthq)
- [Top 10 Best Events APIs: EventBrite, Meetup, Ticketmaster, and others](https://blog.api.rakuten.net/top-10-best-events-apis/)
- [Google Events API - SerpApi](https://serpapi.com/google-events-api)
