# Phase 4: V1 Launch - Beta Testing

**Roadmap:** [2026-02-11-roadmap-phase4-v1-launch.md](./2026-02-11-roadmap-phase4-v1-launch.md) - Track progress

**Created:** 2026-02-11
**Timeline:** Weeks 9-12 (1 month)
**Status:** Pending Phase 3 Completion
**Type:** Production Launch & Validation

---

## Overview

Launch to 100 beta users, validate product-market fit, test monetization willingness, and make the final go/no-go decision based on data. This is the culmination of the 12-week V1 timeline outlined in the PRD.

**Critical Outcome:** By end of Week 12, you should know if the product is worth scaling (Phase 5) or if a pivot is needed.

---

## Success Criteria

**User Metrics (from PRD):**
- ✅ 100 active users
- ✅ >60% week-2 retention
- ✅ >40% suggestion acceptance rate
- ✅ >30% would pay $8-10/month
- ✅ NPS score >40

**Validation Goals:**
- Understand which activity types drive highest acceptance
- Identify user segments with best retention (couples vs families)
- Validate unit economics (cost per user vs willingness to pay)
- Gather qualitative feedback on product value

**Technical Requirements:**
- Payment flow functional (Stripe integrated)
- Free tier limits enforced (1 suggestion/week)
- Premium tier features working (3 suggestions/week)
- Metrics dashboard tracking all KPIs

---

## Prerequisites

**Required Before Starting:**
- ✅ Phase 3 completed (automation working)
- ✅ 50+ beta users from Phase 2 still active
- ✅ Email system delivering reliably (>95% delivery rate)
- ✅ No critical bugs in production
- ✅ Stripe account created

**Marketing Preparation:**
- Landing page optimized for conversions
- Beta waitlist sign-up form ready
- Referral invite codes system built
- Social media accounts created (optional)

---

## Detailed Implementation Steps

### 4A: User Acquisition (Week 9)

**Goal:** Recruit 100 beta users through multiple channels

---

#### Day 1-2: Beta Waitlist & Landing Page

**1. Create Waitlist Form**

Create `src/components/WaitlistForm.tsx`:

```typescript
'use client';

import { useState } from 'react';

export default function WaitlistForm() {
  const [email, setEmail] = useState('');
  const [submitted, setSubmitted] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    await fetch('/api/waitlist', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    });

    setSubmitted(true);
  };

  if (submitted) {
    return (
      <div className="bg-green-50 p-6 rounded-lg">
        <p className="text-green-800 font-semibold">
          You're on the list! We'll send you an invite soon.
        </p>
      </div>
    );
  }

  return (
    <form onSubmit={handleSubmit} className="flex gap-2">
      <input
        type="email"
        placeholder="Enter your email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        required
        className="flex-1 px-4 py-3 border rounded-lg"
      />
      <button
        type="submit"
        className="bg-blue-600 text-white px-6 py-3 rounded-lg font-semibold hover:bg-blue-700"
      >
        Join Waitlist
      </button>
    </form>
  );
}
```

**2. Waitlist API**

Create `src/app/api/waitlist/route.ts`:

```typescript
import { prisma } from '@/lib/db';
import { NextResponse } from 'next/server';

export async function POST(req: Request) {
  const { email } = await req.json();

  await prisma.waitlist.create({
    data: { email },
  });

  // Optional: Send confirmation email via Resend

  return NextResponse.json({ success: true });
}
```

Add to Prisma schema:

```prisma
model Waitlist {
  id        String   @id @default(uuid())
  email     String   @unique
  createdAt DateTime @default(now())
}
```

**3. Update Landing Page**

Add social proof and urgency:

```typescript
<section className="bg-white py-16">
  <div className="container mx-auto px-4 text-center">
    <h2 className="text-3xl font-bold mb-4">Join 50+ Families Already Using FamilyTime</h2>
    <p className="text-xl text-gray-600 mb-8">
      Limited beta spots available. Join the waitlist today.
    </p>
    <WaitlistForm />
  </div>
</section>
```

