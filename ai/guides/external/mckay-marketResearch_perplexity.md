# Market Research: Calendar-Integrated Activity Suggestions for Couples/Families

This is potentially viable as a **niche product**, but only if you narrow the scope, validate in one market, and treat it as a high‑intent tool rather than a mass consumer app.[cite:3][cite:9]

---

## 1. Market Demand and Behavior

- People are actively seeking more local, in-person experiences (workshops, classes, interest-based events), and this has been growing.[cite:3][cite:6]
- Ticketing platforms like Eventbrite are already meaningful discovery channels:
  - In one survey, about a **quarter of attendees** named ticketing platforms as the most useful place to find event recommendations.
  - Roughly **one-third specifically pointed to Eventbrite**.[cite:9]
- Consumers want **local, niche, community-driven activities** (game nights, crafting classes, workshops) rather than just generic “things to do,” which aligns with your idea of surfacing specific, interest-aligned activities.[cite:3]

**Signal:** There is demonstrated appetite for **curated local experiences**; discovery is a real job-to-be-done, not just a made-up problem.[cite:3][cite:9]

---

## 2. Competitive Landscape

- **Facebook/Meta**:
  - Strong local discovery features (Local tab, event discovery based on interests, friends, and location).[cite:1][cite:10]
  - Surfaces “Today,” “This week,” “This weekend” events with category filters.
- **Behavior today**:
  - Users commonly search “things to do near me” or browse Google Maps for ad-hoc date ideas.[cite:5]
- Gaps:
  - Existing tools require users to **proactively browse/search**.
  - They are **not designed around**:
    1. Explicit couple/family preferences,
    2. Both partners’ calendars,
    3. Pushing suggestions directly into Google Calendar.

**Signal:** The space is competitive on **discovery**, but no mainstream product tightly integrates **(a) explicit couple preferences, (b) both calendars, and (c) calendar-native push suggestions.**

---

## 3. Who This Is Most Viable For

Your broad target (“dual-income couples with kids”) is large, but the practical early beachhead is narrower:

- **Location:** One metro with strong local events/activity density (cities where small businesses are actively using platforms like Eventbrite for workshops/classes).[cite:3][cite:6]
- **Mindset:** High-intent couples who:
  - Already spend on experiences,
  - See going out as part of their identity,
  - Already attend local workshops, classes, or events.
- **Use case:** “We want **1–2 good suggestions per weekend** aligned with our shared interests” rather than “solve our entire family logistics problem.”

**Positioning suggestion:** “A date/experience concierge for busy couples in [City]” rather than a generic “family calendar tool.”

---

## 4. Key Risks That Affect Viability

### 4.1 Discovery Quality and Coverage

- Event data is fragmented:
  - Platforms like Eventbrite are strong in **workshops, classes, and small business events**, but coverage varies by city.[cite:3][cite:9]
- Many people still discover events via:
  - Word of mouth,
  - Social platforms,
  - Local guides/newsletters.[cite:6]

**Mitigation:**

- Start with activity types that have **good structured data**:
  - Workshops, classes, ticketed events (Eventbrite-style),
  - Evergreen activities (restaurants, attractions) from Google Places/Yelp.
- Choose **one metro** where:
  - There is visible density of local workshops/classes,
  - Businesses already use online platforms to promote events.[cite:3][cite:9]

---

### 4.2 “Calendar Lies” and Scheduling Complexity

- Assumption: _blank calendar = usable free time_ is often wrong, especially for parents.
  - Domestic work, childcare, fatigue, logistics often **aren’t on the calendar**.
- Risk: If suggestions consistently land in impractical times, users will:
  - Ignore events,
  - Stop trusting the product,
  - Eventually uninstall.

**Mitigation:**

- During onboarding, collect explicit availability preferences:
  - Days and time windows (e.g., Fri night, Sat afternoon, Sun evening).
  - Minimum duration (e.g., 2–3 hours).
  - Required lead time (e.g., ≥3 days ahead).
- For MVP, **limit to weekend date windows**:
  - Friday evening–Sunday evening,
  - Higher likelihood that time is truly flexible.

---

### 4.3 Adoption and Activation

- Getting **both partners** through:
  - Signup,
  - Google OAuth,
  - Interests survey  
    is a non-trivial activation barrier.
- For broad consumer scale, this is a serious drop‑off risk.
- For a **niche, high-intent** audience, heavier onboarding can be acceptable and can even:
  - Signal seriousness,
  - Create perceived value (a “date planning session”).

