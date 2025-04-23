# Actions Security Guidelines

> Last updated: **April 24th, 2025**

## Purpose

These guidelines outline security best practices for GitHub Actions workflows and actions used by Hemi Labs.

---

## Key Principles

- **Least privilege** - Always apply the principle of least privilege to workflows, actions and secrets. Limit the scope
  of permissions and access to only what is strictly necessary.

- **Minimise exposure** - Use minimally scoped secrets and access tokens, reduce the amount of secrets accessible by a
  workflow, and avoid logging secrets or writing them to files.

- **Integrity and trust** - Use only trusted actions and dependencies, verify their integrity and regularly audit and
  update them.

---

## 1. Workflows

## 1.1. Secrets

- **Never hardcode secrets** - Always store sensitive data in GitHub Actions secrets and reference them securely within
  workflows.

- **Limit secret access** - Only expose secrets to the workflows that need them. When possible, avoid using global
  secrets if specific jobs do not require them. Repository-level secrets or environment secrets can be created to
  further limit access to the secrets, which can then have lower scope.

- **Rotate secrets regularly** - Secrets should be rotated periodically to reduce their lifetime and the potential risk
  of a compromised secret being used.

- **Mask secrets** - Ensure secrets are never logged or exposed during workflow executions. GitHub automatically masks
  secrets, however they should still never be printed or written to files.

## 1.2. `GITHUB_TOKEN` permissions

- **Use minimal permissions** - Grant the least amount of permissions required to perform a job. For example, jobs that
  build and test a project rarely require any permissions other than `contents: read`.

- **Always override the default permissions** - Workflows should always specify the permissions for a job or workflow.
  The base permissions should always be `contents: read`.

Use the `permissions:` key at the workflow or job level to explicitly declare the permissions needed. Avoid relying on
the default GitHub token permissions.

A full list of permissions can be found
on [GitHub's documentation: Controlling permissions for GITHUB_TOKEN](https://docs.github.com/en/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token).

```yaml
permissions:
  contents: read
  id-token: write # Only when using OIDC.
```

## 1.3. Actions

- **Use trusted actions** - Prefer actions from trusted sources, such as official GitHub actions or those hosted in the
  `hemilabs/actions` repository. Always review third-party (including GitHub) actions prior to using them.

- **Check transitive dependencies** - Actions, especially composite actions, may depend on other actions. Such
  dependencies should be reviewed (usually possible by viewing the `action.yml` file within the action repository) and
  ensured that all transitive dependencies are pinned to commit hashes.

- **Pin actions to commit hashes** - Third-party actions **must be pinned to commit hashes** to avoid unexpected
  changes. Only commit hashes for released tags should be used, and a comment should be added to the end of the line to
  document the release that is in use. **Never pin to `latest`, branches or tags.** The only exception to this rule is
  actions within `hemilabs/actions`, which should be pinned to a `vX.X` or `vX.X.X` tag.

```yaml
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

---

## 2. Actions

## 2.1. Secrets

- **Consuming secrets** - If an action requires secrets, ensure they are passed securely through GitHub Actions secrets.
- **Mask sensitive data** - Ensure sensitive data is always masked in logs to avoid accidental exposure.

## 2.2. Design

- **Avoid external dependencies unless necessary** - When creating a custom action, avoid unnecessary dependencies to
  minimise possible attack surface.

- **Pin dependencies** - For composite actions, all third-party actions **must be pinned to commit hashes**. Similarly,
  all JavaScript dependencies should be pinned to avoid unexpected and unreviewed changes.

- **Use minimal and pinned DOcker images** - If using Docker-based actions:
  - Use minimal, actively maintained Docker image based on a trusted and well-known base (e.g. `scratch`, `alpine` or
    `debian:slim`) to reduce the surface area and potential vulnerabilities.
  - Pin Docker images by `sha256` digest rather than tags to ensure immutability and unexpected changes.

```Dockerfile
FROM alpine:3.21.3@sha256:a8560b36e8b8210634f77d9f7f9efd7ffa463e380b75e2e74aff4511df3ef88c
```

---

## 3. General Best Practices

### 3.1. Monitoring

- **Audit logs** - Regularly audit the GitHub Actions logs for any signs of unusual behaviour or errors, especially in
  workflows that involve sensitive data or production deployments.

### 3.2. Minimise workflow triggers

- **Control workflow triggers** - Be specific with workflow triggers. For example, avoid unnecessary workflow executions
  for every push to any branch. Limit triggers to the events that are necessary for the workflow.

```yaml
on:
  push:
    branches: ["main"]
```

- **Restrict sensitive workflow runs** - Restrict sensitive workflows, especially deployment workflows, to only trusted
  branches with strict access control to prevent malicious triggers from unauthorised users. Additionally, where
  possible, use GitHub environments with rulesets configured to apply additional security rules for the deployment.

### 3.3. Dependency management

- **Automate dependency scanning** - When appropriate, use tools such as Dependabot or Renovate bot to automatically
  monitor dependencies. This allows teams to be notified when a dependency update is available, so that the update can
  be reviewed and then used.

### 3.4 Script safety

- **Avoid untrusted input in scripts** - Never interpolate untrusted input directly into shell commands. Always validate
  and sanitise inputs from users, pull requests, or other sources before use.

- **Beware of `${{ }}` expression injection** - Inputs from `${{ github.event.* }}` and other contexts may be
  attacker-controlled (e.g. pull request titles, branch names, commit messages, inputs, etc.). Do not interpolate these
  directly into shell scripts without validation.

```yaml
# ❌ Vulnerable to injection
run: |
  echo "PR title: ${{ github.event.pull_request.title }}"

# ✅ Safer
env:
  PR_TITLE: "${{ github.event.pull_request.title }}"
run: |
  echo "PR title: $PR_TITLE"
```

- **Avoid `eval`, backticks, or unescaped interpolation** - Avoid using `eval`, backticks, or dynamically constructed
  commands unless absolutely necessary. These patterns are highly prone to injection vulnerabilities.

- **Avoid `curl | sh`-like patterns** - Instead, download the scripts and preferably validate the checksum prior to
  execution.

```yaml
# ❌ Dangerous
run: curl -sSL https://example.com/install.sh | sh

# ✅ Safer
run: |
  curl -sSL https://example.com/install.sh -o install.sh
  echo "$INSTALLSH_CHECKSUM" | sha256sum --check -
  chmod +x install.sh
  ./install.sh
```

---

## 4. Reporting issues

If you notice a potential security issue in an action or workflow, raise it immediately with your team lead or via
internal security channels. Quick detection and response help us protect from potential attacks.

## 5. Collaboration

- **Collaborate with SecOps** - Work closely with Security Operations (SecOps), currently acting as the security
  point-of-contact, to ensure workflows and actions comply with security best practices.

- **Feedback and improvements** - Continuous improvement of our security practices is crucial. If you have feedback or
  suggestions for these guidelines, please feel free to create an issue or start a discussion internally.
