# Task 6
Implement Version control.


## Details
We shall have following storage structure for Version control:
- experiment: labelled by the experiment name
- each experiment has a history of versions v0 to vXXX

Storage structure then looks like an Object { "<experiment-name>": []}
Additionally we have to store some metadata to know which Experiment is the current published version for all the parameters.

If there is no Experiment then we whall create a new Experiment. The first Experiment labeled as "Experiment 1". The version is v0.
The user has many options:
- `create new` Create a new Experiment with default values for v0 (Button)
- `create copy` Create a new Experiment with the current set values as v0 (Button)
- `open` open any previously stored experiment - starting off by selecting it. (SelectBox)

- `publish` communicate to the Backend that this Experiment with the current set ot values is to be published (inaccessible if not saved). (Button)
- `save` store the current parameters as a new version in the current Experiment e.g. v1 for the first adjusted Parameters after v0 - is market with a blue dot if there is something to be saved or inaccessible if the current version is saved. (Button)
- `rename` simply edit the current Experiment Input and it shall change the experiment name.(Input)
- `select version` change the values to any previously saved version of that experiment.(SelectBox)
When the user starts an Experiment We shall fill the label with a default name of "Experiment X" where X is the lowset integer number (> 0)  for which the corresponding experiment name does not exist. Depending on the way the user created the Experiment v0 is either the default parameters or the copied Parameters.

Each experiment carries all Parameters:
- Normalization parameters of each Economic area (per area: infl{a,b,c}, mrr{a,b}, gdpg{a,b,c}, cot{f,e})
- DiffCurve parameters (infl{b,d}, mrr{b,d}, gdpg{b,d}, cot{b,d})
- Final combination weights (st{i,l,g,c}, ml{i,l,g,c}, lt{i,l,g,c})

Makrodata (raw values like infl, mrr, gdpg, cot6, cot36) are NOT part of experiments - they come from the backend.

When the user manipulates the makrodata this shall not affect the storage state.
If the user manipulates the parameters then we enable the `save` button + indication that we have unsaved changes.
If the user does not have unsaved changes and the current version is not the published version then we have the publish button available while the save button is disabled.
If we are at the current published version - this shall be indicated by special color of the publish icon - the publish icon is still active but will not trigger any update as there was no change.

In the dropboxes for "open" and "select version" we shall see which version is the currently published version.

## Decisions
- Storage: in-memory only (no localStorage). Experiments are lost on refresh. Backend persistence comes later.
- Publish: send the WebSocket command regardless of whether backend supports it yet. Act as if it always succeeds.
- "Show full ranking" is NOT part of this task - will be a separate task.

## Sub Tasks
- [x] 6.0 Create implementation plan and define sub tasks
- [x] 6.1 ExperimentStore class - data model, CRUD
- [x] 6.2 Snapshot/Apply infrastructure
  - [x] EconomicArea: add setParams(p)
  - [x] ScoringModel: add setDiffParams(p), setFinalWeights(w)
  - [x] playgroundcontroller: add snapshotParams(), applyParams(snapshot)
  - [x] playgroundcontroller: generalParamChanged calls forexscoreversion.onParamsChanged()
  - [x] ExperimentStore: replace dirty flag with baseSnapshot/liveSnapshot comparison, remove listeners
- [ ] 6.3 forexscoreversion UI: pug structure + VersionHandle
- [ ] 6.4 Integration: wire store <> controller <> UI (via forexscoreversion.coffee as coordinator)
- [ ] 6.5 Polish: visual indicators (blue dot, publish state, disabled states)
