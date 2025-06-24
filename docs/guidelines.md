# Actions Guidelines

This document outlines a shared set of guidelines for creating and maintaining GitHub Actions files for Hemi Labs.

---

## 1. Reusable actions and workflows

The [`hemilabs/actions`](https://github.com/hemilabs/actions/) repository is the primary repository for GitHub Actions
and workflows.

- Workflows or actions used in multiple repositories should be placed here, unless there's a specific reason not
  to (e.g., internal, or environment-specific logic).
- Aim to make actions and workflows configurable and well-documented to allow reuse.
- Breaking changes to shared actions or workflows should be discussed prior to being made to avoid unexpected issues in
  consuming projects.

### 1.1. Versioning

The `hemilabs/actions` repository uses tagged releases, following Semantic Versioning. Consumers of actions or workflows
from `hemilabs/actions` should pin the action/workflows to a `vX.X` or `vX.X.X` tag to avoid unexpected or unreviewed changes.

When breaking changes may need to be made to a certain action or workflow, an appropriate version bump must be made to
avoid disrupting existing consumers.

### 1.2. Reviews

All changes made to the `hemilabs/actions` repository should be reviewed by two team members from the team that owns
the updated files. When multiple teams are owners, at least one reviewer from each team should approve the changes to
ensure alignment and avoid surprises.

Additionally, for security purposes, all changes require a sign-off from a member of the `@hemilabs/devops` or
`@hemilabs/secops` teams.

### 1.3. Ownership

Each team should be responsible for maintaining the workflows or actions they contribute to this repository.
When contributing workflows or actions, add your team as a [code owner](../.github/CODEOWNERS), unless agreed upon. This
helps ensure appropriate review coverage and accountability.

## 2. Security

Please see [security guidelines](security.md).

## 3. Style and formatting

All workflows and actions must follow consistent formatting rules to improve readability and maintainability.
Consistency also helps reduce unnecessary diff noise during reviews.

- Format YAML files using [`prettier`](https://prettier.io/) with the default config.
- Use the `.yml` extension for YAML files (not `.yaml`).
- Indent using 2 spaces consistently.
- Add inline comments where logic may not be obvious.

## 4. Workflows

- Repository-specific workflows: `.github/workflows/`
- Workflow templates for reuse: `workflow-templates/`
- Reusable workflows should be added to the https://github.com/hemilabs/actions/ repository.

Refer to [Reusable actions and workflows](#1-reusable-actions-and-workflows) for reusable workflows.

### 4.1. File name

Use lowercase kebab-case file names with the `.yml` extension. Keep names short but descriptive.

The file name should roughly match the workflow name.

**Examples:**

- `go.yml`: Build, lint and test a Go project.
- `js.yml`: Build, lint and test a JavaScript project.
- `npm-publish.yml`: Publish an NPM project.
- `kustomize-deploy.yml`: Deploy using Kustomize.

### 4.2. Workflow Name

Workflow names should be a short, clear label - usually a noun (e.g. "Go" or "JavaScript") - that describes the type of
project or task the workflow is for.

```yaml
name: "Go"
name: "JavaScript"
name: "Deploy"
```

Workflow names appear with the job name in the GitHub UI as `<workflow> / <job>`, so both should be readable and
descriptive together.

**Good examples:**

- `Go / Build`
- `JavaScript / Lint`
- `Deploy / Staging`

### 4.3. Jobs

Job names should normally be verbs to describe the action being taken.

**Examples:**

- `Build` - Builds the project.
- `Test` - Runs tests.
- `Deploy` - Deploys to an environment.
- `Lint` - Runs linters.
- `Publish` - Publish a package.

For context-specific jobs (e.g. when using matrix), consider using parentheses:

- `Build (${{ env.GO_VERSION }})` -> `Go / Build (1.24.2)`
- `Deploy (staging)`

### 4.5. Steps

Jobs consist of steps, each of which should be:

- Clear in intent.
- Named descriptively.
- Ordered logically.
- Commented if they contain non-obvious logic.

It is good practice to use step names that clearly describe their function, improving maintainability and readability.

Step keys should be ordered as:

- `name`
- `id`
- `if`
- `shell`
- `working-directory`
- `env`
- `uses` / `run`
- `with`

Ordering keys consistently improves readability and reduces unnecessary diffs during reviews.

https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions
https://docs.github.com/en/actions/sharing-automations/creating-actions/metadata-syntax-for-github-actions

```yaml
- name: "Checkout repository"
  uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
  with:
    path: "./example/"

- name: "Read version"
  id: version
  run: |
    VERSION=$(cat version)
    echo "version=$VERSION" >> "$GITHUB_OUTPUT"

- name: "Install dependencies"
  working-directory: "example/"
  env:
    GO_VERSION: "1.24.x"
    TEST_VERSION: "${{ steps.version.outputs.version }}"
  run: make deps
```

## 5. Actions

- Repository-specific actions: `.github/actions/`
- Reusable actions should be added to the https://github.com/hemilabs/actions/ repository.

Refer to [Reusable actions and workflows](#1-reusable-actions-and-workflows) for reusable actions.

### 5.1. Composite Actions

When multiple steps are commonly reused, it may be helpful to bundle them into a composite action.

- Prefer composite actions for reusing workflow logic.
- Keep composite actions minimal and focused on a single responsibility, for example, installing a tool or setting up an
  environment.
- Document inputs, outputs, and usage clearly in a `README.md` file.

### 5.2. Custom Actions

When it is necessary to implement functionality beyond what is possible in a workflow or composite action, a custom
JavaScript or Docker-based action could be created.

Custom actions introduce more maintenance overhead and complexity (especially Docker-based). Use only when composite
actions are insufficient.

## 6. Collaboration

To encourage clarity and consistency across all actions and workflows:

- **Propose early, discuss openly** - When introducing or updating shared actions or workflows, open a pull request
  early and use it as a space for open discussion.
- **Align on patterns** - Where multiple valid approaches exist, follow established patterns unless there's a strong
  and documented reason to diverge.
- **Add documentation** - Ensure changes and usage are well-documented, making it easy for teams to understand what
  changed, why it changed, and how to adopt it.

These guidelines are intended to support consistency and maintainability across teams. Feedback is welcome, and
improvements to these guidelines are always encouraged.

To suggest changes to these guidelines, feel free to open an issue, pull request, or start a discussion.
