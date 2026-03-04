# Task 6
Implement Version control.

## Details
See original spec in git history. Core design unchanged:
- Experiments with version history, in-memory only
- ExperimentStore class (6.1 ✓), Snapshot/Apply (6.2 ✓), UI skeleton (6.3 ✓)

## Scoring Design Overhaul
The scoring design was significantly reworked (see `sources/source/forexscoreplayground/scoringdesign/`).
The implementation must catch up before version control integration can proceed,
because the snapshot structure (what params exist) must match the new design.

### Parameter Structure (NEW — per experiment snapshot)
- Area params: infl{zeroLow,zeroHigh}, mrr{floor,neutral,ceil,s}, gdpg{zeroLow,zeroHigh}, cot{neutralCOT,e}
- Global params: diffCurves — shape per indicator (b,d derived), factor
- Global params: finalWeights — st{i,l,g,c}, ml{i,l,g,c}, lt{i,l,g,c}

## Sub Tasks — Completed
- [x] 6.0 Create implementation plan
- [x] 6.1 ExperimentStore class
- [x] 6.2 Snapshot/Apply infrastructure
- [x] 6.3 forexscoreversion UI skeleton

## Sub Tasks — Scoring Design Upgrade
Bring implementation in line with scoring-design docs. Each task is independently testable.

- [x] 6.4 Inflation & GDP normalization upgrade
  - EconomicArea: adjsust output range to to [0, 2] (currently only clamps ≥0)
  - normmath: update conversion functions (or inline in EconomicArea)
  - Update default params per area (see inflation-gdp.md tables)

- [x] 6.5 MRR score rewrite
  - EconomicArea: new params {f (floor), n (neutral), c (ceiling), s} replacing {a, b}
  - EconomicArea: new mrrScore() — piecewise linear base + inflation-gated punishment
  - mrrScore needs access to area's own inflation data + inflPeak + inflZeroHigh (cross-indicator)
  - New MrrNormHandle (or repurpose LinNormHandle) for floor/neutral/ceil/s inputs
  - UI: pug structure for the new param layout
  - LinNormHandle: remove or keep only if still used elsewhere
  - Update default params per area (see mrr.md tables)

- [x] 6.6 COT score rewrite
  - EconomicArea: new params {neutralCOT, e} replacing {f, e}
  - EconomicArea: new cotScore() — exponential mapping + weighted geometric mean + projection
  - Formula: cot_norm = 2^((cot - neutralCOT)/50), combined = (c6n * c36n^e)^(1/(1+e)), project to [0,2]
  - CotNormHandle: UI update for neutralCOT input (replacing f), update equation display
  - Update default params per area (see cot.md table)

- [x] 6.7 Diff curve reparametrization
  - ScoreCombinator: parametrize by shape ∈ [0,1] instead of free b,d
  - Derivation: d = shape * 0.625, b = 2.5 * (1 - shape). Constraint: f(2) = 5.
  - DiffHandle: UI input becomes shape (single input instead of b + d)
  - Update defaults: all shape=0.50

- [x] 6.8 Final combination formula upgrade
  - ScoreCombinator: add factor param (default 13)
  - ScoreCombinator.calcFinal: `factor * Σ(w*score) / Σ(w)` instead of direct `Σ(w*score)`
  - ResultBoxHandle: display factor, possibly add factor input
  - Update default weights (ST:{14,28,8,51}, ML:{8,8,4,5}, LT:{8,5,7,1})

- [x] 6.9 Snapshot structure alignment
  - playgroundcontroller.snapshotParams(): output new param structure
  - playgroundcontroller.applyParams(): accept new param structure
  - EconomicArea.copyParams/setParams: use new param shapes
  - ScoreCombinator.setDiffParams/setFinalWeights: use new param shapes
  - ExperimentStore: defaultSnapshot() with correct new defaults

## Sub Tasks — Version Control Wiring (after scoring upgrade)
- [x] 6.10 refreshUI() in forexscoreversion — central UI-update function
- [x] 6.11 downSyncExperimentStore(null) — bootstrap first experiment
- [x] 6.12 Event handlers — createNew, createCopy, save, publish, open, selectVersion, rename
- [x] 6.13 onParamsChanged completion — updateLiveSnapshot + refreshUI
- [x] 6.14 Wire remote API for history entries (datamodule + forexscoreversion)
