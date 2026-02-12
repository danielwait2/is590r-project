# MVP: Chrome Extension Demo

**Version:** 1.0
**Goal:** Build a functional demo to show how the Family Quality Time App works
**Timeline:** 4-5 days
**Type:** Chrome Extension (Hybrid Approach - Real calendar integration + Curated suggestions)

---

## Purpose

This MVP is **not** a production-ready product. It's a **proof-of-concept demo** to:

1. **Show the core value proposition**: "I can see your free time and suggest what to do"
2. **Validate the technical feasibility**: Can we actually read Google Calendar and write back to it?
3. **Tell a compelling story**: Use for pitches, user testing, or investor demos
4. **Learn quickly**: Get feedback on the UX before building the full product

---

## MVP Scope

### What It Does

**Core Workflow:**
```
1. User installs Chrome extension
2. User grants Google Calendar access (OAuth)
3. User opens extension popup
4. Extension shows:
   - Next free time slot detected (e.g., "Saturday 10am-2pm")
   - Simple form to add "Things I want to try" (3-5 items)
5. User clicks "Get Suggestions"
6. Extension shows 2 curated suggestions that match the free time
7. User clicks "Add to Calendar" on a suggestion
8. Event is added to their Google Calendar
9. Success! They can see it in their calendar immediately
```

**Visual Flow:**
```
┌─────────────────────┐
│  Install Extension  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│  Grant Calendar     │
│  Access (OAuth)     │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│  Extension Popup    │
│  Opens              │
│                     │
│  Shows:             │
│  • Free time        │
│    detected         │
│  • "Want to try"    │
│    form             │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│  Click "Get         │
│  Suggestions"       │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│  Shows 2            │
│  Suggestions        │
│                     │
│  [Children's        │
│   Museum]           │
│  [Lunch at          │
│   Farm Cafe]        │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│  Click "Add to      │
│  Calendar"          │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│  Event Added!       │
│  ✓ Success message  │
└─────────────────────┘
```

### What It Doesn't Do (Out of Scope)

- ❌ No backend server (everything runs client-side)
- ❌ No user accounts or database
- ❌ No ML/AI recommendations (suggestions are curated/hard-coded)
- ❌ No multi-calendar support (just one calendar)
- ❌ No family profile setup (demo assumes 2 kids, ages 3 and 7)
- ❌ No weekly automation (user manually opens extension)
- ❌ No email/SMS notifications
- ❌ No feedback collection or learning

---

## Technical Architecture

### Tech Stack

**Chrome Extension:**
- **Manifest V3** (latest Chrome extension format)
- **HTML/CSS/JavaScript** (vanilla JS, no frameworks for simplicity)
- **Google Calendar API** (OAuth + read/write access)

**No Backend:** Everything runs in the browser

**Storage:**
- Chrome local storage (for "want to try" items and OAuth tokens)
- No database

### File Structure

```
family-time-extension/
├── manifest.json          # Extension config
├── popup.html             # Extension popup UI
├── popup.js               # Popup logic
├── popup.css              # Popup styles
├── background.js          # Background service worker (OAuth handling)
├── calendar.js            # Google Calendar API wrapper
├── suggestions.js         # Curated suggestions logic
├── icons/                 # Extension icons
│   ├── icon16.png
│   ├── icon48.png
│   └── icon128.png
└── README.md              # Installation instructions
```

### Google Calendar API Integration

**Permissions needed in manifest.json:**
```json
{
  "permissions": [
    "identity",
    "storage"
  ],
  "host_permissions": [
    "https://www.googleapis.com/calendar/*"
  ],
  "oauth2": {
    "client_id": "YOUR_CLIENT_ID.apps.googleusercontent.com",
    "scopes": [
      "https://www.googleapis.com/auth/calendar.readonly",
      "https://www.googleapis.com/auth/calendar.events"
    ]
  }
}
```

**OAuth Flow:**
1. User clicks "Connect Calendar" button
2. `chrome.identity.getAuthToken()` triggers OAuth consent
3. Extension receives access token
4. Store token in `chrome.storage.local`
5. Use token for all subsequent Calendar API calls

**API Calls:**
- `GET /calendars/primary/events` - Read calendar events
- `POST /calendars/primary/events` - Create new event

---

## Features Breakdown

