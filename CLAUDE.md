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
- Direct REST API calls via fetch() - no SDK needed
- PNG format for lossless image quality (better than JPEG for vein detection)

### 3. Prompt Engineering Learnings
**What didn't work:**
- Low temperature (0.1) - too restrictive
- Forcing JSON immediately - constrained model's reasoning
- Asking for bounding boxes - resulted in oversized annotations

**What works:**
- Expert persona ("phlebotomist with 20 years experience")
- Explicit list of vein types to look for
- Point coordinates [y, x] instead of bounding boxes
- Temperature 0.4 for better detection
- Emphasizing "find ALL veins, even faint ones"

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
You are an expert phlebotomist with 20 years experience. Analyze this image thoroughly for ALL venipuncture sites.

IMPORTANT: Look carefully for ALL visible veins, not just the most obvious one. Check:
- Inner elbow: median cubital, cephalic, basilic veins
- Forearm: accessory cephalic, median antebrachial veins
- Wrist/hand: dorsal venous network, metacarpal veins

Even faint or subtle veins should be marked if they could potentially be used.

Respond with JSON:
{
  "scene": "Brief description (body part, position, lighting quality)",
  "veins": [
    {
      "point": [y, x],
      "name": "specific vein name",
      "description": "size, visibility, suitability for blood draw"
    }
  ],
  "recommendation": "Which vein (#1) is best and brief explanation"
}
```

## API Key Handling

**Current (prototype):** API key hardcoded in index.html for easy mobile testing
```javascript
const DEFAULT_KEY = 'AIzaSyBLcDn54z_tfZApBrXN3wfAIQ0P-nVP0xI';
```

**For production:** Must use backend proxy to keep key secure

## Known Issues / Future Improvements

1. **API key exposed** - Need backend proxy for production
2. **Rate limits** - Free tier can hit daily limits quickly
3. **Lighting dependent** - Works best with good lighting
4. **Single image** - Could add video/continuous scanning mode

## File Structure

```
findthevein/
├── index.html          # Complete app (HTML + CSS + JS)
├── README.md           # User-facing documentation
├── CLAUDE.md           # This file - dev documentation
└── plans/
    └── feat-vein-finder-app.md  # Original plan with research
```

## Where We Left Off

✅ Core app complete and working
✅ Deployed to GitHub Pages
✅ Annotations redesigned (small circles + descriptions)
✅ Multi-vein detection enabled
✅ Scene description added for context

**The app is fully functional.** User tested on iPhone Safari successfully.

## Commands

```bash
# Serve locally
npx serve -s .

# Deploy (auto via GitHub Pages on push to main)
git push origin main
```
