---
name: Google Maps Integration for Pub Golf
description: Add Google Maps display for hole locations and Google Places autocomplete for pub search in hole creation
type: feature-design
date: 2026-04-28
---

# Google Maps Integration Design

## Overview
Integrate Google Maps API to display pub locations when browsing holes, and add Google Places API autocomplete for pub name search during hole creation. This is the first Google API integration for the project.

## Features

### 1. Hole Browsing – Map Display
**What:** When viewing a hole, display a Google Map showing the pub's location.

**User experience:**
- Map appears below hole header/summary, above scoring section
- Map shows a single red marker pin at the pub's coordinates
- Clicking the marker shows an infowindow with pub name and address
- Map has basic controls: zoom and pan
- Map is responsive and adapts to mobile screen sizes
- Map loads after page renders (async loading)

**Data required:**
- Pub coordinates (latitude, longitude) — stored in hole data
- Pub name and address — displayed in marker infowindow

**Constraints:**
- Show only the single pub, no surrounding context
- Map should be visually consistent with app's dark theme

### 2. Hole Creation – Pub Autocomplete
**What:** In the hole creation form, add a pub name input field with autocomplete suggestions powered by Google Places API.

**User experience:**
- Input field labeled "Pub Name" 
- As user types, dropdown appears with matching pub suggestions
- Results limited to Sydney area (location-biased search)
- Each suggestion shows pub name and address
- Clicking a suggestion:
  - Populates the input with pub name
  - Stores pub coordinates (lat/lng) for the map
  - Closes the dropdown
- Hitting Escape or clicking outside dismisses dropdown

**Data flow:**
- User types → Google Places API called with search text + Sydney location bias
- Results returned → dropdown rendered
- User selects → coordinates extracted and stored in hole object

**Constraints:**
- Autocomplete limited to Sydney area
- Only show business/establishment results, not generic locations
- Handle empty search gracefully (no results)

### 3. Technical Setup

#### API Key Management
- Get Google Cloud API key from Google Cloud Console
  - Create project, enable Maps JavaScript API and Places API
  - Restrict key to HTTP referrer (your domain)
- Store API key locally in `googleMapsConfig.js` (git-ignored)
- Load Google Maps library via `<script>` tag with key as query parameter

#### Implementation Structure
- Create `googleMapsConfig.js`:
  ```javascript
  const GOOGLE_MAPS_API_KEY = 'your-api-key-here';
  ```
  Add to `.gitignore`

- In `index.html`, load library:
  ```html
  <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places"></script>
  ```

- Create utility functions (in `index.html` or separate `.js` file):
  - `initializeMapForHole(elementId, lat, lng, pubName, address)` — renders map with marker
  - `initializePubAutocomplete(inputElementId, onSelectCallback)` — sets up autocomplete
  - Helper to extract coordinates from Places result

#### Error Handling
- If API fails to load, show user-friendly message (e.g., "Map unavailable")
- If coordinates missing for a hole, skip map rendering
- Handle autocomplete API errors gracefully (show "Search failed" message)

#### Integration Points
**Hole browsing:**
- When hole detail is displayed, call `initializeMapForHole()` with hole's pub coordinates
- Map container: add `<div id="hole-map"></div>` to hole detail section

**Hole creation:**
- Pub name input field: add id for autocomplete targeting
- Call `initializePubAutocomplete()` when form initializes
- On selection, populate pub name and store coordinates in form data

## Data Model Changes
Each hole needs to store pub coordinates (new fields):
- `pubLat: number` — latitude of pub location
- `pubLng: number` — longitude of pub location
- Auto-populated when user selects a pub from autocomplete during hole creation
- For existing holes without coordinates, the map will not render (no error, just skip)

## Browser Compatibility
Google Maps API works on all modern browsers (Chrome, Firefox, Safari, Edge) and mobile browsers.

## Security Considerations
- API key is visible in client code (intentional for client-side setup)
- Key restricted to domain via Google Cloud Console
- No sensitive user data sent to Google beyond search queries

## Success Criteria
1. Maps display correctly on hole browsing pages with accurate pub location
2. Autocomplete works smoothly in hole creation form with Sydney area filtering
3. Selecting an autocomplete result populates pub coordinates
4. No console errors when APIs load
5. Mobile experience is smooth and responsive