### Feature 1: Detect Free Time

**What it does:**
- Reads user's Google Calendar for next 7 days
- Finds blocks of 2-4 hours with no events
- Focuses on weekends (Saturday/Sunday 9am-6pm)
- Displays first free block found

**Implementation:**
```javascript
// Pseudo-code
async function detectFreeTime() {
  const now = new Date();
  const nextWeek = new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000);

  const events = await getCalendarEvents(now, nextWeek);
  const freeBlocks = findFreeBlocks(events);

  return freeBlocks[0]; // Return first free block
}

function findFreeBlocks(events) {
  // Logic:
  // 1. Generate all possible 2-4 hour blocks on Sat/Sun 9am-6pm
  // 2. Remove blocks that overlap with existing events
  // 3. Return remaining blocks
}
```

**UI Display:**
```html
<div class="free-time-card">
  <h3>🗓️ Next Free Time</h3>
  <p class="date">Saturday, Feb 15</p>
  <p class="time">10:00 AM - 2:00 PM</p>
  <p class="duration">4 hours available</p>
</div>
```

### Feature 2: "Want to Try" Input

**What it does:**
- Simple form to enter 3-5 things the family wants to try
- Stores in Chrome local storage
- Pre-populates with demo examples for first-time users

**Implementation:**
```javascript
// Save to Chrome storage
function saveWantToTry(items) {
  chrome.storage.local.set({ wantToTry: items });
}

// Load from Chrome storage
function loadWantToTry() {
  return chrome.storage.local.get('wantToTry');
}

// Default demo items
const defaultItems = [
  "Children's science museum",
  "New farm-to-table restaurant",
  "Outdoor playground",
  "Local farmers market",
  "Family-friendly art gallery"
];
```

**UI:**
```html
<div class="want-to-try-section">
  <h3>✨ Things We Want to Try</h3>
  <textarea
    placeholder="Enter things your family wants to do (one per line)"
    rows="5"
  ></textarea>
  <button id="save-list">Save List</button>
</div>
```

### Feature 3: Generate Suggestions

**What it does:**
- Takes free time slot + "want to try" items
- Matches against curated database of activities
- Returns 2 suggestions with details

**For Demo: Curated Suggestions Database**

Instead of calling Google Places API, use a hard-coded list of great activities:

```javascript
const curatedActivities = [
  {
    id: 1,
    title: "Children's Discovery Museum",
    type: "museum",
    description: "Interactive science and art exhibits for kids 2-10. Hands-on learning!",
    address: "123 Main St, Chicago, IL",
    imageUrl: "https://example.com/museum.jpg",
    estimatedCost: "$35 (family of 4)",
    duration: "2-3 hours",
    ageRange: "2-10",
    tags: ["science", "museum", "indoor", "educational"],
    bookingLink: "https://discoverymuseum.org"
  },
  {
    id: 2,
    title: "Farm Cafe Brunch",
    type: "restaurant",
    description: "Farm-to-table restaurant with outdoor seating and kids menu. Locally sourced ingredients.",
    address: "456 Oak Ave, Chicago, IL",
    imageUrl: "https://example.com/cafe.jpg",
    estimatedCost: "$60 (family of 4)",
    duration: "1-2 hours",
    ageRange: "all ages",
    tags: ["food", "restaurant", "outdoor", "farm-to-table"],
    bookingLink: "https://farmcafe.com/reservations"
  },
  // Add 8-10 more activities
];
```

**Matching Logic (Simple for Demo):**
```javascript
function generateSuggestions(freeTime, wantToTryList) {
  // Simple keyword matching
  const matches = curatedActivities.filter(activity => {
    return wantToTryList.some(item =>
      activity.tags.some(tag => item.toLowerCase().includes(tag))
    );
  });

  // If no matches, return top 2 general suggestions
  if (matches.length === 0) {
    return curatedActivities.slice(0, 2);
  }

  // Return top 2 matches
  return matches.slice(0, 2);
}
```

