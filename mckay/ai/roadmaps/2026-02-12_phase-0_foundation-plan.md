# Phase 0: Foundation - Implementation Plan

**Created:** 2026-02-12
**Phase:** 0 of 10
**Timeline:** Week 1
**Companion Document:** [Phase 0 Roadmap](./2026-02-12_phase-0_foundation-roadmap.md)

---

## Project Principles

⚠️ **IMPORTANT: This is a clean-slate MVP build**

**DO:**
- Build the simplest thing that works
- Focus on core demo flow
- Use modern patterns and APIs
- Keep dependencies minimal

**DON'T:**
- Add backwards compatibility layers
- Build abstractions "for future flexibility"
- Include features "just in case"
- Copy/paste legacy code patterns
- Add error handling for impossible scenarios

---

## Phase Overview

**Goal:** Complete environment setup and project scaffolding to enable development

**Why this matters:** Solid foundation prevents rework later. Getting Google Cloud and OAuth configured correctly is critical for everything that follows.

**Key deliverables:**
- Google Cloud project with Calendar API enabled
- OAuth 2.0 credentials configured
- Node.js project structure in place
- Development environment ready

---

## Technical Specifications

### 1. Google Cloud Project Setup

**Objective:** Create and configure Google Cloud project for Calendar API access

**Steps:**
1. Navigate to [Google Cloud Console](https://console.cloud.google.com/)
2. Create new project: "Calendar Concierge MVP" (or similar)
3. Enable Google Calendar API in API Library
4. Configure OAuth consent screen:
   - User type: External (for testing with personal accounts)
   - App name: "Calendar Concierge MVP"
   - User support email: Your email
   - Scopes: Add `calendar.events` scope
   - Test users: Add both test Google account emails
5. Create OAuth 2.0 credentials:
   - Application type: Web application
   - Name: "Calendar Concierge Web Client"
   - Authorized redirect URIs: `http://localhost:3000/auth/callback`
6. Download credentials JSON file
7. Note Client ID and Client Secret for .env file

**Configuration details:**
- **OAuth Scopes Required:** `https://www.googleapis.com/auth/calendar.events`
- **Redirect URI:** Must exactly match: `http://localhost:3000/auth/callback`
- **Test Users:** Add both partner test account emails to OAuth consent screen

**Verification:**
- Can access project in Cloud Console
- Calendar API shows as "Enabled" in API Library
- OAuth credentials created with correct redirect URI
- Test users added to consent screen

---

### 2. Project Initialization

**Objective:** Set up Node.js project with required dependencies

**Project structure:**
```
mckay/
├── server/
│   ├── index.js                    # Express app entry point
│   ├── routes/
│   │   ├── auth.js                 # OAuth routes (Phase 1)
│   │   ├── preferences.js          # Preferences routes (Phase 4)
│   │   └── matching.js             # Matching routes (Phase 6)
│   ├── services/
│   │   ├── auth.js                 # OAuth service (Phase 1)
│   │   ├── calendar.js             # Google Calendar API (Phase 2)
│   │   ├── matching.js             # Matching engine (Phase 6)
│   │   └── activities.js           # Activity database (Phase 5)
│   ├── config/
│   │   └── google-credentials.json # OAuth client credentials (gitignored)
│   └── data/
│       ├── users.json              # OAuth tokens (gitignored)
│       ├── preferences.json        # User preferences
│       ├── activities.json         # Activity database
│       └── suggestions.json        # Suggestion history
├── public/
│   ├── index.html                  # Landing page
│   ├── preferences.html            # Preferences form (Phase 4)
│   ├── admin.html                  # Admin dashboard (Phase 8)
│   └── styles.css                  # Basic styling
├── .env                            # Environment variables (gitignored)
├── .gitignore                      # Git ignore rules
├── package.json                    # Dependencies
└── README.md                       # Setup instructions
```

**package.json configuration:**
```json
{
  "name": "calendar-concierge-mvp",
  "version": "1.0.0",
  "description": "Calendar-native weekend experience concierge",
  "main": "server/index.js",
  "scripts": {
    "start": "node server/index.js",
    "dev": "nodemon server/index.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "googleapis": "^118.0.0",
    "dotenv": "^16.0.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.0"
  }
}
```

**Installation:**
```bash
cd mckay
npm init -y
npm install express googleapis dotenv
npm install --save-dev nodemon
```

---

### 3. Environment Configuration

**Objective:** Set up environment variables and sensitive configuration

**.env file structure:**
```
# Google OAuth Configuration
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:3000/auth/callback

# Server Configuration
PORT=3000
NODE_ENV=development

# Couple Configuration (for MVP)
DEFAULT_COUPLE_ID=couple-1
```

**.gitignore configuration:**
```
# Environment
.env
.env.local
.env.*.local

# Dependencies
node_modules/

# Sensitive data
server/config/google-credentials.json
server/data/users.json

# Logs
*.log
npm-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo
```

**Security notes:**
- Never commit .env file
- Never commit google-credentials.json
- Never commit users.json (contains OAuth tokens)
- Keep preferences.json, activities.json, suggestions.json in git (no sensitive data)

---

### 4. Basic Express Server

**Objective:** Create minimal Express server to verify setup

**server/index.js (initial version):**
```javascript
require('dotenv').config();
const express = require('express');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

// Basic route to verify server works
app.get('/', (req, res) => {
  res.send(`
    <html>
      <head><title>Calendar Concierge MVP</title></head>
      <body>
        <h1>Calendar Concierge MVP</h1>
        <p>Server is running!</p>
        <ul>
          <li><a href="/auth/google?partner=1">Connect Partner 1 Calendar</a> (Phase 1)</li>
          <li><a href="/auth/google?partner=2">Connect Partner 2 Calendar</a> (Phase 1)</li>
          <li><a href="/preferences/couple-1">Set Preferences</a> (Phase 4)</li>
          <li><a href="/admin">Admin Dashboard</a> (Phase 8)</li>
        </ul>
        <p><em>Links will be implemented in later phases</em></p>
      </body>
    </html>
  `);
});

// Health check endpoint
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    env: process.env.NODE_ENV
  });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
  console.log(`Environment: ${process.env.NODE_ENV || 'development'}`);
  console.log(`Google OAuth configured: ${!!process.env.GOOGLE_CLIENT_ID}`);
});
```

**Testing:**
```bash
npm start
# Or for development with auto-reload:
npm run dev
```

Expected output:
```
Server running on http://localhost:3000
Environment: development
Google OAuth configured: true
```

Visit `http://localhost:3000` and verify page loads with links.

---

### 5. Data Directory Initialization

**Objective:** Create data files with proper initial structure

**server/data/users.json (initial):**
```json
{
  "couple-1": {
    "partner1": {
      "email": null,
      "accessToken": null,
      "refreshToken": null,
      "tokenExpiry": null
    },
    "partner2": {
      "email": null,
      "accessToken": null,
      "refreshToken": null,
      "tokenExpiry": null
    }
  }
}
```

**server/data/preferences.json (initial):**
```json
{}
```

**server/data/activities.json (initial):**
```json
[]
```

**server/data/suggestions.json (initial):**
```json
[]
```

**Note:** users.json should be in .gitignore. Other data files can be committed as they contain no sensitive data initially.

---

### 6. Landing Page

**Objective:** Create simple landing page with project overview

**public/index.html:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calendar Concierge MVP</title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <div class="container">
        <h1>Calendar Concierge MVP</h1>
        <p class="subtitle">Calendar-native weekend experience concierge</p>

        <section class="setup-section">
            <h2>Setup Steps</h2>
            <ol>
                <li><strong>Connect Calendars:</strong> Both partners authenticate with Google</li>
                <li><strong>Set Preferences:</strong> Share interests and "want to try" list</li>
                <li><strong>Get Suggestions:</strong> System finds free time and suggests activities</li>
            </ol>
        </section>

        <section class="links-section">
            <h2>Quick Links</h2>
            <ul>
                <li><a href="/auth/google?partner=1" class="btn">Connect Partner 1 Calendar</a></li>
                <li><a href="/auth/google?partner=2" class="btn">Connect Partner 2 Calendar</a></li>
                <li><a href="/preferences/couple-1" class="btn">Set Preferences</a></li>
                <li><a href="/admin" class="btn btn-primary">Admin Dashboard</a></li>
            </ul>
            <p class="note"><em>Note: OAuth and other features will be implemented in upcoming phases</em></p>
        </section>

        <section class="status-section">
            <h2>System Status</h2>
            <p>Server: <span class="status-ok">Running</span></p>
            <p>Phase: <strong>0 - Foundation</strong></p>
        </section>
    </div>
</body>
</html>
```

**public/styles.css (basic styling):**
```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
    line-height: 1.6;
    color: #333;
    background: #f5f5f5;
    padding: 20px;
}