---

#### Day 3-4: Referral System

**1. Generate Invite Codes**

Create `src/lib/referrals.ts`:

```typescript
import crypto from 'crypto';
import { prisma } from './db';

export async function generateInviteCodes(userId: string, count: number = 3) {
  const codes = [];

  for (let i = 0; i < count; i++) {
    const code = crypto.randomBytes(4).toString('hex').toUpperCase(); // e.g., "A3F2B9C1"

    await prisma.inviteCode.create({
      data: {
        code,
        userId,
        maxUses: 1,
        usedCount: 0,
      },
    });

    codes.push(code);
  }

  return codes;
}

export async function validateInviteCode(code: string): Promise<boolean> {
  const invite = await prisma.inviteCode.findUnique({
    where: { code },
  });

  if (!invite) return false;
  if (invite.usedCount >= invite.maxUses) return false;

  return true;
}
```

Add to Prisma schema:

```prisma
model InviteCode {
  id         String   @id @default(uuid())
  code       String   @unique
  userId     String
  user       User     @relation(fields: [userId], references: [id])
  maxUses    Int      @default(1)
  usedCount  Int      @default(0)
  createdAt  DateTime @default(now())

  @@index([code])
}
```

**2. Invite Code Redemption**

Update sign-up flow to accept invite codes:

```typescript
// src/app/sign-up/page.tsx
export default function SignUpPage({
  searchParams,
}: {
  searchParams: { invite?: string };
}) {
  const inviteCode = searchParams.invite;

  return (
    <div>
      {inviteCode && (
        <p className="mb-4 text-green-600">
          You've been invited! Code: {inviteCode}
        </p>
      )}
      <SignUp />
    </div>
  );
}
```

**3. Share Invite Page**

Create `src/app/invite/page.tsx`:

```typescript
import { currentUser } from '@clerk/nextjs';
import { prisma } from '@/lib/db';
import { generateInviteCodes } from '@/lib/referrals';

export default async function InvitePage() {
  const user = await currentUser();
  if (!user) return null;

  const dbUser = await prisma.user.findUnique({
    where: { clerkUserId: user.id },
    include: { inviteCodes: true },
  });

  if (!dbUser) return null;

  // Generate codes if user doesn't have any
  if (dbUser.inviteCodes.length === 0) {
    await generateInviteCodes(dbUser.id, 3);
  }

  const codes = await prisma.inviteCode.findMany({
    where: { userId: dbUser.id },
  });

  return (
    <div className="container mx-auto px-4 py-12">
      <h1 className="text-3xl font-bold mb-6">Invite Friends</h1>
      <p className="mb-8">Share FamilyTime with friends and get early access to new features!</p>

      <div className="space-y-4">
        {codes.map((code) => (
          <div key={code.id} className="bg-white p-4 rounded-lg border">
            <p className="font-mono text-xl">{code.code}</p>
            <p className="text-sm text-gray-600">
              {code.usedCount}/{code.maxUses} used
            </p>
            <button
              onClick={() => {
                const url = `${window.location.origin}/sign-up?invite=${code.code}`;
                navigator.clipboard.writeText(url);
                alert('Invite link copied!');
              }}
              className="mt-2 bg-blue-600 text-white px-4 py-2 rounded"
            >
              Copy Invite Link
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

#### Day 5-7: Launch Channels

**1. Reddit/Facebook Launch**

Target subreddits:
- r/Parenting
- r/workingmoms
- r/daddit

Facebook groups:
- Local parent groups
- Working parents communities

**Launch Post Template:**

```
Subject: I built a tool to help busy families actually use their free time

Body:
Hey everyone! I'm a parent who got tired of wasting weekends scrolling instead of doing fun things with my kids.

I built a tool that:
- Scans your Google Calendar for free time
- Suggests 2 local family activities every week
- Based on your kids' ages and interests

It's free during beta. Would love your feedback!

[Link to landing page]

(Mods: Let me know if this isn't allowed and I'll remove)
```

**2. Email Influencer Outreach**

Create list of 50 parent bloggers/influencers:

```
Template:
---
Subject: Partnership idea for your audience

