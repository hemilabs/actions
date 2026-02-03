# Changelog Labeler

Automatically manage changelog labels on pull requests. Adds a "required" label when the changelog hasn't been updated,
and switches to "done" once it has.

## Usage

```yaml
name: "Pull Request"

on:
  pull_request_target:
    types: [opened, synchronize, labeled, unlabeled]

permissions:
  pull-requests: write

concurrency:
  group: "changelog-${{ github.event.pull_request.number }}"
  cancel-in-progress: true

jobs:
  label:
    name: "Label"
    runs-on: ubuntu-latest
    steps:
      - name: "Update changelog labels"
        uses: hemilabs/actions/changelog-labeler@<commit-sha> # vX.Y.Z
```

## Inputs

| Input            | Description                                    | Default               |
| ---------------- | ---------------------------------------------- | --------------------- |
| `changelog-path` | Path to changelog file (relative to repo root) | `CHANGELOG.md`        |
| `label-required` | Label when changelog needs updating            | `changelog: required` |
| `label-skip`     | Label to skip changelog requirement            | `changelog: skip`     |
| `label-done`     | Label when changelog is updated                | `changelog: done`     |
| `required`       | Fail if changelog not updated                  | `true`                |

## Outputs

| Output     | Description                                  |
| ---------- | -------------------------------------------- |
| `changed`  | Whether the changelog was modified           |
| `required` | Whether a changelog update is still required |

## Permissions

Requires `pull-requests: write` to manage labels.

## Security Considerations

This action is designed for use with `pull_request_target`, which runs with write access to the base repository.

Due to this, this action takes extra care to be secure:

- Only reads PR metadata via GitHub API; never checks out or executes PR code.
- All inputs passed via environment variables, not interpolated in shell to avoid shell injections.
- Uses `grep -F` to prevent regex injection from filenames or labels.
- Only requires `pull-requests: write`.

**Recommendations**

- Use concurrency groups to prevent race conditions from multiple runs modifying labels.
- Avoid using `actions/checkout` in the same job, or use `ref: ${{ github.event.pull_request.base.sha }}` to avoid
  checking out untrusted PR code.
