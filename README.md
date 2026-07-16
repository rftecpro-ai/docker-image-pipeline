# docker-image-pipeline

A minimal Flask "Hello World" web app, containerized with Docker and
built/pushed to Docker Hub through a GitHub Actions CI pipeline.

## Folder structure

```
.
├── .github/workflows/ci.yml   # CI pipeline: test, then build & push
├── app/
│   └── app.py                 # Flask app entrypoint
├── conftest.py                # pytest fixtures/config
├── Dockerfile                 # container build
├── requirements.txt           # runtime dependencies
├── requirements-dev.txt       # dev/test dependencies
├── tests/
│   └── test_app.py            # app tests
└── VERSION                    # image tag for the current release
```

## Local development

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements-dev.txt

python app/app.py      # runs on http://localhost:8000
pytest                 # run tests
```

## Docker

```bash
docker build -t ffribeiro/docker-image-pipeline:local .
docker run -p 8000:8000 ffribeiro/docker-image-pipeline:local
```

## CI/CD

`.github/workflows/ci.yml` runs on every push/PR to `main`:

1. **test** — installs dependencies and runs `pytest`.
2. **build-and-push** — on push to `main` only (after tests pass), builds a
   multi-arch (`linux/amd64`, `linux/arm64`) image and pushes it to Docker Hub
   as `ffribeiro/docker-image-pipeline:<VERSION>` and
   `ffribeiro/docker-image-pipeline:<commit-sha>`.

## Releasing a new version

1. When an app update is required, create a branch off `main` (e.g.
   `release/1.1`) and make the code change.
2. Bump the version in [`VERSION`](VERSION) (e.g. `1.0` → `1.1`) — this is
   what produces a new image tag. Without it, `build-and-push` just
   re-pushes the same version tag (the commit-sha tag will still differ).
3. Open a PR against `main` so tests gate it before merge.
4. Merge to `main`. CI runs `test` → `build-and-push`, pushing the new image
   to Docker Hub.
5. Verify on [Docker Hub](https://hub.docker.com/) that the new tag is
   present, and/or pull and run it locally to confirm.

### Required repository secrets

`build-and-push` needs secrets set in the GitHub repo
(**Settings → Secrets and variables → Actions**):

| Secret | Value |
|---|---|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | A Docker Hub [access token](https://hub.docker.com/settings/security) (not your password) |

These require access we don't have (your Docker Hub credentials) — someone
with the right permissions needs to create them manually before merging to
`main`, or the `build-and-push` job will fail.
