# Next.js 14 Documentation

**Library:** Next.js
**Library ID:** `/vercel/next.js`
**Source:** context7 (February 2026)
**Version:** 14.3.0+

## Overview

Next.js is a React framework for building full-stack web applications with server-side rendering, static site generation, and API routes. It extends React with powerful features and optimizations.

## Key Features for Our Project

- **App Router** - File-system based routing with React Server Components
- **Server-Side Rendering (SSR)** - Dynamic content with fresh data on every request
- **API Routes** - Built-in API endpoints without separate backend
- **TypeScript Support** - First-class TypeScript integration
- **Vercel Deployment** - Optimized hosting with zero configuration

## Data Fetching in App Router

### Fetch Dynamic Server-Side Data

```tsx
// app/dashboard/page.tsx
async function getProjects() {
  const res = await fetch(`https://api.example.com/projects`, {
    cache: 'no-store' // Don't cache - always fetch fresh data
  })
  const projects = await res.json()
  return projects
}

export default async function Dashboard() {
  const projects = await getProjects()

  return (
    <ul>
      {projects.map((project) => (
        <li key={project.id}>{project.name}</li>
      ))}
    </ul>
  )
}
```

### Data Fetching Strategies

```tsx
export default async function Page() {
  // Static: Cached until manually invalidated (default)
  const staticData = await fetch(`https://...`, {
    cache: 'force-cache' // Default, can be omitted
  })

  // Dynamic: Refetched on every request
  const dynamicData = await fetch(`https://...`, {
    cache: 'no-store'
  })

  // Revalidated: Cached with TTL (10 seconds)
  const revalidatedData = await fetch(`https://...`, {
    next: { revalidate: 10 }
  })

  return <div>...</div>
}
```

## Server Components

Next.js App Router uses React Server Components by default:

```tsx
// This component runs on the server
export default async function Page() {
  const data = await fetch('https://api.vercel.app/blog')
  const posts = await data.json()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

**Benefits:**
- Direct database/API access without exposing credentials
- Faster initial page load (no client-side JS needed)
- Better SEO
- Automatic code splitting

## Implementation for Our Project

### Phase 2A: Next.js Setup

**File Structure:**
```
app/
├── layout.tsx         # Root layout
├── page.tsx           # Landing page
├── setup/             # Onboarding flow
│   └── page.tsx
├── dashboard/         # User dashboard
│   └── page.tsx
└── api/               # API routes
    └── calendar/
        └── route.ts
```

**Basic Setup:**

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

### API Routes

```tsx
// app/api/calendar/route.ts
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const userId = searchParams.get('userId')

  // Fetch calendar data
  const events = await fetchCalendarEvents(userId)

  return NextResponse.json({ events })
}
```

## Migration from Pages Router (Not Needed)

We're starting fresh with App Router, so no migration needed. The snippets above show the modern App Router patterns we'll use.

## Key Differences from Pages Router

**Old (Pages Router):**
- `getServerSideProps`, `getStaticProps`
- `pages/` directory
- Client-side data fetching

**New (App Router):**
- `async` Server Components
- `app/` directory
- Server-side data fetching by default

## Performance Optimizations

- **Automatic Code Splitting** - Only load what's needed
- **Image Optimization** - `next/image` component
- **Font Optimization** - `next/font`
- **Script Optimization** - `next/script`

## Deployment to Vercel

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel
```

**Environment Variables:**
- Set in Vercel dashboard
- Automatically injected during build
- Encrypted at rest

## Best Practices for Our App

1. **Use Server Components by default** - Only use 'use client' when needed
2. **Colocate data fetching** - Fetch data in the component that needs it
3. **Leverage caching** - Use appropriate cache strategies
4. **TypeScript everywhere** - Use strict mode
5. **Environment variables** - Never commit secrets

## Common Patterns

### Loading States

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <div>Loading...</div>
}
```

### Error Handling

```tsx
// app/dashboard/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={() => reset()}>Try again</button>
    </div>
  )
}
```

### Not Found Pages

```tsx
// app/dashboard/not-found.tsx
export default function NotFound() {
  return <div>Dashboard not found</div>
}
```

## Resources

- Official Docs: https://nextjs.org/docs
- App Router Guide: https://nextjs.org/docs/app
- Deployment: https://vercel.com/docs
- Examples: https://github.com/vercel/next.js/tree/canary/examples
