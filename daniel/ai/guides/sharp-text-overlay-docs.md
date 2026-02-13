# Sharp - Text Overlay Documentation

**Library:** sharp (Node.js image processing)
**Library ID:** `/lovell/sharp`
**Source:** context7 (February 2026)

## Overview

Sharp is a high-performance Node.js library for image manipulation. For our use case, we need text overlay capabilities to add event details to calendar preview images.

## Text Overlay Approach

Sharp uses **compositing** to add text to images. There are two main approaches:

### 1. Create Text Image → Composite onto Base Image

The recommended approach is to:
1. Create a text image using Sharp's text constructor
2. Composite that text image onto your base image

### 2. Text Rendering Options

```javascript
{
  text: {
    text: "Hello, <b>Sharp</b>!",     // UTF-8 string, supports Pango markup
    font: "Arial",                     // Font name
    fontfile: "/path/to/font.ttf",    // Optional: absolute path to font file
    width: 300,                        // Word-wrap width in pixels
    height: 200,                       // Max height (auto-fits, ignores dpi)
    align: "center",                   // 'left', 'centre', 'center', 'right'
    justify: false,                    // Enable justification
    dpi: 72,                          // Resolution (ignored if height specified)
    rgba: true,                        // Enable RGBA for color/emoji
    spacing: 0,                        // Line height in points
    wrap: "word"                       // 'word', 'char', 'word-char', 'none'
  }
}
```

## Code Examples

### Basic Text Image Generation

```javascript
// Generate a simple text image
await sharp({
  text: {
    text: 'Hello, world!',
    width: 400,
    height: 300
  }
}).toFile('text_bw.png');
```

### Rich Text with Pango Markup

```javascript
// Generate RGBA image with colored text using Pango markup
await sharp({
  text: {
    text: '<span foreground="red">Red!</span><span background="cyan">blue</span>',
    font: 'sans',
    rgba: true,
    dpi: 300
  }
}).toFile('text_rgba.png');
```

### Compositing Text onto an Image

```javascript
// This is the approach we'll likely use for calendar events
await sharp('base-image.jpg')
  .composite([
    {
      input: {
        text: {
          text: 'Event Name\nSaturday 2pm',
          font: 'Arial',
          width: 300,
          rgba: true,
          align: 'center'
        }
      },
      gravity: 'north',    // Position at top
      top: 50,             // Offset from top
      left: 100            // Offset from left
    }
  ])
  .toFile('output.jpg');
```

## Composite Method Details

**Method:** `.composite(images)`

**Parameters:**
- `images` (Array<Object>) - Ordered list of overlays

**Each overlay can have:**
- `input` - Buffer, file path, or text/create object
- `input.text` - Text rendering options (see above)
- `gravity` - Placement: 'north', 'south', 'east', 'west', 'centre', etc.
- `top` - Pixel offset from top
- `left` - Pixel offset from left
- `blend` - Blend mode: 'over', 'multiply', 'source', etc. (default: 'over')
- `tile` - Repeat overlay across image (default: false)
- `premultiplied` - Avoid premultiplying (default: false)

## Key Points for Our Use Case

1. **Pango Markup Supported** - We can use HTML-like tags for styling: `<b>`, `<i>`, `<span foreground="color">`
2. **RGBA Required for Color** - Set `rgba: true` to enable colored text
3. **Word Wrapping** - Specify `width` to auto-wrap long text
4. **Positioning** - Use `gravity` for general placement, `top`/`left` for precise positioning
5. **Custom Fonts** - Can load custom fonts via `fontfile` parameter
6. **Multiple Overlays** - Can composite multiple text layers in one call

## Limitations to Note

- Text must be created as a separate image first (can't directly draw on image)
- Requires libvips with Pango support installed
- Font availability depends on system fonts unless using `fontfile`

## Next Steps for Implementation

For the calendar event overlay, we'll likely:
1. Load base calendar image with Sharp
2. Create text overlay with event details (name, time, location)
3. Composite text onto image with appropriate positioning
4. Output final image for Vision API analysis
