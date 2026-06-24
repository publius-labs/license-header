# license-header

Shared reusable GitHub Actions workflow for automated licence header management across Publius repos.

Uses [Apache SkyWalking Eyes](https://github.com/apache/skywalking-eyes) to add and verify proprietary licence headers on source files.

## Quick Start

Add one file to your repo:

**`.github/workflows/license.yaml`**

```yaml
name: License Headers

on:
  push:
    branches:
      - '**'
      - '!main'

permissions:
  contents: write

jobs:
  license:
    uses: publius-labs/license-header/.github/workflows/license-check.yaml@main
```

That's it. On the next push to any non-main branch, the workflow will:

1. **Fix** — add missing licence headers and auto-commit with `[skip ci]`
2. **Check** — verify all files have valid headers (fails the check if any are missing)

## Repo-Specific Ignore Patterns

The common config already ignores universal patterns: `.github/`, `.claude/`, `*.md`, `*.json`, `*.toml`, `requirements*.txt`, `.env*`, `.gitignore`, `.dockerignore`, etc.

If your repo needs **additional** patterns ignored, create `.licenserc.repo.yaml` in your repo root:

```yaml
header:
  paths-ignore:
    - '**/contracts/**'
    - '**/*.mako'
    - 'some/repo-specific/path/**'
```

These are merged with the common ignores at runtime via [yq](https://github.com/mikefarah/yq). If this file is absent, the merge step is skipped automatically.

## How It Works

```
push to non-main branch
  → caller workflow triggers
    → reusable workflow (this repo) runs in caller's context
      → checks out caller repo
      → fetches .licenserc.yaml from this repo
      → merges .licenserc.repo.yaml if present (yq)
      → skywalking-eyes header fix → auto-commit if changes
      → skywalking-eyes header check → fail if any missing
```

## Requirements

- **Self-hosted runner** with Docker (the workflow uses `runs-on: self-hosted`)
- **`permissions: contents: write`** in the caller workflow (required for auto-commit)

## Common Ignore Patterns (built-in)

| Pattern | Reason |
|---------|--------|
| `.github/**` | Workflow files |
| `.claude/**`, `.n1/**` | AI tooling config |
| `**/*.md` | Documentation |
| `**/*.json`, `**/*.toml` | Config files |
| `**/requirements*.txt` | Dependency lists |
| `**/.env*` | Environment files |
| `.gitignore`, `.gitattributes`, `.dockerignore` | Git/Docker config |
| `LICENSE`, `NOTICE` | Legal files |
| `**/venv/**`, `**/.venv/**` | Virtual environments |

## Updating the Header

To change the licence header text or common ignore patterns, edit `.licenserc.yaml` in this repo. All consuming repos pick up the change on their next workflow run — no per-repo updates needed.
