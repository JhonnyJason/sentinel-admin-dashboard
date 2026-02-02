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

## Completed
- [x] Task 0: Project cleanup and orientation
- [x] Removed user-facing modules (account*, summary*, trafficlight*, etc.)
- [x] Rewired navigation (auth, forexscore, usermanagement)
- [x] Created auth stubs (authmodule, authframemodule)
- [x] Fixed all imports and references
- [x] Task 1: Auth implementation
  - Auth behaviour documented
  - Entry state detection & module API
  - Authframe internal views (keySetup, keyLocked, inaccessible)
  - Key Setup flow (server call stubbed)
  - Key Unlock flow with retry logic

## Next Actions
1. Implement ForexScore Playground
2. (Later) User Management feature
3. (Later) Wire up actual server calls when backend ready
