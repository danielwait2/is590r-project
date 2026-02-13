# Phase 1: Foundation - Detailed Implementation Plan

**Timeline:** Week 1-2 (14 days)  
**Status:** Not Started  
**Roadmap:** [2026-02-12_roadmap-phase1.md](./2026-02-12_roadmap-phase1.md) - Track progress

---

## Overview

Set up the core infrastructure for the Family Quality Time app. This phase establishes the foundation: Next.js project, Supabase database, authentication, and basic UI with onboarding flow. By the end, users can sign up with Google OAuth and complete a single-page onboarding.

**What gets built:**
- Next.js 14 project with TypeScript and Tailwind
- Supabase project with PostgreSQL database
- Google OAuth authentication via Supabase Auth
- Landing page, signup flow, empty dashboard
- Single-page onboarding (4 inputs: location, kids, Want to Try list, one sitter)
- Deployed to Vercel with CI/CD

---

## Success Criteria

- [ ] User can visit app at https://[project-name].vercel.app
- [ ] User can click "Sign up with Google" and authenticate
- [ ] User completes onboarding (enters location, kids, Want to Try list, adds 1 sitter)
- [ ] User sees empty dashboard with "No suggestions yet"
- [ ] Database contains user record with onboarding data
- [ ] All environment variables configured (local + Vercel)
- [ ] App loads in <2 seconds

---

## Prerequisites

**Before starting:**
- [ ] Have a Google account for testing
- [ ] Have a GitHub account (for Vercel deployment)
- [ ] Install Node.js 18+ and npm
- [ ] Install VS Code or preferred IDE
- [ ] Read `aiDocs/mvp.md` and `aiDocs/prd.md`

---

## Detailed Implementation Steps

### Day 1-2: Project Setup

#### 1. Create Next.js Project

```bash
npx create-next-app@14 family-time-app
cd family-time-app
```

