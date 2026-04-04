# cascadeguard-exemplar

Example CascadeGuard state repo with a hello-world nginx image. Demonstrates
every task-mode CLI command against a real directory layout.

## Repository layout

```
.
├── local/
│   └── hello-world/
│       ├── Dockerfile               # hello-world nginx image
│       └── index.html               # static page served by nginx
├── images.yaml                      # enrollment configuration
├── .cascadeguard.yaml               # CascadeGuard config (ci.platform, etc.)
├── .github/workflows/               # auto-generated CI pipelines (generate-ci)
│   ├── ci.yaml
│   ├── build-image.yaml
│   ├── scheduled-scan.yaml
│   └── release.yaml
└── cascadeguard/
    └── state/
        ├── images/
        │   └── hello-world.yaml     # application image state
        └── base-images/
            └── nginx-latest.yaml        # tracked upstream base image
```

## Prerequisites

Install the CascadeGuard CLI from the main repository:

```bash
pip install cascadeguard-tool          # or: pip install -e ../cascadeguard/app
```

## Task-mode CLI commands

All commands below assume you are in the root of this repository.

### validate — check images.yaml

```bash
cascadeguard --images-yaml images.yaml validate
```

Validates that every entry in `images.yaml` has the required fields (`name`,
`registry`, `repository`) and that source blocks include `repo` and `provider`.

### enrol — add a new image

```bash
cascadeguard --images-yaml images.yaml enrol \
  --name my-app \
  --registry ghcr.io \
  --repository cascadeguard/my-app \
  --provider github \
  --repo cascadeguard/my-app \
  --dockerfile local/my-app/Dockerfile \
  --branch main \
  --rebuild-delay 7d
```

Appends a new entry to `images.yaml`. Use `validate` afterwards to confirm
the file is still well-formed.

### check — inspect state files

```bash
cascadeguard --state-dir cascadeguard/state check
```

Reads every `*.yaml` file under `cascadeguard/state/images/` and
`cascadeguard/state/base-images/`, printing digest, build, and status info.

### build — trigger a GitHub Actions build

```bash
export GITHUB_TOKEN="ghp_..."
cascadeguard build \
  --image hello-world \
  --tag latest \
  --repo cascadeguard/cascadeguard-exemplar
```

Dispatches the build workflow in the target repository.
Requires a GitHub token with `actions:write` scope.

### test — check build results

```bash
export GITHUB_TOKEN="ghp_..."
cascadeguard test \
  --image hello-world \
  --repo cascadeguard/cascadeguard-exemplar
```

Fetches the latest workflow run and prints status/conclusion.
Exits non-zero when the conclusion is `failure`.

### pipeline — run the full lifecycle

```bash
export GITHUB_TOKEN="ghp_..."
cascadeguard \
  --images-yaml images.yaml \
  --state-dir cascadeguard/state \
  pipeline \
  --image hello-world \
  --tag latest \
  --repo cascadeguard/cascadeguard-exemplar
```

Runs **validate → check → build → test** in sequence. Build and test are
skipped when no `--image` is supplied.

### status — show all image states

```bash
cascadeguard --state-dir cascadeguard/state status
```

Prints a summary table of every application image and base image tracked
under the state directory, including version, digest, build time, and
dependency information.

### generate-ci — emit CI workflow files

```bash
cascadeguard generate-ci \
  --images-yaml images.yaml \
  --output-dir .
```

Reads `images.yaml` and emits workflow files for the configured CI platform.
Works with any supported tool — the target platform is read from
`.cascadeguard.yaml` (`ci.platform: github`). Re-run whenever you add or
remove an image.

| File | Purpose |
|---|---|
| `ci.yaml` | PR checks + push-to-main build, scan, and sign |
| `build-image.yaml` | Reusable workflow called by the other three |
| `scheduled-scan.yaml` | Nightly CVE re-scan of published images |
| `release.yaml` | Tag-triggered sign, push, and GitHub Release |

The pre-generated files in this repo were produced by running this command
against the hello-world entry in `images.yaml`.
