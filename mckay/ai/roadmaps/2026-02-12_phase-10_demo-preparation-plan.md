# Phase 10: Demo Preparation - Implementation Plan

**Created:** 2026-02-12
**Phase:** 10 of 10
**Timeline:** Week 8
**Companion Document:** [Phase 10 Roadmap](./2026-02-12_phase-10_demo-preparation-roadmap.md)

---

## Project Principles

⚠️ **Demo day is about storytelling, not just technical execution.** Prepare narrative, practice timing, have backup plan.

---

## Phase Overview

**Goal:** Ready for live demo presentation

**Key deliverables:**
- Demo script
- Test accounts pre-configured
- Sample calendar events
- Backup plan (screenshots, video)
- Presentation materials (optional)
- 3+ successful practice runs

---

## Demo Preparation

### 1. Demo Script

**Create:** `mckay/ai/notes/demo-script.md`

**Structure:**

```markdown
# Demo Script - Calendar Concierge MVP

## Setup (Before Demo)
- [ ] Server running on localhost:3000
- [ ] Both test accounts authenticated
- [ ] Preferences set
- [ ] Test calendars have sample events
- [ ] Have backup materials ready

## Demo Flow (5-10 minutes)

### Minute 0-1: Introduction
**Say:** "Meet Alex and Jordan, a busy couple who love trying new things together.
They maintain a list of activities they want to do, but coordinating schedules
and actually planning activities takes effort they don't have time for."

**Show:** Their Google Calendars side-by-side (busy schedules)

### Minute 1-2: The Problem
**Say:** "They both have free time this Saturday afternoon, but they don't know it yet.
And even if they did, they'd spend 30 minutes deciding what to do, researching options,
and making plans."

**Show:** Overlapping free time exists but isn't obvious

### Minute 2-3: The Setup
**Say:** "With Calendar Concierge, they did a one-time setup: connected their calendars,
shared their interests, and entered their 'want to try' list."

**Show:** Preferences page (already filled out)
**Point out:** "Pottery class, Italian restaurant, wine tasting - things they've been meaning to do"

### Minute 3-5: The Magic
**Say:** "Now watch what happens when we run the matching engine."

**Do:** Navigate to admin dashboard, click "Generate Suggestions"

**While loading, say:** "The system is:
- Finding when they're both free
- Matching activities to their interests
- Creating suggestions in both their calendars"

**Show:** Results appear
- "Found 3 free time windows"
- "Matched 8 activities"
- "Created 2 suggestions"

### Minute 5-7: The Result
**Say:** "Let's look at what was suggested."

**Show:** Suggestion details
- Activity name, time, location
- Match reason: "Based on your interest in arts and your want-to-try: pottery class"

**Click:** Calendar link for Partner 1

**Say:** "Now when Alex checks her calendar..."

**Show:** Google Calendar with suggestion event
- Shows as tentative
- Full description visible

**Click:** Calendar link for Partner 2

**Say:** "...and when Jordan checks his calendar, they both see the same suggestion."

**Show:** Same event in second calendar

### Minute 7-8: The Value
**Say:** "No app to remember to check. No 'what should we do this weekend' conversation.
Their want-to-try list becomes actual plans. They just see it when planning their week,
accept or decline in 5 seconds."

### Minute 8-9: Technical Highlights
**Say:** "Technically, this demonstrates:
- OAuth 2.0 integration with Google Calendar
- Multi-user coordination logic
- Activity matching algorithm
- Real-time calendar manipulation"

### Minute 9-10: Q&A
Be ready to answer:
- How does matching work? (Scoring algorithm)
- What about privacy? (OAuth, no data shared)
- What's next? (Automated weekly suggestions, more data sources)
```

---

### 2. Test Account Setup

**Test Account 1 (Partner 1):**
- Email: Create/use dedicated test Gmail
- Calendar name: "Partner 1 - Alex"
- Sample events:
  - Monday-Friday: Work meetings (9am-5pm)
  - Wednesday evening: Gym class (6pm-7pm)
  - Thursday evening: Book club (7pm-9pm)
  - Saturday morning: Dentist (10am-11am)
  - Leave Saturday afternoon free (2pm-6pm)
  - Leave Sunday morning free (10am-2pm)

**Test Account 2 (Partner 2):**
- Email: Create/use second test Gmail
- Calendar name: "Partner 2 - Jordan"
- Sample events:
  - Monday-Friday: Work hours (9am-6pm)
  - Tuesday evening: Soccer practice (6pm-8pm)
  - Friday evening: Dinner with parents (6pm-9pm)
  - Saturday morning: Errands (9am-12pm)
  - Leave Saturday afternoon free (2pm-7pm)
  - Sunday: Full day busy (brunch, family time)

**Overlapping free time:** Saturday 2pm-6pm (4 hours)

---

### 3. Preferences Setup

**Pre-configure:**
- Interests: Dining, Arts, Classes
- Want to try:
  - "pottery class"
  - "Italian restaurant downtown"
  - "wine tasting"
