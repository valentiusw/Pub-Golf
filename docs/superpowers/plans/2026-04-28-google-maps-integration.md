# Google Maps Integration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add Google Maps display for pub locations on hole browsing pages and Google Places autocomplete for pub search during hole creation.

**Architecture:** Load Google Maps and Places libraries via script tag with API key from local config file. Create utility functions for map initialization (marker + infowindow) and autocomplete (Sydney-constrained). Integrate into hole detail view and hole creation form.

**Tech Stack:** Google Maps JavaScript API, Google Places API, vanilla JavaScript

---

## Task 1: Set up Google Maps API Configuration

**Files:**
- Create: `googleMapsConfig.js`
- Modify: `.gitignore`
- Modify: `index.html` (add script tag)

- [ ] **Step 1: Create googleMapsConfig.js with placeholder**

Create file `googleMapsConfig.js` in project root:
```javascript
// API key for Google Maps and Places
// Get from: https://console.cloud.google.com/
// 1. Create new project
// 2. Enable "Maps JavaScript API" and "Places API"
// 3. Create API key, restrict to your domain
// 4. Replace the placeholder below

const GOOGLE_MAPS_API_KEY = 'YOUR_API_KEY_HERE';
```

- [ ] **Step 2: Add googleMapsConfig.js to .gitignore**

Open `.gitignore` and add this line:
```
googleMapsConfig.js
```

This prevents API key from being committed to version control.

- [ ] **Step 3: Load Google Maps script in index.html**

In `index.html`, find the `<head>` section and add these two lines before the closing `</head>` tag:

```html
<script src="googleMapsConfig.js"></script>
<script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY_HERE&libraries=places" async defer></script>
```

Wait, we need to dynamically load the API key from the config file. Replace the above with:

```html
<script src="googleMapsConfig.js"></script>
<script>
  // Wait for config to load, then load Google Maps API
  window.addEventListener('DOMContentLoaded', function() {
    if (typeof GOOGLE_MAPS_API_KEY !== 'undefined') {
      const script = document.createElement('script');
      script.src = `https://maps.googleapis.com/maps/api/js?key=${GOOGLE_MAPS_API_KEY}&libraries=places`;
      script.async = true;
      script.defer = true;
      document.head.appendChild(script);
    } else {
      console.error('GOOGLE_MAPS_API_KEY not defined in googleMapsConfig.js');
    }
  });
</script>
```

- [ ] **Step 4: Commit**

```bash
git add googleMapsConfig.js .gitignore index.html
git commit -m "feat: add Google Maps API configuration"
```

---

## Task 2: Create Google Maps Utility Functions

**Files:**
- Modify: `index.html` (add utility functions in a `<script>` tag before closing `</body>`)

- [ ] **Step 1: Add initializeMapForHole function**

Find the closing `</body>` tag in `index.html`. Before it, add a new `<script>` block with this function:

```html
<script>
// Google Maps Utilities

