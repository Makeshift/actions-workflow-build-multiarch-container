# Arch-aware Container Build Reusable Workflow for GitHub Actions

This workflow is designed to be reusable across projects and allow for fast and efficient multi-arch container builds on our self-hosted runners.

Our self-hosted runners currently support x86_64 and aarch64 architectures. If you specify multiple architectures, the workflow will build the container on each architecture simultaneously on separate runners, then push the manifest list to the registry to combine them into a single image.

It currently supports the following architectures:
  - `linux/amd64` (x86_64)
  - `linux/arm64` (aarch64)

## Secrets

Your repository _must_ have the following secrets defined:

  - `AWS_ECR_ACCESS_KEY_ID` - The access key for a service account with push access to the ECR repository
  - `AWS_ECR_SECRET_ACCESS_KEY` - The secret key for a service account with push access to the ECR repository

These secrets are provided to all repositories under the IEA GitHub organisation by default:

  - `ACTIONS_BUILD_CONTAINER_ECR_ACCESS_KEY_ID` - Has read-only access to the container built in [the-iea/github-actions-build-container](https://github.com/the-iea/github-actions-build-container)
  - `ACTIONS_BUILD_CONTAINER_ECR_SECRET_ACCESS_KEY` - Has read-only access to the container built in [the-iea/github-actions-build-container](https://github.com/the-iea/github-actions-build-container)

## Tags

By default, images are tagged with the following (This behaviour can be changed by various [inputs](#inputs)):

  - `:latest` - If this commit is on `master` or `main`
  - `:your-branch-name` - Your branch name, with `/` replaced with `-` (e.g. `feature/my-new-feature` will become `feature-my-new-feature`)
  - `:pr-#` - The pull request number, if applicable (e.g. `pr-123`)
  - `:v1.2.3` - The tag of the commit, if applicable (e.g. `v1.2.3`)
  - `:123ABCD` - The 'short' commit hash (7 characters)
  - `:123ABCD123ABCD123ABCD123ABCD123ABCD123ABCD` - The full commit hash
  - `:20230506` - The date of the commit, in `YYYYMMDD` format

## Example Usage

```yaml
jobs:
  test_multiple_arch:
    uses: the-iea/gha-build-container/.github/workflows/build_container.yml@master
    # This causes the called workflow to inherit the secrets from the parent workflow (this one)
    secrets: inherit
    # Other vars are available, see README for details
    with:
      AWS_ECR_REPOSITORY: 099538280162.dkr.ecr.eu-west-2.amazonaws.com/iea/shared/cr/gh/bld/psh/test
      DOCKERFILE: test_container/Dockerfile
      PLATFORMS: linux/amd64,linux/arm64
```
See [.github/workflows/test_build.yml](.github/workflows/test_build.yml) for a full example.

## Inputs


## Outputs

