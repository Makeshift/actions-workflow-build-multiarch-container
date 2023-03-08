# Arch-aware Container Build Reusable Workflow for GitHub Actions

This workflow is designed to be reusable across projects and allow for fast and efficient multi-arch container builds on our self-hosted runners.

Our self-hosted runners currently support x86_64 and aarch64 architectures. If you specify multiple architectures, the workflow will build the container on each architecture simultaneously on separate runners, then push the manifest list to the registry to combine them into a single image.

It currently supports the following architectures:
  - `linux/amd64` (x86_64)
  - `linux/arm64` (aarch64)

## Secrets

<!-- AUTO-DOC-SECRETS:START - Do not remove or modify this section -->

|                    SECRET                     | REQUIRED |                                                                                                       DESCRIPTION                                                                                                       |
|-----------------------------------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|   ACTIONS_BUILD_CONTAINER_ECR_ACCESS_KEY_ID   |   true   |   The AWS access key ID to use to grab the container built in [the-iea/github-actions-build-container](https://github.com/the-iea/github-actions-build-container) (Provided as an organisation secret within the IEA)   |
| ACTIONS_BUILD_CONTAINER_ECR_SECRET_ACCESS_KEY |   true   | The AWS secret access key to use to grab the container built in [the-iea/github-actions-build-container](https://github.com/the-iea/github-actions-build-container) (Provided as an organisation secret within the IEA) |
|             AWS_ECR_ACCESS_KEY_ID             |   true   |                                                The AWS access key ID to use for the ECR login (must have permissions to push to the repository you're trying to push to)                                                |
|           AWS_ECR_SECRET_ACCESS_KEY           |   true   |                                              The AWS secret access key to use for the ECR login (must have permissions to push to the repository you're trying to push to)                                              |

<!-- AUTO-DOC-SECRETS:END -->

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
    uses: the-iea/gha-workflow-build-single-container/.github/workflows/workflow.yml@master
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

<!-- AUTO-DOC-INPUT:START - Do not remove or modify this section -->

|        INPUT         |  TYPE  | REQUIRED |                          DEFAULT                           |                                                                                                  DESCRIPTION                                                                                                  |
|----------------------|--------|----------|------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|  AWS_ECR_REPOSITORY  | string |   true   |                                                            |                                                                              The AWS ECR repository to push<br>the container to                                                                               |
|      BUILD_ARGS      | string |  false   |                                                            |                                                         The build arguments to pass to<br> the Docker build (Newline separated string<br>of KEY=VAR)                                                          |
|    BUILD_CONTEXT     | string |  false   |                   `"DOCKERFILE_DIRNAME"`                   | Build context used by the docker<br> builder (. to reference top of<br> Git repo, DOCKERFILE_DIRNAME to reference Dockerfile<br> directory - Either BUILD_CONTEXT or DOCKERFILE<br>or both must be specified) |
|      DOCKERFILE      | string |   true   |                                                            |                                                                                            The Dockerfile to build                                                                                            |
|      PLATFORMS       | string |  false   |                      `"linux/amd64"`                       |                                                                            The platforms to build for (comma<br>separated string)                                                                             |
| PLATFORM_TO_ARCH_MAP | string |  false   | `"{\"linux/amd64\": \"x64\", \"linux/arm64\": \"arm64\"}"` |                                                     A JSON object mapping platform to<br> architecture (Generally, you should not need<br>to change this)                                                     |
|  TAG_ALL_AS_LATEST   | string |  false   |                         `"false"`                          |                                                                    If true, tag the image as<br> latest no matter what branch we're<br>on                                                                     |
|  TAG_AS_BRANCH_NAME  | string |  false   |                          `"true"`                          |                                                                                Whether to tag the image as<br>the branch name                                                                                 |
| TAG_MASTER_AS_LATEST | string |  false   |                          `"true"`                          |                                                                  If true, tag the image as<br> latest if the branch is `master`<br>or `main`                                                                  |
|      TAG_SUFFIX      | string |  false   |                                                            |                                                                                      The suffix to append to the<br>tag                                                                                       |

<!-- AUTO-DOC-INPUT:END -->






## Outputs

<!-- AUTO-DOC-OUTPUT:START - Do not remove or modify this section -->

|    OUTPUT     |                    VALUE                    |                               DESCRIPTION                               |
|---------------|---------------------------------------------|-------------------------------------------------------------------------|
| image_sha_tag |     ${{ jobs.setup.outputs.image_sha }}     |               The SHA tag of the image<br>that was built                |
|  image_tags   | ${{ jobs.build_manifest.outputs.TAG_LIST }} | The tags of the image that<br> was built as a stringified JSON<br>array |

<!-- AUTO-DOC-OUTPUT:END -->