**Selections:**
- ✅ TypeScript
- ✅ ESLint
- ✅ Tailwind CSS
- ✅ App Router
- ❌ src/ directory (keep root-level app/)
- ✅ Import alias (@/*)

**File structure created:**
```
family-time-app/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── public/
├── package.json
├── tsconfig.json
├── tailwind.config.ts
└── next.config.js
```

#### 2. Install Core Dependencies

```bash
npm install @supabase/supabase-js @supabase/auth-helpers-nextjs
npm install date-fns
```

**Why these:**
- `@supabase/supabase-js` - Supabase JavaScript client
- `@supabase/auth-helpers-nextjs` - Next.js auth integration
- `date-fns` - Date manipulation (for free time detection later)

**Reference:** See `ai/guides/supabase-api.md` for full Supabase setup

#### 3. Configure Environment Variables

Create `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=https://[project].supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Add to `.gitignore`:**
```
.env.local
.env*.local
.testEnvVars
```

---

### Day 2-3: Supabase Setup

#### 1. Create Supabase Project

1. Go to https://supabase.com/dashboard
2. Click "New Project"
3. Enter:
   - Name: `family-time-mvp`
   - Database Password: (save in password manager)
   - Region: Choose closest to Austin, TX
4. Wait for project to be created (~2 minutes)
5. Copy:
   - Project URL → `NEXT_PUBLIC_SUPABASE_URL`
   - anon/public key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - service_role key → `SUPABASE_SERVICE_ROLE_KEY`

#### 2. Create Database Schema

Go to Supabase Dashboard → SQL Editor → New Query

```sql
-- Users table (extends Supabase auth.users)
create table public.users (
  id uuid primary key references auth.users(id) on delete cascade,
  email text not null,
  name text,
  location_city text,
  location_state text,
  timezone text default 'America/Chicago',
  google_calendar_token text,  -- Will be encrypted
  google_calendar_refresh_token text,  -- Will be encrypted
  onboarding_completed boolean default false,
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);

-- Trigger to auto-update updated_at
create or replace function public.handle_updated_at()
returns trigger as $$
begin
  new.updated_at = now();
  return new;
end;
$$ language plpgsql;

create trigger users_updated_at
  before update on public.users
  for each row
  execute function handle_updated_at();

-- Kids (for onboarding)
create table public.kids (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.users(id) on delete cascade not null,
  age int not null,
  created_at timestamp with time zone default now()
);

-- Want to Try list (interests/activities user wants to do)
create table public.interests (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.users(id) on delete cascade not null,
  name text not null,  -- "kayaking", "try new restaurant", etc.
  added_at timestamp with time zone default now()
);

-- Trusted Sitters
create table public.trusted_sitters (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.users(id) on delete cascade not null,
  name text not null,
  phone text not null,  -- E.164 format: +15551234567
  relationship text,  -- "family", "friend", "regular sitter"
  is_active boolean default true,
  created_at timestamp with time zone default now()
);

-- Activities (populated in Phase 3)
create table public.activities (
  id uuid primary key default uuid_generate_v4(),
  name text not null,
  category text not null,
  address text,
  city text not null default 'Austin',
  state text not null default 'TX',
  lat numeric,
  lng numeric,
  rating numeric,
  price_level int,  -- 1-4
  requires_babysitter boolean default false,
  scraped_from text,  -- 'google_maps', 'yelp', 'manual'
  created_at timestamp with time zone default now()
);

-- Suggestions (created in Phase 4)
create table public.suggestions (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.users(id) on delete cascade not null,
  activity_id uuid references public.activities(id),
  suggested_date date not null,
  suggested_start_time time not null,
  suggested_end_time time not null,
  google_calendar_event_id text,
  status text default 'pending',  -- 'pending', 'accepted', 'declined', 'completed'
  match_reason text,
  created_at timestamp with time zone default now(),
  responded_at timestamp with time zone
);

-- Babysitter Requests (created in Phase 5)
create table public.babysitter_requests (
  id uuid primary key default uuid_generate_v4(),
  suggestion_id uuid references public.suggestions(id) on delete cascade not null,
  sitter_id uuid references public.trusted_sitters(id) not null,
  phone text not null,
  status text default 'sent',  -- 'sent', 'delivered', 'accepted', 'declined'
  sent_at timestamp with time zone default now(),
  responded_at timestamp with time zone,
  twilio_message_sid text
);

-- SMS Messages log (for debugging)
create table public.sms_messages (
  id uuid primary key default uuid_generate_v4(),
  message_sid text unique not null,
  direction text not null,  -- 'inbound', 'outbound'
  from_phone text not null,
  to_phone text not null,
  body text not null,
  status text,
  babysitter_request_id uuid references public.babysitter_requests(id),
  created_at timestamp with time zone default now()
);
```

#### 3. Enable Row Level Security (RLS)

```sql
-- Enable RLS on all tables
alter table public.users enable row level security;
alter table public.kids enable row level security;
alter table public.interests enable row level security;
alter table public.trusted_sitters enable row level security;
alter table public.activities enable row level security;
alter table public.suggestions enable row level security;
alter table public.babysitter_requests enable row level security;
alter table public.sms_messages enable row level security;

-- Users can only see their own data
create policy "Users can view own data"
  on public.users for select
  using (auth.uid() = id);

create policy "Users can update own data"
  on public.users for update
  using (auth.uid() = id);

create policy "Users can insert own data"
  on public.users for insert
  with check (auth.uid() = id);

-- Kids policies
create policy "Users can view own kids"
  on public.kids for select
  using (user_id = auth.uid());

create policy "Users can insert own kids"
  on public.kids for insert
  with check (user_id = auth.uid());

create policy "Users can update own kids"
  on public.kids for update
  using (user_id = auth.uid());

create policy "Users can delete own kids"
  on public.kids for delete
  using (user_id = auth.uid());

-- Similar policies for interests and trusted_sitters
create policy "Users can manage own interests"
  on public.interests for all
  using (user_id = auth.uid());

create policy "Users can manage own sitters"
  on public.trusted_sitters for all
  using (user_id = auth.uid());

-- Activities are public (read-only for users)
create policy "Anyone can view activities"
  on public.activities for select
  to authenticated
  using (true);

-- Suggestions belong to user
create policy "Users can view own suggestions"
  on public.suggestions for select
  using (user_id = auth.uid());

-- Babysitter requests visible to user
create policy "Users can view own babysitter requests"
  on public.babysitter_requests for select
  using (exists (
    select 1 from public.suggestions
    where suggestions.id = babysitter_requests.suggestion_id
    and suggestions.user_id = auth.uid()
  ));
```

**Reference:** See `ai/guides/supabase-api.md` for RLS best practices

---

### Day 3-5: Google OAuth Setup

#### 1. Configure Google Cloud Project

1. Go to https://console.cloud.google.com
2. Create new project: "Family Time MVP"
3. Enable Google Calendar API:
   - APIs & Services → Enable APIs and Services
   - Search "Google Calendar API"
   - Click "Enable"

#### 2. Configure OAuth Consent Screen

1. Go to OAuth consent screen
2. Select "External"
3. Fill in:
   - App name: `Family Time App`
   - User support email: your email
   - Developer contact: your email
4. Add scopes:
   - `../auth/userinfo.email`
   - `../auth/userinfo.profile`
   - `../auth/calendar.events` (will add in Phase 2)
5. Add test users: your Gmail address
6. Save

#### 3. Create OAuth Credentials

1. Go to Credentials
2. Create Credentials → OAuth 2.0 Client ID
3. Application type: Web application
4. Name: `Family Time Web App`
5. Authorized redirect URIs:
   - `http://localhost:3000/auth/callback` (local dev)
   - `https://[your-vercel-domain].vercel.app/auth/callback` (production)
6. Copy:
   - Client ID → `GOOGLE_CLIENT_ID`
   - Client Secret → `GOOGLE_CLIENT_SECRET`

**Add to `.env.local`:**
```
GOOGLE_CLIENT_ID=123456789-abc...apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-abc123...
```

#### 4. Configure Supabase Auth

1. Go to Supabase Dashboard → Authentication → Providers
2. Enable Google:
   - Toggle Google to ON
   - Enter Client ID from Google Cloud Console
   - Enter Client Secret
   - Save

**Reference:** See `ai/guides/google-calendar-api.md` for full OAuth setup

---

### Day 5-7: Authentication Implementation

#### 1. Create Supabase Client

`lib/supabase/client.ts`:
```typescript
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

`lib/supabase/server.ts`:
```typescript
import { createServerClient, type CookieOptions } from '@supabase/ssr'
import { cookies } from 'next/headers'

export function createClient() {
  const cookieStore = cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        set(name: string, value: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value, ...options })
          } catch (error) {
            // Server component, can't set cookies
          }
        },
        remove(name: string, options: CookieOptions) {
          try {
            cookieStore.set({ name, value: '', ...options })
          } catch (error) {
            // Server component, can't remove cookies
          }
        },
      },
    }
  )
}
```

#### 2. Create Auth Callback Route

`app/auth/callback/route.ts`:
```typescript
import { createClient } from '@/lib/supabase/server'
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const requestUrl = new URL(request.url)
  const code = requestUrl.searchParams.get('code')

  if (code) {
    const supabase = createClient()
    await supabase.auth.exchangeCodeForSession(code)
  }

  // URL to redirect to after sign in
  return NextResponse.redirect(requestUrl.origin + '/dashboard')
}
```

#### 3. Create Landing Page with Sign In

`app/page.tsx`:
```typescript
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export default async function Home() {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (user) {
    redirect('/dashboard')
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-50 to-indigo-100">
      <div className="max-w-md w-full space-y-8 p-8 bg-white rounded-xl shadow-lg">
        <div className="text-center">
          <h1 className="text-4xl font-bold text-gray-900 mb-2">
            Family Time
          </h1>
          <p className="text-gray-600">
            Turn "we should do something" into plans
          </p>
        </div>

        <div className="space-y-4">
          <form action="/auth/sign-in" method="post">
            <button
              type="submit"
              className="w-full flex items-center justify-center gap-3 px-4 py-3 border border-gray-300 rounded-lg hover:bg-gray-50 transition"
            >
              <svg className="w-5 h-5" viewBox="0 0 24 24">
                {/* Google logo SVG */}
              </svg>
              Continue with Google
            </button>
          </form>
        </div>

        <p className="text-xs text-center text-gray-500">
          By signing in, you agree to our Terms and Privacy Policy
        </p>
      </div>
    </div>
  )
}
```

#### 4. Create Sign In Action

`app/auth/sign-in/route.ts`:
```typescript
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export async function POST(request: Request) {
  const supabase = createClient()
  const origin = request.headers.get('origin')

  const { error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: `${origin}/auth/callback`,
      queryParams: {
        access_type: 'offline',
        prompt: 'consent',
      },
    },
  })

  if (error) {
    return redirect('/error')
  }

  return redirect('/dashboard')
}
```

---

### Day 7-10: Onboarding Flow

#### 1. Create Dashboard Layout

`app/dashboard/layout.tsx`:
```typescript
import { createClient } from '@/lib/supabase/server'
import { redirect } from 'next/navigation'

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()

  if (!user) {
    redirect('/')
  }

  // Check if onboarding is complete
  const { data: userData } = await supabase
    .from('users')
    .select('onboarding_completed')
    .eq('id', user.id)
    .single()

  if (!userData?.onboarding_completed) {
    redirect('/onboarding')
  }

  return (
    <div className="min-h-screen bg-gray-50">
      {/* Header */}
      <header className="bg-white border-b">
        <div className="max-w-7xl mx-auto px-4 py-4 flex justify-between items-center">
          <h1 className="text-xl font-bold">Family Time</h1>
          <form action="/auth/sign-out" method="post">
            <button className="text-sm text-gray-600 hover:text-gray-900">
              Sign Out
            </button>
          </form>
        </div>
      </header>

      {/* Main content */}
      <main className="max-w-7xl mx-auto px-4 py-8">
        {children}
      </main>
    </div>
  )
}
```

#### 2. Create Onboarding Page (Single Page, 4 Inputs)

`app/onboarding/page.tsx`:
```typescript
'use client'

