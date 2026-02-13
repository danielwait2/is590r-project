# OpenAI Node.js SDK Documentation

**Library:** openai-node
**Library ID:** `/openai/openai-node`
**Source:** context7 (February 2026)
**Latest Version:** v6.1.0

## Overview

The official OpenAI Node.js SDK provides TypeScript/JavaScript access to the OpenAI REST API, including chat completions and vision capabilities.

## Installation

```bash
npm install openai
```

## Basic Setup

```typescript
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY // defaults to this env var
});
```

## Chat Completions API

### Basic Chat Completion

```typescript
const completion = await client.chat.completions.create({
  model: 'gpt-5.2',
  messages: [
    { role: 'developer', content: 'Talk like a pirate.' },
    { role: 'user', content: 'Are semicolons optional in JavaScript?' },
  ],
});

console.log(completion.choices[0].message.content);
```

### Message Roles

- `developer` - System-level instructions (replaces old `system` role)
- `user` - User messages
- `assistant` - AI responses (for conversation history)

### Streaming Chat Completion

```typescript
const stream = await client.chat.completions.create({
  model: 'gpt-5.2',
  messages: [{ role: 'user', content: 'Write a haiku about recursion' }],
  stream: true,
});

for await (const chunk of stream) {
  process.stdout.write(chunk.choices[0]?.delta?.content || '');
}
```

### Accessing Request ID with Streaming

```typescript
const { data: stream, request_id } = await client.chat.completions
  .create({
    model: 'gpt-5.2',
    messages: [{ role: 'user', content: 'Say this is a test' }],
    stream: true,
  })
  .withResponse();
```

## Function/Tool Calling

```typescript
const toolCompletion = await client.chat.completions.create({
  model: 'gpt-5.2',
  messages: [{ role: 'user', content: 'What is the weather in Boston?' }],
  tools: [{
    type: 'function',
    function: {
      name: 'get_weather',
      description: 'Get current weather for a location',
      parameters: {
        type: 'object',
        properties: {
          location: { type: 'string' }
        },
        required: ['location'],
      },
    },
  }],
});

// Check if model wants to call a function
if (toolCompletion.choices[0].message.tool_calls) {
  // Handle the function call
}
```

## Event Handlers (with stream/runTools)

### Available Events

```typescript
stream
  .on('connect', () => {
    // Connection established
  })
  .on('chunk', (chunk, snapshot) => {
    // Received a chunk during streaming
  })
  .on('chatCompletion', (completion) => {
    // Full completion received
  })
  .on('message', (message) => {
    // New message sent/received
  })
  .on('content', (delta, snapshot) => {
    // Content chunk received (streaming)
  })
  .on('functionCall', (functionCall) => {
    // Assistant requested a function call
  });
```

## CRUD Operations on Stored Completions

```typescript
// Retrieve a stored completion
const retrieved = await client.chat.completions.retrieve('completion_id');

// Update completion metadata
const updated = await client.chat.completions.update('completion_id', {
  metadata: { key: 'value' }
});

// List completions
const list = await client.chat.completions.list({ limit: 10 });

// Delete a completion
const deleted = await client.chat.completions.delete('completion_id');
```

## Vision API Usage

**Note:** The Vision API is accessed through the Chat Completions endpoint by providing images in the messages.

```typescript
// Example structure for vision requests
const visionResponse = await client.chat.completions.create({
  model: 'gpt-5.2',  // or appropriate vision-capable model
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'text',
          text: 'What is in this image?'
        },
        {
          type: 'image_url',
          image_url: {
            url: 'https://example.com/image.jpg'
            // or: url: 'data:image/jpeg;base64,...'
          }
        }
      ]
    }
  ],
});
```

## Key Points for Our Use Case

### For Calendar Event Analysis (Vision API)

1. **Images can be provided as:**
   - URL: `https://...`
   - Base64: `data:image/jpeg;base64,...`
   - We'll likely use base64 for locally generated Sharp images

2. **Multi-modal messages** - Can mix text prompts with image inputs

3. **Streaming supported** - Can stream responses even for vision requests

### For Chat Completions (Event Suggestions)

1. **Model:** Use `gpt-5.2` (latest model as of docs)
2. **Streaming:** Available for real-time responses
3. **Function Calling:** Can structure responses or call external APIs
4. **Request IDs:** Available for debugging via `.withResponse()`

## Implementation Pattern for Our App

```typescript
// 1. Generate calendar image with Sharp
const imageBuffer = await sharp(baseImage)
  .composite([/* text overlays */])
  .toBuffer();

// 2. Convert to base64
const base64Image = `data:image/jpeg;base64,${imageBuffer.toString('base64')}`;

// 3. Send to Vision API
const analysis = await client.chat.completions.create({
  model: 'gpt-5.2',
  messages: [
    {
      role: 'user',
      content: [
        {
          type: 'text',
          text: 'Analyze this calendar and suggest family activities for the free time slots.'
        },
        {
          type: 'image_url',
          image_url: { url: base64Image }
        }
      ]
    }
  ],
});

console.log(analysis.choices[0].message.content);
```

## Error Handling

The SDK throws errors for failed requests. Wrap API calls in try/catch:

```typescript
try {
  const completion = await client.chat.completions.create({...});
} catch (error) {
  if (error instanceof OpenAI.APIError) {
    console.error('API Error:', error.status, error.message);
  }
  throw error;
}
```

## Rate Limiting & Best Practices

- Use streaming for better UX on long responses
- Include request_id in logs for debugging
- Handle errors gracefully
- Consider token limits for vision requests
- Cache responses when appropriate

## Next Steps for Implementation

1. Set up OpenAI client with API key
2. Create vision prompt for calendar analysis
3. Implement streaming response handler
4. Add error handling and retry logic
5. Consider structured outputs or function calling for event data
