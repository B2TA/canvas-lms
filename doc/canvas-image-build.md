# Building the Canvas image

This fork publishes the production Canvas image as:

```text
ghcr.io/b2ta/canvas-lms:<tag>
```

The manual GitHub Actions workflow in `.github/workflows/canvas-image.yml`
builds the two architectures on native runners and then publishes one
multi-architecture manifest:

- `linux/amd64` on an x86_64 runner
- `linux/arm64` on an ARM64 runner
- a manifest combining both images under the requested tag

Run it from the repository's **Actions** tab with **Build and publish Canvas
image**. The default tag is `latest`. The workflow also publishes a commit
tag in the form `sha-<commit>`, which is useful when the deployment should
avoid a moving tag.

The workflow passes `POSTGRES_CLIENT=18` explicitly to
`Dockerfile.production`. The production Dockerfile compiles Rails assets
during the image build, and that boot path requires database configuration.
It receives build-only placeholder values from the Dockerfile for that step;
the real database password is supplied by Compose at runtime and is never a
build secret.

The workflow uses the repository `GITHUB_TOKEN` with `packages: write` to
publish to GHCR. Publishing from this workflow links the container package to
this repository automatically. No local Docker credentials or files are used.

The standard ARM64 runner label is `ubuntu-24.04-arm`. Canvas images are large,
so if that runner runs out of disk, select an organization larger-runner label
in the manual workflow input instead. The same override is available for the
amd64 build.

For a local build, use the same production Dockerfile and build argument:

```bash
docker buildx build \
  --platform linux/arm64 \
  --build-arg POSTGRES_CLIENT=18 \
  -f Dockerfile.production \
  -t ghcr.io/b2ta/canvas-lms:local \
  --load .
```
