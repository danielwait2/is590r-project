# BullMQ Job Queue Documentation

**Library:** BullMQ
**Library ID:** `/taskforcesh/bullmq`
**Source:** context7 (February 2026)
**Version:** Latest

## Overview

BullMQ is a Node.js library that implements a fast and robust queue system built on top of Redis. It provides job scheduling, background processing, cron jobs, and priority queues for handling asynchronous tasks.

## Key Features for Our Project

- **Job Scheduling** - Schedule jobs for future execution
- **Cron Jobs** - Recurring tasks with cron patterns
- **Background Processing** - Process tasks asynchronously
- **Redis-Backed** - Fast, reliable, and scalable
- **Job Prioritization** - Handle urgent tasks first

## Installation

```bash
npm install bullmq ioredis
```

## Setup

### Redis Connection

```typescript
// lib/redis.ts
import { Redis } from 'ioredis'

const connection = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  maxRetriesPerRequest: null
})

export default connection
```

### Environment Variables

```bash
# .env.local
REDIS_HOST=localhost
REDIS_PORT=6379

# For production (Upstash Redis)
REDIS_URL=rediss://default:password@hostname:port
```

## Basic Queue Usage

### Create a Queue

```typescript
// lib/queues/email-queue.ts
import { Queue } from 'bullmq'
import connection from '@/lib/redis'

export const emailQueue = new Queue('emails', { connection })
```

### Add Job to Queue

```typescript
import { emailQueue } from '@/lib/queues/email-queue'

async function sendEmail(to: string, subject: string, body: string) {
  await emailQueue.add('send-email', {
    to,
    subject,
    body
  })
}
```

### Create Worker to Process Jobs

```typescript
// workers/email-worker.ts
import { Worker } from 'bullmq'
import connection from '@/lib/redis'
import { sendEmailWithResend } from '@/lib/email'

const emailWorker = new Worker(
  'emails',
  async (job) => {
    const { to, subject, body } = job.data

    console.log(`Processing email job ${job.id} to ${to}`)

    await sendEmailWithResend(to, subject, body)

    return { success: true }
  },
  { connection }
)

emailWorker.on('completed', (job) => {
  console.log(`Job ${job.id} completed`)
})

emailWorker.on('failed', (job, err) => {
  console.error(`Job ${job?.id} failed:`, err)
})
```

## Job Scheduling

### Schedule Job for Future

```typescript
import { emailQueue } from '@/lib/queues/email-queue'

async function scheduleEmail(to: string, subject: string, body: string, sendAt: Date) {
  const delay = sendAt.getTime() - Date.now()

  await emailQueue.add(
    'send-email',
    { to, subject, body },
    { delay } // Delay in milliseconds
  )
}
```

### Schedule with Specific Timestamp

```typescript
const futureDate = new Date('2026-02-15T10:00:00Z')

await emailQueue.add(
  'send-reminder',
  { eventId: 'event_123' },
  {
    delay: futureDate.getTime() - Date.now()
  }
)
```

## Cron Jobs

### Create Repeatable Job

```typescript
import { Queue } from 'bullmq'
import connection from '@/lib/redis'

const reminderQueue = new Queue('reminders', { connection })

// Every day at 9 AM
await reminderQueue.add(
  'daily-reminder',
  { type: 'morning' },
  {
    repeat: {
      pattern: '0 9 * * *', // Cron pattern
      tz: 'America/New_York'
    }
  }
)
```

### Common Cron Patterns

```typescript
// Every minute
{ pattern: '* * * * *' }

// Every hour
{ pattern: '0 * * * *' }

// Every day at 9 AM
{ pattern: '0 9 * * *' }

// Every Monday at 9 AM
{ pattern: '0 9 * * 1' }

// Every 1st of the month at midnight
{ pattern: '0 0 1 * *' }

// Every 15 minutes
{ pattern: '*/15 * * * *' }
```

### Remove Repeatable Job

```typescript
await reminderQueue.removeRepeatable('daily-reminder', {
  pattern: '0 9 * * *'
})
```

