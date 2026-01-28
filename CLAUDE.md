# Vein Finder - Project Documentation

## What We Built

A single-file web app that uses the device camera and Google's Gemini AI to identify veins suitable for blood draws. Designed to help healthcare professionals with venipuncture site selection.

**Live URL:** https://chimezie90.github.io/findthevein/
**GitHub:** https://github.com/chimezie90/findthevein

## Key Technical Decisions

### 1. Single-File Architecture
- Everything in one `index.html` (~450 lines)
- No build step, no npm, no dependencies
- Just open in browser or deploy to any static host

### 2. Gemini API Integration
- Model: `gemini-2.0-flash`
- Calls go through Cloudflare Worker proxy (API key secured)
- PNG format for lossless image quality (better than JPEG for vein detection)

### 3. Prompt Engineering Learnings
**What didn't work:**
- Low temperature (0.1) - too restrictive
- Forcing JSON immediately - constrained model's reasoning
- Asking for bounding boxes - resulted in oversized annotations
- Being too eager to find veins - hallucinated veins on clothing

**What works:**
- Expert persona ("phlebotomist")
- **Checking for bare skin first** - critical to avoid false positives
- Point coordinates [y, x] instead of bounding boxes
- Temperature 0.4 for better detection
- Being conservative - only mark veins that are clearly visible
- Allowing "unspecified vein" instead of requiring exact names

### 4. Coordinate System
**CRITICAL:** Gemini returns coordinates as `[y, x]` (Y-first), not `[x, y]`
- Scale: 0-1000 normalized to image dimensions
- Must convert: `cx = (x / 1000) * canvas.width`

### 5. Annotation Design
- Small numbered circles (radius 12-16px) at exact vein points
- Teal (#00e5cc) for primary/best vein
- Blue (#6496ff) for secondary veins
- Outer glow ring for visibility
- Results panel below image with descriptions

## Current Prompt

```
You are an expert phlebotomist. Analyze this image for veins suitable for blood draws.

CRITICAL - First check:
1. Is bare skin actually visible? (not clothing, fabric, or other materials)
2. Can you see actual veins as blue/green lines under the skin?

If NO bare skin is visible, or you cannot see actual veins, return:
{"scene": "description of what you see", "veins": [], "recommendation": "No visible veins - please show bare skin with good lighting"}

Only mark veins you can ACTUALLY SEE - do not guess or hallucinate.

If veins ARE visible, respond with JSON:
{
  "scene": "Brief description (body part, lighting quality)",
  "veins": [
    {
      "point": [y, x],
      "name": "vein name or 'unspecified vein'",
      "description": "visibility and suitability"
    }
  ],
  "recommendation": "Which vein is best for blood draw"
}

COORDINATES:
- "point" = exact location of the vein [y, x]
- Scale: 0-1000 (normalized to image)
- Y first! [y, x] not [x, y]

Be conservative - only mark veins you can clearly see.
```

## API Key Handling

**Current (production-ready):** Cloudflare Worker proxy
- App calls `https://vein-finder-proxy.chimezie90.workers.dev`
- Worker holds Gemini API key as a secret (not in code)
- CORS headers configured for cross-origin requests

**To update the Gemini API key:**
```bash
cd worker
npx wrangler secret put GEMINI_API_KEY
# Paste new key when prompted
npx wrangler deploy
```

## Known Issues / Future Improvements

1. ~~**API key exposed**~~ - Fixed with Cloudflare Worker proxy
2. **Rate limits** - Free Gemini tier can hit daily limits quickly
3. **Lighting dependent** - Works best with good lighting
4. **Single image** - Could add video/continuous scanning mode

## File Structure

```
findthevein/
├── index.html          # Complete app (HTML + CSS + JS)
├── README.md           # User-facing documentation
├── CLAUDE.md           # This file - dev documentation
├── worker/
│   ├── index.js        # Cloudflare Worker (proxies Gemini API)
│   └── wrangler.toml   # Worker configuration
└── plans/
    └── feat-vein-finder-app.md  # Original plan with research
```

## Where We Left Off

✅ Core app complete and working
✅ Deployed to GitHub Pages
✅ Annotations redesigned (small circles + descriptions)
✅ Multi-vein detection enabled
✅ Scene description added for context
✅ **Cloudflare Worker proxy deployed** (API key secured)
✅ **Prompt improved** - checks for bare skin, conservative detection, allows unspecified veins

**The app is fully functional and production-ready.** API key is secure.

## Commands

```bash
# Serve locally
npx serve -s .

# Deploy app (auto via GitHub Pages on push to main)
git push origin main

# Deploy worker changes
cd worker
npx wrangler deploy

# Update Gemini API key
cd worker
npx wrangler secret put GEMINI_API_KEY
```
