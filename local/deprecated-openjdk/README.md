# deprecated-openjdk — Deprecated Base Image Exemplar

This example demonstrates CascadeGuard's deprecated base image detection.

## What this shows

`openjdk` was officially deprecated on Docker Hub in November 2022. The image
no longer receives security updates. The maintained successor is
[`eclipse-temurin`](https://hub.docker.com/_/eclipse-temurin).

When CascadeGuard scans this Dockerfile it reports a `DEPRECATED` finding:

```
DEPRECATED  openjdk:17-alpine
  The 'openjdk' image is deprecated and no longer receives security updates.
  Recommended replacement: eclipse-temurin:17-alpine
  See: https://hub.docker.com/_/openjdk
```

## Running the scan

From the root of this repository:

```bash
cascadeguard scan local/deprecated-openjdk/Dockerfile
```

Expected output includes a `DEPRECATED` finding with the recommended
`eclipse-temurin` replacement and a link to the Docker Hub deprecation notice.

## Fixing the finding

Replace the `FROM` line in the Dockerfile:

```diff
-FROM openjdk:17-alpine
+FROM eclipse-temurin:17-alpine
```

Re-run the scan to confirm the finding is resolved.

## Why eclipse-temurin?

`eclipse-temurin` is the Eclipse Adoptium distribution of OpenJDK — the
official, actively maintained successor to `openjdk` on Docker Hub. It
receives regular security updates and follows the same versioning scheme.