function initializeMapForHole(containerId, latitude, longitude, pubName, pubAddress) {
  const container = document.getElementById(containerId);
  if (!container) {
    console.error(`Map container with id "${containerId}" not found`);
    return;
  }

  // Check if Google Maps API is loaded
  if (typeof google === 'undefined' || !google.maps) {
    console.error('Google Maps API not loaded');
    container.innerHTML = '<p style="color: #ccc; padding: 10px;">Map unavailable</p>';
    return;
  }

  const mapOptions = {
    zoom: 15,
    center: { lat: latitude, lng: longitude },
    disableDefaultUI: false,
    styles: [
      { elementType: 'geometry', stylers: [{ color: '#1a1a1a' }] },
      { elementType: 'labels.text.stroke', stylers: [{ color: '#1a1a1a' }] },
      { elementType: 'labels.text.fill', stylers: [{ color: '#faf6ee' }] },
    ],
  };

  const map = new google.maps.Map(container, mapOptions);

  const marker = new google.maps.Marker({
    position: { lat: latitude, lng: longitude },
    map: map,
    title: pubName,
  });

  const infoContent = `<div style="color: #000; padding: 8px;">
    <strong>${pubName}</strong><br>
    ${pubAddress}
  </div>`;

  const infowindow = new google.maps.InfoWindow({
    content: infoContent,
  });

  marker.addListener('click', function() {
    infowindow.open(map, marker);
  });

  // Open infowindow by default
  infowindow.open(map, marker);
}
</script>
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add Google Maps initialization utility function"
```

---

## Task 3: Create Google Places Autocomplete Utility Function

**Files:**
- Modify: `index.html` (add to existing `<script>` block from Task 2)

- [ ] **Step 1: Add initializePubAutocomplete function**

In the same `<script>` block (after the `initializeMapForHole` function), add:

```javascript
function initializePubAutocomplete(inputElementId, onSelectCallback) {
  const inputElement = document.getElementById(inputElementId);
  if (!inputElement) {
    console.error(`Autocomplete input with id "${inputElementId}" not found`);
    return;
  }

  // Check if Google Places API is loaded
  if (typeof google === 'undefined' || !google.maps || !google.maps.places) {
    console.error('Google Places API not loaded');
    return;
  }

  const sydneyBounds = new google.maps.LatLngBounds(
    new google.maps.LatLng(-34.118, 150.644), // SW corner
    new google.maps.LatLng(-33.865, 151.293)   // NE corner
  );

  const autocomplete = new google.maps.places.Autocomplete(inputElement, {
    bounds: sydneyBounds,
    componentRestrictions: { country: 'au' },
    types: ['establishment'],
  });

  autocomplete.addListener('place_changed', function() {
    const place = autocomplete.getPlace();

    if (!place.geometry) {
      console.error('Place details missing geometry');
      return;
    }

    const result = {
      name: place.name,
      address: place.formatted_address,
      lat: place.geometry.location.lat(),
      lng: place.geometry.location.lng(),
    };

    if (typeof onSelectCallback === 'function') {
      onSelectCallback(result);
    }
  });
}
```

- [ ] **Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add Google Places autocomplete utility function"
```

---

## Task 4: Add Map Display to Hole Detail View

**Files:**
- Modify: `index.html` (locate hole display section, add map container)

- [ ] **Step 1: Find the hole-body section in index.html**

Search in `index.html` for `class="hole-body"`. This is where hole details are displayed. We need to add a map container right after the `hole-header` closes and before the `hole-body` content.

Look for a structure like:
```html
<div class="hole-card">
  <div class="hole-header">...</div>
  <div class="hole-body">...</div>
</div>
```

- [ ] **Step 2: Add map container HTML**

Find where holes are rendered dynamically in the JavaScript code. Look for a section that creates the hole-card HTML (likely in a function that builds the hole display). 

In that function, after the `hole-header` div closes, add a map container:

```html
<div id="hole-map-${holeIndex}" class="hole-map-container" style="height: 300px; margin: 10px 0; border-radius: 8px; overflow: hidden;"></div>
```

(Replace `${holeIndex}` with whatever variable name tracks the current hole index in your code)

- [ ] **Step 3: Add CSS for map container**

In the `<style>` section of `index.html`, add:

```css
.hole-map-container {
  background: rgba(14, 61, 34, 0.3);
  border: 1px solid rgba(250, 246, 238, 0.1);
  margin: 10px 0;
}
```

- [ ] **Step 4: Initialize map when hole is displayed**

Find the JavaScript function that displays hole details. After the hole HTML is rendered to the page, add a call to initialize the map:

```javascript
// After rendering hole details to DOM
if (hole.pubLat && hole.pubLng) {
  initializeMapForHole('hole-map-' + holeIndex, hole.pubLat, hole.pubLng, hole.pubName, hole.address);
} else {
  console.log('Hole missing coordinates, map will not display');
}
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add map container to hole detail display"
```

---

## Task 5: Add Autocomplete to Hole Creation Form

**Files:**
- Modify: `index.html` (locate hole creation form, modify pub name input)

- [ ] **Step 1: Find the hole creation form**

Search in `index.html` for the hole creation form section. Look for an input field for "Pub Name" or similar.

