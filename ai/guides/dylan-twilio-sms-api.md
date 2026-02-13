# Twilio SMS API - Implementation Guide

**Source:** https://www.twilio.com/docs/messaging/quickstart/node  
**Official Docs:** https://www.twilio.com/docs/sms  
**Last Updated:** February 12, 2026  
**Status:** ✅ VERIFIED - Real API, production-ready

---

## Overview

Twilio Programmable Messaging allows you to:
- ✅ Send SMS messages programmatically
- ✅ Receive SMS messages via webhooks
- ✅ Two-way SMS conversations
- ✅ Message status tracking
- ✅ 180+ countries supported

**Base URL:** `https://api.twilio.com`

---

## Setup

### 1. Create Twilio Account

1. Sign up at [twilio.com/try-twilio](https://www.twilio.com/try-twilio)
2. Get a phone number with SMS capabilities
3. Copy your Account SID and Auth Token

### 2. Install SDK

```bash
npm install twilio
```

### 3. Environment Variables

```bash
# .env
TWILIO_ACCOUNT_SID=your_account_sid
TWILIO_AUTH_TOKEN=your_auth_token
TWILIO_PHONE_NUMBER=+15551234567
```

---

## Sending SMS (Outbound)

### Basic Example

```javascript
const twilio = require('twilio');

const accountSid = process.env.TWILIO_ACCOUNT_SID;
const authToken = process.env.TWILIO_AUTH_TOKEN;
const client = twilio(accountSid, authToken);

async function sendSMS(to, body) {
  const message = await client.messages.create({
    body: body,
    from: process.env.TWILIO_PHONE_NUMBER,
    to: to  // Must be E.164 format: +15551234567
  });
  
  console.log('Message SID:', message.sid);
  return message;
}
```

### Babysitter Request Example (Our Use Case)

```javascript
async function sendBabysitterRequest(sitter, details) {
  const message = await client.messages.create({
    body: `Hi ${sitter.name}!

Dylan & Sarah need a babysitter:

📅 Date: ${details.date}
⏰ Time: ${details.startTime} - ${details.endTime} (${details.duration} hours)
👶 Kids: ${details.numKids} (ages ${details.ages})
💰 Pay: $${details.estimatedPay} ($${sitter.hourlyRate}/hour)
📍 Location: ${details.address}
ℹ️ Reason: ${details.activityName}

Can you do it?
→ Reply YES to accept
→ Reply NO if unavailable

Reply by ${details.replyDeadline}.`,
    from: process.env.TWILIO_PHONE_NUMBER,
    to: sitter.phone,
    statusCallback: 'https://yourdomain.com/api/webhooks/sms-status'
  });
  
  return message;
}
```

### TypeScript Version

```typescript
import twilio from 'twilio';

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID!,
  process.env.TWILIO_AUTH_TOKEN!
);

interface BabysitterRequest {
  sitterName: string;
  sitterPhone: string;
  date: string;
  startTime: string;
  endTime: string;
  duration: number;
  numKids: number;
  ages: string;
  estimatedPay: number;
  hourlyRate: number;
  address: string;
  activityName: string;
  replyDeadline: string;
}

async function sendRequest(details: BabysitterRequest) {
  const message = await client.messages.create({
    body: `Hi ${details.sitterName}!...`,
    from: process.env.TWILIO_PHONE_NUMBER!,
    to: details.sitterPhone
  });
  
  return message;
}
```

---

## Receiving SMS (Inbound)

### Webhook Setup

**1. Configure webhook URL in Twilio Console:**
- Go to Phone Numbers → Active Numbers
- Select your number
- Under "Messaging", set Webhook URL: `https://yourdomain.com/api/webhooks/sms`
- Method: POST

**2. For local development, use ngrok:**
```bash
ngrok http 3000
# Use ngrok URL in Twilio console: https://abc123.ngrok.io/api/webhooks/sms
```

### Webhook Handler (Next.js API Route)

```javascript
// app/api/webhooks/sms/route.js
export async function POST(request) {
  const formData = await request.formData();
  
  const from = formData.get('From');        // Sender's phone: +15551234567
  const body = formData.get('Body');        // Message text: "YES"
  const messageSid = formData.get('MessageSid');
  
  // Parse response
  const response = body.trim().toUpperCase();
  
  if (response === 'YES') {
    // Sitter accepted
    await handleSitterAcceptance(from);
    
    // Send confirmation
    await client.messages.create({
      body: 'Perfect! Dylan will see you Saturday at 1:30pm. Details: https://app.com/sitting/123',
      from: process.env.TWILIO_PHONE_NUMBER,
      to: from
    });
  } else if (response === 'NO') {
    // Sitter declined
    await handleSitterDecline(from);
    
    // Thank them
    await client.messages.create({
      body: 'No problem! Thanks for letting us know.',
      from: process.env.TWILIO_PHONE_NUMBER,
      to: from
    });
  } else {
    // Unclear response - ask for clarification
    await client.messages.create({
      body: 'Please reply YES to accept or NO to decline.',
      from: process.env.TWILIO_PHONE_NUMBER,
      to: from
    });
  }
  
  // Twilio expects 200 OK
  return Response.json({ received: true });
}

async function handleSitterAcceptance(phone) {
  // Find which request this is for
  const request = await db.babysitterRequests.findFirst({
    where: {
      sitter: { phone: phone },
      status: 'pending'
    }
  });
  
  if (request) {
    // Mark as accepted
    await db.babysitterRequests.update({
      where: { id: request.id },
      data: { status: 'accepted', respondedAt: new Date() }
    });
    
    // Notify user
    await notifyUser(request.familyId, 'Babysitter confirmed!');
    
    // Update calendar event
    await updateCalendarWithSitterConfirmation(request);
  }
}
```

---

## Advanced Response Parsing

### Handle Natural Language Responses

```javascript
function parseResponse(body) {
  const normalized = body.trim().toLowerCase();
  
  // Acceptance patterns
  const acceptPatterns = [
    'yes', 'yeah', 'yep', 'sure', 'ok', 'okay',
    'i can do it', 'i can', 'count me in',
    'available', 'works for me'
  ];
  
  // Decline patterns
  const declinePatterns = [
    'no', 'nope', 'cant', "can't", 'sorry',
    'not available', 'unavailable', 'busy'
  ];
  
  if (acceptPatterns.some(p => normalized.includes(p))) {
    return 'accepted';
  }
  
  if (declinePatterns.some(p => normalized.includes(p))) {
    return 'declined';
  }
  
  return 'unclear';
}
```

### For MVP: Keep It Simple

```javascript
function parseResponse(body) {
  const text = body.trim().toUpperCase();
  
  if (text === 'YES' || text === 'Y') return 'accepted';
  if (text === 'NO' || text === 'N') return 'declined';
  
  return 'unclear';
}
```

---

## Message Status Tracking

```javascript
// Send with status callback
const message = await client.messages.create({
  body: 'Babysitter request...',
  from: twilioNumber,
  to: sitterPhone,
  statusCallback: 'https://yourdomain.com/api/webhooks/sms-status'
});

// Status webhook handler
// POST /api/webhooks/sms-status
export async function POST(request) {
  const formData = await request.formData();
  
  const messageSid = formData.get('MessageSid');
  const status = formData.get('MessageStatus');
  
  // Possible statuses:
  // - queued
  // - sent
  // - delivered
  // - failed
  // - undelivered
  
  if (status === 'delivered') {
    console.log('SMS delivered successfully');
  } else if (status === 'failed') {
    console.error('SMS failed to deliver');
    // Retry or use backup method (email)
  }
  
  return Response.json({ received: true });
}
```

---

## Pricing (Pay-As-You-Go)

**US SMS Costs:**
- **Outbound:** ~$0.0079 per message
- **Inbound:** ~$0.0079 per message

**Example for our app (100 users):**
- 100 users × 4 babysitter requests/month = 400 outbound messages
- 400 × 50% response rate = 200 inbound messages
- Total messages: 600/month
- Cost: 600 × $0.0079 = **$4.74/month**

**Free trial:** $15 credit (enough for ~1900 messages)

---

## Rate Limits

**Queues:**
- Long Code (regular number): 1 message/second
- Toll-Free: 3 messages/second
- Short Code: 100 messages/second (enterprise)

**For our use case:** Long code is fine (we're not sending mass messages)

---

## Testing

### Test with Virtual Phone

Twilio provides a Virtual Phone for testing:
- Number: `+18777804236`
- Test without needing real phone

### Test Webhook Locally

```bash
# Start your app
npm run dev

# In another terminal, start ngrok
ngrok http 3000

# Copy ngrok URL to Twilio console webhook settings
# Example: https://abc123.ngrok.io/api/webhooks/sms
```

---

## Error Handling

```javascript
try {
  const message = await client.messages.create({...});
} catch (error) {
  if (error.code === 21211) {
    // Invalid phone number
    console.error('Invalid phone format. Use E.164: +15551234567');
  } else if (error.code === 21608) {
    // Number not allowed (unverified in trial)
    console.error('Phone number not verified. Verify in Twilio console.');
  } else if (error.code === 20003) {
    // Authentication error
    console.error('Invalid credentials');
  } else {
    console.error('Twilio error:', error.message);
  }
}
```

**Common error codes:**
- `21211` - Invalid 'To' phone number
- `21608` - Phone number not verified (trial account)
- `20003` - Authentication failed
- `30003` - Unreachable destination
- `30005` - Unknown destination

---

## Next.js Integration Example

```typescript
// app/api/sitter/request/route.ts
import { NextResponse } from 'next/server';
import twilio from 'twilio';

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID,
  process.env.TWILIO_AUTH_TOKEN
);

export async function POST(request: Request) {
  const { sitterPhone, message } = await request.json();
  
  try {
    const sms = await client.messages.create({
      body: message,
      from: process.env.TWILIO_PHONE_NUMBER,
      to: sitterPhone
    });
    
    return NextResponse.json({ 
      success: true, 
      messageSid: sms.sid 
    });
  } catch (error) {
    console.error('Twilio error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

---

## Best Practices for Our Project

### 1. Phone Number Validation

```javascript
// Validate E.164 format before sending
function isValidPhone(phone) {
  const e164Regex = /^\+[1-9]\d{1,14}$/;
  return e164Regex.test(phone);
}

// Normalize phone input
function normalizePhone(phone) {
  // Remove spaces, dashes, parentheses
  const digits = phone.replace(/\D/g, '');
  
  // Add US country code if missing
  if (digits.length === 10) {
    return `+1${digits}`;
  }
  
  if (digits.length === 11 && digits.startsWith('1')) {
    return `+${digits}`;
  }
  
  return phone;  // Return as-is if unclear
}
```

### 2. Message Template System

```javascript
const SMS_TEMPLATES = {
  babysitterRequest: (data) => `Hi ${data.sitterName}!

${data.parentNames} need a babysitter:

📅 ${data.date}
⏰ ${data.startTime} - ${data.endTime} (${data.duration} hours)
👶 ${data.numKids} kids (ages ${data.ages})
💰 Pay: $${data.pay}
📍 ${data.address}
ℹ️ ${data.activity}

Can you do it?
→ Reply YES or NO by ${data.deadline}.`,

  confirmation: (data) => `Perfect! ${data.parentNames} will see you ${data.date} at ${data.startTime}. Details: ${data.link}`,
  
  decline: () => `No problem! Thanks for letting us know.`,
  
  reminder: (data) => `Reminder: Babysitting for ${data.parentNames} ${data.when} at ${data.address}`,
  
  cancellation: (data) => `Update: ${data.parentNames} cancelled ${data.date}. No longer need babysitter.`
};
```

### 3. Retry Logic

```javascript
async function sendWithRetry(to, body, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const message = await client.messages.create({
        body,
        from: process.env.TWILIO_PHONE_NUMBER,
        to
      });
      return { success: true, message };
    } catch (error) {
      if (i === maxRetries - 1) {
        // Last retry failed
        return { success: false, error: error.message };
      }
      // Wait before retry (exponential backoff)
      await new Promise(r => setTimeout(r, 1000 * Math.pow(2, i)));
    }
  }
}
```

### 4. Store Message History

```javascript
// After sending, store in database
await db.smsMessages.create({
  data: {
    messageSid: message.sid,
    to: sitter.phone,
    from: process.env.TWILIO_PHONE_NUMBER,
    body: message.body,
    status: 'sent',
    direction: 'outbound',
    babysitterRequestId: request.id,
    sentAt: new Date()
  }
});
```

---

## Receiving SMS (Inbound Webhooks)

### Next.js API Route

```typescript
// app/api/webhooks/sms/route.ts
import { NextResponse } from 'next/server';
import twilio from 'twilio';