Hi [Name],

I'm reaching out because I built something I think your audience would love: a tool that helps busy parents turn free time into quality family time.

Would you be interested in:
- Trying it out (free beta access)
- Sharing with your audience if you like it
- Exclusive invite codes for your followers

Let me know if you'd like to learn more!

Best,
[Your name]
```

**3. Track Acquisition Metrics**

Create `src/app/api/admin/acquisition/route.ts`:

```typescript
export async function GET() {
  const stats = await prisma.user.groupBy({
    by: ['source'], // Track where users came from
    _count: true,
  });

  return NextResponse.json(stats);
}
```

Add `source` field to User model (referral, reddit, facebook, influencer, organic)

---

### 4B: User Experience Refinement (Week 10)

**Goal:** Improve UX based on real user feedback and data

---

#### Tasks:

**1. User Interviews (5-10 users)**

Interview script:

```
Introduction (5 min):
- Thank them for their time
- Explain you want honest feedback
- Permission to record

Usage Questions (15 min):
1. How did you first use FamilyTime?
2. What was confusing or unclear?
3. Have you accepted any suggestions?
4. Why or why not?
5. What would make you use it more?

Suggestion Quality (10 min):
6. How relevant were the suggestions? (1-10)
7. Were they activities you'd actually do?
8. What was missing?
9. Any you rejected? Why?

Future Features (5 min):
10. What would make this a "must-have" tool?
11. Would you pay for this? How much?

Thank you (5 min):
- Gift them premium access
- Ask for permission to follow up
```

**2. Analyze Onboarding Drop-off**

Create analytics tracking:

```typescript
// Track funnel steps
await prisma.analyticsEvent.create({
  data: {
    userId,
    event: 'onboarding_step_completed',
    properties: { step: 'family_profile' },
  },
});

// Funnel analysis query
const funnel = await prisma.analyticsEvent.groupBy({
  by: ['event'],
  where: {
    event: {
      in: [
        'signup_started',
        'profile_completed',
        'calendar_connected',
        'first_suggestion_received',
      ],
    },
  },
  _count: true,
});
```

**3. A/B Test Email Subject Lines**

```typescript
const subjectLines = [
  "Your family time ideas for this weekend ✨",
  "2 fun activities for your free time this Saturday",
  "We found free time in your calendar - here's what to do!",
];

const randomSubject = subjectLines[Math.floor(Math.random() * subjectLines.length)];

await resend.emails.send({
  subject: randomSubject,
  // ... rest of email
});

// Track open rates by subject line
```

**4. Optimize Onboarding Flow**

Based on drop-off analysis, simplify:

```
Before: 7 steps (signup → email verify → profile → calendar → preferences → review → done)
After: 4 steps (signup → quick profile → calendar → done)

Move detailed preferences to post-onboarding
```

**5. Fix Top 5 Bugs**

Create priority bug list:

```markdown
# Bug Tracker

## P0 (Critical - Blocks core flow)
1. [BUG-001] Calendar OAuth fails on mobile Safari
   - Impact: 20% of users can't connect calendar
   - Fix: Add fallback OAuth flow

## P1 (High - Degrades experience)
2. [BUG-002] Email "Add to Calendar" button 404s
   - Impact: 15% of clicks fail
   - Fix: Update endpoint URL in email template

...
```

---

### 4C: Monetization Experiment (Week 11)

**Goal:** Integrate Stripe and test willingness to pay

---

#### Day 1-2: Stripe Integration

**1. Install Stripe**

```bash
npm install stripe @stripe/stripe-js
```

**2. Create Stripe Products**

```typescript
// Create products in Stripe dashboard or via API
const products = [
  {
    name: 'FamilyTime Free',
    price: 0,
    features: ['1 suggestion per week', 'Email delivery', 'Basic support'],
  },
  {
    name: 'FamilyTime Premium',
    price: 999, // $9.99/month in cents
    features: [
      '3 suggestions per week',
      'Multi-calendar support',
      'SMS notifications',
      'Priority support',
    ],
  },
];
```

**3. Pricing Page**

Create `src/app/pricing/page.tsx`:

```typescript
import { SignedIn, SignedOut } from '@clerk/nextjs';
import Link from 'next/link';