- Location: Austin, TX (or your target city)
- Min duration: 120 minutes
- Advance notice: 3 days
- Budget: $$$

---

### 4. Activities Database

**Ensure activities.json includes:**
- Pottery class (matches "pottery class" want-to-try, arts interest)
- Italian restaurant (matches "Italian" want-to-try, dining interest)
- Wine tasting (matches "wine tasting" want-to-try, dining interest)
- Other varied activities for demonstration

---

### 5. Backup Plan

**If live demo fails:**

**Option A: Video Recording**
- Record successful demo run before presentation day
- Have video ready to play
- Narrate over video
- Explain: "This is a recording of the system working"

**Option B: Screenshots**
- Screenshot each step of flow
- Present as slides
- Walk through what would happen
- Show code/architecture instead

**Option C: Code Walkthrough**
- Show architecture diagram
- Explain OAuth flow
- Show matching algorithm code
- Demonstrate with mock data

**Always have Options A or B ready**

---

### 6. Presentation Materials (Optional)

**Create simple slides (5-7 slides):**

1. **Title Slide**
   - Project name
   - Your name
   - One-sentence description

2. **The Problem**
   - Busy couples
   - Want to try things together
   - Coordination is hard

3. **The Solution**
   - Calendar-native suggestions
   - Automatic free time detection
   - Personalized activity matching

4. **Architecture**
   - Simple diagram showing:
     - Google Calendar API
     - OAuth
     - Matching Engine
     - Back to Google Calendar

5. **Demo** (live)
   - This slide just says "Live Demo"
   - Or shows backup video

6. **Technical Highlights**
   - OAuth 2.0
   - Google Calendar API v3
   - Algorithm design
   - Node.js/Express

7. **Next Steps**
   - Automated weekly suggestions
   - Real event data integration
   - User feedback loop

---

## Practice Runs

### Run 1: Timing
- Complete full demo start to finish
- Time each section
- Identify slow parts
- Adjust script

### Run 2: Technical
- Test on demo day computer (if different)
- Verify internet connection
- Check screen resolution
- Test projector (if applicable)

### Run 3: Narrative
- Practice speaking parts
- Don't just click silently
- Explain what's happening
- Practice handling questions

---

## Pre-Demo Checklist

**Day Before:**
- [ ] Test accounts authenticated
- [ ] Preferences set
- [ ] Sample events in both calendars
- [ ] Activities database populated
- [ ] Run full flow successfully
- [ ] Create backup video
- [ ] Take backup screenshots
- [ ] Charge laptop fully
- [ ] Test on demo venue WiFi (if possible)

**Morning Of:**
- [ ] Server running
- [ ] Quick test run (5 min)
- [ ] Verify both calendars accessible
- [ ] Clear old suggestions from calendars
- [ ] Have backup materials ready
- [ ] Relax - you're prepared!

---

## Demo Day Troubleshooting

### If OAuth fails:
- Use pre-authenticated accounts
- Show flow with screenshots
- Explain: "Already authenticated for demo"

### If API fails:
- Switch to backup video
- Explain: "Network issue, showing recording"
- Continue with narrative

### If matching produces no results:
- Check preferences loaded
- Verify activities.json loaded
- Fall back to showing algorithm code

### If completely stuck:
- Take a breath
- Show backup materials
- Explain what should happen
- Show code/architecture instead
- This is a learning project - be honest!

---

## Post-Demo

**Immediately after:**
- Save any demo recordings
- Note what worked well
- Note what could improve
- Document any issues

**Follow-up:**
- Clean up test accounts
- Archive demo materials
- Update README with demo notes
- Reflect on lessons learned

---

## Success Criteria

**Minimum for success:**
- Demonstrate OAuth integration
- Show calendar events being read
- Show suggestion being created
- Explain value proposition clearly
- Answer technical questions

**Ideal demo:**
- Live demo works smoothly
- Takes 5-7 minutes (not rushing, not dragging)
- Audience understands problem and solution
- Technical competence evident
- Questions handled confidently

---

## Next Steps

After Phase 10:
- Demo presentation complete!
- Reflect on project
- Document lessons learned
- Consider post-MVP enhancements

**Phase 10 complete when:** Demo delivered successfully!

---

**Reference Documents:**
- [High-Level Plan](./2026-02-12_high-level_mvp-implementation-plan.md)
- [Architecture](../../aiDocs/architecture.md)
- [MVP Spec](../../aiDocs/mvp.md)
- [Phase 10 Roadmap](./2026-02-12_phase-10_demo-preparation-roadmap.md)

---

## Congratulations!

You've reached the end of the roadmap. If you've completed all 10 phases:

- You built a real OAuth 2.0 integration
- You integrated with Google Calendar API
- You designed and implemented a matching algorithm
- You created a working end-to-end system
- You demonstrated technical competence

**This is a significant achievement. Great work!**

Now go deliver that demo with confidence.
