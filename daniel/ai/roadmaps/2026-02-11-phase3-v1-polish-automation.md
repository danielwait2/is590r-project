# Phase 3: V1 Polish - Automation & Delivery

**Roadmap:** [2026-02-11-roadmap-phase3-v1-polish.md](./2026-02-11-roadmap-phase3-v1-polish.md) - Track progress

**Created:** 2026-02-11
**Timeline:** Weeks 5-8 (1 month)
**Status:** Pending Phase 2 Completion
**Type:** Production Enhancement

---

## Overview

Transform the manual web app from Phase 2 into an automated weekly system. Add email delivery via Resend, implement background jobs with BullMQ for calendar scanning and suggestion generation, and build feedback loops to improve recommendations over time.

**Key Addition:** Weekly automation - users no longer need to manually trigger suggestions; the system proactively sends them every Friday.

---

## Success Criteria

**Functional Requirements:**
- ✅ Suggestions sent automatically every week
- ✅ >50% email open rate
- ✅ >70% of accepted suggestions actually happen (validated via follow-up)
- ✅ User feedback collected and stored
- ✅ Weekly cron jobs run reliably (>99% success rate)

**User Experience:**
- Email digest is visually appealing and mobile-friendly
- "Add to Calendar" button works from email (generates .ics file)
- Follow-up surveys are simple (<30 seconds to complete)
- Feedback is reflected in future suggestions (no repeated rejections)

**Technical Requirements:**
- BullMQ + Redis queue operational
- Resend email service integrated
- Email templates render correctly (Gmail, Outlook, Apple Mail tested)
- SPF/DKIM/DMARC records configured
- Admin dashboard shows key metrics

---

## Prerequisites

**Required Before Starting:**
- ✅ Phase 2 completed (web app functional)
- ✅ 10-20 beta users onboarded and active
- ✅ Manual suggestion generation working
- ✅ Resend account created (free tier: 3,000 emails/month)
- ✅ Redis instance available (Railway or Upstash)

**Accounts Needed:**
- Resend.com account (for email delivery)
- Railway Redis add-on or Upstash Redis

---

## Detailed Implementation Steps

### 3A: Email System (Week 5)

**Goal:** Design and implement email delivery system with Resend

---

#### Day 1-2: Resend Integration

**1. Install Dependencies**

```bash
npm install resend
npm install react-email
npm install @react-email/components
```

**2. Configure Resend**

Add to `.env.local`:

```bash
RESEND_API_KEY=re_...
RESEND_FROM_EMAIL=suggestions@familytime.com
```

Set up custom domain in Resend dashboard:
```
1. Go to Resend dashboard → Domains
2. Add domain: familytime.com
3. Add DNS records (MX, TXT for SPF/DKIM)
4. Verify domain
```

**3. Create Email Template with React Email**

Create `src/emails/SuggestionDigest.tsx`:

```typescript
import {
  Body,
  Button,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Img,
  Link,
  Preview,
  Section,
  Text,
} from '@react-email/components';
import * as React from 'react';

interface Suggestion {
  id: string;
  title: string;
  description: string;
  address: string;
  estimatedCost: string;
  imageUrl: string;
  targetDate: string;
  targetTimeSlot: string;
}

interface SuggestionDigestProps {
  userName: string;
  suggestions: Suggestion[];
}

export default function SuggestionDigest({
  userName = 'Parent',
  suggestions = []
}: SuggestionDigestProps) {
  const previewText = `Your family time ideas for this weekend`;

  return (
    <Html>
      <Head />
      <Preview>{previewText}</Preview>
      <Body style={main}>
        <Container style={container}>
          <Heading style={h1}>Hi {userName}! ✨</Heading>

          <Text style={text}>
            We found some free time in your calendar this weekend. Here are 2 activity ideas
            your family might enjoy:
          </Text>

          {suggestions.map((suggestion, index) => (
            <Section key={suggestion.id} style={suggestionCard}>
              <Img
                src={suggestion.imageUrl}
                alt={suggestion.title}
                style={suggestionImage}
              />
              <Heading style={h2}>{suggestion.title}</Heading>
              <Text style={text}>{suggestion.description}</Text>

              <Text style={detailText}>
                📍 {suggestion.address}
              </Text>
              <Text style={detailText}>
                💰 {suggestion.estimatedCost}
              </Text>
              <Text style={detailText}>
                📅 {suggestion.targetDate} • {suggestion.targetTimeSlot}
              </Text>

              <Button
                style={button}
                href={`${process.env.NEXT_PUBLIC_APP_URL}/api/suggestions/${suggestion.id}/add-to-calendar`}
              >
                Add to Calendar
              </Button>

              <Hr style={hr} />
            </Section>
          ))}

          <Text style={footer}>
            Not interested? <Link href={`${process.env.NEXT_PUBLIC_APP_URL}/profile`}>Update your preferences</Link>
          </Text>
        </Container>
      </Body>
    </Html>
  );
}

// Styles
const main = {
  backgroundColor: '#f6f9fc',
  fontFamily: '-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Ubuntu,sans-serif',
};

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  padding: '20px 0 48px',
  marginBottom: '64px',
};

const h1 = {
  color: '#333',
  fontSize: '24px',
  fontWeight: 'bold',
  margin: '40px 0',
  padding: '0 40px',
};

const h2 = {
  color: '#333',
  fontSize: '20px',
  fontWeight: 'bold',
  margin: '16px 0 8px',
};

const text = {
  color: '#333',
  fontSize: '16px',
  lineHeight: '26px',
  padding: '0 40px',
};

const detailText = {
  color: '#666',
  fontSize: '14px',
  lineHeight: '24px',
  margin: '4px 0',
};

const suggestionCard = {
  padding: '24px 40px',
};

const suggestionImage = {
  width: '100%',
  maxWidth: '500px',
  height: 'auto',
  borderRadius: '8px',
  marginBottom: '16px',
};

const button = {
  backgroundColor: '#4F46E5',
  borderRadius: '5px',
  color: '#fff',
  fontSize: '16px',
  fontWeight: 'bold',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'block',
  width: '100%',
  padding: '12px',
  marginTop: '16px',
};

const hr = {
  borderColor: '#e6ebf1',
  margin: '20px 0',
};

const footer = {
  color: '#8898aa',
  fontSize: '12px',
  lineHeight: '16px',
  padding: '0 40px',
  marginTop: '32px',
};
```

**4. Create Email Service**

Create `src/lib/email.ts`:

```typescript
import { Resend } from 'resend';
import SuggestionDigest from '@/emails/SuggestionDigest';

const resend = new Resend(process.env.RESEND_API_KEY);

export async function sendSuggestionEmail(
  toEmail: string,
  userName: string,
  suggestions: any[]
) {
  try {
    const { data, error } = await resend.emails.send({
      from: process.env.RESEND_FROM_EMAIL!,
      to: toEmail,
      subject: "This weekend's family time ideas ✨",
      react: SuggestionDigest({ userName, suggestions }),
    });

    if (error) {
      console.error('Resend error:', error);
      throw error;
    }

    return data;
  } catch (error) {
    console.error('Email send failed:', error);
    throw error;
  }
}

export async function sendFollowUpSurvey(
  toEmail: string,
  userName: string,
  suggestionId: string
) {
  // Implementation similar to above
  // Email template: "How was [Activity]?"
  // Link to feedback form
}
```

**5. Test Email Rendering**

```bash
# Start React Email preview server
npm run email:dev

# This opens http://localhost:3000 with email preview
# Test responsiveness and appearance
```

Test in real email clients:
- Gmail (desktop + mobile)
- Outlook
- Apple Mail
- Verify "Add to Calendar" button works

**Reference:**
- [Resend Documentation](../guides/resend_context7.md)
- [React Email Documentation](../guides/react-email_context7.md)

---

#### Day 3: .ics Calendar File Generation

**1. Install ics Library**

```bash
npm install ics
```

**2. Create Calendar File Endpoint**

Create `src/app/api/suggestions/[id]/add-to-calendar/route.ts`:

```typescript
import { ics } from 'ics';
import { prisma } from '@/lib/db';
import { NextResponse } from 'next/server';

export async function GET(
  req: Request,
  { params }: { params: { id: string } }
) {
  const suggestion = await prisma.suggestion.findUnique({
    where: { id: params.id }
  });

  if (!suggestion) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 });
  }

  // Parse target date and time
  const targetDate = new Date(suggestion.targetDate);
  const [startTime, endTime] = suggestion.targetTimeSlot.split(' - ');

  const event = {
    start: [
      targetDate.getFullYear(),
      targetDate.getMonth() + 1,
      targetDate.getDate(),
      parseInt(startTime.split(':')[0]),
      parseInt(startTime.split(':')[1])
    ],
    duration: { hours: 2 }, // Default 2 hours
    title: suggestion.title,
    description: suggestion.description,
    location: suggestion.address,
    url: suggestion.bookingLink || undefined,
    status: 'TENTATIVE' as const,
    busyStatus: 'BUSY' as const,
  };

  const { error, value } = ics.createEvent(event);

  if (error) {
    return NextResponse.json({ error }, { status: 500 });
  }

  // Mark suggestion as accepted
  await prisma.suggestion.update({
    where: { id: params.id },
    data: { status: 'accepted' }
  });

  return new NextResponse(value, {
    headers: {
      'Content-Type': 'text/calendar',
      'Content-Disposition': `attachment; filename="${suggestion.title}.ics"`,
    },
  });
}
```

**Testing:**
- Click "Add to Calendar" in email
- Verify .ics file downloads
- Open in Google Calendar
- Confirm event appears correctly

---

#### Day 4-5: Email Deliverability Setup

**1. Configure SPF Record**

Add to DNS (your domain registrar):

```
Type: TXT
Name: @
Value: v=spf1 include:_spf.resend.com ~all
```

**2. Configure DKIM**

Resend provides DKIM keys automatically when you verify domain.

**3. Configure DMARC**

Add to DNS:

```
Type: TXT
Name: _dmarc
Value: v=DMARC1; p=none; rua=mailto:dmarc@familytime.com
```

**4. Test Deliverability**

Use https://www.mail-tester.com:
- Send test email to provided address
- Check spam score (target: 10/10)
- Fix any issues flagged

**5. Monitor Email Metrics**

Create `src/app/api/admin/email-metrics/route.ts`:

```typescript
import { prisma } from '@/lib/db';
import { NextResponse } from 'next/server';

export async function GET() {
  // Track email open rates (Resend webhook integration)
  // Track click rates on "Add to Calendar"
  // Display in admin dashboard

  return NextResponse.json({
    totalSent: 0,
    openRate: 0,
    clickRate: 0
  });
}
```

---

### 3B: Background Jobs (Week 6)

**Goal:** Implement automated weekly cron jobs with BullMQ

---

#### Day 1-2: BullMQ + Redis Setup

**1. Add Redis to Railway**

```
1. Go to Railway project
2. Add Redis service
3. Copy connection string (REDIS_URL)
4. Add to .env.local
```

**2. Install BullMQ**

```bash
npm install bullmq ioredis
```

**3. Create Queue Configuration**

Create `src/lib/queue.ts`:

```typescript
import { Queue, Worker, QueueScheduler } from 'bullmq';
import IORedis from 'ioredis';

const connection = new IORedis(process.env.REDIS_URL!, {
  maxRetriesPerRequest: null,
});

// Queues
export const suggestionQueue = new Queue('suggestions', { connection });
export const emailQueue = new Queue('emails', { connection });
export const followUpQueue = new Queue('followups', { connection });

// Scheduler (for cron-like jobs)
export const scheduler = new QueueScheduler('suggestions', { connection });
```

**4. Create Job Processors**

Create `src/workers/suggestionWorker.ts`:

```typescript
import { Worker } from 'bullmq';
import { prisma } from '@/lib/db';
import { generateSuggestions } from '@/lib/recommendations';
import { emailQueue } from '@/lib/queue';

const worker = new Worker(
  'suggestions',
  async (job) => {
    console.log(`Processing suggestion job for user: ${job.data.userId}`);

    const { userId } = job.data;

    // 1. Fetch user profile
    const user = await prisma.user.findUnique({
      where: { id: userId },
      include: { profile: true }
    });

    if (!user || !user.profile) {
      throw new Error('User or profile not found');
    }

    // 2. Generate suggestions
    const suggestions = await generateSuggestions(userId);

    if (suggestions.length === 0) {
      console.log('No suggestions generated for user:', userId);
      return;
    }

    // 3. Queue email job
    await emailQueue.add('send-suggestion-email', {
      userId,
      suggestions: suggestions.map(s => s.id),
    });

    return { suggestionsGenerated: suggestions.length };
  },
  {
    connection: new IORedis(process.env.REDIS_URL!),
  }
);

worker.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

worker.on('failed', (job, err) => {
  console.error(`Job ${job?.id} failed:`, err);
});

export default worker;
```

Create `src/workers/emailWorker.ts`:

```typescript
import { Worker } from 'bullmq';
import { prisma } from '@/lib/db';
import { sendSuggestionEmail } from '@/lib/email';
import IORedis from 'ioredis';

const worker = new Worker(
  'emails',
  async (job) => {
    console.log(`Sending email for user: ${job.data.userId}`);

    const { userId, suggestions: suggestionIds } = job.data;

    const user = await prisma.user.findUnique({
      where: { id: userId },
    });

    const suggestions = await prisma.suggestion.findMany({
      where: { id: { in: suggestionIds } }
    });

    if (!user || suggestions.length === 0) {
      throw new Error('User or suggestions not found');
    }

    await sendSuggestionEmail(user.email, user.email, suggestions);

    return { emailSent: true };
  },
  {
    connection: new IORedis(process.env.REDIS_URL!),
  }
);

export default worker;
```

**5. Start Workers**

Create `src/workers/index.ts`:

```typescript
import suggestionWorker from './suggestionWorker';
import emailWorker from './emailWorker';

console.log('Workers started');

// Keep process alive
process.on('SIGTERM', async () => {
  await suggestionWorker.close();
  await emailWorker.close();
  process.exit(0);
});
```

Add to `package.json`:

```json
{
  "scripts": {
    "worker": "tsx src/workers/index.ts"
  }
}
```

**Reference:**
- [BullMQ Documentation](../guides/bullmq_context7.md)

---

#### Day 3-4: Cron Job Setup

**1. Create Cron Job Scheduler**

Create `src/lib/cron.ts`:

```typescript
import { suggestionQueue, emailQueue, followUpQueue } from './queue';
import { prisma } from './db';

export async function scheduleWeeklySuggestions() {
  console.log('Scheduling weekly suggestion jobs...');

  // Get all active users
  const users = await prisma.user.findMany({
    where: { onboardingCompleted: true },
    select: { id: true }
  });

  // Queue suggestion generation for each user
  for (const user of users) {
    await suggestionQueue.add(
      'generate-suggestions',
      { userId: user.id },
      {
        // Run every Sunday at 8pm
        repeat: {
          pattern: '0 20 * * 0', // Cron: 8pm every Sunday
        },
      }
    );
  }

  console.log(`Scheduled jobs for ${users.length} users`);
}

export async function scheduleWeeklyEmails() {
  // Emails are queued by suggestion worker
  // This function could add retry logic or scheduling
}

export async function scheduleFollowUpSurveys() {
  // Get suggestions from last weekend
  const suggestions = await prisma.suggestion.findMany({
    where: {
      status: 'accepted',
      targetDate: {
        gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000), // Last 7 days
      },
    },
    include: { user: true }
  });

  for (const suggestion of suggestions) {
    await followUpQueue.add(
      'send-follow-up',
      { suggestionId: suggestion.id },
      {
        // Run every Monday at 10am
        repeat: {
          pattern: '0 10 * * 1', // Cron: 10am every Monday
        },
      }
    );
  }
}
```

**2. Initialize Cron Jobs on Server Start**

