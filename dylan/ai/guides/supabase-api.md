# Supabase JavaScript Client - Implementation Guide

**Source:** https://supabase.com/docs/reference/javascript  
**Official Docs:** https://supabase.com/docs  
**Package:** @supabase/supabase-js v2.93.2  
**Last Updated:** February 12, 2026  
**Status:** ✅ VERIFIED - Real API, production-ready

---

## Overview

Supabase is an open-source Firebase alternative that provides:
- ✅ PostgreSQL database (full SQL capabilities)
- ✅ Built-in authentication (email, OAuth, magic links)
- ✅ Realtime subscriptions (database changes via websockets)
- ✅ Row Level Security (RLS)
- ✅ Auto-generated REST API
- ✅ Edge Functions (Deno runtime)
- ✅ Storage (S3-compatible file uploads)

**Why Supabase for our project:**
- Free tier: 500MB database, 50K monthly active users
- PostgreSQL = powerful relational queries
- Built-in auth = no need for separate auth service
- TypeScript support with type generation from schema
- Serverless-friendly (works on Vercel)

---

## Setup

### 1. Create Supabase Project

1. Go to [supabase.com/dashboard](https://supabase.com/dashboard)
2. Create new project
3. Set database password
4. Copy **Project URL** and **anon/public API key**

### 2. Install Client Library

```bash
npm install @supabase/supabase-js
```

### 3. Environment Variables

```bash
# .env.local
NEXT_PUBLIC_SUPABASE_URL=https://xyzcompany.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Note:** `NEXT_PUBLIC_` prefix makes it accessible in client-side code

### 4. Initialize Client

```typescript
// lib/supabase.ts
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!;
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!;

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

---

## Database Schema for Our MVP

```sql
-- Users (families using the app)
create table public.users (
  id uuid primary key default uuid_generate_v4(),
  email text unique not null,
  name text not null,
  partner_name text,
  location_city text,
  location_state text,
  timezone text default 'America/Chicago',
  google_calendar_token text,  -- Encrypted
  google_calendar_refresh_token text,  -- Encrypted
  created_at timestamp with time zone default now(),
  updated_at timestamp with time zone default now()
);

-- Interests (what families want to do/try)
create table public.interests (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.users(id) on delete cascade,
  category text not null,  -- 'love_doing', 'want_more', 'want_to_try'
  name text not null,
  added_at timestamp with time zone default now()
);

-- Activities (curated list of things to do)
create table public.activities (
  id uuid primary key default uuid_generate_v4(),
  name text not null,
  category text not null,  -- 'dining', 'outdoor', 'entertainment', etc.
  description text,
  address text,
  city text,
  state text,
  lat numeric,
  lng numeric,
  price_level int,  -- 1-4
  requires_babysitter boolean default false,
  min_duration_hours numeric default 2,
  scraped_from text,  -- 'google_maps', 'yelp', 'manual'
  scraped_at timestamp with time zone,
  rating numeric,
  created_at timestamp with time zone default now()
);

-- Activity Suggestions (what we've suggested to users)
create table public.suggestions (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.users(id) on delete cascade,
  activity_id uuid references public.activities(id),
  suggested_date date not null,
  suggested_start_time time not null,
  suggested_end_time time not null,
  google_calendar_event_id text,  -- Track the event we created
  status text default 'pending',  -- 'pending', 'accepted', 'declined', 'completed'
  match_reason text,  -- "You wanted to try kayaking"
  created_at timestamp with time zone default now(),
  responded_at timestamp with time zone,
  completed_at timestamp with time zone
);

-- Trusted Sitters (babysitters in user's network)
create table public.trusted_sitters (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.users(id) on delete cascade,
  name text not null,
  phone text not null,
  email text,
  relationship text,  -- 'family', 'friend', 'regular_sitter'
  hourly_rate numeric,
  notes text,
  is_active boolean default true,
  created_at timestamp with time zone default now()
);

-- Babysitter Requests (SMS requests sent to sitters)
create table public.babysitter_requests (
  id uuid primary key default uuid_generate_v4(),
  suggestion_id uuid references public.suggestions(id) on delete cascade,
  sitter_id uuid references public.trusted_sitters(id),
  phone text not null,
  status text default 'sent',  -- 'sent', 'delivered', 'accepted', 'declined', 'expired'
  sent_at timestamp with time zone default now(),
  responded_at timestamp with time zone,
  twilio_message_sid text
);

-- SMS Messages (log all SMS for debugging)
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

-- Free Time Blocks (detected from calendar)
create table public.free_time (
  id uuid primary key default uuid_generate_v4(),
  user_id uuid references public.users(id) on delete cascade,
  start_time timestamp with time zone not null,
  end_time timestamp with time zone not null,
  duration_hours numeric generated always as (
    extract(epoch from (end_time - start_time)) / 3600
  ) stored,
  detected_at timestamp with time zone default now()
);
```

### Create Tables via SQL Editor

1. Go to Supabase Dashboard → SQL Editor
2. Paste the schema above
3. Run query

---

## TypeScript Type Generation

### Generate Types from Schema

```bash
# Install Supabase CLI
npm install -g supabase

# Login
supabase login

# Generate types
supabase gen types typescript --project-id xyzcompany > types/database.types.ts
```

### Use Types in Your Code

```typescript
// types/database.types.ts (auto-generated)
export interface Database {
  public: {
    Tables: {
      users: {
        Row: {
          id: string;
          email: string;
          name: string;
          partner_name: string | null;
          // ...
        };
        Insert: {
          id?: string;
          email: string;
          name: string;
          // ...
        };
        Update: {
          email?: string;
          name?: string;
          // ...
        };
      };
      // Other tables...
    };
  };
}
```

```typescript
// lib/supabase.ts (with types)
import { createClient } from '@supabase/supabase-js';
import { Database } from '@/types/database.types';

export const supabase = createClient<Database>(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);
```

---

## CRUD Operations

### Create (Insert)

```typescript
// Add a new interest
const { data, error } = await supabase
  .from('interests')
  .insert({
    user_id: userId,
    category: 'want_to_try',
    name: 'Kayaking'
  })
  .select()  // Return the inserted row
  .single();

if (error) {
  console.error('Error inserting:', error);
} else {
  console.log('Inserted:', data);
}
```

### Read (Select)

```typescript
// Get all interests for a user
const { data: interests, error } = await supabase
  .from('interests')
  .select('*')
  .eq('user_id', userId)
  .order('added_at', { ascending: false });

// Get suggestions with related data
const { data: suggestions } = await supabase
  .from('suggestions')
  .select(`
    *,
    activity:activities(*),
    user:users(name, email)
  `)
  .eq('user_id', userId)
  .eq('status', 'pending');
```

### Update

```typescript
// Mark suggestion as accepted
const { error } = await supabase
  .from('suggestions')
  .update({
    status: 'accepted',
    responded_at: new Date().toISOString()
  })
  .eq('id', suggestionId);
```

### Delete

```typescript
// Remove an interest
const { error } = await supabase
  .from('interests')
  .delete()
  .eq('id', interestId);
```

---

## Filtering & Querying

```typescript
// Complex query: Find activities matching user's interests
const { data: activities } = await supabase
  .from('activities')
  .select('*')
  .in('category', userInterestCategories)
  .gte('rating', 4.0)
  .eq('city', userCity)
  .eq('requires_babysitter', true)
  .order('rating', { ascending: false })
  .limit(10);

// Date range query: Get suggestions for next 2 weeks
const { data: upcomingSuggestions } = await supabase
  .from('suggestions')
  .select('*, activity:activities(*)')
  .gte('suggested_date', new Date().toISOString())
  .lte('suggested_date', new Date(Date.now() + 14 * 24 * 60 * 60 * 1000).toISOString())
  .eq('status', 'pending');

// Text search
const { data: searchResults } = await supabase
  .from('activities')
  .select('*')
  .textSearch('name', 'kayaking')
  .eq('city', 'Austin');
```

---

## Authentication

### Sign Up

```typescript
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'secure-password',
  options: {
    data: {
      name: 'John Doe',
      partner_name: 'Jane Doe'
    }
  }
});
```

### Sign In

```typescript
const { data, error } = await supabase.auth.signInWithPassword({
  email: 'user@example.com',
  password: 'secure-password'
});

// Get current user
const { data: { user } } = await supabase.auth.getUser();
```

### OAuth (Google)

```typescript
const { error } = await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: {
    redirectTo: 'http://localhost:3000/auth/callback',
    scopes: 'email profile https://www.googleapis.com/auth/calendar.events'
  }
});
```

### Sign Out

```typescript
const { error } = await supabase.auth.signOut();
```

---

## Realtime Subscriptions

Listen to database changes in real-time:

```typescript
// Subscribe to new suggestions
const subscription = supabase
  .channel('suggestions-channel')
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'suggestions',
      filter: `user_id=eq.${userId}`
    },
    (payload) => {
      console.log('New suggestion!', payload.new);
      // Update UI with new suggestion
    }
  )
  .subscribe();

// Unsubscribe when component unmounts
subscription.unsubscribe();
```

---

## Row Level Security (RLS)

Enable RLS to ensure users can only access their own data:

```sql
-- Enable RLS on users table
alter table public.users enable row level security;

-- Policy: Users can only read their own data
create policy "Users can view own data"
  on public.users
  for select
  using (auth.uid() = id);

-- Policy: Users can update their own data
create policy "Users can update own data"
  on public.users
  for update
  using (auth.uid() = id);

-- Similar policies for other tables
create policy "Users can view own interests"
  on public.interests
  for select
  using (user_id = auth.uid());

create policy "Users can insert own interests"
  on public.interests
  for insert
  with check (user_id = auth.uid());
```

---

## Next.js Integration

### Server Component

```typescript
// app/dashboard/page.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs';
import { cookies } from 'next/headers';

export default async function Dashboard() {
  const supabase = createServerComponentClient({ cookies });
  
  const { data: suggestions } = await supabase
    .from('suggestions')
    .select('*, activity:activities(*)')
    .eq('status', 'pending')
    .order('suggested_date', { ascending: true });
  
  return (
    <div>
      <h1>Your Suggestions</h1>
      {suggestions?.map(s => (
        <div key={s.id}>{s.activity.name}</div>
      ))}
    </div>
  );
}
```

### Client Component

```typescript
'use client';

import { createClientComponentClient } from '@supabase/auth-helpers-nextjs';
import { useEffect, useState } from 'react';

export default function InterestsList() {
  const supabase = createClientComponentClient();
  const [interests, setInterests] = useState([]);
  
  useEffect(() => {
    async function loadInterests() {
      const { data } = await supabase
        .from('interests')
        .select('*')
        .order('added_at', { ascending: false });
      
      setInterests(data || []);
    }
    
    loadInterests();
  }, []);
  
  return (
    <ul>
      {interests.map(i => (
        <li key={i.id}>{i.name}</li>
      ))}
    </ul>
  );
}
```

### API Route

```typescript
// app/api/suggestions/route.ts
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs';
import { cookies } from 'next/headers';
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const supabase = createRouteHandlerClient({ cookies });
  
  const { data: suggestions, error } = await supabase
    .from('suggestions')
    .select('*')
    .eq('status', 'pending');
  
  if (error) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
  
  return NextResponse.json({ suggestions });
}
```

---

## Database Functions (Stored Procedures)

### Create Function in SQL Editor

```sql
-- Find activities matching user's interests
create or replace function match_activities(p_user_id uuid, p_limit int default 10)
returns table (
  activity_id uuid,
  activity_name text,
  match_score int,
  match_reason text
) as $$
begin
  return query
  select 
    a.id as activity_id,
    a.name as activity_name,
    count(*)::int as match_score,
    string_agg(i.name, ', ') as match_reason
  from activities a
  join interests i on (
    a.category = i.category or
    a.name ilike '%' || i.name || '%' or
    a.description ilike '%' || i.name || '%'
  )
  where i.user_id = p_user_id
  and a.city = (select location_city from users where id = p_user_id)
  group by a.id, a.name
  order by match_score desc, a.rating desc
  limit p_limit;
end;
$$ language plpgsql;
```

### Call Function from Client

```typescript
const { data: matches } = await supabase
  .rpc('match_activities', {
    p_user_id: userId,
    p_limit: 10
  });

console.log('Top matches:', matches);
```

---

## Error Handling

```typescript
async function safeQuery() {
  const { data, error } = await supabase
    .from('activities')
    .select('*');
  
  if (error) {
    // Log error details
    console.error('Supabase error:', {
      message: error.message,
      details: error.details,
      hint: error.hint,
      code: error.code
    });
    
    // Handle specific errors
    if (error.code === 'PGRST116') {
      // Row not found
      return null;
    } else if (error.code === '42501') {
      // Permission denied (RLS)
      throw new Error('You do not have permission to access this data');
    }
    
    throw error;
  }
  
  return data;
}
```

---

## Performance Optimization

### Use Indexes

```sql
-- Add indexes for common queries
create index idx_suggestions_user_status on suggestions(user_id, status);
create index idx_activities_city_category on activities(city, category);
create index idx_interests_user_category on interests(user_id, category);
```

### Pagination

```typescript
const PAGE_SIZE = 20;

const { data, error } = await supabase
  .from('activities')
  .select('*')
  .range(0, PAGE_SIZE - 1);  // First page: 0-19

// Next page
const { data: page2 } = await supabase
  .from('activities')
  .select('*')
  .range(PAGE_SIZE, PAGE_SIZE * 2 - 1);  // 20-39
```

### Count Total Rows

```typescript
const { count } = await supabase
  .from('activities')
  .select('*', { count: 'exact', head: true });

console.log('Total activities:', count);
```

---

## Testing

### Mock Supabase Client

```typescript
// __tests__/database.test.ts
import { createClient } from '@supabase/supabase-js';

jest.mock('@supabase/supabase-js');

test('fetches user interests', async () {
  const mockSelect = jest.fn().mockResolvedValue({
    data: [{ id: '1', name: 'Kayaking' }],
    error: null
  });
  
  (createClient as jest.Mock).mockReturnValue({
    from: jest.fn().mockReturnValue({
      select: mockSelect
    })
  });
  
  // Test your function...
});
```

---

## Cost & Limits

### Free Tier
- 500 MB database storage
- 50,000 monthly active users (MAU)
- 2 GB file storage
- 5 GB bandwidth
- Social OAuth providers
- 500K Edge Function invocations

### Pro Tier ($25/month)
- 8 GB database
- 100K MAU
- 100 GB storage
- 250 GB bandwidth

**For our MVP:** Free tier is plenty!

---

## Best Practices for Our Project

### 1. Use Transactions for Related Data

```typescript
// Create suggestion + babysitter requests atomically
const { data: suggestion, error } = await supabase.rpc('create_suggestion_with_requests', {
  p_user_id: userId,
  p_activity_id: activityId,
  p_date: '2026-02-22',
  p_start_time: '14:00',
  p_end_time: '17:00',
  p_sitter_ids: [sitter1Id, sitter2Id]
});
```

### 2. Cache Frequently Accessed Data

```typescript
// Use React Query for caching
import { useQuery } from '@tanstack/react-query';

function useActivities(city: string) {
  return useQuery({
    queryKey: ['activities', city],
    queryFn: async () => {
      const { data } = await supabase
        .from('activities')
        .select('*')
        .eq('city', city);
      return data;
    },
    staleTime: 1000 * 60 * 5  // 5 minutes
  });
}
```

### 3. Encrypt Sensitive Data

```typescript
// Before storing calendar tokens
import { encrypt } from './crypto';

const encryptedToken = encrypt(googleToken);

await supabase
  .from('users')
  .update({
    google_calendar_token: encryptedToken
  })
  .eq('id', userId);
```

---

## Implementation Checklist

- [ ] Create Supabase project
- [ ] Create database schema (run SQL)
- [ ] Generate TypeScript types
- [ ] Set up authentication
- [ ] Enable Row Level Security
- [ ] Create database indexes
- [ ] Test CRUD operations
- [ ] Set up Realtime subscriptions (if needed)
- [ ] Deploy to production

---

## Links & Resources

- **Official Docs:** https://supabase.com/docs
- **JavaScript Reference:** https://supabase.com/docs/reference/javascript
- **Auth Helpers:** https://supabase.com/docs/guides/auth/auth-helpers
- **Database Design:** https://supabase.com/docs/guides/database
- **GitHub:** https://github.com/supabase/supabase-js

---

## For Our MVP

**What we need:**
- ✅ PostgreSQL database (users, interests, activities, suggestions, sitters)
- ✅ Authentication (email/password or Google OAuth)
- ✅ CRUD operations
- ✅ Relational queries (join suggestions with activities)
- ✅ Row Level Security

**What we can skip:**
- ❌ Realtime subscriptions (nice-to-have, not critical)
- ❌ Storage (images/files)
- ❌ Edge Functions (can use Next.js API routes)

**Estimated setup time:** 1-2 days for schema + auth