**UI Display:**
```html
<div class="suggestions-container">
  <h3>💡 Suggestions for Saturday 10am-2pm</h3>

  <div class="suggestion-card">
    <img src="museum.jpg" alt="Children's Museum">
    <h4>Children's Discovery Museum</h4>
    <p class="description">Interactive science and art exhibits...</p>
    <div class="details">
      <span class="cost">$35 (family of 4)</span>
      <span class="duration">2-3 hours</span>
    </div>
    <p class="address">📍 123 Main St, Chicago, IL</p>
    <button class="add-to-calendar" data-activity-id="1">
      Add to Calendar
    </button>
  </div>

  <!-- Repeat for suggestion 2 -->
</div>
```

### Feature 4: Add to Calendar

**What it does:**
- Creates a new Google Calendar event
- Includes: title, time, location, description, booking link
- Shows success message
- User can immediately see it in their Google Calendar

**Implementation:**
```javascript
async function addToCalendar(activity, freeTimeSlot) {
  const event = {
    summary: activity.title,
    location: activity.address,
    description: `${activity.description}\n\nCost: ${activity.estimatedCost}\nBook: ${activity.bookingLink}`,
    start: {
      dateTime: freeTimeSlot.startTime, // e.g., "2026-02-15T10:00:00-06:00"
      timeZone: 'America/Chicago'
    },
    end: {
      dateTime: freeTimeSlot.endTime, // e.g., "2026-02-15T12:00:00-06:00"
      timeZone: 'America/Chicago'
    },
    reminders: {
      useDefault: false,
      overrides: [
        { method: 'popup', minutes: 60 }
      ]
    }
  };

  const response = await fetch(
    'https://www.googleapis.com/calendar/v3/calendars/primary/events',
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(event)
    }
  );

  return response.json();
}
```

**Success UI:**
```html
<div class="success-message">
  <h3>✅ Added to your calendar!</h3>
  <p>Children's Discovery Museum</p>
  <p>Saturday, Feb 15 • 10:00 AM - 12:00 PM</p>
  <button id="view-calendar">View in Google Calendar</button>
</div>
```

---

## UI/UX Design (Simple)

### Extension Popup Dimensions
- Width: 400px
- Height: 600px
- Scrollable if content overflows

### Color Scheme (Friendly & Family-Oriented)
- Primary: `#4CAF50` (Green - positive, growth)
- Secondary: `#FF9800` (Orange - energy, fun)
- Background: `#FFFFFF`
- Text: `#333333`
- Accent: `#E3F2FD` (Light blue for cards)

### Typography
- Font: System font stack (Segoe UI, Roboto, Arial)
- Headings: 18px, bold
- Body: 14px, regular
- Small text: 12px

### Key UI Components

**1. Connection Status Banner**
```html
<div class="status-banner connected">
  ✓ Connected to Google Calendar
</div>
```

**2. Free Time Card** (see Feature 1 above)

**3. Want to Try Form** (see Feature 2 above)

**4. Suggestion Cards** (see Feature 3 above)

**5. Empty State**
```html
<div class="empty-state">
  <p>No free time found in the next 7 days</p>
  <p>Check back later!</p>
</div>
```

---

## Demo Script

### Scenario: Investor Pitch or User Testing

**Setup:**
1. Pre-install extension on your Chrome
2. Already connected to your Google Calendar
3. Have a free block this Saturday (or create a fake one in your calendar for demo)
4. Pre-filled "want to try" list with: "science museum, farm restaurant, outdoor park"

**Demo Flow:**

**[Screen 1: Extension Icon]**
> "Here's our Chrome extension. It lives right where families already are—in their Google Calendar. Let me click it..."

**[Screen 2: Popup Opens]**
> "It's already connected to my calendar and detected that I have 4 free hours this Saturday morning."

**[Screen 3: Want to Try List]**
> "I've told it a few things my family wants to try—a science museum, a new restaurant, and an outdoor park."

**[Screen 4: Click "Get Suggestions"]**
> "Now watch this. I click 'Get Suggestions' and..."

**[Screen 5: Suggestions Appear]**
> "...it instantly gives me two specific, actionable ideas: The Children's Discovery Museum and Farm Cafe Brunch. Both match my interests, both fit my time slot, and both are local."

**[Screen 6: Click "Add to Calendar"]**
> "If I like one, I just click 'Add to Calendar' and..."

**[Screen 7: Success Message + Switch to Google Calendar Tab]**
> "...boom. It's in my calendar. That's it. No research, no planning, no decision fatigue. We just turned a vague desire into a concrete plan in 10 seconds."

