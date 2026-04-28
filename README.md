# Foodbook Shiny App

**Disponible en français** | **Available in English**

Foodbook is a bilingual Shiny application for comparing observed case
exposures against Foodbook reference percentages.

## Applications

- `app-public/`: manual entry and CSV upload workflow for external use
- `app-internal/`: CEDARS Excel upload workflow for internal use

Both apps share backend logic from `src/` and use the same bilingual
translation resources from `translations/translation.json`.

## Running locally

From the repository root:

```r
shiny::runApp("app-public")
shiny::runApp("app-internal")
```

## Data

The repository supports two operating modes:

- Full functionality when authoritative microdata is available in
	`upgrade-context/`
- Fallback mode using bundled reference data in `data/` when microdata is
	not available

If `upgrade-context/` is included, keep the repository private unless data
governance explicitly approves another arrangement.

## Testing

```r
testthat::test_dir("tests/testthat")
```

## Documentation

- `DEPLOYMENT.md`: setup, deployment, and maintenance guidance

## Support

Use the repository issue tracker for maintenance tasks and enhancements.
