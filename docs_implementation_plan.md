# Documentation and Read the Docs Plan for PergamDataProcessing

## Summary

Create a Markdown-first documentation site using MkDocs + Material for MkDocs, hosted on Read the Docs, with content sourced from GitHub. Structure the docs for new analysts first, while adding a technical deep-dive section for developers and reviewers, especially around the trigonometry and coordinate-correction math.

The docs should treat the checked-in data files as example datasets only, not as the expected production input set. The plan should also avoid treating the checked-in `*-results.csv` files as canonical reference outputs, because they appear older than the current notebook behavior.

## Implementation Changes

### 1. Documentation architecture

Adopt this docs structure under `docs/`:

- `docs/index.md`
  Brief project overview, what the notebook does, what inputs it expects, what outputs it produces, and where to start.
  Include an "Open in Colab" link near the top of the page that points to the notebook in the GitHub repo.

- `docs/getting-started.md`
  Quickstart for analysts: clone repo, create Python environment, install dependencies, open notebook, set config, run pipeline, inspect outputs.

- `docs/data-model.md`
  Data dictionary for raw inputs and processed outputs.
  Cover CSV vs XLSX differences, required columns, optional columns, derived columns, and example row semantics.

- `docs/pipeline-overview.md`
  End-to-end explanation of the processing flow:
  read -> normalize time -> detect flight -> fill labels -> drift correction -> ground coordinates -> validation -> plots.

- `docs/flight-detection.md`
  Explain hysteresis thresholds, smoothing window, half-open intervals, and why positive altitude is used instead of absolute zero.

- `docs/drift-correction.md`
  Explain the per-flight 3D drift model, assumptions, interpolation over normalized elapsed time, and when it can fail.

- `docs/ground-coordinates.md`
  Technical deep dive on the geometry.
  Include body-to-NED orientation, downward ray intersection, meaning of `wn`, `we`, `wd`, why `t = s / wd`, and how geodetic offsets are computed.

- `docs/tutorials/run-example-dataset.md`
  Guided walkthrough using the checked-in sample data directory.
  Explicitly label it as an example only.

- `docs/tutorials/bring-your-own-data.md`
  Tutorial for users with their own Pergam-like files.
  Focus on required columns, validation checks, and common failure cases.

- `docs/tutorials/interpret-results.md`
  How to read `Flight`, `Names of Flights`, adjusted coordinates, `ground_*` fields, and validation output.

- `docs/reference/configuration.md`
  Document notebook configuration variables and recommended values.

- `docs/reference/functions.md`
  Manual function reference for the key notebook functions and their inputs/outputs.

- `docs/faq.md`
  Short operational answers: stale elapsed values, why on-ground rows are blank, why there is one flight per file, why labels differ between CSV and XLSX.

- `docs/changelog.md`
  Track major notebook and docs behavior changes.

Recommended navigation order in `mkdocs.yml`:
- Home
- Getting Started
- Data Model
- Pipeline Overview
- Tutorials
- Technical Reference
- FAQ
- Changelog

### 2. Content priorities

Write docs in this order:

1. `index.md`
2. `getting-started.md`
3. `pipeline-overview.md`
4. `data-model.md`
5. `tutorials/run-example-dataset.md`
6. `tutorials/bring-your-own-data.md`
7. `ground-coordinates.md`
8. `flight-detection.md`
9. `drift-correction.md`
10. `reference/configuration.md`
11. `reference/functions.md`
12. `faq.md`

Content requirements:
- Every tutorial should say whether it uses the checked-in sample data or user-provided data.
- Every technical page should tie formulas back to notebook function names.
- The trig/geometry page should use one worked numeric example showing how altitude, pitch/roll/heading, and LIDAR range become `ground_lat` and `ground_lon`.
- Add diagrams, plots, screenshots, tables, and other figures wherever they materially improve comprehension.
- Prefer visuals for:
  - pipeline flow,
  - coordinate-frame and trigonometry explanations,
  - input/output data examples,
  - validation summaries,
  - example methane and map outputs.
- The data-model page should explicitly note:
  - CSV `Elapsed` may be unusable and gets rebuilt from `Time`
  - XLSX files carry segment labels
  - CSV unlabeled in-flight rows keep empty `Names of Flights`
  - `ground_*` columns are blank on `On Ground` rows

### 3. Source-of-truth rules for documentation

Document against the current notebook and current processing plan only.

Use these repo artifacts as the documentation truth sources:
- `PergramV2.ipynb` for implementation behavior
- `processing_plan.md` for current status and design decisions
- raw files in `data/3_13_2026_Vernal_Cooked/` as example input shapes

Do not use the checked-in `*-results.csv` files as authoritative examples unless they are first regenerated to match the notebook.
If examples from outputs are needed before regeneration, the docs should say they are illustrative and may lag implementation.

### 4. Read the Docs + MkDocs site scaffolding

Add these repo files:

- `.readthedocs.yaml`
  Place at repo root. Per Read the Docs, this file belongs in the top-most directory and controls builds.
  Use MkDocs builder.
  Recommended baseline:
  - `version: 2`
  - `build.os: ubuntu-24.04`
  - `build.tools.python: "3.12"` or another chosen supported Python version
  - `mkdocs.configuration: mkdocs.yml`
  - `python.install` pointing to `docs/requirements.txt`

