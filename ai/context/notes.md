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
```
appcoremodule.setNavState()
  → auth.isAuthenticated()?
    → false: triggers.toAuth()
    → true: proceed to requested state (default: forexscore)
```

## Reference Code for ForexScore Playground
- `currencytrendframemodule` - Contains forex scoring visualization
- `economicareamodule` - Contains economic area scoring logic

## Completed
- [x] Task 0: Project cleanup and orientation
- [x] Removed user-facing modules (account*, summary*, trafficlight*, etc.)
- [x] Rewired navigation (auth, forexscore, usermanagement)
- [x] Created auth stubs (authmodule, authframemodule)
- [x] Fixed all imports and references

## Next Actions
1. Implement admin authentication
2. Implement ForexScore Playground
3. (Later) User Management feature
