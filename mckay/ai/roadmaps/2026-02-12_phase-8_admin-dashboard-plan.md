# Phase 8: Admin Dashboard - Implementation Plan

**Created:** 2026-02-12
**Phase:** 8 of 10
**Timeline:** Week 7
**Companion Document:** [Phase 8 Roadmap](./2026-02-12_phase-8_admin-dashboard-roadmap.md)

---

## Project Principles

⚠️ **Simple HTML/CSS/Vanilla JS.** No React/Vue complexity for MVP. Focus on functionality.

---

## Phase Overview

**Goal:** Create admin UI to trigger matching and view results

**Key deliverables:**
- Admin HTML page
- Couple status display
- Generate suggestions button
- Results display (free windows, matches, suggestions)
- Activity log
- Calendar links

---

## Technical Specifications

### 1. Admin Dashboard HTML

**File:** `public/admin.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Admin Dashboard - Calendar Concierge</title>
    <link rel="stylesheet" href="/styles.css">
</head>
<body>
    <div class="container">
        <h1>Admin Dashboard</h1>
        <p class="subtitle">Calendar Concierge MVP</p>

        <section class="couple-section">
            <h2>Couple: Alex & Jordan (couple-1)</h2>

            <div id="couple-status" class="status-box">
                <p>Loading status...</p>
            </div>

            <button id="generateBtn" class="btn btn-primary" onclick="generateSuggestions()">
                Generate Suggestions
            </button>

            <div id="results" class="results-section" style="display:none;">
                <h3>Results</h3>
                <div id="results-content"></div>
            </div>
        </section>

        <section class="log-section">
            <h2>Activity Log</h2>
            <div id="activity-log" class="log-box"></div>
        </section>

        <p><a href="/">Back to home</a></p>
    </div>

    <script>
        const coupleId = 'couple-1';

        // Load couple status on page load
        async function loadCoupleStatus() {
            try {
                // Check OAuth status
                const authResponse = await fetch(`/auth/status/${coupleId}`);
                const authData = await authResponse.json();

                // Check preferences
                const prefsResponse = await fetch(`/preferences/${coupleId}/data`);
                const hasPrefs = prefsResponse.ok;

                const statusDiv = document.getElementById('couple-status');
                statusDiv.innerHTML = `
                    <h3>Setup Status</h3>
                    <p>
                        <strong>Partner 1 Calendar:</strong>
                        ${authData.partner1.authenticated ? '✓ Connected' : '✗ Not connected'}
                        ${authData.partner1.email ? `(${authData.partner1.email})` : ''}
                    </p>
                    <p>
                        <strong>Partner 2 Calendar:</strong>
                        ${authData.partner2.authenticated ? '✓ Connected' : '✗ Not connected'}
                        ${authData.partner2.email ? `(${authData.partner2.email})` : ''}
                    </p>
                    <p>
                        <strong>Preferences:</strong>
                        ${hasPrefs ? '✓ Set' : '✗ Not set'}
                    </p>
                    <p class="${authData.partner1.authenticated && authData.partner2.authenticated && hasPrefs ? 'success' : 'warning'}">
                        ${authData.partner1.authenticated && authData.partner2.authenticated && hasPrefs
                            ? '✓ Ready to generate suggestions'
                            : '⚠ Complete setup before generating suggestions'}
                    </p>
                `;

                // Disable button if not ready
                const btn = document.getElementById('generateBtn');
                btn.disabled = !(authData.partner1.authenticated &&
                                 authData.partner2.authenticated &&
                                 hasPrefs);
            } catch (error) {
                console.error('Failed to load status:', error);
                addToLog('Error loading couple status');
            }
        }

        // Generate suggestions
        async function generateSuggestions() {
            const btn = document.getElementById('generateBtn');
            const resultsDiv = document.getElementById('results');
            const resultsContent = document.getElementById('results-content');

            btn.disabled = true;
            btn.textContent = 'Generating...';
            resultsDiv.style.display = 'block';
            resultsContent.innerHTML = '<p>Running matching engine...</p>';

            try {
                const response = await fetch(`/matching/${coupleId}`, {
                    method: 'POST'
                });

                const data = await response.json();

                if (!response.ok) {
                    throw new Error(data.error || 'Failed to generate suggestions');
                }

                displayResults(data);
                addToLog(`Generated ${data.suggestions.length} suggestion(s)`);
            } catch (error) {
                console.error('Error generating suggestions:', error);
                resultsContent.innerHTML = `<p class="error">Error: ${error.message}</p>`;
                addToLog(`Error: ${error.message}`);
            } finally {
                btn.disabled = false;
                btn.textContent = 'Generate Suggestions';
            }
        }

        // Display results
        function displayResults(data) {
            const resultsContent = document.getElementById('results-content');

            if (data.message) {
                resultsContent.innerHTML = `<p class="warning">${data.message}</p>`;
                return;
            }

            let html = `
                <h4>Free Time Windows Found: ${data.freeWindows.length}</h4>
                <ul class="free-windows-list">
                    ${data.freeWindows.slice(0, 5).map(window => `
                        <li>
                            ${new Date(window.start).toLocaleString()} -
                            ${new Date(window.end).toLocaleString()}
                            (${window.durationMinutes} min)
                        </li>
                    `).join('')}
                    ${data.freeWindows.length > 5 ? `<li>...and ${data.freeWindows.length - 5} more</li>` : ''}
                </ul>

                <h4>Activities Matched: ${data.matchedActivities.length}</h4>
                <p>Top matches by score</p>

                <h4>Suggestions Created: ${data.suggestions.length}</h4>
            `;

            if (data.suggestions.length > 0) {
                html += '<div class="suggestions-list">';
                data.suggestions.forEach((suggestion, index) => {
                    html += `
                        <div class="suggestion-card">
                            <h5>${index + 1}. ${suggestion.activity.name}</h5>
                            <p><strong>When:</strong>
                                ${new Date(suggestion.suggestedStart).toLocaleString()}</p>
                            <p><strong>Where:</strong> ${suggestion.activity.location.name}</p>
                            <p><strong>Why:</strong> ${suggestion.matchReasons}</p>
                            <p><strong>Score:</strong> ${suggestion.score} points</p>
                            <p><strong>View in Calendar:</strong>
                                <a href="${suggestion.calendarLinks.partner1}" target="_blank">Partner 1</a> |
                                <a href="${suggestion.calendarLinks.partner2}" target="_blank">Partner 2</a>
                            </p>
                        </div>
                    `;
                });
                html += '</div>';
            }

            resultsContent.innerHTML = html;
        }

        // Add to activity log
        function addToLog(message) {
            const logDiv = document.getElementById('activity-log');
            const timestamp = new Date().toLocaleString();
            const entry = `<div class="log-entry">[${timestamp}] ${message}</div>`;
            logDiv.innerHTML = entry + logDiv.innerHTML;
        }

        // Load status on page load
        loadCoupleStatus();
        addToLog('Dashboard loaded');
    </script>
</body>
</html>
```