export default function PricingPage() {
  return (
    <div className="container mx-auto px-4 py-12">
      <h1 className="text-4xl font-bold text-center mb-12">
        Choose Your Plan
      </h1>

      <div className="grid md:grid-cols-2 gap-8 max-w-4xl mx-auto">
        {/* Free Tier */}
        <div className="bg-white rounded-lg shadow-lg p-8 border-2 border-gray-200">
          <h2 className="text-2xl font-bold mb-4">Free</h2>
          <p className="text-4xl font-bold mb-6">$0<span className="text-lg text-gray-600">/month</span></p>

          <ul className="space-y-3 mb-8">
            <li>✓ 1 suggestion per week</li>
            <li>✓ Email delivery</li>
            <li>✓ Basic support</li>
          </ul>

          <SignedOut>
            <Link href="/sign-up" className="block w-full bg-gray-600 text-white text-center py-3 rounded-lg font-semibold hover:bg-gray-700">
              Get Started
            </Link>
          </SignedOut>
        </div>

        {/* Premium Tier */}
        <div className="bg-blue-600 text-white rounded-lg shadow-lg p-8 border-2 border-blue-700 relative">
          <div className="absolute -top-4 left-1/2 transform -translate-x-1/2 bg-yellow-400 text-blue-900 px-4 py-1 rounded-full text-sm font-semibold">
            RECOMMENDED
          </div>

          <h2 className="text-2xl font-bold mb-4">Premium</h2>
          <p className="text-4xl font-bold mb-6">$9<span className="text-lg">/month</span></p>

          <ul className="space-y-3 mb-8">
            <li>✓ 3 suggestions per week</li>
            <li>✓ Multi-calendar support</li>
            <li>✓ SMS notifications</li>
            <li>✓ Priority support</li>
            <li>✓ Early access to new features</li>
          </ul>

          <SignedIn>
            <Link href="/api/checkout" className="block w-full bg-white text-blue-600 text-center py-3 rounded-lg font-semibold hover:bg-gray-100">
              Upgrade to Premium
            </Link>
          </SignedIn>

          <SignedOut>
            <Link href="/sign-up" className="block w-full bg-white text-blue-600 text-center py-3 rounded-lg font-semibold hover:bg-gray-100">
              Start Free Trial
            </Link>
          </SignedOut>
        </div>
      </div>

      <p className="text-center text-gray-600 mt-8">
        Free tier available forever. Upgrade or cancel anytime.
      </p>
    </div>
  );
}
```

**4. Checkout Flow**

Create `src/app/api/checkout/route.ts`:

```typescript
import { auth, currentUser } from '@clerk/nextjs';
import { NextResponse } from 'next/server';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

