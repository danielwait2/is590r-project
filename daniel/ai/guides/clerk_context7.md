# Clerk Authentication Documentation

**Library:** Clerk
**Library ID:** `/clerk/clerk-docs`
**Source:** context7 (February 2026)
**Version:** Latest

## Overview

Clerk is a complete user authentication and management platform built for React and Next.js. It provides pre-built UI components, secure session management, and social OAuth providers like Google.

## Key Features for Our Project

- **Google OAuth** - One-click sign-in with Google accounts
- **Next.js Integration** - Built-in middleware and server helpers
- **Session Management** - Secure, automatic session handling
- **User Management** - Profile management and user metadata
- **Pre-built Components** - Drop-in sign-in/sign-up UI

## Installation

```bash
npm install @clerk/nextjs
```

## Setup

### 1. Get API Keys

Sign up at https://clerk.com and create an application. Get:
- Publishable Key
- Secret Key

### 2. Environment Variables

```bash
# .env.local
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...

# Redirect URLs
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/setup
```

### 3. Wrap App with ClerkProvider

```tsx
// app/layout.tsx
import { ClerkProvider } from '@clerk/nextjs'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <ClerkProvider>
      <html lang="en">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  )
}
```

### 4. Add Middleware for Route Protection

```typescript
// middleware.ts
import { authMiddleware } from "@clerk/nextjs"

export default authMiddleware({
  publicRoutes: ["/", "/sign-in", "/sign-up", "/api/webhooks/(.*)"]
})

export const config = {
  matcher: ['/((?!.+\\.[\\w]+$|_next).*)', '/', '/(api|trpc)(.*)'],
}
```

## Google OAuth Setup

### Enable Google Provider in Clerk Dashboard

1. Go to Clerk Dashboard > User & Authentication > Social Connections
2. Enable Google
3. Add OAuth redirect URI: `https://your-domain.com/sso-callback`

### Sign-In Component with Google

```tsx
// app/sign-in/[[...sign-in]]/page.tsx
import { SignIn } from "@clerk/nextjs"

export default function SignInPage() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <SignIn
        appearance={{
          elements: {
            rootBox: "mx-auto",
            card: "shadow-lg"
          }
        }}
      />
    </div>
  )
}
```

### Sign-Up Component

```tsx
// app/sign-up/[[...sign-up]]/page.tsx
import { SignUp } from "@clerk/nextjs"

export default function SignUpPage() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <SignUp
        appearance={{
          elements: {
            rootBox: "mx-auto",
            card: "shadow-lg"
          }
        }}
      />
    </div>
  )
}
```

## Authentication in Server Components

### Get Current User

```tsx
// app/dashboard/page.tsx
import { auth, currentUser } from '@clerk/nextjs'

export default async function DashboardPage() {
  const { userId } = auth()

  if (!userId) {
    redirect('/sign-in')
  }

  const user = await currentUser()

  return (
    <div>
      <h1>Welcome, {user?.firstName}!</h1>
      <p>Email: {user?.emailAddresses[0]?.emailAddress}</p>
    </div>
  )
}
```

### Protect Routes

```tsx
import { auth } from '@clerk/nextjs'
import { redirect } from 'next/navigation'

export default async function ProtectedPage() {
  const { userId } = auth()

  if (!userId) {
    redirect('/sign-in')
  }

  return <div>Protected content</div>
}
```

## Authentication in Client Components

```tsx
'use client'

import { useUser, useAuth } from '@clerk/nextjs'

export default function ProfileComponent() {
  const { user, isLoaded, isSignedIn } = useUser()
  const { signOut } = useAuth()

  if (!isLoaded) {
    return <div>Loading...</div>
  }

  if (!isSignedIn) {
    return <div>Please sign in</div>
  }

  return (
    <div>
      <p>Hello, {user.firstName}!</p>
      <button onClick={() => signOut()}>Sign Out</button>
    </div>
  )
}
```

## API Routes Authentication

```typescript
// app/api/families/route.ts
import { auth } from '@clerk/nextjs'
import { NextResponse } from 'next/server'

export async function GET(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Fetch user's families
  const families = await getFamilies(userId)

  return NextResponse.json({ families })
}

export async function POST(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const body = await request.json()

  // Create family for user
  const family = await createFamily(userId, body)

  return NextResponse.json({ family })
}
```

## User Metadata

### Store Custom Data

```typescript
import { clerkClient } from '@clerk/nextjs'

// Update user metadata
await clerkClient.users.updateUserMetadata(userId, {
  publicMetadata: {
    onboardingComplete: true
  },
  privateMetadata: {
    familyId: 'family_123'
  }
})

// Read metadata
const user = await clerkClient.users.getUser(userId)
console.log(user.publicMetadata)
```

### Access Metadata in Components

```tsx
import { currentUser } from '@clerk/nextjs'

export default async function Page() {
  const user = await currentUser()
  const hasCompletedOnboarding = user?.publicMetadata?.onboardingComplete

  if (!hasCompletedOnboarding) {
    redirect('/setup')
  }

  return <div>Dashboard</div>
}
```

## Session Management

### Get Session Information