**[Wrap up]**
> "This is the MVP demo. The real product will do this automatically every week, learn your preferences, and work across multiple platforms. But the magic is right here: free time + desires + local opportunities = quality family time."

---

## Development Plan

### Day 1: Setup & OAuth
- [ ] Create Chrome extension boilerplate (manifest.json, popup.html)
- [ ] Set up Google Cloud Project for Calendar API
- [ ] Implement OAuth flow (connect/disconnect)
- [ ] Test: Can we read calendar events?

### Day 2: Free Time Detection
- [ ] Write logic to fetch calendar events for next 7 days
- [ ] Implement free block detection algorithm
- [ ] Display first free block in UI
- [ ] Test: Does it correctly find free time?

### Day 3: Suggestions & UI
- [ ] Create curated activities database (10-15 activities)
- [ ] Implement simple matching logic
- [ ] Build suggestion card UI
- [ ] Test: Do suggestions look good?

### Day 4: Add to Calendar
- [ ] Implement Google Calendar event creation
- [ ] Handle success/error states
- [ ] Polish UI (styles, animations)
- [ ] Test: Does event appear in calendar?

### Day 5: Testing & Demo Prep
- [ ] End-to-end testing of full flow
- [ ] Fix bugs
- [ ] Record demo video
- [ ] Write installation instructions

---

## What You'll Need

### Before Starting

1. **Google Cloud Project**
   - Go to [console.cloud.google.com](https://console.cloud.google.com)
   - Create new project
   - Enable Google Calendar API
   - Create OAuth 2.0 credentials (Chrome Extension type)
   - Get Client ID

2. **Chrome Developer Mode**
   - Open Chrome → Extensions → Enable "Developer Mode"
   - You'll use "Load unpacked" to install your extension

3. **Test Google Calendar**
   - Use your personal calendar or create a test one
   - Add some fake events to test free time detection

4. **Curated Activity Data**
   - Research 10-15 great family activities in your city
   - Take photos or find images
   - Write descriptions

---

## Success Criteria

**This MVP is successful if:**

✅ It actually connects to Google Calendar (real OAuth, not fake)
✅ It correctly detects a free time block
✅ It shows 2 relevant suggestions
✅ It successfully adds an event to Google Calendar
✅ The demo flow takes <60 seconds
✅ People watching say "Oh wow, that's really useful!"

**It's NOT successful if:**
❌ Too buggy to demo smoothly
❌ Requires too much explanation ("well, imagine if...")
❌ People don't see the value ("seems like extra steps")

---

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| OAuth is too complex to implement in 5 days | Use Chrome's built-in `chrome.identity` API (simpler than web OAuth) |
| Free time detection doesn't work well | Hard-code a known free block for demo purposes |
| Suggestions feel random/irrelevant | Use small curated list (10-15 activities) that always look good |
| Calendar API rate limits | Only scan once per popup open; use caching |
| Demo fails during live presentation | Pre-record a backup video |

---

## After the Demo: What's Next?

**If the demo goes well**, the next steps are:

1. **User Testing (Week 2)**
   - Install extension for 5-10 friends/family
   - Get feedback on suggestions, UI, and value prop
   - Iterate on curated activities

2. **Expand to Web App (Weeks 3-6)**
   - Build full web app version (using PRD spec)
   - Add backend for weekly automation
   - Implement ML-based suggestions

3. **Beta Launch (Weeks 7-12)**
   - Recruit 50 beta users
   - Measure acceptance rate, retention
   - Validate monetization

---

## Questions to Answer

1. **City focus:** Which city will you demo for? (Affects curated activities)
2. **Family profile:** Hard-code ages 3 and 7, or make it configurable?
3. **Recording vs live:** Will you do live demo or pre-record video?
4. **Branding:** What's the extension name/icon for the demo?

---

## Files to Create

```
✓ manifest.json - Extension configuration
✓ popup.html - Main UI
✓ popup.css - Styling
✓ popup.js - UI logic
✓ background.js - OAuth service worker
✓ calendar.js - Google Calendar API wrapper
✓ suggestions.js - Matching logic + curated data
✓ README.md - Installation instructions
✓ icons/ - Extension icons (3 sizes)
```

---

**Ready to build this?** Let me know if you want me to generate the actual code files, or if you want to refine the spec first!