## Advanced Features

### Job Priority

```typescript
// Higher priority (lower number = higher priority)
await emailQueue.add(
  'urgent-email',
  { to: 'user@example.com' },
  { priority: 1 }
)

// Normal priority
await emailQueue.add(
  'normal-email',
  { to: 'user@example.com' },
  { priority: 10 }
)
```

### Job Retry Configuration

```typescript
await emailQueue.add(
  'send-email',
  { to: 'user@example.com' },
  {
    attempts: 3, // Retry up to 3 times
    backoff: {
      type: 'exponential',
      delay: 2000 // Start with 2 second delay
    }
  }
)
```

### Job Timeout

```typescript
const worker = new Worker(
  'emails',
  async (job) => {
    // Job processing
  },
  {
    connection,
    lockDuration: 30000, // 30 seconds
    timeout: 60000 // 1 minute
  }
)
```

### Job Progress Tracking

```typescript
const worker = new Worker(
  'long-task',
  async (job) => {
    await job.updateProgress(0)

    // Do some work
    await doWork1()
    await job.updateProgress(33)

    // More work
    await doWork2()
    await job.updateProgress(66)

    // Final work
    await doWork3()
    await job.updateProgress(100)

    return { completed: true }
  },
  { connection }
)
```

## Implementation for Our Project

### Event Reminder System

```typescript
// lib/queues/reminder-queue.ts
import { Queue } from 'bullmq'
import connection from '@/lib/redis'

export const reminderQueue = new Queue('event-reminders', { connection })

export async function scheduleEventReminder(
  eventId: string,
  eventTitle: string,
  eventDate: Date,
  recipientEmails: string[]
) {
  const reminderDate = new Date(eventDate)
  reminderDate.setHours(reminderDate.getHours() - 24) // 24 hours before

  const delay = reminderDate.getTime() - Date.now()

  if (delay > 0) {
    await reminderQueue.add(
      'send-event-reminder',
      {
        eventId,
        eventTitle,
        eventDate: eventDate.toISOString(),
        recipientEmails
      },
      {
        delay,
        jobId: `reminder-${eventId}`, // Prevent duplicates
        removeOnComplete: true,
        removeOnFail: false
      }
    )
  }
}
```

### Event Reminder Worker

```typescript
// workers/reminder-worker.ts
import { Worker } from 'bullmq'
import connection from '@/lib/redis'
import { sendEventReminder } from '@/lib/email'
import { prisma } from '@/lib/prisma'

const reminderWorker = new Worker(
  'event-reminders',
  async (job) => {
    const { eventId, eventTitle, eventDate, recipientEmails } = job.data

    console.log(`Processing reminder for event ${eventId}`)

    // Verify event still exists and hasn't been cancelled
    const event = await prisma.event.findUnique({
      where: { id: eventId },
      include: { family: true }
    })

    if (!event) {
      console.log(`Event ${eventId} not found, skipping reminder`)
      return { skipped: true }
    }

    // Send reminders to all recipients
    const results = await Promise.all(
      recipientEmails.map(email =>
        sendEventReminder(email, eventTitle, new Date(eventDate), event.family.name)
      )
    )

    // Log reminder sent
    await prisma.event.update({
      where: { id: eventId },
      data: {
        // Could add a reminderSent field to track this
      }
    })

    return { success: true, emailsSent: results.length }
  },
  {
    connection,
    limiter: {
      max: 10, // Max 10 jobs per interval
      duration: 1000 // 1 second
    }
  }
)

reminderWorker.on('completed', (job) => {
  console.log(`Reminder job ${job.id} completed:`, job.returnvalue)
})

reminderWorker.on('failed', (job, err) => {
  console.error(`Reminder job ${job?.id} failed:`, err)
})

export default reminderWorker
```

### Daily Digest Job

