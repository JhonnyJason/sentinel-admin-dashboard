# Operational Notes

## Build Commands
- `pnpm test` - Start dev server with watch mode
- `pnpm run deployment-build` - Create production build
- `pnpm run check-deployment` - Build and serve deployment locally

## Build System Note
All CoffeeScript files are transpiled into a single directory. This means `./modulename.js` imports work regardless of source folder structure.

## Key Files
- `sources/source/allmodules/allmodules.coffee` - Module registry (auto-generated)
- `sources/source/configmodule/configmodule.coffee` - App configuration
- `sources/source/appcoremodule/appcoremodule.coffee` - App initialization & nav state handling
- `sources/page-heads/index/` - Main page template

## App Lifecycle
- `initialize()` - Pre-app-start setup (module registration, DOM caching)
- `setNavState()` - Called on app load and nav changes (real app-start logic happens here)

## Module Pattern
Each module folder typically contains:
- `<modulename>.coffee` - Logic
- `<modulename>.styl` - Styles (optional)
- `<modulename>.pug` - Template (optional)

## Navigation States
| State | Content | Sidenav | Authframe | Header |
|-------|---------|---------|-----------|--------|
| `auth` | hidden | hidden | visible | default |
| `forexscore` | forexscore | forexscore | hidden | logged-in |
| `usermanagement` | usermanagement | usermanagement | hidden | logged-in |

## Auth Flow
See `sources/source/authmodule/README.md` for full documentation.

## Scoring Pipeline (Reference)
1. Raw makro data → area normalization (per-area params)
2. Normalized scores → diff calculation (base - quote)
3. Diff → diff curves (global params)
4. Individual scores → weighted combination (global weights)

## Completed
- [x] Task 0: Project cleanup and orientation
- [x] Task 1: Auth implementation
- [x] Task 2: ForexScore Playground - FocusPair search
- [x] Task 3: ForexScore Playground - FocusPair display (basic)
- [x] Task 4: Makrodata manipulation
- [x] Task 5: Parameter Controls & Calculation Results

## Task 6: Version Control (Experiments)
See `plan/current-task.md` for full details.
See `sources/source/forexscoreversion/README.md` for architecture.

### Key Design Decisions (6.2)
- **No dirty flag** — ExperimentStore keeps baseSnapshot + liveSnapshot, isModified() compares them
- **No listeners on store** — forexscoreversion.coffee is the single coordinator (calls store, calls controller, updates UI directly)
- **forexscoreversion.coffee** is the hub: store ↔ controller ↔ UI all flow through it
- **playgroundcontroller** gets snapshotParams() and applyParams(snapshot) — the bridge between version control and live playground state
- **generalParamChanged** → calls forexscoreversion.onParamsChanged() (currently only norm params; diff/weight params deferred)

## ForexScore Playground Architecture

### Key Classes

| Class | Location | Role |
|-------|----------|------|
| `EconomicArea` | economicareamodule/ | Per-area: data + params + score functions. Has updateListeners + isModified. |
| `ScoringModel` | playgroundcontroller/ | Pair-level: diff params + weights + calculation. Has updateListeners. |
| `playgroundcontroller` | playgroundcontroller/ | Orchestrator: original/live areas, wires handles, triggers recalc. |

### Data Flow

```
economicareasmodule (original backend data)
       │
       ▼
playgroundcontroller.initialize()
       │ store originals, clone to liveAreas
       │ create ScoringModel
       ▼
playgroundcontroller.setFocusPair(baseKey, quoteKey)
       │ add controller listener to liveArea (FIRST)
       │ call handle.setArea(liveArea) → adds UI listener (SECOND)
       │ add reset listener to handle
       │ wire scoringModel to resultBoxHandle
       ▼
User edits data → area.updateData() → listeners fire in order:
  1. Controller: compare to original, set isModified, recalculate
  2. UI: refreshUI sees updated isModified, updates display
```

### Handle Classes
- `MakroDataHandle.coffee` - implemented, wired via playgroundcontroller
- `ResultBoxHandle.coffee` - implemented, wired to ScoringModel
- `QuadNormHandle.coffee` - implemented (pug equation + input wiring + refreshUI)
- `LinNormHandle.coffee` - implemented (neutralRate + sensitivity inputs, equation display)
- `CotNormHandle.coffee` - implemented (f + e inputs, equation n=f·c6·c36^e, COT Faktoren feedback)
- `DiffHandle.coffee` - skeleton
- `uihandles.coffee` - instantiates all handles

### Utility Files
- `normmath.coffee` - Pure math for param conversions (peak/steepness ↔ a,b,c; neutralRate/sensitivity ↔ a,b; defaultWidths)

---

## Deprecated Modules
- `forexscoreframemodule/scoringmodule.coffee` - Conversion functions moved to `forexscoreplayground/normmath.coffee`. The scoringmodule and focuspairmodule in forexscoreframemodule are leftovers from a prior refactoring and should not be imported from.

## Next Actions
1. Task 6: Version Control (sub-tasks 6.1-6.5)
2. (Later) Show full ranking feature
3. (Later) Wire up actual backend calls
4. (Later) User Management feature
