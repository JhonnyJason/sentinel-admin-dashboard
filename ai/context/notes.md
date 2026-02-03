# Operational Notes

## Build Commands
- `pnpm test` - Start dev server with watch mode
- `pnpm run deployment-build` - Create production build
- `pnpm run check-deployment` - Build and serve deployment locally

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

**Note:** UI states and navtriggers are for *navigatable* UI states only. Internal states of smaller UI parts (like auth flow states within authframe) are handled internally by the specific frame and are not navigatable unless explicitly needed.

## Auth Flow
See `sources/source/authmodule/README.md` for full documentation.

**Entry States (on app start):**
| Condition | Action |
|-----------|--------|
| URL has `?otc=<32 hex>` | → Key Setup |
| No otc, has localStorage `key-info` | → Key Unlock |
| No otc, no key-info | → "Not Accessible" |

**Key Setup:** PIN → recover secret → generate curve25519 key → POST /registerAdmin (stubbed) → QR scan → lock & store
**Key Unlock:** read key-info → QR scan (with retries) → XOR unlock → validate via targetHash → proceed

**localStorage `key-info`:**
```
{ lockedKey, targetHash, salt, lockType: "qr" }
```

**Config needed:** pwdSalt (fixed constant for secret recovery)

## Reference Code for ForexScore Playground
- `currencytrendframemodule` - Contains forex scoring visualization
- `economicareamodule` - Contains economic area scoring logic
- `scorehelper.coffee` - Diff curves, color/trend mapping

## ForexScore Playground Key Files
- `forexscoreframemodule/focuspairmodule.coffee` - FocusPair display management
- `configmodule.coffee` - Mock data, neutral defaults, currency→area mapping
- `datamodule/README.md` - Data contract specification

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
- [x] Task 4 Step 1: Revive economicareasmodule
  - Added `initialize()`, `createAllAreas()` with 8 areas
  - Added accessors: `getArea(key)`, `getAllAreas()`, `updateAllAreas(data)`
  - EconomicArea class has `getData()`, `getParams()`, `getInfo()` for cloning
- [x] Task 4 Step 2: Wire datamodule → economicareasmodule
  - datamodule imports economicareasmodule
  - Added `loadMockData()` → calls `areas.updateAllAreas(cfg.mockAreaData)`
  - heartbeat triggers `loadMockData()` when socket OPEN and !dataReceived
  - Real backend flow prepared (commented) for when backend supports protocol
  - economicareasmodule.updateData() handles missing meta fields gracefully
- [x] Task 4 Step 3: focuspairmodule reads from economicareasmodule
- [x] Task 4 Step 4: Data manipulation implemented
  - `workingData` / `realData` state tracking per area (base/quote)
  - Input fields replace static spans (styled invisible, number type)
  - `.modified` class on inputs when value differs from real
  - Reset button per area (hidden until modifications exist)
  - Styles split: combobox.styl, focuspair.styl imported into styles.styl

## Task 5: Parameter Controls & Calculation Results

**Parameter Model:**

Quadratic normalization (Inflation, GDP):
- User inputs: `peak` + `steepness` (steepness=1.0 is default)
- Feedback display: `zeroLow`, `zeroHigh` (derived)
- Internal: `a, b, c` coefficients
- Math: width = default_width / steepness, then derive a,b,c

Linear normalization (Interest/MRR):
- User inputs: `neutralRate` + `sensitivity`
- Internal: `a = -sensitivity * neutralRate`, `b = sensitivity`

COT: just `f` factor

**Layout (3 columns × 5 rows):**
- Row 1: Base Area | Quote Area | Final Results (ST/MLT/LT)
- Row 2: Inf Norm (base) | Inf Norm (quote) | Inf Diff
- Row 3: MRR Norm (base) | MRR Norm (quote) | MRR Diff
- Row 4: GDP Norm (base) | GDP Norm (quote) | GDP Diff
- Row 5: COT Norm (base) | COT Norm (quote) | COT Diff

**Colors:** Inf=blue, MRR=purple, GDP=green, COT=yellow

**Implementation Steps:**
1. [x] scoringmodule - calculation engine
2. [x] Layout restructure (CSS grid)
3. [x] Results display (Column 3, Row 1)
4. Normalization & Diff rows (split for testability):
   - [ ] 4a: Inf + GDP norm cells (cols 1-2) - quadratic: peak + steepness
   - [ ] 4b: MRR norm cells (cols 1-2) - linear: neutralRate + sensitivity
   - [ ] 4c: COT norm cells (cols 1-2) - f factor
   - [ ] 4d: Inf + GDP diff cells (col 3) - b + d curve params
   - [ ] 4e: MRR + COT diff cells (col 3) - b + d curve params
5. Final wiring & test

## Next Actions
1. Implement Step 4a: Inf + GDP normalization cells
2. (Later) Wire up actual backend calls
3. (Later) User Management feature
