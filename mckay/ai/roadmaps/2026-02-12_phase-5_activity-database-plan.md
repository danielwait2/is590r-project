# Phase 5: Activity Database - Implementation Plan

**Created:** 2026-02-12
**Phase:** 5 of 10
**Timeline:** Week 5
**Companion Document:** [Phase 5 Roadmap](./2026-02-12_phase-5_activity-database-roadmap.md)

---

## Project Principles

⚠️ **Hardcoded activities for MVP reliability.** No external APIs. Curate 20-30 quality activities manually.

---

## Phase Overview

**Goal:** Create and populate activity database with curated local activities

**Key deliverables:**
- activities.json with 20-30 activities
- Activity service module
- Activity search/filter functions
- Distance calculation (Haversine formula)

---

## Technical Specifications

### 1. Activity Data Structure

**File:** `server/data/activities.json`

```json
[
  {
    "id": "act-1",
    "name": "Beginner Pottery Workshop",
    "category": "classes",
    "description": "3-hour hands-on pottery class. Learn wheel throwing and hand-building techniques. All materials provided.",
    "location": {
      "name": "Clay Space Studio",
      "address": "456 Art St, Austin, TX 78701",
      "lat": 30.2650,
      "lng": -97.7400
    },
    "duration": 180,
    "price": "$$",
    "priceAmount": 75,
    "keywords": ["pottery", "ceramics", "art", "class", "hands-on", "crafts", "creative"],
    "url": "https://www.clayspacestudio.com/classes",
    "daysAvailable": ["saturday", "sunday"],
    "timeSlots": ["morning", "afternoon"],
    "notes": "Beginner-friendly, couples welcome"
  }
]
```

**Required fields:**
- id: Unique identifier
- name: Activity name
- category: One of the interest categories
- description: What the activity involves
- location: Venue name, address, lat/lng
- duration: Minutes (for time window matching)
- price: $ to $$$$ (for budget matching)
- priceAmount: Numeric price for comparison
- keywords: Array for want-to-try matching
- url: Link to activity/venue
- daysAvailable: Days offered
- timeSlots: When offered

**Activity categories (match interest checkboxes):**
- dining
- arts
- music
- classes
- outdoors
- sports
- entertainment

---

### 2. Activity Service Module

**File:** `server/services/activities.js`

```javascript
const fs = require('fs').promises;
const path = require('path');

const ACTIVITIES_FILE = path.join(__dirname, '../data/activities.json');

async function loadActivities() {
  const data = await fs.readFile(ACTIVITIES_FILE, 'utf8');
  return JSON.parse(data);
}

async function searchActivities(filters) {
  const activities = await loadActivities();
  let results = activities;

  // Filter by category (interests)
  if (filters.interests && filters.interests.length > 0) {
    results = results.filter(activity =>
      filters.interests.includes(activity.category)
    );
  }

  // Filter by keywords (want-to-try)
  if (filters.keywords && filters.keywords.length > 0) {
    results = results.filter(activity =>
      filters.keywords.some(keyword =>
        activity.keywords.some(activityKeyword =>
          activityKeyword.toLowerCase().includes(keyword.toLowerCase())
        )
      )
    );
  }

  // Filter by location (within max distance)
  if (filters.location && filters.maxDistance) {
    results = results.filter(activity => {
      const distance = calculateDistance(
        filters.location.lat,
        filters.location.lng,
        activity.location.lat,
        activity.location.lng
      );
      return distance <= filters.maxDistance;
    });
  }

  // Filter by budget
  if (filters.budget) {
    results = results.filter(activity =>
      comparePriceLevels(activity.price, filters.budget) <= 0
    );
  }

  return results;
}

// Haversine formula for distance between two lat/lng points
function calculateDistance(lat1, lon1, lat2, lon2) {
  const R = 3959; // Earth's radius in miles
  const dLat = toRad(lat2 - lat1);
  const dLon = toRad(lon2 - lon1);

  const a =
    Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(toRad(lat1)) *
      Math.cos(toRad(lat2)) *
      Math.sin(dLon / 2) *
      Math.sin(dLon / 2);

  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c; // Distance in miles
}

function toRad(degrees) {
  return (degrees * Math.PI) / 180;
}

function comparePriceLevels(activityPrice, budgetPrice) {
  const levels = { $: 1, $$: 2, $$$: 3, $$$$: 4 };
  return levels[activityPrice] - levels[budgetPrice];
}

module.exports = {
  loadActivities,
  searchActivities,
  calculateDistance
};
```

---

### 3. Sample Activities Structure

**Create 20-30 activities covering all categories:**

**Dining (5-6 activities):**
- Italian restaurant
- Sushi spot
- Brunch cafe
- Food truck park
- Wine bar

**Arts (4-5 activities):**
- Art museum
- Pottery class
- Painting night
- Gallery walk

**Music (3-4 activities):**
- Live music venue
- Jazz club
- Concert series

**Classes (4-5 activities):**
- Cooking class
- Dance lesson
- Craft workshop
- Wine tasting class

**Outdoors (4-5 activities):**
- Hiking trail
- Botanical garden
- Kayaking
- Bike trail

**Sports (2-3 activities):**
- Rock climbing gym
- Bowling
- Mini golf

**Entertainment (2-3 activities):**
- Comedy club
- Movie theater
- Escape room

---

## Testing Criteria

- [ ] activities.json populated with 20-30 activities
- [ ] All activities have required fields
- [ ] Categories match interest options
- [ ] Keywords relevant for matching
- [ ] Lat/lng coordinates accurate
- [ ] loadActivities() returns array
- [ ] searchActivities() filters correctly
- [ ] Distance calculation works
- [ ] Budget filtering works

---

## Next Steps

After Phase 5:
- Verify activities loaded correctly
- Test search/filter functions
- Move to [Phase 6: Matching Engine](./2026-02-12_phase-6_matching-engine-plan.md)

**Phase 5 complete when:** Have curated activity database ready for matching.

---

**Reference Documents:**
- [High-Level Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture](../../aiDocs/architecture.md)
- [Phase 5 Roadmap](./2026-02-12_phase-5_activity-database-roadmap.md)
