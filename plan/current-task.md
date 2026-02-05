# Task 5
Implement ForexScore Playground - FocusPair display parameter controls and calculation results

# Details
We want to build the parameter manipulation for the ForexScore Playground.
For convenient experimentation we want to see updated results immediately.
On one side we have the makrodata + area-specific normalization parameters stored in the EconomicArea classes. On the other side we have the ScoringModel managing global parameters (diff curves, weights).

## Sub-Tasks
- [x] Reflect and clarify requirements and plan implementation
- [x] Step 1: Create ScoringModel (calculation engine)
- [x] Step 2: Layout restructure (3×5 CSS grid)
- [x] Step 3: Results display (ST/MLT/LT in Column 3, Row 1)
- [~] Step 3.5: Wiring refactoring (PARTIAL - see blocked issue below)
  - [x] ScoringModel created in `playgroundcontroller/`
  - [x] playgroundcontroller basic implementation
  - [x] EconomicArea.isModified property added
  - [x] forexscoreplayground delegates to playgroundcontroller
  - [ ] **BLOCKED: Listener cleanup on pair switch**
- [ ] Step 4a: Inf + GDP normalization cells (cols 1-2, quadratic: peak + steepness)
- [ ] Step 4b: MRR normalization cells (cols 1-2, linear: neutralRate + sensitivity)
- [ ] Step 4c: COT normalization cells (cols 1-2, f factor)
- [ ] Step 4d: Inf + GDP diff cells (col 3, b + d curve params)
- [ ] Step 4e: MRR + COT diff cells (col 3, b + d curve params)
- [ ] Step 5: Final wiring & test

## BLOCKED: Listener Cleanup Issue

When switching pairs, listeners accumulate without cleanup. See `ai/context/notes.md` for full documentation of the problem and proposed solution.

**Summary:** Need to track and remove old listeners in:
1. `playgroundcontroller.setFocusPair` - area update listeners
2. `MakroDataHandle.setArea` - refreshUI listener
3. `MakroDataHandle` - reset listeners
4. `ResultBoxHandle.setModel` - refreshUI listener

## Parameter Model (decided)

**Quadratic (Inflation, GDP):**
- Input: `peak` + `steepness` (1.0 = default)
- Display: `zeroLow`, `zeroHigh` as feedback
- Internal: `a, b, c` coefficients

**Linear (Interest/MRR):**
- Input: `neutralRate` + `sensitivity`
- Internal: `a`, `b`

**COT:** `f` factor and `e` exponent
- cot = f * (cot6 * cot36^e)

## Layout
```
┌─────────────────────────────────────────────────────────────┐
│  [Search Combobox]                                          │
├───────────────────┬───────────────────┬─────────────────────┤
│ Row 1: Base Area  │ Quote Area        │ Results (ST/MLT/LT) │
├───────────────────┼───────────────────┼─────────────────────┤
│ Row 2: Infl Norm  │ Infl Norm         │ Infl Diff           │
├───────────────────┼───────────────────┼─────────────────────┤
│ Row 3: MRR Norm   │ MRR Norm          │ MRR Diff            │
├───────────────────┼───────────────────┼─────────────────────┤
│ Row 4: GDPG Norm  │ GDPG Norm         │ GDPG Diff           │
├───────────────────┼───────────────────┼─────────────────────┤
│ Row 5: COT Norm   │ COT Norm          │ COT Diff            │
└───────────────────┴───────────────────┴─────────────────────┘
```

**Colors:** infl=blue, mrr=purple, gdpg=green, cot=yellow

## Files Changed in This Session

- `economicareamodule/EconomicArea.coffee` - added `@isModified = false`
- `playgroundcontroller/playgroundcontroller.coffee` - full implementation
- `forexscoreplayground/forexscoreplayground.coffee` - simplified to delegate to controller
