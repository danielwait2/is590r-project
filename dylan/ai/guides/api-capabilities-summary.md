# API Capabilities Summary - Real Documentation Verified

**Created:** February 12, 2026  
**Purpose:** Verified capabilities of Sharp and OpenAI for our MVP

---

## ✅ Confirmed: We Can Build This

Both libraries have the capabilities we need. Here's what's real and what we can do:

---

## Sharp (Image Processing)

### Text Overlay - CONFIRMED ✅

**Method:** `composite()` with `text` input

```javascript
await sharp('background.jpg')
  .composite([
    {
      input: {
        text: {
          text: 'Kayaking at Lady Bird Lake',
          font: 'Arial',
          width: 1000,
          height: 200,
          align: 'center',
          rgba: true,
          dpi: 150
        }
      },
      gravity: 'center'
    }
  ])
  .toFile('output.jpg');
```

**What works:**
- ✅ Add text to any image
- ✅ Custom fonts (via `fontfile` parameter)
- ✅ Position anywhere (gravity or exact pixels)
- ✅ Multiple text layers
- ✅ Emoji support (via `rgba: true`)
- ✅ Pango markup for styling (`<b>bold</b>`, `<i>italic</i>`)
- ✅ Word wrapping
- ✅ Background colors/transparency

**Performance:**
- 4-5x faster than ImageMagick
- Non-blocking (async/await)
- Memory efficient

---

## OpenAI SDK (Vision + Chat)

### Vision API - CONFIRMED ✅

**Method:** Chat Completions with image inputs

```typescript
const response = await client.chat.completions.create({
  model: 'gpt-4.1-mini',
  messages: [
    {
      role: 'user',
      content: [
        { type: 'text', text: "What's in this image?" },
        {
          type: 'image_url',
          image_url: {
            url: imageUrl,
            detail: 'low'  // or 'high'
          }
        }
      ]
    }
  ]
});
```

**What works:**
- ✅ Analyze images (URL, Base64, or File ID)
- ✅ Multiple images per request (up to 500)
- ✅ Text extraction from images
- ✅ Object/scene recognition
- ✅ Up to 50MB per request
- ✅ Cost-effective with `detail: 'low'` option

**Cost (gpt-4.1-mini):**
- 1024×1024 image = ~1659 tokens
- At $0.40/1M tokens = ~$0.0007 per image
- Very affordable for our use case

### Chat Completions - CONFIRMED ✅

**Method:** Standard text generation

```typescript
const completion = await client.chat.completions.create({
  model: 'gpt-4.1-mini',
  messages: [
    {
      role: 'developer',
      content: 'Suggest family activities'
    },
    {
      role: 'user',
      content: 'What should we do in Austin Saturday?'
    }
  ]
});
```

**What works:**
- ✅ Text generation
- ✅ Streaming responses
- ✅ System instructions (via 'developer' role)
- ✅ Built-in retry logic
- ✅ Full TypeScript support

---

## What We Can Build (Verified)

### Use Case 1: Generate Calendar Event Images

**Flow:**
1. Take activity details (text)
2. Use Sharp to create image with text overlay
3. Add to calendar as image

**Example:**
```javascript
// Create 1200x630 image for calendar preview
await sharp({
  create: {
    width: 1200,
    height: 630,
    channels: 4,
    background: { r: 59, g: 130, b: 246, alpha: 1 }
  }
})
.composite([
  {
    input: {
      text: {
        text: '<b>Kayaking at Lady Bird Lake</b>',
        font: 'Arial',
        width: 1000,
        align: 'center',
        rgba: true
      }
    },
    top: 200
  },
  {
    input: {
      text: {
        text: 'Saturday, Feb 15 • 2:00 PM - 5:00 PM',
        font: 'Arial',
        width: 1000,
        align: 'center'
      }
    },
    top: 350
  }
])
.toBuffer();
```

**Confirmed capabilities:**
- ✅ Create from scratch or use background image
- ✅ Multiple text layers (title, date, details)
- ✅ Custom colors and fonts
- ✅ Output as Buffer (for Google Calendar API)

---

### Use Case 2: Analyze Scraped Activity Images

**Flow:**
1. Scrape activity image from website
2. Send to OpenAI Vision API
3. Extract details (indoor/outdoor, family-friendly, etc.)

