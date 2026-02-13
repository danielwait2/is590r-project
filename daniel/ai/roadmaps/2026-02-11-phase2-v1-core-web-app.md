# Phase 2: V1 Core - Web App Foundation

**Roadmap:** [2026-02-11-roadmap-phase2-v1-core.md](./2026-02-11-roadmap-phase2-v1-core.md) - Track progress

**Created:** 2026-02-11
**Timeline:** Weeks 1-4 (1 month)
**Status:** Pending Phase 1 Completion
**Type:** Production MVP

---

## Overview

Build the production web application with essential features: user onboarding, Google Calendar integration, family profile management, and suggestion generation. This phase establishes the foundation for the automated weekly cycle that will be added in Phase 3.

**Context:** This begins the 12-week timeline to V1 production launch (Phases 2-4 together = 12 weeks total, as outlined in PRD).

**Key Difference from Phase 1:** Moving from client-side Chrome extension to full-stack web app with database, authentication, and server-side processing.

---

## Success Criteria

**Functional Requirements:**
- ✅ 50 beta users successfully onboarded
- ✅ Users can complete onboarding in <5 minutes
- ✅ Google Calendar sync works reliably (>95% success rate)
- ✅ Suggestion generation produces 2 relevant suggestions per user
- ✅ >40% suggestion acceptance rate (manual triggers only in Phase 2)

**Technical Requirements:**
- ✅ Web app deployed to production (Vercel + Railway)
- ✅ Database schema implemented with Prisma
- ✅ All core APIs functional and documented
- ✅ Authentication flow working (Clerk)
- ✅ Manual calendar sync trigger operational

**Validation Metrics:**
- Onboarding completion rate >70%
- Average onboarding time <5 minutes
- Calendar connection success rate >90%
- Zero critical bugs in production
- 10-20 beta users actively testing

---

## Prerequisites

**Required Before Starting:**
- ✅ Phase 1 (MVP Demo) completed
- ✅ User feedback from MVP analyzed
- ✅ Decision made to proceed with V1
- ✅ Domain name registered (e.g., familytime.com)
- ✅ Accounts created:
  - Vercel account (frontend hosting)
  - Railway account (backend + database)
  - Clerk account (authentication)
  - Google Cloud Project (for Calendar API - reuse from Phase 1)

**Technical Prerequisites:**
- Node.js 18+ installed
- PostgreSQL knowledge (basic)
- Next.js familiarity (or willingness to learn)
- Git/GitHub repository set up

---

## Detailed Implementation Steps

### 2A: Infrastructure Setup (Week 1)

**Goal:** Set up all infrastructure, deployment pipelines, and core services

---

#### Day 1: Next.js Project Initialization

**1. Create Next.js Project**

```bash
# Create new Next.js project
npx create-next-app@latest family-time-app

# During setup, choose:
# - TypeScript: Yes
# - ESLint: Yes
# - Tailwind CSS: Yes
# - src/ directory: Yes
# - App Router: Yes
# - Import alias (@/*): Yes

cd family-time-app
```

**2. Project Structure Setup**

```
family-time-app/
├── src/
│   ├── app/                    # Next.js 14 App Router
│   │   ├── layout.tsx          # Root layout
│   │   ├── page.tsx            # Landing page
│   │   ├── setup/              # Onboarding flow
│   │   │   └── page.tsx
│   │   ├── dashboard/          # Main dashboard
│   │   │   └── page.tsx
│   │   ├── profile/            # Profile management
│   │   │   └── page.tsx
│   │   ├── suggestions/        # Suggestion history
│   │   │   └── page.tsx
│   │   └── api/                # API routes
│   │       ├── auth/
│   │       ├── profile/
│   │       ├── suggestions/
│   │       └── calendar/
│   ├── components/             # Reusable components
│   │   ├── SuggestionCard.tsx
│   │   ├── FamilyProfileForm.tsx
│   │   └── CalendarPreview.tsx
│   ├── lib/                    # Utilities and services
│   │   ├── db.ts              # Prisma client
│   │   ├── calendar.ts        # Google Calendar service
│   │   ├── recommendations.ts # Recommendation engine
│   │   └── auth.ts            # Clerk helpers
│   └── types/                  # TypeScript types
│       └── index.ts
├── prisma/
│   ├── schema.prisma          # Database schema
│   └── migrations/
├── public/
│   └── images/
├── .env.local                  # Environment variables
├── .env.example
├── package.json
├── tsconfig.json
└── README.md
```

