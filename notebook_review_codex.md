# PergramV2.ipynb Review

Reviewed artifacts:
- `PergramV2.ipynb`
- `processing_plan.md`
- `notebook_review.md`
- Existing outputs in `data/3_13_2026_Vernal_Cooked/*-results.csv`

This review is based on the current notebook state, not the earlier draft plan. I also re-executed the notebook functions in a non-mutating way against the existing dataset to confirm behavior.

## Findings

### 1. High: `detect_flights()` includes the landing-transition row in the flight span

Evidence:
- Flight labeling is inclusive at [`PergramV2.ipynb:300`](PergramV2.ipynb#L300): `df.loc[t:l, "Flight"] = f"Flight {i}"`
- The landing row is the first row where `on_ground` becomes true at [`PergramV2.ipynb:276`](PergramV2.ipynb#L276)
- The summary slice excludes that row at [`PergramV2.ipynb:304`](PergramV2.ipynb#L304): `alt.iloc[t:l].max()`

Impact:
- The first on-ground sample after landing is labeled as in-flight.
- Labeling and printed flight summaries are inconsistent.
- Downstream logic that keys off `Flight != "On Ground"` can process a landing row as airborne.

Recommended update:
- Treat the landing transition row as exclusive for the in-flight segment, or redefine the transition logic so `Flight` and reporting use the same interval convention.

### 2. High: absolute-threshold flight detection is brittle at end-of-file because GPS altitude drifts negative after landing

Evidence:
- Ground detection is currently `|smoothed altitude| < 0.5 m` at [`PergramV2.ipynb:276`](PergramV2.ipynb#L276)
- Re-checking the raw files showed false trailing takeoff transitions after the real landing in several files:
  - `2025-11-19-vernaldata.csv`: takeoffs at rows 28 and 6899, landing at 6877
  - `2026-03-13-17-10-14-position.xlsx`: takeoffs at rows 28 and 4864, landing at 4823
  - `2026-03-13-17-20-45-position.xlsx`: takeoffs at rows 27 and 2733, landing at 2694
  - `2026-03-13-18-03-11-position.xlsx`: takeoffs at rows 27 and 13472, landing at 13437

Impact:
- The current pairing logic hides the false trailing takeoff only because there is no later landing.
- The detector works on this dataset because the extra transition is dropped, but the method is not robust to longer files or additional post-flight data.

Recommended update:
- Tighten the flight detector so post-flight GPS drift cannot generate a second takeoff after landing. The fix could be relative-to-baseline logic, hysteresis, minimum airborne duration, or explicit end-of-file handling, but the current behavior should be treated as a real detector weakness.

### 3. Medium: `compute_ground_coordinates()` produces ground points for most `On Ground` rows

Evidence:
- The LIDAR validity mask excludes blocked and saturated rows, but it does not suppress on-ground rows; it only uses `Flight` to detect blocked in-flight zeros at [`PergramV2.ipynb:515`](PergramV2.ipynb#L515)
- Existing outputs show this clearly. For example, the following files contain ground coordinates on nearly every `On Ground` row:
  - `2025-11-19-vernaldata-results.csv`: 113 of 114 `On Ground` rows have `ground_lat`
  - `2026-03-13-17-10-14-position-results.csv`: 89 of 90
  - `2026-03-13-18-03-11-position-results.csv`: 111 of 112
- Ground elevation is computed as `gps_alt - s` at [`PergramV2.ipynb:584`](PergramV2.ipynb#L584), which yields negative `ground_alt_est` on many on-ground rows because GPS altitude is near zero while LIDAR still reports a small positive height.

Impact:
- Output CSVs look like they contain valid laser ground intersections even while the drone is on the ground.
- `ground_alt_est` becomes misleading for those rows.

Recommended update:
- Decide this explicitly in the plan and notebook behavior:
  - either blank out ground-coordinate outputs for `Flight == "On Ground"`,
  - or document that on-ground rows are intentionally retained and should be filtered by consumers.

### 4. Medium: `fill_flight_labels()` triggers pandas dtype warnings on the CSV path

Evidence:
- The CSV path inserts an empty `Names of Flights` column in `read_sensor_file()` at [`PergramV2.ipynb:52`](PergramV2.ipynb#L52)
- `fill_flight_labels()` then assigns strings into that column at [`PergramV2.ipynb:349`](PergramV2.ipynb#L349) and forward-fills it at [`PergramV2.ipynb:356`](PergramV2.ipynb#L356)
- Re-running the functions reproduces pandas `FutureWarning`s for incompatible dtype assignment and future downcasting changes

Impact:
- The notebook still runs today, but it is relying on behavior pandas has warned will change.

Recommended update:
- Normalize `Names of Flights` to an object/string-compatible dtype before assigning `"On Ground"` or calling `ffill()`.

### 5. Medium: `processing_plan.md` is now stale relative to the notebook

Evidence:
- The plan still presents major items as not yet implemented, including `normalize_elapsed`, 3D drift correction, fixed ground-coordinate math, and directory processing in [`processing_plan.md:109`](processing_plan.md#L109)
- Those functions exist in the notebook now:
  - `read_sensor_file()` at [`PergramV2.ipynb:52`](PergramV2.ipynb#L52)
  - `normalize_elapsed()` at [`PergramV2.ipynb:144`](PergramV2.ipynb#L144)
  - `correct_drift_per_flight()` at [`PergramV2.ipynb:380`](PergramV2.ipynb#L380)
  - `compute_ground_coordinates()` at [`PergramV2.ipynb:468`](PergramV2.ipynb#L468)
  - `process_directory()` at [`PergramV2.ipynb:615`](PergramV2.ipynb#L615)

Impact:
- The markdown plan no longer cleanly distinguishes implemented work from remaining work.
- Some review comments are now anchored to obsolete assumptions.

Recommended update:
- Convert `processing_plan.md` from a pre-implementation plan into a status document with only the remaining deltas.

### 6. Low: notebook usability/documentation issues are still present

Evidence:
- There is still no top-level introductory markdown cell before the first step heading.
- Most step markdown cells are one-line headers.
- The runnable configuration remains hardcoded in the execution cell at [`PergramV2.ipynb:987`](PergramV2.ipynb#L987).
- Plot helpers still write fixed filenames that overwrite prior outputs at [`PergramV2.ipynb:795`](PergramV2.ipynb#L795), [`PergramV2.ipynb:846`](PergramV2.ipynb#L846), [`PergramV2.ipynb:912`](PergramV2.ipynb#L912), and [`PergramV2.ipynb:967`](PergramV2.ipynb#L967).

Impact:
- These are not correctness blockers, but they make the notebook harder to operate and easier to misuse.

Recommended update:
- Add an intro/config cell near the top and either parameterize plot output paths or stop saving plots automatically.

## Plan Alignment: What Still Needs Implementation

Most of the original `processing_plan.md` steps are already implemented in the notebook:
- Step 1: unified file reader
- Step 1b: elapsed normalization
- Step 2: GPS-altitude flight detection
- Step 3: flight-label filling
- Step 4: per-flight 3D drift correction
- Step 5: corrected ground-coordinate geometry
- Step 6: directory processing

What still needs implementation or explicit decision:
- Fix the flight-boundary interval bug at landing.
- Harden flight detection against post-flight negative GPS drift.
- Decide whether on-ground rows should produce ground-coordinate outputs.
- Resolve the CSV `Names of Flights` behavior: the plan says "leave empty," but current behavior fills in `"Pre-label"` for unlabeled in-flight rows.
- Clean up the pandas warning path in `fill_flight_labels()`.
- Update documentation/configuration so the notebook's current behavior is explained clearly.

## Comparison Against `notebook_review.md` From Claude

### Valid points worth keeping

Claude's review is still directionally useful on these items:
- Add a top-level intro/configuration section.
- Document or parameterize hardcoded thresholds.
- Fix the pandas dtype warning in `fill_flight_labels()`.
- Avoid saving plots to fixed filenames that overwrite each other.
- Consider splitting the large visualization cell for maintainability.

### Points that should be corrected or removed

Several Claude findings are stale or inaccurate against the current notebook:
- The notebook no longer has 21 cells; the current file has 23 cells.
- `normalize_elapsed()` is already implemented.
- CSV/XLSX dual-format reading is already implemented.
- 3D drift correction is already implemented for latitude, longitude, and altitude.
- The old "test cell left in notebook" claim does not match the current notebook.
- The "CSV drift correction doesn't run" claim is contradicted by current execution; re-running the functions on `2025-11-19-vernaldata.csv` prints `Flight 1: horizontal drift=2.37m, altitude drift=+0.240m`.

### Updates Claude's review missed

The current review should keep these Codex-only findings because they are materially important and were not captured in `notebook_review.md`:
- Landing-row inclusion is an off-by-one bug in flight labeling.
- Post-flight GPS drift can create false trailing takeoffs.
- Ground coordinates are currently generated for most `On Ground` rows.
- `processing_plan.md` is now stale and should be rewritten as a current-state document.
- The plan's intended CSV label behavior does not match the notebook's current `"Pre-label"` fill behavior.

## Suggested Updates To This Review

If this review is revised later, the next version should keep the current structure but tighten two things:
- Add one short appendix table with per-file takeoff/landing rows and trailing false-takeoff rows, so the detector issue is easy to audit.
- Add one short appendix table summarizing `On Ground` rows with non-null `ground_lat` per output file, so the ground-coordinate behavior is explicit.

I would not add speculative geometry criticism beyond what was re-verified here; the strongest remaining issues are the flight-boundary logic, the detector robustness, and the on-ground output semantics.

## Suggested Updates To `processing_plan.md`

Recommended changes to the markdown plan:

1. Replace the current "Code Review Issues" section with a status-oriented section.
   - Remove items that are already fixed in the notebook.
   - Keep only the unresolved issues: landing-row inclusion, trailing false takeoffs, on-ground output semantics, and label dtype cleanup.

2. Mark Steps 1 through 6 as implemented.
   - Keep a short "remaining deltas" note under each step only where work remains.

3. Update Step 2 to reflect the detector edge cases.
   - Note that the current implementation uses absolute smoothed GPS altitude with a 0.5 m threshold.
   - Note that this creates false trailing takeoffs in some files after landing due to negative drift.
   - State that the landing boundary should be treated consistently and not include the first on-ground row in-flight.

4. Update Step 3 to resolve the CSV label policy.
   - Either say CSV rows should keep `Names of Flights` empty, or formally accept the notebook's current `"Pre-label"` behavior.
   - The document should not say one thing while the notebook does another.

5. Update Step 5 to define expected behavior for `On Ground` rows.
   - Specify whether `ground_lat`, `ground_lon`, `ground_alt_est`, and `horizontal_offset_m` should be null for on-ground rows.

6. Update Step 7 to match the current notebook structure.
   - The current notebook layout and cell numbering no longer match the references in the plan.

7. Add a short "Known Remaining Issues" section.
   - Flight-boundary off-by-one at landing
   - False trailing takeoffs from post-flight drift
   - On-ground ground-coordinate semantics
   - `Names of Flights` dtype warning

## Bottom Line

The notebook is much closer to complete than `processing_plan.md` and `notebook_review.md` imply. The remaining issues are not "missing major features"; they are correctness and semantics issues around flight boundaries, detector robustness, on-ground output handling, and a pandas warning path.
