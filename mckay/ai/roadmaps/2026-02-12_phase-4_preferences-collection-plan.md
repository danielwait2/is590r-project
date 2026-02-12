# Phase 4: Preferences Collection - Implementation Plan

**Created:** 2026-02-12
**Phase:** 4 of 10
**Timeline:** Week 4
**Companion Document:** [Phase 4 Roadmap](./2026-02-12_phase-4_preferences-collection-roadmap.md)

---

## Project Principles

⚠️ **Keep forms simple.** Capture what's needed for matching, nothing more.

---

## Phase Overview

**Goal:** Create web form to capture couple preferences for activity matching

**Key deliverables:**
- HTML preferences form
- Routes to save/retrieve preferences
- Preferences service module
- Data validation
- Location geocoding (address to lat/lng)

---

## Technical Specifications

### 1. Preferences Data Structure

**File:** `server/data/preferences.json`

```json
{
  "couple-1": {
    "interests": ["dining", "arts", "classes"],
    "wantToTry": [
      "pottery class",
      "Italian restaurant downtown",
      "wine tasting"
    ],
    "location": {
      "address": "123 Main St, Austin, TX 78701",
      "lat": 30.2672,
      "lng": -97.7431
    },
    "availability": {
      "days": ["friday_evening", "saturday", "sunday"],
      "minDuration": 120,
      "advanceNotice": 3
    },
    "budget": "$$$",
    "updatedAt": "2026-02-12T10:00:00Z"
  }
}
```

---

### 2. Preferences HTML Form

**File:** `public/preferences.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Set Preferences - Calendar Concierge</title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <div class="container">
        <h1>Set Couple Preferences</h1>
        <p>Tell us what you like to do together!</p>

        <form id="preferencesForm">
            <section>
                <h2>Interests</h2>
                <p>Select categories you enjoy:</p>
                <label><input type="checkbox" name="interests" value="dining"> Dining Out</label>
                <label><input type="checkbox" name="interests" value="arts"> Arts & Culture</label>
                <label><input type="checkbox" name="interests" value="music"> Live Music</label>
                <label><input type="checkbox" name="interests" value="classes"> Classes & Workshops</label>
                <label><input type="checkbox" name="interests" value="outdoors"> Outdoor Activities</label>
                <label><input type="checkbox" name="interests" value="sports"> Sports & Fitness</label>
                <label><input type="checkbox" name="interests" value="entertainment"> Entertainment</label>
            </section>

            <section>
                <h2>Want to Try List</h2>
                <p>Things you've been wanting to do (one per line):</p>
                <textarea name="wantToTry" rows="5" placeholder="pottery class&#10;Italian restaurant downtown&#10;wine tasting&#10;..."></textarea>
            </section>

            <section>
                <h2>Location</h2>
                <label>
                    Your home address or general area:
                    <input type="text" name="address" placeholder="123 Main St, Austin, TX 78701" required>
                </label>
                <p class="note">Used to find nearby activities</p>
            </section>

            <section>
                <h2>Availability</h2>
                <label>
                    Minimum activity duration (minutes):
                    <input type="number" name="minDuration" value="120" min="30" step="30">
                </label>
                <label>
                    Advance notice needed (days):
                    <input type="number" name="advanceNotice" value="3" min="1" max="14">
                </label>
            </section>

            <section>
                <h2>Budget</h2>
                <label>
                    Price level:
                    <select name="budget">
                        <option value="$">$ - Budget Friendly</option>
                        <option value="$$">$$ - Moderate</option>
                        <option value="$$$" selected>$$$ - Mid-Range</option>
                        <option value="$$$$">$$$$ - Premium</option>
                    </select>
                </label>
            </section>

            <button type="submit" class="btn btn-primary">Save Preferences</button>
        </form>

        <div id="result" class="result-box" style="display:none;"></div>
        <p><a href="/">Back to home</a></p>
    </div>

    <script>
        const form = document.getElementById('preferencesForm');
        const resultDiv = document.getElementById('result');

        form.addEventListener('submit', async (e) => {
            e.preventDefault();

            // Collect form data
            const formData = new FormData(form);
            const interests = formData.getAll('interests');
            const wantToTry = formData.get('wantToTry')
                .split('\n')
                .map(item => item.trim())
                .filter(item => item.length > 0);

            const preferences = {
                interests,
                wantToTry,
                location: { address: formData.get('address') },
                availability: {
                    days: ['friday_evening', 'saturday', 'sunday'],
                    minDuration: parseInt(formData.get('minDuration')),
                    advanceNotice: parseInt(formData.get('advanceNotice'))
                },
                budget: formData.get('budget')
            };

            // Submit to server
            try {
                const response = await fetch('/preferences/couple-1', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(preferences)
                });

                const data = await response.json();

                resultDiv.style.display = 'block';
                if (response.ok) {
                    resultDiv.innerHTML = '<p class="success">✓ Preferences saved successfully!</p>';
                    resultDiv.className = 'result-box success';
                } else {
                    resultDiv.innerHTML = `<p class="error">Error: ${data.error}</p>`;
                    resultDiv.className = 'result-box error';
                }
            } catch (error) {
                resultDiv.style.display = 'block';
                resultDiv.innerHTML = `<p class="error">Failed to save: ${error.message}</p>`;
                resultDiv.className = 'result-box error';
            }
        });
    </script>
</body>
</html>
```

