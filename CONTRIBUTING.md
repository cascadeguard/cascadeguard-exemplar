# Contributing to cascadeguard-exemplar

Thank you for your interest in the CascadeGuard exemplar repository!

## Process

The full CascadeGuard SDLC — covering feature lifecycle, issue management, PR process, testing requirements, and the definition of done — lives in:

**[cascadeguard-docs/docs/sdlc.md](https://github.com/cascadeguard/cascadeguard-docs/blob/main/docs/sdlc.md)**

## What Lives Here

This repo is a reference implementation showing how to integrate CascadeGuard into a project:

- `images.yaml` — example image catalog following the CascadeGuard schema
- `cascadeguard/` — CascadeGuard configuration and scan output
- `.cascadeguard.yaml` — project-level CascadeGuard configuration

## Local Development

```bash
# Run a local scan using the CascadeGuard CLI
cascadeguard scan

# Validate images.yaml against the schema
cascadeguard validate
```

## Branch Naming

Use `<identifier>/<short-description>`:
- `CAS-42/update-openjdk-exemplar`
- `feat/add-python-example`

## Security

Report vulnerabilities via the [security policy](https://github.com/cascadeguard/cascadeguard/blob/main/SECURITY.md). Do not open public issues for security bugs.
