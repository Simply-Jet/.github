# Shared security check

`.github/workflows/security-check.yml` is a reusable workflow that runs on every call:

- gitleaks: secrets in the full git history (pinned binary with checksum, no licence needed)
- dependency-scan: OSV-Scanner on all lockfiles (`--allow-no-lockfiles`: a repository with no supported lockfile passes instead of failing with exit 128)
- image-scan: Trivy (vulnerabilities, fixed only, `--ignore-unfixed`), only when `scan-image` is set
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
- Trivy: `.trivyignore` (optional, used when present) with a comment above each CVE giving the reason.
- Semgrep: `// nosemgrep: <rule-id> reason: <why>` on the line.

An ignore without a reason is rejected in review.

## Pinned tools

gitleaks v8.30.1, OSV-Scanner v2.7.1 and Trivy v0.76.2 (release binaries with sha256) and Semgrep (`==1.185.0`) are pinned to exact versions. Semgrep is not hash-verified: `pip --require-hashes` needs a hashed lockfile of about 67 transitive packages, and a reusable workflow cannot read a file from this repository at run time (checkout is the caller's repo). Bump versions deliberately in a PR.

## Evidence

Each scanner was tested by calling this workflow from a throwaway branch of this repository in both modes, `report-only: true` and `false`. The scratch branches were deleted on purpose after each run, so there is no permanent PR-triggered workflow here. The run logs stay available at the links below.

| Run | Workflow commit | Result |
| --- | --- | --- |
| [42966514495](https://github.com/Simply-Jet/.github/actions/runs/42966514495), [42966736224](https://github.com/Simply-Jet/.github/actions/runs/42966736224) | `408eafa4`, `c67d9def` | Planted secret, vulnerable lockfile and code finding, no `scan-image`. Report-only: all jobs and `security-check` green. Enforce: gitleaks, dependency-scan, semgrep and `security-check` red. |
| [43011149807](https://github.com/Simply-Jet/.github/actions/runs/43011149807) | `eb3e1b95` | image-scan failed in both modes: template error on `continue-on-error` of the trivy-action step |
| [43011451242](https://github.com/Simply-Jet/.github/actions/runs/43011451242) | `e61e6078` | image-scan failed in both modes: trivy-action stops when `.trivyignore` is missing, and report-only was not honoured |
| [43012131901](https://github.com/Simply-Jet/.github/actions/runs/43012131901) | `c2634efa` | `scan-image: alpine:3.18.0`, no `.trivyignore`. Trivy found `Total: 9 (HIGH: 6, CRITICAL: 3)` in both modes. Report-only: every job and `security-check` green. Enforce: image-scan and `security-check` red; gitleaks, dependency-scan and semgrep green (no findings in this repository). |
