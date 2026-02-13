# Phase 1: MVP - Chrome Extension Demo

**Roadmap:** [2026-02-11-roadmap-phase1-mvp.md](./2026-02-11-roadmap-phase1-mvp.md) - Track progress

**Created:** 2026-02-11
**Timeline:** 4-5 days
**Status:** Ready to Start
**Type:** Proof-of-Concept / Demo

---

## Overview

Build a functional Chrome extension that validates the core value proposition: "Free time + desires → specific plan in 10 seconds." This is a client-side only demo to test user reactions to calendar access, suggestion quality, and the overall UX before building the full production web app.

**Key Constraint:** This is NOT production software. It's a demo to validate assumptions. Keep it simple, ship fast, get feedback.

---

## Success Criteria

**Must Have:**
- ✅ Real Google Calendar OAuth integration (not mocked)
- ✅ Correctly detects free time from actual user calendar
- ✅ Shows 2 relevant, curated suggestions
- ✅ Successfully creates event in Google Calendar
- ✅ Demo completes in <60 seconds
- ✅ Gets "wow, that's useful!" reaction from 5+ testers

**Validation Metrics:**
- 5 people say "this is really useful"
- Demo video gets shared organically
- Real calendar OAuth works reliably
- At least 3 of 5 testers accept a suggestion

---

## Prerequisites

**Before Starting Phase 1:**
- ✅ Google Cloud Project account created
- ✅ Read [Google Calendar API documentation](../guides/google-calendar-api_context7.md)
- ✅ Chrome browser installed for testing
- ✅ Basic understanding of Manifest V3 format
- ✅ Text editor ready (VS Code recommended)

**Required Accounts:**
- Google Cloud Console access
- Google account for testing calendar

---

## Detailed Implementation Steps

### 1A: Setup & Calendar Access (Day 1)

**Goal:** Get Google Calendar OAuth working in Chrome extension

#### Tasks:

**1. Google Cloud Project Setup**
```
- Go to https://console.cloud.google.com
- Create new project: "FamilyTime MVP"
- Enable Google Calendar API v3
- Create OAuth 2.0 credentials:
  - Application type: Chrome App
  - Authorized JavaScript origins: chrome-extension://[YOUR_EXTENSION_ID]
  - Scopes:
    - https://www.googleapis.com/auth/calendar.readonly
    - https://www.googleapis.com/auth/calendar.events
- Note down Client ID
```

**2. Extension Boilerplate**

Create directory: `family-time-extension/`

**manifest.json:**
```json
{
  "manifest_version": 3,
  "name": "FamilyTime - Activity Suggestions",
  "version": "1.0.0",
  "description": "Get personalized family activity suggestions based on your free time",
  "permissions": [
    "identity",
    "storage"
  ],
  "host_permissions": [
    "https://www.googleapis.com/*"
  ],
  "oauth2": {
    "client_id": "YOUR_CLIENT_ID.apps.googleusercontent.com",
    "scopes": [
      "https://www.googleapis.com/auth/calendar.readonly",
      "https://www.googleapis.com/auth/calendar.events"
    ]
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "background": {
    "service_worker": "background.js"
  },
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}
```

**popup.html:**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>FamilyTime</title>
  <link rel="stylesheet" href="popup.css">
</head>
<body>
  <div id="app">
    <div id="auth-section">
      <h2>FamilyTime</h2>
      <p>Get personalized family activity suggestions</p>
      <button id="connect-btn">Connect Google Calendar</button>
    </div>

    <div id="main-section" style="display:none;">
      <div id="free-time-display"></div>
      <div id="preferences-section">
        <h3>What do you want to try?</h3>
        <textarea id="want-to-try" placeholder="e.g., science museum, hiking, new restaurants"></textarea>
        <button id="get-suggestions-btn">Get Suggestions</button>
      </div>
      <div id="suggestions-container"></div>
    </div>
  </div>

  <script src="calendar.js"></script>
  <script src="suggestions.js"></script>
  <script src="popup.js"></script>
</body>
</html>
```

**popup.css:**
```css
body {
  width: 400px;
  min-height: 500px;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

#app {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

button {
  background: #4285f4;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

button:hover {
  background: #357ae8;
}

.suggestion-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 12px;
}

.suggestion-card img {
  width: 100%;
  border-radius: 4px;
  margin-bottom: 8px;
}

#want-to-try {
  width: 100%;
  height: 80px;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  resize: vertical;
}
```

**background.js:**
```javascript
// Service worker for OAuth token management
chrome.runtime.onInstalled.addListener(() => {
  console.log('FamilyTime extension installed');
});

