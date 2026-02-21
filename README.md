# Shipguard Action

GitHub Action that scans your Next.js App Router codebase for unprotected routes, missing rate limiting, and SSRF surfaces. Comments on PRs with findings, resolves HOF wrapper implementations, verifies auth/rate-limit enforcement via TypeScript AST, and groups unverified wrappers into single findings.

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
      - uses: Fourteen-Systems/shipguard-action@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

The action posts a PR comment with score, findings, and detected stack. Adds inline annotations on flagged files and fails the check if thresholds are violated.

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

### What's New in v1.2

- **Auth-aware RL suppression**: routes with strongly enforced auth no longer produce rate-limit findings
- **General rate limit detection**: recognizes any function with `rateLimit`/`ratelimit`/`rate_limit` in the name
- **`shipguard:public-intent`**: annotation for intentionally public routes — suppresses auth findings, floors RL severity at HIGH, escalates to CRITICAL for SSRF surfaces
- **Improved auth detection**: catches `updateSession()` and similar patterns

### PR Comment

The PR comment includes:

- **Header** with score and PASS/WARN/FAIL status
- **Detected stack** (auth provider, ORM, rate limiter, middleware)
- **Severity summary** with color-coded counts (critical, high, med, low)
- **Findings table** with severity icon, rule, file location, and human-readable message
- **Evidence & confidence** (collapsible) — detailed evidence and confidence rationale per finding
- **Suggested fixes** (collapsible) — deduplicated remediation steps
- **Baseline delta** — score change and new/resolved finding counts when a baseline is provided

When no auth provider is detected, the comment warns that public mutation endpoints will be treated as high risk.

### Inline Annotations

Findings are added as inline annotations on the PR diff:

- **critical** findings → error annotations
- **high** findings → warning annotations
- **med/low** findings → notice annotations

Disable with `annotations: false`.

## Monorepos

For monorepos (Turborepo, pnpm workspaces), point to the Next.js app directory:

```yaml
- uses: Fourteen-Systems/shipguard-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    working-directory: apps/web
```

Shipguard automatically reads dependencies from both the app and workspace root, resolves tsconfig path aliases across the monorepo, and follows wrapper imports through barrel re-exports.

## Configuration

Most projects need no configuration. Shipguard auto-detects your stack, resolves wrapper implementations, and verifies auth/rate-limit enforcement automatically. Run `npx @fourteensystems/shipguard init` locally to generate a config if you need custom hints for edge cases.

See [shipguard](https://github.com/Fourteen-Systems/shipguard) for full documentation.

## License

MIT
