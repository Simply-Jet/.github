# Shared security check

`.github/workflows/security-check.yml` is a reusable workflow that runs on every call:

- gitleaks: secrets in the full git history (pinned binary with checksum, no licence needed)
- dependency-scan: OSV-Scanner on all lockfiles
- image-scan: Trivy, only when `scan-image` is set
- semgrep: code patterns (`--config=auto`)
- `security-check`: the aggregate job. Make this the required status check.

No secrets are used, so it works on fork pull requests. Do not add private values to this repository, it is public.

## Inputs

| Input | Default | Meaning |
| --- | --- | --- |
| `scan-image` | empty | Image ref for Trivy. Must be pullable without credentials. Empty skips the image scan. |
| `fail-on` | `high` | `critical`, `high`, `medium` or `low`. Applies to Trivy and Semgrep. OSV fails on any finding. |
| `report-only` | `true` | Findings are logged but do not block. Set `false` once the baseline is clean. |

## Calling it

Pin to a commit SHA of this repository, never a branch or tag.

```yaml
name: security
on:
  pull_request:
  push:
    branches: [main]
permissions:
  contents: read
jobs:
  security-check:
    uses: Simply-Jet/.github/.github/workflows/security-check.yml@<40-char-sha> # <short note, e.g. main 2026-11-10>
    with:
      report-only: true
```

The job id `security-check` gives the status check name `security-check / security-check`.

## Ignoring a finding

An accepted finding must be ignored in the repository with a written reason, and reviewed in the PR:

- gitleaks: `.gitleaksignore` (fingerprint per line) or `.gitleaks.toml` allowlist, with a comment line above giving the reason.
- OSV: `osv-scanner.toml` `[[IgnoredVulns]]` with `id`, `ignoreUntil` and `reason`.
- Trivy: `.trivyignore` with a comment above each CVE giving the reason.
- Semgrep: `// nosemgrep: <rule-id> reason: <why>` on the line.

An ignore without a reason is rejected in review.
