# Phase 0: Foundation - Roadmap

**Created:** 2026-02-12
**Phase:** 0 of 10
**Timeline:** Week 1
**Status:** Not Started
**Companion Document:** [Phase 0 Plan](./2026-02-12_phase-0_foundation-plan.md)

---

## Overview

**Goal:** Complete environment setup and project scaffolding

**Duration:** Week 1 (4-6 hours of work)

**Why critical:** Everything else depends on proper foundation. Getting Google Cloud and project structure right prevents rework.

---

## Task Checklist

### Google Cloud Setup
- [ ] Create Google Cloud project ("Calendar Concierge MVP")
- [ ] Enable Google Calendar API in API Library
- [ ] Configure OAuth consent screen
  - [ ] Set user type to External
  - [ ] Add app name and support email
  - [ ] Add calendar.events scope
  - [ ] Add both test account emails as test users
- [ ] Create OAuth 2.0 credentials
  - [ ] Set application type to Web application
  - [ ] Add redirect URI: `http://localhost:3000/auth/callback`
  - [ ] Download credentials JSON
  - [ ] Save Client ID and Client Secret
- [ ] Verify Calendar API shows as "Enabled"
- [ ] Verify test users added to consent screen

### Project Initialization
- [ ] Create project directory structure (per architecture.md)
  - [ ] server/ directory with subdirectories
  - [ ] public/ directory
  - [ ] server/config/ directory
  - [ ] server/data/ directory
  - [ ] server/routes/ directory
  - [ ] server/services/ directory
- [ ] Initialize Node.js project (`npm init -y`)
- [ ] Install dependencies
  - [ ] `npm install express googleapis dotenv`
  - [ ] `npm install --save-dev nodemon`
- [ ] Update package.json scripts
  - [ ] Add "start" script
  - [ ] Add "dev" script with nodemon

### Environment Configuration
- [ ] Create .env file
  - [ ] Add GOOGLE_CLIENT_ID
  - [ ] Add GOOGLE_CLIENT_SECRET
  - [ ] Add GOOGLE_REDIRECT_URI
  - [ ] Add PORT=3000
  - [ ] Add NODE_ENV=development
  - [ ] Add DEFAULT_COUPLE_ID=couple-1
- [ ] Create .gitignore file
  - [ ] Add .env
  - [ ] Add node_modules/
  - [ ] Add server/config/google-credentials.json
  - [ ] Add server/data/users.json
  - [ ] Add *.log
  - [ ] Add OS and IDE specific files
- [ ] Save google-credentials.json to server/config/
- [ ] Verify .gitignore working (git status should not show sensitive files)

### Server Setup
- [ ] Create server/index.js with Express app
  - [ ] Add dotenv configuration
  - [ ] Add Express middleware (json, urlencoded, static)
  - [ ] Add root route (/)
  - [ ] Add health check route (/health)
  - [ ] Add server listen on PORT
  - [ ] Add console logging for startup info
- [ ] Test server starts without errors
- [ ] Test can access http://localhost:3000
- [ ] Test health check responds: http://localhost:3000/health

### Data Files
- [ ] Create server/data/users.json with initial structure
- [ ] Create server/data/preferences.json (empty object)
- [ ] Create server/data/activities.json (empty array)
- [ ] Create server/data/suggestions.json (empty array)
- [ ] Verify data directory accessible by server

### Landing Page
- [ ] Create public/index.html
  - [ ] Add page title and header
  - [ ] Add setup steps overview
  - [ ] Add links to future features (OAuth, preferences, admin)
  - [ ] Add system status section
- [ ] Create public/styles.css
  - [ ] Add basic layout styles
  - [ ] Add button styles
  - [ ] Add container styles
  - [ ] Add responsive design basics
- [ ] Test landing page loads at http://localhost:3000
- [ ] Test styles applied correctly
- [ ] Test links present (will be implemented in later phases)

### Testing & Verification
- [ ] Run `npm start` - should start without errors
- [ ] Run `npm run dev` - should start with nodemon
- [ ] Visit http://localhost:3000 - page loads
- [ ] Visit http://localhost:3000/health - JSON response
- [ ] Check console shows:
  - [ ] "Server running on http://localhost:3000"
  - [ ] "Google OAuth configured: true"
  - [ ] Environment loaded correctly