**3. Install Core Dependencies**

```bash
# Authentication
npm install @clerk/nextjs

# Database
npm install @prisma/client
npm install -D prisma

# Google Calendar API
npm install googleapis

# Email (for later phases, but install now)
npm install resend

# Utilities
npm install date-fns zod
```

**4. Environment Configuration**

Create `.env.local`:

```bash
# App
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/setup
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/setup

# Database (Railway PostgreSQL)
DATABASE_URL=postgresql://user:password@host:port/dbname

# Google Calendar API
GOOGLE_CLIENT_ID=your-client-id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-client-secret
GOOGLE_REDIRECT_URI=http://localhost:3000/api/auth/google/callback
```

Create `.env.example` (same as above but with placeholder values)

**Reference:**
- [Next.js Documentation](../guides/nextjs_context7.md)
- [Clerk Setup Guide](../guides/clerk_context7.md)

---

#### Day 2: Database Setup (PostgreSQL + Prisma)

**1. Create Railway PostgreSQL Database**

```
1. Go to Railway.app
2. Create new project: "FamilyTime"
3. Add PostgreSQL database
4. Copy connection string (DATABASE_URL)
5. Paste into .env.local
```

**2. Initialize Prisma**

```bash
npx prisma init
```

**3. Define Database Schema**

Edit `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id                    String   @id @default(uuid())
  clerkUserId           String   @unique
  email                 String   @unique
  googleCalendarId      String?
  googleRefreshToken    String?  // Encrypted
  googleAccessToken     String?  // Encrypted
  googleTokenExpiresAt  DateTime?
  onboardingCompleted   Boolean  @default(false)
  createdAt             DateTime @default(now())
  updatedAt             DateTime @updatedAt

  profile         FamilyProfile?
  suggestions     Suggestion[]
  calendarScans   CalendarScan[]

  @@index([clerkUserId])
  @@index([email])
}

model FamilyProfile {
  id                  String   @id @default(uuid())
  userId              String   @unique
  user                User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  numKids             Int
  kidAges             Int[]
  interests           String[]  // Array of interest tags
  budgetTier          String    @default("medium") // low, medium, high
  maxDriveTimeMinutes Int       @default(30)
  wantToTryList       String[]  // User's "want to try" items
  location            String?   // City, State (e.g., "Chicago, IL")
  zipCode             String?

  createdAt           DateTime @default(now())
  updatedAt           DateTime @updatedAt

  @@index([userId])
}

model Suggestion {
  id              String    @id @default(uuid())
  userId          String
  user            User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  generatedAt     DateTime  @default(now())
  targetDate      DateTime
  targetTimeSlot  String    // e.g., "10:00 AM - 2:00 PM"

  activityType    String    // museum, park, restaurant, event
  title           String
  description     String?
  address         String?
  bookingLink     String?
  estimatedCost   String?
  imageUrl        String?

  status          String    @default("sent") // sent, accepted, rejected, ignored
  feedback        String?   // thumbs_up, thumbs_down
  feedbackComment String?

  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt

  @@index([userId])
  @@index([status])
  @@index([targetDate])
}

model CalendarScan {
  id          String   @id @default(uuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  scannedAt   DateTime @default(now())
  freeBlocks  Json     // Array of { date, startTime, endTime, durationHours }

  createdAt   DateTime @default(now())

  @@index([userId])
  @@index([scannedAt])
}

model CuratedActivity {
  id              String   @id @default(uuid())

  title           String
  type            String   // museum, park, restaurant, event
  description     String
  address         String
  city            String
  state           String
  zipCode         String?
  imageUrl        String?
  bookingLink     String?
  estimatedCost   String?
  duration        String?
  ageRange        Int[]    // [minAge, maxAge]
  tags            String[] // Array of tags for matching

  isActive        Boolean  @default(true)
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  @@index([city])
  @@index([type])
}
```

**4. Run First Migration**

```bash
npx prisma migrate dev --name init
```

**5. Create Prisma Client Helper**

Create `src/lib/db.ts`:

```typescript
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

**6. Seed Curated Activities**

Create `prisma/seed.ts`:

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  console.log('Seeding curated activities...');

  const activities = [
    {
      title: "Children's Discovery Museum",
      type: "museum",
      description: "Interactive science exhibits perfect for curious kids. Hands-on learning about physics, biology, and space.",
      address: "123 Main St",
      city: "Chicago",
      state: "IL",
      zipCode: "60601",
      imageUrl: "https://via.placeholder.com/400x300",
      estimatedCost: "$35 for family",
      duration: "2-3 hours",
      ageRange: [2, 10],
      tags: ["science", "museum", "indoor", "educational", "kids"]
    },
    // Add 10-15 more from Phase 1 curated list
  ];

  for (const activity of activities) {
    await prisma.curatedActivity.create({ data: activity });
  }

  console.log(`Seeded ${activities.length} activities`);
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Run seed:

```bash
npx prisma db seed
```

**Reference:**
- [Prisma Documentation](../guides/prisma_context7.md)

---

#### Day 3: Clerk Authentication Integration

**1. Install and Configure Clerk**

Already installed in Day 1. Now configure:

```bash
# Get API keys from clerk.com dashboard
# Add to .env.local (already done above)
```

**2. Add Clerk Middleware**

Create `src/middleware.ts`:

```typescript
import { authMiddleware } from "@clerk/nextjs";

export default authMiddleware({
  publicRoutes: ["/", "/sign-in", "/sign-up"],
});

export const config = {
  matcher: ["/((?!.+\\.[\\w]+$|_next).*)", "/", "/(api|trpc)(.*)"],
};
```

**3. Update Root Layout**

Edit `src/app/layout.tsx`:

```typescript
import { ClerkProvider } from '@clerk/nextjs';
import './globals.css';
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'FamilyTime - Activity Suggestions',
  description: 'Get personalized family activity suggestions based on your free time',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body className={inter.className}>{children}</body>
      </html>
    </ClerkProvider>
  );
}
```

**4. Create Sign In/Up Pages**

```bash
mkdir -p src/app/sign-in src/app/sign-up
```

`src/app/sign-in/page.tsx`:

```typescript
import { SignIn } from '@clerk/nextjs';

export default function SignInPage() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <SignIn />
    </div>
  );
}
```

`src/app/sign-up/page.tsx`:

```typescript
import { SignUp } from '@clerk/nextjs';

export default function SignUpPage() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <SignUp />
    </div>
  );
}
```

**5. Create User Sync Webhook**

Create `src/app/api/webhooks/clerk/route.ts`:

```typescript
import { Webhook } from 'svix';
import { headers } from 'next/headers';
import { WebhookEvent } from '@clerk/nextjs/server';
import { prisma } from '@/lib/db';

export async function POST(req: Request) {
  const WEBHOOK_SECRET = process.env.CLERK_WEBHOOK_SECRET;

  if (!WEBHOOK_SECRET) {
    throw new Error('Missing CLERK_WEBHOOK_SECRET');
  }

  const headerPayload = headers();
  const svix_id = headerPayload.get('svix-id');
  const svix_timestamp = headerPayload.get('svix-timestamp');
  const svix_signature = headerPayload.get('svix-signature');

  if (!svix_id || !svix_timestamp || !svix_signature) {
    return new Response('Missing svix headers', { status: 400 });
  }

  const payload = await req.json();
  const body = JSON.stringify(payload);

  const wh = new Webhook(WEBHOOK_SECRET);
  let evt: WebhookEvent;

  try {
    evt = wh.verify(body, {
      'svix-id': svix_id,
      'svix-timestamp': svix_timestamp,
      'svix-signature': svix_signature,
    }) as WebhookEvent;
  } catch (err) {
    console.error('Webhook verification failed:', err);
    return new Response('Invalid signature', { status: 400 });
  }

  const eventType = evt.type;

  if (eventType === 'user.created') {
    const { id, email_addresses } = evt.data;

    await prisma.user.create({
      data: {
        clerkUserId: id,
        email: email_addresses[0].email_address,
      },
    });
  }

  return new Response('Webhook processed', { status: 200 });
}
```

**Reference:**
- [Clerk Documentation](../guides/clerk_context7.md)

---

#### Day 4: Deployment Pipeline Setup

**1. Deploy Frontend to Vercel**

```bash
# Install Vercel CLI
npm install -g vercel

# Login to Vercel
vercel login

# Link project
vercel link