// Handle OAuth token refresh
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === 'getAuthToken') {
    chrome.identity.getAuthToken({ interactive: true }, (token) => {
      if (chrome.runtime.lastError) {
        sendResponse({ error: chrome.runtime.lastError.message });
      } else {
        sendResponse({ token: token });
      }
    });
    return true; // Will respond asynchronously
  }
});
```

**calendar.js:**
```javascript
// Google Calendar API wrapper
const CalendarAPI = {
  async getAuthToken() {
    return new Promise((resolve, reject) => {
      chrome.runtime.sendMessage({ action: 'getAuthToken' }, (response) => {
        if (response.error) {
          reject(response.error);
        } else {
          resolve(response.token);
        }
      });
    });
  },

  async fetchEvents(timeMin, timeMax) {
    try {
      const token = await this.getAuthToken();

      const url = new URL('https://www.googleapis.com/calendar/v3/calendars/primary/events');
      url.searchParams.append('timeMin', timeMin.toISOString());
      url.searchParams.append('timeMax', timeMax.toISOString());
      url.searchParams.append('singleEvents', 'true');
      url.searchParams.append('orderBy', 'startTime');

      const response = await fetch(url, {
        headers: {
          'Authorization': `Bearer ${token}`
        }
      });

      if (!response.ok) {
        throw new Error(`API error: ${response.status}`);
      }

      const data = await response.json();
      return data.items || [];
    } catch (error) {
      console.error('Error fetching calendar events:', error);
      throw error;
    }
  },

  async createEvent(eventData) {
    try {
      const token = await this.getAuthToken();

      const response = await fetch('https://www.googleapis.com/calendar/v3/calendars/primary/events', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(eventData)
      });

      if (!response.ok) {
        throw new Error(`API error: ${response.status}`);
      }

      return await response.json();
    } catch (error) {
      console.error('Error creating event:', error);
      throw error;
    }
  },

  detectFreeTime(events) {
    // Implementation in Day 2 (1B)
    return null;
  }
};
```

**3. Icons Setup**
```
- Create icons/ directory
- Generate 3 icon sizes: 16x16, 48x48, 128x128
- Use simple calendar + family icon design
- Tools: Canva, Figma, or use placeholder from icons8.com
```

**4. Test OAuth Flow**
```
- Load unpacked extension in Chrome (chrome://extensions, Developer Mode ON)
- Click extension icon
- Click "Connect Google Calendar"
- Verify OAuth consent screen appears
- Grant permissions
- Check chrome.storage.local for token
```

**Reference:**
- [Google Calendar API - OAuth](../guides/google-calendar-api_context7.md)
- [Chrome Identity API Docs](https://developer.chrome.com/docs/extensions/reference/identity/)

---

### 1B: Free Time Detection (Day 2)

**Goal:** Accurately find 2-4 hour free blocks on weekends

#### Tasks:

**1. Implement Free Time Detection Algorithm**

Add to **calendar.js:**

```javascript
CalendarAPI.detectFreeTime = function(events) {
  const now = new Date();
  const nextWeek = new Date(now);
  nextWeek.setDate(nextWeek.getDate() + 7);

  // Get weekend dates in next 7 days
  const weekendDates = [];
  for (let d = new Date(now); d <= nextWeek; d.setDate(d.getDate() + 1)) {
    const day = d.getDay();
    if (day === 0 || day === 6) { // Sunday or Saturday
      weekendDates.push(new Date(d));
    }
  }

  const freeBlocks = [];

  weekendDates.forEach(date => {
    const dayStart = new Date(date);
    dayStart.setHours(9, 0, 0, 0); // 9am

    const dayEnd = new Date(date);
    dayEnd.setHours(18, 0, 0, 0); // 6pm

    // Get events on this day
    const dayEvents = events.filter(event => {
      const eventStart = new Date(event.start.dateTime || event.start.date);
      return eventStart >= dayStart && eventStart < dayEnd;
    }).sort((a, b) => {
      const aStart = new Date(a.start.dateTime || a.start.date);
      const bStart = new Date(b.start.dateTime || b.start.date);
      return aStart - bStart;
    });

    // Find gaps between events
    let currentTime = dayStart;

    dayEvents.forEach(event => {
      const eventStart = new Date(event.start.dateTime || event.start.date);
      const eventEnd = new Date(event.end.dateTime || event.end.date);

      const gapHours = (eventStart - currentTime) / (1000 * 60 * 60);

      if (gapHours >= 2) { // At least 2 hours free
        freeBlocks.push({
          date: new Date(currentTime),
          startTime: currentTime.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' }),
          endTime: eventStart.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' }),
          durationHours: gapHours
        });
      }

      currentTime = eventEnd;
    });

    // Check final gap until day end
    const finalGapHours = (dayEnd - currentTime) / (1000 * 60 * 60);
    if (finalGapHours >= 2) {
      freeBlocks.push({
        date: new Date(currentTime),
        startTime: currentTime.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' }),
        endTime: dayEnd.toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' }),
        durationHours: finalGapHours
      });
    }
  });

  return freeBlocks.length > 0 ? freeBlocks[0] : null; // Return first available
};
```

**2. Display Free Time in UI**

Add to **popup.js:**

```javascript
async function loadFreeTime() {
  try {
    const now = new Date();
    const nextWeek = new Date(now);
    nextWeek.setDate(nextWeek.getDate() + 7);

    const events = await CalendarAPI.fetchEvents(now, nextWeek);
    const freeBlock = CalendarAPI.detectFreeTime(events);

    const display = document.getElementById('free-time-display');

    if (freeBlock) {
      display.innerHTML = `
        <div style="background: #e8f5e9; padding: 16px; border-radius: 8px; margin-bottom: 16px;">
          <h3 style="margin: 0 0 8px 0;">Next Free Time</h3>
          <p style="margin: 0; font-size: 18px; font-weight: bold;">
            ${freeBlock.date.toLocaleDateString('en-US', { weekday: 'long', month: 'short', day: 'numeric' })}
          </p>
          <p style="margin: 4px 0 0 0; color: #666;">
            ${freeBlock.startTime} - ${freeBlock.endTime} (${Math.floor(freeBlock.durationHours)} hours)
          </p>
        </div>
      `;
    } else {
      display.innerHTML = `
        <div style="background: #fff3e0; padding: 16px; border-radius: 8px; margin-bottom: 16px;">
          <p style="margin: 0;">No free time blocks found in the next week.</p>
        </div>
      `;
    }
  } catch (error) {
    console.error('Error loading free time:', error);
  }
}
```

**3. Test Edge Cases**
```
Test scenarios:
- Empty calendar (all weekend free)
- Fully booked weekend (no free time)
- Single event (creates two free blocks)
- Back-to-back events
- All-day events
- Events spanning midnight
```

**Decision Point:**
- If detection accuracy is poor, consider adding user confirmation: "Is Saturday 2pm-5pm free for you?"

---

### 1C: Suggestions Engine (Day 3)

**Goal:** Show 2 relevant suggestions based on curated activities and user preferences

#### Tasks:

**1. Create Curated Activities Database**

Create **suggestions.js:**

```javascript
const CURATED_ACTIVITIES = [
  {
    id: 'act_001',
    title: 'Children\'s Discovery Museum',
    type: 'museum',
    description: 'Interactive science exhibits perfect for curious kids. Hands-on learning about physics, biology, and space.',
    address: '123 Main St, Chicago, IL',
    imageUrl: 'https://via.placeholder.com/300x200/4CAF50/ffffff?text=Museum',
    estimatedCost: '$35 for family',
    duration: '2-3 hours',
    ageRange: [2, 10],
    tags: ['science', 'museum', 'indoor', 'educational', 'kids'],
    bookingLink: 'https://example.com/museum'
  },
  {
    id: 'act_002',
    title: 'Lincoln Park Zoo',
    type: 'outdoors',
    description: 'Free admission zoo with diverse animal exhibits. Great for a casual family day outdoors.',
    address: '2001 N Clark St, Chicago, IL',
    imageUrl: 'https://via.placeholder.com/300x200/2196F3/ffffff?text=Zoo',
    estimatedCost: 'Free (donations welcome)',
    duration: '3-4 hours',
    ageRange: [1, 12],
    tags: ['outdoors', 'zoo', 'animals', 'nature', 'free'],
    bookingLink: null
  },
  {
    id: 'act_003',
    title: 'Paint Your Own Pottery',
    type: 'arts',
    description: 'Creative pottery painting studio. Kids choose a piece, paint it, and pick it up glazed next week.',
    address: '456 Art Lane, Chicago, IL',
    imageUrl: 'https://via.placeholder.com/300x200/FF9800/ffffff?text=Pottery',
    estimatedCost: '$25-40',
    duration: '1-2 hours',
    ageRange: [4, 12],
    tags: ['arts', 'crafts', 'creative', 'indoor', 'kids'],
    bookingLink: 'https://example.com/pottery'
  },
  {
    id: 'act_004',
    title: 'Family Bike Trail - Lakefront Path',
    type: 'outdoors',
    description: 'Scenic 18-mile paved bike path along Lake Michigan. Bring your bikes or rent from local shops.',
    address: 'Chicago Lakefront',
    imageUrl: 'https://via.placeholder.com/300x200/4CAF50/ffffff?text=Biking',
    estimatedCost: 'Free (or $15 bike rentals)',
    duration: '2-3 hours',
    ageRange: [5, 15],
    tags: ['outdoors', 'biking', 'nature', 'exercise', 'free'],
    bookingLink: null
  },
  {
    id: 'act_005',
    title: 'Cooking Class for Families',
    type: 'food',
    description: 'Learn to make pizza from scratch together. Each family makes their own pie to take home.',
    address: '789 Chef St, Chicago, IL',
    imageUrl: 'https://via.placeholder.com/300x200/F44336/ffffff?text=Cooking',
    estimatedCost: '$60 per family',
    duration: '2 hours',
    ageRange: [5, 12],
    tags: ['food', 'cooking', 'indoor', 'learning', 'kids'],
    bookingLink: 'https://example.com/cooking'
  },
  {
    id: 'act_006',
    title: 'Brookfield Zoo',
    type: 'outdoors',
    description: 'Large zoo with unique habitats and animal shows. Plan for a full day trip.',
    address: '8400 W 31st St, Brookfield, IL',
    imageUrl: 'https://via.placeholder.com/300x200/2196F3/ffffff?text=Zoo',
    estimatedCost: '$25 adults, $18 kids',
    duration: '4-6 hours',
    ageRange: [2, 15],
    tags: ['outdoors', 'zoo', 'animals', 'nature', 'day-trip'],
    bookingLink: 'https://brookfieldzoo.org'
  },
  {
    id: 'act_007',
    title: 'Indoor Trampoline Park',
    type: 'recreation',
    description: 'Wall-to-wall trampolines, foam pits, dodgeball courts. Perfect for burning energy indoors.',
    address: '321 Jump Ave, Chicago, IL',
    imageUrl: 'https://via.placeholder.com/300x200/9C27B0/ffffff?text=Trampoline',
    estimatedCost: '$20 per child',
    duration: '2 hours',
    ageRange: [4, 12],
    tags: ['indoor', 'active', 'recreation', 'kids', 'energy'],
    bookingLink: 'https://example.com/trampoline'
  },
  {
    id: 'act_008',
    title: 'Chicago Botanic Garden',
    type: 'outdoors',
    description: 'Beautiful gardens with themed areas. Peaceful walking paths and seasonal exhibits.',
    address: '1000 Lake Cook Rd, Glencoe, IL',
    imageUrl: 'https://via.placeholder.com/300x200/4CAF50/ffffff?text=Garden',
    estimatedCost: 'Free admission ($30 parking)',
    duration: '2-3 hours',
    ageRange: [3, 15],
    tags: ['outdoors', 'nature', 'gardens', 'walking', 'peaceful'],
    bookingLink: 'https://chicagobotanic.org'
  },
  {
    id: 'act_009',
    title: 'Movie Matinee - Family Films',
    type: 'entertainment',
    description: 'Latest family-friendly movies in theaters. Check local listings for showtimes.',
    address: 'Various theaters',
    imageUrl: 'https://via.placeholder.com/300x200/F44336/ffffff?text=Movies',
    estimatedCost: '$10-15 per ticket',
    duration: '2-3 hours',
    ageRange: [3, 15],
    tags: ['indoor', 'movies', 'entertainment', 'relaxed'],
    bookingLink: null
  },
  {
    id: 'act_010',
    title: 'Mini Golf Adventure',
    type: 'recreation',
    description: 'Themed mini golf course with fun obstacles. Great for all skill levels.',
    address: '555 Putt Drive, Chicago, IL',
    imageUrl: 'https://via.placeholder.com/300x200/FF9800/ffffff?text=Mini+Golf',
    estimatedCost: '$12 per person',
    duration: '1-2 hours',
    ageRange: [4, 15],
    tags: ['outdoors', 'recreation', 'mini-golf', 'competitive', 'fun'],
    bookingLink: null
  }
];

// Simple keyword matching algorithm
function matchActivities(wantToTryList, kidAges = [3, 7]) {
  // Default assumption: 2 kids, ages 3 and 7 (from mvp.md hard-coded demo)

  // 1. Tokenize want-to-try items
  const keywords = wantToTryList
    .toLowerCase()
    .split(/[,\n]/)
    .map(item => item.trim())
    .filter(item => item.length > 0);

  // 2. Score each activity
  const scored = CURATED_ACTIVITIES.map(activity => {
    let score = 0;

    // Tag matching (2 points per match)
    keywords.forEach(keyword => {
      activity.tags.forEach(tag => {
        if (tag.includes(keyword) || keyword.includes(tag)) {
          score += 2;
        }
      });

      // Title/description matching (1 point)
      if (activity.title.toLowerCase().includes(keyword)) {
        score += 1;
      }
    });

    // Age appropriateness (bonus 3 points if all kids fit)
    const allKidsFit = kidAges.every(age =>
      age >= activity.ageRange[0] && age <= activity.ageRange[1]
    );
    if (allKidsFit) {
      score += 3;
    }

    return { activity, score };
  });

  // 3. Sort by score, return top 2
  return scored
    .filter(item => item.score > 0) // Only return matches
    .sort((a, b) => b.score - a.score)
    .slice(0, 2)
    .map(item => item.activity);
}
```

**2. Implement Suggestion Display**

Add to **popup.js:**

```javascript
async function showSuggestions() {
  // Get preferences
  const wantToTryText = document.getElementById('want-to-try').value;

  if (!wantToTryText.trim()) {
    alert('Please tell us what you want to try!');
    return;
  }

  // Save to storage
  chrome.storage.local.set({ wantToTry: wantToTryText });

  // Match activities
  const suggestions = matchActivities(wantToTryText);

  // Display
  const container = document.getElementById('suggestions-container');

  if (suggestions.length === 0) {
    container.innerHTML = `
      <div style="background: #ffebee; padding: 16px; border-radius: 8px;">
        <p style="margin: 0;">No matching activities found. Try different keywords like "museum", "outdoors", "arts".</p>
      </div>
    `;
    return;
  }

  container.innerHTML = suggestions.map(activity => `
    <div class="suggestion-card">
      <img src="${activity.imageUrl}" alt="${activity.title}">
      <h3 style="margin: 0 0 8px 0;">${activity.title}</h3>
      <p style="margin: 0 0 8px 0; color: #666; font-size: 14px;">${activity.description}</p>
      <p style="margin: 0 0 4px 0; font-size: 13px;">📍 ${activity.address}</p>
      <p style="margin: 0 0 4px 0; font-size: 13px;">💰 ${activity.estimatedCost}</p>
      <p style="margin: 0 0 12px 0; font-size: 13px;">⏱️ ${activity.duration}</p>
      <button class="add-to-calendar-btn" data-activity-id="${activity.id}">Add to Calendar</button>
    </div>
  `).join('');

  // Add event listeners to "Add to Calendar" buttons
  document.querySelectorAll('.add-to-calendar-btn').forEach(btn => {
    btn.addEventListener('click', (e) => {
      const activityId = e.target.dataset.activityId;
      addToCalendar(activityId);
    });
  });
}

document.getElementById('get-suggestions-btn').addEventListener('click', showSuggestions);
```

**3. Load Saved Preferences**

Add to **popup.js** initialization:

```javascript
// Load saved preferences on popup open
chrome.storage.local.get(['wantToTry'], (result) => {
  if (result.wantToTry) {
    document.getElementById('want-to-try').value = result.wantToTry;
  }
});
```

**Testing:**
```
Test cases:
- Enter "museum" → Should show museum activities
- Enter "outdoors, nature" → Should show zoo, gardens, biking
- Enter "arts, crafts" → Should show pottery
- Empty input → Should show error
- No matches → Should show helpful message
```

---

### 1D: Calendar Integration (Day 4)

**Goal:** Successfully add suggested activities back to Google Calendar

#### Tasks:

**1. Implement Event Creation**

Add to **popup.js:**

```javascript
async function addToCalendar(activityId) {
  try {
    // Find activity
    const activity = CURATED_ACTIVITIES.find(a => a.id === activityId);
    if (!activity) {
      alert('Activity not found');
      return;
    }

    // Get free time block (re-fetch or use cached)
    const now = new Date();
    const nextWeek = new Date(now);
    nextWeek.setDate(nextWeek.getDate() + 7);
    const events = await CalendarAPI.fetchEvents(now, nextWeek);
    const freeBlock = CalendarAPI.detectFreeTime(events);

    if (!freeBlock) {
      alert('No free time available');
      return;
    }

    // Build event object
    const eventData = {
      summary: activity.title,
      description: `${activity.description}\n\n📍 ${activity.address}\n💰 ${activity.estimatedCost}\n⏱️ ${activity.duration}\n\n${activity.bookingLink ? `Book here: ${activity.bookingLink}` : ''}`,
      location: activity.address,
      start: {
        dateTime: freeBlock.date.toISOString(),
        timeZone: Intl.DateTimeFormat().resolvedOptions().timeZone
      },
      end: {
        dateTime: new Date(freeBlock.date.getTime() + 2 * 60 * 60 * 1000).toISOString(), // 2 hours
        timeZone: Intl.DateTimeFormat().resolvedOptions().timeZone
      },
      reminders: {
        useDefault: false,
        overrides: [
          { method: 'popup', minutes: 24 * 60 }, // 1 day before
          { method: 'popup', minutes: 60 } // 1 hour before
        ]
      }
    };

    // Create event
    const createdEvent = await CalendarAPI.createEvent(eventData);

    // Show success
    alert(`✅ Added "${activity.title}" to your calendar!\n\nEvent link: ${createdEvent.htmlLink}`);

    // Optional: Open calendar in new tab
    const calendarUrl = `https://calendar.google.com/calendar/r/eventedit/${createdEvent.id}`;
    chrome.tabs.create({ url: calendarUrl });

  } catch (error) {
    console.error('Error adding to calendar:', error);
    alert('Failed to add to calendar. Please try again.');
  }
}
```

**2. Polish UI States**

Add loading states to **popup.css:**

```css
.add-to-calendar-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.add-to-calendar-btn.loading::after {
  content: '...';
  animation: dots 1s steps(3, end) infinite;
}

@keyframes dots {
  0%, 20% { content: '.'; }
  40% { content: '..'; }
  60%, 100% { content: '...'; }
}
```

Update button click handler:

```javascript
async function addToCalendar(activityId) {
  const btn = document.querySelector(`[data-activity-id="${activityId}"]`);
  btn.disabled = true;
  btn.classList.add('loading');
  btn.textContent = 'Adding';

  try {
    // ... existing code ...
  } catch (error) {
    // ... error handling ...
  } finally {
    btn.disabled = false;
    btn.classList.remove('loading');
    btn.textContent = 'Add to Calendar';
  }
}
```

**3. Error Handling**

Add comprehensive error handling:

```javascript
// In calendar.js
CalendarAPI.handleError = function(error) {
  if (error.message.includes('401')) {
    // Token expired, clear and prompt re-auth
    chrome.identity.removeCachedAuthToken({ token: lastToken }, () => {
      alert('Session expired. Please reconnect your calendar.');
      window.location.reload();
    });
  } else if (error.message.includes('403')) {
    alert('Calendar access denied. Please grant permissions.');
  } else if (error.message.includes('429')) {
    alert('Too many requests. Please wait a moment and try again.');
  } else {
    alert(`Error: ${error.message}`);
  }
};
```

**Testing:**
```
Test scenarios:
- Successfully add event → Opens in Google Calendar
- Token expired → Prompts re-auth
- API rate limit → Shows error message
- No internet connection → Graceful failure
- Event already exists → Should create anyway (no deduplication in MVP)
```

---

### 1E: Testing & Demo Prep (Day 5)

**Goal:** Polish, test, and prepare for user demos

#### Tasks:

**1. End-to-End Testing Checklist**

```
[ ] Fresh install test (no prior auth)
[ ] OAuth flow completes successfully
[ ] Free time detection works on various calendar patterns
[ ] Suggestions match user input keywords
[ ] "Add to Calendar" creates event correctly
[ ] Event appears in Google Calendar within 5 seconds
[ ] Re-opening extension shows saved preferences
[ ] Works across different timezones
[ ] Handles API errors gracefully
[ ] Extension doesn't crash with invalid input
```

**2. Edge Cases Testing**

```
Edge cases to test:
- No free time available (fully booked weekend)
- Empty calendar (all weekend free)
- Want to try list with special characters
- Very long activity descriptions
- Network disconnection during API call
- Multiple extension instances open
- Calendar with all-day events
```

**3. Bug Fixes**

Create **bugfixes.md** to track issues:

```markdown
# Known Issues

## High Priority
- [ ] Issue description
- [ ] Steps to reproduce
- [ ] Expected vs actual behavior
- [ ] Fix implemented

## Medium Priority
...

## Low Priority (Won't Fix in MVP)
...
```

**4. Demo Video Recording**

Record <2 minute demo video:

```
Script:
1. [0:00-0:15] Problem statement: "It's hard to find time for family activities"
2. [0:15-0:30] Install extension, click icon
3. [0:30-0:45] Connect Google Calendar (show OAuth)
4. [0:45-1:00] Extension shows free time: "Saturday 2-6pm"
5. [1:00-1:15] Enter preferences: "science, outdoors, museums"
6. [1:15-1:30] Show 2 suggestions
7. [1:30-1:45] Click "Add to Calendar" on one suggestion
8. [1:45-2:00] Event appears in Google Calendar, show success state

Tools:
- Screen recording: Loom, QuickTime, OBS
- Add captions for clarity
- Background music (optional, keep subtle)
```

**5. Installation Guide**

Create **README.md:**

```markdown
# FamilyTime - Chrome Extension Demo

Get personalized family activity suggestions based on your Google Calendar free time.

## Installation

1. Download this repository as ZIP
2. Unzip to a local folder
3. Open Chrome and go to `chrome://extensions`
4. Enable "Developer mode" (top right)
5. Click "Load unpacked"
6. Select the `family-time-extension` folder
7. The extension icon should appear in your toolbar

## Setup

1. Click the FamilyTime icon in your toolbar
2. Click "Connect Google Calendar"
3. Grant calendar permissions
4. Enter activities you want to try (e.g., "museums, hiking, arts")
5. Click "Get Suggestions"
6. Click "Add to Calendar" on any suggestion you like

## Requirements

- Google Chrome browser
- Google account with Google Calendar
- Active calendar with some events (for accurate free time detection)

## Privacy

- This extension only reads your calendar to find free time
- No data is stored on external servers
- All processing happens locally in your browser
- You can revoke access anytime in Google Account settings

## Support

Questions or issues? Contact: [your-email]
```

**6. User Testing Sessions**

Recruit 3-5 testers:

```
Tester profile:
- Parents with 1-3 kids
- Use Google Calendar regularly
- Willing to provide honest feedback
- Available for 30-min demo session

Testing script:
1. No guidance - let them try to figure it out
2. Observe where they get stuck
3. Ask questions:
   - What was confusing?
   - What did you expect to happen?
   - Would you use this weekly?
   - What's missing?
4. Record feedback in notes

Success metric:
- 3 of 5 testers say "this is really useful"
- <2 friction points per tester
- Average rating >4/5
```

**7. Feedback Collection**

Create **user-feedback.md:**

```markdown
# User Testing Feedback

## Tester 1 - [Name]
**Date:** [Date]
**Profile:** Parent of 2 (ages 4, 7), uses Google Calendar daily

### Observations
- [What they did, where they got stuck]

### Feedback
- [Direct quotes]

### Rating: X/5

---

## Tester 2 - [Name]
...
```

---

## Technical Stack & Tools

**Core Technologies:**
- **Chrome Extension:** Manifest V3 format
- **Language:** Vanilla JavaScript (ES6+)
- **UI:** HTML5 + CSS3 (no frameworks)
- **Storage:** Chrome Storage API (`chrome.storage.local`)
- **Authentication:** Chrome Identity API (`chrome.identity`)

**APIs:**
- **Google Calendar API v3** - Event read/write access
  - Reference: [Google Calendar API Guide](../guides/google-calendar-api_context7.md)
  - Scopes: `calendar.readonly`, `calendar.events`
  - Rate limit: 1M requests/day (free tier)

**Development Tools:**
- **Code Editor:** VS Code (recommended extensions: Chrome Extension Kit)
- **Testing:** Chrome DevTools, chrome://extensions
- **Version Control:** Git (optional for MVP)
- **Screen Recording:** Loom, QuickTime, or OBS for demo video

**No Backend Required:**
- All logic runs client-side in browser
- No database (uses Chrome local storage)
- No API server needed
- No hosting costs

---

## File Structure

```
family-time-extension/
├── manifest.json           # Extension configuration (OAuth, permissions)
├── popup.html              # Main UI (extension popup)
├── popup.css               # Styles for popup
├── popup.js                # UI logic and event handlers
├── background.js           # Service worker (OAuth token management)
├── calendar.js             # Google Calendar API wrapper
├── suggestions.js          # Curated activities + matching algorithm
├── icons/
│   ├── icon16.png         # Toolbar icon (16x16)
│   ├── icon48.png         # Extension manager icon (48x48)
│   └── icon128.png        # Chrome Web Store icon (128x128)
├── README.md              # Installation and usage instructions
├── demo-video.mp4         # <2 minute demo recording
└── user-feedback.md       # Testing notes and feedback
```

**Total Files:** 13
**Total Lines of Code:** ~500-700 (excluding comments)

---

## Testing Strategy

### Unit Testing (Manual)

**calendar.js Functions:**
```
✓ getAuthToken() - Returns valid OAuth token
✓ fetchEvents() - Returns array of calendar events
✓ createEvent() - Successfully creates event in calendar
✓ detectFreeTime() - Finds 2-4 hour blocks on weekends
```

**suggestions.js Functions:**
```
✓ matchActivities() - Returns top 2 matches based on keywords
✓ Handles empty input gracefully
✓ Handles no matches scenario
```

### Integration Testing

**Full Flow Tests:**
```
1. Install Extension
   ✓ Extension loads without errors
   ✓ Popup opens when icon clicked

2. OAuth Flow
   ✓ "Connect Calendar" button triggers OAuth
   ✓ Consent screen shows correct scopes
   ✓ Token stored in chrome.storage

3. Free Time Detection
   ✓ Fetches events from calendar
   ✓ Displays next free weekend block
   ✓ Shows message if no free time

4. Suggestion Generation
   ✓ Accepts user input
   ✓ Shows 2 matching activities
   ✓ Displays activity cards with images

5. Add to Calendar
   ✓ Creates event with correct details
   ✓ Event appears in Google Calendar
   ✓ Success message shown
```

### User Acceptance Testing (UAT)

**3-5 Real Users:**
- Install extension themselves
- Complete full flow unsupervised
- Provide feedback via form
- Rate experience 1-5

**Success Criteria:**
- 80%+ can complete flow without help
- Average rating >4/5
- 60%+ say they'd use it weekly

---

## Risks & Mitigation

### High Risk

**1. OAuth Complexity**
- **Risk:** Users struggle with Google permissions flow
- **Mitigation:**
  - Add clear instructions before OAuth
  - Show example permissions screen
  - Provide "Why we need this" explanation
- **Contingency:** Offer 1-on-1 setup help for testers

**2. Poor Suggestion Quality**
- **Risk:** Curated activities don't match user interests
- **Mitigation:**
  - Curate diverse activity types (10+ categories)
  - Use keyword matching on multiple fields (title, tags, description)
  - Add feedback mechanism: "Was this useful? Yes/No"
- **Contingency:** Manually add activities based on user feedback

**3. Free Time Detection Accuracy**
- **Risk:** Algorithm misses free time or suggests unavailable slots
- **Mitigation:**
  - Test with various calendar patterns
  - Add buffer before/after events (15-30 min)
  - Show detected free time for user confirmation
- **Contingency:** Let users manually select time slot

### Medium Risk

**4. Calendar API Quota Limits**
- **Risk:** Exceed free tier limits during testing
- **Mitigation:**
  - Cache calendar data (5-10 min TTL)
  - Limit to 1 API call per popup open
- **Contingency:** Request quota increase from Google

**5. Extension Installation Friction**
- **Risk:** Non-technical testers can't install unpacked extension
- **Mitigation:**
  - Provide step-by-step video tutorial
  - Offer remote screen-sharing setup help
- **Contingency:** Do live demos instead of self-installation

### Low Risk

**6. Cross-Browser Compatibility**
- **Risk:** Doesn't work in non-Chrome browsers
- **Mitigation:** This is Chrome-only for MVP (by design)
- **Contingency:** N/A - explicitly Chrome Extension

---

## Deliverables

### Code Artifacts
- ✅ Working Chrome extension (unpacked .zip)
- ✅ manifest.json with Google Calendar OAuth configuration
- ✅ Curated activities database (10+ items in suggestions.js)
- ✅ Free time detection algorithm (calendar.js)
- ✅ Simple keyword matching engine (suggestions.js)

### Documentation
- ✅ README.md - Installation and usage guide
- ✅ user-feedback.md - Testing notes from 3-5 users
- ✅ bugfixes.md - Known issues tracker

### Demo Materials
- ✅ Demo video (<2 minutes, shows full flow)
- ✅ Screenshots of key screens (OAuth, suggestions, calendar event)
- ✅ Tester feedback summary (what worked, what didn't)

### Learnings Document
```markdown
# MVP Learnings

## What We Validated
- [ ] Do people grant calendar access? (Y/N, X% of testers)
- [ ] Are curated suggestions relevant? (Rating 1-5)
- [ ] What types of activities get accepted most?
- [ ] Is the UX intuitive? (Average time to complete flow: X min)

## Key Insights
- [Insight 1]
- [Insight 2]

## What We'll Change for V1
- [Decision 1 based on feedback]
- [Decision 2 based on feedback]

## Recommendation
- [ ] Proceed to Phase 2 (V1 Web App)
- [ ] Pivot to different approach
- [ ] Kill project
```

---

## Decision Points & Alternatives

### Key Decisions Made

**1. Chrome Extension vs Web App for MVP**
- **Decision:** Chrome Extension
- **Rationale:** Faster to build, no backend needed, users already in browser
- **Alternative Considered:** Web app with manual calendar upload
- **Trade-off:** Limited to Chrome users, harder to scale

**2. Curated Activities vs API Integration**
- **Decision:** Hard-coded curated list (10-15 activities)
- **Rationale:** Faster, higher quality, no API costs
- **Alternative Considered:** Google Places API from the start
- **Trade-off:** Limited variety, manual maintenance

**3. OAuth vs Screenshot Upload**
- **Decision:** Real OAuth integration
- **Rationale:** Want to validate if users will grant access
- **Alternative Considered:** Vision API screenshot analysis
- **Trade-off:** Higher friction, but validates core assumption

### Alternatives to Consider if MVP Fails

**If Users Don't Grant Calendar Access:**
- Pivot to screenshot upload approach (Vision API)
- Manual time entry (dropdown: "When are you free?")
- Focus on couples calendars only (less sensitive)

**If Suggestions Aren't Relevant:**
- Switch to API-first (Google Places, Eventbrite)
- Add more detailed preference form (30+ questions)
- Focus on single activity type (e.g., restaurants only)

**If UX Is Too Complex:**
- Remove "Want to Try" form, auto-generate suggestions
- Pre-fill demo data, make it one-click
- Build as chatbot instead of extension

---

## References

### Main Project Documents
- [High-Level Roadmap](./2026-02-11-high-level_mvp-project-roadmap.md) - Overall project plan
- [PRD](../../aiDocs/prd.md) - Product vision and requirements
- [MVP Spec](../../aiDocs/mvp.md) - Chrome extension detailed spec
- [Architecture](../../aiDocs/architecture.md) - MVP technical architecture

### Technical Guides
- [Google Calendar API](../guides/google-calendar-api_context7.md) - OAuth, events, free/busy
- [Chrome Extensions Manifest V3](https://developer.chrome.com/docs/extensions/mv3/)
- [Chrome Identity API](https://developer.chrome.com/docs/extensions/reference/identity/)

### Inspiration & Research
- [Market Research](../guides/quality-time-app-market-research.md) - Competitive analysis
- [User Pain Points Research](../../aiDocs/prd.md#user-pain-points) - From PRD

---

**Last Updated:** 2026-02-11
**Next Review:** After Phase 1 completion (Day 5)
**Version:** 1.0
