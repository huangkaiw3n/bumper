# Bumper

A GitHub Action that analyzes PostgreSQL migrations for potential lock issues and posts results as PR comments.

## What it detects

- **CRITICAL**: Table rewrites (GENERATED STORED columns, ALTER COLUMN TYPE, ADD CONSTRAINT without NOT VALID)
- **HIGH**: Write-blocking locks on existing tables (foreign key references, non-concurrent indexes)
- **MEDIUM**: Instant ACCESS EXCLUSIVE locks (metadata-only operations)
- **LOW**: Safe operations (CONCURRENTLY indexes, no locks on existing tables)

## Usage

Add to your workflow (e.g., `.github/workflows/migration-review.yml`):

```yaml
name: Migration Review

on:
  pull_request:
    paths:
      # Match these to your migration-paths input (defaults shown below)
      - "migrations/**/*.sql"
      - "migrations/**/*.ts"
      - "migrations/**/*.js"
      - "src/database/migrations/**/*.ts"
      - "src/database/migrations/**/*.js"
      - "src/database/migrations/**/*.sql"

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: huangkaiw3n/bumper@v1
        with:
          anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          # Default migration-paths (uncomment to customize):
          # migration-paths: 'migrations/**/*.sql,migrations/**/*.ts,migrations/**/*.js,src/database/migrations/**/*.ts,src/database/migrations/**/*.js,src/database/migrations/**/*.sql'
```

## Inputs

| Input               | Required | Default                   | Description                     |
| ------------------- | -------- | ------------------------- | ------------------------------- |
| `anthropic-api-key` | Yes      | -                         | Anthropic API key               |
| `github-token`      | Yes      | -                         | GitHub token for PR comments    |
| `migration-paths`   | No       | `migrations/**/*.sql,...` | Glob patterns (comma-separated) |

## Setup

1. Get an Anthropic API key from [console.anthropic.com](https://console.anthropic.com)
2. Add `ANTHROPIC_API_KEY` to your repository secrets (Settings → Secrets and variables → Actions)
3. Add the workflow file to your repository

## Custom migration paths

```yaml
- uses: huangkaiw3n/bumper@v1
  with:
    anthropic-api-key: ${{ secrets.ANTHROPIC_API_KEY }}
    github-token: ${{ secrets.GITHUB_TOKEN }}
    migration-paths: "db/migrate/**/*.rb,prisma/migrations/**/*.sql"
```

## Example output

> ## 🚧 Bumper
>
> ### 📄 `migrations/001_add_user_status.sql`
>
> ## Migration Lock Analysis
>
> ### Tables Affected
>
> | Table | Lock Type        | Blocks Reads | Blocks Writes | Duration |
> | ----- | ---------------- | ------------ | ------------- | -------- |
> | users | ACCESS EXCLUSIVE | Yes          | Yes           | Instant  |
>
> ### Risk Assessment
>
> **Risk Level:** MEDIUM
>
> ### Notes
>
> None.

## Cost

Uses Claude Sonnet. Typical cost: ~$0.003-0.01 per migration file.

## License

MIT