# Deploy
vercel --prod
```

Configure environment variables in Vercel dashboard.

**2. Deploy Backend API to Railway**

Since Next.js API routes are deployed with the frontend, no separate backend deployment needed for Phase 2. (BullMQ jobs will be added in Phase 3.)

**3. Set up GitHub Actions for CI/CD**

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run Prisma migrations
        run: npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

      - name: Build
        run: npm run build

      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

**4. Configure Custom Domain**

```
1. In Vercel dashboard, go to project settings
2. Add domain: familytime.com
3. Add DNS records as instructed by Vercel
4. Wait for SSL certificate provisioning
```

**5. Test Deployment**

```
- Visit production URL
- Verify Clerk authentication works
- Check database connection
- Confirm environment variables loaded
```

---

#### Day 5: Testing & Documentation

**1. Write API Tests**

Create `src/__tests__/api/auth.test.ts`:

```typescript
// Basic smoke tests
describe('API Routes', () => {
  it('should return 401 for unauthenticated requests', async () => {
    // Test implementation
  });
});
```

**2. Create API Documentation**

Create `docs/api.md`:

```markdown
# API Documentation

## Authentication
All API routes require Clerk authentication except public routes.

## Endpoints

### GET /api/user/profile
Returns current user and family profile.

**Response:**
\`\`\`json
{
  "user": { ... },
  "profile": { ... }
}
\`\`\`

[Add more endpoints...]
```

**3. Update README**

```markdown
# FamilyTime Web App

## Local Development

1. Clone repository
2. Install dependencies: `npm install`
3. Copy `.env.example` to `.env.local` and fill in values
4. Run migrations: `npx prisma migrate dev`
5. Seed database: `npx prisma db seed`
6. Start dev server: `npm run dev`

## Deployment

- Frontend: Vercel (auto-deploy from `main` branch)
- Database: Railway PostgreSQL
- Environment variables configured in Vercel dashboard
```

**Week 1 Complete!** Infrastructure is ready for feature development.

---

### 2B: User Onboarding (Week 2)

**Goal:** Build landing page and complete onboarding flow

---

#### Tasks:

**1. Landing Page Design**

Edit `src/app/page.tsx`:

```typescript
import Link from 'next/link';
import { SignedIn, SignedOut } from '@clerk/nextjs';

export default function HomePage() {
  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-green-50">
      {/* Hero Section */}
      <header className="container mx-auto px-4 py-20">
        <div className="max-w-3xl mx-auto text-center">
          <h1 className="text-5xl font-bold text-gray-900 mb-6">
            Turn Free Time Into Quality Time
          </h1>
          <p className="text-xl text-gray-600 mb-8">
            Get personalized family activity suggestions delivered to your inbox every week,
            based on your actual calendar availability.
          </p>

          <SignedOut>
            <Link
              href="/sign-up"
              className="inline-block bg-blue-600 text-white px-8 py-4 rounded-lg text-lg font-semibold hover:bg-blue-700 transition"
            >
              Get Started Free
            </Link>
          </SignedOut>

          <SignedIn>
            <Link
              href="/dashboard"
              className="inline-block bg-blue-600 text-white px-8 py-4 rounded-lg text-lg font-semibold hover:bg-blue-700 transition"
            >
              Go to Dashboard
            </Link>
          </SignedIn>
        </div>
      </header>

      {/* How It Works */}
      <section className="container mx-auto px-4 py-16">
        <h2 className="text-3xl font-bold text-center mb-12">How It Works</h2>
        <div className="grid md:grid-cols-3 gap-8 max-w-5xl mx-auto">
          <div className="text-center">
            <div className="text-4xl mb-4">📅</div>
            <h3 className="text-xl font-semibold mb-2">Connect Calendar</h3>
            <p className="text-gray-600">
              We scan your Google Calendar to find free time blocks on weekends.
            </p>
          </div>
          <div className="text-center">
            <div className="text-4xl mb-4">🎯</div>
            <h3 className="text-xl font-semibold mb-2">Set Preferences</h3>
            <p className="text-gray-600">
              Tell us about your family: ages, interests, budget, location.
            </p>
          </div>
          <div className="text-center">
            <div className="text-4xl mb-4">✨</div>
            <h3 className="text-xl font-semibold mb-2">Get Suggestions</h3>
            <p className="text-gray-600">
              Receive 2 personalized activity ideas every Friday morning.
            </p>
          </div>
        </div>
      </section>

      {/* CTA */}
      <section className="container mx-auto px-4 py-16 text-center">
        <h2 className="text-3xl font-bold mb-4">Ready to make family time happen?</h2>
        <SignedOut>
          <Link
            href="/sign-up"
            className="inline-block bg-green-600 text-white px-8 py-4 rounded-lg text-lg font-semibold hover:bg-green-700 transition"
          >
            Start Your First Week Free
          </Link>
        </SignedOut>
      </section>
    </div>
  );
}
```