export async function POST(request: Request) {
  const formData = await request.formData();
  
  const from = formData.get('From') as string;      // +15551234567
  const to = formData.get('To') as string;          // Your Twilio number
  const body = formData.get('Body') as string;      // Message text
  const messageSid = formData.get('MessageSid') as string;
  
  console.log(`Received SMS from ${from}: "${body}"`);
  
  // Find which babysitter request this is responding to
  const sitter = await db.trustedSitter.findFirst({
    where: { phone: from }
  });
  
  if (!sitter) {
    console.log('SMS from unknown number');
    return NextResponse.json({ received: true });
  }
  
  // Find pending request for this sitter
  const pendingRequest = await db.babysitterRequest.findFirst({
    where: {
      sitterId: sitter.id,
      status: 'pending'
    },
    include: { family: true, suggestion: true }
  });
  
  if (!pendingRequest) {
    console.log('No pending request for this sitter');
    return NextResponse.json({ received: true });
  }
  
  // Parse response
  const response = parseResponse(body);
  
  if (response === 'accepted') {
    await handleAcceptance(pendingRequest, from);
  } else if (response === 'declined') {
    await handleDecline(pendingRequest, from);
  } else {
    // Ask for clarification
    await sendClarificationRequest(from);
  }
  
  // Log the interaction
  await db.smsMessages.create({
    data: {
      messageSid,
      from,
      to,
      body,
      direction: 'inbound',
      babysitterRequestId: pendingRequest.id,
      receivedAt: new Date()
    }
  });
  
  return NextResponse.json({ received: true });
}

