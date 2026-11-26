# Lockwise GitHub Action (`shiprig/lockwise-action`)

Lint changed PostgreSQL migration files (Drizzle Kit, Prisma Migrate, or raw SQL) on every pull request, post a single idempotent sticky PR comment with safe ORM-specific rewrites, and optionally emit SARIF v2.1.0 for GitHub Code Scanning.

## Usage

```yaml
name: Lockwise Migration Check

on:
  pull_request:
    paths:
      - 'drizzle/**/*.sql'
      - 'prisma/migrations/**/migration.sql'

permissions:
  contents: read
  pull-requests: write

jobs:
  lockwise:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: shiprig/lockwise-action@v1
```

## Inputs

| Input          | Default                                               | Description                                                             |
| -------------- | ----------------------------------------------------- | ----------------------------------------------------------------------- |
| `paths`        | `drizzle/**/*.sql,prisma/migrations/**/migration.sql` | Comma-separated glob patterns for SQL migration files                   |
| `orm`          | `auto`                                                | ORM mode: `auto`, `drizzle`, `prisma`, or `sql`                         |
| `fail-on`      | `error`                                               | Minimum severity to fail the check: `error`, `warn`, or `none`          |
| `comment`      | `true`                                                | Post or update a sticky PR summary comment (`<!-- lockwise-report -->`) |
| `sarif-file`   | `""`                                                  | Optional relative output path for a SARIF v2.1.0 report                 |
| `pg-version`   | `17`                                                  | Target PostgreSQL major version (`12`–`18`)                             |
| `config`       | `""`                                                  | Optional relative path to `lockwise.config.{ts,mts,mjs,js,json}`        |
| `github-token` | `${{ github.token }}`                                 | Token for sticky PR comments and changed-file API fallback              |

## Outputs

| Output           | Description                                               |
| ---------------- | --------------------------------------------------------- |
| `findings-count` | Total number of findings reported                         |
| `error-count`    | Total number of error-severity findings reported          |
| `sarif-path`     | Path to generated SARIF report (when `sarif-file` is set) |

## Security & Fork PRs

- **Never trigger this Action on `pull_request_target`** — Lockwise can load JS/TS config files from the repository and explicitly blocks JS/TS config execution on `pull_request_target`.
- On public fork PRs, GitHub provides a read-only `GITHUB_TOKEN`. Lockwise catches `403` permission errors gracefully, logs a warning, and still completes migration linting and check status reporting.
