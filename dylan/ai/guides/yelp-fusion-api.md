# Yelp Fusion API - Implementation Guide

**Source:** https://docs.developer.yelp.com/docs/fusion-intro  
**Official Docs:** https://docs.developer.yelp.com  
**API Version:** v3 (Yelp Places API)  
**Last Updated:** February 12, 2026  
**Status:** ✅ VERIFIED - Real API, production-ready  
**MVP Status:** ⚠️ POST-MVP (use web scraping first)

---

## Overview

Yelp Fusion API provides access to:
- ✅ 8+ million businesses across 32 countries
- ✅ User reviews and ratings
- ✅ Business hours and photos
- ✅ Search by location, keyword, category
- ✅ Phone search and business matching

**Why we're documenting this (but not using in MVP):**
- **Cost:** $229/month base plan (5K free API calls during 30-day trial)
- **MVP approach:** Use web scraping instead (free, good enough for demo)
- **Post-MVP:** Great for restaurant/local business data

**Base URL:** `https://api.yelp.com/v3`

---

## Authentication

Yelp uses **API Key authentication** (simpler than OAuth):

### 1. Create Yelp Developer Account

1. Go to [yelp.com/developers](https://www.yelp.com/developers)
2. Create an app
3. Copy your API Key

### 2. Use API Key in Requests

```javascript
const axios = require('axios');

const YELP_API_KEY = process.env.YELP_API_KEY;

const headers = {
  'Authorization': `Bearer ${YELP_API_KEY}`
};
```

---

## Key Endpoints

### 1. Business Search (Most Useful for Us)

**Use case:** Find restaurants, activities, entertainment

```javascript
async function searchBusinesses(term, location) {
  const response = await axios.get('https://api.yelp.com/v3/businesses/search', {
    headers: {
      'Authorization': `Bearer ${process.env.YELP_API_KEY}`
    },
    params: {
      term: term,           // "kayaking", "italian restaurant"
      location: location,   // "Austin, TX"
      limit: 10,
      sort_by: 'rating',    // or 'best_match', 'review_count', 'distance'
      price: '1,2,3',       // 1 = $, 2 = $$, 3 = $$$, 4 = $$$$
      open_now: true
    }
  });
  
  return response.data.businesses;
}

// Usage
const businesses = await searchBusinesses('kayaking', 'Austin, TX');
```

**Response:**
```json
{
  "businesses": [
    {
      "id": "north-india-restaurant-san-francisco",
      "name": "Lady Bird Lake Rowing Club",
      "image_url": "https://s3-media1.fl.yelpcdn.com/bphoto/abc.jpg",
      "url": "https://www.yelp.com/biz/lady-bird-lake-rowing-club",
      "rating": 4.5,
      "review_count": 234,
      "price": "$$",
      "phone": "+15125550123",
      "location": {
        "address1": "2418 Stratford Dr",
        "city": "Austin",
        "state": "TX",
        "zip_code": "78746"
      },
      "coordinates": {
        "latitude": 30.2672,
        "longitude": -97.7431
      },
      "categories": [
        {
          "alias": "paddleboarding",
          "title": "Paddleboarding"
        }
      ]
    }
  ],
  "total": 125
}
```

### 2. Business Details

**Use case:** Get full details about a specific business

```javascript
async function getBusinessDetails(businessId) {
  const response = await axios.get(
    `https://api.yelp.com/v3/businesses/${businessId}`,
    {
      headers: {
        'Authorization': `Bearer ${process.env.YELP_API_KEY}`
      }
    }
  );
  
  return response.data;
}
```

**Response includes:**
```json
{
  "id": "north-india-restaurant-san-francisco",
  "name": "Lady Bird Lake Rowing Club",
  "hours": [
    {
      "hours_type": "REGULAR",
      "open": [
        {
          "is_overnight": false,
          "start": "0800",
          "end": "2200",
          "day": 0
        }
      ],
      "is_open_now": true
    }
  ],
  "photos": [
    "https://s3-media1.fl.yelpcdn.com/bphoto/abc.jpg",
    "https://s3-media2.fl.yelpcdn.com/bphoto/def.jpg"
  ],
  "rating": 4.5,
  "price": "$$"
}
```

### 3. Reviews (Up to 3 Excerpts)

```javascript
async function getBusinessReviews(businessId) {
  const response = await axios.get(
    `https://api.yelp.com/v3/businesses/${businessId}/reviews`,
    {
      headers: {
        'Authorization': `Bearer ${process.env.YELP_API_KEY}`
      }
    }
  );
  
  return response.data.reviews;
}
```

**Response:**
```json
{
  "reviews": [
    {
      "id": "abc123",
      "rating": 5,
      "user": {
        "name": "Sarah M.",
        "image_url": "https://s3-media2.fl.yelpcdn.com/photo/xyz.jpg"
      },
      "text": "Amazing kayaking experience! Staff was super friendly...",
      "time_created": "2025-12-15 14:30:00",
      "url": "https://www.yelp.com/biz/abc?hrid=123"
    }
  ]
}
```

### 4. Autocomplete

**Use case:** Help users type activity names

```javascript
async function autocomplete(text, latitude, longitude) {
  const response = await axios.get('https://api.yelp.com/v3/autocomplete', {
    headers: {
      'Authorization': `Bearer ${process.env.YELP_API_KEY}`
    },
    params: {
      text: text,           // "kaya"
      latitude: latitude,
      longitude: longitude
    }
  });
  
  return response.data;
}
```

**Response:**
```json
{
  "terms": [
    { "text": "kayaking" },
    { "text": "kayak rental" }
  ],
  "businesses": [
    {
      "id": "lady-bird-lake-rowing-club",
      "name": "Lady Bird Lake Rowing Club"
    }
  ],
  "categories": [
    {
      "alias": "paddleboarding",
      "title": "Paddleboarding"
    }
  ]
}
```

---

## Useful Categories for Our Project

Yelp has 1,000+ categories. Most relevant for us:

**Dining:**
- `restaurants` - All restaurants
- `italian`, `mexican`, `japanese` - Cuisine types
- `bars` - Bars & pubs
- `coffee` - Coffee shops

**Entertainment:**
- `movietheaters` - Movie theaters
- `bowling` - Bowling alleys
- `arcades` - Arcades
- `museums` - Museums
- `artgalleries` - Art galleries

**Outdoor:**
- `parks` - Parks
- `hiking` - Hiking trails
- `beaches` - Beaches
- `bicycling` - Bike trails
- `paddleboarding` - Water sports

**Family:**
- `amusementparks` - Theme parks
- `zoos` - Zoos
- `aquariums` - Aquariums
- `playgrounds` - Playgrounds

Full list: https://docs.developer.yelp.com/docs/resources-categories

---

## Query Parameters

### Search Parameters

```javascript
{
  term: 'pizza',           // Search term
  location: 'Austin, TX',  // OR use lat/lng
  latitude: 30.2672,       // Precise location
  longitude: -97.7431,
  radius: 10000,           // Meters (max 40,000)
  categories: 'italian,pizza',  // Filter by categories
  price: '1,2',            // Price level(s): 1=$, 2=$$, 3=$$$, 4=$$$$
  open_now: true,          // Only open businesses
  sort_by: 'rating',       // rating, review_count, distance, best_match
  limit: 20                // Results per page (max 50)
}
```

---

## Pricing

**Base Plan:** $229/month
- 5,000 free API calls (30-day trial)
- Additional calls: $0.046 each

**Enhanced Plan:** $549/month
- More data fields
- Higher rate limits

**Example cost for 100 users:**
- 100 users × 20 searches/month = 2,000 API calls
- Within free trial limit
- After trial: 2,000 / 5,000 × $229 = **$91.60/month**

**For MVP:** Too expensive. Use web scraping instead ($0).

---

## Rate Limits

**Base Plan:**
- 25 calls per second
- 5,000 daily calls

**Plenty for our use case** (if we were using it)

---

## Node.js SDK

```bash
npm install yelp-fusion
```

```javascript
const yelp = require('yelp-fusion');