- `mkdocs.yml`
  Configure:
  - `site_name`
  - `site_url: !ENV READTHEDOCS_CANONICAL_URL`
  - Material theme
  - nav structure
  - Markdown extensions
  - repo URL / edit URI
  - home-page support for an "Open in Colab" link to the notebook
  - optional search/version integration

- `docs/requirements.txt`
  Recommended pinned minimum set:
  - `mkdocs`
  - `mkdocs-material`
  - `mkdocs-mermaid2-plugin` or another diagram plugin only if diagrams are actually used
  - `mkdocs-glightbox` only if image enlargement is desired
  Keep this small unless the docs truly need more.

Optional but recommended:
- `docs/javascripts/readthedocs.js`
  Only if you want Material search to trigger Read the Docs search and/or want the Read the Docs version selector integrated into the header.
- `docs/overrides/main.html`
  Only if you decide to integrate the Read the Docs version menu into Material navigation.

### 5. Documentation-writing conventions

Adopt these conventions from the start:
- Use Markdown only.
- Keep one concept per page.
- Prefer diagrams and tables for pipeline/data relationships.
- Use plots, screenshots, and annotated figures when they explain behavior faster than prose.
- Include “Why this exists”, “Inputs”, “Outputs”, and “Failure modes” on technical pages.
- Include “When to use this” and “Expected result” on tutorial pages.
- Mark example-data pages with a note that users will usually substitute their own files.
- Keep formulas in fenced math or clear monospace blocks and define all symbols the first time they appear.

## Read the Docs Setup Steps

Use official Read the Docs guidance for MkDocs and project import:
- Read the Docs MkDocs guide: https://docs.readthedocs.com/platform/stable/intro/mkdocs.html
- Read the Docs config file overview: https://docs.readthedocs.com/platform/stable/config-file/
- Read the Docs tutorial import flow: https://docs.readthedocs.com/platform/stable/tutorial/index.html
- MkDocs getting started: https://www.mkdocs.org/getting-started/
- Material for MkDocs install guide: https://squidfunk.github.io/mkdocs-material/getting-started/

Exact setup sequence:

1. Prepare the repo
   - Add `docs/`
   - Add `mkdocs.yml`
   - Add `.readthedocs.yaml`
   - Add `docs/requirements.txt`
   - Commit all docs scaffolding to GitHub default branch

2. Configure local docs build
   - Create a Python virtual environment
   - Install `docs/requirements.txt`
   - Run `mkdocs serve`
   - Fix nav, broken links, and missing files until the site builds locally

3. Create the Read the Docs project
   - Sign in to Read the Docs with GitHub
   - Import the GitHub repository from the dashboard
   - Confirm project name, repository URL, and default branch
   - Let the first build run automatically

4. Verify Read the Docs build
   - Open the build logs
   - Confirm Read the Docs detected `.readthedocs.yaml`
   - Confirm MkDocs is the selected builder
   - Confirm the site publishes successfully

5. Configure project settings in Read the Docs
   - Add project description
   - Enable build failure email notifications
   - Enable pull request builds
   - Decide whether `latest` or `stable` should be the default version once release branches/tags exist

6. Configure MkDocs for Read the Docs
   - Set `site_url: !ENV READTHEDOCS_CANONICAL_URL`
   - Set `theme.name: material`
   - Add repo/edit links
   - Optionally integrate Read the Docs search and version menu into Material once the basic site is stable

7. Establish docs publishing workflow
   - Build docs on every push to default branch
   - Build PR previews for docs changes
   - Require local `mkdocs build` to pass before merging docs changes

## Test Plan

### Documentation content checks
- Each page has a clear audience and purpose.
- Technical pages reference actual notebook function names and current behavior.
- Tutorials explicitly distinguish example bundled data from user data.
- The geometry page includes at least one worked example.
- The docs include diagrams/plots where they materially help, rather than relying on text alone.
- The home page includes a working "Open in Colab" link to the repository notebook.

### Build and site checks
- `mkdocs build` passes locally.
- Read the Docs build passes using `.readthedocs.yaml`.
- Navigation works and all pages are reachable from the top nav.
- Internal links and image paths resolve correctly.
- Search indexes the main pages.
- PR preview builds render docs changes correctly.

### Accuracy checks against repo
- Data dictionary matches current raw input columns and current notebook-derived output columns.
- Flight detection docs match current hysteresis thresholds and half-open interval semantics.
- Label policy docs match current CSV/XLSX behavior.
- Ground-coordinate docs match the current on-ground null-output policy.
- Validation page matches the checks emitted by `validate_processed_results()`.

## Assumptions

- Default doc stack: MkDocs + Material for MkDocs.
- Primary audience: new analysts, with a dedicated technical deep-dive for geometry/trigonometry.
- Docs source format: Markdown files committed to GitHub.
- The current notebook remains the implementation source of truth.
- Checked-in data files are examples only.
- Checked-in `*-results.csv` files should not be treated as canonical docs examples unless regenerated from the current notebook.
