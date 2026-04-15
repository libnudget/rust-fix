# Rust Fix

Auto-fix Rust code with cargo fmt and clippy.

## What it does

- Runs `cargo fmt` to format code
- Runs `cargo clippy --fix` to apply auto-fixes
- Commits and pushes changes

## Trigger

Via comment on PR:
- `/rust-fix fmt` - format only
- `/rust-fix clippy` - clippy only
- `/rust-fix all` - both fmt and clippy
- `/confirm` - confirm to execute

## Usage

```yaml
name: Rust Auto-Fix Bot

on:
  issue_comment:
    types: [created]

jobs:
  rust-fix:
    if: github.event.issue.pull_request && contains(github.event.comment.body, '/rust-fix')
    runs-on: ubuntu-latest
    steps:
      - uses: libnudget/rust-fix@v1
        with:
          fixes: all
```

## Trigger Commands

```bash
# In a PR comment
/rust-fix fmt      # Just format
/rust-fix clippy  # Just clippy
/rust-fix all     # Both
/confirm         # Execute after confirmation
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| fixes | `fmt`, `clippy`, or `all` | all |

## Example

**Comment:** `/rust-fix all`
**Bot:** Applies cargo fmt and clippy fixes, commits changes

## Flow

1. User sends `/rust-fix <type>`
2. Bot confirms with `/confirm`
3. User sends `/confirm`
4. Bot applies fixes and commits

## License

MIT