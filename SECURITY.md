# Security Policy

## Reporting a vulnerability

If you discover a security vulnerability in any Simply Jet repository, please report it privately to **it@simply-jet.ch** instead of opening a public issue.

Include in your report:
- A description of the vulnerability and its potential impact
- Steps to reproduce the issue
- The affected repository, file path, or endpoint
- Any suggested mitigation

We aim to acknowledge reports within **2 business days** and provide a remediation timeline within **5 business days**.

## Supported versions

Security patches apply to the current `main` branch of each repository. Older tagged releases are not retroactively patched unless the vulnerability is critical and the affected version is still running in production.

## Coordinated disclosure

We follow responsible disclosure principles. Please give us reasonable time to remediate before publishing details. Reporters will be credited in release notes unless anonymity is preferred.

## Scope

This policy covers all repositories under the [Simply-Jet GitHub organisation](https://github.com/Simply-Jet). For issues in third-party dependencies, report to the upstream maintainers. Dependency vulnerabilities affecting our deployments are monitored via Dependabot and triaged internally.

## Out of scope

- Issues that require physical access to a user's device
- Reports based solely on missing security headers without demonstrable impact
- Findings from automated scanners without a reproducible exploit
- Vulnerabilities in third-party services we do not control (Stripe, AWS, etc.)

## Acknowledgement

Researchers who help improve our security posture in good faith may be acknowledged in release notes or on this page, with their consent.
