# Read the Docs Setup

This page is for maintainers who want to publish these Markdown docs on Read the Docs.

## Files in This Repository

The docs site is configured by:

- `.readthedocs.yaml`
- `mkdocs.yml`
- `docs/requirements.txt`

## Local Preparation

Install the docs dependencies:

```powershell
python -m pip install -r docs/requirements.txt
```

Run a local build:

```powershell
mkdocs build
```

Run the local dev server:

```powershell
mkdocs serve
```

## Import the Project Into Read the Docs

1. Sign in to Read the Docs with your GitHub account.
2. Import `gus-p-williams/PergamDataProcessing`.
3. Confirm the default branch is correct.
4. Let the initial build run.

## What Read the Docs Uses

- MkDocs as the site builder
- Material for MkDocs as the theme
- the repository root `.readthedocs.yaml` to control the build

## Recommended Project Settings

- enable pull request builds
- keep build notifications enabled
- confirm the default published version is what you expect

## Current Colab Link

The docs home page links to:

`https://colab.research.google.com/github/gus-p-williams/PergamDataProcessing/blob/main/PergramV2.ipynb`

If the repository owner, branch name, or notebook path changes, update that link in `docs/index.md`.