**Example:**
```typescript
// Analyze venue image
const analysis = await client.chat.completions.create({
  model: 'gpt-4.1-mini',
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'text',
          text: 'Is this venue family-friendly? Indoor or outdoor? Describe briefly.'
        },
        {
          type: 'image_url',
          image_url: {
            url: scrapedImageUrl,
            detail: 'low'  // Save tokens
          }
        }
      ]
    }
  ],
  max_tokens: 200
});
```

**Confirmed capabilities:**
- ✅ Understand venue type
- ✅ Detect indoor/outdoor
- ✅ Identify activities
- ✅ Read signage/text in images
- ✅ Assess family-friendliness

---

### Use Case 3: Generate Activity Suggestions

**Flow:**
1. User provides preferences
2. Send to OpenAI Chat API
3. Get personalized suggestions

**Example:**
```typescript
const suggestion = await client.chat.completions.create({
  model: 'gpt-4.1-mini',
  messages: [
    {
      role: 'developer',
      content: 'You suggest specific family activities with actionable details.'
    },
    {
      role: 'user',
      content: `User wants: outdoor activity
Location: Austin, TX
Time: Saturday afternoon
Kids: ages 4 and 7
Suggest a specific activity with location.`
    }
  ],
  max_tokens: 300
});
```

**Confirmed capabilities:**
- ✅ Context-aware suggestions
- ✅ Specific locations
- ✅ Age-appropriate filtering
- ✅ Actionable details
- ✅ Natural language output

---

## Cost Estimates (Real Numbers)

### Sharp
- **Free** (open source library)
- CPU cost only (negligible on modern servers)

### OpenAI (gpt-4.1-mini)

**Vision API:**
- 1024×1024 image = 1659 tokens
- Input: $0.40/1M tokens
- Cost per image: ~$0.0007

**Chat Completions:**
- Activity suggestion ~300 tokens
- Input: $0.40/1M tokens
- Output: $1.60/1M tokens
- Cost per suggestion: ~$0.0006

**Monthly estimate (100 users):**
- 100 users × 8 suggestions/month = 800 suggestions
- 800 × $0.0006 = $0.48/month
- Add vision: 800 images × $0.0007 = $0.56/month
- **Total: ~$1/month** for OpenAI

---

## Tech Stack Compatibility

### Sharp + Next.js ✅
```javascript
// Works in Next.js API routes
import sharp from 'sharp';

export async function POST(request) {
  const buffer = await sharp('input.jpg')
    .composite([...])
    .toBuffer();
  
  return new Response(buffer, {
    headers: { 'Content-Type': 'image/jpeg' }
  });
}
```

### OpenAI + Next.js ✅
```typescript
// Works in Next.js API routes
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export async function POST(request) {
  const completion = await client.chat.completions.create({...});
  return Response.json(completion);
}
```

### Both work with:
- ✅ Next.js 14+
- ✅ Node.js 20+
- ✅ TypeScript
- ✅ Vercel serverless functions
- ✅ Async/await

---

## Limitations (Real Ones)

### Sharp
- Text rendering uses Pango (system font required)
- Advanced typography limited
- No native PDF generation

### OpenAI Vision
- ❌ Medical images not supported
- ⚠️ Small text may need enlarging
- ⚠️ Non-English text less accurate
- ⚠️ Approximate counting only
- 50MB max payload size

---

## What We DON'T Need to Worry About

### Hallucinated concerns dispelled:
- ❌ Sharp CAN do text overlay (confirmed)
- ❌ Vision API DOES work with URLs (confirmed)
- ❌ Cost is NOT prohibitive (~$1/month for 100 users)
- ❌ TypeScript support EXISTS for both (confirmed)
- ❌ Performance is GOOD (Sharp is 4-5x faster than alternatives)

---

## Next Steps

1. ✅ APIs confirmed - ready to plan implementation
2. ✅ Cost is affordable - no budget concerns
3. ✅ TypeScript compatible - matches our stack
4. ✅ Next.js compatible - works in API routes
5. ✅ Performance adequate - Sharp is fast, OpenAI has caching

**Ready to proceed with implementation planning using these verified capabilities.**

---

## Reference Documentation

- Sharp API: `ai/guides/sharp-api-reference.md`
- OpenAI SDK: `ai/guides/openai-sdk-reference.md`
- Official Sharp: https://sharp.pixelplumbing.com/
- Official OpenAI: https://platform.openai.com/docs