import { useState } from 'react'
import { createClient } from '@/lib/supabase/client'
import { useRouter } from 'next/navigation'

export default function OnboardingPage() {
  const router = useRouter()
  const supabase = createClient()
  const [loading, setLoading] = useState(false)
  
  const [formData, setFormData] = useState({
    location: '',
    state: 'TX',
    kids: [] as { age: number }[],
    wantToTry: [] as string[],
    sitterName: '',
    sitterPhone: '',
  })

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setLoading(true)

    try {
      const { data: { user } } = await supabase.auth.getUser()
      if (!user) throw new Error('Not authenticated')

      // Insert user data
      const { error: userError } = await supabase
        .from('users')
        .upsert({
          id: user.id,
          email: user.email,
          name: user.user_metadata.full_name,
          location_city: formData.location,
          location_state: formData.state,
          onboarding_completed: true,
        })

      if (userError) throw userError

      // Insert kids
      if (formData.kids.length > 0) {
        const { error: kidsError } = await supabase
          .from('kids')
          .insert(
            formData.kids.map(kid => ({
              user_id: user.id,
              age: kid.age,
            }))
          )
        if (kidsError) throw kidsError
      }

      // Insert Want to Try list
      if (formData.wantToTry.length > 0) {
        const { error: interestsError } = await supabase
          .from('interests')
          .insert(
            formData.wantToTry.map(item => ({
              user_id: user.id,
              name: item,
            }))
          )
        if (interestsError) throw interestsError
      }

      // Insert trusted sitter
      if (formData.sitterName && formData.sitterPhone) {
        const { error: sitterError } = await supabase
          .from('trusted_sitters')
          .insert({
            user_id: user.id,
            name: formData.sitterName,
            phone: formData.sitterPhone,
            relationship: 'friend',
          })
        if (sitterError) throw sitterError
      }

      router.push('/dashboard')
    } catch (error) {
      console.error('Onboarding error:', error)
      alert('Error completing onboarding. Please try again.')
    } finally {
      setLoading(false)
    }
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50 px-4">
      <div className="max-w-2xl w-full bg-white rounded-xl shadow-lg p-8">
        <h1 className="text-3xl font-bold mb-2">Let's get started</h1>
        <p className="text-gray-600 mb-8">
          Tell us a bit about your family so we can suggest great activities
        </p>

        <form onSubmit={handleSubmit} className="space-y-6">
          {/* 1. Location */}
          <div>
            <label className="block text-sm font-medium mb-2">
              Where are you located?
            </label>
            <div className="flex gap-4">
              <input
                type="text"
                placeholder="City"
                value={formData.location}
                onChange={(e) => setFormData({ ...formData, location: e.target.value })}
                required
                className="flex-1 px-4 py-2 border rounded-lg"
              />
              <select
                value={formData.state}
                onChange={(e) => setFormData({ ...formData, state: e.target.value })}
                className="px-4 py-2 border rounded-lg"
              >
                <option value="TX">TX</option>
                {/* Add more states as needed */}
              </select>
            </div>
          </div>

          {/* 2. Kids */}
          <div>
            <label className="block text-sm font-medium mb-2">
              How many kids? (Optional)
            </label>
            <div className="space-y-2">
              {formData.kids.map((kid, index) => (
                <div key={index} className="flex gap-2">
                  <input
                    type="number"
                    placeholder="Age"
                    value={kid.age}
                    onChange={(e) => {
                      const newKids = [...formData.kids]
                      newKids[index].age = parseInt(e.target.value)
                      setFormData({ ...formData, kids: newKids })
                    }}
                    min="0"
                    max="18"
                    className="w-24 px-4 py-2 border rounded-lg"
                  />
                  <button
                    type="button"
                    onClick={() => {
                      const newKids = formData.kids.filter((_, i) => i !== index)
                      setFormData({ ...formData, kids: newKids })
                    }}
                    className="text-red-600 hover:text-red-800"
                  >
                    Remove
                  </button>
                </div>
              ))}
              <button
                type="button"
                onClick={() => setFormData({ ...formData, kids: [...formData.kids, { age: 0 }] })}
                className="text-blue-600 hover:text-blue-800 text-sm"
              >
                + Add child
              </button>
            </div>
          </div>

          {/* 3. Want to Try list */}
          <div>
            <label className="block text-sm font-medium mb-2">
              What do you want to try? (One per line)
            </label>
            <textarea
              placeholder="kayaking&#10;try new restaurant&#10;visit museum"
              value={formData.wantToTry.join('\n')}
              onChange={(e) => setFormData({ ...formData, wantToTry: e.target.value.split('\n').filter(Boolean) })}
              rows={4}
              className="w-full px-4 py-2 border rounded-lg"
            />
          </div>

          {/* 4. One Trusted Sitter */}
          <div>
            <label className="block text-sm font-medium mb-2">
              Add a trusted babysitter (Optional)
            </label>
            <div className="space-y-3">
              <input
                type="text"
                placeholder="Name"
                value={formData.sitterName}
                onChange={(e) => setFormData({ ...formData, sitterName: e.target.value })}
                className="w-full px-4 py-2 border rounded-lg"
              />
              <input
                type="tel"
                placeholder="Phone (e.g., +1 555-123-4567)"
                value={formData.sitterPhone}
                onChange={(e) => setFormData({ ...formData, sitterPhone: e.target.value })}
                className="w-full px-4 py-2 border rounded-lg"
              />
            </div>
          </div>

          {/* Submit */}
          <button
            type="submit"
            disabled={loading || !formData.location}
            className="w-full px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:bg-gray-300 disabled:cursor-not-allowed"
          >
            {loading ? 'Setting up...' : 'Complete Setup'}
          </button>
        </form>
      </div>
    </div>
  )
}
```

#### 3. Create Empty Dashboard

`app/dashboard/page.tsx`:
```typescript
import { createClient } from '@/lib/supabase/server'

