# Importing worksheet data locally

This guide shows how to mimic Bubble's upload flow from the npm workspace. The same helpers power automated tests and give contributors a quick way to reproduce issues with real spreadsheets.

## What ships with the repository?

- `core/test/fixtures/example.csv` - sample worksheet that exercises chi-square, Fisher, Mann-Whitney, and other tests.
- `core/test/fixtures/plugin-output.json` - the row-wise JSON payload that Bubble's file reader sends to `window.Statz.parseColumns`. The `parseFixture` helper regenerates it automatically from the CSV fixture so the flow matches Bubble's upload.
- `core/test/fixtures/column-hashes.json` - placeholder column hashes passed by Bubble when creating the database records. The hashes uniquely identify each column (Bubble produces them via the `formatted as` operator set to MD5 hash), so worksheets must avoid duplicate column names.
- `core/scripts/dev/load-fixture.mjs` - helper that loads `example.csv`, writes row-wise data in `plugin-output.json`, and passes the row-wise payload to `Statz.parseColumns` for you.
- `core/scripts/dev/run-fixture.mjs` - CLI script used by `npm run test:fixture` to dump the parsed database JSON.

## Running the end-to-end simulation

1. Install dependencies inside `core/` if you have not already:
   ```bash
   npm install
   ```
2. Execute the fixture script:
   ```bash
   npm run test:fixture
   ```
   The command prints the same `{ columns, history }` JSON Bubble writes into its database after invoking `window.Statz.parseColumns`.
3. Pipe the output into whatever analysis you want to inspect. For example, inside a Node REPL you can `import { parseFixture } from './scripts/dev/load-fixture.mjs'` and then pass `result.parsed.columns` into statistical helpers. You can also pass the dataset to helpers such as `getColumn` (see `docs/tutorials/dataset-helpers.md`) when you need to extract column values from the database payload.

## Swapping in your own worksheet

1. Drop a new `.csv` under `core/test/fixtures/` (use semicolon or comma delimiters - the helper does not enforce the format).
2. Copy `column-hashes.json` or craft your own array of unique identifiers that matches the column order.
3. Call the helper with custom parameters. It will regenerate the row-wise JSON for you:
   ```js
   import { parseFixture } from '../scripts/dev/load-fixture.mjs';

   const { parsed } = parseFixture({
     csvFile: 'your-file.csv',
     hashesFile: 'column-hashes.json',
     filename: 'your-file.csv'
   });
   ```
   Provide `rowsFile` if you want to persist the intermediate JSON to a different filename; otherwise the default `plugin-output.json` is reused.
4. Feed `parsed.columns` into the statistical modules (e.g., `Statz.summarize_q_q`, `Statz.test_mann_whitney`, etc.) and lean on helpers like `getColumn` from `docs/tutorials/dataset-helpers.md` to read values from the database payload.

## How this relates to Bubble

Bubble returns the worksheet as a row-wise JSON array and keeps the column hashes in a Bubble list. The helper mirrors that behaviour so you can:
- Write unit or integration tests around `parseColumns` without the Bubble runtime.
- Debug edge cases by pasting user spreadsheets into `core/test/fixtures`.
- Share reproducible bug reports with contributors by committing CSV/JSON pairs.

Because these helpers live under `core/scripts/dev/` (with fixtures in `core/test/fixtures/`), they remain dev-only assets and are excluded from the production bundle in `core/bubble-html/statz-bundle.html`.
