# Phase 3 Roadmap: Activity Discovery

**Status:** Not Started  
**Timeline:** Week 4 (7 days)  
**Detailed Plan:** [2026-02-12_phase3-activities.md](./2026-02-12_phase3-activities.md)

---

## ⚠️ Development Philosophy

**Avoid over-engineering, cruft, and legacy-compatibility features in this clean code project.**

- Build only what's needed for this phase
- No "just in case" features
- Delete unused code immediately
- Keep it simple and focused

---

## Quick Overview

Populate database with 100-150 activities in Austin, TX via web scraping and manual curation. Build activity browsing UI in dashboard.

---

## Task Checklist

### Setup (Day 1)
- [ ] Install cheerio, axios, and tsx
- [ ] Create scripts/scrapers/ directory
- [ ] Define TypeScript types for activities
- [ ] Create scraper utilities (fetchHTML, delay, etc.)

### Manual Curation (Day 1-2)
- [ ] Create activities-manual.json file
- [ ] Manually curate 30-50 top activities from Google Maps
- [ ] Include key categories (dining, outdoor, entertainment, family)
- [ ] Verify addresses and ratings

### Build Scrapers (Day 2-3)
- [ ] Create Yelp scraper (scrapeYelp function)
- [ ] Define 10 category queries
- [ ] Test scraper on 1-2 categories
- [ ] Handle scraping errors gracefully

### Seed Database (Day 3-4)
- [ ] Create seed-activities.ts script
- [ ] Combine manual + scraped data
- [ ] Remove duplicates
- [ ] Insert into Supabase activities table
- [ ] Run seed script: `npm run seed`
- [ ] Verify 100-150 activities in database

### Data Cleaning (Day 4-5)
- [ ] Create clean-activities.ts script
- [ ] Remove duplicate activities
- [ ] Fix missing city/state data
- [ ] Verify data quality with SQL queries
- [ ] Achieve 100-150 unique activities

### Activity Browsing UI (Day 5-6)
- [ ] Create /dashboard/activities page
- [ ] Display activities grouped by category
- [ ] Show name, address, rating, price level
- [ ] Add navigation link in dashboard layout
- [ ] Test responsive design

### Testing & Validation (Day 6-7)
- [ ] Verify 100-150 activities in database
- [ ] Check 10+ categories exist
- [ ] Confirm no duplicates
- [ ] Test activity browsing UI
- [ ] Verify data displays correctly
- [ ] Check mobile responsiveness

---

## Success Criteria

- [ ] Database contains 100-150 activities for Austin, TX
- [ ] At least 10 activity categories
- [ ] Activity browsing UI working
- [ ] Data quality verified (no duplicates, complete info)
- [ ] Scripts documented and reusable
- [ ] Activities display correctly on all devices

---

## Key Deliverables

- [ ] 100-150 activities in Supabase
- [ ] Activity browsing page (/dashboard/activities)
- [ ] Seed script (scripts/seed-activities.ts)
- [ ] Clean script (scripts/clean-activities.ts)
- [ ] Manual activities JSON file
- [ ] Scraper utilities

---

## Notes & Decisions

<!-- Track decisions during implementation -->

---

## Completion Checklist

Before moving to Phase 4:
- [ ] All tasks completed
- [ ] Success criteria met
- [ ] 100-150 activities in database
- [ ] Data quality verified
- [ ] Activity browsing UI works
- [ ] Scripts tested and documented
- [ ] Ready for Phase 4 (Matching & Suggestions)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026  
**Previous Phase:** [Phase 2: Calendar Integration](./2026-02-12_roadmap-phase2.md)  
**Next Phase:** [Phase 4: Free Time & Matching](./2026-02-12_roadmap-phase4.md)
