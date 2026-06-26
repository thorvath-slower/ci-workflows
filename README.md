# ci-workflows — shared reusable CI workflows

The single source of truth for cross-repo CI gating on the platform (CZID-310). Instead of each repo
carrying its own copy of the same scan (which drifts), every repo calls the reusable workflow here. One
definition, updated once, used everywhere.

Mirrors the `thorvath-slower/flake8-action@v2` SSOT pattern — pin callers to a tag (`@v1`).

## Available workflows

### `security.yml` — reusable security scan
gitleaks (secrets) + trivy (vuln/misconfig/secret, hard-fail HIGH/CRITICAL) + tflint + opt-in checkov.

**Call it from a repo** (`.github/workflows/security.yml`):

```yaml
name: security
on:
  push:
  pull_request:
  merge_group:
  workflow_dispatch:
    inputs:
      run_checkov:
        type: boolean
        default: false

jobs:
  security:
    uses: thorvath-slower/ci-workflows/.github/workflows/security.yml@v1
    with:
      run_checkov: ${{ inputs.run_checkov || false }}
```

- The reusable's jobs check out the **caller's** repo, so each repo keeps its own **`.trivyignore`** baseline
  (CZID-264: accept inherited findings, hard-fail on NEW).
- Public repo, so any platform repo (public or private) can call it without an org-access setting.

## Versioning

Pin to `@v1` (a moving major tag). Breaking changes bump the major. Renovate keeps the pinned tool/action
versions inside the reusable current.
