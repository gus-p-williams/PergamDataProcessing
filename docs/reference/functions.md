# Function Reference

This page summarizes the primary notebook functions and the role each one plays.

## `read_sensor_file(path)`

- standardize CSV and XLSX input into a single dataframe shape
- insert `Names of Flights` for CSV
- preserve file-type metadata in `df.attrs`

## `normalize_elapsed(df)`

- ensure a usable monotonic `Elapsed` column
- use `Elapsed` directly when valid
- rebuild from `Time` if needed

## `detect_flights(...)`

- detect takeoff and landing from smoothed GPS altitude
- use positive-altitude hysteresis
- return half-open intervals
- write `Flight`

## `fill_flight_labels(...)`

- propagate segment labels within each detected flight
- forward-fill XLSX labels
- keep CSV unlabeled in-flight rows empty
- mark on-ground rows as `On Ground`

## `correct_drift_per_flight(df, flights, n_avg=10)`

- apply linear 3D GPS drift correction per flight
- emit `Latitude_adj`, `Longitude_adj`, and `Altitude_adj`

## `compute_ground_coordinates(...)`

- project the laser beam to the ground using adjusted pose and LIDAR range
- emit `ground_lat`, `ground_lon`, `ground_alt_est`, and `horizontal_offset_m`
- filter blocked, saturated, invalid-direction, and on-ground rows

## `process_directory(...)`

- batch-process all supported files in a directory
- write one `-results.csv` file per input
- store per-file processing metadata

## `validate_processed_results(...)`

- summarize whether the key notebook policies were satisfied
- check expected flight count, landing boundaries, ground-output policy, and CSV label policy

## Visualization Functions

- `_maybe_save_figure(...)`: optional figure saving with file-specific names
- `summarize_methane(...)`: methane summary statistics
- `plot_methane_hist_seaborn(...)`: histogram and optional KDE
- `plot_methane_hist_box_notched(...)`: histogram plus notched boxplot
