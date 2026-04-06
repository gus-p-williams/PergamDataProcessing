# Pergam Data Processing Status

## Summary

`PergramV2.ipynb` now implements the full planned processing workflow and the major review findings have been addressed in code.

Implemented end-to-end:
- unified CSV/XLSX reading,
- elapsed-time normalization,
- flight detection with positive-altitude hysteresis,
- flight-label filling with explicit CSV policy,
- per-flight 3D GPS drift correction,
- ground-coordinate computation with explicit on-ground policy,
- directory-wide processing,
- validation summaries,
- notebook configuration and plot-saving cleanup.

## Final Design Decisions

### Flight detection

Implemented behavior:
- Use GPS `Altitude` as the flight-boundary signal.
- Smooth altitude with a rolling window.
- Detect takeoff only after a sustained run above the takeoff threshold.
- Detect landing at the first sustained run below the landing threshold.
- Store flight ranges as half-open intervals: `(start_row, end_row)`.

This resolves:
- landing-row off-by-one labeling,
- false trailing takeoffs caused by post-flight negative GPS drift,
- mismatch between flight labeling and downstream drift-correction ranges.

### `Names of Flights` policy

Implemented behavior:
- XLSX labels are forward-filled within each detected flight.
- Rows before the first native XLSX label within a flight are filled as `"Pre-label"`.
- CSV files keep unlabeled in-flight rows empty.
- On-ground rows are labeled `"On Ground"`.

This resolves:
- the earlier mismatch between the draft plan and notebook behavior for CSV-derived rows,
- the pandas dtype warning path by normalizing the label column before assignment.

### Ground-coordinate policy

Implemented behavior:
- `ground_lat`, `ground_lon`, `ground_alt_est`, and `horizontal_offset_m` are emitted only for in-flight rows.
- `On Ground` rows remain null for those derived ground-coordinate columns.

This resolves:
- misleading ground-coordinate outputs on ground rows,
- negative-looking on-ground `ground_alt_est` artifacts.

### Plot output policy

Implemented behavior:
- plot saving is disabled by default,
- plots are only saved when explicitly enabled,
- saved plot filenames are file-specific rather than fixed shared names.

## Current Pipeline Status

### Step 1 - Unified File Reader

Status: Implemented

Current behavior:
- Reads `.csv` and `.xlsx`.
- Standardizes `Names of Flights`.
- Preserves session description metadata.
- Removes CSV-only `ALT:ID` from processing outputs.

### Step 1b - Normalize Time Column

Status: Implemented

Current behavior:
- Uses valid `Elapsed` values directly.
- Rebuilds CSV elapsed time from `Time` when Excel export has collapsed `Elapsed`.
- Supports both `MM:SS.s` and `datetime.time` representations.

### Step 2 - Flight Boundary Detection

Status: Implemented

Current behavior:
- Uses positive-altitude hysteresis rather than absolute altitude.
- Returns half-open flight intervals.
- Excludes the first on-ground landing row from the in-flight label span.

### Step 3 - Fill Flight Labels

Status: Implemented

Current behavior:
- Forward-fills native XLSX labels.
- Leaves CSV unlabeled in-flight rows empty.
- Uses `"On Ground"` for on-ground rows.

### Step 4 - Per-Flight 3D GPS Drift Correction

Status: Implemented

Current behavior:
- Corrects latitude, longitude, and altitude independently per detected flight.
- Uses the same corrected half-open flight ranges returned by the detector.

### Step 5 - Compute Ground Coordinates

Status: Implemented

Current behavior:
- Uses corrected ray-ground intersection math.
- Filters blocked and saturated LIDAR rows.
- Suppresses ground-coordinate outputs for `On Ground` rows.

### Step 6 - Process Directory

Status: Implemented

Current behavior:
- Processes all `.csv` and `.xlsx` source files in a directory.
- Skips existing `-results.csv` files as inputs.
- Writes `<stem>-results.csv`.
- Records per-file metadata used by validation.

### Step 7 - Visualization and Notebook Use

Status: Implemented

Current behavior:
- Includes a top-level intro section.
- Includes a visible configuration cell for paths, thresholds, policies, and plotting.
- Uses file-specific plot names when saving is enabled.

### Step 8 - Validation

Status: Implemented

Current behavior:
- Produces a per-file validation summary.
- Checks:
  - expected flight count,
  - landing boundary correctness,
  - on-ground ground-coordinate policy,
  - CSV label policy.

## Validation Result On Current Dataset

Validation was re-run after implementation against the current files in `data/3_13_2026_Vernal_Cooked`.

Observed result:
- all 7 source files passed validation,
- exactly 1 flight was detected per file,
- landing boundaries were correct,
- on-ground rows produced 0 ground-coordinate outputs,
- the CSV file produced empty unlabeled in-flight `Names of Flights` rows and no CSV `"Pre-label"` rows.

## Current Output Columns

Processed outputs now contain:
- original source sensor columns retained from the input file,
- `Flight`,
- `Names of Flights`,
- `Latitude_adj`,
- `Longitude_adj`,
- `Altitude_adj`,
- `ground_lat`,
- `ground_lon`,
- `ground_alt_est`,
- `horizontal_offset_m`.

Derived-column semantics:
- `ground_*` columns are populated only for in-flight rows that pass the geometry and LIDAR validity filters.
- CSV unlabeled in-flight rows keep `Names of Flights` empty.

## Remaining Work

No open implementation items remain from the prior plan.

Possible future improvements, if needed:
- tune detector thresholds for new datasets,
- add formal automated tests outside the notebook,
- split visualization helpers into smaller notebook cells or a standalone module if the notebook grows further.
