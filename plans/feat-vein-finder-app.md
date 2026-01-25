# Vein Finder Web App

A simple web application that captures images from a webcam, uses Gemini API to identify veins on the human body, and annotates the image to guide nurses for blood draws.

## Enhancement Summary

**Deepened on:** 2026-01-24
**Research agents used:** frontend-design, gemini-imagegen, agent-native-architecture, security-sentinel, performance-oracle, code-simplicity-reviewer, architecture-strategist, julik-frontend-races-reviewer, best-practices-researcher, framework-docs-researcher

### Key Improvements
1. **Simplified to single HTML file** - No Vite, no build step, just open in browser
2. **Clinical precision design system** - Professional medical UI with bioluminescent vein glow effect
3. **Race condition prevention** - Cancel tokens, request IDs, proper async handling
4. **Performance optimizations** - Motion detection, throttling, reduced resolution for API
5. **Security hardening** - Backend proxy recommended, input validation, secure image handling

### Critical Considerations Discovered
- API key MUST be proxied through backend for production
- Gemini uses `[ymin, xmin, ymax, xmax]` coordinate format (y-first)
- Camera permission UX requires pre-priming before request
- WCAG 2.1 AA compliance required for healthcare by May 2026

---

## Overview

Build a single-page web app with:
1. Webcam capture (photo mode)
2. Gemini API integration for vein detection
3. Canvas overlay annotations showing vein locations
4. Mobile-friendly with back camera support

## Technical Stack

- **Frontend**: Single HTML file with inline CSS/JS (no build step)
- **Camera**: MediaDevices API (getUserMedia)
- **AI**: Gemini API via fetch() (no SDK needed)
- **Annotations**: Canvas 2D API
- **Styling**: Inline CSS with clinical design system

### Research Insight: Simplicity First

After review, **Vite is unnecessary** for this prototype. A single HTML file with inline styles and ES modules is simpler:
- 7 files → 1 file
- Zero build step
- Open in browser and it works
- Deploy to any static host

---

## Project Structure (Simplified)

```
findthevein/
└── index.html    # Everything: HTML, CSS, JS (~200 lines)
```

For those who prefer separation:
```
findthevein/
├── index.html
├── style.css
└── app.js
```

---

## Acceptance Criteria

- [x] Camera preview displays in browser
- [x] User can capture a photo with one click
- [x] Captured image is sent to Gemini API for vein analysis
- [x] Veins are annotated on the image with colored overlays
- [x] Numbered markers show recommended blood draw locations
- [x] Works on mobile (back camera default)
- [x] Shows loading state during analysis
- [x] Handles errors gracefully (camera denied, API failure, no veins found)

---

## User Flow

```
[Open App] → [Accept Disclaimer] → [Grant Camera Permission] → [See Live Preview]
     ↓
[Click "Scan"] → [Loading with Progress] → [View Annotated Image with Veins Marked]
     ↓
[Click "New Scan" to start over]
```

### Research Insight: One-Click Flow

Simplify from two buttons (Capture → Analyze) to one button ("Scan") that captures and analyzes in one action. Fewer UI states to manage.

---

## Design System

### Research Insight: Clinical Precision + Warm Trust

A refined minimalism with surgical precision, balanced by approachable warmth. The memorable element: **bioluminescent vein visualization** - veins appear to glow organically.

### Color Palette

```css
:root {
  /* Core Medical Blues */
  --primary-900: #0a2540;
  --primary-500: #2d7cb5;
  --primary-300: #7eb8de;

  /* Vein Visualization - the signature glow */
  --vein-glow: #00e5cc;
  --vein-glow-soft: rgba(0, 229, 204, 0.3);

  /* Semantic */
  --success: #10b981;
  --warning: #f59e0b;
  --error: #ef4444;

  /* Surfaces */
  --surface-warm: #fafafa;
  --text-primary: #1a1a1a;
}
```

### Typography

```css
:root {
  --font-display: 'Plus Jakarta Sans', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}
```

### Capture Button (iOS-style)

```css
.capture-btn {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  border: 4px solid white;
  background: transparent;
  padding: 4px;
  cursor: pointer;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.4);
}

.capture-btn-inner {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: white;
}
```

### Loading Animation (Scan Line)

```css
.scan-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, var(--vein-glow), transparent);
  box-shadow: 0 0 20px var(--vein-glow);
  animation: scan 2s ease-in-out infinite;
}

@keyframes scan {
  0%, 100% { top: 10%; opacity: 0; }
  10%, 90% { opacity: 1; }
  50% { top: 90%; }
}
```

