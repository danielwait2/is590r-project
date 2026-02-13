# React Email Documentation

**Library:** React Email
**Library ID:** `/resend/react-email`
**Source:** context7 (February 2026)
**Version:** Latest

## Overview

React Email is a library for building responsive HTML emails using React components. It renders React to HTML and provides a set of pre-built components optimized for email clients.

## Key Features for Our Project

- **React Components** - Build emails like web pages
- **Responsive Design** - Works across all email clients
- **TypeScript Support** - Type-safe email templates
- **Preview Server** - See emails during development
- **Pre-built Components** - Button, Container, Text, etc.

## Installation

```bash
npm install react-email @react-email/components
```

## Setup

### Create Email Directory

```bash
mkdir emails
```

### Add Dev Script

```json
// package.json
{
  "scripts": {
    "email:dev": "email dev"
  }
}
```

### Start Preview Server

```bash
npm run email:dev
```

Access at `http://localhost:3000` to preview all email templates.

## Basic Email Template

```tsx
// emails/Welcome.tsx
import * as React from 'react'
import {
  Body,
  Container,
  Head,
  Heading,
  Html,
  Preview,
  Text,
} from '@react-email/components'

interface WelcomeEmailProps {
  userName: string
}

export default function WelcomeEmail({ userName }: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Preview>Welcome to Family Time!</Preview>
      <Body style={main}>
        <Container style={container}>
          <Heading style={h1}>Welcome, {userName}!</Heading>
          <Text style={text}>
            Thanks for joining Family Time. We're excited to help you organize
            quality time with your family.
          </Text>
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

const h1 = {
  color: '#333',
  fontSize: '24px',
  fontWeight: 'bold',
  margin: '40px 0',
  padding: '0',
}

const text = {
  color: '#333',
  fontSize: '16px',
  lineHeight: '26px',
}
```

## Core Components

### Html & Head

```tsx
import { Html, Head } from '@react-email/components'

<Html>
  <Head />
  {/* Email content */}
</Html>
```

### Preview Text

```tsx
import { Preview } from '@react-email/components'

<Preview>This text appears in email preview</Preview>
```

### Body & Container

```tsx
import { Body, Container } from '@react-email/components'

<Body style={main}>
  <Container style={container}>
    {/* Content */}
  </Container>
</Body>
```

### Text & Heading

```tsx
import { Text, Heading } from '@react-email/components'

<Heading as="h1">Main Heading</Heading>
<Heading as="h2">Subheading</Heading>
<Text>Paragraph text</Text>
```

### Button & Link

```tsx
import { Button, Link } from '@react-email/components'

<Button href="https://example.com" style={buttonStyle}>
  Click Me
</Button>

<Link href="https://example.com">Visit Website</Link>
```

### Image

```tsx
import { Img } from '@react-email/components'

<Img
  src="https://example.com/logo.png"
  width="200"
  height="50"
  alt="Logo"
/>
```

### Section & Row

```tsx
import { Section, Row, Column } from '@react-email/components'

<Section>
  <Row>
    <Column>Left content</Column>
    <Column>Right content</Column>
  </Row>
</Section>
```

## Advanced Components

### Horizontal Rule

```tsx
import { Hr } from '@react-email/components'

<Hr style={{ borderColor: '#e6e6e6', margin: '20px 0' }} />
```

### Code Block

```tsx
import { Code } from '@react-email/components'

<Code>const greeting = 'Hello World'</Code>
```

## Implementation for Our Project

### Event Reminder Email

