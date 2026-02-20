# Shipguard Action

GitHub Action that scans your Next.js App Router codebase for unprotected mutation routes and comments on PRs with findings.

## Usage

```yaml
name: Shipguard
on: [pull_request]

permissions:
  contents: read
  pull-requests: write

jobs:
  shipguard:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - uses: fourteensystems/shipguard-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

The action posts a PR comment with findings, adds inline annotations on flagged files, and fails the check if thresholds are violated.

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `fail-on` | `critical` | Minimum severity to fail the check |
| `min-confidence` | `high` | Minimum confidence to include |
| `min-score` | `70` | Minimum passing score (0-100) |
| `baseline` | — | Path to baseline file for regression detection |
| `max-new-critical` | `0` | Max new critical findings allowed |
| `max-new-high` | — | Max new high findings allowed (baseline mode) |
| `comment` | `true` | Post a PR comment with findings |
| `annotations` | `true` | Add inline file annotations |
| `working-directory` | — | Directory to scan (for monorepos) |

## Outputs

| Output | Description |
|--------|-------------|
| `score` | Shipguard score (0-100) |
| `findings` | Total number of findings |
| `result` | `PASS`, `WARN`, or `FAIL` |

## What It Does

- Comments on PRs with score, detected stack, and findings table
- Adds inline annotations on flagged files in the PR diff
- Shows score delta when a baseline is provided
- Updates the same comment on re-runs (no spam)
- Fails the check if score or severity thresholds are violated

## Configuration

Most projects need no configuration. Run `npx shipguard init` locally to generate a config if you need custom hints.

See [shipguard](https://github.com/fourteensystems/shipguard) for full documentation.

## License

MIT