**2. Family Profile Form Component**

Create `src/components/FamilyProfileForm.tsx`:

```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';

const INTERESTS = [
  'Outdoors', 'Museums', 'Arts & Crafts', 'Sports', 'Food & Dining',
  'Science', 'Animals & Nature', 'Music', 'Theater', 'Parks'
];

export default function FamilyProfileForm() {
  const router = useRouter();
  const [formData, setFormData] = useState({
    numKids: 1,
    kidAges: [''],
    interests: [] as string[],
    budgetTier: 'medium',
    maxDriveTimeMinutes: 30,
    wantToTryList: '',
    location: '',
    zipCode: ''
  });

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    const payload = {
      ...formData,
      kidAges: formData.kidAges.map(Number),
      wantToTryList: formData.wantToTryList.split('\n').filter(Boolean)
    };

    const res = await fetch('/api/profile', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    if (res.ok) {
      router.push('/dashboard');
    }
  };

  const toggleInterest = (interest: string) => {
    setFormData(prev => ({
      ...prev,
      interests: prev.interests.includes(interest)
        ? prev.interests.filter(i => i !== interest)
        : [...prev.interests, interest]
    }));
  };

  return (
    <form onSubmit={handleSubmit} className="max-w-2xl mx-auto space-y-8">
      {/* Number of Kids */}
      <div>
        <label className="block text-sm font-medium mb-2">Number of Kids</label>
        <input
          type="number"
          min="1"
          max="10"
          value={formData.numKids}
          onChange={(e) => setFormData({ ...formData, numKids: Number(e.target.value) })}
          className="w-full px-4 py-2 border rounded-lg"
        />
      </div>

      {/* Kid Ages */}
      <div>
        <label className="block text-sm font-medium mb-2">Kid Ages</label>
        {Array.from({ length: formData.numKids }).map((_, i) => (
          <input
            key={i}
            type="number"
            placeholder={`Kid ${i + 1} age`}
            value={formData.kidAges[i] || ''}
            onChange={(e) => {
              const newAges = [...formData.kidAges];
              newAges[i] = e.target.value;
              setFormData({ ...formData, kidAges: newAges });
            }}
            className="w-full px-4 py-2 border rounded-lg mb-2"
          />
        ))}
      </div>

      {/* Interests */}
      <div>
        <label className="block text-sm font-medium mb-2">Interests (select all that apply)</label>
        <div className="grid grid-cols-2 gap-2">
          {INTERESTS.map(interest => (
            <button
              key={interest}
              type="button"
              onClick={() => toggleInterest(interest)}
              className={`px-4 py-2 rounded-lg border ${
                formData.interests.includes(interest)
                  ? 'bg-blue-600 text-white'
                  : 'bg-white text-gray-700'
              }`}
            >
              {interest}
            </button>
          ))}
        </div>
      </div>

      {/* Budget Tier */}
      <div>
        <label className="block text-sm font-medium mb-2">Budget Tier</label>
        <select
          value={formData.budgetTier}
          onChange={(e) => setFormData({ ...formData, budgetTier: e.target.value })}
          className="w-full px-4 py-2 border rounded-lg"
        >
          <option value="low">Low (Free - $20)</option>
          <option value="medium">Medium ($20 - $60)</option>
          <option value="high">High ($60+)</option>
        </select>
      </div>

      {/* Max Drive Time */}
      <div>
        <label className="block text-sm font-medium mb-2">
          Max Drive Time: {formData.maxDriveTimeMinutes} minutes
        </label>
        <input
          type="range"
          min="10"
          max="120"
          step="10"
          value={formData.maxDriveTimeMinutes}
          onChange={(e) => setFormData({ ...formData, maxDriveTimeMinutes: Number(e.target.value) })}
          className="w-full"
        />
      </div>

      {/* Location */}
      <div>
        <label className="block text-sm font-medium mb-2">City, State</label>
        <input
          type="text"
          placeholder="Chicago, IL"
          value={formData.location}
          onChange={(e) => setFormData({ ...formData, location: e.target.value })}
          className="w-full px-4 py-2 border rounded-lg"
        />
      </div>

      {/* Want to Try List */}
      <div>
        <label className="block text-sm font-medium mb-2">
          What do you want to try? (one per line)
        </label>
        <textarea
          rows={5}
          placeholder="e.g., science museum&#10;hiking trails&#10;pottery class"
          value={formData.wantToTryList}
          onChange={(e) => setFormData({ ...formData, wantToTryList: e.target.value })}
          className="w-full px-4 py-2 border rounded-lg"
        />
      </div>

      <button
        type="submit"
        className="w-full bg-blue-600 text-white py-3 rounded-lg font-semibold hover:bg-blue-700"
      >
        Complete Setup
      </button>
    </form>
  );
}
```

