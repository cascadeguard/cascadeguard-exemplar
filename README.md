# cascadeguard-exemplar

Example CascadeGuard state repo. Demonstrates task-mode CLI commands and
deprecation detection against real directory layouts.

## Examples

| Example | Base image | Purpose |
|---|---|---|
| [`local/hello-world`](local/hello-world/) | `nginx` | Basic enrolled image — full task-mode CLI walkthrough |
| [`local/deprecated-openjdk`](local/deprecated-openjdk/) | `openjdk:17-alpine` | Deprecated base image — shows `DEPRECATED` finding + `eclipse-temurin` recommendation |

## Repository layout

```
.
├── local/
│   ├── hello-world/
│   │   ├── Dockerfile               # hello-world nginx image
│   │   └── index.html               # static page served by nginx
│   └── deprecated-openjdk/
│       ├── Dockerfile               # openjdk:17-alpine (deprecated — exemplar only)
│       └── README.md                # deprecation detection walkthrough
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
            └── nginx-1.27-alpine.yaml  # tracked upstream base image
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

### images enrol — add a new image

```bash
cascadeguard images --images-yaml images.yaml enrol \
  --name my-app \
  --registry ghcr.io \
  --repository cascadeguard/my-app \
  --provider github \
  --repo cascadeguard/my-app \
  --dockerfile local/my-app/Dockerfile \
  --branch main \
  --rebuild-delay 7d
```

Appends a new entry to `images.yaml`. Use `images validate` afterwards to confirm
the file is still well-formed.

### images check — inspect state files

```bash
cascadeguard images --state-dir cascadeguard/state check
```

Reads every `*.yaml` file under `cascadeguard/state/images/` and
`cascadeguard/state/base-images/`, printing digest, build, and status info.

### pipeline build — trigger a GitHub Actions build

```bash
export GITHUB_TOKEN="ghp_..."
cascadeguard pipeline build \
  --image hello-world \
  --tag latest \
  --repo cascadeguard/cascadeguard-exemplar
```

Dispatches the build workflow in the target repository.
Requires a GitHub token with `actions:write` scope.

### pipeline test — check build results

```bash
export GITHUB_TOKEN="ghp_..."
cascadeguard pipeline test \
  --image hello-world \
  --repo cascadeguard/cascadeguard-exemplar
```

Fetches the latest workflow run and prints status/conclusion.
Exits non-zero when the conclusion is `failure`.

### pipeline run — run the full lifecycle

```bash
export GITHUB_TOKEN="ghp_..."
cascadeguard pipeline run \
  --images-yaml images.yaml \
  --state-dir cascadeguard/state \
  --image hello-world \
  --tag latest \
  --repo cascadeguard/cascadeguard-exemplar
```

Runs **validate → check → build → test** in sequence. Build and test are
skipped when no `--image` is supplied.

### images status — show all image states

```bash
cascadeguard images --state-dir cascadeguard/state status
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

## Deprecated image detection

CascadeGuard detects base images that have been officially deprecated on
Docker Hub and recommends actively maintained replacements.

### Example: openjdk → eclipse-temurin

`openjdk` was deprecated in November 2022 and no longer receives security
patches. The `local/deprecated-openjdk/Dockerfile` uses `openjdk:17-alpine`
to demonstrate the detection flow:

```bash
cascadeguard scan local/deprecated-openjdk/Dockerfile
```

Expected output:

```
DEPRECATED  openjdk:17-alpine
  The 'openjdk' image is deprecated and no longer receives security updates.
  Recommended replacement: eclipse-temurin:17-alpine
  See: https://hub.docker.com/_/openjdk
```

Fix the Dockerfile by replacing the deprecated base image:

```diff
-FROM openjdk:17-alpine
+FROM eclipse-temurin:17-alpine
```

See [`local/deprecated-openjdk/README.md`](local/deprecated-openjdk/README.md)
for the full walkthrough.
