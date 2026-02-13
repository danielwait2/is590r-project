# Phase 3: Activity Discovery - Detailed Implementation Plan

**Timeline:** Week 4 (7 days)  
**Status:** Not Started  
**Roadmap:** [2026-02-12_roadmap-phase3.md](./2026-02-12_roadmap-phase3.md) - Track progress

---

## Overview

Scrape activity data from Google Maps and Yelp to populate the database with 100-150 activities in Austin, TX. Build basic activity browsing UI. This provides the content for Phase 4's suggestion engine.

**What gets built:**
- Web scraping scripts (Cheerio + Axios)
- Activity data pipeline (scrape → clean → store)
- 100-150 curated activities for Austin
- Activity browsing UI in dashboard
- Admin script to run scrapers

---

## Success Criteria

- [ ] Database contains 100-150 activities for Austin, TX
- [ ] Activities include: name, address, category, rating
- [ ] At least 10 different categories (dining, outdoor, entertainment, etc.)
- [ ] User can browse activities in dashboard
- [ ] Activities can be filtered by category
- [ ] Data quality is good (no duplicates, valid addresses)
- [ ] Scraping scripts are documented and reusable

---

## Prerequisites

**From Phase 2:**
- [ ] App deployed and working
- [ ] Database schema includes `activities` table
- [ ] User can sign in and see dashboard

**New Setup:**
- [ ] Install Cheerio and Axios
- [ ] Understand web scraping basics

---

## Detailed Implementation Steps

### Day 1: Install Dependencies & Set Up Scrapers

#### 1. Install Scraping Libraries

```bash
npm install cheerio axios
npm install --save-dev @types/cheerio
```

#### 2. Create Scraper Directory Structure

```
scripts/
├── scrapers/
│   ├── google-maps.ts      # Google Maps scraper
│   ├── yelp.ts              # Yelp scraper
│   ├── utils.ts             # Shared utilities
│   └── types.ts             # TypeScript types
├── seed-activities.ts       # Main script to run all scrapers
└── clean-activities.ts      # Clean and deduplicate data
```

#### 3. Define Activity Type

`scripts/scrapers/types.ts`:

```typescript
export interface ScrapedActivity {
  name: string
  category: string
  address?: string
  city: string
  state: string
  lat?: number
  lng?: number
  rating?: number
  priceLevel?: number  // 1-4
  requiresBabysitter: boolean
  scrapedFrom: 'google_maps' | 'yelp' | 'manual'
}

export interface Category {
  name: string
  query: string
  requiresBabysitter: boolean
}
```

---

### Day 1-2: Build Google Maps Scraper

#### 1. Create Scraper Utility

`scripts/scrapers/utils.ts`:

```typescript
import axios from 'axios'

export async function fetchHTML(url: string): Promise<string> {
  const response = await axios.get(url, {
    headers: {
      'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36',
      'Accept': 'text/html,application/xhtml+xml,application/xml',
      'Accept-Language': 'en-US,en;q=0.9',
    },
  })
  return response.data
}

export function delay(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms))
}

export function normalizePhone(phone: string): string {
  // Remove non-digits
  const digits = phone.replace(/\D/g, '')
  
  // Add US country code if missing
  if (digits.length === 10) {
    return `+1${digits}`
  }
  
  return phone
}
```

#### 2. Create Google Maps Scraper

`scripts/scrapers/google-maps.ts`:

**Approach:** Since Google Maps HTML structure is complex and changes frequently, use a simpler approach:

1. Search Google Maps URLs manually for each category
2. Use Puppeteer (headless browser) for dynamic content
3. Or: Manually curate from Google Maps (copy-paste into spreadsheet)

**For MVP:** Start with manual curation, build scraper if time permits.

**Categories to cover:**
- Restaurants (Italian, Mexican, American, BBQ, Sushi)
- Outdoor (Parks, Hiking trails, Kayaking, Paddleboarding)
- Entertainment (Museums, Art galleries, Movie theaters, Bowling)
- Family (Playgrounds, Zoos, Aquariums, Trampoline parks)

#### 3. Manual Curation Template

Create `scripts/activities-manual.json`:

```typescript
[
  {
    "name": "Lady Bird Lake Rowing Club",
    "category": "outdoor",
    "address": "2418 Stratford Dr, Austin, TX 78746",
    "city": "Austin",
    "state": "TX",
    "lat": 30.2672,
    "lng": -97.7431,
    "rating": 4.7,
    "priceLevel": 2,
    "requiresBabysitter": true,
    "scrapedFrom": "manual"
  },
  {
    "name": "Uchi",
    "category": "dining",
    "address": "801 S Lamar Blvd, Austin, TX 78704",
    "city": "Austin",
    "state": "TX",
    "rating": 4.6,
    "priceLevel": 3,
    "requiresBabysitter": true,
    "scrapedFrom": "manual"
  }
  // ... 100-150 more activities
]
```

