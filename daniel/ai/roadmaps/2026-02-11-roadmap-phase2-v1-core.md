# Phase 2 Roadmap: V1 Core Web App

**Status:** Pending Phase 1 Completion
**Timeline:** Weeks 1-4 (1 month)
**Detailed Plan:** [2026-02-11-phase2-v1-core-web-app.md](./2026-02-11-phase2-v1-core-web-app.md)

---

## ⚠️ Development Philosophy

**Avoid over-engineering, cruft, and legacy-compatibility features in this clean code project.**

- Build only what's needed for this phase
- No "just in case" features
- Delete unused code immediately
- Keep it simple and focused

---

## Quick Overview

Transform the Chrome extension into a production web application with user onboarding, Google Calendar integration, family profile management, and manual suggestion generation. Foundation for automation to be added in Phase 3.

---

## Task Checklist

### 2A: Infrastructure Setup (Week 1)
- [ ] Next.js project initialized
- [ ] PostgreSQL database on Railway
- [ ] Prisma schema defined and migrated
- [ ] Clerk authentication integrated
- [ ] Deployed to Vercel
- [ ] Custom domain configured
- [ ] GitHub CI/CD pipeline working

### 2B: User Onboarding (Week 2)
- [ ] Landing page designed and deployed
- [ ] Family profile form component created
- [ ] Onboarding flow functional
- [ ] Profile API endpoints working
- [ ] Onboarding completes in <5 minutes

### 2C: Calendar Sync & Analysis (Week 3)
- [ ] Google Calendar OAuth flow working
- [ ] Calendar events fetching correctly
- [ ] Free time detection algorithm ported
- [ ] Manual sync trigger functional

### 2D: Suggestion Generation (Week 4)
- [ ] Recommendation engine implemented
- [ ] Manual trigger for suggestions working
- [ ] Suggestions displayed in dashboard
- [ ] User can accept/reject suggestions
- [ ] Feedback collection working

---

## Success Criteria

- [ ] 50 beta users successfully onboarded
- [ ] Users can complete onboarding in <5 minutes
- [ ] Google Calendar sync works reliably (>95% success rate)
- [ ] Suggestion generation produces 2 relevant suggestions
- [ ] >40% suggestion acceptance rate
- [ ] Web app deployed to production
- [ ] Zero critical bugs

---

## Key Deliverables

- [ ] Production web app on Vercel
- [ ] PostgreSQL database with schema
- [ ] Onboarding flow (<5 min completion time)
- [ ] Manual suggestion generation working
- [ ] 10-20 beta users actively testing
- [ ] API documentation created

---

## Notes & Decisions

(Track key architectural decisions, tech stack choices, etc.)

---

## Completion Checklist

Before moving to Phase 3:
- [ ] All tasks completed
- [ ] Success criteria met
- [ ] Deliverables created
- [ ] 50+ beta users onboarded
- [ ] Manual suggestion flow validated
- [ ] Database schema stable
- [ ] Ready for automation layer

---

**Created:** 2026-02-11
**Last Updated:** 2026-02-11
**Previous Phase:** [2026-02-11-roadmap-phase1-mvp.md](./2026-02-11-roadmap-phase1-mvp.md)
**Next Phase:** [2026-02-11-roadmap-phase3-v1-polish.md](./2026-02-11-roadmap-phase3-v1-polish.md)
