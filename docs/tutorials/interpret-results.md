# Tutorial: Interpret Results

This page explains how to read the derived output columns and validation table.

## Main Output Columns

### `Flight`

- `On Ground` for rows outside the detected airborne interval
- `Flight 1`, `Flight 2`, etc. for in-flight rows

### `Names of Flights`

- XLSX files: propagated segment labels
- CSV files: empty for unlabeled in-flight rows
- on-ground rows: `On Ground`

### Adjusted coordinates

- `Latitude_adj`
- `Longitude_adj`
- `Altitude_adj`

These are the drift-corrected coordinates used for downstream geometry.

### Ground coordinates

- `ground_lat`
- `ground_lon`
- `ground_alt_est`
- `horizontal_offset_m`

These fields are populated only for valid in-flight rows.

## Validation Table

`validate_processed_results()` reports:

| Column | Meaning |
| --- | --- |
| `flights_detected` | number of detected flights |
| `flight_count_ok` | whether flight count matched expectation |
| `landing_boundary_ok` | whether landing rows were labeled correctly |
| `on_ground_ground_rows` | number of on-ground rows with non-null `ground_*` outputs |
| `on_ground_policy_ok` | whether the current ground-output policy held |
| `csv_label_policy_ok` | whether CSV label propagation matched policy |
| `all_checks_pass` | aggregate success flag |