const client = yelp.client(process.env.YELP_API_KEY);

// Search
const response = await client.search({
  term: 'kayaking',
  location: 'Austin, TX',
  limit: 10
});

console.log(response.jsonBody.businesses);

// Business details
const business = await client.business('lady-bird-lake-rowing-club');
console.log(business.jsonBody);

// Reviews
const reviews = await client.reviews('lady-bird-lake-rowing-club');
console.log(reviews.jsonBody.reviews);
```

---

## Alternative: Web Scraping (MVP Approach)

Instead of paying for Yelp API, scrape Yelp:

```javascript
import axios from 'axios';
import * as cheerio from 'cheerio';

async function scrapeYelp(query, location) {
  const searchUrl = `https://www.yelp.com/search?find_desc=${encodeURIComponent(query)}&find_loc=${encodeURIComponent(location)}`;
  
  const { data } = await axios.get(searchUrl, {
    headers: {
      'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
    }
  });
  
  const $ = cheerio.load(data);
  
  const businesses = [];
  
  $('.container__09f24__FeTO6').each((i, element) => {
    const name = $(element).find('h3 a').text();
    const rating = $(element).find('[aria-label*="star rating"]').attr('aria-label');
    const priceCategory = $(element).find('[aria-label="Price range"]').text();
    const address = $(element).find('[data-testid="address"]').text();
    
    if (name) {
      businesses.push({
        name,
        rating: parseFloat(rating),
        priceCategory,
        address,
        source: 'yelp'
      });
    }
  });
  
  return businesses;
}

