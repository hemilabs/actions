# Auto PR for Allowed Branches

This composite GitHub Action checks if the current branch is part of an allowed list and creates a Pull Request (PR) automatically if it matches.

## Use cases

- Automating PR creation from branches triggered by FluxCD image automation.
- Enforcing allowed branch controls for automated merges.
- Simplifying the flow of image updates into main or staging branches.

## Inputs

| Input Name        | Description                                                   | Required | Example                        |
|-------------------|---------------------------------------------------------------|----------|---------------------------------|
| `allowed-branches`| Space-separated list of allowed branch names                  | ✅        | `subgraph-api token-princes`   |
| `pr-base-branch`  | The base branch to merge into                                 | ✅        | `main`                         |
| `pr-title`        | Title of the Pull Request                                     | ✅        | `Automated PR: Update main`    |
| `pr-body`         | Body content of the Pull Request                              | ✅        | `This is an automated PR.`     |
| `reviewers`       | Comma-separated list of GitHub usernames for review requests  | ✅        | `user1,user2`                  |

## Usage Example

```yaml
jobs:
  auto-pr:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Auto PR for allowed branches
        uses: hemilabs/actions/auto-pr@main
        with:
          allowed-branches: "token-prices-staging token-prices-cron-staging"
          pr-base-branch: "main"
          pr-title: "Merge $BRANCH_NAME into main" 
          pr-body: "*An automated PR* — contact DevOps if needed"
          reviewers: "dhidalgX,gabmontes,gndelia"