---

### Day 2-3: Build Yelp Scraper (Alternative to API)

#### 1. Create Yelp Scraper

`scripts/scrapers/yelp.ts`:

```typescript
import * as cheerio from 'cheerio'
import { fetchHTML, delay } from './utils'
import { ScrapedActivity } from './types'

export async function scrapeYelp(
  query: string,
  location: string,
  limit: number = 20
): Promise<ScrapedActivity[]> {
  const activities: ScrapedActivity[] = []
  
  const searchUrl = `https://www.yelp.com/search?find_desc=${encodeURIComponent(query)}&find_loc=${encodeURIComponent(location)}`
  
  console.log(`Scraping Yelp: ${query} in ${location}`)
  
  try {
    const html = await fetchHTML(searchUrl)
    const $ = cheerio.load(html)
    
    // Yelp's HTML structure (subject to change)
    $('[data-testid="serp-ia-card"]').each((i, element) => {
      if (i >= limit) return false  // Stop at limit
      
      const $el = $(element)
      
      const name = $el.find('h3 a').text().trim()
      const rating = $el.find('[aria-label*="star rating"]').attr('aria-label')
      const priceText = $el.find('[aria-label="Price range"]').text().trim()
      const address = $el.find('[data-testid="address"]').text().trim()
      
      if (name) {
        activities.push({
          name,
          category: query.toLowerCase().replace(/\s+/g, '_'),
          address,
          city: 'Austin',
          state: 'TX',
          rating: rating ? parseFloat(rating) : undefined,
          priceLevel: priceText ? priceText.length : 2,
          requiresBabysitter: true,  // Default for restaurants
          scrapedFrom: 'yelp',
        })
      }
    })
    
    console.log(`Scraped ${activities.length} activities from Yelp`)
    return activities
  } catch (error) {
    console.error(`Yelp scraping error for "${query}":`, error)
    return []
  }
}
```

**Note:** Web scraping is fragile. HTML selectors may break. Plan to manually curate if scraping fails.

---

### Day 3-4: Build Seed Script

#### 1. Create Main Seed Script

`scripts/seed-activities.ts`:

```typescript
import { createClient } from '@supabase/supabase-js'
import { scrapeYelp } from './scrapers/yelp'
import { delay } from './scrapers/utils'
import manualActivities from './activities-manual.json'

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!  // Use service role for admin operations
)

const CATEGORIES = [
  { name: 'Italian Restaurant', query: 'italian restaurant', requiresBabysitter: true },
  { name: 'Mexican Restaurant', query: 'mexican restaurant', requiresBabysitter: true },
  { name: 'Coffee Shop', query: 'coffee shop', requiresBabysitter: false },
  { name: 'Park', query: 'park', requiresBabysitter: false },
  { name: 'Hiking Trail', query: 'hiking trail', requiresBabysitter: false },
  { name: 'Museum', query: 'museum', requiresBabysitter: false },
  { name: 'Bowling Alley', query: 'bowling', requiresBabysitter: false },
  { name: 'Movie Theater', query: 'movie theater', requiresBabysitter: true },
  { name: 'BBQ Restaurant', query: 'bbq restaurant', requiresBabysitter: true },
  { name: 'Kayaking', query: 'kayaking rental', requiresBabysitter: true },
]

async function seedActivities() {
  console.log('Starting activity seeding...')
  
  const allActivities: any[] = []
  
  // 1. Add manual activities first
  console.log(`Adding ${manualActivities.length} manual activities...`)
  allActivities.push(...manualActivities)
  
  // 2. Scrape Yelp for each category
  for (const category of CATEGORIES) {
    console.log(`Scraping: ${category.name}...`)
    
    try {
      const activities = await scrapeYelp(category.query, 'Austin, TX', 10)
      allActivities.push(...activities)
      
      // Be polite to Yelp - delay between requests
      await delay(2000)
    } catch (error) {
      console.error(`Failed to scrape ${category.name}:`, error)
    }
  }
  
  // 3. Remove duplicates by name
  const uniqueActivities = Array.from(
    new Map(allActivities.map(a => [a.name.toLowerCase(), a])).values()
  )
  
  console.log(`Total unique activities: ${uniqueActivities.length}`)
  
  // 4. Insert into database
  const { data, error } = await supabase
    .from('activities')
    .insert(uniqueActivities)
    .select()
  
  if (error) {
    console.error('Error inserting activities:', error)
    throw error
  }
  
  console.log(`✅ Successfully seeded ${data.length} activities`)
}

