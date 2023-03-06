# Arch-aware Container Build Reusable Workflow for GitHub Actions

This workflow is designed to be reusable across projects and allow for fast and efficient multi-arch container builds on our self-hosted runners.

Our self-hosted runners currently support x86_64 and aarch64 architectures. If you specify multiple architectures, the workflow will build the container on each architecture simultaneously on separate runners, then push the manifest list to the registry to combine them into a single image.

It currently supports the following architectures:
  - `linux/amd64` (x86_64)
  - `linux/arm64` (aarch64)

## Usage

For usage details, see `workflows/test_build.yml` in this repository.