```typescript
// lib/queues/digest-queue.ts
import { Queue } from 'bullmq'
import connection from '@/lib/redis'

export const digestQueue = new Queue('daily-digest', { connection })

// Schedule daily digest at 8 AM
export async function scheduleDailyDigest() {
  await digestQueue.add(
    'send-daily-digest',
    {},
    {
      repeat: {
        pattern: '0 8 * * *', // Every day at 8 AM
        tz: 'America/New_York'
      }
    }
  )
}
```

### Daily Digest Worker

```typescript
// workers/digest-worker.ts
import { Worker } from 'bullmq'
import connection from '@/lib/redis'
import { prisma } from '@/lib/prisma'
import { sendDailyDigest } from '@/lib/email'

const digestWorker = new Worker(
  'daily-digest',
  async (job) => {
    console.log('Processing daily digest')

    // Get all families
    const families = await prisma.family.findMany({
      include: {
        members: {
          include: {
            user: true
          }
        },
        events: {
          where: {
            startTime: {
              gte: new Date(),
              lte: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // Next 7 days
            }
          },
          orderBy: {
            startTime: 'asc'
          }
        }
      }
    })

    // Send digest to each family member
    const emailPromises = families.flatMap(family =>
      family.members.map(member =>
        sendDailyDigest(
          member.user.email,
          family.name,
          family.events
        )
      )
    )

    await Promise.all(emailPromises)

    return {
      success: true,
      familiesProcessed: families.length,
      emailsSent: emailPromises.length
    }
  },
  { connection }
)

export default digestWorker
```

### API Route to Trigger Jobs

```typescript
// app/api/events/schedule-reminder/route.ts
import { auth } from '@clerk/nextjs'
import { NextResponse } from 'next/server'
import { scheduleEventReminder } from '@/lib/queues/reminder-queue'
import { prisma } from '@/lib/prisma'

export async function POST(request: Request) {
  const { userId } = auth()

  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { eventId } = await request.json()

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

  const recipientEmails = event.family.members.map(m => m.user.email)

  await scheduleEventReminder(
    event.id,
    event.title,
    event.startTime,
    recipientEmails
  )

  return NextResponse.json({ success: true })
}
```

## Queue Monitoring

### Check Queue Status

```typescript
import { emailQueue } from '@/lib/queues/email-queue'

const jobCounts = await emailQueue.getJobCounts('wait', 'active', 'completed', 'failed')
console.log(jobCounts)
// { wait: 5, active: 2, completed: 100, failed: 3 }
```

### Get All Jobs

```typescript
const waitingJobs = await emailQueue.getJobs('waiting')
const failedJobs = await emailQueue.getJobs('failed')
```

### Clean Old Jobs

```typescript
// Remove completed jobs older than 1 hour
await emailQueue.clean(3600 * 1000, 0, 'completed')

// Remove failed jobs older than 1 day
await emailQueue.clean(24 * 3600 * 1000, 0, 'failed')
```

## Best Practices for Our App

1. **Single Redis Connection** - Reuse connection across queues and workers
2. **Job IDs** - Use unique job IDs to prevent duplicate scheduling
3. **Error Handling** - Always handle worker errors and log failures
4. **Cleanup** - Remove completed jobs to prevent Redis bloat
5. **Retries** - Configure retries with exponential backoff
6. **Monitoring** - Track job counts and failures
7. **Graceful Shutdown** - Close workers properly on app shutdown

## Graceful Shutdown

```typescript
// workers/index.ts
import reminderWorker from './reminder-worker'
import digestWorker from './digest-worker'

const workers = [reminderWorker, digestWorker]

async function gracefulShutdown() {
  console.log('Shutting down workers...')

  await Promise.all(workers.map(worker => worker.close()))

  console.log('Workers closed')
  process.exit(0)
}

process.on('SIGTERM', gracefulShutdown)
process.on('SIGINT', gracefulShutdown)
```

## Resources

- Official Docs: https://docs.bullmq.io
- API Reference: https://api.docs.bullmq.io
- Redis Setup: https://redis.io/docs/getting-started
- Upstash Redis: https://upstash.com/docs/redis
- Cron Patterns: https://crontab.guru