```tsx
import { auth } from '@clerk/nextjs'

export default async function Page() {
  const { sessionId, userId, sessionClaims } = auth()

  console.log('Session ID:', sessionId)
  console.log('User ID:', userId)
  console.log('Session Claims:', sessionClaims)

  return <div>Session active</div>
}
```

### Custom Session Claims

```typescript
// Add custom claims in Clerk Dashboard or via API
await clerkClient.sessions.createSessionToken(sessionId, {
  familyId: 'family_123',
  role: 'admin'
})
```

## Webhooks for User Sync

### Setup Webhook in Clerk Dashboard

1. Go to Webhooks in Clerk Dashboard
2. Add endpoint: `https://your-domain.com/api/webhooks/clerk`
3. Select events: `user.created`, `user.updated`, `user.deleted`

### Handle Webhook

```typescript
// app/api/webhooks/clerk/route.ts
import { Webhook } from 'svix'
import { headers } from 'next/headers'
import { WebhookEvent } from '@clerk/nextjs/server'
import { prisma } from '@/lib/prisma'

export async function POST(req: Request) {
  const WEBHOOK_SECRET = process.env.CLERK_WEBHOOK_SECRET

  if (!WEBHOOK_SECRET) {
    throw new Error('Missing CLERK_WEBHOOK_SECRET')
  }

  const headerPayload = headers()
  const svix_id = headerPayload.get("svix-id")
  const svix_timestamp = headerPayload.get("svix-timestamp")
  const svix_signature = headerPayload.get("svix-signature")

  if (!svix_id || !svix_timestamp || !svix_signature) {
    return new Response('Error: Missing svix headers', { status: 400 })
  }

  const payload = await req.json()
  const body = JSON.stringify(payload)

  const wh = new Webhook(WEBHOOK_SECRET)
  let evt: WebhookEvent

  try {
    evt = wh.verify(body, {
      "svix-id": svix_id,
      "svix-timestamp": svix_timestamp,
      "svix-signature": svix_signature,
    }) as WebhookEvent
  } catch (err) {
    return new Response('Error: Verification failed', { status: 400 })
  }

  const eventType = evt.type

  if (eventType === 'user.created') {
    const { id, email_addresses, first_name, last_name } = evt.data

    await prisma.user.create({
      data: {
        clerkId: id,
        email: email_addresses[0].email_address,
        name: `${first_name} ${last_name}`.trim() || null
      }
    })
  }

  if (eventType === 'user.updated') {
    const { id, email_addresses, first_name, last_name } = evt.data

    await prisma.user.update({
      where: { clerkId: id },
      data: {
        email: email_addresses[0].email_address,
        name: `${first_name} ${last_name}`.trim() || null
      }
    })
  }

  if (eventType === 'user.deleted') {
    const { id } = evt.data

    await prisma.user.delete({
      where: { clerkId: id }
    })
  }

  return new Response('Webhook processed', { status: 200 })
}
```

## Implementation for Our Project

### Onboarding Flow

```tsx
// app/setup/page.tsx
import { auth, currentUser } from '@clerk/nextjs'
import { redirect } from 'next/navigation'
import { prisma } from '@/lib/prisma'

export default async function SetupPage() {
  const { userId } = auth()
  const user = await currentUser()

  if (!userId) {
    redirect('/sign-in')
  }

  // Check if user has completed onboarding
  const dbUser = await prisma.user.findUnique({
    where: { clerkId: userId },
    include: { families: true }
  })

  if (dbUser && dbUser.families.length > 0) {
    redirect('/dashboard')
  }

  return (
    <div>
      <h1>Welcome, {user?.firstName}!</h1>
      <OnboardingForm userId={userId} />
    </div>
  )
}
```

### User Button Component

```tsx
'use client'

import { UserButton } from '@clerk/nextjs'

export default function Header() {
  return (
    <header>
      <nav>
        <UserButton
          afterSignOutUrl="/"
          appearance={{
            elements: {
              avatarBox: "w-10 h-10"
            }
          }}
        />
      </nav>
    </header>
  )
}
```

## Best Practices for Our App

1. **Always use middleware** - Protect routes at the edge
2. **Sync users to database** - Use webhooks to keep Prisma in sync with Clerk
3. **Use server-side auth** - Prefer `auth()` over client-side hooks
4. **Store app data separately** - Use metadata for flags, Prisma for app data
5. **Handle sign-out everywhere** - Use `<UserButton>` for consistent UX
6. **Validate on server** - Never trust client-side auth alone

## Common Patterns

### Check Auth Status

```typescript
import { auth } from '@clerk/nextjs'

const { userId } = auth()
const isAuthenticated = !!userId
```

### Get User Email

```typescript
const user = await currentUser()
const email = user?.emailAddresses[0]?.emailAddress
```

### Conditional Rendering

```tsx
import { auth } from '@clerk/nextjs'

export default async function Page() {
  const { userId } = auth()

  return (
    <div>
      {userId ? (
        <AuthenticatedContent />
      ) : (
        <PublicContent />
      )}
    </div>
  )
}
```

## Resources

- Official Docs: https://clerk.com/docs
- Next.js Integration: https://clerk.com/docs/nextjs
- Google OAuth Setup: https://clerk.com/docs/authentication/social-connections/google
- Webhooks Guide: https://clerk.com/docs/integrations/webhooks
- API Reference: https://clerk.com/docs/reference/clerkjs