---

### 3. Preferences Service

**File:** `server/services/preferences.js`

```javascript
const fs = require('fs').promises;
const path = require('path');

const PREFERENCES_FILE = path.join(__dirname, '../data/preferences.json');

async function getPreferences(coupleId) {
  try {
    const data = await fs.readFile(PREFERENCES_FILE, 'utf8');
    const allPreferences = JSON.parse(data);
    return allPreferences[coupleId] || null;
  } catch (error) {
    if (error.code === 'ENOENT') return null;
    throw error;
  }
}

async function savePreferences(coupleId, preferences) {
  // Add geocoded location
  if (preferences.location && preferences.location.address) {
    const coords = await geocodeAddress(preferences.location.address);
    preferences.location.lat = coords.lat;
    preferences.location.lng = coords.lng;
  }

  // Add timestamp
  preferences.updatedAt = new Date().toISOString();

  // Load existing preferences
  let allPreferences = {};
  try {
    const data = await fs.readFile(PREFERENCES_FILE, 'utf8');
    allPreferences = JSON.parse(data);
  } catch (error) {
    if (error.code !== 'ENOENT') throw error;
  }

  // Update
  allPreferences[coupleId] = preferences;

  // Save
  await fs.writeFile(PREFERENCES_FILE, JSON.stringify(allPreferences, null, 2));

  return preferences;
}

// Simple geocoding (mock for MVP, use real API in production)
async function geocodeAddress(address) {
  // For MVP: Use hardcoded coords or simple lookup
  // For production: Use Google Geocoding API or similar

  // Mock implementation - Austin, TX coords
  return {
    lat: 30.2672,
    lng: -97.7431
  };

  // Real implementation would call geocoding API:
  // const response = await fetch(`https://maps.googleapis.com/maps/api/geocode/json?address=${encodeURIComponent(address)}&key=${API_KEY}`);
  // const data = await response.json();
  // return data.results[0].geometry.location;
}

module.exports = {
  getPreferences,
  savePreferences
};
```

---

### 4. Preferences Routes

**File:** `server/routes/preferences.js`

```javascript
const express = require('express');
const router = express.Router();
const preferencesService = require('../services/preferences');

// Get preferences form (HTML)
router.get('/:coupleId', (req, res) => {
  res.sendFile('preferences.html', { root: 'public' });
});

// Get preferences data (JSON)
router.get('/:coupleId/data', async (req, res) => {
  try {
    const { coupleId } = req.params;
    const preferences = await preferencesService.getPreferences(coupleId);

    if (!preferences) {
      return res.status(404).json({ error: 'Preferences not found' });
    }

    res.json(preferences);
  } catch (error) {
    console.error('Error getting preferences:', error);
    res.status(500).json({ error: error.message });
  }
});

// Save preferences
router.post('/:coupleId', async (req, res) => {
  try {
    const { coupleId } = req.params;
    const preferences = req.body;

    // Basic validation
    if (!preferences.interests || preferences.interests.length === 0) {
      return res.status(400).json({ error: 'At least one interest required' });
    }
    if (!preferences.location || !preferences.location.address) {
      return res.status(400).json({ error: 'Location address required' });
    }

    const saved = await preferencesService.savePreferences(coupleId, preferences);
    res.json({ success: true, preferences: saved });
  } catch (error) {
    console.error('Error saving preferences:', error);
    res.status(500).json({ error: error.message });
  }
});

module.exports = router;
```

---

## Testing Criteria

- [ ] Form loads at /preferences/couple-1
- [ ] Can fill out form fields
- [ ] Form validation works (required fields)
- [ ] Can submit form
- [ ] Preferences saved to JSON file
- [ ] Can retrieve preferences via GET
- [ ] Location geocoded to lat/lng
- [ ] Want-to-try list parsed correctly
- [ ] Interests checkboxes work

---

## Next Steps

After Phase 4:
- Verify preferences save/load works
- Test form with various inputs
- Move to [Phase 5: Activity Database](./2026-02-12_phase-5_activity-database-plan.md)

**Phase 4 complete when:** Can save and retrieve preferences, ready for matching.

---

**Reference Documents:**
- [High-Level Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture](../../aiDocs/architecture.md)
- [Phase 4 Roadmap](./2026-02-12_phase-4_preferences-collection-roadmap.md)
