# Google Places API (New) - Implementation Guide

**Source:** https://developers.google.com/maps/documentation/places/web-service  
**Official Docs:** https://developers.google.com/maps  
**Last Updated:** February 12, 2026  
**Status:** ✅ VERIFIED - Real API, production-ready  
**MVP Status:** ⚠️ POST-MVP (use web scraping first)

---

## Overview

Google Places API (New) allows you to:
- ✅ Search for places by text query ("restaurants near me")
- ✅ Search nearby places within a radius
- ✅ Get detailed place information (hours, ratings, reviews)
- ✅ Access high-quality photos
- ✅ Get autocomplete suggestions

**Why we're documenting this (but not using in MVP):**
- **Cost:** Starts at $17 per 1,000 requests (Text Search)
- **MVP approach:** Use web scraping instead (free, good enough for demo)
- **Post-MVP:** Integrate real API for production quality data

**Base URL:** `https://places.googleapis.com/v1`

---

## Authentication

Same as Google Calendar API - use OAuth 2.0 and API Key:

```javascript
import { google } from 'googleapis';

// Enable Places API in Google Cloud Console
// Create API Key with Places API restrictions

const apiKey = process.env.GOOGLE_PLACES_API_KEY;
```

---

## Key APIs

### 1. Text Search (Most Useful for Us)

**Use case:** Find activities matching user interests

```http
POST https://places.googleapis.com/v1/places:searchText
Content-Type: application/json
X-Goog-Api-Key: YOUR_API_KEY
X-Goog-FieldMask: places.displayName,places.formattedAddress,places.rating,places.priceLevel,places.googleMapsUri

{
  "textQuery": "kayaking near Austin, TX",
  "includedType": "tourist_attraction",
  "maxResultCount": 10,
  "rankPreference": "POPULARITY",
  "locationBias": {
    "circle": {
      "center": {
        "latitude": 30.2672,
        "longitude": -97.7431
      },
      "radius": 10000.0
    }
  }
}
```

**Response:**
```json
{
  "places": [
    {
      "displayName": {
        "text": "Lady Bird Lake Rowing Club",
        "languageCode": "en"
      },
      "formattedAddress": "2418 Stratford Dr, Austin, TX 78746",
      "rating": 4.7,
      "priceLevel": "PRICE_LEVEL_MODERATE",
      "googleMapsUri": "https://maps.google.com/?cid=12345"
    }
  ]
}
```

### 2. Nearby Search

**Use case:** Find activities within a specific area

```http
POST https://places.googleapis.com/v1/places:searchNearby
Content-Type: application/json
X-Goog-Api-Key: YOUR_API_KEY
X-Goog-FieldMask: places.displayName,places.formattedAddress

{
  "includedTypes": ["restaurant", "park"],
  "maxResultCount": 10,
  "locationRestriction": {
    "circle": {
      "center": {
        "latitude": 30.2672,
        "longitude": -97.7431
      },
      "radius": 5000.0
    }
  }
}
```

### 3. Place Details

**Use case:** Get full details about a specific place

```http
GET https://places.googleapis.com/v1/places/{place_id}
X-Goog-Api-Key: YOUR_API_KEY
X-Goog-FieldMask: displayName,formattedAddress,nationalPhoneNumber,rating,regularOpeningHours,priceLevel
```

**Response:**
```json
{
  "displayName": { "text": "Lady Bird Lake Rowing Club" },
  "formattedAddress": "2418 Stratford Dr, Austin, TX 78746",
  "nationalPhoneNumber": "(512) 555-0123",
  "rating": 4.7,
  "priceLevel": "PRICE_LEVEL_MODERATE",
  "regularOpeningHours": {
    "openNow": true,
    "periods": [
      {
        "open": { "day": 0, "hour": 8, "minute": 0 },
        "close": { "day": 0, "hour": 18, "minute": 0 }
      }
    ]
  }
}
```

---

## Place Types (Categories)

Useful types for our project:
- `restaurant` - Dining
- `cafe` - Coffee shops
- `bar` - Bars & pubs
- `park` - Parks
- `museum` - Museums
- `art_gallery` - Art galleries
- `tourist_attraction` - Tourist spots
- `amusement_park` - Family fun
- `movie_theater` - Entertainment
- `bowling_alley` - Activities
- `shopping_mall` - Shopping

Full list: https://developers.google.com/maps/documentation/places/web-service/place-types

