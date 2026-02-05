# Sentinel Admin Dashboard - Overview

## Purpose
Admin UI for the EWAG Sentinel system. Enables administrators to:
- Secure admin login (different from user auth)
- ForexScore Parameter Playground (v0.1.0 main feature)
- User Management (placeholder for future)

## System Context
Part of the EWAG Sentinel ecosystem:
- **Sentinel Dashboard** - Public user interface
- **Sentinel Access Manager** - Authentication & access control
- **Sentinel Backend** - Makrodata aggregation
- **Sentinel Datahub** - Historic EOD data aggregation
- **Sentinel Admin Dashboard** - This project (admin UI)

## Tech Stack
- CoffeeScript (primary language)
- Pug (templates)
- Stylus (CSS)
- Webpack (bundling)
- Custom "thingy-build-system" toolchain
- PWA architecture (no service worker)

## Project Structure
```
sources/
  source/           # CoffeeScript modules
  page-heads/       # Pug page templates
  ressources/       # Static assets
  patches/          # Build patches
toolset/            # Build system (git submodule)
output/             # Deployment build output
testing/            # Development server root
plan/               # Project planning docs
ai/context/         # AI assistant context files
```

## Key Modules

**Core/Infrastructure:**
- allmodules, appcoremodule, configmodule, debugmodule, utilsmodule

**Auth:**
- authmodule (logic), authframemodule (UI)

**UI/Layout:**
- headermodule, sidenavmodule, contentmodule, uistatemodule

**ForexScore Playground:**
- forexscoreframemodule - Container frame
- forexscoreplayground - Main grid, delegates to playgroundcontroller
- playgroundcontroller - Orchestrates data flow and handle wiring
- economicareamodule - EconomicArea class + area instances
- focuspairselection - Pair selection combobox

**Data/Communication:**
- datamodule - WebSocket data handling
- scimodule - Service Communication Interface

## Navigation States
- `auth` - Not authenticated (shows authframe)
- `forexscore` - Main feature, default after login
- `usermanagement` - Placeholder

## Current State
Phase: Implementation (v0.1.0)
- Task 0-4: Complete
- Task 5: Parameter controls & calculation results (in progress)
  - [x] ScoringModel class
  - [x] Layout restructure (3×5 grid)
  - [x] Results display structure
  - [~] Wiring refactoring (partial - listener cleanup pending)
  - [ ] Normalization cells (Inf, MRR, GDP, COT)
  - [ ] Diff cells
  - [ ] Final wiring & test

## ForexScore Playground Architecture

### Handle Pattern
**Structure (Pug):** Static declarative layout in `components/*.pug`
**Styles (Stylus):** Separated per component in `components/*.styl`
**Logic (Handles):** `*Handle.coffee` classes manage DOM interaction

### Key Classes

| Class | Role |
|-------|------|
| `EconomicArea` | Per-area data + normalization params + score functions |
| `ScoringModel` | Pair-level diff params + weights + calculation |
| `playgroundcontroller` | Orchestrator: original/live areas, handle wiring |
| `MakroDataHandle` | UI for economic area data display/edit |
| `ResultBoxHandle` | UI for ST/MLT/LT score results |

### Data Flow
```
economicareasmodule (backend data)
       │
playgroundcontroller.initialize()
       │ originalAreas ← backend areas
       │ liveAreas ← clones for UI manipulation
       │ scoringModel ← new ScoringModel()
       │
playgroundcontroller.setFocusPair(baseKey, quoteKey)
       │ Wire listeners: controller first, then UI
       │ area.updateData() → onAreaUpdate → isModified + recalculate
       │                   → refreshUI (sees updated state)
       │
       │ Reset: handle.resetButton → resetArea(key) → restore from original
```

## Data Architecture
See `sources/source/datamodule/README.md` for full spec.

**Data Flow:**
- `getAllData` → initial load (makro data + params)
- WebSocket → fine-grained updates
- Admin actions → `saveParams` / `publishParams`

**Parameter Structure:**
- Area params (per economic area): inflation, interest, gdp, cot normalization
- Global params: diff curves + combination weights
