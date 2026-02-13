# Resend Email Service Documentation

**Library:** Resend
**Library ID:** `/resend/resend-node`
**Source:** context7 (February 2026)
**Version:** Latest

## Overview

Resend is a modern email API built for developers. It provides a simple API for sending transactional emails with support for React templates, scheduling, and advanced email features.

## Key Features for Our Project

- **Simple API** - Send emails with minimal code
- **React Templates** - Build emails with React components
- **Email Scheduling** - Schedule emails for future delivery
- **Attachments** - Send files with emails
- **Custom Headers** - Add tags, metadata, and tracking

## Installation

```bash
npm install resend
```

## Setup

### Get API Key

1. Sign up at https://resend.com
2. Verify your domain (or use Resend's test domain)
3. Get API key from dashboard

### Environment Variables

```bash
# .env.local
RESEND_API_KEY=re_...
```

## Basic Usage

### Send Simple Email

```typescript
import { Resend } from 'resend'

const resend = new Resend(process.env.RESEND_API_KEY)

async function sendEmail() {
  const data = await resend.emails.send({
    from: 'Family Time <noreply@familytime.app>',
    to: ['user@example.com'],
    subject: 'Welcome to Family Time!',
    html: '<p>Thanks for joining Family Time!</p>'
  })

  console.log(data.id) // Email ID for tracking
}
```

### Send with Plain Text

```typescript
await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user@example.com'],
  subject: 'Welcome!',
  text: 'Thanks for joining Family Time!',
  html: '<p>Thanks for joining <strong>Family Time</strong>!</p>'
})
```

## React Email Templates

### Create Email Template

```tsx
// emails/WelcomeEmail.tsx
import * as React from 'react'
import {
  Body,
  Button,
  Container,
  Head,
  Html,
  Preview,
  Section,
  Text,
} from '@react-email/components'

interface WelcomeEmailProps {
  userName: string
  familyName: string
}

export default function WelcomeEmail({
  userName,
  familyName
}: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Preview>Welcome to {familyName}!</Preview>
      <Body style={main}>
        <Container style={container}>
          <Section>
            <Text style={heading}>Hi {userName}!</Text>
            <Text style={paragraph}>
              You've been invited to join {familyName} on Family Time.
            </Text>
            <Button style={button} href="https://familytime.app/dashboard">
              View Dashboard
            </Button>
          </Section>
        </Container>
      </Body>
    </Html>
  )
}

const main = {
  backgroundColor: '#f6f9fc',
  fontFamily: '-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Ubuntu,sans-serif',
}

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  padding: '20px 0 48px',
  marginBottom: '64px',
}

const heading = {
  fontSize: '32px',
  lineHeight: '1.3',
  fontWeight: '700',
  color: '#484848',
}

const paragraph = {
  fontSize: '18px',
  lineHeight: '1.4',
  color: '#484848',
}

const button = {
  backgroundColor: '#5469d4',
  borderRadius: '5px',
  color: '#fff',
  fontSize: '16px',
  fontWeight: 'bold',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'block',
  width: '100%',
  padding: '10px',
}
```

### Send React Email Template

```typescript
import { Resend } from 'resend'
import WelcomeEmail from '@/emails/WelcomeEmail'

const resend = new Resend(process.env.RESEND_API_KEY)

async function sendWelcomeEmail(
  to: string,
  userName: string,
  familyName: string
) {
  const data = await resend.emails.send({
    from: 'Family Time <noreply@familytime.app>',
    to: [to],
    subject: `Welcome to ${familyName}!`,
    react: WelcomeEmail({ userName, familyName })
  })

  return data
}
```

## Advanced Features

### Send to Multiple Recipients

```typescript
await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user1@example.com', 'user2@example.com', 'user3@example.com'],
  subject: 'Family Event Reminder',
  react: EventReminderEmail({ eventName: 'Family Dinner' })
})
```

### Add CC and BCC

```typescript
await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user@example.com'],
  cc: ['admin@familytime.app'],
  bcc: ['archive@familytime.app'],
  subject: 'Important Family Update',
  html: '<p>Update content</p>'
})
```

### Add Attachments

```typescript
import fs from 'fs'

await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user@example.com'],
  subject: 'Event Calendar Attachment',
  html: '<p>See attached calendar</p>',
  attachments: [
    {
      filename: 'calendar.ics',
      content: fs.readFileSync('./calendar.ics')
    }
  ]
})
```

### Add Tags for Organization

```typescript
await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user@example.com'],
  subject: 'Event Reminder',
  html: '<p>Reminder content</p>',
  tags: [
    { name: 'category', value: 'reminder' },
    { name: 'family_id', value: 'family_123' }
  ]
})
```

### Custom Headers

```typescript
await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user@example.com'],
  subject: 'Test Email',
  html: '<p>Content</p>',
  headers: {
    'X-Entity-Ref-ID': 'event_456'
  }
})
```

## Email Scheduling

### Schedule Email for Later

```typescript
await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user@example.com'],
  subject: 'Scheduled Reminder',
  html: '<p>This email was scheduled!</p>',
  scheduledAt: '2026-02-15T10:00:00Z' // ISO 8601 format
})
```

### Schedule Email with Date Object

```typescript
const scheduledDate = new Date()
scheduledDate.setDate(scheduledDate.getDate() + 1) // Tomorrow

await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user@example.com'],
  subject: 'Tomorrow Reminder',
  html: '<p>Reminder for tomorrow</p>',
  scheduledAt: scheduledDate.toISOString()
})
```

## Implementation for Our Project

### Email Utility Module

```typescript
// lib/email.ts
import { Resend } from 'resend'
import WelcomeEmail from '@/emails/WelcomeEmail'
import EventReminderEmail from '@/emails/EventReminderEmail'
import InvitationEmail from '@/emails/InvitationEmail'

const resend = new Resend(process.env.RESEND_API_KEY)

export async function sendWelcomeEmail(
  to: string,
  userName: string,
  familyName: string
) {
  try {
    const data = await resend.emails.send({
      from: 'Family Time <noreply@familytime.app>',
      to: [to],
      subject: `Welcome to ${familyName}!`,
      react: WelcomeEmail({ userName, familyName }),
      tags: [
        { name: 'category', value: 'onboarding' }
      ]
    })
    return { success: true, id: data.id }
  } catch (error) {
    console.error('Failed to send welcome email:', error)
    return { success: false, error }
  }
}

export async function sendEventReminder(
  to: string,
  eventTitle: string,
  eventDate: Date,
  familyName: string
) {
  const reminderDate = new Date(eventDate)
  reminderDate.setHours(reminderDate.getHours() - 24) // 24 hours before

  try {
    const data = await resend.emails.send({
      from: 'Family Time <noreply@familytime.app>',
      to: [to],
      subject: `Reminder: ${eventTitle} tomorrow`,
      react: EventReminderEmail({
        eventTitle,
        eventDate,
        familyName
      }),
      scheduledAt: reminderDate.toISOString(),
      tags: [
        { name: 'category', value: 'reminder' },
        { name: 'event_type', value: 'upcoming' }
      ]
    })
    return { success: true, id: data.id }
  } catch (error) {
    console.error('Failed to schedule event reminder:', error)
    return { success: false, error }
  }
}

export async function sendFamilyInvitation(
  to: string,
  inviterName: string,
  familyName: string,
  invitationLink: string
) {
  try {
    const data = await resend.emails.send({
      from: 'Family Time <noreply@familytime.app>',
      to: [to],
      subject: `${inviterName} invited you to ${familyName}`,
      react: InvitationEmail({
        inviterName,
        familyName,
        invitationLink
      }),
      tags: [
        { name: 'category', value: 'invitation' }
      ]
    })
    return { success: true, id: data.id }
  } catch (error) {
    console.error('Failed to send invitation:', error)
    return { success: false, error }
  }
}
```

### Event Reminder Email Template

```tsx
// emails/EventReminderEmail.tsx
import * as React from 'react'
import {
  Body,
  Button,
  Container,
  Head,
  Html,
  Preview,
  Section,
  Text,
} from '@react-email/components'

interface EventReminderEmailProps {
  eventTitle: string
  eventDate: Date
  familyName: string
}

export default function EventReminderEmail({
  eventTitle,
  eventDate,
  familyName
}: EventReminderEmailProps) {
  const formattedDate = new Intl.DateTimeFormat('en-US', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: 'numeric',
    minute: '2-digit',
  }).format(eventDate)

  return (
    <Html>
      <Head />
      <Preview>Reminder: {eventTitle} is tomorrow</Preview>
      <Body style={main}>
        <Container style={container}>
          <Section>
            <Text style={heading}>Event Reminder</Text>
            <Text style={paragraph}>
              Don't forget: <strong>{eventTitle}</strong> is happening tomorrow!
            </Text>
            <Text style={paragraph}>
              <strong>When:</strong> {formattedDate}
            </Text>
            <Text style={paragraph}>
              <strong>Family:</strong> {familyName}
            </Text>
            <Button style={button} href="https://familytime.app/dashboard">
              View Event Details
            </Button>
          </Section>
        </Container>
      </Body>
    </Html>
  )
}

const main = {
  backgroundColor: '#f6f9fc',
  fontFamily: '-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,sans-serif',
}

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  padding: '20px 0 48px',
}

const heading = {
  fontSize: '28px',
  fontWeight: '700',
  color: '#484848',
}

const paragraph = {
  fontSize: '16px',
  lineHeight: '1.4',
  color: '#484848',
}

const button = {
  backgroundColor: '#5469d4',
  borderRadius: '5px',
  color: '#fff',
  fontSize: '16px',
  fontWeight: 'bold',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'block',
  padding: '12px',
}
```

### API Route for Sending Emails

```typescript
// app/api/emails/send-reminder/route.ts
import { auth } from '@clerk/nextjs'
import { NextResponse } from 'next/server'
import { sendEventReminder } from '@/lib/email'
import { prisma } from '@/lib/prisma'

export async function POST(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { eventId } = await request.json()

  // Get event and family members
  const event = await prisma.event.findUnique({
    where: { id: eventId },
    include: {
      family: {
        include: {
          members: {
            include: {
              user: true
            }
          }
        }
      }
    }
  })

  if (!event) {
    return NextResponse.json({ error: 'Event not found' }, { status: 404 })
  }

  // Send reminder to all family members
  const results = await Promise.all(
    event.family.members.map(member =>
      sendEventReminder(
        member.user.email,
        event.title,
        event.startTime,
        event.family.name
      )
    )
  )

  return NextResponse.json({ success: true, results })
}
```

## Best Practices for Our App

1. **Error Handling** - Always wrap email sends in try/catch
2. **Use Tags** - Tag emails by category for analytics
3. **Test with Test Domain** - Use Resend test domain in development
4. **Schedule Carefully** - Validate scheduledAt dates before sending
5. **React Templates** - Use React Email for maintainable templates
6. **Track Email IDs** - Store email IDs in database for tracking
7. **Rate Limiting** - Respect Resend's rate limits

## Error Handling

```typescript
import { Resend } from 'resend'

const resend = new Resend(process.env.RESEND_API_KEY)

async function sendEmailSafely() {
  try {
    const data = await resend.emails.send({
      from: 'Family Time <noreply@familytime.app>',
      to: ['user@example.com'],
      subject: 'Test',
      html: '<p>Test</p>'
    })

    console.log('Email sent:', data.id)
    return { success: true, id: data.id }
  } catch (error) {
    console.error('Email send failed:', error)
    return { success: false, error }
  }
}
```

## Testing

### Development Mode

Use Resend's test domain for development:

```typescript
const from = process.env.NODE_ENV === 'production'
  ? 'Family Time <noreply@familytime.app>'
  : 'onboarding@resend.dev'
```

## Resources

- Official Docs: https://resend.com/docs
- React Email: https://react.email/docs
- API Reference: https://resend.com/docs/api-reference
- Email Templates: https://react.email/examples
- Best Practices: https://resend.com/docs/best-practices
