# Phase 6: Matching Engine - Implementation Plan

**Created:** 2026-02-12
**Phase:** 6 of 10
**Timeline:** Week 5-6
**Companion Document:** [Phase 6 Roadmap](./2026-02-12_phase-6_matching-engine-roadmap.md)

---

## Project Principles

⚠️ **Simple scoring algorithm.** Don't over-optimize. Focus on getting relevant matches.

---

## Phase Overview

**Goal:** Core logic to match activities to free time windows

**Key deliverables:**
- Matching service module
- Activity scoring algorithm
- Time window matching
- Match reason generation
- Main orchestration function

---

## Technical Specifications

### 1. Matching Service Module

**File:** `server/services/matching.js`

```javascript
const calendarService = require('./calendar');
const activitiesService = require('./activities');
const preferencesService = require('./preferences');

async function generateSuggestions(coupleId) {
  // 1. Load preferences
  const preferences = await preferencesService.getPreferences(coupleId);
  if (!preferences) {
    throw new Error('Preferences not set for this couple');
  }

  // 2. Get overlapping free time windows
  const freeWindows = await calendarService.getAvailableTimeForCouple(
    coupleId,
    preferences
  );

  if (freeWindows.length === 0) {
    return {
      success: true,
      freeWindows: [],
      matchedActivities: [],
      suggestions: [],
      message: 'No overlapping free time found in next 2 weeks'
    };
  }

  // 3. Search for matching activities
  const activities = await activitiesService.searchActivities({
    interests: preferences.interests,
    keywords: preferences.wantToTry,
    location: preferences.location,
    maxDistance: 15,
    budget: preferences.budget
  });

  if (activities.length === 0) {
    return {
      success: true,
      freeWindows,
      matchedActivities: [],
      suggestions: [],
      message: 'No activities match your preferences'
    };
  }

  // 4. Score and rank activities
  const scoredActivities = activities.map(activity => ({
    ...activity,
    score: calculateActivityScore(activity, preferences),
    matchReasons: generateMatchReasons(activity, preferences)
  }));

  scoredActivities.sort((a, b) => b.score - a.score);

  // 5. Match activities to time windows
  const suggestions = matchActivitiesToTimeWindows(
    scoredActivities,
    freeWindows
  );

  // 6. Return top 1-2 suggestions
  return {
    success: true,
    freeWindows,
    matchedActivities: scoredActivities,
    suggestions: suggestions.slice(0, 2)
  };
}

function calculateActivityScore(activity, preferences) {
  let score = 0;

  // Category match (interests): +10
  if (preferences.interests.includes(activity.category)) {
    score += 10;
  }

  // Keyword match (want-to-try): +5 each
  const wantToTryKeywords = preferences.wantToTry.map(item =>
    item.toLowerCase()
  );
  const matchingKeywords = activity.keywords.filter(keyword =>
    wantToTryKeywords.some(want =>
      want.includes(keyword.toLowerCase()) ||
      keyword.toLowerCase().includes(want)
    )
  );
  score += matchingKeywords.length * 5;

  // Location proximity
  const distance = activitiesService.calculateDistance(
    preferences.location.lat,
    preferences.location.lng,
    activity.location.lat,
    activity.location.lng
  );
  if (distance < 5) score += 3;
  else if (distance < 10) score += 2;
  else if (distance < 15) score += 1;

  // Price match: +2 if within budget
  const priceLevels = { $: 1, $$: 2, $$$: 3, $$$$: 4 };
  if (priceLevels[activity.price] <= priceLevels[preferences.budget]) {
    score += 2;
  }

  return score;
}

function generateMatchReasons(activity, preferences) {
  const reasons = [];

  // Category match
  if (preferences.interests.includes(activity.category)) {
    reasons.push(`Matches your interest in ${activity.category}`);
  }

  // Want-to-try match
  const matchedKeywords = [];
  preferences.wantToTry.forEach(want => {
    activity.keywords.forEach(keyword => {
      if (
        want.toLowerCase().includes(keyword.toLowerCase()) ||
        keyword.toLowerCase().includes(want.toLowerCase())
      ) {
        matchedKeywords.push(want);
      }
    });
  });

  if (matchedKeywords.length > 0) {
    reasons.push(`On your want-to-try list: "${matchedKeywords[0]}"`);
  }

  // Location
  const distance = activitiesService.calculateDistance(
    preferences.location.lat,
    preferences.location.lng,
    activity.location.lat,
    activity.location.lng
  );
  reasons.push(`${distance.toFixed(1)} miles from you`);

  return reasons.join('. ');
}

function matchActivitiesToTimeWindows(activities, timeWindows) {
  const suggestions = [];

  for (const window of timeWindows) {
    const windowDuration = (new Date(window.end) - new Date(window.start)) / (1000 * 60);

    // Find activity that fits this window
    const matchingActivity = activities.find(
      activity => activity.duration <= windowDuration
    );

    if (matchingActivity) {
      suggestions.push({
        activity: matchingActivity,
        timeWindow: window,
        suggestedStart: window.start,
        suggestedEnd: new Date(
          new Date(window.start).getTime() +
            matchingActivity.duration * 60 * 1000
        ).toISOString(),
        score: matchingActivity.score,
        matchReasons: matchingActivity.matchReasons
      });

      // Remove this activity so we don't suggest it twice
      activities = activities.filter(a => a.id !== matchingActivity.id);
    }

    // Stop after 2 suggestions
    if (suggestions.length >= 2) break;
  }

  return suggestions;
}

module.exports = {
  generateSuggestions
};
```

---

### 2. Matching Routes

**File:** `server/routes/matching.js`

```javascript
const express = require('express');
const router = express.Router();
const matchingService = require('../services/matching');

// Generate suggestions for a couple
router.post('/:coupleId', async (req, res) => {
  try {
    const { coupleId } = req.params;

    const result = await matchingService.generateSuggestions(coupleId);

    res.json(result);
  } catch (error) {
    console.error('Error generating suggestions:', error);
    res.status(500).json({
      success: false,
      error: error.message
    });
  }
});

module.exports = router;
```

---

### 3. Scoring Algorithm

**Per architecture.md:**

- **Category match** (interest): +10 points
- **Keyword match** (want-to-try): +5 points each
- **Location proximity:**
  - <5 miles: +3 points
  - <10 miles: +2 points
  - <15 miles: +1 point
- **Price match** (within budget): +2 points

**Higher score = better match**

---

## Testing Criteria

- [ ] generateSuggestions() orchestrates full flow
- [ ] Activity scoring works correctly
- [ ] Match reasons are human-readable
- [ ] Time window matching works
- [ ] Returns top 1-2 suggestions
- [ ] Handles no free time gracefully
- [ ] Handles no matching activities
- [ ] Scores reflect relevance

---

## Next Steps

After Phase 6:
- Verify matching logic produces good suggestions
- Test scoring algorithm
- Move to [Phase 7: Suggestion Creation](./2026-02-12_phase-7_suggestion-creation-plan.md)

**Phase 6 complete when:** Matching engine produces scored, ranked suggestions.

---

**Reference Documents:**
- [High-Level Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture](../../aiDocs/architecture.md)
- [Phase 6 Roadmap](./2026-02-12_phase-6_matching-engine-roadmap.md)
