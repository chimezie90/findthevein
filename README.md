# Vein Finder

A simple web application that uses your device's camera and Google's Gemini AI to identify veins suitable for blood draws. Designed to assist healthcare professionals with venipuncture site selection.

## Features

- **Camera Capture**: Uses device camera (back camera on mobile)
- **AI Analysis**: Gemini 2.0 Flash analyzes images for visible veins
- **Visual Annotations**: Overlays bounding boxes on detected veins
- **Primary/Secondary Classification**: Recommends best venipuncture sites
- **Mobile-Friendly**: Works on phones and tablets

## Quick Start

1. Open `index.html` in a browser (or serve via HTTPS for camera access)
2. Enter your Gemini API key when prompted
3. Point camera at inner arm/hand
4. Tap the capture button to analyze

## Getting a Gemini API Key

1. Go to [Google AI Studio](https://aistudio.google.com/)
2. Click "Get API Key"
3. Create a new key or use an existing one

## Development

```bash
# Serve locally (requires npx)
npx serve -s .

# Or use Python
python -m http.server 8000
```

## Security Note

This prototype stores the API key in localStorage for convenience. For production use, implement a backend proxy to keep the API key secure.

## Medical Disclaimer

**For informational purposes only.** This tool provides visual guidance and is not a substitute for professional medical judgment. Results should be verified by qualified healthcare personnel. This application is NOT a medical device and has NOT been evaluated by the FDA or any regulatory body.

## License

MIT