- [ ] Test git status - no sensitive files shown
- [ ] Verify file structure matches architecture.md

### Documentation
- [ ] Add README.md with setup instructions
- [ ] Document environment variables needed
- [ ] Document Google Cloud setup steps
- [ ] Add troubleshooting section for common issues

---

## Timeline

### Day 1 (2-3 hours)
- Morning: Google Cloud setup (1-2 hours)
  - Create project, enable API, configure OAuth
  - Download credentials
- Afternoon: Project initialization (1 hour)
  - npm setup, install dependencies
  - Create directory structure

### Day 2 (2-3 hours)
- Morning: Configuration (1 hour)
  - .env setup
  - .gitignore configuration
  - Data files initialization
- Afternoon: Server & UI (1-2 hours)
  - Create Express server
  - Create landing page
  - Test everything works

---

## Success Criteria

### Must Pass All:
1. **Server running:** Can execute `npm start` and access localhost:3000
2. **OAuth ready:** Google Cloud project configured with Calendar API enabled
3. **Environment loaded:** .env variables accessible in code
4. **File structure:** Matches architecture.md specification exactly
5. **Git safe:** No sensitive files tracked (.env, credentials, tokens)
6. **Health check:** /health endpoint returns valid JSON
7. **Landing page:** Loads with all links present (even if not functional yet)

### Phase 0 Complete When:
- All task checklist items checked
- All success criteria pass
- Server can run continuously without errors
- Ready to begin Phase 1 (OAuth Integration)

---

## Dependencies

**Required before starting:**
- Node.js 18+ installed
- npm installed
- Git installed
- Text editor (VS Code recommended)
- Google account with Cloud Console access
- Two Google accounts for testing

**External dependencies:**
- Google Cloud Console access
- Internet connection for npm installs

**No dependencies on other phases** (this is the foundation)

---

## Risks & Mitigation

### Risk: Google Cloud account issues
**Impact:** Can't proceed with OAuth
**Mitigation:**
- Ensure Google account in good standing
- Have credit card ready if verification required
- Use personal account, not work account (may have restrictions)
**Fallback:** Create new Google account if needed

### Risk: Port 3000 already in use
**Impact:** Server won't start
**Mitigation:**
- Check for processes on port 3000 before starting
- Use PORT=3001 in .env if needed
- Document port number clearly
**Fallback:** Use any available port (3001, 4000, etc.)

### Risk: OAuth redirect URI misconfigured
**Impact:** Phase 1 will fail
**Mitigation:**
- Triple-check redirect URI exactly: `http://localhost:3000/auth/callback`
- No trailing slash, http not https
- Copy-paste to avoid typos
**Fallback:** Can fix in Phase 1, but will waste time

### Risk: npm install fails
**Impact:** Can't install dependencies
**Mitigation:**
- Ensure Node.js 18+ installed (check with `node --version`)
- Clear npm cache if issues: `npm cache clean --force`
- Try removing node_modules and reinstalling
**Fallback:** Use different Node version manager (nvm)

---

## Blocked By

Nothing - this is Phase 0 (foundation)

---

## Blocks

- Phase 1: OAuth Integration (needs server + Google Cloud setup)
- Phase 2: Calendar Reading (needs OAuth)
- All subsequent phases depend on this foundation

---

## Milestone: Foundation Complete

**Definition of Done:**
- [ ] All tasks completed
- [ ] All success criteria met
- [ ] Server runs without errors
- [ ] Can access landing page
- [ ] Git configured properly
- [ ] Ready to begin Phase 1

**Celebration checkpoint:** Take screenshot of server running and landing page loaded!

---

## Next Phase

**Phase 1: OAuth Integration**
- [Phase 1 Plan](./2026-02-12_phase-1_oauth-integration-plan.md)
- [Phase 1 Roadmap](./2026-02-12_phase-1_oauth-integration-roadmap.md)
- Timeline: Weeks 1-2
- Critical path: Everything depends on OAuth working

---

**Reference Documents:**
- [High-Level Implementation Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture Documentation](../../aiDocs/architecture.md)
- [MVP Specification](../../aiDocs/mvp.md)
- [Phase 0 Plan](./2026-02-12_phase-0_foundation-plan.md)
