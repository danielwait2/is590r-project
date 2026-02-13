# Phase 5 Roadmap: Babysitter Feature

**Status:** Not Started  
**Timeline:** Week 6 (7 days)  
**Detailed Plan:** [2026-02-12_phase5-babysitter.md](./2026-02-12_phase5-babysitter.md)

---

## ⚠️ Development Philosophy

**Avoid over-engineering, cruft, and legacy-compatibility features in this clean code project.**

- Build only what's needed for this phase
- No "just in case" features
- Delete unused code immediately
- Keep it simple and focused

---

## Quick Overview

Implement automated babysitter coordination via Twilio SMS. When suggestion is generated, send SMS to trusted sitters, parse YES/NO responses, update calendar with confirmation.

---

## Task Checklist

### Twilio Setup (Day 1)
- [ ] Create Twilio account (free trial)
- [ ] Purchase phone number with SMS capability
- [ ] Copy credentials to .env (Account SID, Auth Token, Phone)
- [ ] Install ngrok for local webhook testing
- [ ] Configure webhook URL in Twilio console

### SMS Sending (Day 1-2)
- [ ] Install twilio npm package
- [ ] Create Twilio client helper (lib/twilio/client.ts)
- [ ] Create babysitter request SMS template
- [ ] Implement sendSMS function
- [ ] Create send-requests.ts function
- [ ] Test SMS sending to your phone

### Webhook Handler (Day 3-4)
- [ ] Create /api/webhooks/sms route
- [ ] Parse inbound SMS (from, body, messageSid)
- [ ] Implement response parsing (YES/NO/UNCLEAR)
- [ ] Handle YES: accept request, update calendar, send confirmation
- [ ] Handle NO: decline request, send acknowledgment
- [ ] Notify other sitters when position filled
- [ ] Log all SMS in database

### Babysitter Roster UI (Day 4-5)
- [ ] Create /dashboard/sitters page
- [ ] Display list of trusted sitters
- [ ] Create add-sitter-form component
- [ ] Implement phone number validation (E.164 format)
- [ ] Create /api/sitters/add endpoint
- [ ] Create /api/sitters/delete endpoint
- [ ] Add navigation link to sitters page

### Calendar Update (Day 5)
- [ ] Update calendar event when sitter accepts
- [ ] Add "Babysitter: [Name] confirmed ✓" to description
- [ ] Test calendar update appears in Google Calendar

### End-to-End Testing (Day 5-6)
- [ ] Start ngrok tunnel
- [ ] Update Twilio webhook to ngrok URL
- [ ] Add your phone as test sitter
- [ ] Generate suggestion (requires babysitter)
- [ ] Verify SMS received
- [ ] Reply YES, verify acceptance flow
- [ ] Check calendar for sitter confirmation
- [ ] Test with multiple sitters (first YES wins)

### Polish & Error Handling (Day 6-7)
- [ ] Handle SMS send failures gracefully
- [ ] Validate phone numbers before saving
- [ ] Add error messages in UI
- [ ] Test invalid phone format
- [ ] Test webhook with unclear responses
- [ ] Update production webhook URL (Vercel)

---

## Success Criteria

- [ ] User can add multiple babysitters to roster
- [ ] SMS sent to all sitters when suggestion generated
- [ ] SMS includes all necessary info (date, time, pay, activity)
- [ ] Sitter can reply YES or NO
- [ ] First YES accepts, others get "position filled"
- [ ] Calendar updates with sitter confirmation
- [ ] Confirmation SMS sent to accepted sitter
- [ ] All SMS logged in database

---

## Key Deliverables

- [ ] Twilio SMS integration
- [ ] Babysitter roster UI (/dashboard/sitters)
- [ ] SMS webhook handler (/api/webhooks/sms)
- [ ] Babysitter request function
- [ ] Calendar update with sitter confirmation
- [ ] SMS message logging

---

## Notes & Decisions

<!-- Track decisions during implementation -->

---

## Completion Checklist

Before moving to Phase 6:
- [ ] All tasks completed
- [ ] Success criteria met
- [ ] End-to-end SMS flow tested
- [ ] Calendar updates with sitter name
- [ ] Multiple sitters tested
- [ ] Webhook deployed to production
- [ ] Ready for Phase 6 (Polish & Demo)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026  
**Previous Phase:** [Phase 4: Free Time & Matching](./2026-02-12_roadmap-phase4.md)  
**Next Phase:** [Phase 6: Polish & Demo](./2026-02-12_roadmap-phase6.md)