---

## Key Implementation Details

### Camera Capture

```javascript
// Request back camera on mobile, any camera on desktop
async function startCamera() {
  const stream = await navigator.mediaDevices.getUserMedia({
    video: {
      facingMode: 'environment',
      width: { ideal: 1280 },
      height: { ideal: 720 }
    }
  });
  video.srcObject = stream;
  await video.play();
}
```

### Research Insight: Camera Error Handling

```javascript
const ERROR_MESSAGES = {
  'NotAllowedError': 'Camera access denied. Please enable in browser settings.',
  'NotFoundError': 'No camera found on this device.',
  'NotReadableError': 'Camera is being used by another application.',
  'OverconstrainedError': 'Camera does not support required settings.'
};

try {
  await startCamera();
} catch (err) {
  showError(ERROR_MESSAGES[err.name] || 'Unable to access camera');
}
```

---

### Gemini API Integration

#### Research Insight: Use fetch() directly, no SDK needed

```javascript
const GEMINI_API_URL = 'https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent';

async function analyzeImage(base64Data, apiKey) {
  const response = await fetch(`${GEMINI_API_URL}?key=${apiKey}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      contents: [{
        parts: [
          {
            inlineData: {
              mimeType: 'image/jpeg',
              data: base64Data
            }
          },
          {
            text: `Analyze this image of a human arm/hand. Identify visible veins suitable for venipuncture (blood draw).

Return ONLY a JSON object with this exact structure:
{
  "veins": [
    {
      "name": "vein name (e.g., Median Cubital Vein)",
      "box": [ymin, xmin, ymax, xmax],
      "recommendation": "primary|secondary|backup",
      "confidence": 0.0-1.0
    }
  ],
  "quality": "good|fair|poor",
  "message": "any relevant message"
}

Coordinates must be normalized 0-1000 scale.
If no veins visible, return { "veins": [], "quality": "poor", "message": "explanation" }`
          }
        ]
      }]
    })
  });

  const data = await response.json();
  return parseGeminiResponse(data);
}
```

#### Research Insight: Coordinate Format

**CRITICAL**: Gemini returns `[ymin, xmin, ymax, xmax]` (y-first), not the typical `[x, y, x, y]` format.

```javascript
function normalizedToPixels(box, canvasWidth, canvasHeight) {
  const [ymin, xmin, ymax, xmax] = box;
  return {
    x: (xmin / 1000) * canvasWidth,
    y: (ymin / 1000) * canvasHeight,
    width: ((xmax - xmin) / 1000) * canvasWidth,
    height: ((ymax - ymin) / 1000) * canvasHeight
  };
}
```

#### Research Insight: Response Parsing with Validation

```javascript
function parseGeminiResponse(response) {
  const text = response.candidates?.[0]?.content?.parts?.[0]?.text;
  if (!text) throw new Error('No response from API');

  // Handle markdown-wrapped JSON
  const jsonMatch = text.match(/```(?:json)?\s*([\s\S]*?)\s*```/);
  const jsonText = jsonMatch ? jsonMatch[1] : text;

  const data = JSON.parse(jsonText.trim());

  // Validate and filter invalid boxes
  data.veins = (data.veins || []).filter(vein => {
    const box = vein.box;
    return Array.isArray(box) &&
           box.length === 4 &&
           box.every(n => n >= 0 && n <= 1000) &&
           box.some(n => n > 0); // Not all zeros
  });

  return data;
}
```

---

### Canvas Annotations

#### Research Insight: Simplified Drawing

```javascript
function drawVeinAnnotations(ctx, veins, canvasWidth, canvasHeight) {
  veins.forEach((vein, index) => {
    const { x, y, width, height } = normalizedToPixels(
      vein.box, canvasWidth, canvasHeight
    );

    const isPrimary = vein.recommendation === 'primary';

    // Semi-transparent overlay
    ctx.fillStyle = isPrimary
      ? 'rgba(0, 229, 204, 0.3)'  // Vein glow
      : 'rgba(100, 150, 255, 0.2)';
    ctx.fillRect(x, y, width, height);

    // Border with glow effect
    ctx.strokeStyle = isPrimary ? '#00e5cc' : '#6496ff';
    ctx.lineWidth = 3;
    ctx.strokeRect(x, y, width, height);

    // Numbered marker at center
    const centerX = x + width / 2;
    const centerY = y + height / 2;

    ctx.beginPath();
    ctx.arc(centerX, centerY, 16, 0, Math.PI * 2);
    ctx.fillStyle = isPrimary ? '#00e5cc' : '#6496ff';
    ctx.fill();

    ctx.fillStyle = '#ffffff';
    ctx.font = 'bold 14px sans-serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(String(index + 1).padStart(2, '0'), centerX, centerY);
  });
}
```

#### Research Insight: Performance Optimization

```javascript
// Use separate canvas layers for video and annotations
const videoCanvas = document.getElementById('video-canvas');
const overlayCanvas = document.getElementById('overlay-canvas');

