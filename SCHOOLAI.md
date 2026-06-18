# SchoolAI

This is SchoolAI's fork of [Unstructured-IO/unstructured-api](https://github.com/Unstructured-IO/unstructured-api). We run it internally for document partitioning in the web-app RAG pipeline.

There is no CI/CD in this repo. Image builds are infrequent enough to do manually from a local machine.

## Image

Staging and production pull:

`us-central1-docker.pkg.dev/schoolai-global/docker/unstructured-api:latest-amd64`

That reference lives in web-app at `ops/unstructured-api/values-global.yaml`. The cluster runs **amd64** only (`kubernetes.io/arch: amd64`).

## Build and publish

Prerequisites:

- Docker with buildx
- `gcloud` CLI with access to the `schoolai-global` project

```bash
gcloud auth login
gcloud auth configure-docker us-central1-docker.pkg.dev
```

From the repo root, build for linux/amd64 and push:

```bash
IMAGE=us-central1-docker.pkg.dev/schoolai-global/docker/unstructured-api

docker buildx build \
  --platform linux/amd64 \
  --build-arg PIPELINE_PACKAGE=general \
  -t "${IMAGE}:latest-amd64" \
  --push \
  .
```

On Apple Silicon, `--platform linux/amd64` is required so the image runs on GKE. The build may be slow because it cross-compiles.

Optional versioned tag (useful for tracking what was deployed):

```bash
VERSION="$(date +'%Y_%m_%d-%H_%M')-$(git rev-parse --short HEAD)"
docker buildx build \
  --platform linux/amd64 \
  --build-arg PIPELINE_PACKAGE=general \
  -t "${IMAGE}:${VERSION}-amd64" \
  -t "${IMAGE}:latest-amd64" \
  --push \
  .
```

## Roll out

Pushing `latest-amd64` does not restart running pods by itself. To pick up the new image in staging:

1. Trigger a web-app staging deploy (e.g. push to web-app `main` that touches `ops/unstructured-api/**`, or run the webapp staging deploy workflow manually), or
2. Restart the `unstructured-api` deployment in the `webapp` namespace on the target cluster.

## Local development

To build and run locally without publishing:

```bash
make docker-build
make docker-start-api
```

The API is available at `http://localhost:8000`. See the main [README](README.md) for API usage examples.

web-app's `docker-compose.yml` can also run the published image directly for local integration testing.