Create `src/app/api/cron/init/route.ts`:

```typescript
import { scheduleWeeklySuggestions } from '@/lib/cron';
import { NextResponse } from 'next/server';

export async function POST(req: Request) {
  // Secure this endpoint (check admin token)
  const authHeader = req.headers.get('authorization');
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  await scheduleWeeklySuggestions();

  return NextResponse.json({ success: true });
}
```

**3. Deploy Workers to Railway**

Create separate Railway service for workers:

```
1. In Railway, add new service
2. Link to same GitHub repo
3. Set start command: npm run worker
4. Add all environment variables
5. Deploy
```

**4. Monitor Job Queue**

Use BullBoard for UI dashboard:

```bash
npm install @bull-board/express @bull-board/api
```

Create `src/app/api/admin/queue/route.ts`:

```typescript
import { createBullBoard } from '@bull-board/api';
import { BullMQAdapter } from '@bull-board/api/bullMQAdapter';
import { ExpressAdapter } from '@bull-board/express';
import { suggestionQueue, emailQueue } from '@/lib/queue';

const serverAdapter = new ExpressAdapter();

createBullBoard({
  queues: [
    new BullMQAdapter(suggestionQueue),
    new BullMQAdapter(emailQueue),
  ],
  serverAdapter,
});

serverAdapter.setBasePath('/api/admin/queue');

// Export as Next.js API route handler
```

Access at: `https://familytime.com/api/admin/queue`

---

#### Day 5: Testing & Monitoring

**Test Scenarios:**
```
1. Manual trigger: POST /api/cron/init
2. Verify jobs queued for all users
3. Check worker logs in Railway
4. Confirm emails sent
5. Test failure scenarios (invalid user, API errors)
```

**Monitoring Setup:**
```
- Railway logs for worker errors
- BullBoard dashboard for queue health
- Resend dashboard for email delivery
- Database queries for job completion rates
```

---

### 3C: Feedback Collection (Week 7)

**Goal:** Build feedback system and admin analytics dashboard

---

#### Implementation Tasks:

**1. Feedback API Endpoints**

Create `src/app/api/suggestions/[id]/feedback/route.ts`:

```typescript
import { prisma } from '@/lib/db';
import { NextResponse } from 'next/server';

export async function POST(
  req: Request,
  { params }: { params: { id: string } }
) {
  const body = await req.json();
  const { feedback, comment } = body; // feedback: 'thumbs_up' | 'thumbs_down'

  const updated = await prisma.suggestion.update({
    where: { id: params.id },
    data: {
      feedback,
      feedbackComment: comment || null,
    },
  });

  return NextResponse.json({ success: true, suggestion: updated });
}
```

**2. Feedback Form Component**

Create `src/components/FeedbackWidget.tsx`:

```typescript
'use client';

import { useState } from 'react';

export default function FeedbackWidget({ suggestionId }: { suggestionId: string }) {
  const [submitted, setSubmitted] = useState(false);

  const handleFeedback = async (type: 'thumbs_up' | 'thumbs_down') => {
    await fetch(`/api/suggestions/${suggestionId}/feedback`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ feedback: type }),
    });

    setSubmitted(true);
  };

  if (submitted) {
    return <p className="text-green-600">Thanks for your feedback!</p>;
  }

  return (
    <div className="flex gap-4 items-center">
      <p>Was this suggestion helpful?</p>
      <button onClick={() => handleFeedback('thumbs_up')}>👍</button>
      <button onClick={() => handleFeedback('thumbs_down')}>👎</button>
    </div>
  );
}
```

**3. Admin Analytics Dashboard**

Create `src/app/admin/page.tsx`:

```typescript
import { prisma } from '@/lib/db';

export default async function AdminDashboard() {
  const totalSuggestions = await prisma.suggestion.count();
  const acceptedCount = await prisma.suggestion.count({
    where: { status: 'accepted' }
  });
  const acceptanceRate = (acceptedCount / totalSuggestions) * 100;

  const feedbackStats = await prisma.suggestion.groupBy({
    by: ['feedback'],
    _count: true,
  });

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-8">Admin Dashboard</h1>

      <div className="grid grid-cols-3 gap-6">
        <div className="bg-white p-6 rounded-lg shadow">
          <h3 className="text-gray-500">Total Suggestions</h3>
          <p className="text-4xl font-bold">{totalSuggestions}</p>
        </div>

        <div className="bg-white p-6 rounded-lg shadow">
          <h3 className="text-gray-500">Acceptance Rate</h3>
          <p className="text-4xl font-bold">{acceptanceRate.toFixed(1)}%</p>
        </div>

        <div className="bg-white p-6 rounded-lg shadow">
          <h3 className="text-gray-500">Thumbs Up Rate</h3>
          <p className="text-4xl font-bold">
            {/* Calculate from feedbackStats */}
          </p>
        </div>
      </div>

      {/* Add charts with Chart.js or Recharts */}
    </div>
  );
}
```

---

### 3D: Recommendation Improvements (Week 8)

**Goal:** Use feedback to improve future suggestions

---

#### Implementation:

**1. Feedback-Based Filtering**

Update `src/lib/recommendations.ts`:

```typescript
export async function generateSuggestions(userId: string) {
  // ... existing code ...

  // Get rejected activity types
  const rejectedTypes = await prisma.suggestion.findMany({
    where: {
      userId,
      feedback: 'thumbs_down',
    },
    select: { activityType: true },
  });

  const rejectedTypesList = rejectedTypes.map(s => s.activityType);

  // Filter out rejected types from candidates
  const filteredActivities = activities.filter(
    activity => !rejectedTypesList.includes(activity.type)
  );

  // ... continue with scoring ...
}
```

**2. Recency Penalty**

```typescript
// Don't suggest same place twice within 4 weeks
const recentSuggestions = await prisma.suggestion.findMany({
  where: {
    userId,
    generatedAt: {
      gte: new Date(Date.now() - 28 * 24 * 60 * 60 * 1000), // 4 weeks
    },
  },
});

const recentTitles = recentSuggestions.map(s => s.title);

const newActivities = activities.filter(
  activity => !recentTitles.includes(activity.title)
);
```

**3. Diversity Scoring**

```typescript
// Vary activity types week-to-week
const lastWeekType = await prisma.suggestion.findFirst({
  where: {
    userId,
    generatedAt: {
      gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000),
    },
  },
  orderBy: { generatedAt: 'desc' },
});

// Penalize same type as last week
if (lastWeekType) {
  scoredActivities.forEach(item => {
    if (item.activity.type === lastWeekType.activityType) {
      item.score -= 10;
    }
  });
}
```

---

## Deliverables

**Code:**
- ✅ Email templates (React Email)
- ✅ BullMQ workers for background jobs
- ✅ Cron job scheduler
- ✅ Feedback API endpoints
- ✅ Admin analytics dashboard

**Infrastructure:**
- ✅ Resend email service configured
- ✅ Redis queue operational on Railway
- ✅ SPF/DKIM/DMARC DNS records
- ✅ Worker service deployed

**Documentation:**
- ✅ Email template style guide
- ✅ Cron job schedule documentation
- ✅ Admin dashboard user guide

---

## Risks & Mitigation

**Email Deliverability Issues:**
- Mitigation: Proper DNS configuration, test with mail-tester.com
- Contingency: Use SendGrid as backup

**Job Queue Failures:**
- Mitigation: BullMQ retry logic, monitoring
- Contingency: Manual trigger endpoint as fallback

**Suggestions Not Improving:**
- Mitigation: Collect detailed feedback, A/B test algorithms
- Contingency: Revert to manual curation

---

## References

- [High-Level Roadmap](./2026-02-11-high-level_mvp-project-roadmap.md)
- [Architecture Document](../../aiDocs/architecture.md)
- [Resend Docs](../guides/resend_context7.md)
- [BullMQ Docs](../guides/bullmq_context7.md)
- [React Email Docs](../guides/react-email_context7.md)

---

**Last Updated:** 2026-02-11
**Version:** 1.0