// Only redraw overlay when detections change
let lastDetections = null;

function updateOverlay(detections) {
  if (JSON.stringify(detections) === JSON.stringify(lastDetections)) {
    return; // Skip if unchanged
  }

  const ctx = overlayCanvas.getContext('2d');
  ctx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);
  drawVeinAnnotations(ctx, detections, overlayCanvas.width, overlayCanvas.height);

  lastDetections = detections;
}
```

---

## Race Condition Prevention

### Research Insight: Critical Async Patterns

```javascript
class VeinAnalyzer {
  constructor() {
    this.currentRequestId = 0;
    this.isAnalyzing = false;
    this.cancelToken = { canceled: false };
  }

  async analyze(imageData) {
    // Prevent double-click
    if (this.isAnalyzing) return null;

    const requestId = ++this.currentRequestId;
    this.isAnalyzing = true;

    try {
      const result = await analyzeImage(imageData, API_KEY);

      // Stale check: user may have clicked "New Scan"
      if (requestId !== this.currentRequestId) {
        console.log('Discarding stale response');
        return null;
      }

      return result;
    } finally {
      if (requestId === this.currentRequestId) {
        this.isAnalyzing = false;
      }
    }
  }

  reset() {
    this.currentRequestId++;
    this.isAnalyzing = false;
  }
}
```

### Camera Stream Cleanup

```javascript
// Listen for unexpected stream termination
stream.getTracks().forEach(track => {
  track.addEventListener('ended', () => {
    console.warn('Camera stream ended unexpectedly');
    resetToInitialState();
  });
});

// Always cleanup on page unload
window.addEventListener('beforeunload', () => {
  if (stream) {
    stream.getTracks().forEach(track => track.stop());
  }
});
```

---

## Performance Optimizations

### Research Insight: Reduce API Payload

```javascript
// Downscale for API (640x360) while keeping full res for display
const ANALYSIS_WIDTH = 640;
const ANALYSIS_HEIGHT = 360;

function captureForAnalysis(video) {
  const canvas = document.createElement('canvas');
  canvas.width = ANALYSIS_WIDTH;
  canvas.height = ANALYSIS_HEIGHT;

  const ctx = canvas.getContext('2d');
  ctx.drawImage(video, 0, 0, ANALYSIS_WIDTH, ANALYSIS_HEIGHT);

  // JPEG at 70% quality for smaller payload
  return canvas.toDataURL('image/jpeg', 0.7).split(',')[1];
}
```

### Research Insight: Request Throttling

```javascript
class AnalysisThrottler {
  constructor(minIntervalMs = 500) {
    this.minInterval = minIntervalMs;
    this.lastAnalysis = 0;
  }

