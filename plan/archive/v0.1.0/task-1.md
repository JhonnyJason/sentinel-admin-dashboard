# Task 1
Implement shell for authentication behaviours.


## Details
To For the purpose of proceedeing faster to implement the forexscore playground, we only need to implement the shell and core necessities for the Authentication behaviour for now.
Such that we may go through the states appropriately


## Sub-Task
- [x] Reflect on and document desired authentication behaviour.
- [x] Implement top-level APIs towards the rest of the application to be usable with the correct Flow inside the UI
- [x] Implement core necessities - e.g. locked key storage etc. such that we may distinguish the auth-states based on actual "real" state.

## Notes
- Server call (`registerAdmin`) is stubbed pending backend implementation
- PIN validation requires 4 digits
- QR reading uses `readWithRetries` helper for robustness 