```tsx
// emails/EventReminder.tsx
import * as React from 'react'
import {
  Body,
  Button,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Preview,
  Section,
  Text,
} from '@react-email/components'

interface EventReminderProps {
  userName: string
  eventTitle: string
  eventDate: string
  eventTime: string
  familyName: string
}

export default function EventReminder({
  userName,
  eventTitle,
  eventDate,
  eventTime,
  familyName,
}: EventReminderProps) {
  return (
    <Html>
      <Head />
      <Preview>Reminder: {eventTitle} is tomorrow</Preview>
      <Body style={main}>
        <Container style={container}>
          <Heading style={h1}>Event Reminder</Heading>

          <Text style={text}>Hi {userName},</Text>

          <Text style={text}>
            This is a friendly reminder that <strong>{eventTitle}</strong> is
            happening tomorrow!
          </Text>

          <Section style={eventBox}>
            <Text style={eventTitle}>{eventTitle}</Text>
            <Text style={eventDetail}>
              <strong>When:</strong> {eventDate} at {eventTime}
            </Text>
            <Text style={eventDetail}>
              <strong>Family:</strong> {familyName}
            </Text>
          </Section>

          <Button style={button} href="https://familytime.app/dashboard">
            View Event Details
          </Button>

          <Hr style={hr} />

          <Text style={footer}>
            You're receiving this because you're a member of {familyName}.
          </Text>
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
  marginBottom: '64px',
  maxWidth: '600px',
}

const h1 = {
  color: '#333',
  fontSize: '24px',
  fontWeight: 'bold',
  margin: '40px 0 20px',
  padding: '0',
}

const text = {
  color: '#333',
  fontSize: '16px',
  lineHeight: '26px',
}

const eventBox = {
  backgroundColor: '#f9f9f9',
  borderRadius: '8px',
  padding: '20px',
  margin: '20px 0',
}

const eventTitle = {
  fontSize: '20px',
  fontWeight: 'bold',
  color: '#333',
  margin: '0 0 10px',
}

const eventDetail = {
  fontSize: '16px',
  color: '#666',
  margin: '5px 0',
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
  padding: '12px',
}

const hr = {
  borderColor: '#e6e6e6',
  margin: '20px 0',
}

const footer = {
  color: '#8898aa',
  fontSize: '12px',
  lineHeight: '16px',
}
```

### Family Invitation Email

```tsx
// emails/FamilyInvitation.tsx
import * as React from 'react'
import {
  Body,
  Button,
  Container,
  Head,
  Heading,
  Html,
  Img,
  Preview,
  Section,
  Text,
} from '@react-email/components'

interface FamilyInvitationProps {
  inviterName: string
  familyName: string
  invitationLink: string
}

export default function FamilyInvitation({
  inviterName,
  familyName,
  invitationLink,
}: FamilyInvitationProps) {
  return (
    <Html>
      <Head />
      <Preview>{inviterName} invited you to join {familyName}</Preview>
      <Body style={main}>
        <Container style={container}>
          <Section style={header}>
            <Img
              src="https://familytime.app/logo.png"
              width="100"
              height="100"
              alt="Family Time"
            />
          </Section>

          <Heading style={h1}>You're Invited!</Heading>

          <Text style={text}>
            <strong>{inviterName}</strong> has invited you to join{' '}
            <strong>{familyName}</strong> on Family Time.
          </Text>

          <Text style={text}>
            Family Time helps families organize quality time together with
            shared calendars and smart scheduling.
          </Text>

          <Button style={button} href={invitationLink}>
            Accept Invitation
          </Button>

          <Text style={footer}>
            This invitation will expire in 7 days.
          </Text>
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
  marginBottom: '64px',
  maxWidth: '600px',
}

const header = {
  textAlign: 'center' as const,
  padding: '20px 0',
}

const h1 = {
  color: '#333',
  fontSize: '28px',
  fontWeight: 'bold',
  margin: '20px 0',
  padding: '0',
  textAlign: 'center' as const,
}

const text = {
  color: '#333',
  fontSize: '16px',
  lineHeight: '26px',
  textAlign: 'center' as const,
}

const button = {
  backgroundColor: '#5469d4',
  borderRadius: '5px',
  color: '#fff',
  fontSize: '18px',
  fontWeight: 'bold',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'block',
  width: '100%',
  padding: '15px',
  margin: '30px 0',
}

const footer = {
  color: '#8898aa',
  fontSize: '12px',
  lineHeight: '16px',
  textAlign: 'center' as const,
  marginTop: '20px',
}
```

### Weekly Digest Email