export async function GET() {
  const { userId } = auth();
  const user = await currentUser();

  if (!userId || !user) {
    return NextResponse.redirect('/sign-in');
  }

  const session = await stripe.checkout.sessions.create({
    customer_email: user.emailAddresses[0].emailAddress,
    line_items: [
      {
        price: process.env.STRIPE_PRICE_ID!, // Premium price ID
        quantity: 1,
      },
    ],
    mode: 'subscription',
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard?upgraded=true`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
    metadata: {
      userId,
    },
  });

  return NextResponse.redirect(session.url!);
}
```

**5. Webhook Handler**

Create `src/app/api/webhooks/stripe/route.ts`:

```typescript
import { NextResponse } from 'next/server';
import Stripe from 'stripe';
import { prisma } from '@/lib/db';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

export async function POST(req: Request) {
  const body = await req.text();
  const sig = req.headers.get('stripe-signature')!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(
      body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch (err) {
    return NextResponse.json({ error: 'Webhook signature verification failed' }, { status: 400 });
  }

  if (event.type === 'checkout.session.completed') {
    const session = event.data.object as Stripe.Checkout.Session;

    await prisma.user.update({
      where: { clerkUserId: session.metadata!.userId },
      data: {
        subscriptionTier: 'premium',
        stripeCustomerId: session.customer as string,
      },
    });
  }

  return NextResponse.json({ received: true });
}
```

**Reference:**
- [Stripe Documentation](../guides/stripe_context7.md)

---

#### Day 3-5: Paywall Implementation

**1. Enforce Free Tier Limits**

```typescript
// In suggestion generation worker
const user = await prisma.user.findUnique({
  where: { id: userId },
  include: { suggestions: true },
});

const thisWeekSuggestions = user.suggestions.filter(
  s => s.generatedAt >= startOfWeek(new Date())
);

if (user.subscriptionTier === 'free' && thisWeekSuggestions.length >= 1) {
  console.log('Free tier limit reached for user:', userId);
  return; // Don't generate more suggestions
}

if (user.subscriptionTier === 'premium' && thisWeekSuggestions.length >= 3) {
  console.log('Premium tier limit reached for user:', userId);
  return;
}
```

**2. Upgrade Prompts**

Show upgrade prompts in UI when free user hits limits:

```typescript
// In dashboard
{user.subscriptionTier === 'free' && thisWeekSuggestions.length >= 1 && (
  <div className="bg-yellow-50 border border-yellow-200 rounded-lg p-6 mb-6">
    <h3 className="font-semibold mb-2">You've received your free suggestion this week</h3>
    <p className="text-sm mb-4">Upgrade to Premium to get 2 more suggestions this week!</p>
    <Link href="/pricing" className="bg-blue-600 text-white px-4 py-2 rounded">
      Upgrade Now
    </Link>
  </div>
)}
```

**3. Track Conversion Metrics**

```typescript
await prisma.analyticsEvent.create({
  data: {
    userId,
    event: 'paywall_shown',
    properties: { context: 'weekly_limit_reached' },
  },
});

await prisma.analyticsEvent.create({
  data: {
    userId,
    event: 'upgrade_clicked',
  },
});
```

---

### 4D: Data Analysis & Decision (Week 12)

**Goal:** Compile all metrics and make go/no-go decision

---

#### Tasks:

**1. Metrics Dashboard**

Create comprehensive report:

```typescript
export default async function MetricsPage() {
  // User Growth
  const totalUsers = await prisma.user.count();
  const newUsersThisWeek = await prisma.user.count({
    where: {
      createdAt: { gte: startOfWeek(new Date()) },
    },
  });

  // Retention
  const week1Users = await prisma.user.findMany({
    where: {
      createdAt: {
        gte: subWeeks(new Date(), 2),
        lt: subWeeks(new Date(), 1),
      },
    },
  });

  const week2RetainedCount = await prisma.suggestion.groupBy({
    by: ['userId'],
    where: {
      userId: { in: week1Users.map(u => u.id) },
      generatedAt: { gte: subWeeks(new Date(), 1) },
    },
  });

  const retentionRate = (week2RetainedCount.length / week1Users.length) * 100;

  // Acceptance Rate
  const totalSuggestions = await prisma.suggestion.count();
  const acceptedSuggestions = await prisma.suggestion.count({
    where: { status: 'accepted' },
  });
  const acceptanceRate = (acceptedSuggestions / totalSuggestions) * 100;

  // Monetization
  const premiumUsers = await prisma.user.count({
    where: { subscriptionTier: 'premium' },
  });
  const conversionRate = (premiumUsers / totalUsers) * 100;

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-8">Week 12 Metrics Report</h1>

      <div className="grid grid-cols-2 md:grid-cols-4 gap-6">
        <MetricCard title="Total Users" value={totalUsers} target={100} />
        <MetricCard title="Week 2 Retention" value={`${retentionRate.toFixed(1)}%`} target="60%" />
        <MetricCard title="Acceptance Rate" value={`${acceptanceRate.toFixed(1)}%`} target="40%" />
        <MetricCard title="Conversion Rate" value={`${conversionRate.toFixed(1)}%`} target="5%" />
      </div>

      {/* Add charts, breakdowns, insights */}
    </div>
  );
}
```

**2. User Feedback Synthesis**

Compile qualitative feedback:

```markdown
# User Feedback Summary (Week 12)

## Positive Themes
- "Saves me from decision paralysis on weekends" (mentioned by 12 users)
- "Suggestions are spot-on for my kids' ages" (mentioned by 8 users)
- "Love that it's automated, I don't have to think about it" (mentioned by 15 users)

## Pain Points
- "Wish it supported Apple Calendar" (mentioned by 6 users)
- "Sometimes suggestions are too far away" (mentioned by 4 users)
- "Would pay if it also handled dinner reservations" (mentioned by 3 users)

## Most Requested Features
1. Multi-calendar support (18 users)
2. SMS notifications (12 users)
3. Group event coordination (7 users)

## Willingness to Pay
- Would pay $5/month: 40%
- Would pay $10/month: 30%
- Would not pay: 30%
```

**3. Cost Analysis**

Calculate unit economics:

```markdown
# Cost per User (Month 3)

**Fixed Costs:**
- Vercel hosting: $20/month
- Railway (DB + Redis + Workers): $100/month
- Resend (12,000 emails/month): $40/month
- Total fixed: $160/month

**Variable Costs (per user per month):**
- Google Places API: ~$0.50
- Eventbrite API: $0 (free tier)
- Google Calendar API: $0 (free tier)
- Total variable: ~$0.50/user

**At 100 users:**
- Fixed: $160
- Variable: $50
- Total: $210/month
- Cost per user: $2.10/month

**Revenue (if 30% convert at $9/month):**
- Premium users: 30
- MRR: $270
- Net: $60 profit

**Break-even: 18 premium users (18% conversion at $9/month)**
```

**4. Go/No-Go Decision Framework**

```markdown
# Decision Matrix

## GO Criteria (need 4 of 5):
- [ ] 100+ users acquired ✓/✗
- [ ] >60% week-2 retention ✓/✗
- [ ] >40% acceptance rate ✓/✗
- [ ] >20% willing to pay $8-10/month ✓/✗
- [ ] Positive unit economics at scale ✓/✗

## If GO:
→ Proceed to Phase 5 (V2 features)
→ Focus on scaling user acquisition
→ Invest in retention improvements

## If PIVOT:
→ Analyze what's working vs not
→ Consider focus shift (couples vs families, restaurants vs activities)
→ Re-evaluate positioning and messaging

## If NO-GO:
→ Sunset product gracefully
→ Notify users with 30-day notice
→ Open-source codebase for community
→ Document learnings
```

---

## Deliverables

**Functional:**
- ✅ Stripe payment flow
- ✅ Free/Premium tier enforcement
- ✅ Pricing page
- ✅ Referral system

**Data & Analysis:**
- ✅ Full metrics dashboard
- ✅ User feedback synthesis
- ✅ Unit economics report
- ✅ Go/no-go decision document

**Marketing:**
- ✅ 100 beta users recruited
- ✅ Launch posts on Reddit/Facebook
- ✅ Influencer outreach results
- ✅ Acquisition channel performance

---

## Risks & Mitigation

**Can't Reach 100 Users:**
- Mitigation: Lower bar to 75 users if metrics are strong
- Contingency: Extend beta period by 2 weeks

**Low Conversion (<10%):**
- Mitigation: Add more premium features
- Contingency: Reconsider pricing ($5/month instead of $10)

**High Churn (>40% week-2):**
- Mitigation: Improve onboarding, increase suggestion quality
- Contingency: Pivot focus or pause development

---

## References

- [High-Level Roadmap](./2026-02-11-high-level_mvp-project-roadmap.md)
- [PRD Success Metrics](../../aiDocs/prd.md)
- [Stripe Docs](../guides/stripe_context7.md)

---

**Last Updated:** 2026-02-11
**Version:** 1.0
