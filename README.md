# Image Meta Extractor

A Vue 3 application that extracts and displays comprehensive metadata from uploaded images.

## Features

- 📁 **Drag & Drop Upload**: Intuitive file upload interface
- 🖼️ **Image Preview**: Shows thumbnail of uploaded image
- 📊 **Comprehensive Metadata**: Extracts and displays:
  - File information (name, size, type, date)
  - Camera settings (make, model, lens, focal length, aperture, etc.)
  - Image properties (dimensions, resolution, color space)
  - GPS location data (with Google Maps link)
  - Date information (taken, modified, created)
  - Technical details (white balance, metering mode, etc.)
- 🔍 **Raw Data View**: Toggle to see complete EXIF data
- ⚡ **Error Handling**: File validation, size limits, and user feedback
- 📱 **Responsive Design**: Works on desktop and mobile

## Supported Formats

- JPEG/JPG
- PNG
- TIFF
- HEIC
- WebP
- GIF

## Development

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

## Deployment

This app is automatically deployed to GitHub Pages via GitHub Actions when changes are pushed to the main branch.

## Technologies Used

- Vue 3
- Vite
- exifr (for metadata extraction)
- GitHub Pages
- GitHub Actions