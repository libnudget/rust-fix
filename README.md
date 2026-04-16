# Rust Fix

Auto-fix Rust code with cargo fmt and clippy.

## What it does

| Step | Command | Description |
|------|---------|-------------|
| 1 | `cargo fmt` | Format code |
| 2 | `cargo clippy --fix` | Apply clippy autofixes |

## Before / After Examples

### Unused Import
```rust
// Before
use std::io::Read;

// After
// (unused import removed automatically)
```

### Missing Derive
```rust
// Before
struct Config { name: String }

// After
#[derive(Debug)]
struct Config { name: String }
```

### Formatting
```rust
// Before
fn foo(){let x=1;}

// After
fn foo() {
    let x = 1;
}
```

### Clippy Autofixes (examples)
- `redundant_field_names` → removes duplicate field names
- `redundant_static_lifetime` → removes unnecessary `'static`
- `unnecessary_cast` → removes unnecessary casts
- `unused_variables` → prefixes with `_`

## Workflow Setup

```yaml
# .github/workflows/rust-auto-fix.yml
name: Rust Auto-Fix Bot

on:
  issue_comment:
    types: [created]

jobs:
  rust-auto-fix-request:
    if: github.event.issue.pull_request && contains(github.event.comment.body, '/rust-fix')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - name: Check fix type
        id: check
        run: |
          comment="${{ github.event.comment.body }}"
          fixes=""
          if [[ "$comment" == *"/rust-fix all"* ]]; then fixes="all"
          elif [[ "$comment" == *"/rust-fix clippy"* ]]; then fixes="clippy"
          elif [[ "$comment" == *"/rust-fix fmt"* ]]; then fixes="fmt"
          fi
          echo "fixes=$fixes" >> $GITHUB_OUTPUT

      - name: Confirm request
        if: steps.check.outputs.fixes != ''
        uses: actions/github-script@v7
        with:
          script: |
            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `Applying: ${{ steps.check.outputs.fixes }}`
            });

  rust-auto-fix-execute:
    if: github.event.issue.pull_request && contains(github.event.comment.body, '/confirm')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v6
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          fetch-depth: 0
      - name: Checkout PR
        run: gh pr checkout ${{ github.event.issue.number }}
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - name: Apply fix
        uses: libnudget/rust-fix@v1
        with:
          fixes: all
```

## Trigger Commands

```bash
# Comment on PR
/rust-fix fmt      # Format only
/rust-fix clippy  # Clippy fixes only
/rust-fix all     # Both (default)
/confirm          # Execute after request
```

## Inputs

| Input | Description | Default | Options |
|-------|-------------|---------|---------|
| fixes | Type of fixes to apply | all | fmt, clippy, all |

## Flow

```
1. User comments: /rust-fix clippy
   → Bot replies: Applying: clippy

2. User comments: /confirm
   → Workflow runs:
      - cargo fmt
      - cargo clippy --fix
      - git commit & push
```

## Notes

- Clippy fixes are limited to lints with `#[clippy::fix]` attribute
- Not all clippy warnings have autofixes (e.g., `needless_range_loop`)
- Workflow requires `workflow_dispatch: inputs:` for manual triggers

## License

MIT