seedActivities()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error('Seeding failed:', error)
    process.exit(1)
  })
```

#### 2. Add Script to package.json

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "seed": "tsx scripts/seed-activities.ts"
  }
}
```

Install `tsx` for running TypeScript scripts:
```bash
npm install --save-dev tsx
```

#### 3. Run Seed Script

```bash
npm run seed
```

---

### Day 4-5: Build Activity Browsing UI

#### 1. Create Activities Page

`app/dashboard/activities/page.tsx`:

```typescript
import { createClient } from '@/lib/supabase/server'

export default async function ActivitiesPage() {
  const supabase = createClient()

  const { data: activities } = await supabase
    .from('activities')
    .select('*')
    .eq('city', 'Austin')
    .order('rating', { ascending: false })
    .limit(50)

  // Group by category
  const byCategory = activities?.reduce((acc, activity) => {
    if (!acc[activity.category]) {
      acc[activity.category] = []
    }
    acc[activity.category].push(activity)
    return acc
  }, {} as Record<string, any[]>)

  return (
    <div className="space-y-8">
      <div>
        <h2 className="text-2xl font-bold mb-2">Activities in Austin</h2>
        <p className="text-gray-600">
          {activities?.length || 0} activities available
        </p>
      </div>

      {Object.entries(byCategory || {}).map(([category, items]) => (
        <div key={category} className="space-y-4">
          <h3 className="text-lg font-semibold capitalize">
            {category.replace(/_/g, ' ')}
          </h3>
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            {items.map((activity) => (
              <div key={activity.id} className="bg-white border rounded-lg p-4 hover:shadow-md transition">
                <h4 className="font-medium text-gray-900 mb-2">
                  {activity.name}
                </h4>
                <p className="text-sm text-gray-600 mb-2">
                  {activity.address}
                </p>
                <div className="flex items-center gap-3 text-sm">
                  {activity.rating && (
                    <span className="flex items-center gap-1">
                      ⭐ {activity.rating}
                    </span>
                  )}
                  {activity.price_level && (
                    <span className="text-gray-500">
                      {'$'.repeat(activity.price_level)}
                    </span>
                  )}
                </div>
              </div>
            ))}
          </div>
        </div>
      ))}
    </div>
  )
}
```

#### 2. Add Navigation to Dashboard Layout

Update `app/dashboard/layout.tsx` to include navigation:

```typescript
<nav className="flex gap-6 text-sm">
  <Link href="/dashboard" className="text-gray-600 hover:text-gray-900">
    Home
  </Link>
  <Link href="/dashboard/activities" className="text-gray-600 hover:text-gray-900">
    Activities
  </Link>
</nav>
```

---

### Day 5-6: Manual Curation & Data Cleaning

#### 1. Manually Curate Top Activities

For MVP, supplement scraped data with manually curated activities from:
- Google Maps searches ("best restaurants Austin", "things to do Austin")
- Yelp top-rated lists
- Local blogs and guides

**Target distribution:**
- 30-40 dining (restaurants, cafes, bars)
- 20-30 outdoor (parks, trails, water activities)
- 15-20 entertainment (museums, theaters, bowling)
- 15-20 family (playgrounds, kid activities)
- 10-15 misc (shopping, experiences)

#### 2. Create Cleaning Script

`scripts/clean-activities.ts`:

```typescript
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)

async function cleanActivities() {
  // 1. Fetch all activities
  const { data: activities } = await supabase
    .from('activities')
    .select('*')
  
  console.log(`Total activities before cleaning: ${activities?.length}`)
  
  // 2. Find duplicates (same name, case-insensitive)
  const seen = new Set<string>()
  const duplicates: string[] = []
  
  activities?.forEach(activity => {
    const key = activity.name.toLowerCase().trim()
    if (seen.has(key)) {
      duplicates.push(activity.id)
    } else {
      seen.add(key)
    }
  })
  
  // 3. Delete duplicates
  if (duplicates.length > 0) {
    console.log(`Removing ${duplicates.length} duplicates...`)
    const { error } = await supabase
      .from('activities')
      .delete()
      .in('id', duplicates)
    
    if (error) console.error('Error deleting duplicates:', error)
  }
  
  // 4. Fix missing data
  const { data: activitiesWithoutCity } = await supabase
    .from('activities')
    .select('*')
    .is('city', null)
  
  if (activitiesWithoutCity && activitiesWithoutCity.length > 0) {
    console.log(`Fixing ${activitiesWithoutCity.length} activities without city...`)
    
    for (const activity of activitiesWithoutCity) {
      await supabase
        .from('activities')
        .update({ city: 'Austin', state: 'TX' })
        .eq('id', activity.id)
    }
  }
  
  // 5. Final count
  const { count } = await supabase
    .from('activities')
    .select('*', { count: 'exact', head: true })
  
  console.log(`✅ Final activity count: ${count}`)
}

cleanActivities()
  .then(() => process.exit(0))
  .catch(error => {
    console.error('Cleaning failed:', error)
    process.exit(1)
  })
```