export default async function DashboardPage() {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()

  const { data: userData } = await supabase
    .from('users')
    .select('*, interests(*), trusted_sitters(*)')
    .eq('id', user!.id)
    .single()

  return (
    <div className="space-y-8">
      <div>
        <h2 className="text-2xl font-bold mb-4">
          Welcome, {userData?.name || 'there'}!
        </h2>
        <p className="text-gray-600">
          We'll start suggesting activities once you connect your calendar.
        </p>
      </div>

      {/* Empty state */}
      <div className="bg-white rounded-lg border-2 border-dashed border-gray-300 p-12 text-center">
        <h3 className="text-lg font-medium text-gray-900 mb-2">
          No suggestions yet
        </h3>
        <p className="text-gray-600 mb-6">
          Connect your Google Calendar to get started
        </p>
        <button className="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700">
          Connect Calendar (Coming in Week 3)
        </button>
      </div>

      {/* Show their interests */}
      {userData?.interests && userData.interests.length > 0 && (
        <div className="bg-white rounded-lg p-6">
          <h3 className="font-medium mb-4">Your Want to Try List</h3>
          <ul className="space-y-2">
            {userData.interests.map((interest: any) => (
              <li key={interest.id} className="flex items-center gap-2">
                <span className="text-gray-600">→</span>
                {interest.name}
              </li>
            ))}
          </ul>
        </div>
      )}
    </div>
  )
}
```

---

### Day 10-12: Deployment & Testing

#### 1. Deploy to Vercel

1. Commit code to GitHub:
```bash
git init
git add .
git commit -m "Initial commit: Phase 1 complete"
git branch -M main
git remote add origin https://github.com/[username]/family-time-app.git
git push -u origin main
```

2. Go to https://vercel.com/new
3. Import GitHub repository
4. Configure:
   - Framework: Next.js
   - Root Directory: ./
   - Environment Variables: (add all from `.env.local`)
5. Deploy

#### 2. Configure Production Environment Variables

In Vercel Dashboard → Settings → Environment Variables:
```
NEXT_PUBLIC_SUPABASE_URL=[production value]
NEXT_PUBLIC_SUPABASE_ANON_KEY=[production value]
SUPABASE_SERVICE_ROLE_KEY=[production value]
GOOGLE_CLIENT_ID=[same as local]
GOOGLE_CLIENT_SECRET=[same as local]
```

#### 3. Update Google OAuth Redirect URI

Go to Google Cloud Console → Credentials:
- Add: `https://[your-app].vercel.app/auth/callback`