function parseResponse(body: string): 'accepted' | 'declined' | 'unclear' {
  const text = body.trim().toUpperCase();
  
  if (['YES', 'Y', 'YEP', 'YEAH', 'SURE', 'OK'].includes(text)) {
    return 'accepted';
  }
  
  if (['NO', 'N', 'NOPE', 'SORRY', 'CANT', "CAN'T"].includes(text)) {
    return 'declined';
  }
  
  return 'unclear';
}
```

---

## Message Delivery Status

### Track Delivery

```javascript
// Set status callback when sending
const message = await client.messages.create({
  body: 'Your message',
  from: twilioNumber,
  to: recipientNumber,
  statusCallback: 'https://yourdomain.com/api/webhooks/sms-status'
});

// Status webhook handler
export async function POST(request: Request) {
  const formData = await request.formData();
  
  const messageSid = formData.get('MessageSid');
  const status = formData.get('MessageStatus');
  
  // Update database
  await db.smsMessages.update({
    where: { messageSid },
    data: { 
      status,
      lastUpdated: new Date()
    }
  });
  
  // Status values:
  // - queued: Message accepted by Twilio
  // - sent: Sent to carrier
  // - delivered: Successfully delivered
  // - undelivered: Failed to deliver
  // - failed: Permanent failure
  
  if (status === 'failed' || status === 'undelivered') {
    // Notify user that SMS failed, suggest backup method
    await notifyUserOfSMSFailure(messageSid);
  }
  
  return Response.json({ received: true });
}
```

---

## Security: Validate Webhook Requests

**Twilio signs all webhooks** - verify signature to ensure requests are from Twilio:

```javascript
import twilio from 'twilio';

