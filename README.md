# hemilabs/actions

Contains reusable GitHub Actions and GitHub Actions workflows for the [Hemi Labs](https://github.com/hemilabs)'s
repositories.

## Actions

- `docker-build-push`: Build and push Docker images to Docker Hub.
- `notify-deploy-to-slack`: Sends a notification to Slack using a pre-configured webhook.
- `publish-to-npm`: Sets up Node, runs the prepublishOnly script and publishes the package to the NPM registry.
- `setup-node-env`: Sets up Node, restores NPM cache and installs dependencies.

## Reusable workflows

- `deploy-kustomize`: Deploy to Kubernetes using Kustomize.
- `docker-checks`: Locally builds a Docker image and uses `aquasecurity/trivy-action` to check for vulnerabilities.
- `docker-registry-secret`: Create a Docker Hub secret.
- `docker`: Build and push docker images.
- `js-checks`: Run standard checks as linter, code formatter; then run tests against different Node versions.
- `npm-publish`: Publish a package to NPM.

## Usage

To use the actions and workflows in this repository, add a reference as follows:

```yml
# .github/workflows/example.yml

jobs:
  job-one:
    steps:
      - uses: hemilabs/actions/setup-node-env@v1.0.0
  job-two:
    uses: hemilabs/actions/.github/workflows/js-checks.yml@v1
```

See the GitHub Actions docs for details about [how to use a public action](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#example-using-a-public-action) and how to reference a [reusable workflow](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#jobsjob_iduses).

## Release process

Once the PRs with the changes are approved, either merge the branch locally and push `main` or merge the PR in GitHub and pull `main`. Then update and push the tags:

```sh
./bump-version <major, minor, patch>
```

## Contact

If you wish to contact us, please use one of the following methods:

- [Discord server](https://discord.gg/hemixyz) - Ask questions here for the fastest response.
- Email [`support@hemi.xyz`](mailto:support@hemi.xyz)
- Email [`security@hemi.xyz`](mailto:security@hemi.xyz) - Security-related matters only.

## License

The contents of this repository are licensed under the terms of the [MIT License](LICENSE).