#### 4. Test Production Deployment

- [ ] Visit https://[your-app].vercel.app
- [ ] Click "Continue with Google"
- [ ] Complete OAuth flow
- [ ] Complete onboarding
- [ ] See dashboard
- [ ] Sign out and sign back in

---

## Technical Stack & Tools

**Frontend:**
- Next.js 14 (App Router)
- TypeScript
- Tailwind CSS
- React Server Components

**Backend:**
- Next.js API Routes (for future use)
- Supabase (PostgreSQL + Auth)

**Authentication:**
- Supabase Auth (Google OAuth)

**Deployment:**
- Vercel (automatic deploys from GitHub)

**References:**
- `ai/guides/supabase-api.md` - Complete Supabase setup guide
- `ai/guides/google-calendar-api.md` - OAuth setup (Phase 2)

---

## File Structure Created

```
family-time-app/
├── app/
│   ├── layout.tsx              # Root layout
│   ├── page.tsx                # Landing page (sign in)
│   ├── globals.css             # Global styles
│   ├── auth/
│   │   ├── callback/
│   │   │   └── route.ts        # OAuth callback handler
│   │   ├── sign-in/
│   │   │   └── route.ts        # Sign in action
│   │   └── sign-out/
│   │       └── route.ts        # Sign out action
│   ├── onboarding/
│   │   └── page.tsx            # Single-page onboarding
│   └── dashboard/
│       ├── layout.tsx          # Dashboard layout (protected)
│       └── page.tsx            # Empty dashboard
├── lib/
│   └── supabase/
│       ├── client.ts           # Browser client
│       └── server.ts           # Server client
├── public/                     # Static assets
├── .env.local                  # Environment variables (gitignored)
├── .gitignore
├── package.json
├── tsconfig.json
├── tailwind.config.ts
├── next.config.js
└── README.md
```