```tsx
// emails/WeeklyDigest.tsx
import * as React from 'react'
import {
  Body,
  Button,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Preview,
  Section,
  Text,
} from '@react-email/components'

interface Event {
  title: string
  date: string
  time: string
}

interface WeeklyDigestProps {
  userName: string
  familyName: string
  events: Event[]
}

export default function WeeklyDigest({
  userName,
  familyName,
  events,
}: WeeklyDigestProps) {
  return (
    <Html>
      <Head />
      <Preview>Your week with {familyName}</Preview>
      <Body style={main}>
        <Container style={container}>
          <Heading style={h1}>This Week with {familyName}</Heading>

          <Text style={text}>Hi {userName},</Text>

          <Text style={text}>
            Here's what's coming up for your family this week:
          </Text>

          {events.length > 0 ? (
            <Section>
              {events.map((event, index) => (
                <React.Fragment key={index}>
                  <Section style={eventItem}>
                    <Text style={eventTitle}>{event.title}</Text>
                    <Text style={eventDetail}>
                      {event.date} at {event.time}
                    </Text>
                  </Section>
                  {index < events.length - 1 && <Hr style={eventDivider} />}
                </React.Fragment>
              ))}
            </Section>
          ) : (
            <Text style={noEvents}>No events scheduled this week.</Text>
          )}

          <Button style={button} href="https://familytime.app/dashboard">
            View Calendar
          </Button>
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
  maxWidth: '600px',
}

const h1 = {
  color: '#333',
  fontSize: '24px',
  fontWeight: 'bold',
  margin: '40px 0 20px',
}

const text = {
  color: '#333',
  fontSize: '16px',
  lineHeight: '26px',
}

const eventItem = {
  padding: '15px 0',
}

const eventTitle = {
  fontSize: '18px',
  fontWeight: 'bold',
  color: '#333',
  margin: '0 0 5px',
}

const eventDetail = {
  fontSize: '14px',
  color: '#666',
  margin: '0',
}

const eventDivider = {
  borderColor: '#e6e6e6',
  margin: '0',
}

const noEvents = {
  color: '#666',
  fontSize: '16px',
  textAlign: 'center' as const,
  fontStyle: 'italic',
  margin: '30px 0',
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
  padding: '12px',
  margin: '30px 0 0',
}
```

## Render to HTML

```typescript
import { render } from '@react-email/render'
import WelcomeEmail from '@/emails/Welcome'

const html = render(<WelcomeEmail userName="John" />)
```

## Use with Resend

```typescript
import { Resend } from 'resend'
import EventReminder from '@/emails/EventReminder'

const resend = new Resend(process.env.RESEND_API_KEY)

await resend.emails.send({
  from: 'Family Time <noreply@familytime.app>',
  to: ['user@example.com'],
  subject: 'Event Reminder',
  react: EventReminder({
    userName: 'John',
    eventTitle: 'Family Dinner',
    eventDate: 'Tomorrow',
    eventTime: '6:00 PM',
    familyName: 'Smith Family'
  })
})
```

## Best Practices for Our App

1. **Inline Styles** - Use inline styles for maximum email client compatibility
2. **Mobile-First** - Design for mobile, enhance for desktop
3. **Plain Text Alternative** - Always provide text version
4. **Test Across Clients** - Preview in Gmail, Outlook, Apple Mail
5. **Accessible** - Use semantic HTML and proper alt text
6. **Brand Consistent** - Maintain consistent colors and fonts
7. **CTA Clarity** - Make buttons obvious and actionable

## Common Patterns

### Conditional Content

```tsx
{events.length > 0 ? (
  <EventList events={events} />
) : (
  <Text>No events scheduled</Text>
)}
```

### Dynamic Lists

```tsx
{events.map((event) => (
  <EventItem key={event.id} event={event} />
))}
```

### Reusable Styles

```tsx
const heading = {
  fontSize: '24px',
  fontWeight: 'bold',
  color: '#333',
}

const colors = {
  primary: '#5469d4',
  text: '#333',
  muted: '#666',
}
```

## Resources

- Official Docs: https://react.email/docs
- Components: https://react.email/docs/components
- Examples: https://react.email/examples
- Email Client Support: https://react.email/docs/introduction
