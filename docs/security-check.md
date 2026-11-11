# Shared security check

`.github/workflows/security-check.yml` is a reusable workflow that runs on every call:

- gitleaks: secrets in the full git history (pinned binary with checksum, no licence needed)
- dependency-scan: OSV-Scanner on all lockfiles (`--allow-no-lockfiles`: a repository with no supported lockfile passes instead of failing with exit 128)
- image-scan: Trivy, only when `scan-image` is set
- semgrep: code patterns (`--config=p/default`, since `auto` requires metrics). `p/default` is a registry rule set that changes over time, so a new finding can appear without a code change.
- `security-check`: the aggregate job. Make this the required status check.

No secrets are used, so it works on fork pull requests. Do not add private values to this repository, it is public.

## Inputs

| Input | Default | Meaning |
| --- | --- | --- |
| `scan-image` | empty | Image ref for Trivy. Must be pullable without credentials. Empty skips the image scan. |
| `fail-on` | `high` | `critical`, `high`, `medium` or `low`. Applies to Trivy and Semgrep. OSV fails on any finding. An invalid value fails the check in every mode, including report-only. |
| `report-only` | `true` | The scan steps do not block. This covers findings AND scanner errors (crash, vulnerability DB or image fetch failure): both show green, so read the scan step logs. Install, checksum and `fail-on` errors still fail. Set `false` once the baseline is clean. |

## Calling it

Pin to a commit SHA of this repository, never a branch or tag. Replace `<40-char-sha>` with the commit of this repository you want, i.e. the merge commit of this PR once merged (`git ls-remote https://github.com/Simply-Jet/.github main`). A SHA cannot be embedded here, because a commit cannot contain its own hash; the placeholder is intentional. To test before merge, use the PR head SHA (`gh pr view 1 -R Simply-Jet/.github --json headRefOid -q .headRefOid`).

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
    uses: Simply-Jet/.github/.github/workflows/security-check.yml@<40-char-sha> # <short note>
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

## Pinned tools

gitleaks, OSV-Scanner (binary and sha256), Trivy (`version: v0.76.2`) and Semgrep (`==1.185.0`) are pinned to exact versions. Semgrep is not hash-verified: `pip --require-hashes` needs a hashed lockfile of about 67 transitive packages, and a reusable workflow cannot read a file from this repository at run time (checkout is the caller's repo). Bump versions deliberately in a PR.
