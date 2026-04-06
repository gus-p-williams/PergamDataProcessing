# Getting Started

This page is the fastest way to get from repository checkout to a working notebook run.

## Prerequisites

- Python 3.11 or 3.12
- Jupyter Notebook or JupyterLab
- A local clone of the repository
- Pergam-style CSV or XLSX files

## Basic Setup

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install jupyter pandas numpy matplotlib seaborn openpyxl
```

If you also want to build this documentation site locally:

```powershell
pip install -r docs/requirements.txt
```

## Open the Notebook

```powershell
jupyter notebook
```

Then open `PergramV2.ipynb`.

You can also open the notebook directly in Colab:

[Open In Colab](https://colab.research.google.com/github/gus-p-williams/PergamDataProcessing/blob/main/PergramV2.ipynb)

## Notebook Configuration

The main configuration lives near the top of the notebook:

- `INPUT_DIR`
- `OUTPUT_DIR`
- `TAKEOFF_THRESHOLD_M`
- `LANDING_THRESHOLD_M`
- `ALTITUDE_SMOOTHING_WINDOW`
- `CSV_UNLABELED_POLICY`
- `EMIT_GROUND_COORDS_ON_GROUND`
- `SAVE_PLOTS`

## Quickstart

1. Set `INPUT_DIR` to your raw data directory.
2. Leave `OUTPUT_DIR = None` to write `-results.csv` beside the inputs, or set a dedicated output directory.
3. Run the notebook from top to bottom.
4. Inspect the validation summary.
5. Review the generated `-results.csv` outputs.
6. Use the map and methane plot cells for visual inspection.