---

### Day 6-7: Testing & Validation

#### 1. Verify Data Quality

Run queries in Supabase SQL Editor:

```sql
-- Count by category
select category, count(*) 
from activities 
group by category 
order by count(*) desc;

-- Check for missing addresses
select count(*) from activities where address is null;

-- Check rating distribution
select 
  case 
    when rating >= 4.5 then '4.5+'
    when rating >= 4.0 then '4.0-4.5'
    when rating >= 3.5 then '3.5-4.0'
    else 'below 3.5'
  end as rating_range,
  count(*)
from activities
where rating is not null
group by rating_range;

-- Total count
select count(*) from activities where city = 'Austin';
```

**Target metrics:**
- Total: 100-150 activities
- At least 10 categories
- 80%+ have addresses
- 70%+ have ratings
- No duplicates

#### 2. Test Activity Browsing

- [ ] Visit `/dashboard/activities`
- [ ] Verify all categories display
- [ ] Check activity cards show correctly
- [ ] Verify ratings and price levels display
- [ ] Test on mobile viewport (responsive)

---

## Technical Stack & Tools

**New in Phase 3:**
- Cheerio (HTML parsing)
- Axios (HTTP requests)
- TSX (run TypeScript scripts)

**Data Sources:**
- Google Maps (manual curation primary)
- Yelp (scraping or manual)
- Local guides (manual)

**References:**
- `aiDocs/mvp.md` (lines 377-477) - Scraping implementation examples

---

## File Structure Created

```
app/
└── dashboard/
    └── activities/
        └── page.tsx              # Activity browsing UI

scripts/
├── scrapers/
│   ├── types.ts                  # TypeScript types
│   ├── utils.ts                  # Shared utilities
│   ├── google-maps.ts            # Google Maps scraper (optional)
│   └── yelp.ts                   # Yelp scraper
├── activities-manual.json        # Manually curated activities
├── seed-activities.ts            # Main seed script
└── clean-activities.ts           # Data cleaning

package.json                      # Added "seed" script
```

---

## Testing Strategy

### Data Quality Checks
- [ ] 100-150 total activities in database
- [ ] At least 10 different categories
- [ ] No duplicate names (case-insensitive)
- [ ] 80%+ have complete addresses
- [ ] 70%+ have ratings
- [ ] All have city: "Austin", state: "TX"

### UI Testing
- [ ] Activity browsing page loads
- [ ] Activities grouped by category
- [ ] Activity cards display all info
- [ ] Responsive design works on mobile
- [ ] No console errors

### Script Testing
- [ ] Seed script runs without errors
- [ ] Clean script removes duplicates
- [ ] Can re-run scripts without issues

---

## Risks & Mitigation

### Risk 1: Web Scraping Breaks
**Impact:** High - no activity data  
**Mitigation:** Start with manual curation (50 activities), scraping is bonus  
**Contingency:** 100% manual curation from Google Maps/Yelp

### Risk 2: Data Quality Issues
**Impact:** Medium - poor suggestions later  
**Mitigation:** Manual review of scraped data, clean script  
**Contingency:** Remove low-quality entries

### Risk 3: Takes Longer Than Expected
**Impact:** Medium - delays Phase 4  
**Mitigation:** Set target to 100 activities (not 150)  
**Contingency:** Move to Phase 4 with 50-75 activities

---

## Deliverables

- [ ] 100-150 activities in Supabase database
- [ ] Activity browsing UI in dashboard
- [ ] Seed script for adding activities
- [ ] Clean script for deduplication
- [ ] Manual activities JSON file
- [ ] Data quality report (SQL queries)

---

## Definition of Done

Phase 3 is complete when:

- ✅ Database contains 100-150 activities for Austin, TX
- ✅ At least 10 activity categories
- ✅ Activity browsing UI working
- ✅ Data quality is good (verified via SQL)
- ✅ No duplicate activities
- ✅ Scripts documented and reusable
- ✅ All success criteria met
- ✅ Ready for Phase 4 (Matching & Suggestions)

**Next Phase:** Phase 4 - Free Time Detection & Matching (Week 5)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026