---

## Pricing

**Text Search:** $17 per 1,000 requests  
**Nearby Search:** $17 per 1,000 requests  
**Place Details (Basic):** $17 per 1,000 requests  
**Place Details (Contact):** $3 per 1,000 requests  
**Place Details (Atmosphere):** $5 per 1,000 requests  
**Place Photos:** $7 per 1,000 requests

**Example cost for 100 users:**
- 100 users × 10 searches/month = 1,000 searches
- Cost: 1,000 / 1,000 × $17 = **$17/month**

**For MVP:** Too expensive. Use web scraping instead ($0).

---

## Node.js Client Library

```bash
npm install @googlemaps/places
```

```typescript
import { PlacesClient } from '@googlemaps/places';

const client = new PlacesClient({
  apiKey: process.env.GOOGLE_PLACES_API_KEY
});

// Text search
const response = await client.searchText({
  textQuery: 'kayaking near Austin',
  maxResultCount: 10
});

console.log(response.places);
```

---

## Field Masks (Cost Optimization)

**Only request fields you need** to reduce costs:

```typescript
// ✅ GOOD: Only basic fields
const response = await client.searchText({
  textQuery: 'restaurants',
  fieldMask: 'places.displayName,places.formattedAddress,places.rating'
});

// ❌ BAD: All fields (expensive)
const response = await client.searchText({
  textQuery: 'restaurants'
  // No field mask = returns everything
});
```

**Available field masks:**
- Basic: `displayName`, `formattedAddress`, `location`, `googleMapsUri`
- Advanced: `rating`, `userRatingCount`, `priceLevel`, `types`
- Contact: `nationalPhoneNumber`, `internationalPhoneNumber`, `websiteUri`
- Atmosphere: `photos`, `reviews`, `regularOpeningHours`

---

## Alternative: Web Scraping (MVP Approach)

Instead of paying for Google Places API, scrape Google Maps:

```javascript
import axios from 'axios';
import * as cheerio from 'cheerio';

async function scrapeGoogleMaps(query, location) {
  const searchUrl = `https://www.google.com/maps/search/${encodeURIComponent(query)}+${encodeURIComponent(location)}`;
  
  const { data } = await axios.get(searchUrl, {
    headers: {
      'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    }
  });
  
  const $ = cheerio.load(data);
  
  const activities = [];
  
  // Parse results
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

// Usage
const results = await scrapeGoogleMaps('kayaking', 'Austin, TX');
```

**See `aiDocs/mvp.md` for full web scraping implementation.**

---

## When to Migrate to Real API (Post-MVP)

**Migrate when:**
1. You have paying customers (revenue covers API costs)
2. Web scraping breaks (Google changes HTML structure)
3. You need real-time data (hours, photos, reviews)
4. You need consistent, reliable data quality

**Migration path:**
1. Create background job to fetch activity data via API
2. Store in database (cache for 24 hours)
3. Gradually replace scraped data with API data
4. Keep scraping as fallback

---

## Alternative: Yelp Fusion API

**Cost:** $229/month base plan  
**Better for:** Restaurants, bars, local businesses  
**See:** `ai/guides/yelp-fusion-api.md`

---

## Implementation Checklist (Post-MVP)

- [ ] Enable Google Places API in Cloud Console
- [ ] Create API key with Places restrictions
- [ ] Set up billing (requires credit card)
- [ ] Install `@googlemaps/places` package
- [ ] Implement Text Search for activity discovery
- [ ] Cache results in database (24-48 hours)
- [ ] Monitor API usage and costs
- [ ] Set usage limits to avoid unexpected charges

---

## Links & Resources

- **Official Docs:** https://developers.google.com/maps/documentation/places/web-service
- **Place Types:** https://developers.google.com/maps/documentation/places/web-service/place-types
- **Node.js Client:** https://github.com/googlemaps/google-maps-services-js
- **Pricing:** https://mapsplatform.google.com/pricing/

---

## For Our MVP

**Decision:** ❌ Skip for MVP, use web scraping

**Reasons:**
1. **Cost:** $17 per 1K requests is too expensive for demo
2. **Overkill:** Don't need real-time data for MVP
3. **Alternative:** Web scraping is free and good enough

**Post-MVP:** Migrate to real API when:
- Have revenue
- Need production-quality data
- Web scraping becomes unreliable