export async function POST(request: Request) {
  // Get Twilio signature from header
  const twilioSignature = request.headers.get('x-twilio-signature');
  
  // Get full URL
  const url = new URL(request.url).toString();
  
  // Get form data as object
  const formData = await request.formData();
  const params = Object.fromEntries(formData);
  
  // Validate signature
  const isValid = twilio.validateRequest(
    process.env.TWILIO_AUTH_TOKEN!,
    twilioSignature!,
    url,
    params
  );
  
  if (!isValid) {
    console.error('Invalid Twilio signature - possible spoofing attempt');
    return Response.json({ error: 'Invalid signature' }, { status: 403 });
  }
  
  // Process webhook...
  return Response.json({ received: true });
}
```

---

## Testing Strategy

### 1. Manual Testing (Development)
```javascript
// Send test SMS to your own phone
await client.messages.create({
  body: 'Test message',
  from: process.env.TWILIO_PHONE_NUMBER,
  to: '+15555551234'  // Your phone
});
```

### 2. Automated Testing (Mock Twilio)

```javascript
// __tests__/sms.test.js
jest.mock('twilio');

test('sends babysitter request', async () => {
  const mockCreate = jest.fn().mockResolvedValue({
    sid: 'SM123',
    status: 'sent'
  });
  
  twilio.mockReturnValue({
    messages: { create: mockCreate }
  });
  
  await sendBabysitterRequest(sitter, details);
  
  expect(mockCreate).toHaveBeenCalledWith({
    body: expect.stringContaining('babysitter'),
    from: expect.any(String),
    to: sitter.phone
  });
});
```

---

## Cost Optimization

### 1. Batch Messages (If Possible)
For multiple sitters, send individually but track at batch level:

```javascript
async function sendToMultipleSitters(sitters, message) {
  const results = await Promise.all(
    sitters.map(sitter => 
      client.messages.create({
        body: message,
        from: twilioNumber,
        to: sitter.phone
      }).catch(error => ({ error, sitter }))
    )
  );
  
  return results;
}
```

### 2. Use Status Callbacks Selectively
Only use `statusCallback` for critical messages:

```javascript
// Critical: Babysitter confirmation
await client.messages.create({
  body: 'Important confirmation...',
  statusCallback: webhookUrl  // Track delivery
});

