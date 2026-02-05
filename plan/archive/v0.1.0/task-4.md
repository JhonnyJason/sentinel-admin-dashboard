# Task 4
Implement manipulation of makrodata on FocusPair

## Details
We always have a FocusPair on display - two elements where each represents an economic area containing the makrodata.
These makrodata originally come from the backend (mocked for now) and are the current "real" data. However for experimenting we might want to test different makroconditions for this purpose we want to be able to manipulate each value.
We only keep them temporarily while we experiment with one focus pair. If we select a different focuspair, we start off with the "real" values.
Also we want some indication to know which values are currently manipulated.
And last but not least we want to be able to reset those values to their current "real" value.

---

## Prerequisite: Data Pipeline Fix

Before implementing manipulation, we need to establish the correct data flow architecture.

### Problem
Currently `focuspairmodule` imports `mockAreaData` directly from `configmodule`, bypassing the natural data pipeline. This needs correction.

### Target Architecture
```
Backend (WebSocket)
       ↓
datamodule (receives allData, relays to consumers)
       ↓
economicareasmodule (stores all 8 EconomicArea objects)
       ↓
focuspairmodule (reads "real" data, clones for volatile experiments)
```

### Admin Dashboard Data Flow Characteristics
- **One-time read**: Data arrives once on connection (mocked for now)
- **Write-only after**: After initial population, admin only pushes changes (params, history queries)
- **No live updates**: Unlike user dashboard, no ongoing data stream from backend

### EconomicArea Pattern
The `EconomicArea` class maintains:
- Data storage (`@data`, `@metaData`)
- Normalization parameters (`@inflParams`, etc.)
- Internal DOM element (always updated, but detached — can be mounted or ignored)
- Scoring/normalization functions

This allows:
- User dashboard: mounts `area.getElement()` directly
- Admin dashboard: reads `area.copyData()` / `area.copyParams()`, renders custom UI

---

## Implementation Plan

### Step 1: Revive economicareasmodule
- Uncomment the 8 area definitions
- Add `getArea(key)` accessor
- Add `updateAllAreas(data)` function for bulk population
- Ensure areas work with detached DOM (no dependency on being mounted)

### Step 2: Wire datamodule → economicareasmodule
- On mock "connection": call `updateAllAreas(mockAreaData)`
- After population: call `focuspairmodule.populateCurrentFocusPair()`
- This ensures default FocusPair displays with real data

### Step 3: Update focuspairmodule
- Remove direct import of `mockAreaData` from configmodule
- Import from `economicareasmodule.getArea(key)` instead
- Add `populateCurrentFocusPair()` function (called after data arrives)
- Clone area data for volatile experiment state

### Step 4: Implement manipulation (original Task 4)
- Working data state (cloned from real)
- Input fields for editing values
- Visual indication of modified values (CSS class)
- Reset button per area

---

## Sub-Tasks
- [x] Reflect and plan how we should implement the manipulation of these values
- [x] Step 1: Revive economicareasmodule
- [x] Step 2: Wire datamodule → economicareasmodule
- [x] Step 3: Update focuspairmodule to use new data flow
- [x] Step 4a: Implement manipulation (input fields, working state)
- [x] Step 4b: Make manipulated values visible (CSS indication)
- [x] Step 4c: Implement reset functionality
- [x] Test and fix issues