**3. Onboarding Flow Page**

Create `src/app/setup/page.tsx`:

```typescript
import { currentUser } from '@clerk/nextjs';
import { redirect } from 'next/navigation';
import { prisma } from '@/lib/db';
import FamilyProfileForm from '@/components/FamilyProfileForm';

export default async function SetupPage() {
  const user = await currentUser();
  if (!user) redirect('/sign-in');

  const dbUser = await prisma.user.findUnique({
    where: { clerkUserId: user.id },
    include: { profile: true }
  });

  if (dbUser?.profile) {
    redirect('/dashboard'); // Already completed onboarding
  }

  return (
    <div className="min-h-screen bg-gray-50 py-12 px-4">
      <div className="max-w-3xl mx-auto">
        <h1 className="text-3xl font-bold text-center mb-8">
          Tell Us About Your Family
        </h1>
        <div className="bg-white rounded-lg shadow p-8">
          <FamilyProfileForm />
        </div>
      </div>
    </div>
  );
}
```

**4. Profile API Endpoint**

Create `src/app/api/profile/route.ts`:

```typescript
import { auth } from '@clerk/nextjs';
import { NextResponse } from 'next/server';
import { prisma } from '@/lib/db';

export async function GET() {
  const { userId } = auth();
  if (!userId) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });

  const user = await prisma.user.findUnique({
    where: { clerkUserId: userId },
    include: { profile: true }
  });

  return NextResponse.json({ user, profile: user?.profile });
}

export async function POST(req: Request) {
  const { userId } = auth();
  if (!userId) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });

  const body = await req.json();

  const user = await prisma.user.findUnique({
    where: { clerkUserId: userId }
  });

  if (!user) {
    return NextResponse.json({ error: 'User not found' }, { status: 404 });
  }

  const profile = await prisma.familyProfile.create({
    data: {
      userId: user.id,
      numKids: body.numKids,
      kidAges: body.kidAges,
      interests: body.interests,
      budgetTier: body.budgetTier,
      maxDriveTimeMinutes: body.maxDriveTimeMinutes,
      wantToTryList: body.wantToTryList,
      location: body.location,
      zipCode: body.zipCode
    }
  });

  await prisma.user.update({
    where: { id: user.id },
    data: { onboardingCompleted: true }
  });

  return NextResponse.json({ profile });
}
```

**Testing:**
- Sign up new account
- Complete onboarding form (<5 min)
- Verify profile saved to database
- Test validation (empty fields, invalid ages)

---

### 2C: Calendar Sync & Analysis (Week 3)

**Goal:** Integrate Google Calendar API and detect free time

[Implementation details continue...]

---

## Technical Stack & Tools

**Frontend:**
- Next.js 14 (App Router, TypeScript)
- Tailwind CSS
- React components

**Backend:**
- Next.js API Routes
- Node.js runtime

**Database:**
- PostgreSQL (Railway)
- Prisma ORM

**Authentication:**
- Clerk (OAuth, user management)

**APIs:**
- Google Calendar API v3
- Google Places API (Phase 2D)
- Eventbrite API (Phase 2D)

**Deployment:**
- Vercel (frontend)
- Railway (database)

**Development Tools:**
- TypeScript
- ESLint
- Prettier
- Git/GitHub

---

## References

- [High-Level Roadmap](./2026-02-11-high-level_mvp-project-roadmap.md)
- [Architecture Document](../../aiDocs/architecture.md)
- [Next.js Docs](../guides/nextjs_context7.md)
- [Prisma Docs](../guides/prisma_context7.md)
- [Clerk Docs](../guides/clerk_context7.md)
- [Google Calendar API](../guides/google-calendar-api_context7.md)

---

**Last Updated:** 2026-02-11
**Version:** 1.0 (Partial - Week 1-2 detailed, Weeks 3-4 to be completed)