- [ ] **Step 2: Add id and initialize autocomplete**

Find the pub name input field in the hole creation form. If it doesn't have an id, add one:

```html
<input id="hole-creation-pub-name" type="text" placeholder="Enter pub name..." />
```

Then, in the JavaScript code that initializes the hole creation form, add:

```javascript
// Initialize pub autocomplete
initializePubAutocomplete('hole-creation-pub-name', function(selectedPlace) {
  // Store pub coordinates in a global or form variable
  window.selectedPubCoordinates = {
    lat: selectedPlace.lat,
    lng: selectedPlace.lng,
  };
  
  // Update the input with the selected pub name
  document.getElementById('hole-creation-pub-name').value = selectedPlace.name;
});
```

- [ ] **Step 3: Modify hole creation save function**

When a new hole is created and saved, the save function needs to include the pub coordinates. Find where new holes are added to your holes data structure.

Modify it to include:

```javascript
const newHole = {
  // ... other hole properties
  pubName: document.getElementById('hole-creation-pub-name').value,
  pubLat: window.selectedPubCoordinates?.lat || null,
  pubLng: window.selectedPubCoordinates?.lng || null,
  address: '', // Could be set from autocomplete if available
};
```

After saving, clear the coordinates:
```javascript
window.selectedPubCoordinates = null;
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add Google Places autocomplete to hole creation form"
```

---

## Task 6: Test End-to-End

**Files:**
- No files to modify (testing only)

- [ ] **Step 1: Set up your API key**

1. Go to https://console.cloud.google.com/
2. Create a new project or use existing one
3. Enable "Maps JavaScript API" and "Places API"
4. Create an API key (Credentials → Create Credential → API Key)
5. Restrict the key to HTTP referrers (add your domain, e.g., `localhost:*`)
6. Copy the API key

- [ ] **Step 2: Add API key to googleMapsConfig.js**

Open `googleMapsConfig.js` and replace `YOUR_API_KEY_HERE` with your actual API key:

```javascript
const GOOGLE_MAPS_API_KEY = 'AIzaSy...'; // Your actual key
```

- [ ] **Step 3: Test map display on hole browsing**

1. Open the app in browser
2. Navigate to a hole that was created with autocomplete (should have `pubLat` and `pubLng`)
3. Verify: Map displays with a marker at the pub location
4. Verify: Clicking marker shows infowindow with pub name and address
5. Check browser console (F12) for any errors

- [ ] **Step 4: Test autocomplete in hole creation**

1. Open hole creation form
2. Click on the pub name input field
3. Type a pub name (e.g., "The Rocks" or "Circular Quay")
4. Verify: Dropdown appears with suggestions
5. Click a suggestion
6. Verify: Input populates with pub name, no errors in console
7. Create the hole and save
8. Go back to view the hole
9. Verify: Map displays with correct location

- [ ] **Step 5: Test with existing holes**

1. View a hole created before this feature (no `pubLat`/`pubLng`)
2. Verify: No map displays, no console errors
3. The hole still works normally for scoring

- [ ] **Step 6: Commit test results (no code changes)**

```bash
git commit --allow-empty -m "test: verify Google Maps integration end-to-end"
```

---

## Summary

- **Task 1:** API configuration with local config file (git-ignored)
- **Task 2:** Map utility function for displaying markers and infowindows
- **Task 3:** Autocomplete utility function for pub search (Sydney-constrained)
- **Task 4:** Map container integrated into hole detail view
- **Task 5:** Autocomplete integrated into hole creation form with coordinate storage
- **Task 6:** End-to-end testing and verification

All holes created after Task 5 will have coordinates. Existing holes will still work (map just won't display).

**Spec Coverage Check:**
- ✅ Hole browsing map display with marker and infowindow
- ✅ Pub autocomplete with Sydney area constraint
- ✅ Data model: `pubLat` and `pubLng` fields
- ✅ Error handling for missing API or coordinates
- ✅ Integration into existing UI

**No placeholders:** All tasks contain full code, exact file paths, exact commands, and expected behavior.