.container {
    max-width: 800px;
    margin: 0 auto;
    background: white;
    padding: 40px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

h1 {
    color: #2c3e50;
    margin-bottom: 10px;
}

.subtitle {
    color: #7f8c8d;
    margin-bottom: 30px;
    font-size: 1.1em;
}

h2 {
    color: #34495e;
    margin: 30px 0 15px 0;
    border-bottom: 2px solid #3498db;
    padding-bottom: 5px;
}

.setup-section ol {
    margin-left: 20px;
}

.setup-section li {
    margin: 10px 0;
}

.links-section ul {
    list-style: none;
    margin: 20px 0;
}

.links-section li {
    margin: 10px 0;
}

.btn {
    display: inline-block;
    padding: 10px 20px;
    background: #3498db;
    color: white;
    text-decoration: none;
    border-radius: 4px;
    transition: background 0.3s;
}

.btn:hover {
    background: #2980b9;
}

.btn-primary {
    background: #27ae60;
}

.btn-primary:hover {
    background: #229954;
}

.note {
    margin-top: 20px;
    color: #7f8c8d;
    font-size: 0.9em;
}

.status-section {
    margin-top: 30px;
    padding: 20px;
    background: #ecf0f1;
    border-radius: 4px;
}

.status-ok {
    color: #27ae60;
    font-weight: bold;
}
```

---

## Data Structures

### Users Data Structure
```javascript
{
  "couple-1": {
    "partner1": {
      "email": "alex@example.com",          // Set during OAuth
      "accessToken": "ya29.a0AfH6SMBx...",  // From Google OAuth
      "refreshToken": "1//0gQ1K5Z...",      // For token refresh
      "tokenExpiry": "2026-02-12T15:00:00Z" // ISO 8601 timestamp
    },
    "partner2": {
      "email": "jordan@example.com",
      "accessToken": "ya29.a0AfH6SMCy...",
      "refreshToken": "1//0gR2L6A...",
      "tokenExpiry": "2026-02-12T15:30:00Z"
    }
  }
}
```

This structure will be populated in Phase 1 (OAuth Integration).

---

## Testing Criteria

### Environment Setup Verification
- [ ] Google Cloud project created and accessible
- [ ] Calendar API enabled in project
- [ ] OAuth credentials created with correct redirect URI
- [ ] Test users added to OAuth consent screen
- [ ] google-credentials.json downloaded

### Local Development Verification
- [ ] Node.js installed (version 18+)
- [ ] npm installed and working
- [ ] Project dependencies installed (no errors)
- [ ] .env file created with correct values
- [ ] .gitignore configured properly

### Server Verification
- [ ] Can run `npm start` without errors
- [ ] Server starts on port 3000
- [ ] Can access http://localhost:3000
- [ ] Landing page loads correctly
- [ ] Health check endpoint responds: http://localhost:3000/health
- [ ] Environment variables load (check console output)

### File Structure Verification
- [ ] All directories created per architecture.md
- [ ] Data files initialized with correct structure
- [ ] Static files (HTML/CSS) accessible via browser
- [ ] No errors in browser console

### Git Verification
- [ ] .gitignore working (sensitive files not tracked)
- [ ] Can commit non-sensitive files
- [ ] google-credentials.json NOT in git status
- [ ] users.json NOT in git status
- [ ] .env NOT in git status

---

## Common Issues & Solutions

### Issue: Google Cloud requires credit card
**Solution:** Calendar API is usually free tier, but you may need to add a card for account verification. No charges for our usage volume.

### Issue: OAuth redirect URI mismatch error
**Solution:** Ensure redirect URI in Google Cloud Console exactly matches: `http://localhost:3000/auth/callback` (no trailing slash, http not https)

### Issue: GOOGLE_CLIENT_ID not loading from .env
**Solution:**
- Check .env file is in project root (mckay/)
- Ensure no quotes around values in .env
- Restart server after .env changes
- Verify with: `console.log(process.env.GOOGLE_CLIENT_ID)`

### Issue: Port 3000 already in use
**Solution:**
- Kill process using port: `lsof -ti:3000 | xargs kill` (Mac/Linux) or change PORT in .env
- Or use different port: `PORT=3001 npm start`

### Issue: Can't access test Google accounts
**Solution:** Create two new Gmail accounts specifically for testing (alex.test@gmail.com, jordan.test@gmail.com or similar)

---

## Blockers & Dependencies

### Blockers
- **Google account access:** Need Google account with Cloud Console access
- **Test accounts:** Need two Google accounts for testing OAuth
- **Credit card:** May be required for Google Cloud (usually not for Calendar API)

### Dependencies
- None (this is Phase 0 - foundation for everything else)

### Risks
- **Google Cloud configuration:** First time can be confusing, follow docs carefully
- **OAuth setup:** Most common source of errors in Phase 1, get foundation right now
- **Test account access:** Make sure both test accounts accessible for next phase

---

## Next Steps

After completing Phase 0:
1. Verify all testing criteria passed
2. Ensure server runs without errors
3. Move to [Phase 1: OAuth Integration](./2026-02-12_phase-1_oauth-integration-plan.md)
4. Review [Phase 1 Roadmap](./2026-02-12_phase-1_oauth-integration-roadmap.md)

**Phase 0 is complete when:**
- Server runs on localhost:3000
- Google Cloud project configured
- Environment variables loaded
- File structure in place
- No errors in setup

---

**Reference Documents:**
- [High-Level Implementation Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture Documentation](../../aiDocs/architecture.md)
- [MVP Specification](../../aiDocs/mvp.md)
- [Phase 0 Roadmap](./2026-02-12_phase-0_foundation-roadmap.md)
