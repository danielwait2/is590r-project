# Stripe Payment Processing Documentation

**Library:** Stripe Node.js
**Library ID:** `/stripe/stripe-node`
**Source:** context7 (February 2026)
**Version:** Latest

## Overview

Stripe is a complete payment platform for the internet. It provides APIs for accepting payments, managing subscriptions, and handling customer billing. The Node.js library makes it easy to integrate Stripe into server-side applications.

## Key Features for Our Project

- **Subscription Management** - Recurring billing for family plans
- **Checkout Sessions** - Pre-built payment pages
- **Customer Management** - Track family accounts
- **Webhooks** - Handle payment events
- **Payment Methods** - Store and charge payment methods

## Installation

```bash
npm install stripe
```

## Setup

### Get API Keys

1. Sign up at https://stripe.com
2. Get test keys from Dashboard > Developers > API keys
3. For production, activate account and get live keys

### Environment Variables

```bash
# .env.local
STRIPE_SECRET_KEY=sk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### Initialize Stripe

```typescript
// lib/stripe.ts
import Stripe from 'stripe'

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-11-20.acacia',
  typescript: true,
})
```

## Subscription Setup

### Create Price and Product

Do this in Stripe Dashboard or via API:

```typescript
// Create product (one-time setup)
const product = await stripe.products.create({
  name: 'Family Time Pro',
  description: 'Premium family organization features'
})

// Create price
const price = await stripe.prices.create({
  product: product.id,
  unit_amount: 999, // $9.99 in cents
  currency: 'usd',
  recurring: {
    interval: 'month'
  }
})
```

### Create Checkout Session

```typescript
import { stripe } from '@/lib/stripe'

async function createCheckoutSession(
  familyId: string,
  priceId: string,
  customerEmail: string
) {
  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    payment_method_types: ['card'],
    line_items: [
      {
        price: priceId,
        quantity: 1,
      },
    ],
    customer_email: customerEmail,
    metadata: {
      familyId: familyId,
    },
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
  })

  return session
}
```

### Redirect to Checkout

```typescript
// app/api/checkout/route.ts
import { auth } from '@clerk/nextjs'
import { NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe'
import { prisma } from '@/lib/prisma'

export async function POST(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { familyId } = await request.json()

  const user = await prisma.user.findUnique({
    where: { clerkId: userId }
  })

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    payment_method_types: ['card'],
    line_items: [
      {
        price: process.env.STRIPE_PRICE_ID!,
        quantity: 1,
      },
    ],
    customer_email: user!.email,
    metadata: {
      familyId,
      userId: user!.id,
    },
    success_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard?success=true`,
    cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing?canceled=true`,
  })

  return NextResponse.json({ sessionId: session.id, url: session.url })
}
```

```tsx
// Client-side checkout button
'use client'

export default function CheckoutButton({ familyId }: { familyId: string }) {
  async function handleCheckout() {
    const response = await fetch('/api/checkout', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ familyId })
    })

    const { url } = await response.json()
    window.location.href = url
  }

  return (
    <button onClick={handleCheckout}>
      Subscribe to Pro
    </button>
  )
}
```

## Customer Management

### Create Customer

```typescript
const customer = await stripe.customers.create({
  email: 'user@example.com',
  name: 'Smith Family',
  metadata: {
    familyId: 'family_123'
  }
})
```

### Retrieve Customer

```typescript
const customer = await stripe.customers.retrieve('cus_xxx')
```

### Update Customer

```typescript
const customer = await stripe.customers.update('cus_xxx', {
  email: 'newemail@example.com'
})
```

## Subscription Management

### Retrieve Subscription

```typescript
const subscription = await stripe.subscriptions.retrieve('sub_xxx')

console.log(subscription.status) // 'active', 'past_due', 'canceled'
console.log(subscription.current_period_end) // Unix timestamp
```

### Cancel Subscription

```typescript
// Cancel at period end
const subscription = await stripe.subscriptions.update('sub_xxx', {
  cancel_at_period_end: true
})

// Cancel immediately
const subscription = await stripe.subscriptions.cancel('sub_xxx')
```

### Update Subscription

```typescript
// Change price/plan
const subscription = await stripe.subscriptions.update('sub_xxx', {
  items: [
    {
      id: subscription.items.data[0].id,
      price: 'price_new_plan',
    },
  ],
  proration_behavior: 'always_invoice' // Charge/credit immediately
})
```

## Webhooks

### Setup Webhook Endpoint

```typescript
// app/api/webhooks/stripe/route.ts
import { headers } from 'next/headers'
import { NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe'
import { prisma } from '@/lib/prisma'
import Stripe from 'stripe'

export async function POST(request: Request) {
  const body = await request.text()
  const signature = headers().get('stripe-signature')!

  let event: Stripe.Event

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    )
  } catch (err) {
    console.error('Webhook signature verification failed:', err)
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 })
  }

  try {
    switch (event.type) {
      case 'checkout.session.completed': {
        const session = event.data.object as Stripe.Checkout.Session
        await handleCheckoutComplete(session)
        break
      }

      case 'customer.subscription.created':
      case 'customer.subscription.updated': {
        const subscription = event.data.object as Stripe.Subscription
        await handleSubscriptionUpdate(subscription)
        break
      }

      case 'customer.subscription.deleted': {
        const subscription = event.data.object as Stripe.Subscription
        await handleSubscriptionDeleted(subscription)
        break
      }

      case 'invoice.payment_succeeded': {
        const invoice = event.data.object as Stripe.Invoice
        await handlePaymentSucceeded(invoice)
        break
      }

      case 'invoice.payment_failed': {
        const invoice = event.data.object as Stripe.Invoice
        await handlePaymentFailed(invoice)
        break
      }
    }

    return NextResponse.json({ received: true })
  } catch (err) {
    console.error('Webhook handler failed:', err)
    return NextResponse.json({ error: 'Webhook handler failed' }, { status: 500 })
  }
}