// Usage
const results = await scrapeYelp('kayaking', 'Austin, TX');
```

**See `aiDocs/mvp.md` for full web scraping implementation.**

---

## Comparison: Google Places vs Yelp

| Feature | Google Places | Yelp |
|---------|--------------|------|
| **Coverage** | Global, all place types | Strong in US/CA/UK, businesses |
| **Best for** | Tourist attractions, parks | Restaurants, bars, local services |
| **Pricing** | $17 per 1K requests | $229/month base |
| **Reviews** | Yes | Yes (better quality) |
| **Photos** | Yes | Yes |
| **Hours** | Yes | Yes |
| **API Ease** | OAuth required | API Key (simpler) |

**Recommendation:**
- **MVP:** Use web scraping for both (free)
- **Post-MVP:** Combine both APIs for best coverage

---

## When to Migrate to Real API (Post-MVP)

**Migrate when:**
1. Generating revenue to cover $229/month
2. Need consistent, reliable data
3. Web scraping breaks
4. Need real-time business hours
5. Want to display Yelp reviews

**Migration strategy:**
1. Start with free trial (5K calls for 30 days)
2. Cache results aggressively (24-48 hours)
3. Only fetch new data when necessary
4. Keep web scraping as fallback

---

## Error Handling

```javascript
try {
  const businesses = await client.search({...});
} catch (error) {
  if (error.statusCode === 400) {
    // Bad request (invalid params)
    console.error('Invalid search parameters');
  } else if (error.statusCode === 401) {
    // Unauthorized (invalid API key)
    console.error('Invalid Yelp API key');
  } else if (error.statusCode === 429) {
    // Rate limit exceeded
    console.error('Rate limit exceeded - slow down');
  } else if (error.statusCode === 403) {
    // Access forbidden (subscription issue)
    console.error('Yelp API subscription issue');
  }
}
```

---

## Best Practices

### 1. Cache Aggressively

```javascript
// Cache business data for 24 hours
const cachedData = await redis.get(`yelp:business:${businessId}`);

if (cachedData) {
  return JSON.parse(cachedData);
}

const data = await client.business(businessId);

await redis.set(
  `yelp:business:${businessId}`,
  JSON.stringify(data.jsonBody),
  'EX',
  86400  // 24 hours
);

return data.jsonBody;
```

### 2. Batch Requests

Don't make sequential API calls - batch them:

```javascript
// ❌ BAD: Sequential
for (const id of businessIds) {
  const business = await client.business(id);
}

// ✅ GOOD: Parallel
const businesses = await Promise.all(
  businessIds.map(id => client.business(id))
);
```

### 3. Use Search Over Business Details

Search is cheaper than fetching individual details:

```javascript
// ✅ GOOD: Get multiple businesses in one call
const response = await client.search({
  term: 'restaurants',
  location: 'Austin, TX',
  limit: 20
});

// ❌ BAD: 20 individual API calls
for (const id of businessIds) {
  const business = await client.business(id);
}
```

---

## Implementation Checklist (Post-MVP)

- [ ] Create Yelp Developer account
- [ ] Create app and get API key
- [ ] Install `yelp-fusion` package
- [ ] Implement business search
- [ ] Cache results in database or Redis
- [ ] Monitor API usage
- [ ] Set up error handling
- [ ] Track API costs

---

## Links & Resources

- **Official Docs:** https://docs.developer.yelp.com
- **Getting Started:** https://docs.developer.yelp.com/docs/fusion-intro
- **API Reference:** https://docs.developer.yelp.com/reference
- **Categories:** https://docs.developer.yelp.com/docs/resources-categories
- **GitHub (yelp-fusion):** https://github.com/Yelp/yelp-fusion
- **Pricing:** https://fusion.yelp.com

---

## For Our MVP

**Decision:** ❌ Skip for MVP, use web scraping

**Reasons:**
1. **Cost:** $229/month is too expensive for demo
2. **Overkill:** Don't need real-time data for MVP
3. **Alternative:** Web scraping is free and good enough

**Post-MVP:** Migrate to real API when:
- Have revenue to cover costs
- Need production-quality data
- Want to display Yelp reviews
- Web scraping becomes unreliable