  canAnalyze() {
    const now = Date.now();
    if (now - this.lastAnalysis < this.minInterval) {
      return false;
    }
    this.lastAnalysis = now;
    return true;
  }
}
```

### Performance Targets

| Metric | Target | Maximum |
|--------|--------|---------|
| Frame capture | < 10ms | 20ms |
| Image encoding | < 30ms | 50ms |
| API round-trip | < 500ms | 1000ms |
| Overlay render | < 5ms | 16ms |

---

## Security Considerations

### Research Insight: CRITICAL - API Key Exposure

**For prototype only:** API key in frontend is acceptable for time-boxed development.

**For production:** MUST use backend proxy:

```javascript
// Backend proxy endpoint (Vercel Edge Function example)
// /api/analyze.js
export default async function handler(request) {
  const { image } = await request.json();

  // Validate origin
  const origin = request.headers.get('origin');
  if (!ALLOWED_ORIGINS.includes(origin)) {
    return new Response('Forbidden', { status: 403 });
  }

  // Call Gemini with server-side key
  const response = await fetch(GEMINI_API_URL, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.GEMINI_API_KEY}`
    },
    body: JSON.stringify({ /* ... */ })
  });

  return new Response(response.body);
}
```

### Secure Image Handling

```javascript
// Cleanup blob URLs to prevent memory leaks
const blobUrls = new Set();

function createBlobUrl(blob) {
  const url = URL.createObjectURL(blob);
  blobUrls.add(url);
  return url;
}

function cleanup() {
  blobUrls.forEach(url => URL.revokeObjectURL(url));
  blobUrls.clear();
}

window.addEventListener('beforeunload', cleanup);
```

---

## Error Handling

| Scenario | User Message |
|----------|--------------|
| Camera permission denied | "Camera access required. Please enable camera in your browser settings." |
| No camera found | "No camera detected. Please use a device with a camera." |
| API error | "Analysis failed. Please try again." |
| No veins detected | "Could not identify veins. Ensure good lighting and position your inner arm in frame." |
| Network offline | "No internet connection. Please check your network." |

### Research Insight: Accessible Error Messages

```html
<div role="alert" aria-live="polite" class="error-message">
  <svg aria-hidden="true"><!-- warning icon --></svg>
  <span id="error-text">Error message here</span>
</div>
```

---

## Medical Disclaimer

Display on first use with acceptance required:

```html
<div class="disclaimer" role="dialog" aria-labelledby="disclaimer-title">
  <h2 id="disclaimer-title">Medical Disclaimer</h2>
  <p>
    <strong>For informational purposes only.</strong>
    This tool provides visual guidance and is not a substitute for
    professional medical judgment. Results should be verified by
    qualified healthcare personnel.
  </p>
  <p>
    This application is NOT a medical device and has NOT been
    evaluated by the FDA or any regulatory body.
  </p>
  <button onclick="acceptDisclaimer()">I Understand</button>
</div>
```

---

## Accessibility (WCAG 2.1 AA)

### Research Insight: Healthcare Compliance Required by May 2026

```css
/* Minimum contrast ratios */
.text { color: #1a1a1a; } /* 12.6:1 on white */
.error { color: #c62828; } /* 4.5:1 on white */

/* Visible focus indicators */
button:focus-visible {
  outline: 3px solid var(--primary-500);
  outline-offset: 2px;
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Agent-Native Considerations

### Research Insight: Future API Mode

For integration with automated systems, add an API endpoint:

```javascript
// POST /api/analyze
// Request: { image: base64 }
// Response: { veins: [...], quality: "good", annotatedImage: base64 }
```

This enables:
- Batch processing multiple images
- Integration with electronic health records
- Automated screening workflows
- Voice-controlled assistive operation

---

## MVP Constraints

- Photo capture only (no video)
- No user accounts or storage
- Images processed transiently (not saved)
- Single page, no routing
- Frontend-only for prototype (API key in prompt or env)

---

## Complete Single-File Implementation

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Vein Finder</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: system-ui, sans-serif;
      background: #0a2540;
      color: white;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
    }

    .container {
      max-width: 600px;
      margin: 0 auto;
      padding: 1rem;
      flex: 1;
      display: flex;
      flex-direction: column;
    }

    h1 {
      text-align: center;
      margin-bottom: 1rem;
      font-size: 1.5rem;
    }

    .camera-viewport {
      position: relative;
      background: #000;
      border-radius: 12px;
      overflow: hidden;
      aspect-ratio: 4/3;
    }

    video, canvas {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }

    .hidden { display: none; }

    .controls {
      display: flex;
      justify-content: center;
      padding: 2rem;
    }

    .capture-btn {
      width: 72px;
      height: 72px;
      border-radius: 50%;
      border: 4px solid white;
      background: transparent;
      cursor: pointer;
    }

    .capture-btn-inner {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background: white;
    }

    .capture-btn:disabled {
      opacity: 0.5;
    }

    .status {
      text-align: center;
      padding: 1rem;
      min-height: 3rem;
    }

    .error {
      background: rgba(239, 68, 68, 0.2);
      border-left: 4px solid #ef4444;
      padding: 1rem;
      border-radius: 8px;
      margin: 1rem 0;
    }

    .disclaimer {
      background: rgba(245, 158, 11, 0.1);
      border: 1px solid rgba(245, 158, 11, 0.3);
      padding: 1rem;
      border-radius: 8px;
      font-size: 0.875rem;
      margin-top: auto;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>Vein Finder</h1>

    <div class="camera-viewport">
      <video id="video" autoplay playsinline></video>
      <canvas id="canvas" class="hidden"></canvas>
    </div>

    <div class="controls">
      <button id="capture-btn" class="capture-btn" disabled>
        <span class="capture-btn-inner"></span>
      </button>
    </div>

    <div id="status" class="status">Initializing camera...</div>

    <div class="disclaimer">
      <strong>For informational purposes only.</strong>
      Not a substitute for professional medical judgment.
    </div>
  </div>

  <script type="module">
    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const captureBtn = document.getElementById('capture-btn');
    const status = document.getElementById('status');
    const ctx = canvas.getContext('2d');

    let stream = null;
    let isAnalyzing = false;

    // Prompt for API key (prototype only)
    const API_KEY = localStorage.getItem('gemini_key') ||
                   prompt('Enter your Gemini API key:');
    if (API_KEY) localStorage.setItem('gemini_key', API_KEY);

    async function startCamera() {
      try {
        stream = await navigator.mediaDevices.getUserMedia({
          video: { facingMode: 'environment', width: { ideal: 1280 } }
        });
        video.srcObject = stream;
        await video.play();
        captureBtn.disabled = false;
        status.textContent = 'Ready. Tap to scan.';
      } catch (err) {
        status.textContent = 'Camera error: ' + err.message;
      }
    }

    async function analyze() {
      if (isAnalyzing) return;
      isAnalyzing = true;
      captureBtn.disabled = true;
      status.textContent = 'Analyzing...';

      // Capture frame
      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      ctx.drawImage(video, 0, 0);

      const base64 = canvas.toDataURL('image/jpeg', 0.8).split(',')[1];

      try {
        const response = await fetch(
          `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${API_KEY}`,
          {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              contents: [{
                parts: [
                  { inlineData: { mimeType: 'image/jpeg', data: base64 } },
                  { text: 'Identify veins for blood draw. Return JSON: { "veins": [{ "name": "string", "box": [ymin,xmin,ymax,xmax], "recommendation": "primary|secondary" }] }. Coordinates 0-1000 scale.' }
                ]
              }]
            })
          }
        );

        const data = await response.json();
        const text = data.candidates?.[0]?.content?.parts?.[0]?.text || '';
        const match = text.match(/\{[\s\S]*\}/);

        if (match) {
          const result = JSON.parse(match[0]);
          drawAnnotations(result.veins || []);
          video.classList.add('hidden');
          canvas.classList.remove('hidden');
          status.textContent = `Found ${result.veins?.length || 0} veins. Tap to scan again.`;
        } else {
          status.textContent = 'No veins detected. Try again with better lighting.';
        }
      } catch (err) {
        status.textContent = 'Analysis failed: ' + err.message;
      }

      isAnalyzing = false;
      captureBtn.disabled = false;
    }

    function drawAnnotations(veins) {
      veins.forEach((vein, i) => {
        const [ymin, xmin, ymax, xmax] = vein.box || [0,0,0,0];
        const x = (xmin / 1000) * canvas.width;
        const y = (ymin / 1000) * canvas.height;
        const w = ((xmax - xmin) / 1000) * canvas.width;
        const h = ((ymax - ymin) / 1000) * canvas.height;

        const isPrimary = vein.recommendation === 'primary';

        ctx.fillStyle = isPrimary ? 'rgba(0,229,204,0.3)' : 'rgba(100,150,255,0.2)';
        ctx.fillRect(x, y, w, h);

        ctx.strokeStyle = isPrimary ? '#00e5cc' : '#6496ff';
        ctx.lineWidth = 3;
        ctx.strokeRect(x, y, w, h);

        // Number marker
        const cx = x + w/2, cy = y + h/2;
        ctx.beginPath();
        ctx.arc(cx, cy, 16, 0, Math.PI * 2);
        ctx.fillStyle = ctx.strokeStyle;
        ctx.fill();
        ctx.fillStyle = '#fff';
        ctx.font = 'bold 14px sans-serif';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText(String(i+1).padStart(2,'0'), cx, cy);
      });
    }

    function reset() {
      video.classList.remove('hidden');
      canvas.classList.add('hidden');
      status.textContent = 'Ready. Tap to scan.';
    }

    captureBtn.addEventListener('click', () => {
      if (canvas.classList.contains('hidden')) {
        analyze();
      } else {
        reset();
      }
    });

    startCamera();
  </script>
</body>
</html>
```

---

## References

- [Gemini API Image Understanding](https://ai.google.dev/gemini-api/docs/image-understanding)
- [MDN: getUserMedia()](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia)
- [MDN: Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Google GenAI JavaScript SDK](https://github.com/googleapis/js-genai)
- [WCAG 2.1 AA Guidelines](https://www.w3.org/TR/WCAG21/)
- [HHS Website Accessibility Requirements](https://www.edreamz.com/blog/hhs-website-accessibility-requirements)