---

## Testing Strategy

### Manual Testing Checklist

**Landing Page:**
- [ ] Page loads without errors
- [ ] "Continue with Google" button works
- [ ] Clicking button redirects to Google OAuth

**OAuth Flow:**
- [ ] Google login page appears
- [ ] After login, redirects back to app
- [ ] User is authenticated
- [ ] Redirects to onboarding (first time) or dashboard (returning)

**Onboarding:**
- [ ] All 4 inputs display correctly
- [ ] Can add multiple kids with different ages
- [ ] Can add multiple items to Want to Try list
- [ ] Can enter sitter name and phone
- [ ] Submit button works
- [ ] Data is saved to Supabase
- [ ] Redirects to dashboard after submit

**Dashboard:**
- [ ] Shows welcome message with user's name
- [ ] Shows empty state message
- [ ] Shows Want to Try list (if any)
- [ ] Sign out button works

**Database:**
- [ ] Check Supabase Dashboard → Table Editor
- [ ] Verify `users` table has record
- [ ] Verify `interests` table has records
- [ ] Verify `trusted_sitters` table has record (if added)
- [ ] Verify RLS policies are working (can't see other users' data)

### Edge Cases to Test

- [ ] User closes browser during OAuth (should be able to sign in again)
- [ ] User skips optional fields in onboarding (should still work)
- [ ] User tries to access /dashboard without signing in (should redirect to /)
- [ ] User tries to access /dashboard without completing onboarding (should redirect to /onboarding)
- [ ] User signs in from different browser (should work)