---

### 2. Admin Dashboard Styles

**Add to:** `public/styles.css`

```css
/* Admin Dashboard Styles */

.couple-section {
    margin: 30px 0;
    padding: 20px;
    background: #f8f9fa;
    border-radius: 8px;
}

.status-box {
    padding: 15px;
    background: white;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin: 20px 0;
}

.status-box p {
    margin: 10px 0;
}

.status-box .success {
    color: #27ae60;
    font-weight: bold;
}

.status-box .warning {
    color: #f39c12;
    font-weight: bold;
}

.results-section {
    margin-top: 30px;
    padding: 20px;
    background: white;
    border: 2px solid #3498db;
    border-radius: 8px;
}

.free-windows-list {
    list-style: none;
    padding: 0;
}

.free-windows-list li {
    padding: 8px;
    border-bottom: 1px solid #eee;
}

.suggestions-list {
    margin-top: 20px;
}

.suggestion-card {
    padding: 15px;
    margin: 15px 0;
    background: #f8f9fa;
    border-left: 4px solid #27ae60;
    border-radius: 4px;
}

.suggestion-card h5 {
    margin-top: 0;
    color: #2c3e50;
}

.suggestion-card p {
    margin: 8px 0;
}

.log-section {
    margin: 30px 0;
}

.log-box {
    padding: 15px;
    background: #2c3e50;
    color: #ecf0f1;
    border-radius: 4px;
    font-family: 'Courier New', monospace;
    font-size: 12px;
    max-height: 300px;
    overflow-y: auto;
}

.log-entry {
    padding: 5px 0;
    border-bottom: 1px solid #34495e;
}

.error {
    color: #e74c3c;
}

button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
}
```

---

### 3. Admin Route

**Update:** `server/index.js`

```javascript
// Serve admin dashboard
app.get('/admin', (req, res) => {
  res.sendFile(path.join(__dirname, '../public/admin.html'));
});
```

---

## Testing Criteria

- [ ] Dashboard loads at /admin
- [ ] Shows couple status (calendars + preferences)
- [ ] Generate button works
- [ ] Shows loading state during generation
- [ ] Displays results (free windows, matches, suggestions)
- [ ] Calendar links work
- [ ] Activity log updates
- [ ] Button disabled if setup incomplete
- [ ] Error messages display properly

---

## Next Steps

After Phase 8:
- Verify dashboard functionality
- Test full flow from dashboard
- Move to [Phase 9: Integration & Testing](./2026-02-12_phase-9_integration-testing-plan.md)

**Phase 8 complete when:** Can trigger matching from dashboard and view results.

---

**Reference Documents:**
- [High-Level Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture](../../aiDocs/architecture.md)
- [Phase 8 Roadmap](./2026-02-12_phase-8_admin-dashboard-roadmap.md)
