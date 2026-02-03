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
  source/           # CoffeeScript modules (25 modules)
  page-heads/       # Pug page templates
  ressources/       # Static assets
  patches/          # Build patches
toolset/            # Build system (git submodule)
output/             # Deployment build output
testing/            # Development server root
plan/               # Project planning docs
```

## Source Modules (25 total)

**Core/Infrastructure:**
- allmodules (auto-generated), appcoremodule, configmodule
- debugmodule, domconnect, index, utilsmodule

**Auth:**
- authmodule (logic: entry state detection, key management, crypto ops)
- authframemodule (UI: setup/unlock/inaccessible views)
- See `sources/source/authmodule/README.md` for detailed auth flow

**UI/Layout:**
- headermodule, sidenavmodule, contentmodule
- globalstyles, stylereset, svgiconsmodule, buttonstylemodule, uistatemodule

**Features:**
- forexscoreframemodule - ForexScore Playground (main feature)
- usermanagementmodule - User Management (placeholder)
- placeholderframemodule - Generic placeholder UI

**Data/Communication:**
- datamodule - WebSocket data handling
- scimodule - Service Communication Interface (API)
- navtriggers - Navigation routing

**Reference Code (forex scoring):**
- currencytrendframemodule
- economicareamodule

## Navigation States
- `auth` - Not authenticated (shows authframe)
- `forexscore` - Main feature, default after login
- `usermanagement` - Placeholder

## Current State
Phase: Implementation (v0.1.0)
- Task 0 complete: Project cleanup done
- Task 1 complete: Auth implementation
- Task 2 complete: FocusPair search (combobox with 28 pairs)
- Task 3 complete: FocusPair display (basic)
- Task 4 complete: Makrodata manipulation
- Task 5 in progress: Parameter controls & calculation results
  - [x] Step 1: Create scoringmodule (calculation engine)
  - [x] Step 2: Layout restructure (3×5 grid)
  - [x] Step 3: Results display (ST/MLT/LT)
  - [ ] Step 4a: Inf + GDP norm cells (quadratic)
  - [ ] Step 4b: MRR norm cells (linear)
  - [ ] Step 4c: COT norm cells (f factor)
  - [ ] Step 4d: Inf + GDP diff cells
  - [ ] Step 4e: MRR + COT diff cells
  - [ ] Step 5: Final wiring & test

## Data Architecture (ForexScore Playground)
See `sources/source/datamodule/README.md` for full spec.

**Data Flow:**
- `getAllData` → initial load (makro data + params)
- WebSocket → fine-grained updates
- Admin actions → `saveParams` (experimental) / `publishParams` (checkpoint)

**Parameter Structure:**
- Area params (per economic area): inflation, interest, gdp, cot normalization
- Global params: diff curves + combination weights

**Defaults:**
- Neutral defaults → UI config only (calculated balanced values)
- Last published → server-side checkpoint for reset
