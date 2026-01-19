# Property Hunter - Property Cards & UX Fixes Implementation Plan

**Date:** 19 January 2026
**Status:** Ready for Implementation
**Target File:** `/home/user/property-hunter/index.html`
**Estimated Tasks:** 4 major issues, ~20 subtasks

---

## Table of Contents

1. [Overview](#overview)
2. [Issue 1: Property Cards Display](#issue-1-property-cards-display)
3. [Issue 2: Hamburger Menu Touch Issues](#issue-2-hamburger-menu-touch-issues)
4. [Issue 3: Custom Category Deletion](#issue-3-custom-category-deletion)
5. [Issue 4: Status Colors & Icons](#issue-4-status-colors--icons)
6. [Testing Guide](#testing-guide)
7. [Commit Strategy](#commit-strategy)

---

## Overview

### Codebase Context

This is a **single-file vanilla JavaScript application** (`index.html`, ~2300 lines). No build process, no framework. Everything lives in one file:

- **Lines 1-12:** HTML head, meta tags, Leaflet CSS CDN
- **Lines 13-1006:** Embedded `<style>` block (all CSS)
- **Lines 1007-1100:** HTML body structure
- **Lines 1101-2307:** Embedded `<script>` block (all JavaScript)

### Key Architecture Concepts

```
Store object (lines 1136-1275)
├── data.properties[]     → Array of property objects
├── data.customCategories[] → User-added scoring categories
├── getProperty(id)       → Fetch single property
├── addProperty(prop)     → Create new property
├── updateProperty(id, updates) → Modify existing
└── save()               → Persist to LocalStorage

Rendering Functions
├── renderApp()          → Entry point, calls applyFilters()
├── applyFilters()       → Filters properties, calls render functions
├── renderPropertyList() → Creates card HTML for each property
├── createPropertyCard() → Returns HTML string for one card
└── openPropertyDetail() → Opens modal with full property info
```

### Data Model (Property Object)

```javascript
{
  id: "uuid-string",
  url: "https://www.immoweb.be/...",      // Link to listing
  address: "Straat 123, Gemeente",         // Display title
  price: 250000,                           // Number
  coordinates: { lat: 50.123, lng: 4.456 },
  status: "nieuw",                         // Decision status
  propertyStatus: "beschikbaar",           // Availability status
  scores: { locatie: 5, energie: 3, ... }, // Category ratings
  pros: "Text",
  cons: "Text",
  visits: [{ date, notes }],
  bids: [{ amount, date, status, notes }],
  dateAdded: "ISO string"
}
```

---

## Issue 1: Property Cards Display

### Problem Statement

Property cards are missing key information and layout:
- **Missing:** Property image on the right side
- **Current order:** address → price → badges → score
- **Required order:** address → price → badges → score (correct, but need image)

### Root Cause Analysis

1. **No image field in data model** - `addProperty()` at line 1186 doesn't include `imageUrl`
2. **No image input in add form** - `openAddPropertyStep1()` at line 2006 lacks image field
3. **No image in card HTML** - `createPropertyCard()` at line 1735 doesn't render images
4. **Detail view missing clickable URL** - Actually present at line 1429, but verify it's prominent

### Files & Line Numbers

| What | Where | Lines |
|------|-------|-------|
| Property data model | `addProperty()` function | 1186-1205 |
| Add property form (Step 1) | `openAddPropertyStep1()` function | 2006-2058 |
| Property card HTML | `createPropertyCard()` function | 1735-1755 |
| Property card CSS | `.property-card` styles | 294-362 |
| Detail view header | `openPropertyDetail()` function | 1408-1430 |

### Implementation Tasks

#### Task 1.1: Add imageUrl to Data Model

**File:** `index.html`
**Location:** Line 1186-1205, inside `addProperty()` function

**Current code:**
```javascript
addProperty(property) {
  const newProperty = {
    id: generateId(),
    url: property.url || '',
    address: property.address || '',
    price: property.price || 0,
    coordinates: property.coordinates || { lat: 0, lng: 0 },
    status: 'nieuw',
    // ...
  };
```

**Change to:**
```javascript
addProperty(property) {
  const newProperty = {
    id: generateId(),
    url: property.url || '',
    address: property.address || '',
    price: property.price || 0,
    imageUrl: property.imageUrl || '',  // ADD THIS LINE
    coordinates: property.coordinates || { lat: 0, lng: 0 },
    status: 'nieuw',
    // ...
  };
```

**Test:** Open browser console, run:
```javascript
Store.addProperty({ address: 'Test', imageUrl: 'https://example.com/img.jpg' });
console.log(Store.getProperties().slice(-1)[0].imageUrl); // Should print URL
```

---

#### Task 1.2: Add Image URL Input to Add Property Form

**File:** `index.html`
**Location:** Line 2019-2033, inside `openAddPropertyStep1()` HTML template

**Current code (around line 2020-2032):**
```javascript
<div class="form-group">
  <label class="form-label">Link (immoweb/zimmo/biddit)</label>
  <input type="url" class="form-input" id="addUrl" placeholder="https://www.immoweb.be/...">
  <p class="form-hint">Optioneel - plak de link naar de advertentie</p>
</div>
<div class="form-group">
  <label class="form-label form-label--required">Adres</label>
  <input type="text" class="form-input" id="addAddress" placeholder="Straat 123, Gemeente">
</div>
<div class="form-group">
  <label class="form-label">Prijs (€)</label>
  <input type="number" class="form-input" id="addPrice" placeholder="250000">
</div>
```

**Change to:**
```javascript
<div class="form-group">
  <label class="form-label">Link (immoweb/zimmo/biddit)</label>
  <input type="url" class="form-input" id="addUrl" placeholder="https://www.immoweb.be/...">
  <p class="form-hint">Optioneel - plak de link naar de advertentie</p>
</div>
<div class="form-group">
  <label class="form-label form-label--required">Adres</label>
  <input type="text" class="form-input" id="addAddress" placeholder="Straat 123, Gemeente">
</div>
<div class="form-group">
  <label class="form-label">Prijs (€)</label>
  <input type="number" class="form-input" id="addPrice" placeholder="250000">
</div>
<div class="form-group">
  <label class="form-label">Afbeelding URL</label>
  <input type="url" class="form-input" id="addImageUrl" placeholder="https://...jpg">
  <p class="form-hint">Optioneel - kopieer afbeelding-URL van de advertentie</p>
</div>
```

**Also update:** The data collection in the "Next" button handler (around line 2052-2054):

**Current:**
```javascript
addPropertyData.url = document.getElementById('addUrl').value.trim();
addPropertyData.address = address;
addPropertyData.price = parseInt(document.getElementById('addPrice').value) || 0;
```

**Change to:**
```javascript
addPropertyData.url = document.getElementById('addUrl').value.trim();
addPropertyData.address = address;
addPropertyData.price = parseInt(document.getElementById('addPrice').value) || 0;
addPropertyData.imageUrl = document.getElementById('addImageUrl').value.trim();
```

**Also update:** The `addPropertyData` initialization (around line 2007-2012):

**Current:**
```javascript
addPropertyData = {
  url: '',
  address: '',
  price: '',
  coordinates: null
};
```

**Change to:**
```javascript
addPropertyData = {
  url: '',
  address: '',
  price: '',
  imageUrl: '',
  coordinates: null
};
```

**Also update:** The `saveNewProperty()` function (around line 2113-2118):

**Current:**
```javascript
const property = Store.addProperty({
  url: addPropertyData.url,
  address: addPropertyData.address,
  price: addPropertyData.price,
  coordinates: addPropertyData.coordinates
});
```

**Change to:**
```javascript
const property = Store.addProperty({
  url: addPropertyData.url,
  address: addPropertyData.address,
  price: addPropertyData.price,
  imageUrl: addPropertyData.imageUrl,
  coordinates: addPropertyData.coordinates
});
```

**Test:** Add a new property with an image URL. Check LocalStorage in DevTools to verify imageUrl is saved.

---

#### Task 1.3: Add Property Card CSS for Image Layout

**File:** `index.html`
**Location:** After line 362 (after `.property-card__score-value`), add new CSS

**Add this CSS:**
```css
/* Property card with image layout */
.property-card {
  display: flex;
  gap: var(--spacing-md);
}

.property-card__content {
  flex: 1;
  min-width: 0; /* Prevent flex child overflow */
}

.property-card__image {
  width: 80px;
  height: 80px;
  border-radius: var(--radius-sm);
  object-fit: cover;
  flex-shrink: 0;
  background: var(--color-gray-100);
}

.property-card__image--placeholder {
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--color-gray-400);
  font-size: 24px;
}
```

**Test:** Add a property with an image URL, verify the card shows image on right.

---

#### Task 1.4: Update createPropertyCard() to Show Image

**File:** `index.html`
**Location:** Lines 1735-1755, the `createPropertyCard()` function

**Current code:**
```javascript
function createPropertyCard(property) {
  const avgScore = Store.calculateAverageScore(property);
  const scoreDisplay = avgScore > 0
    ? `<span class="property-card__score">
         ${renderStars(avgScore)}
         <span class="property-card__score-value">${avgScore.toFixed(1)}</span>
       </span>`
    : '<span class="property-card__score property-card__score-value">Niet beoordeeld</span>';

  return `
    <div class="property-card" data-id="${property.id}">
      <div class="property-card__address">${escapeHtml(property.address)}</div>
      <div class="property-card__price">€${property.price.toLocaleString('nl-BE')}</div>
      <div class="property-card__badges">
        <span class="badge badge--${property.status}">${STATUS_LABELS[property.status]}</span>
        <span class="badge badge--${property.propertyStatus}">${PROPERTY_STATUS_LABELS[property.propertyStatus]}</span>
      </div>
      ${scoreDisplay}
    </div>
  `;
}
```

**Change to:**
```javascript
function createPropertyCard(property) {
  const avgScore = Store.calculateAverageScore(property);
  const scoreDisplay = avgScore > 0
    ? `<span class="property-card__score">
         ${renderStars(avgScore)}
         <span class="property-card__score-value">${avgScore.toFixed(1)}</span>
       </span>`
    : '<span class="property-card__score property-card__score-value">Niet beoordeeld</span>';

  const imageHtml = property.imageUrl
    ? `<img src="${escapeHtml(property.imageUrl)}" alt="" class="property-card__image" loading="lazy">`
    : `<div class="property-card__image property-card__image--placeholder">🏠</div>`;

  return `
    <div class="property-card" data-id="${property.id}">
      <div class="property-card__content">
        <div class="property-card__address">${escapeHtml(property.address)}</div>
        <div class="property-card__price">€${property.price.toLocaleString('nl-BE')}</div>
        <div class="property-card__badges">
          <span class="badge badge--${property.status}">${STATUS_LABELS[property.status]}</span>
          <span class="badge badge--${property.propertyStatus}">${PROPERTY_STATUS_LABELS[property.propertyStatus]}</span>
        </div>
        ${scoreDisplay}
      </div>
      ${imageHtml}
    </div>
  `;
}
```

**Test:**
1. Property with image URL → shows actual image
2. Property without image URL → shows placeholder with house emoji
3. Card layout: content on left, image on right

---

#### Task 1.5: Add Image to Property Detail View

**File:** `index.html`
**Location:** Lines 1426-1430, inside `openPropertyDetail()` function, in the detail-header section

**Current code:**
```javascript
<!-- Header -->
<div class="detail-header">
  <div class="detail-address">${escapeHtml(property.address)}</div>
  <div class="detail-price">€${property.price.toLocaleString('nl-BE')}</div>
  ${property.url ? `<a href="${escapeHtml(property.url)}" target="_blank" class="detail-link">🔗 Bekijk advertentie</a>` : ''}
</div>
```

**Change to:**
```javascript
<!-- Header -->
<div class="detail-header">
  ${property.imageUrl ? `<img src="${escapeHtml(property.imageUrl)}" alt="" class="detail-image">` : ''}
  <div class="detail-address">${escapeHtml(property.address)}</div>
  <div class="detail-price">€${property.price.toLocaleString('nl-BE')}</div>
  ${property.url ? `<a href="${escapeHtml(property.url)}" target="_blank" class="detail-link">🔗 Bekijk advertentie</a>` : ''}
</div>
```

**Add CSS** (find `.detail-header` styles, around line 450-470):
```css
.detail-image {
  width: 100%;
  max-height: 200px;
  object-fit: cover;
  border-radius: var(--radius-md);
  margin-bottom: var(--spacing-md);
}
```

**Test:** Open a property with imageUrl, verify image displays at top of detail view.

---

#### Task 1.6: Add Image URL Edit Field to Detail View

**File:** `index.html`
**Location:** Inside `openPropertyDetail()`, after the URL link (around line 1430), add an edit section

**Add this after the detail-header section, before the Status section:**
```javascript
<!-- Image URL -->
<div class="detail-section">
  <div class="detail-section__title">Afbeelding</div>
  <input type="url" class="form-input" id="detailImageUrl"
         value="${escapeHtml(property.imageUrl || '')}"
         onchange="updatePropertyField('imageUrl', this.value)"
         placeholder="https://...jpg">
</div>
```

**Test:**
1. Open property detail
2. Paste an image URL
3. Close and reopen - image should appear
4. Card should now show the image

---

## Issue 2: Hamburger Menu Touch Issues

### Problem Statement

The hamburger menu (☰) button doesn't respond to taps on iOS Safari and Android browsers.

### Root Cause Analysis

1. **Missing touch-specific CSS** - No `-webkit-tap-highlight-color` or `touch-action`
2. **Event propagation issues** - The `<span>☰</span>` inside the button may intercept touch events
3. **Click event timing** - iOS Safari has 300ms delay on click events without proper meta tags
4. **Button content may not be "clickable"** - Some mobile browsers need explicit pointer-events handling

### Files & Line Numbers

| What | Where | Lines |
|------|-------|-------|
| Menu button HTML | `<button class="header__menu-btn">` | 1013-1015 |
| Menu button CSS | `.header__menu-btn` | 104-115 |
| Menu dropdown HTML | `<div class="menu-dropdown">` | 1019-1022 |
| Menu JS handler | `menuBtn.addEventListener('click')` | 2145-2148 |
| Click outside handler | `document.addEventListener('click')` | 2151-2157 |

### Implementation Tasks

#### Task 2.1: Add Touch-Friendly CSS to Menu Button

**File:** `index.html`
**Location:** Lines 104-115, the `.header__menu-btn` CSS rule

**Current code:**
```css
.header__menu-btn {
  width: var(--touch-target);
  height: var(--touch-target);
  background: none;
  border: none;
  color: white;
  font-size: var(--font-size-xl);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

**Change to:**
```css
.header__menu-btn {
  width: var(--touch-target);
  height: var(--touch-target);
  background: none;
  border: none;
  color: white;
  font-size: var(--font-size-xl);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  /* Touch fixes for iOS/Android */
  -webkit-tap-highlight-color: rgba(255, 255, 255, 0.2);
  touch-action: manipulation;
  user-select: none;
  -webkit-user-select: none;
}

.header__menu-btn span {
  pointer-events: none;
}
```

**Why this works:**
- `touch-action: manipulation` - Disables double-tap-to-zoom delay
- `-webkit-tap-highlight-color` - Shows visual feedback on tap
- `user-select: none` - Prevents text selection on long press
- `pointer-events: none` on span - Ensures touch events reach the button, not the span

---

#### Task 2.2: Add Touch Event Handler as Fallback

**File:** `index.html`
**Location:** Lines 2144-2148, the menu button event listener

**Current code:**
```javascript
// Toggle menu dropdown
document.getElementById('menuBtn').addEventListener('click', () => {
  const dropdown = document.getElementById('menuDropdown');
  dropdown.hidden = !dropdown.hidden;
});
```

**Change to:**
```javascript
// Toggle menu dropdown - handle both click and touch
function toggleMenu(e) {
  e.preventDefault();
  e.stopPropagation();
  const dropdown = document.getElementById('menuDropdown');
  dropdown.hidden = !dropdown.hidden;
}

const menuBtn = document.getElementById('menuBtn');
menuBtn.addEventListener('click', toggleMenu);
menuBtn.addEventListener('touchend', toggleMenu);
```

**Note:** Using both `click` and `touchend` ensures it works on all devices. The `preventDefault()` on touchend prevents the subsequent "ghost click" that some browsers fire.

---

#### Task 2.3: Fix Menu Item Touch Handling

**File:** `index.html`
**Location:** Lines 1019-1022, the menu dropdown HTML

**Current code:**
```html
<div class="menu-dropdown" id="menuDropdown" hidden>
  <button class="menu-item" onclick="exportData()">📤 Exporteren</button>
  <button class="menu-item" onclick="openImportModal()">📥 Importeren</button>
</div>
```

**Note:** The `onclick` attributes should work, but let's add touch-friendly CSS for consistency.

**Add to CSS** (after `.menu-item:active` around line 936):
```css
.menu-item {
  /* Add these properties to existing .menu-item rule */
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0.1);
  touch-action: manipulation;
  user-select: none;
  -webkit-user-select: none;
}
```

---

#### Task 2.4: Verify Meta Viewport Tag

**File:** `index.html`
**Location:** Line 5

**Current code:**
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

**This is correct** - no change needed. But if issues persist, this alternative may help:
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```

**Warning:** Disabling zoom is an accessibility concern. Only use if absolutely necessary.

---

## Issue 3: Custom Category Deletion

### Problem Statement

Custom categories, once added, cannot be deleted from the property detail view.

### Root Cause Analysis

1. **Missing `removeCustomCategory()` function** - Only `addCustomCategory()` exists in Store
2. **No delete UI** - Score rows don't have delete buttons
3. **Need to distinguish default vs custom categories** - Only custom ones should be deletable

### Files & Line Numbers

| What | Where | Lines |
|------|-------|-------|
| DEFAULT_CATEGORIES | Constant definition | 1106-1107 |
| Store.addCustomCategory() | Add function | 1242-1255 |
| Store.getAllCategories() | Get all categories | 1237-1240 |
| Score rows HTML | Inside openPropertyDetail() | 1458-1468 |
| Add category UI | Input + button | 1470-1473 |

### Implementation Tasks

#### Task 3.1: Add removeCustomCategory() to Store

**File:** `index.html`
**Location:** After `addCustomCategory()` (around line 1255), add new function

**Add this code:**
```javascript
// Remove custom category
removeCustomCategory(key) {
  const index = this.data.customCategories.indexOf(key);
  if (index === -1) return false;  // Not a custom category or doesn't exist

  // Remove from customCategories array
  this.data.customCategories.splice(index, 1);

  // Remove from CATEGORY_LABELS
  delete CATEGORY_LABELS[key];

  // Remove from all properties' scores
  this.data.properties.forEach(p => {
    delete p.scores[key];
  });

  this.save();
  return true;
},
```

**Test:**
```javascript
// In console:
Store.addCustomCategory('test');
console.log(Store.getAllCategories()); // Should include 'test'
Store.removeCustomCategory('test');
console.log(Store.getAllCategories()); // Should NOT include 'test'
```

---

#### Task 3.2: Add CSS for Delete Button in Score Rows

**File:** `index.html`
**Location:** Find `.score-row` CSS (search for it, likely around line 550-580)

**Add this CSS:**
```css
.score-row {
  display: flex;
  align-items: center;
  gap: var(--spacing-sm);
  /* existing styles */
}

.score-row__delete {
  width: 24px;
  height: 24px;
  padding: 0;
  background: none;
  border: none;
  color: var(--color-gray-400);
  cursor: pointer;
  font-size: 14px;
  opacity: 0;
  transition: opacity 0.2s;
}

.score-row:hover .score-row__delete {
  opacity: 1;
}

.score-row__delete:hover {
  color: var(--color-danger);
}
```

---

#### Task 3.3: Update Score Rows to Show Delete Button for Custom Categories

**File:** `index.html`
**Location:** Lines 1458-1468, inside `openPropertyDetail()`, the category score rows

**Current code:**
```javascript
${categories.map(cat => `
  <div class="score-row">
    <span class="score-label">${escapeHtml(CATEGORY_LABELS[cat] || cat)}</span>
    <div class="score-stars">
      ${[1,2,3,4,5].map(n => `
        <button class="score-star ${property.scores[cat] >= n ? 'score-star--active' : ''}"
                onclick="updateScore('${cat}', ${n})">★</button>
      `).join('')}
    </div>
  </div>
`).join('')}
```

**Change to:**
```javascript
${categories.map(cat => {
  const isCustom = !DEFAULT_CATEGORIES.includes(cat);
  const deleteBtn = isCustom
    ? `<button class="score-row__delete" onclick="deleteCategory('${cat}')" title="Verwijder categorie">✕</button>`
    : '';
  return `
    <div class="score-row">
      <span class="score-label">${escapeHtml(CATEGORY_LABELS[cat] || cat)}</span>
      <div class="score-stars">
        ${[1,2,3,4,5].map(n => `
          <button class="score-star ${property.scores[cat] >= n ? 'score-star--active' : ''}"
                  onclick="updateScore('${cat}', ${n})">★</button>
        `).join('')}
      </div>
      ${deleteBtn}
    </div>
  `;
}).join('')}
```

---

#### Task 3.4: Add deleteCategory() Function

**File:** `index.html`
**Location:** Near `addNewCategory()` function (search for it, likely around line 1600-1620)

**Add this function:**
```javascript
function deleteCategory(categoryKey) {
  if (!confirm('Deze categorie verwijderen voor alle woningen?')) return;

  Store.removeCustomCategory(categoryKey);

  // Re-render detail view if open
  if (currentPropertyId) {
    openPropertyDetail(currentPropertyId);
  }

  showToast('Categorie verwijderd');
}
```

**Test:**
1. Open a property detail view
2. Add a custom category "Test"
3. Hover over "Test" row - delete button should appear
4. Click delete - should prompt confirmation
5. After delete, category should disappear from all properties

---

## Issue 4: Status Colors & Icons

### Problem Statement

Status colors need updating to a cleaner, more property-workflow-friendly system with:
- One color per state (no duplicates)
- Strong contrast
- Always an icon
- Subtle effects to draw attention only where it helps

### Required Status Mapping

| Status | Color | Icon | Notes |
|--------|-------|------|-------|
| `nieuw` | Blue | ＋ | Fresh item |
| `bezoek_gepland` | Violet | 📅 | Scheduled |
| `bezocht` | Teal | ✓ | Done step (NOT a decision) |
| `ja` | Green | ★ | Positive decision |
| `misschien` | Amber | ? | Uncertain |
| `nee` | Red | ✕ | Negative decision |
| `onder_optie` | Blue-gray + hatch | ⏳ | Temporarily unavailable |
| `verkocht` | Gray + slash | S | Final, inactive |

### Current State Analysis

**Current issues:**
- `bezocht` uses same green as `ja` - confusing
- No icons on badges
- `onder_optie` and `verkocht` only have visual effects on map markers, not badges

### Files & Line Numbers

| What | Where | Lines |
|------|-------|-------|
| CSS color variables | `:root` | 21-35 |
| Badge CSS | `.badge--*` classes | 341-350 |
| Marker icon CSS | `.marker-icon--*` classes | 273-292 |
| Marker icon creation | `createMarkerIcon()` function | 1327-1348 |
| Badge HTML in cards | `createPropertyCard()` | 1748-1750 |
| Badge HTML in detail | `openPropertyDetail()` | 1437-1448 |

### Implementation Tasks

#### Task 4.1: Add New Color Variables

**File:** `index.html`
**Location:** Lines 21-35, the `:root` CSS variables

**Current code:**
```css
:root {
  /* Colors */
  --color-primary: #2563eb;
  --color-primary-dark: #1d4ed8;
  --color-success: #16a34a;
  --color-warning: #f59e0b;
  --color-danger: #dc2626;
  --color-purple: #9333ea;
  /* gray scale... */
}
```

**Change to:**
```css
:root {
  /* Colors */
  --color-primary: #2563eb;      /* Blue - nieuw */
  --color-primary-dark: #1d4ed8;
  --color-success: #16a34a;      /* Green - ja */
  --color-warning: #f59e0b;      /* Amber - misschien */
  --color-danger: #dc2626;       /* Red - nee */
  --color-purple: #9333ea;       /* Violet - bezoek_gepland */
  --color-teal: #0d9488;         /* Teal - bezocht (NEW) */
  --color-bluegray: #64748b;     /* Blue-gray - onder_optie (NEW) */
  /* gray scale... */
}
```

---

#### Task 4.2: Update Badge CSS with Icons and New Colors

**File:** `index.html`
**Location:** Lines 333-350, the badge CSS

**Current code:**
```css
.badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 9999px;
  font-size: 12px;
  font-weight: 500;
}

.badge--nieuw { background: #dbeafe; color: #1e40af; }
.badge--bezoek_gepland { background: #f3e8ff; color: #6b21a8; }
.badge--bezocht { background: #dcfce7; color: #166534; }
.badge--ja { background: #dcfce7; color: #166534; }
.badge--nee { background: #fee2e2; color: #991b1b; }
.badge--misschien { background: #fef3c7; color: #92400e; }

.badge--beschikbaar { background: #ecfdf5; color: #065f46; }
.badge--onder_optie { background: #fef3c7; color: #92400e; }
.badge--verkocht { background: #f3f4f6; color: #6b7280; }
```

**Change to:**
```css
.badge {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 2px 8px;
  border-radius: 9999px;
  font-size: 12px;
  font-weight: 500;
}

/* Decision statuses */
.badge--nieuw { background: #dbeafe; color: #1e40af; }
.badge--bezoek_gepland { background: #f3e8ff; color: #6b21a8; }
.badge--bezocht { background: #ccfbf1; color: #0f766e; }  /* Teal */
.badge--ja { background: #dcfce7; color: #166534; }       /* Green */
.badge--misschien { background: #fef3c7; color: #92400e; } /* Amber */
.badge--nee { background: #fee2e2; color: #991b1b; }      /* Red */

/* Property statuses */
.badge--beschikbaar { background: #ecfdf5; color: #065f46; }
.badge--onder_optie {
  background: #e2e8f0;
  color: #475569;
  background-image: repeating-linear-gradient(
    45deg,
    transparent,
    transparent 2px,
    rgba(100, 116, 139, 0.15) 2px,
    rgba(100, 116, 139, 0.15) 4px
  );
}
.badge--verkocht {
  background: #f1f5f9;
  color: #94a3b8;
  text-decoration: line-through;
}
```

---

#### Task 4.3: Create STATUS_ICONS Constant

**File:** `index.html`
**Location:** After `STATUS_LABELS` (around line 1126), add new constant

**Add this code:**
```javascript
const STATUS_ICONS = {
  nieuw: '+',
  bezoek_gepland: '📅',
  bezocht: '✓',
  ja: '★',
  misschien: '?',
  nee: '✕'
};

const PROPERTY_STATUS_ICONS = {
  beschikbaar: '✓',
  onder_optie: '⏳',
  verkocht: 'S'
};
```

---

#### Task 4.4: Update Badge Rendering to Include Icons

**File:** `index.html`
**Location:** Lines 1748-1750, inside `createPropertyCard()`

**Current code:**
```javascript
<div class="property-card__badges">
  <span class="badge badge--${property.status}">${STATUS_LABELS[property.status]}</span>
  <span class="badge badge--${property.propertyStatus}">${PROPERTY_STATUS_LABELS[property.propertyStatus]}</span>
</div>
```

**Change to:**
```javascript
<div class="property-card__badges">
  <span class="badge badge--${property.status}">${STATUS_ICONS[property.status]} ${STATUS_LABELS[property.status]}</span>
  <span class="badge badge--${property.propertyStatus}">${PROPERTY_STATUS_ICONS[property.propertyStatus]} ${PROPERTY_STATUS_LABELS[property.propertyStatus]}</span>
</div>
```

---

#### Task 4.5: Update Badge Rendering in Detail View Status Dropdowns

**File:** `index.html`
**Location:** Lines 1437-1448, inside `openPropertyDetail()`, the status select options

The select dropdowns don't need icons (they're dropdowns, not badges). But we could add the current status badge display above or beside each dropdown for visual feedback.

**Optional enhancement** - Add badge preview above dropdowns:

Find the status section (around line 1433-1451) and update to show current badge:

**After line 1433 (`<!-- Status -->`) add:**
```javascript
<div class="detail-section__badges" style="margin-bottom: var(--spacing-sm);">
  <span class="badge badge--${property.status}">${STATUS_ICONS[property.status]} ${STATUS_LABELS[property.status]}</span>
  <span class="badge badge--${property.propertyStatus}">${PROPERTY_STATUS_ICONS[property.propertyStatus]} ${PROPERTY_STATUS_LABELS[property.propertyStatus]}</span>
</div>
```

---

#### Task 4.6: Update Map Marker Icons

**File:** `index.html`
**Location:** Lines 273-292, marker icon CSS

**Current code:**
```css
.marker-icon--nieuw { background: var(--color-primary); }
.marker-icon--bezoek_gepland { background: var(--color-purple); }
.marker-icon--bezocht { background: var(--color-success); }
.marker-icon--ja { background: var(--color-success); }
.marker-icon--nee { background: var(--color-danger); }
.marker-icon--misschien { background: var(--color-warning); }

.marker-icon--onder_optie {
  background-image: repeating-linear-gradient(
    45deg,
    transparent,
    transparent 3px,
    rgba(255,255,255,0.3) 3px,
    rgba(255,255,255,0.3) 6px
  );
}

.marker-icon--verkocht {
  opacity: 0.5;
}
```

**Change to:**
```css
.marker-icon--nieuw { background: var(--color-primary); }
.marker-icon--bezoek_gepland { background: var(--color-purple); }
.marker-icon--bezocht { background: var(--color-teal); }  /* Changed from success to teal */
.marker-icon--ja { background: var(--color-success); }
.marker-icon--nee { background: var(--color-danger); }
.marker-icon--misschien { background: var(--color-warning); }

.marker-icon--onder_optie {
  background: var(--color-bluegray) !important;
  background-image: repeating-linear-gradient(
    45deg,
    transparent,
    transparent 3px,
    rgba(255,255,255,0.3) 3px,
    rgba(255,255,255,0.3) 6px
  ) !important;
}

.marker-icon--verkocht {
  background: var(--color-gray-500) !important;
  opacity: 0.6;
}

/* Slash overlay for verkocht */
.marker-icon--verkocht::after {
  content: '';
  position: absolute;
  top: 50%;
  left: -2px;
  right: -2px;
  height: 2px;
  background: rgba(255,255,255,0.8);
  transform: rotate(-45deg);
}
```

---

#### Task 4.7: Update createMarkerIcon() to Show All Icons

**File:** `index.html`
**Location:** Lines 1327-1348, the `createMarkerIcon()` function

**Current code:**
```javascript
function createMarkerIcon(property) {
  const status = property.status;
  const propertyStatus = property.propertyStatus;

  let symbol = '';
  if (status === 'ja') symbol = '★';
  else if (status === 'nee') symbol = '✕';
  else if (status === 'misschien') symbol = '?';

  let classes = `marker-icon marker-icon--${status}`;
  if (propertyStatus === 'onder_optie') classes += ' marker-icon--onder_optie';
  if (propertyStatus === 'verkocht') classes += ' marker-icon--verkocht';

  return L.divIcon({
    className: 'marker-wrapper',
    html: `<div class="${classes}"><span>${symbol}</span></div>`,
    iconSize: [32, 32],
    iconAnchor: [16, 32],
    popupAnchor: [0, -32]
  });
}
```

**Change to:**
```javascript
function createMarkerIcon(property) {
  const status = property.status;
  const propertyStatus = property.propertyStatus;

  // Always show an icon based on status
  const symbol = STATUS_ICONS[status] || '';

  let classes = `marker-icon marker-icon--${status}`;
  if (propertyStatus === 'onder_optie') classes += ' marker-icon--onder_optie';
  if (propertyStatus === 'verkocht') classes += ' marker-icon--verkocht';

  return L.divIcon({
    className: 'marker-wrapper',
    html: `<div class="${classes}"><span>${symbol}</span></div>`,
    iconSize: [32, 32],
    iconAnchor: [16, 32],
    popupAnchor: [0, -32]
  });
}
```

---

## Testing Guide

### Manual Testing Checklist

#### Issue 1: Property Cards
- [ ] Add property WITHOUT image URL → card shows placeholder (🏠)
- [ ] Add property WITH image URL → card shows actual image on right
- [ ] Image loads lazily (check Network tab)
- [ ] Detail view shows image at top
- [ ] Can edit image URL in detail view
- [ ] Image updates after editing URL
- [ ] URL link in detail view is clickable and opens in new tab

#### Issue 2: Hamburger Menu
- [ ] Test on iOS Safari (real device or BrowserStack)
- [ ] Test on Android Chrome
- [ ] Test on Android Firefox
- [ ] Menu opens on first tap
- [ ] Menu closes when tapping outside
- [ ] Export button works
- [ ] Import button works
- [ ] Visual feedback (tap highlight) appears

#### Issue 3: Custom Categories
- [ ] Add custom category "Test" → appears in score list
- [ ] Default categories (locatie, energie, etc.) → NO delete button
- [ ] Custom categories → delete button visible on hover
- [ ] Delete button click → shows confirmation
- [ ] After delete → category gone from current property
- [ ] After delete → category gone from ALL properties
- [ ] Can still add scores to remaining categories

#### Issue 4: Status Colors
- [ ] `nieuw` badge → Blue + "+"
- [ ] `bezoek_gepland` badge → Violet + 📅
- [ ] `bezocht` badge → Teal + ✓ (NOT green like `ja`)
- [ ] `ja` badge → Green + ★
- [ ] `misschien` badge → Amber + ?
- [ ] `nee` badge → Red + ✕
- [ ] `onder_optie` badge → Blue-gray with hatch pattern + ⏳
- [ ] `verkocht` badge → Gray with strikethrough + S
- [ ] Map markers match badge colors
- [ ] Map markers show icons

### Browser Testing Matrix

| Browser | Desktop | Mobile |
|---------|---------|--------|
| Chrome | ✓ | Android ✓ |
| Safari | ✓ | iOS ✓ |
| Firefox | ✓ | Android ✓ |
| Edge | ✓ | - |

### LocalStorage Testing

After each change, verify data integrity:
```javascript
// In browser console:
const data = JSON.parse(localStorage.getItem('propertyHunter'));
console.log(data.properties[0]); // Check property structure
console.log(data.customCategories); // Check categories
```

---

## Commit Strategy

Make small, focused commits after each task or logical group.

### Suggested Commit Sequence

1. **feat: add imageUrl field to property data model**
   - Task 1.1

2. **feat: add image URL input to add property form**
   - Task 1.2

3. **feat: display property images in cards and detail view**
   - Tasks 1.3, 1.4, 1.5, 1.6

4. **fix: hamburger menu touch handling for iOS/Android**
   - Tasks 2.1, 2.2, 2.3, 2.4

5. **feat: allow deletion of custom categories**
   - Tasks 3.1, 3.2, 3.3, 3.4

6. **style: update status colors and add icons**
   - Tasks 4.1, 4.2, 4.3

7. **style: update badge rendering with icons**
   - Tasks 4.4, 4.5

8. **style: update map marker colors and icons**
   - Tasks 4.6, 4.7

### Commit Message Format

```
<type>: <short description>

- Bullet point of specific change
- Another change
```

Types:
- `feat` - New feature
- `fix` - Bug fix
- `style` - CSS/visual changes
- `refactor` - Code restructuring
- `docs` - Documentation
- `test` - Test additions

---

## Appendix: Quick Reference

### Key Functions to Know

| Function | Purpose | Line |
|----------|---------|------|
| `Store.addProperty()` | Create property | 1186 |
| `Store.updateProperty()` | Modify property | 1207 |
| `Store.getProperty()` | Fetch by ID | 1181 |
| `createPropertyCard()` | Card HTML | 1735 |
| `openPropertyDetail()` | Detail modal | 1408 |
| `renderApp()` | Refresh everything | 1790 |
| `showToast()` | User feedback | ~1680 |

### Key CSS Variables

| Variable | Value | Used For |
|----------|-------|----------|
| `--spacing-sm` | 8px | Small gaps |
| `--spacing-md` | 16px | Standard gaps |
| `--radius-md` | 8px | Border radius |
| `--touch-target` | 44px | Min touch size |

### File Structure (within index.html)

```
Lines 1-12:      HTML head, meta, Leaflet CSS
Lines 13-1006:   <style> block
Lines 1007-1100: HTML body structure
Lines 1101-1140: Script: constants, labels
Lines 1141-1280: Script: Store object
Lines 1281-1400: Script: Map functions
Lines 1401-1700: Script: Property detail
Lines 1701-1800: Script: Cards & rendering
Lines 1801-1930: Script: Filters
Lines 1931-2000: Script: Modals
Lines 2001-2140: Script: Add property flow
Lines 2141-2307: Script: Import/Export, init
```
