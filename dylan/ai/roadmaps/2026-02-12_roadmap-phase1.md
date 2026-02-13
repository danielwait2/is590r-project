# Phase 1 Roadmap: Foundation

**Status:** Not Started  
**Timeline:** Week 1-2 (14 days)  
**Detailed Plan:** [2026-02-12_phase1-foundation.md](./2026-02-12_phase1-foundation.md)

---

## ⚠️ Development Philosophy

**Avoid over-engineering, cruft, and legacy-compatibility features in this clean code project.**

- Build only what's needed for this phase
- No "just in case" features
- Delete unused code immediately
- Keep it simple and focused

---

## Quick Overview

Set up Next.js 14 app with Supabase database, Google OAuth authentication, and single-page onboarding flow. User can sign up, complete onboarding, and see empty dashboard.

---

## Task Checklist

### Project Setup (Day 1-2)
- [ ] Create Next.js 14 project with TypeScript
- [ ] Install core dependencies (@supabase/supabase-js, date-fns)
- [ ] Configure environment variables (.env.local)
- [ ] Set up .gitignore

### Supabase Setup (Day 2-3)
- [ ] Create Supabase project
- [ ] Copy project URL and API keys
- [ ] Create database schema (8 tables)
- [ ] Enable Row Level Security (RLS)
- [ ] Create RLS policies for all tables
- [ ] Test database access in Table Editor

### Google OAuth Setup (Day 3-5)
- [ ] Create Google Cloud project
- [ ] Enable Google Calendar API
- [ ] Configure OAuth consent screen
- [ ] Create OAuth 2.0 credentials
- [ ] Add redirect URIs (local + production)
- [ ] Configure Supabase Auth with Google provider

### Authentication Implementation (Day 5-7)
- [ ] Create Supabase client helpers (client.ts, server.ts)
- [ ] Create auth callback route (/auth/callback)
- [ ] Build landing page with "Sign in with Google"
- [ ] Create sign-in action (/auth/sign-in/route.ts)
- [ ] Create sign-out action (/auth/sign-out/route.ts)
- [ ] Test OAuth flow end-to-end

### Onboarding Flow (Day 7-10)
- [ ] Create dashboard layout (protected route)
- [ ] Build single-page onboarding form
  - [ ] Location input (city + state)
  - [ ] Kids input (add multiple with ages)
  - [ ] Want to Try list (textarea)
  - [ ] Trusted sitter input (name + phone)
- [ ] Implement form submission logic
- [ ] Save data to Supabase (users, kids, interests, trusted_sitters)
- [ ] Redirect to dashboard after completion
- [ ] Test with various input combinations

### Dashboard (Day 10-11)
- [ ] Create empty dashboard page
- [ ] Show welcome message with user's name
- [ ] Display Want to Try list
- [ ] Show empty state ("No suggestions yet")
- [ ] Add placeholder "Connect Calendar" button
- [ ] Test sign out functionality

### Deployment (Day 11-12)
- [ ] Initialize git repository
- [ ] Create GitHub repository
- [ ] Push code to GitHub
- [ ] Connect Vercel to GitHub repo
- [ ] Configure environment variables in Vercel
- [ ] Deploy to production
- [ ] Update Google OAuth redirect URIs with Vercel URL
- [ ] Test production deployment

### Testing & Polish (Day 12-14)
- [ ] Test full flow on production
- [ ] Fix any bugs discovered
- [ ] Test edge cases (skip optional fields, etc.)
- [ ] Verify database records are created correctly
- [ ] Check for console errors
- [ ] Clean up unused code
- [ ] Update README with setup instructions

---

## Success Criteria

- [ ] User can visit app at https://[project].vercel.app
- [ ] User can sign in with Google OAuth
- [ ] User completes onboarding (enters location, kids, Want to Try list, sitter)
- [ ] User sees empty dashboard with "No suggestions yet"
- [ ] Database contains user record with all onboarding data
- [ ] All environment variables configured (local + Vercel)
- [ ] App loads in <2 seconds
- [ ] No console errors on any page

---

## Key Deliverables

- [ ] Next.js 14 project deployed to Vercel
- [ ] Supabase project with 8 tables and RLS policies
- [ ] Google OAuth authentication working
- [ ] Single-page onboarding form (4 inputs)
- [ ] Empty dashboard page
- [ ] GitHub repository with code
- [ ] README with setup instructions
- [ ] .env.local.example with required variables

---

## Notes & Decisions

<!-- Track decisions during implementation -->

---

## Completion Checklist

Before moving to Phase 2:
- [ ] All tasks completed
- [ ] Success criteria met
- [ ] Deliverables created
- [ ] Production deployment tested
- [ ] Documentation updated (README)
- [ ] No critical bugs remaining
- [ ] Code pushed to GitHub
- [ ] Ready for Phase 2 (Calendar Integration)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026  
**Next Phase:** [Phase 2: Calendar Integration](./2026-02-12_roadmap-phase2.md)
