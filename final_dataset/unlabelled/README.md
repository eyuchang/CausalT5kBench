# Unlabelled Cases

This folder contains cases that have been removed from the main `final_dataset` because they do not meet the dataset requirements.

## NO_cases_missing_traps.json

This file contains **332 NO cases** that were removed from the final dataset because they are missing trap type information.

### Requirements
According to the dataset rules, all NO cases (at L1, L2, and L3 levels) must have exactly one trap type specified. These cases were identified as NO cases but did not have trap information.

### Distribution
- **Total cases**: 332
- **By Pearl Level**:
  - L1: 42 cases
  - L2: 240 cases
  - L3: 50 cases
- **By Domain**:
  - Medicine & Health (D4): 332 cases (all cases)

### Next Steps
These cases need manual review to:
1. Identify the appropriate trap type for each case
2. Add the trap information to the case
3. Move the cases back to the appropriate domain files in `final_dataset/`

### Case Format
Each case in this file maintains the same structure as cases in `final_dataset/`, but with the `trap` field set to `null` or an empty object.