// Non-critical: Reminder
await client.messages.create({
  body: 'Just a reminder...'
  // No status callback (save bandwidth)
});
```

---

## Alternatives to Consider

### If SMS is too expensive:
1. **Email + SMS fallback** (Email via SendGrid is cheaper)
2. **In-app notifications** (Push notifications via Firebase)
3. **WhatsApp Business API** (Twilio also supports this)

### For MVP:
Stick with SMS - it has highest open rate (98% vs 20% for email)

---

## Implementation Checklist

- [ ] Create Twilio account
- [ ] Get phone number with SMS capability
- [ ] Install `twilio` package
- [ ] Set up environment variables
- [ ] Create send SMS function
- [ ] Create webhook endpoint for receiving SMS
- [ ] Test with your own phone
- [ ] Implement response parsing (YES/NO)
- [ ] Add to database (store message history)
- [ ] Test with ngrok for local development
- [ ] Deploy webhook endpoint
- [ ] Update Twilio console with production webhook URL

---

## Links & Resources

- **Official Docs:** https://www.twilio.com/docs/sms
- **Node.js Quickstart:** https://www.twilio.com/docs/messaging/quickstart/node
- **API Reference:** https://www.twilio.com/docs/sms/api
- **GitHub (twilio-node):** https://github.com/twilio/twilio-node
- **Pricing:** https://www.twilio.com/sms/pricing

---

## For Our MVP

**What we need:**
- ✅ Send SMS to babysitters (`messages.create`)
- ✅ Receive YES/NO responses (webhook)
- ✅ Parse simple responses
- ✅ Send confirmations back

**What we can skip:**
- ❌ MMS (images)
- ❌ Complex NLP parsing
- ❌ Message scheduling
- ❌ Two-way conversations beyond YES/NO

**Estimated implementation time:** 3-5 days for full SMS integration
