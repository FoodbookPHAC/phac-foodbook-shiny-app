# Deployment Guide

This guide covers setup, deployment, and maintenance of the Foodbook Shiny
apps for PHAC OMD.

## Repository Setup

Clone the repository and confirm the expected structure is present:

```bash
git clone https://github.com/<owner>/phac-foodbook-shiny-app.git
cd phac-foodbook-shiny-app
```

Required runtime content:

- `app-public/`
- `app-internal/`
- `src/`
- `data/`
- `config/`
- `translations/`

Optional full-functionality content:

- `upgrade-context/` for authoritative microdata and label files

Maintenance-only content that should normally stay in the repository:

- `tests/` for regression coverage
- `scripts/` for supporting data maintenance tasks

If `upgrade-context/` is present, keep the repository private unless data
governance explicitly approves another arrangement.

## Local Environment

### R Version

- Required: R 4.5.2

### Install Packages

This project does not use `renv`. Install packages directly:

```r
install.packages(c(
  "shiny", "bslib", "thematic",
  "dplyr", "purrr", "tidyr", "stringr", "rlang",
  "data.table", "DT", "ggplot2",
  "shinyjs", "shinycssloaders",
  "shiny.i18n",
  "readxl", "haven",
  "rsconnect"
))
```

### Run the Apps Locally

```r
shiny::runApp("app-public")
shiny::runApp("app-internal")
```

On Windows workstations where `Rscript` is not on `PATH`, use:

```powershell
& "C:\Program Files\R\R-4.5.2\bin\x64\Rscript.exe" -e "testthat::test_dir('tests/testthat')"
```

When authoritative microdata is available, startup logs should include:

```
Loading authoritative microdata from upgrade-context/...
Loaded authoritative microdata from upgrade-context/ (... respondents)
Loaded authoritative labels from upgrade-context/foodbook variable labeling.do
```

If `upgrade-context/` is absent, the apps fall back to bundled reference data.

The authoritative data files currently expected in `upgrade-context/` are:

- `foodbook.dta`
- `foodbook2v2.dta`
- `foodbook data.do`
- `foodbook variable labeling.do`
- `Exposures_FoodBook2.do`

## Deployment Options

### Option 1: Private Repository With Full Functionality

Use this option when `upgrade-context/` remains in the repository.

- Keep the repository private.
- Deploy on infrastructure that supports private GitHub repositories.
- Publish `app-public/app.R` and `app-internal/app.R` as separate Shiny apps.

Recommended runtime settings for each app:

| Setting | Recommended Value |
|---------|-------------------|
| Memory | 4 GB |
| CPUs | 2 |
| Inactivity Timeout | 15 minutes |

### Option 2: Public Deployment Repository Without Microdata

Use this option when the deployment target only supports public repositories
or when the repository must exclude `upgrade-context/`.

- Create a deployment-only copy of the repository without `upgrade-context/`.
- Deploy the public copy.
- Expect fallback mode only.

In fallback mode:

- reference values come from `data/exposure_proportions_by_pt.csv`
- PT, age group, and month filtering are not available

## Posit Connect Cloud Notes

If using Posit Connect Cloud free accounts:

- private GitHub repositories are not supported
- each app can be published as its own Shiny application
- automatic republish on push should be enabled after initial publish

If the repository stays private, use a deployment target that can read private
repositories or deploy from a separate public copy that excludes
`upgrade-context/`.

For each app:

1. Create or select the target account.
2. Click **Publish**.
3. Select **Shiny**.
4. Choose the repository and branch.
5. Set the primary file to `app-public/app.R` or `app-internal/app.R`.
6. Publish and verify logs.

## Verification Checklist

After deployment, verify:

- both apps launch successfully
- bilingual language switching works in EN and FR
- the public app supports manual entry, CSV upload, and exports
- the internal app supports CEDARS Excel upload and exports
- log output matches the expected data mode

If full-functionality mode is expected, verify that startup logs reference
`upgrade-context/`. If fallback mode is expected, confirm the app starts
without those files and still produces reference values.

## Updating the Apps

### Routine Changes

1. Make code changes locally.
2. Run the relevant app locally.
3. Run tests with `testthat::test_dir("tests/testthat")`.
4. Commit and push.
5. Confirm both app deployments succeed.

### Regenerate Manifests

When dependencies change, update manifests with `rsconnect`:

```r
rsconnect::writeManifest(appDir = "app-public", appPrimaryDoc = "app.R")
rsconnect::writeManifest(appDir = "app-internal", appPrimaryDoc = "app.R")
```

### Translation Updates

Add or update entries in `translations/translation.json` using the existing
`{"en": "...", "fr": "..."}` structure.

## Troubleshooting

### `upgrade-context/` Not Found

Cause:

- authoritative microdata is not present in the repository or deployment

Effect:

- the app runs in fallback mode using bundled reference data

### Missing Package Errors

Cause:

- dependency not installed locally or not captured in the manifest

Fix:

- install the missing package
- update `force_deps.R` if needed
- regenerate the manifests

### Language Switching Issues

Cause:

- missing or invalid `translations/translation.json`

Fix:

- verify the file is present and valid JSON

### CEDARS Upload Issues

Verify the uploaded workbook is `.xlsx` format and includes the expected
columns:

- `NationalID`
- `ExposureCode`
- `HasExposureOccurred`

### Manifest Drift

If deployment starts failing after dependency changes, regenerate both
manifests directly with `rsconnect::writeManifest(...)` and commit the updated
`app-public/manifest.json` and `app-internal/manifest.json` files.

## Project Overview

Both applications compare observed outbreak case exposures against Foodbook
population reference percentages.

| | Public App | Internal App |
|-|-----------|-------------|
| Audience | External partners | Internal epidemiology workflows |
| Data entry | Manual counts or CSV/Excel upload | CEDARS Excel upload |
| Custom exposures | Yes | No |
| Bilingual | EN/FR | EN/FR |

Both apps share backend code in `src/`.

## Documentation

- `README.md` for quick start and repository overview
- `DEPLOYMENT.md` for setup, deployment, and maintenance

## Repository Notes

- Repository name: `phac-foodbook-shiny-app`
- Keep the repository private while `upgrade-context/` is included unless data
  governance approves a different model

## Useful Links

- [Posit Connect Cloud Dashboard](https://connect.posit.cloud/)
- [Connect Cloud Documentation](https://docs.posit.co/connect-cloud/)
- [Deploy Shiny R to Connect Cloud](https://docs.posit.co/connect-cloud/how-to/r/shiny-r.html)
- [shiny.i18n Documentation](https://github.com/Appsilon/shiny.i18n)
