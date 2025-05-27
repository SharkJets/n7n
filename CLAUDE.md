# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

n7n is a hybrid mobile/web application for viewing and managing n8n workflow execution data. It's built with Astro as a web app and packaged as an Android app using Capacitor.

## Key Technologies

- **Astro** - Static site generator with Vue components
- **Vue 3** - Component framework for interactive UI
- **Capacitor** - Cross-platform native runtime
- **TypeScript** - Type-safe development
- **Bugsnag** - Error tracking

## Development Commands

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Sync web assets to Android project
npx cap sync android

# Open Android Studio for native development
npx cap open android
```

## Architecture

The app has two main components:

1. **Setup Component** (`src/components/vue/Setup.vue`):
   - Handles API configuration (domain and API key)
   - Stores credentials using Capacitor Preferences
   - Validates n8n instance connectivity

2. **N8nDataViewer Component** (`src/components/vue/N8nDataViewer.vue`):
   - Fetches and displays workflow execution data
   - Filters by workflow and execution status
   - Supports pagination and data refresh
   - Uses CapacitorHttp for CORS-free API calls

## Important Notes

- The app uses CapacitorHttp plugin (enabled in `capacitor.config.ts`) to bypass CORS restrictions when calling n8n APIs
- Web assets build to `/dist/` directory
- Android-specific code is in `/android/` directory
- API credentials are stored securely using Capacitor Preferences API