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

## Task 5: Parameter Controls & Calculation Results

**Parameter Model:**

Quadratic normalization (Inflation, GDP):
- User inputs: `peak` + `steepness` (steepness=1.0 is default)
- Feedback display: `zeroLow`, `zeroHigh` (derived)
- Internal: `a, b, c` coefficients

Linear normalization (Interest/MRR):
- User inputs: `neutralRate` + `sensitivity`
- Internal: `a = -sensitivity * neutralRate`, `b = sensitivity`

COT: `f` factor + `e` exponent

**Layout (3 columns × 5 rows):**
- Row 1: Base Area | Quote Area | Final Results (ST/MLT/LT)
- Row 2-5: Norm cells (cols 1-2) | Diff cells (col 3)

**Colors:** Inf=blue, MRR=purple, GDP=green, COT=yellow

**Implementation Steps:**
1. [x] ScoringModel class
2. [x] Layout restructure (CSS grid)
3. [x] Results display structure
4. [~] **Step 3.5: Wiring refactoring** ← CURRENT (partial)
   - [x] ScoringModel created
   - [x] playgroundcontroller implemented (basic)
   - [x] EconomicArea.isModified property added
   - [ ] **BLOCKED: Listener cleanup issue** (see below)
5. Normalization & Diff rows (pending)
6. Final wiring & test

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
- `LinNormHandle.coffee` - skeleton
- `CotNormHandle.coffee` - skeleton
- `DiffHandle.coffee` - skeleton
- `uihandles.coffee` - instantiates all handles

### Utility Files
- `normmath.coffee` - Pure math for param conversions (peak/steepness ↔ a,b,c; neutralRate/sensitivity ↔ a,b; defaultWidths)

---

## KNOWN ISSUE: Listener Accumulation on Pair Switch

### Problem
When `setFocusPair` is called multiple times (user switches pairs), listeners accumulate without cleanup:

| Location | What's Added | Added To |
|----------|--------------|----------|
| `playgroundcontroller.setFocusPair` | `-> onAreaUpdate(key)` | liveArea (EconomicArea) |
| `MakroDataHandle.setArea` | `@refreshUI` | area (EconomicArea) |
| `playgroundcontroller.setFocusPair` | `-> resetArea(key)` | handle (MakroDataHandle) |
| `ResultBoxHandle.setModel` | `@refreshUI` | scoringModel |

**Scenario:** User selects EURUSD, then EURJPY
- EUR liveArea gets duplicate `onAreaUpdate` listeners
- EUR liveArea gets duplicate `refreshUI` listeners
- baseAreaHandle gets duplicate reset listeners

### Solution Approach (TODO)

**1. playgroundcontroller** - track and cleanup:
```coffee
prevBaseKey = null
prevQuoteKey = null
baseListener = null   # store function reference
quoteListener = null

# In setFocusPair, before adding new:
if prevBaseKey and baseListener
    liveAreas[prevBaseKey]?.removeUpdateListener(baseListener)
# ... then create new listener, store reference, add it
```

**2. MakroDataHandle.setArea** - cleanup old listener:
```coffee
# Store @areaListener = @refreshUI bound
# If @area exists, @area.removeUpdateListener(@areaListener)
# Then add new listener
```

**3. MakroDataHandle reset listeners** - clear on setArea:
```coffee
@resetListeners = []  # in setArea, before adding new
```

**4. ResultBoxHandle.setModel** - same pattern as MakroDataHandle

### Status
Documented for later. Current implementation works for single pair selection but will have issues if user switches pairs frequently.

---

## Deprecated Modules
- `forexscoreframemodule/scoringmodule.coffee` - Conversion functions (peakSteepnessToCoeffs etc.) moved to `forexscoreplayground/normmath.coffee`. The scoringmodule and focuspairmodule in forexscoreframemodule are leftovers from a prior refactoring and should not be imported from.

## Next Actions
1. **Fix listener cleanup issue** (documented above)
2. Complete skeleton Handle classes (LinNorm, CotNorm, Diff)
3. Wire QuadNormHandle into playgroundcontroller
4. Test full wiring flow
5. (Later) Wire up actual backend calls
6. (Later) User Management feature
