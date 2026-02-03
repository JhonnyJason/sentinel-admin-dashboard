# Task 5
Implement ForexScore Playground - FocusPair display parameter controls and calculation results


# Details
We need a good visualizaion for the current parameters. Therefore we need to full picture and the full pipline what needs to be shown. Probably we need som sort of 3 Column layout while the top with the search combobox is outside of that.
So currently we have 2 elements. Naturally they are row 1 and column 1 and  column 2 respectively. Column 3 is emtpy - Here we can add the Endresults of the current calculation.

It would be great if the Endresult looks similar to the results in currencytrendframemodule.
Also all the inspiration how it is calculated shall com from there.

Currently we only have the stWeights applied. They are for short term results. For real we also want to apply different weights for middle longterm results (mlWeights) and long-term results (ltWeights).
We want to have the 3 result Elements in the element on row 1 and column 3. If we have space we may add on top of each element the calculation for it where we see the weights as number and the InfScore, mrrScore, gdpScore and cotScore as uneditable fields with ther color as background simply being their respective end-results.

We need colors that symbolize each of those 4 parts. maybe some yellowish for the cotScore, some purpoe for mrrScore some blueish for infScore and some greenish for gdpScore.

in Row 2 we need the normalization functions. Here we can take the smaller elements in height where each is dealing with one of the values.
Naturally column 1 is dealing with the normalizations regarding the baseArea while column 2 is is dealing with the normalization regarding the quoteArea. For most this function is a polynomial of second order and a negative one. To be more intuitive, we want to be able to control the location for the peak  and steepness of the function.
As easy visualization we display the value for lowZero and the highZero.
As defined in the calcuation and scoring-design.md the results are to be between 0-3 - probably we should upgrade this to be in between 0-2 As a normalized result.
The normalized result for the specific value is displayed as constant uneditable number in a similar style as the other uneditable endresult (backgroud color).

For each of those Rows the 3rd column display the diffFunction where the global Parameters for the diffFunction are to be edited, the inputs from the basArea and quoteArea are similar uneditable constants with the respective background color. And also the result is such a uneditable constant there.

To be limited in with for a number we should always calculate them "toFixed(2)" for display.


## Sub-Task
- [x] reflect and clarify requirements and plan implementation
- [x] Step 1: Create scoringmodule (calculation engine)
- [x] Step 2: Layout restructure (3×5 CSS grid)
- [x] Step 3: Results display (ST/MLT/LT in Column 3, Row 1)
- [ ] Step 4a: Inf + GDP normalization cells (cols 1-2, quadratic: peak + steepness)
- [ ] Step 4b: MRR normalization cells (cols 1-2, linear: neutralRate + sensitivity)
- [ ] Step 4c: COT normalization cells (cols 1-2, f factor)
- [ ] Step 4d: Inf + GDP diff cells (col 3, b + d curve params)
- [ ] Step 4e: MRR + COT diff cells (col 3, b + d curve params)
- [ ] Step 5: Final wiring & test

## Parameter Model (decided)

**Quadratic (Inflation, GDP):**
- Input: `peak` + `steepness` (1.0 = default)
- Display: `zeroLow`, `zeroHigh` as feedback
- Internal: `a, b, c` coefficients

**Linear (Interest/MRR):**
- Input: `neutralRate` + `sensitivity`
- Internal: `a`, `b`

**COT:** `f` factor only

## Layout
```
┌─────────────────────────────────────────────────────────────┐
│  [Search Combobox]                                          │
├───────────────────┬───────────────────┬─────────────────────┤
│ Row 1: Base Area  │ Quote Area        │ Results (ST/MLT/LT) │
├───────────────────┼───────────────────┼─────────────────────┤
│ Row 2: Inf Norm   │ Inf Norm          │ Inf Diff            │
├───────────────────┼───────────────────┼─────────────────────┤
│ Row 3: MRR Norm   │ MRR Norm          │ MRR Diff            │
├───────────────────┼───────────────────┼─────────────────────┤
│ Row 4: GDP Norm   │ GDP Norm          │ GDP Diff            │
├───────────────────┼───────────────────┼─────────────────────┤
│ Row 5: COT Norm   │ COT Norm          │ COT Diff            │
└───────────────────┴───────────────────┴─────────────────────┘
```

**Colors:** Inf=blue, MRR=purple, GDP=green, COT=yellow