**Mitigation:**

- Treat early versions as **invite-only concierge** in one city:
  - Expect **low volume, high intent**.
  - Make onboarding feel like part of the experience:
    - “We’ll design your perfect date profile” vs. “fill out this form.”

---

## 5. Viable Product Shape for MVP

A more viable shape, given the market and competition:

### 5.1 Segment

- Couples (with or without kids) in **one selected city** who:
  - Already enjoy local experiences,
  - Are willing to try a “set it and forget it” suggestion engine,
  - Are comfortable connecting calendars for convenience.

### 5.2 Scope

- **Timebound:**
  - Weekends only (Friday evening–Sunday night).
- **Volume:**
  - 1–2 suggestions per week per couple (avoid spam).
- **Content focus:**
  - Categories with strong data:
    - Classes/workshops (e.g., cooking, pottery, dance),
    - Tastings, shows, small venue events,
    - “Places you said you wanted to try” (restaurants, cafes, museums).[cite:3][cite:9]

### 5.3 Differentiator

- Uses **both partners’ calendars** to propose options only when:
  - They’re both free,
  - Within their preferred windows.
- Delivers suggestions **directly into Google Calendar** as tentative events:
  - No separate app to check,
  - Leverages existing planning behavior.
- Learns from **accept/decline** behavior over time:
  - Time-of-day preferences,
  - Activity types they actually do vs. ignore.

In this form, you’re not trying to beat Facebook/Google at broad discovery. You’re solving:

> “Give us one well-timed, high-fit idea this weekend without us having to search or coordinate manually.”

---

## 6. Go / No-Go Guidance

### 6.1 As a Product Concept

- **Not viable** (as currently framed) as:
  - A mass-market, generalized “family calendar + local discovery” app,
  - A direct competitor to Meta/Google local discovery tools.
- **Potentially viable** as:
  - A focused, city-specific **“calendar-native date concierge”** for experience-seeking couples,
  - Especially if:
    - You validate via **manual “wizard behind the curtain” pilots**, and
    - You keep the scope very narrow initially.[cite:3][cite:9]

### 6.2 As a Class Project

- Very strong fit:
  - Teaches OAuth, Google Calendar API, external event APIs,
  - Involves multi-user coordination (two calendars, shared preferences),
  - Clear narrative for demos:
    - “Here’s the couple profile,
    - Here’s their free time window,
    - Here’s the suggestion in their calendar,
    - Here’s what they did with it.”

---

## 7. Recommended Next Steps (Validation-Focused)

Before treating this as a real startup idea, run **lean validation**:

1. **Manual Pilot (Wizard of Oz)**
   - Recruit 5–10 couples in a single city.
   - Have them:
     - Share read-only calendar access,
     - Fill out a simple interests + availability form.
   - Each week:
     - You manually inspect their next 1–2 weekends,
     - Pick 1–2 high-quality suggestions (from Eventbrite, Google Maps, local guides),
     - Manually create tentative calendar events for them.
   - Measure:
     - % of suggestions accepted,
     - % of accepted suggestions actually done,
     - Qualitative feedback: “Was this useful? What would you change?”

2. **Decision Criteria**
   - If **2+ out of 5 couples** consistently:
     - Accept suggestions,
     - Actually do the activities,
     - Say they’d miss the service if you stopped,
     - → Strong signal to build a lightweight automated MVP.
   - If most:
     - Ignore suggestions,
     - Report that timing/activities don’t fit their reality,
     - Prefer to just search themselves,
     - → Good signal to pivot/kill or rethink.

---

## 8. Bottom Line

- There **is** real demand around local experience discovery and execution, and established platforms (like Eventbrite) show that people use digital tools to find and book local experiences.[cite:3][cite:9]
- No major player currently combines:
  - Shared couple preferences,
  - Both calendars,
  - Direct, calendar-native suggestions.
- Viability hinges on:
  - Narrowing the scope (one city, weekends, a few activity types),
  - Treating this as a **high-intent, concierge-like tool** rather than a low-friction viral app,
  - Validating behavior through **small manual pilots** before heavy development.

As a **class project**, it’s excellent and realistic.  
As a **startup**, it is _conditionally viable_ if you can validate that couples will (a) connect calendars, (b) accept suggestions at a meaningful rate, and (c) see the experience as better than just searching when they feel like going out.
