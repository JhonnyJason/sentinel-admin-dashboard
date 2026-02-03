# Task 3
Implement ForexScore Playground - FocusPair display

## Details
What is necessary and helpful for the scores is that we display the current data.
We have the elements with their style and function from the customer facing dashboard available in summaryframe.styl, summaryframe.pug and summaryframemodule.coffee for reference.

We should display the chosen FocusPair as 2 of these Makrodata summaries, side by side right beneath the search field.

## Sub-Tasks
- [x] check out the summaryframe code and document what dataconnection we require
- [x] create implementation plan / data contract spec (`datamodule/README.md`)
- [x] implement the element display for the Focus Pair (`focuspairmodule.coffee`)
- [x] implement the wiring for the data (mock data in config, admin stubs)
- [x] test and fix issues

## Implementation Notes
- Data contract defined in `sources/source/datamodule/README.md`
- Mock area data with params in `configmodule.coffee`
- Simplified display (no info popups) - later will add "thought" value inputs
- Admin actions: `saveParams` (experimental) vs `publishParams` (checkpoint)
- Neutral defaults stored in UI config only (not server-relevant)