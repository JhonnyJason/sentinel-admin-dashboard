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
- Task 1 in progress: Auth shell implementation
- Auth behaviour documented (see authmodule/README.md)
