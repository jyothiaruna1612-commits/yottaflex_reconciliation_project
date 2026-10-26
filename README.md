# YottaFlex Payment Reconciliation Synthetic Data Project

A deterministic, locally runnable synthetic-data generator and reconciliation pipeline for payment operations. It creates linked merchant, payment, attempt, processor-event, settlement, bank-statement, match, exception, resolution, and ground-truth records.

> This is a candidate project starter implementation. Read the problem statement, run the checks, inspect the output, and make your own improvements before submitting. Do not claim work or validation you have not personally performed.

## Requirements

- Python 3.10+
- No third-party packages required
- Works locally; no credentials, APIs, or external data required

## Quick start

From the repository root:

```bash
python src/generate_dataset.py --scale starter --seed 20261022 --output data/starter
python src/validate_dataset.py --input data/starter
```

The `starter` scale is intentionally small so you can test quickly. Generate the full target volume with:

```bash
python src/generate_dataset.py --scale full --seed 20261022 --output data/full
python src/validate_dataset.py --input data/full
```

To check reproducibility, generate twice to different directories using the same seed and compare the `manifest.json` files (the generator records row counts and SHA-256 checksums):

```bash
python src/generate_dataset.py --scale starter --seed 20261022 --output data/run_a
python src/generate_dataset.py --scale starter --seed 20261022 --output data/run_b
```

The files should have the same checksums for the same scale and seed.

## Dataset contents

CSV files are UTF-8 with a header row. `ground_truth.csv` is a separate evaluation-only file; it must not be used as a feature/input when building any model.

- `merchants.csv`
- `payment_orders.csv`
- `payment_attempts.csv`
- `processor_events.csv`
- `settlement_batches.csv`
- `settlement_lines.csv`
- `bank_statement_lines.csv`
- `reconciliation_matches.csv`
- `exceptions.csv`
- `resolutions.csv`
- `ground_truth.csv`
- `manifest.json`
- `validation_summary.json`

The generator first creates a valid baseline and then injects named, labelled anomaly scenarios. The exact scenario labels are recorded in ground truth. Anomaly scenarios include duplicate settlement line, bank amount discrepancy, refund-before-capture event ordering, unmatched bank record, currency mismatch, and duplicate processor event ingestion.

## Business rules and assumptions

1. Amounts are stored as integer minor units (for example, paise/cents) to avoid binary floating-point errors. `currency` is an ISO-like three-letter code in this synthetic dataset.
2. A payment attempt can have multiple processor events; a payment order can have one or more attempts.
3. Captured/refunded activity is represented in settlement lines with signed amounts. Processor fees are stored separately and are deducted from net settlement.
4. Settlement batches group settlement lines by synthetic settlement date and currency.
5. Bank reconciliation compares reference, currency, amount within a configured tolerance, and date window. A candidate that fails one of these checks is not marked as an exact match.
6. Unmatched bank lines and settlement batches produce exception records. A resolution is an append-only record linked to an exception; the source exception is not overwritten.
7. `ground_truth.csv` records injected issues and expected decisions. It is not mixed into operational input tables.
8. The generator uses a fixed seed and derived values are calculated from source records.

These assumptions are intentionally documented and should be reviewed against the supplied assessment statement. Update this section if you change a rule.

## Validation

The validator checks:
- primary-key uniqueness
- foreign-key integrity for core relationships
- allowed currency values
- required columns and parseable CSVs
- settlement line totals against batch totals
- duplicate processor-event IDs
- manifest row counts and checksums when present

Some injected anomaly cases are expected to be present and labelled. Therefore, not every anomaly means the dataset generation itself failed; distinguish expected labelled anomalies from structural integrity failures.

## Suggested review workflow

1. Run the starter scale and read the CSVs.
2. Read `docs/DATA_DICTIONARY.md` and `docs/DESIGN_NOTES.md`.
3. Run the validator and inspect `validation_summary.json`.
4. Generate the full scale.
5. Generate twice with the same seed and compare checksums.
6. Add tests or extra rules that you can explain.
7. Update `docs/AI_PROMPT_HISTORY.md` with the actual prompts you used in every AI tool and your own review notes.
8. Commit meaningful increments and keep the repository public only after checking that no secrets or private assessment materials are included.

## Suggested Git workflow

```bash
git init
git add README.md src docs .gitignore
git commit -m "Add deterministic synthetic data generator and design docs"
git add .
git commit -m "Add generated dataset validation and review results"
```

Only commit generated datasets if they are reasonably sized and the assessment expects them. The generator, seed, manifest, and validation summary are often more reproducible than committing every full CSV. Check the assessment portal instructions before deciding.

## Limitations / next steps

- This is synthetic data, not a production payment integration.
- The generator intentionally uses simplified processor and bank behaviors.
- The matching engine is a transparent rule-based baseline, not an ML model.
- Review anomaly placement and extend tests for every business rule before submission.
- No hosted demo or automated test suite is mandatory according to the supplied FAQ, but clear setup instructions and evidence of verification are expected.