---

## Risks & Mitigation

### Risk 1: Google OAuth Configuration Issues
**Symptoms:** OAuth fails, redirect doesn't work  
**Mitigation:**
- Double-check redirect URIs match exactly
- Add test users to OAuth consent screen
- Check browser console for errors
- Verify environment variables are set correctly

**Contingency:** Skip OAuth for now, hardcode a test user ID to continue with other phases

### Risk 2: Supabase RLS Blocking Access
**Symptoms:** Can't read/write to database, "new row violates row-level security policy"  
**Mitigation:**
- Check RLS policies in Supabase Dashboard
- Test with `authenticated` role
- Verify `auth.uid()` matches user ID

**Contingency:** Temporarily disable RLS on development tables (re-enable before production)

### Risk 3: Onboarding Form Bugs
**Symptoms:** Form submission fails, data doesn't save  
**Mitigation:**
- Add console.log statements
- Check Network tab in DevTools
- Verify Supabase error messages

**Contingency:** Simplify onboarding to just location (add other fields later)

---

## Deliverables

By end of Week 2:

- [ ] **Live App:** https://[your-app].vercel.app
- [ ] **GitHub Repo:** Code pushed to main branch
- [ ] **Supabase Project:** Database with 8 tables, RLS enabled
- [ ] **Google OAuth:** Working authentication flow
- [ ] **Onboarding Flow:** Single page with 4 inputs working
- [ ] **Empty Dashboard:** User can see dashboard after onboarding
- [ ] **Environment Variables:** Configured in local and Vercel
- [ ] **Documentation:** README with setup instructions

---

## Definition of Done

Phase 1 is complete when:

- ✅ User can visit app at production URL
- ✅ User can sign in with Google
- ✅ User completes onboarding with all 4 inputs
- ✅ User sees empty dashboard
- ✅ Database contains user data
- ✅ App is deployed to Vercel with CI/CD
- ✅ All environment variables are configured
- ✅ No console errors on any page
- ✅ All success criteria met
- ✅ Roadmap checkboxes completed

**Next Phase:** Phase 2 - Calendar Integration (Week 3)

---

**Created:** February 12, 2026  
**Last Updated:** February 12, 2026
