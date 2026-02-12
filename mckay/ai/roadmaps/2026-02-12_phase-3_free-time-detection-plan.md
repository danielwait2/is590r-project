# Phase 3: Free Time Detection - Implementation Plan

**Created:** 2026-02-12
**Phase:** 3 of 10
**Timeline:** Week 3-4
**Companion Document:** [Phase 3 Roadmap](./2026-02-12_phase-3_free-time-detection-roadmap.md)

---

## Project Principles

⚠️ **Build the simplest algorithm that works.** Don't over-engineer. Weekend filtering is core requirement.

---

## Phase Overview

**Goal:** Calculate overlapping free time between both partners' calendars

**Key deliverables:**
- Algorithm to find free windows from busy blocks
- Algorithm to find overlapping windows between partners
- Filter by preferences (weekends only, min duration, advance notice)
- Test endpoint to visualize free time

---

## Technical Specifications

### 1. Free Time Algorithm

**Add to:** `server/services/calendar.js`

#### findFreeWindows(busyBlocks, dateRange, preferences)
```javascript
/**
 * Finds free time windows given busy blocks
 * @param {Array} busyBlocks - Sorted busy blocks [{start, end}, ...]
 * @param {object} dateRange - {start, end} ISO dates
 * @param {object} preferences - Filtering preferences
 * @returns {Array} freeWindows - Available time slots
 */
```

**Algorithm:**
1. Create timeline from start to end of date range
2. Mark busy blocks as unavailable
3. Find gaps (free windows)
4. Filter by day of week (weekend only: Fri 6pm-Sun 11pm)
5. Filter by minimum duration (default 2 hours)
6. Filter by advance notice (default 3 days from now)
7. Return sorted array of free windows

#### findOverlappingFreeTime(partner1Free, partner2Free)
```javascript
/**
 * Finds time windows when BOTH partners are free
 * @param {Array} partner1Free - Partner 1 free windows
 * @param {Array} partner2Free - Partner 2 free windows
 * @returns {Array} overlappingWindows - Windows when both free
 */
```

**Algorithm:**
1. For each window in partner1Free
2. For each window in partner2Free
3. Find intersection (overlapping time)
4. If overlap exists and meets min duration, add to results
5. Return overlapping windows

#### getAvailableTimeForCouple(coupleId, preferences)
```javascript
/**
 * Main function: Gets available time windows for a couple
 * @param {string} coupleId - Couple identifier
 * @param {object} preferences - User preferences (or defaults)
 * @returns {Promise<Array>} availableWindows - When both partners free
 */
```

**Implementation:**
1. Load preferences (or use defaults)
2. Calculate date range (next 2 weeks)
3. Get busy blocks for both partners
4. Find free windows for each partner
5. Find overlapping free windows
6. Apply filters (weekend, duration, advance notice)
7. Return sorted list

---

### 2. Weekend Filtering

**Weekend definition (per MVP spec):**
- Friday: 6:00 PM - 11:59 PM
- Saturday: 12:00 AM - 11:59 PM
- Sunday: 12:00 AM - 11:00 PM

**Implementation:**
```javascript
function isWeekendTime(dateTime) {
  const date = new Date(dateTime);
  const day = date.getDay(); // 0=Sun, 5=Fri, 6=Sat
  const hour = date.getHours();

  if (day === 5) { // Friday
    return hour >= 18; // 6pm or later
  }
  if (day === 6) { // Saturday
    return true; // All day
  }
  if (day === 0) { // Sunday
    return hour < 23; // Before 11pm
  }
  return false;
}

function filterWeekendWindows(freeWindows) {
  return freeWindows.filter(window => {
    return isWeekendTime(window.start) && isWeekendTime(window.end);
  });
}
```

---

### 3. Preference Filters

**Default preferences (if not set):**
```javascript
const DEFAULT_PREFERENCES = {
  minDuration: 120, // minutes (2 hours)
  advanceNotice: 3, // days
  weekendOnly: true,
  maxDistance: 15 // miles (for activity matching in Phase 6)
};
```

**Duration filter:**
```javascript
function filterByDuration(windows, minDurationMinutes) {
  return windows.filter(window => {
    const durationMs = new Date(window.end) - new Date(window.start);
    const durationMinutes = durationMs / (1000 * 60);
    return durationMinutes >= minDurationMinutes;
  });
}
```

**Advance notice filter:**
```javascript
function filterByAdvanceNotice(windows, advanceNoticeDays) {
  const cutoffDate = new Date();
  cutoffDate.setDate(cutoffDate.getDate() + advanceNoticeDays);

  return windows.filter(window => {
    return new Date(window.start) >= cutoffDate;
  });
}
```

---

### 4. Test Endpoint

**Route: GET /calendar/:coupleId/free**

```javascript
router.get('/:coupleId/free', async (req, res) => {
  try {
    const { coupleId } = req.params;

    // Load preferences or use defaults
    const preferences = await preferencesService.getPreferences(coupleId)
      || DEFAULT_PREFERENCES;

    const freeWindows = await calendarService.getAvailableTimeForCouple(
      coupleId,
      preferences
    );

    res.json({
      coupleId,
      windowsFound: freeWindows.length,
      preferences: {
        minDuration: preferences.minDuration,
        advanceNotice: preferences.advanceNotice,
        weekendOnly: preferences.weekendOnly
      },
      freeWindows
    });
  } catch (error) {
    console.error('Error finding free time:', error);
    res.status(500).json({ error: error.message });
  }
});
```

---

## Data Structures

### Free Window Object
```javascript
{
  start: '2026-02-22T14:00:00-06:00',  // ISO 8601
  end: '2026-02-22T17:00:00-06:00',    // ISO 8601
  durationMinutes: 180,
  dayOfWeek: 'Saturday',
  isWeekend: true
}
```

---

## Testing Criteria

- [ ] Find free windows from busy blocks
- [ ] Find overlapping windows between partners
- [ ] Weekend filter works correctly (Fri 6pm-Sun 11pm)
- [ ] Duration filter works (min 2 hours)
- [ ] Advance notice filter works (3+ days out)
- [ ] Test endpoint returns correct data
- [ ] Empty result when no free time (doesn't crash)
- [ ] Works with various calendar states

---

## Common Issues & Solutions

### Issue: Time zone confusion
**Solution:** Keep all times in ISO 8601 with timezone. JavaScript Date handles this.

### Issue: Edge cases at midnight
**Solution:** Treat windows that cross midnight as separate days.

### Issue: No overlapping free time
**Solution:** Return empty array, show friendly message in UI.

---

## Next Steps

After Phase 3:
- Verify free time detection works
- Test with various calendar scenarios
- Move to [Phase 4: Preferences Collection](./2026-02-12_phase-4_preferences-collection-plan.md)

**Phase 3 complete when:** Can find overlapping free time windows that meet all criteria.

---

**Reference Documents:**
- [High-Level Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture](../../aiDocs/architecture.md)
- [MVP Spec](../../aiDocs/mvp.md)
- [Phase 3 Roadmap](./2026-02-12_phase-3_free-time-detection-roadmap.md)
