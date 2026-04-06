# Configuration Reference

These variables are defined near the top of `PergramV2.ipynb`.

## Paths

- `INPUT_DIR`: directory containing raw CSV/XLSX inputs
- `OUTPUT_DIR`: where `-results.csv` outputs are written

## Flight Detection

- `FLIGHT_ALT_COL`
- `TAKEOFF_THRESHOLD_M`
- `LANDING_THRESHOLD_M`
- `ALTITUDE_SMOOTHING_WINDOW`
- `MIN_TAKEOFF_RUN`
- `MIN_LANDING_RUN`

## Drift Correction

- `DRIFT_AVG_POINTS`

## Label Policy

- `ON_GROUND_LABEL`
- `PRE_LABEL_VALUE`
- `CSV_UNLABELED_POLICY`

Current recommended value:

```python
CSV_UNLABELED_POLICY = "empty"
```

## Ground Geometry

- `EMIT_GROUND_COORDS_ON_GROUND`
- `LIDAR_BLOCKED_THRESHOLD_M`
- `LIDAR_SATURATION_M`
- `MIN_DOWN_COMPONENT`

Current recommended value:

```python
EMIT_GROUND_COORDS_ON_GROUND = False
```

## Plotting

- `SAVE_PLOTS`
- `PLOT_OUTPUT_DIR`
- `FILE_TO_PLOT`
- `METHANE_CLIP_MAX`