async function handleCheckoutComplete(session: Stripe.Checkout.Session) {
  const familyId = session.metadata!.familyId
  const customerId = session.customer as string
  const subscriptionId = session.subscription as string

  await prisma.subscription.create({
    data: {
      familyId,
      stripeCustomerId: customerId,
      stripeSubscriptionId: subscriptionId,
      stripePriceId: session.line_items?.data[0]?.price?.id || '',
      status: 'active',
      currentPeriodEnd: new Date(session.expires_at * 1000),
    }
  })
}

async function handleSubscriptionUpdate(subscription: Stripe.Subscription) {
  const familyId = subscription.metadata.familyId

  await prisma.subscription.upsert({
    where: {
      stripeSubscriptionId: subscription.id
    },
    update: {
      status: subscription.status,
      stripePriceId: subscription.items.data[0].price.id,
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
    create: {
      familyId,
      stripeCustomerId: subscription.customer as string,
      stripeSubscriptionId: subscription.id,
      stripePriceId: subscription.items.data[0].price.id,
      status: subscription.status,
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    }
  })
}

async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  await prisma.subscription.update({
    where: {
      stripeSubscriptionId: subscription.id
    },
    data: {
      status: 'canceled',
    }
  })
}

async function handlePaymentSucceeded(invoice: Stripe.Invoice) {
  const subscriptionId = invoice.subscription as string

  await prisma.subscription.update({
    where: {
      stripeSubscriptionId: subscriptionId
    },
    data: {
      status: 'active',
    }
  })
}

async function handlePaymentFailed(invoice: Stripe.Invoice) {
  const subscriptionId = invoice.subscription as string

  await prisma.subscription.update({
    where: {
      stripeSubscriptionId: subscriptionId
    },
    data: {
      status: 'past_due',
    }
  })

  // Send email to family admin about failed payment
}
```

### Register Webhook in Stripe

1. Go to Stripe Dashboard > Developers > Webhooks
2. Add endpoint: `https://your-domain.com/api/webhooks/stripe`
3. Select events to listen for
4. Copy webhook signing secret to `.env`

## Implementation for Our Project

### Check Subscription Status

```typescript
// lib/subscription.ts
import { prisma } from '@/lib/prisma'

export async function hasActiveSubscription(familyId: string): Promise<boolean> {
  const subscription = await prisma.subscription.findUnique({
    where: { familyId }
  })

  if (!subscription) return false

  return subscription.status === 'active'
}

export async function getSubscriptionStatus(familyId: string) {
  const subscription = await prisma.subscription.findUnique({
    where: { familyId }
  })

  return subscription
}
```

### Protect Premium Features

```typescript
// app/dashboard/premium/page.tsx
import { auth } from '@clerk/nextjs'
import { redirect } from 'next/navigation'
import { hasActiveSubscription } from '@/lib/subscription'
import { getUserFamilies } from '@/lib/queries'

export default async function PremiumPage() {
  const { userId } = auth()

  if (!userId) {
    redirect('/sign-in')
  }

  const families = await getUserFamilies(userId)
  const primaryFamily = families[0]

  if (!primaryFamily) {
    redirect('/setup')
  }

  const hasSubscription = await hasActiveSubscription(primaryFamily.familyId)

  if (!hasSubscription) {
    redirect('/pricing')
  }

  return <div>Premium content</div>
}
```

### Customer Portal

```typescript
// app/api/customer-portal/route.ts
import { auth } from '@clerk/nextjs'
import { NextResponse } from 'next/server'
import { stripe } from '@/lib/stripe'
import { prisma } from '@/lib/prisma'

export async function POST(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { familyId } = await request.json()

  const subscription = await prisma.subscription.findUnique({
    where: { familyId }
  })

  if (!subscription) {
    return NextResponse.json({ error: 'No subscription found' }, { status: 404 })
  }

  const session = await stripe.billingPortal.sessions.create({
    customer: subscription.stripeCustomerId,
    return_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard`,
  })

  return NextResponse.json({ url: session.url })
}
```

## Best Practices for Our App

1. **Use Webhooks** - Never rely on client-side confirmation alone
2. **Test Mode First** - Use test keys and test cards during development
3. **Idempotency** - Stripe handles this automatically with idempotency keys
4. **Error Handling** - Always wrap Stripe calls in try/catch
5. **Metadata** - Store familyId and userId in metadata for webhooks
6. **Customer Portal** - Let customers manage their own subscriptions
7. **Security** - Never expose secret key, validate webhooks

## Test Cards

```
4242 4242 4242 4242 - Success
4000 0025 0000 3155 - Requires authentication
4000 0000 0000 9995 - Declined
```

## Common Patterns

### Check if Family Has Active Subscription

```typescript
const subscription = await prisma.subscription.findUnique({
  where: { familyId },
  select: { status: true }
})

const isActive = subscription?.status === 'active'
```

### Get Subscription End Date

```typescript
const subscription = await prisma.subscription.findUnique({
  where: { familyId }
})

const endDate = subscription?.currentPeriodEnd
```

## Resources

- Official Docs: https://stripe.com/docs
- Node.js Library: https://stripe.com/docs/api/node
- Webhooks Guide: https://stripe.com/docs/webhooks
- Testing: https://stripe.com/docs/testing
- Checkout Sessions: https://stripe.com/docs/payments/checkout
- Subscriptions: https://stripe.com/docs/billing/subscriptions
